---
title: SSH多网卡下指定IP
tags:
  - - SSH
categories:
  - [学习笔记, Linux]
abbrlink: '421700e0'
date: 2021-05-15 20:50:58
---

> 一个虚拟机设置了多张网卡，默认的ssh有时候连接失败。
>
> 之前没有注意设置，需要专门设置一下IP保证正常运行。

编辑配置文件`vim /etc/ssh/sshd_config`

```ini
#Port 22

#AddressFamily any

#ListenAddress 0.0.0.0 
```

修改其中的`ListenAddress`即可

```ini
#Port 22

#AddressFamily any

ListenAddress 192.168.205.3
```

重启服务生效`systemctl restart sshd`



