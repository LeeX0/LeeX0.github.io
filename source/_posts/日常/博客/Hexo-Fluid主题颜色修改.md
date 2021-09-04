---
title: Hexo Fluid主题颜色修改
tags:
  - - blog
  - - Hexo
  - - Fluid
categories:
  - [日常, 博客]
abbrlink: c31bada6
date: 2021-06-22 14:25:23
---

> `Hexo Fluid`当前配色下，段内代码显示不明显，更改一下配色。
>
> 更改段内代码配色后顺便修改了配套的超链接配色。

## 1. 定位

段内代码的样式在`Blog/node_modules/hexo-theme-fluid/layout/_partial/css.ejs`中指明为

```css
<%- css_ex(theme.static_prefix.github_markdown, 'github-markdown.min.css') %>
```

其中前缀在`Blog/_config.fluid.yml`中指明为

```yaml
github_markdown: https://cdn.jsdelivr.net/npm/github-markdown-css@4.0.0/
```

## 2. 修改内容

### 2.1 静态css

可以将`github-markdown.min.css`修改后放入`Blog/node_modules/hexo-theme-fluid/source/lib/github-markdown`。

在`Blog/_config.fluid.yml`中指明内部静态位置

```yaml
github_markdown: /lib/github_markdown
```

### 2.2 覆盖

直接修改`Blog/node_modules/hexo-theme-fluid/source/css/_pages/_post/post.styl`内容，会覆盖`github-markdown.min.css`展示。

此处我仅修改了`markdown`中段内代码样式，在文件尾增加

```stylus
.markdown-body code{
  color #ff7473
  background #f4f9ec
}
```

## 2.3 超链接

修改段内代码颜色后，想要修改超链接颜色保持一致。

Fluid的配置文件`Blog/_config.fluid.yml`中提供选项修改。

```yaml
color:
	···
  # 文章超链接字体色
  # Color of post link
  post_link_color: "#0366d6"
  post_link_color_dark: "#1589e9"

# 修改为：

color:
	···
  # 文章超链接字体色
  # Color of post link
  post_link_color: "#8f2d56"
  post_link_color_dark: "#1589e9"
```

---

## 2021年08月05日更新

*2.2*中的修改，会将Code Block中默认颜色修改，会导致Code Block中大面积普通文本变为彩色，重点不突出，目前收回修改，之后有时间看看能不能有更好的方案。

```stylus
.markdown-body code{
  color #ff7473
  background #f4f9ec
}
```

---

> 参考：
>
> Fluid配置指南：https://hexo.fluid-dev.com/docs/guide/
>
> 颜色搭配：https://www.webdesignrankings.com/resources/lolcolors/
>
> Fluid mod：https://github.com/lirengui/hexo-theme-fluid-mod/
