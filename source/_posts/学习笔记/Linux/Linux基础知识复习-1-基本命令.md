---
title: Linux基础知识复习(1)--基本命令
tags:
  - - Linux
categories:
  - [学习笔记, Linux]
abbrlink: 331f8832
date: 2020-06-01 15:29:08
---

很久没有使用Linux，基础知识也忘了不少。

找了本[参考书](https://www.linuxprobe.com/docs/LinuxProbe.pdf)顺带复习一下Linux的部分基础知识。

## 基本命令

### 常用工作命令

- `echo`：用于在终端输出字符串或变量提取后的值

  ```bash
  $ echo Hello!
  Hello!
  $ echo $SHELL
  /bin/bash
  ```

- `reboot`：重启系统

- `poweroff`：关闭系统

- `wget`：下载网络文件

  ```bash
  $ wget http://www.linuxprobe.com/docs/LinuxProbe.pdf
  ```

- `ps`：查看系统中的进程状态

  ```bash
  $ ps -a(all) -u(user)
  ```

- `top`：动态地监视进程活动与系统负载等信息

- `pidof`：查询某个指定服务进程的 PID 值

  ```bash
  $ pidof sshd
  23587 798
  ```

- `kill`：终止某个指定 PID 的服务进程

  ```bash
  $ kill 23587
  ```

- `killall`：终止某个指定名称的服务所对应的全部进程

  ```bash
  $ killall sshd
  ```

### 系统状态命令

- `ifconfig`：用于获取网卡配置与网络状态等信息
- `uname`：用于查看系统内核与系统版本等信息 -a
- `free`：用于显示当前系统中内存的使用量信息
- `who`：用于查看当前登入主机的用户终端信息(whoami)
- `history`：用于显示历史执行过的命令（清除 -c）

### 目录切换命令

- `pwd`：显示用户当前所处的工作目录
- `cd`：切换工作路径
- `ls`：显示目录中文件信息（所有文件-a，详细信息-l）

### 文件编辑命令

- `cat`：查看纯文本文件（较少）

- `more `：查看纯文本文件（较多）

- `head`：查看纯文本文档的前 n行

  ```bash
  $ head -n 20 /etc/passwd
  ```

- `tail`：查看纯文本文档的后 N 行或持续刷新内容

  ```bash
  $ tail -n 20 /etc/passwd
  $ tail -f /var/log/message
  ```

- `tr`：替换文本文件中的字符

    ```bash
  $ cat anaconda-ks.cfg | tr [a-z] [A-Z]
  ```

- `wc`：统计指定文本的行数-l、字数-w、字节数-c

    ```bash
  $ wc -l /etc/passwd
  ```

- `stat`：查看文件的具体存储信息和时间等信息

    ```bash
  $ stat /etc/passwd
  ```

- `diff`：比较多个文本文件的差异

    ```bash
  $ diff diff_a.txt diff_b.txt
  ```

### 目录管理命令

- `touch`：创建空白文件或设置文件的时间

    ```bash
  $ touch touch_a.txt
  ```

- `mkdir`：创建空白目录

    ```bash
  $ mkdir ~/mkdir_a
  ```

- `cp`：用于复制文件和目录

    ```bash
  $ cp install.log x.log
  $ cp -r /etc ~/mkdir_a 
  ```

- `mv`：剪切文件或将文件重命名

    ```bash
  $ mv x.log linux.log
  ```

- `rm`：删除文件或目录

    ```bash
  $ rm -f install.log
  $ rm -rf ~/mkdir_a
  ```

- `dd`：按照指定大小和个数的数据块来复制文件或转换文件

    ```bash
  $ dd if=/dev/zero of=560_file count=1 bs=560M
  # if为输入文件，of为输出文件，count为块数 bs为每个块大小
  1+0 records in
  1+0 records out
  587202560 bytes (587 MB, 560 MiB) copied, 3.49432 s, 168 MB/s
  ```

- `file`：查看文件的类型

    ```bash
  $ file 560_file
  560_file: data
  ```

### 压缩搜索命令

- `tar`：对文件进行打包压缩或解压

    ```bash
  $ tar -czvf etc.tar.gz /etc	# 压缩
  $ tar -xzvf etc.tar.gz -C /root/etc # 解压
  ```

- `grep`：在文本中执行关键词搜索，并显示匹配的结果

    ```bash
  $ grep /sbin/nologin /etc/passwd
  # -n 显示行号； -v 反选信息
  ```

- `find`：按照指定条件来查找文件

    ```bash
  $ find /etc -name "host*" -print
  # 获取到/etc中所有以 host 开头的文件列表
  $ find / -perm -4000 -print
  # 在整个系统中搜索权限中包括 SUID 权限的所有文件
  $ find / -user linuxprobe -exec cp -a {} /root/findresults/ \;
  # 在整个文件系统中找出所有归属于 linuxprobe 用户的文件并复制到/root/findresults 目录
  # 命令的结尾必须是“\;”！！！
  ```

