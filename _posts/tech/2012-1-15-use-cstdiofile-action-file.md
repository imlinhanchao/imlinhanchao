---
author: hancel.lin
date: 2012-1-15
title: 使用CStdioFile操作文件
tags: C++, CFile, Create, CStdioFile, file, mode, Read, Write
category: tech
layout: default
---
前阵子做毕业设计，需要用到一些对本地文件的操作，没有使用传统的CFile，用了`CFile`的继承类`CStdioFile`，感觉简易直接许多。下面简单介绍一下一些比较常用的文件操作：

打开文件
---

{% highlight cpp %}
file.Open(sFileName, CFile::modeCreate | CFile::modeReadWrite | CFile::modeNoTruncate);  
{% endhighlight %}

打开文件主要需要传入两个参数，`sFileName`——文件名；文件打开模式。
几种比较常见的文件打开模式：
- `CFile::modeCreate` 以新建方式打开，如果文件不存在，新建；如果文件已存在，把该文件长度置零，即清除文件原有内容；
- `CFile::modeNoTruncate` 以追加方式打开，如果文件存在，打开并且不将文件长度置零，如果文件不存在，会抛出异常。一般与`CFile::modeCreate`一起使用，则文件不存在时，新建一个文件；存在就进行追加操作；   
- `CFile::modeWrite` 以只写模式打开；
- `CFile::modeRead` 以只读模式打开；
- `CFile::modeReadWrite` 以读写模式打开。

读文件
---

{% highlight cpp %}
file.ReadString(sLine);  
{% endhighlight %}

将文件逐行读出，写入到`sLine`字符串里。
如果需要读出文件所有内容，可以用下面的方法：

{% highlight cpp %}
while(file.ReadString(sLine))  
{  
    sFileData += sLine + "\r\n";  
}  
{% endhighlight %}
这里用`\r\n`来为字符串加上换行。

写入文件
---

{% highlight cpp %}
file.WriteString(sLine);  
{% endhighlight %}

这里很值得注意一下，如果文件的打开模式设置了`CFile::modeNoTruncate`，那么字符串将以追加的形式写入，并且是从文件指针现在所处位置写起。
比如：

{% highlight cpp %}
CString sFileName("test.txt"), sLine("");  
CStdioFile file;  
// 创建文件"test.txt"，写入"1234567890"  
file.Open(sFileName, CFile::modeCreate | CFile::modeWrite);  
file.WriteString("1234567890");  
file.Close();  
// 追写入"abc"  
file.Open(sFileName, CFile::modeCreate | CFile::modeNoTruncate | CFile::modeWrite);  
file.WriteString("abc");  
file.Close();  
// 读出第一行字符串，并用消息框弹出  
file.Open(sFileName, CFile::modeRead);  
file.ReadString(sLine);  
file.Close();  

MessageBox(sLine);  
{% endhighlight %}
最终将弹出`abc4567890`
那么如果我们其实是想写入在文件末尾，即弹出`1234567890abc`，那该如何？
只要在`file.WriteString("abc");`前加入一句`file.SeekToEnd();`。这一句的作用在于将文件指针移动到文件末尾。

关闭文件
---
每个打开的文件都需要关闭，否则天知道会出什么事儿~

{% highlight cpp %}
file.Close();  
{% endhighlight %}
