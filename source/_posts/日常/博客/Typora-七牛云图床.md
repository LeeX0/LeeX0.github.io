---
title: Typora+七牛云图床
tags:
  - - Typora
  - - 七牛云
  - - 图床
categories:
  - - 日常
    - 博客
abbrlink: d6a29df8
date: 2021-06-28 18:15:50
---

> 在`Typora`中使用七牛云进行云端图床保存图片。
>
> 至于最后我的Hexo博客图片，并没有采用七牛云图床保存，原因参见[此文](https://leex0.top/posts/9e17967/)。

## 1. 七牛云

### 1.1 注册七牛云

登录[七牛云官网](https://www.qiniu.com/)，完成注册相关内容。

### 1.2 实名认证

如果使用免费的存储内容需要进行实名认证，有微信扫码即可完成的比较容易。

### 1.3 创建对象存储实例

在`管理控制台`中左侧添加`对象存储Kodo`。需要起一个空间名称与选定一个存储区域，之后都要用相关信息。

### 1.4 域名绑定

需要绑定一个备案的域名（若未备案，覆盖区域则只能为海外）。没有直接绑定的话七牛云会分配一个CDN测试域名可以使用一个月。

## 2. Typora

Typora内置了很多图片上传服务选项iPic、uPic、PicGo、自定义等。

这里选用了`PicGo-Core`进行上传。

### 2.1 安装`PicGo-Core`

```bash
 npm install picgo -g
```

安装后可用`picgo -h`查看说明。

### 2.2 修改配置信息

默认配置文件在`~/.picgo/config.json`。

PicGo使用七牛云的相关配置如下：

```json
{
  "picBed": {
    "uploader": "qiniu",
    "qiniu": {
      "accessKey": "xxxxxxxxxxxxxxx",
      "secretKey": "aaaaaaaaaaaaaaa",
      "bucket": "bbbbbbbbbbbbb", // 存储空间名
      "url": "ccccccccccccccc", // 自定义域名
      "area":  "z1", // 存储区域编号
      "options": "", // 网址后缀，比如？imgslim
      "path": "img/" // 自定义存储路径，比如 img/
    }
  },
  "picgoPlugins": {}
}
```

其中`accessKey`与`secretKey`，在`七牛云->个人中心->密钥管理`处。`bucket`是*1.3*中对象存储实例`空间名称`。`url`即*1.4*中的`CDN加速/测试域名`。`area`为*1.3*中的`存储区域编号`。

配置好后随便复制一个图片使用`picgo upload`，即可上传剪贴板最近的图片用以测试上传是否成功。

### 2.3 Typora上传服务设置

`PicGo-Core`的具体使用，我在Typora的图像上传服务中选用的是`Custom Command`，命令为`/usr/local/bin/node /usr/local/bin/picgo upload`。具体要看你的工具安装在什么位置了。

> 按理说命令直接写为`picgo upload`应该也可以。
>
> 但是我这里会提示`bash`找不到命令，可能是因为我使用的是`zsh`，没有深究。同时我也没有在`env`中写入`node`。所以全部使用绝对路径即可成功。

### 2.4 Typora插入图片操作

可以选为在**插入图片时上传图片**，这样最为方便。

但会**有一个问题**，如果你只是临时复制进去看，或者需要更换和修改，但你复制进去就已经上传，**在图床中会有临时垃圾文件产生**，维护起来不方便（如果你根本不看只是用当然也不用维护）。

同时可以勾选上`插入时自动转义图片URL`。

但如果我使用的话，还是会选择**插入图片时无操作**，在文件编辑完成后进行`格式->图像->上传所有本地图片`。

---

> 参考：
>
> https://support.typora.io/Upload-Image/
>
> https://picgo.github.io/PicGo-Core-Doc/zh/guide/config.html
