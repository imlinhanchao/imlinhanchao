---
author: hancel.lin
date: 2011-10-13
title: 关于sizeof与&a的疑问
tags: 
    - C语言
    - sizeof
    - 指针
    - 数组
category: tech
layout: post
guid: urn:uuid:9971330a-192e-4921-95f6-a5143871204a
---
`sizeof`是一个在C语言里常被人误会的的东西。许多人的以为它是个函数，其实不然。只要查一下 [C语言保留关键字列表](http://zh.wikipedia.org/wiki/C%E8%AF%AD%E8%A8%80#.E4.BF.9D.E7.95.99.E5.85.B3.E9.94.AE.E5.AD.97)，就会发现`sizeof`赫然在表上。或者查一下C语言运算符优先级方面的资料，也可以在里面发现`sizeof`的身影。

不过，也难怪人们会误会`sizeof`是个函数，谁叫它的身后总是跟着一对括号呢~那么大家可以去试编译下面这个程序:

``` c
int main()  
{  
    int n, l;  
    l = sizeof n;  
    printf("%d", l);  
    return 0;
}  
```

你会发现，原来这样也是可以的！那么如果把第二句的`sizeof n`换成`sizeof int`。又会如何呢？大家可以去试一下。

其实，这次文章主要不是想讲`sizeof`，而是`&a`，前几日在拿`sizeof`做实验时发现的一些问题。

我们知道，在C语言里，无论是什么指针变量，其空间总是占有**4**个字节。也就是说：
<!--more-->

``` c
int main()  
{  
    int n;  
    char c;  
    printf("%d, %d", sizeof(&n), sizeof(&c));  
    return 0;
}  
```
输出结果都是`4`。
但是，如果将程序改变一下：

``` c
int main()  
{  
    int a[5];  
    char c[5];  
    printf("%d,%d",sizeof(&a),sizeof(&c));  
}  
```
这样，他们输出的结果就不同了，变成了 `20,5` 。在Turbo C 中则是 `10,5` 。

这是肿么了？！
