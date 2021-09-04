---
title: 搭建Hexo个人博客
tags:
  - - blog
  - - Hexo
  - - Fluid
categories:
  - [日常, 博客]
abbrlink: 25ad3a99
date: 2020-05-24 16:08:12
---

[Hexo](https://hexo.io/zh-cn/)是一个快速、简洁且高效的博客框架。

Hexo主题较多，使用也比较方便，搭建一个博客记录生活学习。

环境搭建比较简单，记录一下中间遇到的小问题和主题的设置。

## 准备工作

- 下载安装node.js（建议安装10.0版本以上）
- 下载安装git
- 下载安装hexo， `npm install -g hexo` （建议终端走代理）

### 本地搭建测试

- 本地创建一个文件夹，如xxxblog
- bash中切换到xxxblog目录下，输入 `hexo init`
- 运行`hexo s`打开服务
- 本地localhost:4000上应该有博客的基本模板，本地搭建完成

### 关联git

- git与本地连接

绑定信息：

```bash
$ git config --global user.name "你的GitHub用户名"
$ git config --global user.email "你的GitHub绑定的邮箱"
```

然后生成密钥SSH key：

```bash
$ ssh-keygen -t rsa -C "你的GitHub绑定的邮箱"
```

获取生成的密钥信息放入GitHub->Settings->SSH and GPG keys：

```bash
$ cat ~/.ssh/id_rsa.pub
```

- git与博客绑定

在Github上创建名字为`xxx.github.io`的项目，xxx为你的GitHub用户名，之后均用LeeX0示例。

打开xxxblog中的_config.yml，将其中的deployment选项的内容改为：

```xml
deploy:
  type: git
  repo: 
    github: git@github.com:LeeX0/LeeX0.github.io.git,master
```

ps：如果之后推送时显示403错误，将其中的`repo: github: git@github.com:LeeX0/LeeX0.github.io.git,master`

改为`repo: https://GitHub用户名:GitHub密码@github.com/LeeX0/LeeX0.github.io.git`尝试

- 运行`npm install hexo-deployer-git –save`安装部署工具

- 运行推送

  ```bash
  $ hexo clean	# clean 清除本地静态文件
  $ hexo g	# generate 生成本地静态文件
  $ hexo d	# deploy 推送部署文件至GitHub
  ```

- 访问leex0.github.io即可查看博客



## 博客基本使用

- 新建文章

```bash
$ hexo new "postname"	# 创建新文章
```

会在source->_posts文件夹内生成一个postname.md文件，用markdown格式进行编辑。

其中front-matter字段主要有：title 文章的标题、date 创建日期 、tags 标签、categories 分类。

tags 、categories写法建议:

```xml
tags: 
- [tag1]
- [tag2]
categories:
- [cate1]
- [cate2-1,cate2-2]
```

- 插入图片

文章插入图片source->image下，对应文件夹markdown相对路径即可/image/xxx.jpg。

- 生成推送

之后运行推送即可

```bash
$ hexo clean	# clean 清除本地静态文件
$ hexo g	# generate 生成本地静态文件
$ hexo d	# deploy 推送部署文件至GitHub
```



## 主题设置

其中[官网展示的主题](https://hexo.io/themes/)已经比较多了，[知乎的问答](https://www.zhihu.com/question/24422335)也有比较多推荐。

我是用的[Fluid](https://github.com/fluid-dev/hexo-theme-fluid)主题，介绍一下这个主题的设置。

- 下载主题

[下载](https://github.com/fluid-dev/hexo-theme-fluid/archive/v1.8.0.zip)最新主题版本，下载后解压到 themes 目录下并重命名为 fluid。

- 修改配置

修改Hexo目录下的`_config.yml`：

```bash
theme: fluid  # 指定主题

Language: zh-CN  # 指定语言，可不改
```

之后正常生成推送即可，更多设置可参考[文档](https://github.com/fluid-dev/hexo-theme-fluid)。