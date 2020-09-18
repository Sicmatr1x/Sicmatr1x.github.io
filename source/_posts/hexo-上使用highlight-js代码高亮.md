---
title: hexo 上使用highlight.js代码高亮
date: 2018-08-16 11:43:46
categories:
    - 杂项
tags: 
    - hexo
---

Hexo 自带了在渲染网页时进行代码高亮的功能

然鹅本站使用的theme: [Shen-Yu/hexo-theme-ayer](https://github.com/Shen-Yu/hexo-theme-ayer)本身不含有代码高亮且与hexo默认的代码高亮不兼容，顾使用 highlight.js 来进行代码高亮～

首先要把 Hexo 和 Material Theme 自带的高亮关掉(。・`ω´・)

打开根目录下的`_config.yml`文件：

```yml
# Writing
highlight:
  enable: false
  line_number: false
  auto_detect: false
  tab_replace:
```

然后需要在theme里面引入 highlight.js 的本体和启用初始化方法( つ•̀ω•́)つ

以`Shen-Yu/hexo-theme-ayer`为例:

在`\themes\ayer\layout\_partial\head.ejs`文件的`<head>`DOM里增加：

```html
  <link rel="stylesheet" href="https://cdn.jsdelivr.net/gh/highlightjs/cdn-release@latest/build/styles/vs2015.min.css"><!- highlight.js ->
  <script src="https://cdn.jsdelivr.net/gh/highlightjs/cdn-release@latest/build/highlight.min.js"></script>
  <script>
    hljs.initHighlightingOnLoad();
  </script>
```

[参考](https://akarin.dev/2019/04/19/use-hljs-in-material-theme/)
