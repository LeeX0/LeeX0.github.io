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

在`finder`中前往`/private/var/folders`。

在`folders`中搜索`com.apple.dock.launchpad`。

在`终端`进入`com.apple.dock.launchpad/db`目录

执行以下命令，`What you want del`为你想要清除的残留图标名。

```bash
sqlite3 db "delete from apps where title='What you want del';"&&killall Dock
```

