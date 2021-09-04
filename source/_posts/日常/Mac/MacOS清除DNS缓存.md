---
title: MacOS清除DNS缓存
tags:
  - - MacOS
  - - DNS
categories:
  - [日常, Mac]
abbrlink: 8ea63c18
date: 2021-01-15 16:18:35
---

```bash
sudo killall -HUP mDNSResponder
sudo killall mDNSResponderHelper
sudo dscacheutil -flushcache
```

