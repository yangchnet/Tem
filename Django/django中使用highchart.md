---
title: "django中使用highchart"
date: 2021-01-22T17:27:21+08:00
draft: false
---
## django中使用highchart

### 引入js文件
> 顺序不能错

```html
<script src="https://code.jquery.com/jquery-3.1.1.min.js"></script>  
<script src="https://code.highcharts.com/highcharts.js"></script>  
```


### 一点小插曲
在刚开始使用highchart时，由于使用的是继承模板，导致js文件引入的顺序没有把握好，导致不能显示图像，因此将上面两句提到base.html的前面，然后可以显示图像。
