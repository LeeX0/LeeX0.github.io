---
title: MacOS中Dock栏使用空白分隔
tags:
  - - MacOS
  - - Dock
categories:
  - [日常, Mac]
abbrlink: f3c59bf0
date: 2020-12-05 18:03:57
---

> 给Mac的Dock栏添加空白的图标用以分隔。

## 添加空白分割区

- **打开**`终端（Terminal.app）`

- **输入下列指令后，按回车键运行，空白区域就会添加到Dock中：**

  ```bash
  defaults write com.apple.dock persistent-apps -array-add '{"tile-type"="spacer-tile";}'; killall Dock
  ```

  

- **空白区域就是个透明图标，可以移动位置或拖离Dock栏，重复上方指令可添加多个**



## 只显示当前运行的应用

- **打开**`终端（Terminal.app）`

- **输入下列指令后，按回车键运行，Dock栏只显示当前运行中的应用程序：**

  ```bash
  defaults write com.apple.dock static-only -bool TRUE; killall Dock
  ```

  

- **想恢复原来状态，输入下列指令，按回车键运行即可：**

  ```bash
  defaults write com.apple.dock static-only -bool FALSE; killall Dock
  ```

---

> 参考：
>
> https://zhuanlan.zhihu.com/p/190175194