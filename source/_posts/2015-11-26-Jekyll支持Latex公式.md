---
title: "Jekyll使用MathJax来显示数学式"
date: 2015-11-26 00:00:00
category: 疑难解惑
tags: [Latex公式,MathJax,Jekyll]
---
## 网页中嵌入公式的方法
> 我自己使用后的感觉是，虽然，MathJax很方便，但是有些时候会出现无法识别的问题。所有很多时候我是结合起来用的。

1. 使用网上的一些服务生成公式图片，如[这里](http://www.codecogs.com/latex/eqneditor.php)
2. 使用[MathJax](http://docs.mathjax.org/)来显示数学式.

## 使用MathJax显示公式
1. 修改html头部

在需要使用的页面开头加上这么一句，在Jekyll下可以通过修改default.html加上。

```JavaScript
<script type="text/javascript"
  src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML">
</script>
```

2. 然后直接调用
公式和Latex公式一致的，没有太多难点，可以参考[这个站点](http://www.codecogs.com/latex/eqneditor.php)