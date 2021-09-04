---
title: brew权限问题与Mac使用指定程序打开
tags:
  - - HomeBrew
  - - MacOS
categories:
  - [日常, Mac]
abbrlink: 1ffeecc0
date: 2021-03-11 19:20:04
---

### Mac命令行指定特定程序打开文件 -a

```bash
open -a /Applications/Sublime\ Text.app/ httpd.conf
```



### brew **Permission Denied** 问题解决

```bash
sudo chown -R $(whoami) /usr/local
```

