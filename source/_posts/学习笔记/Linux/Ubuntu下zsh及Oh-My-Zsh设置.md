---
title: Ubuntu下zsh及Oh-My-Zsh设置
tags:
  - - Linux
  - - Shell
  - - Zsh
categories:
  - [学习笔记, Linux]
abbrlink: b6373879
date: 2020-10-14 18:24:44
---

> Z shell（Zsh）是一款可用作交互式登录的shell及脚本编写的命令解释器。内置于Ubuntu与OSX中，功能强大。
>
> Oh My Zsh是一个开源的的框架，用于管理你的Zsh配置。有很多主题与扩展。

## 安装Zsh

```bash
# 查看系统已有shell
cat /etc/shells

# 查看当前默认的 Shell
echo $SHELL 

# 安装zsh（Ubuntu与OSX内置有，若无用apt安装）
sudo apt install zsh

# 将 Zsh 设置为默认 Shell（chsh-change shell）
chsh -s /bin/zsh

# 重启 Shell（reboot）。
```

## 安装 Oh My Zsh

```bash
# 安装Oh My Zsh
wget https://github.com/robbyrussell/oh-my-zsh/raw/master/tools/install.sh -O - | sh
```

出现如下界面则安装成功。安装于默认`.oh-my-zsh`。

```csharp
__                                     __   
  ____  / /_     ____ ___  __  __   ____  _____/ /_  
 / __ \/ __ \   / __ `__ \/ / / /  /_  / / ___/ __ \ 
/ /_/ / / / /  / / / / / / /_/ /    / /_(__  ) / / / 
\____/_/ /_/  /_/ /_/ /_/\__, /    /___/____/_/ /_/  
                        /____/                       ....is now installed!


Please look over the ~/.zshrc file to select plugins, themes, and options.

p.s. Follow us at https://twitter.com/ohmyzsh.

p.p.s. Get stickers and t-shirts at http://shop.planetargon.com.
```

## Zsh配置

### 主题配置

[Oh My Zsh主题](https://github.com/ohmyzsh/ohmyzsh/wiki/themes)较多，默认使用的是*robbyrussell*，*agnoster*也比较不错。内置主题位于`~/.oh-my-zsh/themes`中，其他的主题也可下载在此处，在配置文件中直接启用。主题文件可直接打开编辑，重新source配置文件即可生效。

```bash
# 编辑配置文件修改主题
vim ~/.zshrc
# 修改其中的内容
ZSH_THEME="agnoster"
```

使用*agnoster*主题出现乱码，可通过安装解决字体问题。

```bash
sudo apt-get install powerline fonts-powerline
```

### 插件配置

[Oh My Zsh插件](https://github.com/ohmyzsh/ohmyzsh/wiki/Plugins)，内置很多插件位于`/.oh-my-zsh/plugins`。可在`~/.zshrc`中的`plugins`直接配置打开。

- z 是一个相当实用的 cd 命令增强脚本（记录你的cd使用习惯 通过z命令模糊匹配）

> 若未安装zsh，在源码仓库里可以看到，**Z** 其实也就是一个 **.sh** 脚本，所以不管你用的是什么Terminal，只用按以下步骤就能马上使用 **Z** 。
>
> 1. 将[z.sh](https://github.com/rupa/z/blob/master/z.sh)下载到本地目录`git clone https://github.com/rupa/z.git`
> 2. 在根目录对应Terminal的文件（如果是默认的，一般是.bashrc）里加上`source`和z.sh所在目录
> 3. 之后重启Terminal就可以开始用了

- extract是功能强大的解压插件，所有类型的文件解压一个命令x全搞定

- 同时额外装了几个常用的插件。可置于`~/.oh-my-zsh/custom/plugins`中便于管理。

```bash
# 安装 zsh-autosuggestions
# 命令行命令键入时的历史命令建议插件
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions

# 安装 zsh-syntax-highlighting
# 命令行语法高亮插件
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
echo "source ${(q-)PWD}/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh" >> ${ZDOTDIR:-$HOME}/.zshrc

```

修改配置文件`vim ~/.zshrc`。

```bash
# 最好把zsh-syntax-highlighting放在最后
source /opt/z/z.sh
source ~/.oh-my-zsh/custom/plugins/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh

plugins=(
  git extract zsh-autosuggestions z zsh-syntax-highlighting
)
```

---

> 参考：
>
> https://juejin.im/post/6844903620333289486
>
> https://www.jianshu.com/p/fa82d932888b
>
> https://linuxtoy.org/archives/z.html