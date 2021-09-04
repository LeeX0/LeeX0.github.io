---
title: Docker实践——操作容器
tags:
  - - Docker
  - - Container
categories:
  - [学习笔记, Docker]
abbrlink: e4b4fa4b
date: 2021-03-22 10:04:54
---

## 1. 启动

### 新建并启动

所需要的命令主要为 `docker run`。

例如，下面的命令输出一个 “Hello World”，之后终止容器。

```bash
$ docker run ubuntu:18.04 /bin/echo 'Hello world'
Hello world
```

这跟在本地直接执行 `/bin/echo 'hello world'` 几乎感觉不出任何区别。

下面的命令则启动一个 bash 终端，允许用户进行交互。

```bash
$ docker run -t -i ubuntu:18.04 /bin/bash
root@af8bae53bdd3:/#
```

其中，`-t` 选项让Docker分配一个伪终端（pseudo-tty）并绑定到容器的标准输入上， `-i` 则让容器的标准输入保持打开。

在交互模式下，用户可以通过所创建的终端来输入命令，例如

```bash
root@74e4ce548f2e:/# pwd
/
root@74e4ce548f2e:/# ls
bin  boot  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
root@74e4ce548f2e:/#
```

当利用 `docker run` 来创建容器时，Docker 在后台运行的标准操作包括：

- 检查本地是否存在指定的镜像，不存在就从 [registry]() 下载
- 利用镜像创建并启动一个容器
- 分配一个文件系统，并在只读的镜像层外面挂载一层可读写层
- 从宿主主机配置的网桥接口中桥接一个虚拟接口到容器中去
- 从地址池配置一个 ip 地址给容器
- 执行用户指定的应用程序
- 执行完毕后容器被终止

### 启动已终止容器

可以利用 `docker container start` 命令，直接将一个已经终止（`exited`）的容器启动运行。

容器的核心为所执行的应用程序，所需要的资源都是应用程序运行所必需的。除此之外，并没有其它的资源。可以在伪终端中利用 `ps` 或 `top` 来查看进程信息。

```bash
root@ba267838cc1b:/# ps
  PID TTY          TIME CMD
    1 ?        00:00:00 bash
   11 ?        00:00:00 ps
```

可见，容器中仅运行了指定的 bash 应用。这种特点使得 Docker 对资源的利用率极高，是货真价实的轻量级虚拟化。

## 2. 守护态运行

更多的时候，需要让 Docker 在后台运行而不是直接把执行命令的结果输出在当前宿主机下。此时，可以通过添加 `-d` 参数来实现。

下面举两个例子来说明一下。

如果不使用 `-d` 参数运行容器。

```bash
$ docker run ubuntu:18.04 /bin/sh -c "while true; do echo hello world; sleep 1; done"
hello world
hello world
hello world
hello world
```

容器会把输出的结果 (STDOUT) 打印到宿主机上面

如果使用了 `-d` 参数运行容器。

```bash
$ docker run -d ubuntu:18.04 /bin/sh -c "while true; do echo hello world; sleep 1; done"
439564c20b1720238739ce01c54e7d6dcda6284a3a8f8994c9d28097cad72790
```

此时容器会在后台运行并不会把输出的结果 (STDOUT) 打印到宿主机上面(输出结果可以用 `docker logs` 查看)。

> **注：** 容器是否会长久运行，是和 `docker run` 指定的命令有关，和 `-d` 参数无关。

使用 `-d` 参数启动后会返回一个唯一的 id，也可以通过 `docker container ls` 命令来查看容器信息。

```bash
$ docker container ls
CONTAINER ID   IMAGE          COMMAND                  CREATED         STATUS         PORTS     NAMES
439564c20b17   ubuntu:18.04   "/bin/sh -c 'while t…"   6 minutes ago   Up 6 minutes             ecstatic_antonelli
```

要获取容器的输出信息，可以通过 `docker container logs` 命令。

```bash
$ docker container logs [container ID or NAMES]
hello world
hello world
hello world
. . .
```

## 3. 终止

可以使用 `docker container stop` 来终止一个运行中的容器。

```bash
$ docker container stop 439
439
```

此外，当 Docker 容器中指定的应用终结时，容器也自动终止。

例如对于上一章节中只启动了一个终端的容器，用户通过 `exit` 命令或 `Ctrl+d` 来退出终端时，所创建的容器立刻终止。

终止状态的容器可以用 `docker container ls -a` 命令看到。例如

```bash
$ docker container ls -a
CONTAINER ID   IMAGE          COMMAND                  CREATED          STATUS                        PORTS     NAMES
439564c20b17   ubuntu:18.04   "/bin/sh -c 'while t…"   9 minutes ago    Exited (137) 38 seconds ago             ecstatic_antonelli
98f27ac5ddc4   ubuntu:18.04   "/bin/sh -c 'while t…"   9 minutes ago    Exited (0) 9 minutes ago                unruffled_shirley
74e4ce548f2e   ubuntu:18.04   "/bin/bash"              16 minutes ago   Exited (0) 15 minutes ago               ecstatic_jemison
5cc8c294fe40   ubuntu:18.04   "/bin/echo 'Hello wo…"   17 minutes ago   Exited (0) 17 minutes ago               vibrant_buck
```

处于终止状态的容器，可以通过 `docker container start` 命令来重新启动。

```bash
$ docker container start 439
439
```

此外，`docker container restart` 命令会将一个运行态的容器终止，然后再重新启动它。

## 4. 进入容器

在使用 `-d` 参数时，容器启动后会进入后台。

某些时候需要进入容器进行操作，包括使用 `docker attach` 命令或 `docker exec` 命令，推荐大家使用 `docker exec` 命令，原因会在下面说明。

### `attach` 命令

下面示例如何使用 `docker attach` 命令。

```bash
$ docker run -dit ubuntu
f780a070123a203ab14b1632c9fe51850568baa8ae906e1fc006f21b1b763cde
$ docker container ls
CONTAINER ID   IMAGE     COMMAND       CREATED         STATUS         PORTS     NAMES
f780a070123a   ubuntu    "/bin/bash"   7 seconds ago   Up 6 seconds             interesting_almeida
$ docker attach f780
root@f780a070123a:/# pwd
/
root@f780a070123a:/#
```

*注意：* 如果从这个 stdin 中 exit，会导致容器的停止。

### `exec` 命令

`-i` `-t` 参数

`docker exec` 后边可以跟多个参数，这里主要说明 `-i` `-t` 参数。

只用 `-i` 参数时，由于没有分配伪终端，界面没有我们熟悉的 Linux 命令提示符，但命令执行结果仍然可以返回。

当 `-i` `-t` 参数一起使用时，则可以看到我们熟悉的 Linux 命令提示符。

```bash
$ docker run -dit ubuntu
bf85753fd26a3af4ee9a600dd41010e5f6f6ea2a7eb059dd017010d7f47f99b7
$ docker container ls
CONTAINER ID   IMAGE     COMMAND       CREATED         STATUS         PORTS     NAMES
bf85753fd26a   ubuntu    "/bin/bash"   7 seconds ago   Up 7 seconds             loving_tharp
$ docker exec -i bf857 bash
ls
bin
boot
...

$ docker exec -it bf857 bash
root@bf85753fd26a:/# pwd
/
root@bf85753fd26a:/#
```

如果从这个 stdin 中 exit，不会导致容器的停止。这就是为什么推荐大家使用 `docker exec` 的原因。

更多参数说明请使用 `docker exec --help` 查看。

## 5. 导出和导入

### 导出容器

如果要导出本地某个容器，可以使用 `docker export` 命令。

```bash
$ docker container ls -a
CONTAINER ID   IMAGE          COMMAND                  CREATED          STATUS                       PORTS     NAMES
bf85753fd26a   ubuntu         "/bin/bash"              5 minutes ago    Up 5 minutes                           loving_tharp
f780a070123a   ubuntu         "/bin/bash"              7 minutes ago    Exited (0) 6 minutes ago               interesting_almeida
$ docker export bf8 > ubuntu.tar
```

这样将导出容器快照到本地文件。

### 导入容器快照

可以使用 `docker import` 从容器快照文件中再导入为镜像，例如

```bash
$ docker import ubuntu.tar test/ubuntu:v1.0
sha256:a2096fd67aaa28187fda28a9c3c6a1f9184a13c589f0f5d9eb11dd554cb8d6e5
$ docker image ls
REPOSITORY    TAG       IMAGE ID       CREATED          SIZE
test/ubuntu   v1.0      a2096fd67aaa   10 seconds ago   72.9MB
hello-world   latest    d1165f221234   2 weeks ago      13.3kB
ubuntu        latest    f643c72bc252   3 months ago     72.9MB
ubuntu        18.04     2c047404e52d   3 months ago     63.3MB
```

此外，也可以通过指定 URL 或者某个目录来导入，例如

```bash
$ docker import http://example.com/exampleimage.tgz example/imagerepo
```

*注：用户既可以使用* *`docker load`* *来导入镜像存储文件到本地镜像库，也可以使用* *`docker import`* *来导入一个容器快照到本地镜像库。这两者的区别在于容器快照文件将丢弃所有的历史记录和元数据信息（即仅保存容器当时的快照状态），而镜像存储文件将保存完整记录，体积也要大。此外，从容器快照文件导入时可以重新指定标签等元数据信息。*

## 6. 删除容器

可以使用 `docker container rm` 来删除一个处于终止状态的容器。例如

```bash
$ docker container ls
CONTAINER ID   IMAGE     COMMAND       CREATED          STATUS          PORTS     NAMES
bf85753fd26a   ubuntu    "/bin/bash"   43 minutes ago   Up 43 minutes             loving_tharp
$ docker container rm bf85
Error response from daemon: You cannot remove a running container bf85753fd26a3af4ee9a600dd41010e5f6f6ea2a7eb059dd017010d7f47f99b7. Stop the container before attempting removal or force remove
$ docker container stop bf85
bf85
$ docker container rm bf85
bf85
```

如果要删除一个运行中的容器，可以添加 `-f` 参数。Docker 会发送 `SIGKILL` 信号给容器。

### 清理所有处于终止状态的容器

用 `docker container ls -a` 命令可以查看所有已经创建的包括终止状态的容器，如果数量太多要一个个删除可能会很麻烦，用下面的命令可以清理掉所有处于终止状态的容器。

```bash
$ docker container prune
```

---

> 参考：
>
> https://yeasy.gitbook.io/docker_practice/