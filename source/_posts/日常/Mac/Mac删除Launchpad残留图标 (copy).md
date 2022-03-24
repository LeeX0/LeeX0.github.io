---
title: Mac删除Launchpad残留图标
tags:
  - - MacOS
  - - Launchpad
categories:
  - [日常, Mac]
abbrlink: '98807702'
date: 2021-04-06 15:58:48
---

> 使用脚本与alias进行Launchpad残留图标删除。

脚本存放于`~/opt/script/clnicon.sh`

```bash
cd /private/var/folders/s0/t99r9bmj2fj4jvhjydggf71m0000gn/0/com.apple.dock.launchpad/db
sqlite3 db "delete from apps where title='$1';"&&killall Dock
```

> 上述第一行的地址查找：
>
> 在`finder`中前往`/private/var/folders`。
>
> 在`folders`中搜索`com.apple.dock.launchpad`。
>
> 即为所在目录

在`~/.zshrc`中添加`alias`后，`source ~/.zshrc`

```bash
alias clnicon=". ~/opt/script/clnicon.sh"
```

之后直接使用（icon_name需完全与图标相同，且大小写敏感）

```bash
clnicon icon_name
```

