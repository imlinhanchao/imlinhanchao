---
author: hancel.lin
head: /blog/img/logo/logo_128x128.png
date: 2013-12-15
title: 读取不定长字符串输入
tags: C语言,字符串,输入
category: tech
layout: default
summary: C语言通常使用`scanf`处理输入，如果要读取字符串，那么就需要定义一个字符数组（`char[]`）。可是，如果数组定义长度不足，就可能发生溢出。

在C语言里有个可以用来读取字符的函数(`getchar`)，我们可以利用这个函数来实现不定长的字符串输入...
---
C语言通常使用`scanf`处理输入，如果要读取字符串，那么就需要定义一个字符数组（`char[]`）。可是，如果数组定义长度不足，就可能发生溢出。在C语言里有个可以用来读取字符的函数(`getchar`)，我们可以利用这个函数来实现不定长的字符串输入。下面我们就来讲讲如何做到这一点。

首先，说一下原理：`getchar`每次只能读取一个字符。因此，我通过循环使用`getchar`逐个读取字符的方式，将所有输入字符读取。

那么，我们要先解决一个问题：

>什么时候结束循环不再读取呢？

当我们输入字符串后，按下Enter键，那么输入的字符串就会被程序接收，写入输入缓冲区的除了刚才输入的字符串，还会有一个换行符`\n`，因此`getchar`当读取到字符`\n`时，即可跳出循环，完成读取。跳出循环后我们还要在后面加上`\0`，这样，它才能成为一个真正的字符串。

第二问题：

>存放在哪儿？

当我们使用`scanf`读取字符串时，我们将字符串存放在字符数组（`char[]`）里面，那么我们使用循环读取字符时，就需要有一个同样连续的内存空间来存放读取到的字符。而且，我们因为不知道到底会读取到多长的字符串，长度是不固定的，所以使用`malloc`来动态申请一个连续的内存空间。

因此，我们准备两块内存指针：

{% highlight c %}
char* str;
char* _str;
```

先给其中一个分配2个`char`的内存空间(一个用来存`\0`)，同时用i来记录输入字符串的个数。

{% highlight c %}
int i = 1;
str = (char*)malloc(sizeof(char) * (i + 1));
```

然后，再用循环读取字符，并把它存到申请的内存空间。

{% highlight c %}
while('\n' != (str[i - 1] = getchar()))
{
     i++;
     ...
}
```

每次我们读取到一个字符时，就将`i`加一。所以循环体开始的时候是`i++`（刚读完一个字符）。

现在，重点来了，因为我们要预先申请多一个长度的内存，然后才能继续存放接下来要读的字符。所以我们需要把str释放掉，然后重新申请空间，可是，直接释放会把原来读的字符都弄丢，所以就到`_str`出场了。

我们先给`_str`申请与`str`相同长的内存空间 。然后，把`str`的内容拷贝到_str里。这时，就可以把`str`释放掉了。在给`str`重新申请内存空间成功后，把`_str`的内容拷贝回来，然后释放掉`_str`就好了。

{% highlight c %}
_str = (char*)malloc(strlen(str) + 1);
str[i - 1] = '\0';
strcpy(_str, str);
free(str);
str = (char*)malloc(sizeof(char) * (i + 1));
if(NULL == str)
{
     free(_str);
     printf("No enough memory!");
     return NULL;
}
strcpy(str, _str);
free(_str);
```

值得注意的是，在给`str`重新申请内存空间后，需要判断一下`str`内存申请是否成功。如果失败（`NULL == str`），我们需要先将`_str`释放掉（防止出现内存泄漏），再`return NULL`。

最后，我们只要将`\0`加上，把`str`的内存地址返回，就大功告成了。

{% highlight c %}
str[i - 1]='\0';
return str;
```

附上完整代码：

{% highlight c %}
char* getstr()
{
        char* str;
        char* _str;
        int i = 1;
        str = (char*)malloc(sizeof(char) * (i + 1));
        while('\n' != (str[i - 1] = getchar()))
        {
                i ++;
                _str = (char*)malloc(strlen(str) + 1);
                str[i - 1] = '\0';
                strcpy(_str, str);
                free(str);
                str = (char*)malloc(sizeof(char) * (i + 1));
                if(NULL == str)
                {
                        free(_str);
                        printf("No enough memory!");
                        return NULL;
                }
                strcpy(str, _str);
                free(_str);
        }
        str[i - 1] = '\0';
        return str;
}
```
