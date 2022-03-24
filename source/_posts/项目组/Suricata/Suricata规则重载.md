---
title: Suricata规则重载
tags:
  - Suricata
  - rules
  - 入侵检测
categories:
  - - 项目组
    - Suricata
abbrlink: bbe9896b
date: 2021-10-09 11:25:41
---

> Suricata 不重启情况下更新规则rules。

## 1. Suricata  规则重载步骤

1. 加载新的配置配置，更新规则变量
2. 加载新的规则
3. 构建新的检测引擎
4. 用新的检测引擎替换旧的引擎
5. 确保所有线程已更新
6. 释放旧的检测引擎

*注：系统要有足够内存用于启动两个检测引擎（替换过程中有两个引擎同时存在）*

## 2. 两种规则重载方式

### 2.1 `Signal`

```bash
kill -USR2 $(pidof suricata)
```

> SIGUSR2
>
> Causes Suricata to perform a live rule reload.

### 2.2 `Unix socket`

```bash
suricatasc -c reload-rules
```

> Unix socket方式需要安装suricatasc脚本，通过普通编译方式安装则默认安装完成。
>
> 若未安装，通过如下命令在 `python/suricatasc` 下安装
>
> ```bash
> sudo python setup.py install
> ```

## 3. 测试

分别测试了上述两种方式下的两项内容，共四项测试。

1. 测试Signal方式，对于yaml中已载入的.rules文件修改后重载
2. 测试Signal方式，对于yaml中已载入的.rules文件修改后重载
3. 测试Unix socket方式，对于yaml中未载入的.rules文件添加后重载
4. 测试Unix socket方式，对于yaml中未载入的.rules文件添加后重载

测试均成功，以下为其中一项测试的日志情况`/var/log/suricata/suricata.log`，其他类似。

```bash
10/8/2021 -- 00:09:51 - <Notice> - rule reload starting
10/8/2021 -- 00:09:51 - <Info> - 1 rule files processed. 1 rules successfully loaded, 0 rules failed
10/8/2021 -- 00:09:51 - <Info> - Threshold config parsed: 0 rule(s) found
10/8/2021 -- 00:09:51 - <Info> - 1 signatures processed. 0 are IP-only rules, 1 are inspecting packet payload, 0 inspect application layer, 0 are decoder event only
10/8/2021 -- 00:09:51 - <Info> - cleaning up signature grouping structure... complete
10/8/2021 -- 00:09:51 - <Notice> - rule reload complete

10/8/2021 -- 00:10:27 - <Notice> - rule reload starting
10/8/2021 -- 00:10:27 - <Info> - 1 rule files processed. 1 rules successfully loaded, 0 rules failed
10/8/2021 -- 00:10:27 - <Info> - Threshold config parsed: 0 rule(s) found
10/8/2021 -- 00:10:27 - <Info> - 1 signatures processed. 0 are IP-only rules, 1 are inspecting packet payload, 0 inspect application layer, 0 are decoder event only
10/8/2021 -- 00:10:27 - <Info> - cleaning up signature grouping structure... complete
10/8/2021 -- 00:10:27 - <Notice> - rule reload complete
10/8/2021 -- 00:10:27 - <Info> - Unix socket: lost connection with client00000000
```

---

> 参考：
>
> https://suricata.readthedocs.io/en/suricata-6.0.3/manpages/suricata.html#signals
