---
author: hancel.lin
date: 2019-01-16
title: 64 位系统下注册表的读写问题
tags:
    - Windows
    - 注册表
    - 64 bit
category: tech
status: publish
layout: post
guid: urn:uuid:2AA9A909-775A-41F4-866B-F0999EAD6BD5
---

最近需要做一个自动化程序，需要自动进入安全模式后自动执行我的程序。找到资料只要在注册表`HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon`中的`Userinit`最后加上程序路径，就可以达到目的。

嗯，操作注册表，是挺简单的。一顿操作猛如虎，就写好了。

```cpp
const TCHAR* szSubKey= _T("SOFTWARE\\Microsoft\\Windows NT\\CurrentVersion\\Winlogon");
HKEY hKey;
TCHAR szValue[100] = _T("");
TCHAR szAppPath[100] = _T("D:\\App.exe");
DWORD dwType = REG_SZ;
DWORD dwSize = sizeof(szValue);
LSTATUS lstc = RegOpenKeyEx(HKEY_LOCAL_MACHINE, szSubKey, 0, KEY_READ | KEY_WRITE, &hKey);
if(ERROR_SUCCESS != lstc) return 0;
lstc = RegQueryValueEx(hKey, _T("Userinit"), NULL, &dwType, (BYTE*)szValue, &dwSize);
if(ERROR_SUCCESS != lstc) goto CLOSE;
_tcscat(szValue, szAppPath);
lstc = RegSetValueEx(hKey, _T("Userinit"), NULL, dwType, (BYTE*)szValue, sizeof(TCHAR) * (_tcslen(szValue) + 1));
CLOSE: 
RegCloseKey(hKey);
```

结果，调了大半天，所有执行都是成功的，但是结果就是愣是没改到！

一筹莫展之际，突发奇想搜索一下注册表，结果居然搜索到了我写进去的值！在`HKEY_LOCAL_MACHINE\SOFTWARE\Wow6432Node\Microsoft\Windows NT\CurrentVersion\Winlogon`里的`Userinit`。看着这个`Wow6432Node`，猛地一拍大腿，对吼！太久没写 C++ 了，居然忘记了 64 位的坑。

问题发生的原因清楚了，怎么解决呢？

首先，祭出[微软文档](https://docs.microsoft.com/en-us/windows/desktop/winprog64/accessing-an-alternate-registry-view)，第一段：
>By default, a 32-bit application running on WOW64 accesses the 32-bit registry view and a 64-bit application accesses the 64-bit registry view. The following flags enable 32-bit applications to access redirected keys in the 64-bit registry view and 64-bit applications to access redirected keys in the 32-bit registry view. These flags have no effect on shared registry keys. For more information, see [Registry Keys Affected by WOW64](https://docs.microsoft.com/en-us/windows/desktop/winprog64/shared-registry-keys).

微软对于运行在 64 位操作系统的 32 位应用程序，其实际上是在一个叫`WoW64`的子系统上运行的。那么在注册表里，则会分出一个叫`Wow6432Node`的节点，作为 32 位应用程序的重定向。那么默认情况下 32 位的应用程序，是会访问到`Wow6432Node`下的注册表的。当然不是所有注册表项都是如此，具体哪些受到影响，可以看上面的链接。

为此，注册表访问权限掩码多了两种 Flag：

|Flag name|Value|Description|
|--|--|--|
|KEY_WOW64_64KEY|0x0100|Access a 64-bit key from either a 32-bit or 64-bit application.|
|KEY_WOW64_32KEY|0x0200|Access a 32-bit key from either a 32-bit or 64-bit application.Windows 10 on ARM: This refers to the 32-bit ARM registry view for 32-bit ARM processes and the 32-bit x86 registry view for 32-bit x86 and 64-bit ARM64 processes.|

通过这个就可以用来控制我们的应用程序访问的是 64 位的注册表，还是 32 位的注册表重定向。那么，上面的代码修改如下：

```cpp
const TCHAR* szSubKey= _T("SOFTWARE\\Microsoft\\Windows NT\\CurrentVersion\\Winlogon");
HKEY hKey;
TCHAR szValue[100] = _T("");
TCHAR szAppPath[100] = _T("D:\\App.exe");
DWORD dwType = REG_SZ;
DWORD dwSize = sizeof(szValue);
LSTATUS lstc = RegOpenKeyEx(HKEY_LOCAL_MACHINE, szSubKey, 0, KEY_READ | KEY_WRITE | KEY_WOW64_64KEY, &hKey); // 新增 Flag： KEY_WOW64_64KEY
if(ERROR_SUCCESS != lstc) return 0;
lstc = RegQueryValueEx(hKey, _T("Userinit"), NULL, &dwType, (BYTE*)szValue, &dwSize);
if(ERROR_SUCCESS != lstc) goto CLOSE;
_tcscat(szValue, szAppPath);
lstc = RegSetValueEx(hKey, _T("Userinit"), NULL, dwType, (BYTE*)szValue, sizeof(TCHAR) * (_tcslen(szValue) + 1));
CLOSE: 
RegCloseKey(hKey);
```

另外，有两点值得注意：
1. 这两个 Flag 对于 32 位系统自动无效，所以不需要特地去判断用户系统。
2. 不要直接使用`Wow6432Node`和`WowAA32Node`的 Key，一定要使用上面的 Flag 进行控制，这是官方文档的要求（有人就曾经[踩坑](https://www.cnblogs.com/walfud/articles/2311065.html)）：
>Note  
>The Wow6432Node and WowAA32Node keys are reserved. For compatibility, applications should not use these keys directly.
