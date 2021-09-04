---
title: aria2与Homebrew简记
tags:
  - - aria2
  - - Homebrew
  - - Terminal
categories:
  - [日常, Mac]
abbrlink: e8d82254
date: 2021-03-10 15:32:49
---

### aria2

> [aria2](https://aria2.github.io/)是一个自由、开源、轻量级多协议和多源的命令行下载工具，它支持 HTTP/HTTPS、FTP、SFTP、 BitTorrent 和 Metalink 协议。
>
> brew install aria2

```bash
# 下载单个文件（支持使用种子文件与磁力链接）
aria2c https://x/owncloud-9.0.0.tar.bz2
# 下载多个文件
aria2c -Z https://x/owncloud-9.0.0.tar.bz2 https://y/owncloud-9.0.0.tar.bz2
# 另存文件名字
aria2c -o owncloud.zip https://x/owncloud-9.0.0.tar.bz2
# 续传未完成的下载（中断使用.aria2后缀保存文件，下次重启任务后续传）
aria2c -c https://x/owncloud-9.0.0.tar.bz2
# 从文件获取输入（可传入URL列表）
aria2c -i test-aria2.txt
# 其他
-D 后台下载
--conf-path="~/.aria2/aria2.conf" 配置文件
-s 设置线程数
-max-download-limit 设置最大下载速度
-x3 到服务器的连接数
--http-user=xxx --http-password=xxx http密码下载
--ftp-user=xxx --ftp-password=xxx ftp密码下载
```

配置文件可参考：

```bash
#用户名
#rpc-user=user
#密码
#rpc-passwd=passwd
#上面的认证方式不建议使用,建议使用下面的token方式
#设置加密的密钥
#rpc-secret=token
#允许rpc
enable-rpc=true
#允许所有来源, web界面跨域权限需要
rpc-allow-origin-all=true
#允许外部访问，false的话只监听本地端口
rpc-listen-all=true
#RPC端口, 仅当默认端口被占用时修改
#rpc-listen-port=6800
#最大同时下载数(任务数), 路由建议值: 3
max-concurrent-downloads=5
#断点续传
continue=true
#同服务器连接数
max-connection-per-server=5
#最小文件分片大小, 下载线程数上限取决于能分出多少片, 对于小文件重要
min-split-size=10M
#单文件最大线程数, 路由建议值: 5
split=10
#下载速度限制
max-overall-download-limit=0
#单文件速度限制
max-download-limit=0
#上传速度限制
max-overall-upload-limit=0
#单文件速度限制
max-upload-limit=0
#断开速度过慢的连接
#lowest-speed-limit=0
#验证用，需要1.16.1之后的release版本
#referer=*
#文件保存路径, 默认为当前启动位置
dir=/Users/xxx/Downloads
#文件缓存, 使用内置的文件缓存, 如果你不相信Linux内核文件缓存和磁盘内置缓存时使用, 需要1.16及以上版本
#disk-cache=0
#另一种Linux文件缓存方式, 使用前确保您使用的内核支持此选项, 需要1.15及以上版本(?)
#enable-mmap=true
#文件预分配, 能有效降低文件碎片, 提高磁盘性能. 缺点是预分配时间较长
#所需时间 none < falloc ? trunc « prealloc, falloc和trunc需要文件系统和内核支持
file-allocation=prealloc
```

设置WebUI可参考[链接](https://ziahamza.github.io/webui-aria2/)。



### HomeBrew

```bash
## 管理HomeBrew
# 安装 HomeBrew
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
# 卸载 HomeBrew
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/uninstall)"
# 更新 HomeBrew
brew update
# 检视HomeBrew
brew doctor

## 管理软件
# 安装卸载更新软件
brew install wget
brew uninstall wget
brew uograde wget
# 列出安装的软件
brew list
# 列出安装的软件信息
brew info wget
# 检查软件库
brew search wget

## 管理服务
# 列出服务
brew services list
# 注销未使用服务
brew services cleanup
# 运行服务（不注册为跟随系统启动）
brew services run nginx
# 运行后台服务（注册为跟随系统启动）
brew services start nginx
# 暂停并注销服务
brew services stop nginx
# 重启并注册服务
brew services restart nginx
```

---

> 参考：
>
> https://wild-flame.github.io/guides/docs/mac-os-x-setup-guide/aria_2/readme
>
> https://www.linuxprobe.com/aria2-download.html
>
> https://segmentfault.com/a/1190000018928643
>
> 