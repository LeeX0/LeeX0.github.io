---
title: 'Hexo链接优化:abbrlink'
tags:
  - - blog
  - - Hexo
  - - abbrlink
categories:
  - [日常, 博客]
abbrlink: 6e344d64
date: 2021-04-01 14:45:44
---

> `Hexo`博客的链接如果显示中文，在一些场景的超链接识别可能不完全，如果使用转码则中文转码较长较乱。
>
> 这里使用[hexo-abbrlink](https://github.com/rozbo/hexo-abbrlink)插件进行优化链接。

### 安装插件

```bash
npm install hexo-abbrlink --save
```

### 修改配置文件

修改博客配置文件`_config.yml`内容

```yaml
permalink: posts/:abbrlink/
## abbrlink conf
abbrlink:
  alg: crc32  # 算法：crc16(default) and crc32
  rep: hex    # 进制：dec(default) and hex
```

之后`md`文件的`Front-matter` 内会增加`abbrlink` 字段。无需操作。

---

> 参考：
>
> https://github.com/rozbo/hexo-abbrlink