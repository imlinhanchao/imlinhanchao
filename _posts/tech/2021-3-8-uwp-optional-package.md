---
author: hancel.lin
date: 2021-03-08
title: UWP 可选包开发
tags: 
    - UWP
    - 可选包
    - XAML
category: tech
layout: post
guid: urn:uuid:a22f9a73-64a1-4702-b118-ad3d520aea2c
---

最近公司的需求，要把控制设备的软体都开始转移到 UWP 架构上。因为涉及设备众多，开发单位也很多，所以就想实现用户按需加载对应设备代码的方式来减小包大小。研究一通发现，我所需要的正是 UWP 的可选包。

可选包，有点类似游戏的 DLC，是对主应用进行功能扩展的一个应用包。发布应用包时可以和主应用关联，用户安装了可选包，就具备了可选包的内置功能。

<!--more-->

这里是微软官方的[可选包文档](https://docs.microsoft.com/zh-cn/windows/msix/package/optional-packages-with-executable-code#c-optional-packages-with-executable-code)。

但是里面只有关于如何创建一个工程的简述。实际实验遇到了一些坑，

1. 引用可选包 XAML；
   在主包引用可选包的 XAML 页面时，一直会报 `XAML parsing failed.` 的错误。最后在 [Stackoverflow](https://stackoverflow.com/a/63823702/4123782) 上找到解决方案：  
   XAML 界面创建后，需修改构造函数，移除其中的`this.InitializeComponent();`，添加如下代码：
    ```csharp
    Uri resourceLocator = new Uri("ms-appx://<Optional Package Id>/<XAML Name>.xaml");
    Application.LoadComponent(this, resourceLocator, ComponentResourceLocation.Nested);
    ```
    其中的`<Optional Package Id>`填写可选包的 `Package Id`，可以在 `Package.appxmanifest` 的 `Identity` 标签的 `Name` 属性找到。`<XAML Name>` 这是其对应的 XAML 文件名。
2. 无法进行 Debug。这个目前还没找到解决方案。