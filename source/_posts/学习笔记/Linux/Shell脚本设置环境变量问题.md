---
title: Shell脚本设置环境变量问题
tags:
  - - Linux
  - - Shell
  - - 环境变量
categories:
  - [学习笔记, Linux]
abbrlink: 74cee2a8
date: 2020-08-11 13:31:45
---

> 在编写框架的初始化编译脚本的过程中，需要向系统导入一个环境变量。
>
> 期望导入`LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$path`
>
> 使用`sh init.sh`后，发现环境变量并未导入。

脚本内容简单，如下：

```sh
./configure && make clean && make
path=$(pwd)
LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$path
export LD_LIBRARY_PATH
ldconfig
```

## 解决方法

- 使用`source init.sh`可以解决问题（具有权限情况下）。



## 问题原因

- 使用 `sh` 命令来执行shell脚本的时候，脚本实际是在`sh`创建的子shell中执行。

- 所以当`sh`进程完成的时候并没有修改系统变量，所以通过执行 `sh init.sh `来修改系统变量是无效的。



### Q1. 父子Shell 与 环境变量

- 执行程序通常可以理解为parent process所产生的child process，child执行完后再返回到parent。这一现象在Linux中成为`fork`。子进程产生时会从父进程处**获得资源分配与继承环境**，**所谓环境变量其实就是会传给子进程的变量**。
- 通常是，**子shell会继承所有父shell的变量**（可以直接引用）。父shell的变量包括**所有export导出的环境变量和当前环境下设置的变量**（形如var=value）的命令。
- 从 process 的观念来看，是 **parent process 产生一个 child process 去执行**，当 child 结束后，会返回 parent ，但 parent 的环境是不会因 child 的改变而改变的。

### Q2. source(.) 与 sh 与 ./xxx 执行脚本的区别

- 对于脚本xxx.sh来说，`. ./xxx.sh`与`source ./xxx.sh`相同，与`./xxx.sh`和`sh xxx.sh`均不同。
- `./xxx.sh`——首先你会查找脚本第一行是否指定了解释器，如果没指定，那么就用当前系统默认的shell(大多数linux默认是bash)，如果指定了解释器，那么就将该脚本交给指定的解释器。
- `sh xxx.sh`——表示我使用sh来解释这个脚本，可以不要执行权限。

- 正常来说，当我们执行一个 shell script 时，其实是先**产生一个 sub-shell 的子进程**，然后 sub-shell 再去产生命令行的子进程。（`sh`执行脚本的一般方式）

- 所谓`source`就是让 script 在**当前 shell 内执行而不是产生一个 sub-shell 来执行**。
  由于所有执行结果均于当前 shell 内完成，若 script 的环境有所改变，当然也会改变当前环境。
  *可以理解为source是把脚本内容一行一行读到父shell里挨着执行。*

---

> 参考：
>
> http://bbs.chinaunix.net/thread-2211666-1-1.html
>
> https://www.zhihu.com/question/41441630/answer/91061860  - Virtual的回答