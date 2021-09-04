---
title: Makefile应知应会
tags:
  - - Makefile
  - - C/C++
categories:
  - - 学习笔记
    - C
abbrlink: '86862577'
date: 2021-08-30 14:30:32
---
> `Makefile` 可以简单的认为是一个工程文件的编译规则，描述了整个工程的编译和链接等规则。其中包含了哪些文件需要编译，哪些文件不需要编译，哪些文件需要先编译，哪些文件需要后编译，哪些文件需要重建等等。编译整个工程需要涉及到的，在 `Makefile` 中都可以进行描述。换句话说，`Makefile` 可以使得我们的项目工程的编译变得自动化，不需要每次都手动输入一堆源文件和参数。
> 
> 把要链接的库文件放在 `Makefile` 中，制定相应的规则和对应的链接顺序。这样只需要执行 `make` 命令，工程就会自动编译。每次想要编译工程的时候就执行 `make` ，省略掉手动编译中的参数选项和命令，非常的方便。
> 
> `Makefile` 支持多线程并发操作，会极大的缩短我们的编译时间，并且当我们修改了源文件之后，编译整个工程的时候，`make` 命令只会编译我们修改过的文件，没有修改的文件不用重新编译，也极大的解决了我们耗费时间的问题。
> 
> `gcc`使用相关内容见[链接](https://leex0.top/posts/4f2a16e6/)

## 1. 格式

```text
targets : prerequisites
command
```

相关说明如下：
-   targets：规则的目标，可以是 Object File（一般称它为中间文件），也可以是可执行文件，还可以是一个标签；
-   prerequisites：是我们的依赖文件，要生成 targets 需要的文件或者是目标。可以是多个，也可以是没有；
-   command：make 需要执行的命令（任意的 shell 命令）。可以有多条命令，每一条命令占一行。

Example：
```makefile
test:test.c
gcc -o test test.c
```
上述代码实现的功能就是编译 test.c 文件，通过这个实例可以详细的说明 Makefile 的具体的使用。其中 test 是的目标文件，也是我们的最终生成的可执行文件。依赖文件就是 test.c 源文件，重建目标文件需要执行的操作是`gcc -o test test.c`。这就是 Makefile 的基本的语法规则的使用。

使用 Makefile 的方式：首先需要编写好 Makefile 文件，然后在 shell 中执行 make 命令，程序就会自动执行，得到最终的目标文件。

## 2. 规则

### 2.1 显式规则

显式规则说明了，如何生成一个或多的的目标文件。这是由 Makefile 的书写者明显指出，要生成的文件，文件的依赖文件，生成的命令。

### 2.2隐晦规则

由于我们的 make 命名有自动推导的功能，所以隐晦的规则可以让我们比较粗糙地简略地书写 Makefile，这是由 make 命令所支持的。

### 2.3 变量的定义

在 Makefile 中我们要定义一系列的变量，变量一般都是字符串，这个有点像C语言中的宏，当 Makefile 被执行时，其中的变量都会被扩展到相应的引用位置上。

### 2.4 文件指示

其包括了三个部分，一个是在一个 Makefile 中引用另一个 Makefile，就像C语言中的 include 一样；另一个是指根据某些情况指定 Makefile 中的有效部分，就像C语言中的预编译 if 一样；还有就是定义一个多行的命令。有关这一部分的内容，我会在后续的部分中讲述。

### 2.5 注释

Makefile 中只有行注释，和 UNIX 的 Shell 脚本一样，其注释是用“#”字符，这个就像 C/C++ 中的“//”一样。如果你要在你的 Makefile 中使用“#”字符，可以用反斜框进行转义，如：“\#”。

## 3. 工作流程

在我们编译项目文件的时候，默认情况下，make 执行的是 Makefile 中的第一规则（Makefile 中出现的第一个依赖关系），此规则的第一目标称之为“最终目标”或者是“终极目标”
```makefile
main:main.o test1.o test2.o 《---终极  
gcc main.o test1.o test2.o -o main  
main.o:main.c test.h  
gcc -c main.c -o main.o  
test1.o:test1.c test.h  
gcc -c test1.c -o test1.o  
test2.o:test2.c test.h  
gcc -c test2.c -o test2.o
```
对这些 ".o" 文件为目标的规则处理有下列三种情况：

-   目标 ".o" 文件不存在，使用其描述规则创建它；
-   目标 ".o" 文件存在，目标 ".o" 文件所依赖文件在上一次 make 之后被修改过，则根据规则重新编译生成它；
-   目标 ".o" 文件存在，目标 ".o" 文件在上一次 make 之后没有被修改，则什么也不做。

## 4. 变量定义与使用

调用变量的时候可以用 "$(VALUE_LIST)" 或者是 "${VALUE_LIST}" 来替换，这就是变量的引用。实例：
```makefile
OBJ=main.o test.o test1.o test2.o
test:$(OBJ) 
gcc -o test $(OBJ)
```

### 变量的基本赋值

知道了如何定义，下面我们来说一下 Makefile 的变量的四种基本赋值方式：

1. 简单赋值 ( := ) 编程语言中常规理解的赋值方式，只对当前语句的变量有效。
2. 递归赋值 ( = ) 赋值语句可能影响多个变量，所有目标变量相关的其他变量都受影响。
3. 条件赋值 ( ?= ) 如果变量未定义，则使用符号中的值定义变量。如果该变量已经赋值，则该赋值语句无效。
4. 追加赋值 ( += ) 原变量用空格隔开的方式追加一个新值。

- 简单赋值
```makefile
x:=foo
y:=$(x)b
x:=new
test：
@echo "y=>$(y)"
@echo "x=>$(x)"
```
在 shell 命令行执行`make test`我们会看到:
```text
y=>foob  
x=>new
```

- 递归赋值
```makefile
x=foo
y=$(x)b
x=new
test：
@echo "y=>$(y)"
@echo "x=>$(x)"
```
在 shell 命令行执行`make test`我们会看到:
```text
y=>newb  
x=>new
```

- 条件赋值
```makefile
x:=foo
y:=$(x)b
x?=new
test：
@echo "y=>$(y)"
@echo "x=>$(x)"
```
在 shell 命令行执行`make test`我们会看到:
```text
y=>foob  
x=>foo
```

- 追加赋值
```makefile
x:=foo
y:=$(x)b
x+=$(y)
test：
@echo "y=>$(y)"
@echo "x=>$(x)"
```
在 shell 命令行执行`make test`我们会看到:
```text
y=>foob  
x=>foo foob
```

- 自动化变量
```makefile
test:test.o test1.o test2.o  
gcc -o $@ $^  
test.o:test.c test.h  
gcc -o $@ $<  
test1.o:test1.c test1.h  
gcc -o $@ $<  
test2.o:test2.c test2.h  
gcc -o $@ $<
```
这个规则模式中用到了 "$@" 、"$<" 和 "$^" 这三个自动化变量，对比之前写的 Makefile 中的命令，我们可以发现 "$@" 代表的是目标文件test，“$^”代表的是依赖的文件，“$<”代表的是依赖文件中的第一个。我们在执行 make 的时候，make 会自动识别命令中的自动化变量，并自动实现自动化变量中的值的替换，这个类似于编译C语言文件的时候的预处理的作用。

## 5. 目标文件搜索

在 Makefile 中可以这样写：

```text
VPATH := src
```

我们可以这样理解，把 src 的值赋值给变量 VPATH，所以在执行 make 的时候会从 src 目录下找我们需要的文件。

当存在多个路径的时候我们可以这样写：

```text
VPATH := src car
```

## 6. 示例

```makefile
exe=../release/libA.so

### 链接目标文件
$(exe):libA.o
    gcc -o $(exe) -lstdc++ -fPIC -shared -Xlinker libA.o
    
### 编译源文件
libA.o:libA.cpp
    gcc -lstdc++ -c libA.cpp
    
clean:
 -rm *.out *.o *.bak

exe=../release/main.out
### 链接目标文件
### -L ../release  用于指定libA.so所在目录
### -lA  链接库文件libA.so
$(exe):main.o
    gcc -o $(exe) -lstdc++ -Xlinker main.o -L ../release -lA
    
### 编译源文件
main.o:main.cpp
 gcc -lstdc++ -c main.cpp
clean:
 -rm *.out *.o *.bak
```

---
> 参考：
> https://zhuanlan.zhihu.com/p/359033384
