---
title: 如何用css画一个三角形？
date: 2019-07-20 17:52:39
tags: CSS
categories: 前端
---

**首先观察测试如下代码：**


```html
<style type="text/css">
  div {
    width: 100px;
    height: 100px;
    border-color: red orange yellow green;
    border-width: 20px;
    border-style: solid;
    background: white;
  }
</style>
```
<!-- more -->

**按照均分法则，上、下、左、右四个边框是占比完全等分盒子四周的，要想等分四条边框就只有这样：**

![image](https://img-blog.csdnimg.cn/20190720174259267.png)


**现在尝试把其中三条边框颜色设置成透明色：**

```html
border-color: transparent transparent yellow transparent;
```


**再来观察：**

![image](https://img-blog.csdnimg.cn/2019072017473071.png)

**现在的形状是梯形，很容易想到是因为现在盒子是有宽度和高度的，把宽高设成0：**
```html
width: 0;
height: 0;
```

**就成了这个样子：**

![image](https://img-blog.csdnimg.cn/20190720175142505.png)


**大功告成，这就是我们要得到的三角形了！**