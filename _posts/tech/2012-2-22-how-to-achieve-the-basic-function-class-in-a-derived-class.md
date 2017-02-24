---
author: hancel.lin
date: 2012-2-22
title: 如何在派生类中实现类的基本函数
tags: 
    - C++
    - 构造函数
    - 析构函数
    - 派生类
    - 继承
    - 赋值函数
category: tech
layout: post
guid: urn:uuid:7fbd5d95-7459-4c36-89f9-4a5ab3a713d2
---
基类的构造函数、析构函数、赋值函数都不能被派生类继承。如果类之间存在继承关系，在编写上述基本函数时应注意以下事项：

派生类的构造函数应在其初始化表里调用基类的构造函数。

基类与派生类的析构函数应该为虚（即加`virtual`关键字）。例如：

```cpp
#include<iostream.h>  
class Base  
{  
   public:  
       virtual ~Base() { cout<< "~Base" << endl ; }
};

class Derived : public Base  
{  
   public:  
       virtual ~Derived() { cout<< "~Derived" << endl ; }
};

void main(void)  
{  
   Base * pB = new Derived; // upcast  
   delete pB;  
}  
```
<!--more-->

输出结果为：

```
~Derived
~Base
```

如果析构函数不为虚，那么输出结果为

`~Base`

在编写派生类的赋值函数时，注意不要忘记对基类的数据成员重新赋值。例如：

```cpp
class Base  
{  
   public:  
       …  
       Base & operate =(const Base &other); // 类Base的赋值函数  
   private:  
       int m_i, m_j, m_k;  
};  
class Derived : public Base  
{  
   public:  
       …  
       Derived & operate =(const Derived &other); // 类Derived的赋值函数  
   private:  
       int m_x, m_y, m_z;  
};  

Derived & Derived::operate =(const Derived &other)  
{  
   //(1)检查自赋值  
   if(this == &other)  
       return *this;  
   //(2)对基类的数据成员重新赋值  
      Base::operate =(other); // 因为不能直接操作私有数据成员  
   //(3)对派生类的数据成员赋值  
      m_x = other.m_x;  
      m_y = other.m_y;  
      m_z = other.m_z;  
   //(4)返回本对象的引用  
   return *this;  
}  
```
