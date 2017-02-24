---
author: hancel.lin
date: 2011-04-03
title: C语言实现月历
tags: 
    - C语言
    - 编程
    - 月历
category: tech
layout: post
guid: urn:uuid:35feec64-b6de-4b3f-a22b-d0845783b459
---

这是大一时做的一个用C语言实现的月历程序。

首先通过计算星期的通用公式计算出输入年份的1月1日是星期几：

```c
iw = iy + (iy - 1) / 4 - (iy - 1) / 100 + (iy - 1) / 400;
iw = iw - (iw / 7) * 7;
```
由于年份存在一个闰年与否的问题，因此，单输入月份`>1`时，需要判断月份是否为闰年，给变量`ir` 赋值2月份的日期数：

```c
if (im > 1)
{
	if (((iy % 4 == 0) && (iy % 100 != 0)) || (iy % 400 == 0)) ir = 29;
	else ir = 28;
}
```

那么，对于输出月历，只要知道了该月1号星期，则可输出整个月历。因此，我通过在1月1日星期数（现存于变量`iw`中）加上输入月份1号到1月1日天数再取余7，则可算出该月1日的星期数：
<!--more-->

```c
switch(im)
{
	case 12:
		iw=iw+30;     // 没有添加break;可以通过逐月累加，
	case 11:        // 计算出距离1月1日的天数
		iw=iw+31;
	case 10:
		iw=iw+30;
	case 9:
		iw=iw+31;
	case 8:
		iw=iw+31;
	case 7:
		iw=iw+30;
	case 6:
		iw=iw+31;
	case 5:
		iw=iw+30;
	case 4:
		iw=iw+31;
	case 3:
		iw=iw+ir;
	case 2:
		iw=iw+31;
	case 1:
		break;
	default:
		printf("输入错误！重新输入!n"); // 判断用户是否输入非1~12月
		goto stop;
}
iw=iw%7;
```

针对不同月份的日期数，使用if语句判断分类，然后调用不同的mon 函数参数输出月历：

```c
if(im == 1 || im == 3 || im == 5 || im == 7 || im == 8 || im == 10 || im == 12)
	mon(iw, 31);
```
初始输出星期时，用空格将星期数分隔开，一个汉字占位两个空格符，因此每个日期占位三个空格符。而对于个位数日期，则需先输出俩个空格符：

```c
void mon30(int z,int day)
{
	int i,n;
	n=z;
	for(i=0;i<z;i++)         //先输出1号前的所有空格，
		printf("   ");//每个星期占位3个空格符
	for(i=1;i<=day;i++)
	{
		if(i<10)
			printf(" ");
		printf(" %d",i);
		n++;
		if(n==7)
		{
			printf("n");
			n=0;
		}
	}
}
```
这样，就可以输出某年某月的月历了~

完整代码：

```c
#include<stdio.h>
void mon(int z, int day)；
void main()
{
	int im, iy, iw, ir; // im为月份，iy为年份，iw为星期，ir为闰年
	stop:printf("请输入年份：");
	scanf("%d", &iy);
	printf("请输入月份：");
	scanf("%d", &im);
	iw=iy + (iy - 1) / 4 - (iy - 1) / 100 + (iy - 1) / 400;
	iw=iw - (iw / 7) * 7;

	if(im > 1)
	{
		if(((iy % 4 == 0) && (iy % 100 != 0)) || (iy % 400 == 0)) ir=29;
		else ir=28;
	}

	switch(im)
	{
		case 12:
			iw = iw + 30;
		case 11:
			iw = iw + 31;
		case 10:
			iw = iw + 30;
		case 9:
			iw = iw + 31;
		case 8:
			iw = iw + 31;
		case 7:
			iw = iw + 30;
		case 6:
			iw = iw + 31;
		case 5:
			iw = iw + 30;
		case 4:
			iw = iw + 31;
		case 3:
			iw = iw + ir;
		case 2:
			iw = iw + 31;
		case 1:
			break;
		default:
		{
			printf("输入错误！重新输入!\n");
			goto stop;
		}
	}
	iw = iw % 7;
	printf(" 日 一 二 三 四 五 六\n");
	if(im == 1 || im == 3 || im == 5 || im == 7 || im == 8 || im == 10 || im == 12)
		mon(iw, 31);
	if(im == 2 && ir == 28)
		mon(iw, 28);
	if(im == 2 && ir == 29)
		mon(iw, 29);
	if(im == 4 || im == 6 || im == 9 || im == 11)
		mon(iw, 30);
	printf("\n");
	getch();
}

void mon(int z, int day)
{
	int i,n;
	n=z;
	for(i = 0; i < z; i++)
		printf("   ");
	for(i = 1; i <= day; i++)
	{
		if(i < 10) printf(" ");
		printf(" %d", i);
		n++;
		if(n == 7)
		{
			printf("\n");
			n = 0;
		}
	}
}
```

后记
---
时隔三年重制版：https://coding.net/u/imlinhanchao/p/lite-code/git/tree/master/calendar (2015-12-11)
