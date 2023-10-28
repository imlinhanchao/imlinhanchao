---
author: hancel.lin
date: 2023-10-28
title: 如何将程序转为 Windows Service
tags: 
    - Windows
    - Service
    - 安装包
category: tech
layout: post
guid: urn:uuid:8ebcea68-19f3-4107-a24f-5841c0b57886
---

有时候我们需要在 Windows 常驻一个应用，同时还需要进行保活。那么我们就可以利用 Windows 的 Service 机制来实现这一点。那么有没有什么方法可以任意程序制作为 Windows Service 呢？我们可以利用开源项目 [winsw](https://github.com/winsw/winsw) 来实现。
<!--more-->
## 准备

在制作 Windows Service 前，我们需要准备一些东西：

1. 要制作为 Service 的程序，需要保证程序不会在启动后立即结束，如果是控制台程序，可以加入 `while(true);` 来实现。
2. winsw 程序，在 [Release](https://github.com/winsw/winsw/releases) 下载最新 Release 版本的程序，根据系统架构下载 `WinSW-x86.exe` 或 `WinSW-x64.exe`。
3. winsw 配置档案，这个待会儿说。

## 安装测试

先来测试一下。将 `WinSW` 程序放在 Service 程序的目录下，修改文件名为 `service.exe` 。再创建一个 `service.xml` 的 WinSW 配置档。内容如下：
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<service>
	<id>Your Service Name</id>
	<name>Your Service Name</name>
	<description>Service Feature Description</description>
	<executable>C:\Your\Program\Path\Program.exe</executable>
	<logpath>C:\Your\Program\Path\</logpath>
	<logmode>roll</logmode>
	<depend></depend>
	<startargument></startargument>
	<stopexecutable>taskkill</stopexecutable>
	<stopargument>/im Program.exe /f</stopargument>
	<stoptimeout>10sec</stoptimeout>
</service>
```
上面的 `Your Service Name` 就是你的 Windows 服务命名，`C:\Your\Program\Path\Program.exe` 就是要作为 Service 启动的程序的完整绝对路径（可以在程序内通过命令行参数来创建配置文件，这样就能获得程序的绝对路径）。如果你有需要加入的参数，可以写在 `<startargument>` 中。`<stopexecutable>` 是停止服务的终止程序的方法，这里使用了 Windows 的 taskkill 来杀死程序。

> 在这里，你也可以给自己的程序添加诸如 `stop` 之类的命令行，可以用来更加安全的中止程序，那么 `stopexecutable` 就只要修改为程序的绝对路径，`stopargument` 修改为 `stop` 即可

然后在当前目录打开 cmd，执行 `service.exe install`。再打开任务管理器，切换到 `服务` 即可看到一个 `Your Service Name` 的 服务，状态为`停止`。右键执行 `启动服务` 即可启动服务。

到这里，你就已经成功创建了一个 Windows Service，但若你使用任务管理器前置杀掉程序进程，会发现对应的 Windows Service 也变为了停止状态，并没有自动重新唤醒。因此，我们其实还需要设置一下服务的自动唤醒，在 cmd 中执行：

```bash
sc failure "Your Service Name" reset=5  actions=restart/3/restart/10/restart/60
```

这样，再中止程序运行，就会被自动唤醒了。

如果不希望服务开机启动，可以执行 `sc config "Your Service Name" start= demand` 

更多 Windows Service 的修改指令，可以直接参考 `sc` 指令的文档：https://learn.microsoft.com/zh-cn/windows-server/administration/windows-commands/sc-config

## 卸载服务

如果你是通过安装程序将服务安装到系统，则需要在卸载时将服务删除，执行`sc delete "Your Service Name"` 即可删除。这取决于你是用的哪个程序做安装包的打包。如果是 inno setup，则需要在 `UninstallRun` 添加。

> 得空再写个 Sample 程序示例一下。
