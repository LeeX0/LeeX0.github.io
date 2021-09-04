---
title: Docker实践——Dockerfile指令
tags:
  - - Docker
  - - Dockerfile
categories:
  - [学习笔记, Docker]
abbrlink: 8ef91b9d
date: 2021-03-24 15:28:02
---

> `FROM`与`RUN`命令不再介绍，见[链接](https://leex0.top/2021/03/24/Docker%E5%AE%9E%E8%B7%B5%E2%80%94%E2%80%94%E6%9E%84%E5%BB%BA%E9%95%9C%E5%83%8F/)。

## COPY复制文件

格式：

- `COPY [--chown=<user>:<group>] <源路径>... <目标路径>`
- `COPY [--chown=<user>:<group>] ["<源路径1>",... "<目标路径>"]`

和 `RUN` 指令一样，也有两种格式，一种类似于命令行，一种类似于函数调用。

`COPY` 指令将从构建上下文目录中 `<源路径>` 的文件/目录复制到新的一层的镜像内的 `<目标路径>` 位置。比如：

```dockerfile
COPY package.json /usr/src/app/
```

`<源路径>` 可以是多个，甚至可以是通配符，其通配符规则要满足 Go 的 [`filepath.Match`](https://golang.org/pkg/path/filepath/#Match) 规则，如：

```dockerfile
COPY hom* /mydir/
COPY hom?.txt /mydir/
```

`<目标路径>` 可以是容器内的绝对路径，也可以是相对于工作目录的相对路径（工作目录可以用 `WORKDIR` 指令来指定）。目标路径不需要事先创建，如果目录不存在会在复制文件前先行创建缺失目录。

此外，还需要注意一点，使用 `COPY` 指令，源文件的各种元数据都会保留。比如读、写、执行权限、文件变更时间等。这个特性对于镜像定制很有用。特别是构建相关文件都在使用 Git 进行管理的时候。

在使用该指令的时候还可以加上 `--chown=<user>:<group>` 选项来改变文件的所属用户及所属组。

```dockerfile
COPY --chown=55:mygroup files* /mydir/
COPY --chown=bin files* /mydir/
COPY --chown=1 files* /mydir/
COPY --chown=10:11 files* /mydir/
```

如果源路径为文件夹，复制的时候不是直接复制该文件夹，而是将文件夹中的内容复制到目标路径。

## ADD 更高级的复制文件

> **结论：此在 `COPY` 和 `ADD` 指令中选择的时候，可以遵循这样的原则，所有的文件复制均使用 `COPY` 指令，仅在需要自动解压缩的场合使用 `ADD`。**

`ADD` 指令和 `COPY` 的格式和性质基本一致。但是在 `COPY` 基础上增加了一些功能。

如果 `<源路径>` 为一个 `tar` 压缩文件的话，压缩格式为 `gzip`, `bzip2` 以及 `xz` 的情况下，`ADD` 指令将会自动解压缩这个压缩文件到 `<目标路径>` 去。

在某些情况下，这个自动解压缩的功能非常有用，比如官方镜像 `ubuntu` 中：

```dockerfile
FROM scratch
ADD ubuntu-xenial-core-cloudimg-amd64-root.tar.gz /
...
```

## CMD 容器启动命令

`CMD` 指令的格式和 `RUN` 相似，也是两种格式：

- `shell` 格式：`CMD <命令>`
- `exec` 格式：`CMD ["可执行文件", "参数1", "参数2"...]`
- 参数列表格式：`CMD ["参数1", "参数2"...]`。在指定了 `ENTRYPOINT` 指令后，用 `CMD` 指定具体的参数。

**之前介绍容器的时候曾经说过，Docker 不是虚拟机，容器就是进程。既然是进程，那么在启动容器的时候，需要指定所运行的程序及参数。`CMD` 指令就是用于指定默认的容器主进程的启动命令的。**

在运行时可以指定新的命令来替代镜像设置中的这个默认命令，比如，`ubuntu` 镜像默认的 `CMD` 是 `/bin/bash`，如果我们直接 `docker run -it ubuntu` 的话，会直接进入 `bash`。我们也可以在运行时指定运行别的命令，如 `docker run -it ubuntu cat /etc/os-release`。这就是用 `cat /etc/os-release` 命令替换了默认的 `/bin/bash` 命令了，输出了系统版本信息。

在指令格式上，一般推荐使用 `exec` 格式，这类格式在解析时会被解析为 JSON 数组，因此一定要使用双引号 `"`，而不要使用单引号。

如果使用 `shell` 格式的话，实际的命令会被包装为 `sh -c` 的参数的形式进行执行。比如：

```dockerfile
CMD echo $HOME
```

在实际执行中，会将其变更为：

```dockerfile
CMD [ "sh", "-c", "echo $HOME" ]
```

这就是为什么我们可以使用环境变量的原因，因为这些环境变量会被 shell 进行解析处理。

# ENTRYPOINT

`ENTRYPOINT` 的格式和 `RUN` 指令格式一样，分为 `exec` 格式和 `shell` 格式。

`exec`形式下`ENTRYPOINT`与`CMD`组成的完整命令：

```dockerfile
executable CMD
```

`shell`形式下`ENTRYPOINT`与`CMD`组成的完整命令（不建议使用）：

```bash
sh -c "executable" CMD
```

`ENTRYPOINT`作为目标入口。例：在`ubuntu`中默认的`ENTRYPOINT`其实就是`/bin/bash`。

`ENTRYPOINT + CMD `加起来组成了容器起动时的指令。当我们想把目标镜像作为一个一次性运行的程序时，则在`ENTRYPOINT`中设置可执行命令，在`CMD`中设置参数。

比如有一个二进制的可执行文件square，需要一个参数，对其求平方值。那么我们可以打一个平方工具的镜像：

```dockerfile
FROM baseImage
ADD square /usr/bin/
RUN chmod +x /usr/bin/square
ENTRYPOINT ["/usr/bin/square"]
CMD ["2"]
```

那么，打出来镜像后（假设镜像名为squareImage:1.0），我们就可以这样使用镜像，这个命令会直接输出结果4，然后容器退出。

```bash
docker run squareImage:1.0
```

当然，我们还可以在docker run的时候覆盖CMD，如下命令会直接输出结果9，然后容器退出。

```bash
docker run squareImage:1.0 3
```



## 其他指令

### ENV 设置环境变量

格式有两种：

- `ENV <key> <value>`
- `ENV <key1>=<value1> <key2>=<value2>...`

这个指令很简单，就是设置环境变量而已，无论是后面的其它指令，如 `RUN`，还是运行时的应用，都可以直接使用这里定义的环境变量。

```dockerfile
ENV VERSION=1.0 DEBUG=on NAME="Happy Feet"
```

### ARG 构建参数

格式：`ARG <参数名>[=<默认值>]`

构建参数和 `ENV` 的效果一样，都是设置环境变量。所不同的是，`ARG` 所设置的是**构建环境的环境变量（参数）**，在将来容器运行时是不会存在这些环境变量的。但是不要因此就使用 `ARG` 保存密码之类的信息，因为 `docker history` 还是可以看到所有值的。

`Dockerfile` 中的 `ARG` 指令是定义参数名称，以及定义其默认值。该默认值可以在构建命令 `docker build` 中用 `--build-arg <参数名>=<值>` 来覆盖。

### VOLUME 定义匿名卷

格式为：

- `VOLUME ["<路径1>", "<路径2>"...]`
- `VOLUME <路径>`

为了防止运行时用户忘记将动态文件所保存目录挂载为卷，在 `Dockerfile` 中，我们可以事先指定某些目录挂载为匿名卷，这样在运行时如果用户不指定挂载，其应用也可以正常运行，不会向容器存储层写入大量数据。

```dockerfile
VOLUME /data
```

### EXPOSE 暴露端口

格式为 `EXPOSE <端口1> [<端口2>...]`。

`EXPOSE` 指令是声明容器运行时提供服务的端口，这只是一个声明，在容器运行时并不会因为这个声明应用就会开启这个端口的服务。

### WORKDIR 指定工作目录

格式为 `WORKDIR <工作目录路径>`。

使用 `WORKDIR` 指令可以来指定工作目录（或者称为当前目录），以后各层的当前目录就被改为指定的目录，如该目录不存在，`WORKDIR` 会帮你建立目录。类似于`Dockerfile`中的`cd`指令。

> 一些初学者常犯的错误是把 `Dockerfile` 等同于 Shell 脚本来书写，这种错误的理解还可能会导致出现下面这样的错误：
>
> ```dockerfile
> RUN cd /app
> RUN echo "hello" > world.txt
> ```
>
> 如果将这个 `Dockerfile` 进行构建镜像运行后，会发现找不到 `/app/world.txt` 文件，或者其内容不是 `hello`。原因其实很简单，在 Shell 中，连续两行是同一个进程执行环境，因此前一个命令修改的内存状态，会直接影响后一个命令；而在 `Dockerfile` 中，这两行 `RUN` 命令的执行环境根本不同，是两个完全不同的容器。这就是对 `Dockerfile` 构建分层存储的概念不了解所导致的错误。
>
> 之前说过每一个 `RUN` 都是启动一个容器、执行命令、然后提交存储层文件变更。第一层 `RUN cd /app` 的执行仅仅是当前进程的工作目录变更，一个内存上的变化而已，其结果不会造成任何文件变更。而到第二层的时候，启动的是一个全新的容器，跟第一层的容器更完全没关系，自然不可能继承前一层构建过程中的内存变化。

### USER 指定当前用户

格式：`USER <用户名>[:<用户组>]`

`USER` 是改变之后层的执行 `RUN`, `CMD` 以及 `ENTRYPOINT` 这类命令的身份。注意，`USER` 只是帮助你切换到指定用户而已，这个用户必须是事先建立好的，否则无法切换。



## # 容器中应用前台执行与后台执行问题

> 注：区别与容器在`docker`中的前后台运行（`docker run -d`），是两码事。本处指容器中的应用在容器中前后台运行问题。
>
> **结论：**
>
> **容器必须要一个前台进程并且作为`pid=1`的主进程。（大概等于bash必须始终在占用）**
>
> **容器会随着`pid=1`进程的结束而结束。**

### 情景一

> 默认`nginx`镜像启动容器，或`FROM nginx`但未指定`CMD`。

`nginx`应用会正常启动并运行在后台，原因是`nginx`默认镜像中`CMD`为前台运行：

```dockerfile
# docker-nginx/Dockerfile-alpine.template
···
CMD ["nginx", "-g", "daemon off;"]
```

### 情景二

> ```dockerfile
> FROM nginx
> CMD ["nginx","-g","daemon off;"]
> ```

`nginx`应用会正常启动并运行在后台，且`nginx`会作为`pid=1`进程。

```bash
UID         PID   PPID  C STIME TTY          TIME CMD
root          1      0  0 01:29 ?        00:00:00 nginx: master process nginx -g daemon off;
nginx         5      1  0 01:29 ?        00:00:00 nginx: worker process
```

### 情景三

> ```dockerfile
> FROM nginx
> CMD [nginx -g daemon off;]
> ```

`nginx`应用会正常启动并运行在后台，但`nginx`不会作为`pid=1`进程。

因为`CMD`的`shell`模式实际会被理解成`CMD ["sh", "-c", "nginx -g daemon off;"]`

其中`sh`所存在的进程会被作为第一个前台进程，进而`sh`的`pid=1`，执行内容为`nginx -g daemon off;`。

因为`sh`（`pid=1`）进程在启动`nginx`（`pid=5`）b这个子进程后会阻塞，直到`nginx`子进程退出返回，`sh`父进程才会继续往前走。而`nginx`是一个循环任务，它不会返回，所以`sh`进程会一直阻塞，不会结束。既然`pid=1`的进程一直都在，那么容器也不会退出。

```bash
UID         PID   PPID  C STIME TTY          TIME CMD
root          1      0  0 01:34 ?        00:00:00 /bin/sh -c nginx -g 'daemon off;'
root          5      1  0 01:34 ?        00:00:00 nginx: master process nginx -g daemon off;
nginx         6      5  0 01:34 ?        00:00:00 nginx: worker process
```

### 情景四

> ```dockerfile
> FROM nginx
> CMD ["service", "nginx", "start"]
> ```

`nginx`应用不会正常启动且运行在后台。

因为`bash`执行了`service nginx start`这个瞬时命令后，继续向后。`service`进程结束，容器结束。

### 情景五

> ```dockerfile
> FROM nginx
> CMD [service nginx start]
> ```

`nginx`应用不会正常启动且运行在后台。

因为`bash`执行了`sh -c service nginx start`这个瞬时命令，`sh`（`pid=1`）在执行`service`后结束。`sh`进程结束，容器结束。

---

> 参考：
>
> https://yeasy.gitbook.io/docker_practice/
>
> https://github.com/nginxinc/docker-nginx/blob/master/Dockerfile-alpine.template
>
> https://pshizhsysu.gitbook.io/docker/dockerfile/qian-tai-yun-xing