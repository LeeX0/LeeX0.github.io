---
title: go mod使用
tags:
  - - Go
  - - gomod
categories:
  - - 学习笔记
    - Go
abbrlink: 316ef7c8
date: 2022-04-22 18:18:10
---

> `go mod`被用以替代`GOPATH`式本地化管理仓库与进行依赖的版本控制。

### 1.Go寻找包的方式

1、 当执行Go程序，需要获取import的包时，编译器回先去`GOROOT`路径下的`src`文件夹找有没有我们在程序中import的包

`GOROOT`为Go的安装目录，其中包括了大量Go自带的包`fmt`等等。

2、如果在`GOROOT`下没有找到，就会去`GOPATH`下`src`下找这个包

### 2.使用`go mod`所下载的包存放位置

下载下来的第三方库，存放在`GOPATH/pkg/mod`文件夹里面

### 3.`go mod`使用

在Go的环境配置中启用`GO111MODUL`后，在需要管理的文件夹下进行mod初始化，`<module name>`缺省为当前文件夹名。

```ruby
go mod init <module name>
```

之后会生成一个`go.mod`文件，此时文件内容为**mod名**与**go版本号**，之后相关包依赖信息会记录在该文件内容中。

```go
module mypackage

go 1.16
```

### 4安装方式

假设需要安装[github.com/spf13/cobra](github.com/spf13/cobra) v1.3.0这个包。

#### 方式一

在生成好的go.mod文件中添加require

```go
module mypackage

go 1.16

require github.com/spf13/cobra v1.3.0 // indirect
```

然后通过命令进行下载。

```go
go mod download
```

会发现当前文件夹下多了个`go.sum`的文件，该文件基本上用来记录包版本的关系，不怎么需要理会。

**需要的包装在`GOPATH/pkg/mod`文件夹下。**

当然也可以不执行go mod download而直接运行go build或者go install也会自动下载包。

#### 方式二

直接使用go get进行下载。

```Bash
go get github.com/spf13/cobra
```

只要有开启go mod功能，go get就不会再src下放package，而是会放在/pkg/mod里面，并且mod会写好引入。也就不需要go mod download了。

---

> 参考：
>
> [https://www.jianshu.com/p/a244f48f49b3](https://www.jianshu.com/p/a244f48f49b3)
