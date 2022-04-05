---
title: Craft导出md文章两种方式
tags:
  - - Craft
categories:
  - - 日常
    - App体验
abbrlink: f9481444
date: 2022-03-23 17:42:44
---

> `Craft` 导出文章有通常选用**分享**按钮，若导出`.md`文件，文章中的图片均以craft的url形式出现。不便于导出归档做博客等。

### 1. 分享按钮方式

导出样式如下，见引言。

![image-20220405224615466](/Users/buddyholly/Library/Application Support/typora-user-images/image-20220405224615466.png)

### 2. 从边栏以文件导出

在右侧边栏，右键选中文件再进行导出`.md`，会生成一个`.md`文件以及文件名对应的`xxx.assets`文件夹。

且`.md`中图片超链接与导出的`assets`文件夹保持一致。

> 这个设计应该是方便用户整个文件夹导出资料，所以将文章中的内容以文件保存。
>
> ‼️**对于本地归档的需求，可以选用本导出方式。**

```bash
 ~/Downloads ls
Exported Documents
 ~/Downloads ls Exported\ Documents
微醺日记——日本酒简单分类.assets                微醺日记——日本酒简单分类.md
 ~/Downloads ls Exported\ Documents/微醺日记——日本酒简单分类.assets
classification.png             classification_png_preview.png rice.png                       rice_png_preview.png
```

