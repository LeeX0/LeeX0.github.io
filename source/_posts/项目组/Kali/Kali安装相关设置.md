---
title: Kali安装相关设置
tags:
  - - Kali
  - - Ubuntu
categories:
  - [项目组, Kali]
abbrlink: c77e9c76
date: 2020-10-17 14:48:48
---

> Kali安装后进行一些常规设置，以此记录。

## 设置root用户

```bash
sudo passwd
```

## 文件夹名

若安装系统时选择为中文安装，初始的用户文件夹将为中文用起来不方便。

先把系统语言改为英文，在终端中输入命令:  

```bash
export LANG=en_US
xdg-user-dirs-gtk-update
```

跳出对话框询问是否将目录转化为英文路径，同意并关闭。

再将系统语言改为中文，在终端中输入命令：

```bash
export LANG=zh_CN
```

重启系统，下次进入系统，系统会提示是否把转化好的目录改回中文，选择不再提示，并取消修改。主目录的中文转英文就完成了。

## 换源与更新

修改`/etc/apt/sources.list `，注释掉原来的源，写入新的源。

```yaml
deb http://mirrors.tuna.tsinghua.edu.cn/kali kali-rolling main contrib non-free
deb-src https://mirrors.tuna.tsinghua.edu.cn/kali kali-rolling main contrib non-free
```

更新：

```bash
# 根据源里面的metadata更新本地软件包信息，包括这个源有什么包, 每个包什么版本之类的
apt-get update
# 根据metadata更新本地软件，如果有依赖变化问题则不更新相应package
apt-get upgrade
# 在更新新软件时会更新依赖
apt-get dist-upgrade
```

## Gnome

```bash
apt-get install kali-desktop-gnome
```

在中间选择`显示管理器`时，选择`gdm3`之后重启。

### gnome允许root登入

将 `/etc/pam.d/gdm-password` 文件中的root登录检查注释掉即可。

```yaml
# auth required pam_succeed_if.so user != root quiet_success
```

## 输入法

安装输入法框架

```bash
apt-get install fcitx
```

安装谷歌输入法

```bash
apt-get install fcitx-googlepinyin
```

重启

```bash
reboot
```

在fcitx配置中将Google拼音设置为首选项。

## 设置SSH

将`/etc/ssh/sshd_config`中

```yaml
# PermitRootLogin prohibit-password
···
# PasswordAuthentication yes
```

改为：

```yaml
PermitRootLogin yes
···
PasswordAuthentication yes
```

启动服务并设为开机启动：

```bash
service ssh restart
update-rc.d ssh enable
```

## 美化

在`优化`中，可以选择[主题](https://www.gnome-look.org/p/1013714)和[图标](https://www.opendesktop.org/p/1305429/)等。放于`/usr/share/themes`与`/usr/share/icons`文件夹下可直接使用。

---

> 参考：
>
> https://www.zhihu.com/question/64374670/answer/1354272784