---
title: Docker实践——访问仓库
tags:
  - - Docker
  - - Repository
categories:
  - [学习笔记, Docker]
abbrlink: 87094ab1
date: 2021-03-24 12:20:42
---

> 对于仓库地址 docker.io/ubuntu 来说，docker.io 是注册服务器地址，ubuntu 是仓库名。注册服务器是管理仓库的具体服务器，每个服务器上可以有多个仓库，而每个仓库下面有多个镜像。

### Docker Hub

[Docker Hub](https://hub.docker.com/)是Docker官方维护的一个公共仓库。其中已经包括了数量超过 [2,650,000](https://hub.docker.com/search/?type=image) 的镜像。大部分需求都可以通过在 Docker Hub 中直接下载镜像来实现。免费[注册](https://hub.docker.com)后可登陆使用。

可以通过执行 `docker login` 命令交互式的输入用户名及密码来完成在命令行界面登录 Docker Hub。你可以通过 `docker logout` 退出登录。

### 拉取镜像

你可以通过 `docker search` 命令来查找官方仓库中的镜像，并利用 `docker pull` 命令来将它下载到本地。

例如以 `centos` 为关键词进行搜索：

```bash
> docker search centos
NAME                               DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
centos                             The official build of CentOS.                   6476      [OK]
ansible/centos7-ansible            Ansible on Centos7                              133                  [OK]
consol/centos-xfce-vnc             Centos container with "headless" VNC session…   127                  [OK]
···
```

可以看到返回了很多包含关键字的镜像，其中包括镜像名字、描述、收藏数、是否官方创建（`OFFICIAL`）、是否自动构建 （`AUTOMATED`）。

根据是否是官方提供，可将镜像分为两类。

一种是类似 `centos` 这样的镜像，被称为基础镜像或根镜像。这些基础镜像由 Docker 公司创建、验证、支持、提供。这样的镜像往往使用单个单词作为名字。还有一种类型，比如 `ansible/centos7-ansible` 镜像，它是由 Docker Hub 的注册用户创建并维护的，往往带有用户名称前缀。可以通过前缀 `username/` 来指定使用某个用户提供的镜像，比如 ansible 用户。

另外，在查找的时候通过 `--filter=stars=N` 参数可以指定仅显示收藏数量为 `N` 以上的镜像。

```bash
> docker search --filter=stars=100 centos
NAME                      DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
centos                    The official build of CentOS.                   6476      [OK]
ansible/centos7-ansible   Ansible on Centos7                              133                  [OK]
···
```

下载官方 `centos` 镜像到本地。

```bash
> docker pull centos
Using default tag: latest
latest: Pulling from library/centos
7a0437f04f83: Pull complete
Digest: sha256:5528e8b1b1719d34604c87e11dcd1c0a20bedf46e83b5632cdeac91b8c04efc1
Status: Downloaded newer image for centos:latest
docker.io/library/centos:latest
```

### 推送镜像

用户也可以在登录后通过 `docker push` 命令来将自己的镜像推送到 Docker Hub。

以下命令中的 `username` 请替换为你的 Docker 账号用户名。

```bash
> docker image ls
REPOSITORY    TAG       IMAGE ID       CREATED        SIZE
ubuntu        18.04     2c047404e52d   3 months ago   63.3MB
> docker tag ubuntu:18.04 leex0/ubuntu:18.04
> docker image ls
REPOSITORY     TAG       IMAGE ID       CREATED        SIZE
leex0/ubuntu   18.04     2c047404e52d   3 months ago   63.3MB
ubuntu         18.04     2c047404e52d   3 months ago   63.3MB
> docker push leex0/ubuntu:18.04
The push refers to repository [docker.io/leex0/ubuntu]
fe6d8881187d: Mounted from library/ubuntu
23135df75b44: Mounted from library/ubuntu
b43408d5f11b: Mounted from library/ubuntu
18.04: digest: sha256:a7fa45fb43d471f4e66c5b53b1b9b0e02f7f1d37a889a41bbe1601fac70cb54e size: 943
> docker search leex0
NAME                      DESCRIPTION   STARS     OFFICIAL   AUTOMATED
leex0/docker101tutorial                 0
```



### *私有仓库

创建私有仓库相关可参考[链接](https://yeasy.gitbook.io/docker_practice/repository/registry)。

---

> 参考：
>
> https://yeasy.gitbook.io/docker_practice/

