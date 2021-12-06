---
title: Hexo推送及备份
tags:
  - - Hexo
  - - Github
categories:
  - - 日常
    - 博客
abbrlink: 554fe90e
date: 2021-09-06 11:28:16
---
> Hexo推送通过渲染`./source`形成`./public`，将`./public`推送至Git下`master`主分支显示页面。
>
> 推送上传的都是渲染后的html内容，期望在推送前端页面同时，将所有博客资源进行推送保存，方便版本管理以及多设备同步编辑。
>
> 方式：**在hexo博客git项目下增加分支backup保存博客所有内容，推送博客内容同时推送分支。**

## 布置分支

在A电脑上完成hexo搭建以及正常deploy后，在hexo目录下进行操作：

```bash
git init
git remote add origin https://github.com/Github_username/Github_username.github.io.git
git checkout -b branch_name
git add .
git commit -m "comment info"
git push origin backup # backup随意填写，分支名
```

## 拉取分支

在B电脑上[布置git与node环境](https://leex0.top/posts/25ad3a99/)，拉取分支

```bash
git clone -b backup https://github.com/Github_username/Github_username.github.io.git

cd Github_username.github.io  
sudo npm install -g hexo-cli
sudo npm install
sudo npm install hexo-deployer-git
```

## 内容同步

在一处电脑完成写作后，推送内容

```bash
hexo g -d
git add .
git commit -m "comment info"
git push origin backup
```

在另一处电脑需保持同步

```bash
git pull https://github.com/Github_username/Github_username.github.io backup
```

## 补：添加命令进行推送

> 每次在推送hexo都需要输入多个指令，现在还增加了分支备份，添加`alias`进行一键推送。
>
> git提交的comment使用日期与时间戳。

### 方案一：直接`alias`（遇到问题）

在`~/.zshrc`中(根据使用shell变化)添加如下`alias`：

```bash
alias hexopush="hexo clean && hexo g && hexo d && git commit -am "$(date "+%Y-%m-%d-%s")" && git push origin backup"
```

问题在于，进行`source ~./zshrc`后，通过终端查看`alias`发现`hexopus`h中的**时间戳固定不变**，为source操作时的时间。

应该是在`source ~./zshrc`时`alias`中的变量`$(date "+%Y-%m-%d-%s")`会被计算出并且固定保存。

于是采用方案二。

### 方案二：引用脚本

将方案一中的内容保存到脚本中，通过alias执行脚本。

`~/opt/script/hexopush.sh`:

```bash
cd /Users/buddyholly/LeeX0Blog

echo " "
echo "#################################"
echo "### Hexo deployment Start!    ###"
echo "#################################"

hexo clean
hexo g -d

echo " "
echo "#################################"
echo "### Hexo deployment finished! ###"
echo "### Git Push Start!           ###"
echo "#################################"

git add .
git commit -am "$(date "+%Y-%m-%d-%s")"
git push origin backup

echo " "
echo "#################################"
echo "### Git Push finished!        ###"
echo "#################################"
```

在`~/.zshrc`中添加如下`alias hexopush='. ~/opt/script/hexopush.sh'`

**使用时，在`hexo`根目录下执行`hexopush`即可完成master与backup分支推送。**


---
> 参考：
> https://mmmbo.com/2019/12/29/Hexo-Github-Pages-MWeb-Editor-blog/