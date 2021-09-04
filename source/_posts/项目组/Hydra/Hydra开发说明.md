---
title: Hydra开发说明
tags:
  - - Hydra
  - - 渗透测试
  - - 弱口令扫描
categories:
  - [项目组, Hydra]
abbrlink: 34a2c706
date: 2020-09-29 16:35:43
---

> hydra工具集成进入扫描系统，与WEB开发端结合，通过开发说明明确输入输出格式与业务逻辑。

### 一、 输入格式

1. 命令行范式

```bash
hydra [some command line options] TARGET PROTOCOL 
```

2. 输入内容

包括**协议选择**，**IP地址与端口指定**

- 协议（必选项），具有`RDP`、`MYSQL`、`SSH`、`TELNET`选项
- IP地址（必选项）与端口（可选项），具有输入IP与上传IP地址文件两种模式

> 1) 输入IP模式：支持`IP[:port]`与`CIDR`模式(192.168.0.0/24)
>
> 2) 上传IP地址文件模式：文件一行表示一个地址，格式应为`IP[:port]`，例：
>
> foo.bar.com
> target.com:21
> unusual.port.com:2121
> default.used.here.com
> 127.0.0.1
> 127.0.0.1:2121

*（注：后期考虑将其他模块扫描得到的存活主机作为本模块地址参数）*

3. 其他已选相关参数

- -L：指定用户名字典（根据协议选定内置字典）
- -P：指定密码字典（根据协议选定内置字典）

*（注：内置字典可置于服务器静态文件中，为行数较多的txt文件）*

- -e ns：空密码与账密相同探测
- -o：指定输出文件
- -b：指定输出格式（本次选用json）

4. 输入示例

- 扫描文件地址中的SSH弱口令

```bash
hydra -L ssh_login.txt -P ssh_passwd.txt -e ns -o result.json -b json -M IP_addr.txt ssh
```

- 扫描指定IP的RDP弱口令

```bash
hydra -L rdp_login.txt -P rdp_passwd.txt -e ns -o result.json -b json 127.0.0.1 rdp
```

- 扫描文件地址的TELNET弱口令

```bash
hydra -L telnet_login.txt -P telnet_passwd.txt -e ns -o result.json -b json -M IP_addr.txt telnet 
```

- 扫描指定IP的MYSQL弱口令

```bash
hydra -L mysql_login.txt -P mysql_passwd.txt -e ns -o result.json -b json 127.0.0.1 mysql
```

### 二、输出格式

> 采用本地缓存json格式，输出文件由-o选项的参数指定。

SSH测试成功样例，扫描记录信息记录在上，结果存于`results`中。

```json
{ "generator": {
	"software": "Hydra", "version": "v9.1", "built": "2020-09-29 15:44:28",
	"server": "127.0.0.1", "service": "ssh", "jsonoutputversion": "1.00",
	"commandline": "hydra -l root -p root1 -e ns -o b.json -b json 127.0.0.1 ssh"
	},
"results": [
	{"port": 22, "service": "ssh", "host": "127.0.0.1", "login": "root", "password": "root"}
	],
"success": true,
"errormessages": [  ],
"quantityfound": 1   }
```

为扫描出弱口令则`results`为空。

```json
{ "generator": {
	"software": "Hydra", "version": "v9.1", "built": "2020-09-29 15:44:43",
	"server": "127.0.0.1", "service": "ssh", "jsonoutputversion": "1.00",
	"commandline": "hydra -l root1 -p root1 -e ns -o b.json -b json 127.0.0.1 ssh"
	},
"results": [
	],
"success": true,
"errormessages": [  ],
"quantityfound": 0   }
```