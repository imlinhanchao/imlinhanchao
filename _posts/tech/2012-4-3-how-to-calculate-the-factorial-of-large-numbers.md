---
author: hancel.lin
date: 2012-04-03
title: 如何计算大数阶乘
tags: 
    - C语言
    - 大数计算
    - 数组
    - 进位
    - 阶乘
category: tech
layout: post
guid: urn:uuid:1eb6a092-705b-49d7-ab96-0fb7411dde41
---

[阶乘](https://zh.wikipedia.org/wiki/%E9%9A%8E%E4%B9%98)，有很多数学的方法可以计算出来。而对于计算机来说，最简单的自然是按部就班的从1乘到n。但是，在C语言里，本来挺简单的程序却被一个东西给限制住了——数据类型。C语言里没有长度很大的数据类型，如果只是计算10!还好，但是如果要计算1000!，C语言的数据类型就吃不消了。

但是，通过一些算法的设计，我们可以让C语言进行这样大数据的处理。

我们可以使用数组来进行处理，定义一个足够多位的数组，每一个数组元素存放数据的一位。这样，就可以计算巨大的数据类型了。

首先，我们自然需要先定义一个足够长的数组，同时给它初始化：

```c
int data[10000] = {0};
```
<!--more-->

在这里，我们可以计算位数达到10000位的数字了。所以计算1000!绰绰有余~ :mrgreen:

由用户输入将计算的阶乘：

```c
scanf("%d", &n);
```

我们先将第一位初始化为1：

```c
data[0] = 1;
```

接着我们需要开始书写一个循环：

```c
for (i = 1 ; i <= n; i++)
```

从`i`一直循环到`n`，这没有任何问题。关键在于循环体的内容。我们需要让每个位数都乘以`i`，然后再让`data[]`数组里大于9的数字进位。因此，我们需要先写一个循环来让每个位数都乘以`i`：

```c
for (j = 0; j <= len; j++)  
    data[j] *= i;  
```
也许你注意到了`len`，这是一个`int`变量，用来记录当前`data[]`数组里有多少位是有效的。

接着，就是进行进位了：

```c
for (j = 0; j <= len; j++)  
{  
     if (data[j] > 9)  
     {  
          for (k = j; k <= len; k++)  
          {  
               if (data[len] > 9) len++; // 最顶端数字大于9时，就需要增加有效位数了  
               data[k+1] = data[k] / 10 + data[k + 1];  
               data[k] = data[k] % 10;  
           }  
      }  
}  
```

最后，我们需要将`data[]`数组倒序输出，因为我们是将数字从头记录的：

```c
for (i = len; i >= 0; i--)  
    printf("%d", data[i]);  
```

完整代码：

```c
#include "stdio.h"  
#default B 10000  
int main()  
{  
    int data[B] = {0}, n = 1, i = 1, len = 1, j = 1, num, k = 1, m = 0;  
    data[0] = 1;  
    scanf("%d", &n);  
    for (i = 1; i <= n && len < B; i++)  
    {  
        for (j = 0; j <= len; j++)  
            data[j] *= i;  
        for (j = 0; j <= len; j++)  
        {  
            if (data[j] > 9)  
            {  
                for (k = j; k <= len; k++)  
                {  
                    if (data[len] > 9) len++;  
                    data[k+1] = data[k] / 10 + data[k + 1];  
                    data[k] = data[k] % 10;  
                }  
            }  
        }  
    }  
    if (i <= n)  
    {  
        printf("There is no sufficient number of bits.");  
        return 1;  
    }  
    for (i = len; i >= 0; i--)  
        printf("%d", data[i]);  
    printf("\n");  
    return 0;  
}  
```
编译结果：http://ideone.com/NfMt5
