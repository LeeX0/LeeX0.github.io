---
title: Mac上查看文件的编码格式
tags:
  - - MacOS
  - - 编码
categories:
  - - 日常
    - Mac
abbrlink: 2a640f66
date: 2021-02-06 15:58:48
---

> 中文文件编码有些问题，之前并不知道相关系统命令来修改文件编码，使用编辑器修改过于复杂。

查到了一个比较易用的命令:`enca`

1. 查看你的系统有没有安装此命令，直接在终端执行`enca`命令，查看
2. 如果没有安装，直接`brew install enca`即可，安装完毕即可使用。

```bash
# 查看file文件的编码格式
enca file 
# 将file文件转换为utf-8的格式
enca -x utf-8 file
# 查看帮助
enca --help
```

---

> 参考：
>
> http://glanwang.com/2016/07/08/Mac/Mac%E4%B8%8A%E6%9F%A5%E7%9C%8B%E6%96%87%E4%BB%B6%E7%9A%84%E7%BC%96%E7%A0%81%E6%A0%BC%E5%BC%8F/

