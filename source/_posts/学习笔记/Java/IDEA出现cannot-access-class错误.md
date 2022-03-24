---
title: IDEA出现cannot access class错误
tags:
  - - Java
  - - IDEA
categories:
  - - 学习笔记
    - Java
abbrlink: d3c91af1
date: 2022-03-02 20:50:58
---

> IDEA缓存引起的一点小BUG。

如果明明有这个类存在但仍然提示cannot access，这可能是IDE的bug，可以清除缓存并重启。

在IDEA中：

``` 
1. Go to File

2. Invalidate Caches

3. You can choose only Invalidate and restart
```

---

> 参考：
>
> https://stackoverflow.com/questions/38549649/java-cannot-access-class-class-file-not-found

