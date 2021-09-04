---
title: Suricata规则重载问题解决
tags:
  - - Suricata
  - - rules
categories:
  - [项目组, Suricata]
date: 2021-08-25 14:01:46
---
> [Suricata+DPDK项目](https://github.com/lukashino/suricata/tree/integrate-dpdk-runmode-v5)中，使用Suricata自带的规则重载方式[[Suricata规则重载]]进行规则重载会失败。
> 正在考虑使用何种方式进行处理。

## 项目检索
`delayed_detect`参数比较重要，一会用的到
`suricata.c`
SuricataMain->PostRunStartedDetectSetup->SuricataMainLoop
```c
if (sigusr2_count > 0) {
            if (!(DetectEngineReloadIsStart())) {
                DetectEngineReloadStart();
                DetectEngineReload(suri);
                DetectEngineReloadSetIdle();
                sigusr2_count--;
            }

        } else if (DetectEngineReloadIsStart()) {
            DetectEngineReload(suri);
            DetectEngineReloadSetIdle();
        }
```
PostRunStartedDetectSetup注册内容为
```c
UtilSignalHandlerSetup(SIGUSR2, SignalHandlerSigusr2);
```
SignalHandlerSigusr2是增加计数sigusr2_count。
```c
static void SignalHandlerSigusr2(int sig)
{
    if (sigusr2_count < 2)
        sigusr2_count++;
}
```

`detect-engine.c`
```c
/* tell main to start reloading */
int DetectEngineReloadStart(void)
{
    int r = 0;
    SCMutexLock(&detect_sync.m);
    if (detect_sync.state == IDLE) {
        detect_sync.state = RELOAD;
    } else {
        r = -1;
    }
    SCMutexUnlock(&detect_sync.m);
    return r;
}

/** \brief Reload the detection engine
 *
 *  \param filename YAML file to load for the detect config
 *
 *  \retval -1 error
 *  \retval 0 ok
 */
int DetectEngineReload(const SCInstance *suri)
{
    DetectEngineCtx *new_de_ctx = NULL;
    DetectEngineCtx *old_de_ctx = NULL;

    char prefix[128];
    memset(prefix, 0, sizeof(prefix));

    SCLogNotice("rule reload starting");

    if (suri->conf_filename != NULL) {
        snprintf(prefix, sizeof(prefix), "detect-engine-reloads.%d", reloads++);
        if (ConfYamlLoadFileWithPrefix(suri->conf_filename, prefix) != 0) {
            SCLogError(SC_ERR_CONF_YAML_ERROR, "failed to load yaml %s",
                    suri->conf_filename);
            return -1;
        }

        ConfNode *node = ConfGetNode(prefix);
        if (node == NULL) {
            SCLogError(SC_ERR_CONF_YAML_ERROR, "failed to properly setup yaml %s",
                    suri->conf_filename);
            return -1;
        }
#if 0
        ConfDump();
#endif
    }

    /* get a reference to the current de_ctx */
    old_de_ctx = DetectEngineGetCurrent();
    if (old_de_ctx == NULL)
        return -1;
    SCLogDebug("get ref to old_de_ctx %p", old_de_ctx);
    DatasetReload();

    /* only reload a regular 'normal' and 'delayed detect stub' detect engines */
    if (!(old_de_ctx->type == DETECT_ENGINE_TYPE_NORMAL ||
          old_de_ctx->type == DETECT_ENGINE_TYPE_DD_STUB))
    {
        DetectEngineDeReference(&old_de_ctx);
        SCLogNotice("rule reload complete");
        return -1;
    }

    /* get new detection engine */
    new_de_ctx = DetectEngineCtxInitWithPrefix(prefix);
    if (new_de_ctx == NULL) {
        SCLogError(SC_ERR_INITIALIZATION, "initializing detection engine "
                "context failed.");
        DetectEngineDeReference(&old_de_ctx);
        return -1;
    }
    if (SigLoadSignatures(new_de_ctx,
                          suri->sig_file, suri->sig_file_exclusive) != 0) {
        DetectEngineCtxFree(new_de_ctx);
        DetectEngineDeReference(&old_de_ctx);
        return -1;
    }
    SCLogDebug("set up new_de_ctx %p", new_de_ctx);

    /* add to master */
    DetectEngineAddToMaster(new_de_ctx);

    /* move to old free list */
    DetectEngineMoveToFreeList(old_de_ctx);
    DetectEngineDeReference(&old_de_ctx);

    SCLogDebug("going to reload the threads to use new_de_ctx %p", new_de_ctx);
    /* update the threads */
    DetectEngineReloadThreads(new_de_ctx);
    SCLogDebug("threads now run new_de_ctx %p", new_de_ctx);

    /* walk free list, freeing the old_de_ctx */
    DetectEnginePruneFreeList();

    DatasetPostReloadCleanup();

    DetectEngineBumpVersion();

    SCLogDebug("old_de_ctx should have been freed");

    SCLogNotice("rule reload complete");
    return 0;
}

/* main thread sets done when it's done */
void DetectEngineReloadSetIdle(void)
{
    SCMutexLock(&detect_sync.m);
    detect_sync.state = IDLE;
    SCMutexUnlock(&detect_sync.m);
}
```
主要切换功能在`DetectEngineReloadThreads`
```c
/** \internal
 *  \brief Update detect threads with new detect engine
 *
 *  Atomically update each detect thread with a new thread context
 *  that is associated to the new detection engine(s).
 *
 *  If called in unix socket mode, it's possible that we don't have
 *  detect threads yet.
 *
 *  \retval -1 error
 *  \retval 0 no detection threads
 *  \retval 1 successful reload
 */
static int DetectEngineReloadThreads(DetectEngineCtx *new_de_ctx)
{
    SCEnter();
    uint32_t i = 0;

    /* count detect threads in use */
    uint32_t no_of_detect_tvs = TmThreadCountThreadsByTmmFlags(TM_FLAG_DETECT_TM);
    /* can be zero in unix socket mode */
    if (no_of_detect_tvs == 0) {
        return 0;
    }

    /* prepare swap structures */
    DetectEngineThreadCtx *old_det_ctx[no_of_detect_tvs];
    DetectEngineThreadCtx *new_det_ctx[no_of_detect_tvs];
    ThreadVars *detect_tvs[no_of_detect_tvs];
    memset(old_det_ctx, 0x00, (no_of_detect_tvs * sizeof(DetectEngineThreadCtx *)));
    memset(new_det_ctx, 0x00, (no_of_detect_tvs * sizeof(DetectEngineThreadCtx *)));
    memset(detect_tvs, 0x00, (no_of_detect_tvs * sizeof(ThreadVars *)));

    /* start the process of swapping detect threads ctxs */

    /* get reference to tv's and setup new_det_ctx array */
    SCMutexLock(&tv_root_lock);
    for (ThreadVars *tv = tv_root[TVT_PPT]; tv != NULL; tv = tv->next) {
        if ((tv->tmm_flags & T™M_FLAG_DETECT_TM) == 0) {
            continue;
        }
        for (TmSlot *s = tv->tm_slots; s != NULL; s = s->slot_next) {
            TmModule *tm = TmModuleGetById(s->tm_id);
            if (!(tm->flags & TM_FLAG_DETECT_TM)) {
                continue;
            }

            if (suricata_ctl_flags != 0) {
                SCMutexUnlock(&tv_root_lock);
                goto error;
            }

            old_det_ctx[i] = FlowWorkerGetDetectCtxPtr(SC_ATOMIC_GET(s->slot_data));
            detect_tvs[i] = tv;

            new_det_ctx[i] = DetectEngineThreadCtxInitForReload(tv, new_de_ctx, 1);
            if (new_det_ctx[i] == NULL) {
                SCLogError(SC_ERR_LIVE_RULE_SWAP,
                        "Detect engine thread init "
                        "failure in live rule swap.  Let's get out of here");
                SCMutexUnlock(&tv_root_lock);
                goto error;
            }
            SCLogDebug("live rule swap created new det_ctx - %p and de_ctx "
                       "- %p\n",
                    new_det_ctx[i], new_de_ctx);
            i++;
            break;
        }
    }
    BUG_ON(i != no_of_detect_tvs);

    /* atomically replace the det_ctx data */
    i = 0;
    for (ThreadVars *tv = tv_root[TVT_PPT]; tv != NULL; tv = tv->next) {
        if ((tv->tmm_flags & TM_FLAG_DETECT_TM) == 0) {
            continue;
        }
        for (TmSlot *s = tv->tm_slots; s != NULL; s = s->slot_next) {
            TmModule *tm = TmModuleGetById(s->tm_id);
            if (!(tm->flags & TM_FLAG_DETECT_TM)) {
                continue;
            }
            SCLogDebug("swapping new det_ctx - %p with older one - %p", new_det_ctx[i],
                    SC_ATOMIC_GET(s->slot_data));
            FlowWorkerReplaceDetectCtx(SC_ATOMIC_GET(s->slot_data), new_det_ctx[i++]);
            break;
        }
    }
    SCMutexUnlock(&tv_root_lock);

    /* threads now all have new data, however they may not have started using
     * it and may still use the old data */

    SCLogDebug("Live rule swap has swapped %d old det_ctx's with new ones, "
               "along with the new de_ctx",
            no_of_detect_tvs);

    InjectPackets(detect_tvs, new_det_ctx, no_of_detect_tvs);

    for (i = 0; i < no_of_detect_tvs; i++) {
        int break_out = 0;
        usleep(1000);
        while (SC_ATOMIC_GET(new_det_ctx[i]->so_far_used_by_detect) != 1) {
            if (suricata_ctl_flags != 0) {
                break_out = 1;
                break;
            }

            BreakCapture();
            usleep(1000);
        }
        if (break_out)
            break;
        SCLogDebug("new_det_ctx - %p used by detect engine", new_det_ctx[i]);
    }

    /* this is to make sure that if someone initiated shutdown during a live
     * rule swap, the live rule swap won't clean up the old det_ctx and
     * de_ctx, till all detect threads have stopped working and sitting
     * silently after setting RUNNING_DONE flag and while waiting for
     * THV_DEINIT flag */
    if (i != no_of_detect_tvs) { // not all threads we swapped
        for (ThreadVars *tv = tv_root[TVT_PPT]; tv != NULL; tv = tv->next) {
            if ((tv->tmm_flags & TM_FLAG_DETECT_TM) == 0) {
                continue;
            }

            while (!TmThreadsCheckFlag(tv, THV_RUNNING_DONE)) {
                usleep(100);
            }
        }
    }

    /* free all the ctxs */
    for (i = 0; i < no_of_detect_tvs; i++) {
        SCLogDebug("Freeing old_det_ctx - %p used by detect", old_det_ctx[i]);
        DetectEngineThreadCtxDeinit(NULL, old_det_ctx[i]);
    }

    SRepReloadComplete();

    return 1;

error:
    for (i = 0; i < no_of_detect_tvs; i++) {
        if (new_det_ctx[i] != NULL)
            DetectEngineThreadCtxDeinit(NULL, new_det_ctx[i]);
    }
    return -1;
}
```

### 调试问题
调试不方便，可能我每一次改动都**需要重新编译**，编译后需要在**具有DPDK环境下的运行起来后测试**。
但是我们目前**能够运行这个组合版本的机器只有一台**，**这台设备同时还在测试规则**。

---
> 参考：
> 