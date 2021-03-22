---
author: "李昌"
title: "hugo中的公式问题"
date: "2021-03-18"
tags: ["Latex", "hugo"]
categories: ["杂记"]
ShowToc: true
TocOpen: true
---

hugo默认不支持latex公式，为了在我们的博客上显示数学公式，我们需要使用`mathjax`.

**使用方法**
对于hugo来说，我们只需要为每个页面加上
```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.2/MathJax.js?config=TeX-MML-AM_SVG"></script>
```
就行了。  
可以通过在`themes/{themeName}/layouts/partials/footer.html`中添加来使mathjax包含到每个页面中。

**书写公式**
行内公式可以使用`$$f(x)= \cos x$$`来编辑,效果为$f(x)= \cos x$  
行间公式可使用如下格式：
```markdown
$$
\frac{ x^{2} }{ k+1 }\qquad
$$
```
效果为：
$$
\frac{ x^{2} }{ k+1 }\qquad
$$
