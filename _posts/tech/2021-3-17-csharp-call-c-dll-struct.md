---
author: hancel.lin
date: 2021-03-17
title: C# 调用 C 类型 dll 入参为 struct 的问题示例
tags: 
    - C#
    - C
    - dll
    - struct
category: tech
layout: post
guid: urn:uuid:a368179e-380b-455a-b113-4aee4488cdb4
---

C# 可以通过 `DllImport` 的方式引用 C 类型的 dll。但很多 dll 的参数不会是简单的基础类型，而是结构体 `struct`。因此就需要在 C# 端定义同样的结构体类型，才能实现调用 C 类型 dll。这里例举几种不同的结构体情况，以及其对应的解决方案。

<!--more-->

## 基础调用方式

对于一个结构体类型：
```c
typedef struct DATA
{
    int nNumber;
    float fDecimal;
};
```
在 C# 端就需要定义为
```csharp
[StructLayout(LayoutKind.Sequential)]
public struct DATA
{
    public int nNumber;
    public float fDecimal;
}
```

## 包含字符数组

对于一个包含字符数组的结构体类型：
```c
typedef struct DATA
{
    int nNumber;
    float fDecimal;
    char szString[256];
};
```
在 C# 端就需要使用 `Marshal`设置数据空间大小，同时最好定义一个初始化函数与 `get` 的定义
```csharp
[StructLayout(LayoutKind.Sequential)]
public struct DATA
{
    void alloc() {
        szString = new char[256];
    }

    string sString {
        get {
            int nLength = 256;
            string sData = "";
            for (int i = 0; i < nLength; i++)
            {
                if (szData[i] == '\0') break;
                sData += szData[i];
            }
            return sData;
        }
    }

    public int nNumber;
    public float fDecimal;
    [MarshalAs(UnmanagedType.ByValArray, SizeConst = 256)]
    char[] szString;
}
```

## 包含字符二维数组
对于一个包含字符二维数组的结构体类型：
```c
typedef struct DATA
{
    int nNumber;
    float fDecimal;
    char szString[6][256];
};
```
在 C# 端同样需要使用 `Marshal`设置数据空间大小，需要将两个 Size 相乘，并定义一个初始化函数。同时在做一个 `get` 的定义。
```csharp
[StructLayout(LayoutKind.Sequential)]
public struct DATA
{
    void alloc() {
        szString = new char[256 * 6];
    }

    public string[] sStrings
    {
        get {
            int nSize = 6, nLength = 256;
            string[] sDatas = new string[nSize];
            for (int i = 0; i < nSize; i++)
            {
                for (int j = 0; j < nLength; j++)
                {
                    if (szData[i * nLength + j] == '\0') break;
                    sData += szData[i * nLength + i];
                }
                sDatas[i] = sData;
            }
            return sDatas;
        }
    }

    public int nNumber;
    public float fDecimal;
    [MarshalAs(UnmanagedType.ByValArray, SizeConst = 256 * 6)]
    char[] szStrings;
}
```

## dll 入参为结构体数组
若有一个这样的 C dll 函数定义：
```c
void FnCall(DATA* datas);

// 调用方式
DATA datas[10];
fnCall(datas);
```
那么，在 C# 中要实现等价调用：
```csharp
// 首先 Import 函数
[DllImport("Module.dll")]
public static extern void FnCall(IntPtr pInfo); // 注意入参要定义为指针

// 再定义定义结构体数组
int nCount = 10;
DATA datas = new DATA[nCount];

// 再分配内存空间
int nSize = Marshal.SizeOf(typeof(DEVICE_INFO));
IntPtr Dataptr = Marshal.AllocHGlobal(nSize * nCount);

// 调用函数
FnCall(Dataptr);

// 复制数据到结构体中
for (int i = 0; i < nCount; i++)
{
    IntPtr ptr = (IntPtr)((UInt32)Dataptr + i * size);
    datas[i] = (DEVICE_INFO)Marshal.PtrToStructure(ptr, typeof(DEVICE_INFO));
}

// 释放内存空间
Marshal.FreeHGlobal(Dataptr);
```

另外，如果你要调用的 dll 是非 C 类型 dll，而是 C++ Class。那么我们就可以将其再包装一层，转换为 C 类型 dll。

例如：

```cpp
class Example {
public:
  int MethodCall();
};
```

那么就可以编写 C 类型的 dll。
```c
extern "C" {
    Example* Example_New() { 
        return new Example(); 
    }
    int Example_MethodCall(Example* p) { 
        return p->MethodCall(); 
    }
    void Example_Delete(Example* p) { 
        delete p; 
    }
}
```
C# 那边就这样导入
```csharp
[DllImport("Module.dll")]
public static extern IntPtr Example_Create();

[DllImport("Module.dll")]
public static extern int Example_MethodCall(IntPtr value);

[DllImport("Module.dll")]
public static extern void Example_Delete(IntPtr value);

// 调用方式
IntPtr p = Example_Create();
Example_MethodCall(p);
Example_Delete(p);
```

至于 C 类型 dll 中其他类型变量在 C# 的对应，则可以参考 Microsoft 的[文档](https://docs.microsoft.com/zh-cn/dotnet/framework/interop/marshaling-data-with-platform-invoke)。