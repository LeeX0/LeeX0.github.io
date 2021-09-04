---
title: Hexo博客与Fluid主题更新
tags:
  - - blog
  - - Hexo
  - - Fluid
categories:
  - [日常, 博客]
abbrlink: 76079a7b
date: 2021-03-22 13:30:58
---

> 博客较旧未更新，记录一下更新过程。
>
> 同时更改一下目前的代码高亮相关配置。
>
> **注：操作前建议将博客内容自行备份。**
>
> **所有操作建议都在博客目录下进行。**

## 更新npm

```bash
> sudo npm install -g npm
> npm -v
6.14.11
```

期间遇到错误：

```
reason: Hostname/IP does not match certificate's altnames: Host: registry.cnpmjs.org. is not in the cert's altnames: DNS:r.cnpmjs.org
```

执行`npm config set registry http://registry.npmjs.org`重新配置npm的registry即可。

## 更新node

```bash
> sudo npm install -g n
> sudo n stable
node -v
v14.16.0
```

## 更新Hexo

```bash
# 以下指令均在Hexo目录下操作，先定位到Hexo目录
# 查看当前版本，判断是否需要升级
> hexo version

# 全局升级hexo-cli
> npm i hexo-cli -g

# 再次查看版本，看hexo-cli是否升级成功
> hexo version

# 安装npm-check，若已安装可以跳过
> npm install -g npm-check

# 检查系统插件是否需要升级
> npm-check

# 安装npm-upgrade，若已安装可以跳过
> npm install -g npm-upgrade

# 更新package.json
> npm-upgrade

# 更新全局插件
> npm update -g

# 更新系统插件
> npm update --save

# 再次查看版本，判断是否升级成功
> hexo version
INFO  Validating config
hexo: 5.4.0
hexo-cli: 4.2.0
os: Darwin 20.3.0 darwin x64
node: 14.16.0
v8: 8.4.371.19-node.18
uv: 1.40.0
zlib: 1.2.11
brotli: 1.0.9
ares: 1.16.1
modules: 83
nghttp2: 1.41.0
napi: 7
llhttp: 2.1.3
openssl: 1.1.1j
cldr: 37.0
icu: 67.1
tz: 2020a
unicode: 13.0
```

## 更新Fluid主题

> 根据[主题介绍](https://github.com/fluid-dev/hexo-theme-fluid)，较为简单的方式为使用npm安装也易于维护。

```bash
npm install --save hexo-theme-fluid
npm update --save hexo-theme-fluid
```

### 主题配置说明

若之前采用解压方式使用主题，主题文件将会在`/blog/themes/fluid`下存储。

采用npm安装后，主题文件存放于`/blog/node_modules/hexo-theme-fluid`。

**采用npm安装后注意将之前存放的静态文件与配置同步到新存放位置。**

同时将两者配置文件进行同步，因为新版本中字段使用可能与老版本不同，直接覆盖配置文件可能导致无法使用。

Hexo 5.0.0 版本以上的用户，在博客目录下创建 `/blog/_config.fluid.yml` 文件，将主题的`/blog/node_modules/hexo-theme-fluid/_config.yml`内容复制过去。

以后如果修改任何主题配置，都只需修改 `_config.fluid.yml` 的配置即可。

### 代码高亮显示

之前的高亮显示不明显，修改一下高亮显示配置。

在之前的配置文件中`/blog/_config.yml`，高亮为如下配置：

```json
highlight:
  enable: true
  auto_detect: false
  line_number: true
  tab_replace: ''
  wrap: true
  hljs: false
prismjs:
  enable: false
  preprocess: true
  line_number: true
  tab_replace: ''
```

现选择`prismjs`为高亮配置：

```json
highlight:
  enable: false
  auto_detect: false
  line_number: true
  tab_replace: ''
  wrap: true
  hljs: false
prismjs:
  enable: true
  preprocess: true
  line_number: true
  tab_replace: ''
```

同时在`/blog/_config.fluid.yml`中将`lib: "highlightjs"`：

```json
highlight:
    enable: true
    line_number: true
    # Highlight library
    # Options: highlightjs | prismjs
    lib: "highlightjs"
    
    highlightjs:
      style: "Github Gist"
      bg_color: false
    prismjs:
      style: "default"
      preprocess: false
```

修改为`lib: "prismjs"`：

```json
highlight:
    enable: true
    line_number: true
    # Highlight library
    # Options: highlightjs | prismjs
    lib: "prismjs"
    
    highlightjs:
      style: "Github Gist"
      bg_color: false
    prismjs:
      style: "default"
      preprocess: false
```

---

> 参考：
>
> https://hexo.io/zh-cn/docs/syntax-highlight.html
>
> https://github.com/fluid-dev/hexo-theme-fluid
>
> https://www.imczw.com/post/tech/hexo5-next8-updated.html