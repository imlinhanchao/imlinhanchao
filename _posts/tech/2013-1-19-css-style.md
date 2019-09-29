---
author: hancel.lin
date: 2013-01-15
title: CSS 样式层叠算法
tags: 
    - CSS
    - 层叠
    - 样式
    - HTML
    - 前端
    - 样式表
    - 算法
category: tech
layout: post
guid: urn:uuid:deb64e00-606e-4cb9-b75f-42b00442f7b7
---

通常，我们会使用 CSS 选择器来选择样式的应用范围。但是，这就有可能使不同 CSS 选择器相互覆盖范围。如：
```css
h1#header { color:navy; } 
#header { color:blue; }  
h1[title] { color:gray; } 
h1.title { color:purple; } 
.title { color:pink; } 
body h1 { color:green; } 
body>h1 { color:maroon; } 
h1 { color:red; } 
body { color:black; } 
```
那么，为了解决相同声明的冲突问题，层叠登场了！

下面将来讲一下层叠算法，也就是计算选择器特殊性的方法。

特殊性分为四个部分，用逗号隔开。比如：`0,0,0,0` 。依据特殊性权值的不同，选择器分为下面最基础的四类：

ID 选择器：特殊性权值加 `0,1,0,0`；

类选择器，属性选择器，伪类选择器：特殊性权值加 `0,0,1,0`；

元素选择器，为元素选择器：特殊性权值加 `0,0,0,1`；

结合符与通配符，如 `*` 、`+`、 `>` ：无特殊性；

下面给出几个范例：
```css
h1 { color:red; }                       /* = 0,0,0,1 */
p em { color:purple; }                  /* = 0,0,0,2 */
.grape { color:purple; }                /* = 0,0,1,0 */
*.bright { color:yellow; }              /* = 0,0,1,0 */
p.bright em.dark { color:maroon; }      /* = 0,0,2,2 */
#id216 { color:blue; }                  /* = 0,1,0,0 */
div#sidebar *[href] { color:silver; }   /* = 0,1,1,1 */
```
如果第 2 条规则与第 5 条同时用于一个`em`元素，那么第 5 条会胜出，元素将呈现紫红色。因为第 5 条的特殊性更强。

在这里值得注意的是如何比较特殊性的强弱。第5条强于第2条的关键是在于第二位的 2，而和第一位的 2 没有关系，有没有它都没关系。也就是第 3 条实际比第 2 条特殊性要强。特殊性的比较是从高位比较起的。一条特殊性为 `0,0,1,0` 的样式要比 特殊性为 `0,0,0,13` 要更强！

那么现在我们来看看文章开始时举的例子，计算一下他们的特殊性：
```css
h1#header { color:navy; } /* = 0,1,0,1 */
#header { color:blue; }   /* = 0,1,0,0 */
h1[title] { color:gray; } /* = 0,0,1,1 */
h1.title { color:purple; }/* = 0,0,1,1 */
.title { color:pink; }    /* = 0,0,1,0 */
body h1 { color:green; }  /* = 0,0,0,2 */
body>h1 { color:maroon; } /* = 0,0,0,2 */
h1 { color:red; }         /* = 0,0,0,1 */
body { color:black; }     /* = 0,0,0,1 */
```
这里面有几个值得注意的例子，第 6 条`body h1`与 第7条`body>h1`，他们的特殊性权重值是一致的（`>`没有任何特殊性），那么谁会最后胜出呢？神奇的是，CSS 样式除了可以通过特殊性（权值）来决定胜负外，还有一个“拳怕少壮”的规则。越靠近后面的样式越强。所以，那两条样式，谁写在后面，谁就会胜出。

接下来我们再来看看第 8 条`h1`和第 9 条`body`，如果在页面有个`h1`元素，应用了第 9 条样式，那么它会显现为黑色，那在第 9 条前面再写上第 8 条呢？如下：
```html
<html>
<head>
    <title>demo</title>
    <style type="text/css">
    h1 { color:red; }
    body { color:black; }
    </style>
</head>
<body>
    <h1>This is a head !</h1>
</body>
</html>
```
**This is a head !** 将显示什么颜色呢？正解是红色。虽然`body`写在后面，但是它对`h1`元素样式的影响却是通过继承`inherit`的方式应用的。而不是选择器，因此从权重上的计算，是没有任何特殊性的。因此`h1 { color:red; }`会胜出。在 CSS 中，有些样式声明可以被子元素继承，怎么才知道声明可以被继承呢？有两种方法：尝试，或查[文档](https://developer.mozilla.org/zh-CN/docs/Web/CSS)。

也许，细心的朋友会注意到，计算特殊性的四位数，目前为止我们只用了其中 3 位。那么第四位是用来做什么呢？

事实上，第四位正是留给样式权重的老大——内联样式。如下：
```html
<p style="color: red; margin-left: 20px">
This is a paragraph
</p>
```
内联样式正是通过 HTML 的`style`属性来声明的样式。它的特殊性是最高的。可以直接无视 CSS 样式表。除了……
```css
!important
```
没错！就是`important!`。从名字我们就看出来了，它真的是相当的重要。如下：
```html
<html>
<head>
    <title>demo</title>
    <style type="text/css">
    h1{color:blue!important;}
    </style>
</head>
<body>
    <h1 style="color:red;">This is a head !</h1>
</body>
</html>
```
虽然`h1`元素已经声明了内联样式`color:red`，但是在`!important`面前，不得不退位让贤。最后显示为蓝色。

关于 CSS 样式层叠的内容，就是这样~