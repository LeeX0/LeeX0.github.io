---
title: Ubuntu升级内核libssl依赖问题
tags:
  - - Ubuntu
  - - libssl
  - - 内核升级
categories:
  - [学习笔记, Linux]
abbrlink: 9a2c4456
date: 2021-04-26 20:59:23
---



> 升级ubuntu 16.04的内核时出现依赖libssl1.1.0问题

### 问题描述

```bash
下列软件包有未满足的依赖关系：

 linux-headers-5.3.0-050300rc4-generic : 依赖: libssl1.1 (>= 1.1.0) 但无法安装它

E: 有未能满足的依赖关系。请尝试不指明软件包的名字来运行“apt-get -f install”(也可以指定一个解决办法)。
```

### 解决方案

单独下载安装一个`libssl1.1_1.1.0g-2ubuntu4.1_amd64.deb`文件然后再升级就可以了

```bash
wget http://archive.ubuntu.com/ubuntu/pool/main/o/openssl/libssl1.1_1.1.0g-2ubuntu4_amd64.deb
sudo dpkg -i libssl1.1_1.1.0g-2ubuntu4_amd64.deb
```

---

> 参考：
>
> https://blog.csdn.net/z952957407/article/details/99690571
