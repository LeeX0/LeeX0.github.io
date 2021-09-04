---
title: 使用conda管理环境
tags:
  - - Python
  - - conda
categories:
  - [学习笔记, Python]
abbrlink: b42d039
date: 2020-10-24 19:05:08
---

> Python中具有数量庞大且功能相对完善的标准库和第三方库。通过对库的引用，能够实现对不同领域业务的开发。然而，正是由于库的数量庞大，对于管理这些库以及对库作及时的维护成为既重要但复杂度又高的事情。
>
> [Anaconda](https://www.anaconda.com/products/individual#macos)就是可以便捷获取包且对包能够进行管理，同时对环境可以统一管理的发行版本。Anaconda包含了conda、Python在内的超过180个科学包及其依赖项。

### 1. Anaconda安装

在[Anaconda](https://www.anaconda.com/products/individual)中下载对应的安装器安装。

例通过图形化界面安装完MacOS Anaconda，如果成功安装，通过`condal list`可以看到相关信息。

在终端中输入`python`。如果Anaconda被成功安装并且可以运行，则可看到Anaconda字样。

```python
Python 3.7.6 (default, Jan  8 2020, 13:42:34)
[Clang 4.0.1 (tags/RELEASE_401/final)] :: Anaconda, Inc. on darwin
```

### 2. 管理conda环境

> ```bash
> conda --version # 验证conda安装
> ```

#### 更新conda

```bash
conda update conda
```

#### 创建新环境

```xml
conda create --name <env_name> <package_names>
```

注意：

- `<env_name>`即创建的环境名。建议以英文命名，且不加空格，名称两边不加尖括号“<>”。

- `<package_names>`即安装在环境中的包名。名称两边不加尖括号“<>”。

- `--name`同样可以替换为`-n`。

  ```bash
  # 创建名为python2的环境，python版本为2.7
  conda create --name python2_test python=2.7
  # 创建名为python3的环境，python版本为3.5，同时创建numpy和pandas包
  conda create -n python3 python=3.5 numpy pandas
  ```

> 注：默认情况下，新创建的环境将会被保存在`/Users/<user_name>/anaconda3/envs`目录下，其中，`<user_name>`为当前用户的用户名。

#### 使用环境

```bash
# 激活环境
conda activate <env_name>
source activate <env_name>

# 示例
conda activate python2_test
source activate python2_test

# 退出环境
conda deactivate
source deactivate

# 显示已创建环境
conda env list
conda info -e
conda info --envs
```

#### 复制环境

```csharp
conda create --name <new_env_name> --clone <copied_env_name>
```

#### 删除环境

```csharp
conda remove --name <env_name> --all
```

### 3. 管理包

```bash
### 查找包 ###
# 精确查找
conda search --full-name <package_full_name>
# 模糊查找
conda search <text>
# 获取当前环境包信息
conda list
```

```bash
### 安装包 ###
# 在指定环境中安装包
conda install --name <env_name> <package_name>
# 在当前环境中安装包
conda install <package_name>
```

```bash
### 卸载包 ###
# 在指定环境中卸载包
conda remove --name <env_name> <package_name>
# 在当前环境中卸载包
conda remove <package_name>
```

```bash
### 更新包 ### 
conda update --all
conda upgrade --all
conda update <package_name>
conda upgrade <package_name>
# 更新多个指定包，空格隔开即可
conda update pandas numpy matplotlib
```

---

> 参考：
>
> https://www.jianshu.com/p/62f155eb6ac5