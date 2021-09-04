---
title: VSCode中配置Go相关插件失败问题
tags:
  - - VSCode
  - - Go
categories:
  - [日常, 其他]
abbrlink: 3d084d62
date: 2021-04-13 20:10:27
---

##  背景

安装完VSCode的Go插件后，一般会提示安装一些提醒类的辅助插件。当在常规安装中经常失败。。

一部分工具是需要从 https://golang.org/x/tools 上进行获取，然而，由于网络的原因，经常导致安装失败。

即使，当我们将科学上网配置为全局代理后，也是不能访问到工具的下载地址。

因为**在直接打开VSCode时，其中的操作没有经过代理**。

## 解决办法

终端设置为系统代理后，通过终端打开VSCode再进行安装插件。

```bash
# 地址端口等按照自己的配置进行调整
# 终端代理没有写入配置文件的情况下，仅在当前环境生效(参考环境变量)
export https_proxy=http://127.0.0.1:7890 http_proxy=http://127.0.0.1:7890 all_proxy=socks5://127.0.0.1:7890
# 打开VSCode后，在控制台进行 go:install 安装插件
code ./
```

---

> 参考：
>
> https://monsoir.github.io/Notes/Go/vscode-go.html