---
title: Parallels虚拟机迁移问题记录
tags:
  - - MacOS
  - - DNS
  - - Parallels
categories:
  - [日常, Mac]
abbrlink: f99bebc2
date: 2021-03-29 21:50:04
---

> 之前使用Mac上的Parallels安装Windows 10时，默认选择了`/var/root/Parallels`路径存放虚拟机文件。
>
> 而其他虚拟机文件在`/Users/buddyholly/Parallels`下。整体管理不方便，故选择将Windows 10迁移到Users下。

### 问题一：`Finder`文件夹不显示隐藏文件

使用命令使隐藏文件在`Finder`中显示

```bash
defaults write com.apple.finder AppleShowAllFiles TRUE && killall Finder
```

若需要重新隐藏将`TRUE`写为`FALSE`即可。

为了省事可以写一个`alias`，使用`zsh`的话将以下内容写入`~/.zshrc`，立即生效需要`source`这个文件。

```bash
# alias
showhidenfile="defaults write com.apple.finder AppleShowAllFiles TRUE && killall Finder"
notshowhidenfile="defaults write com.apple.finder AppleShowAllFiles FALSE && killall Finder"
```

### 问题二：迁移`Parallels`虚拟机文件`Windows 10.pvm`

我使用的方式是先克隆一个虚拟机，再将原虚拟机删除。

在`Parallels`中右键需要操作的虚拟机，选中`克隆`，可以选择克隆的目的路径与目标文件名（实际上pvm是个目录，包含了虚拟机所用的相关配置与文件）。我将了`/var/root/Parallels/Windows 10.pvm`克隆在了`/Users/buddyholly/Parallels/Windows 10.pvm`。

之后将`/var/root/Parallels/Windows 10.pvm`与`/var/root/Parallels/`下相关文件删除。这里我遇到了一个问题见*问题4*。

### 问题三：克隆虚拟机的所有者与组

因为原虚拟机文件在`/var/root`下，克隆至`/Users/buddyholly/Parallels/Windows 10.pvm`后仍为`root`的`own`与`group`。不变更的话操作不方便，而且显示新的pvm文件大小有误，但在电脑上肯定占用了空间。变更一下，变更后正常使用，文件大小显示正确。

```bash
# 在/Users/buddyholly/Parallels/下进行
# 两个操作都应选择递归-r，否则仍有错误
sudo chown -r leex0 Windows 10.pvm
sudo chgrp -r staff Windows 10.pvm
```

### 问题四：清理root下的.Trash

在解决*问题三*前，发现自己**Mac的空间被其他类别占用很多**，应该是虚拟机迁移的相关文件导致，搜了一下文件所在。发现`/var/root/.Trash`占用很大，应该是之前的操作不当导致垃圾箱有很多无用文件。

而在这里的垃圾文件不会被诸如CleanMyMac等软件扫描出来（可能是这些东西在/root下的原因）。手动删除一下`/var/root/.Trash`内的文件，再看Mac的存储空间，已经多出了应有的剩余空间。