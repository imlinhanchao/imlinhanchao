---
author: hancel.lin
date: 2021-03-17
title: 如何在 Windows 控制臺程序捕捉關閉消息
tags: 
    - C#
    - C++
    - Windows
    - Console
category: tech
layout: post
guid: urn:uuid:0bbcd67e-5a10-4d14-a2c2-8392302468a8
---

有时候我们会写一些常驻系统的控制台程序。虽然一般可以直接关闭，但有时候我们需要在关闭前释放一些系统资源或通知。那么就需要捕捉其关闭的消息。这个时候，我们就可以用 Windows 的 Console API [`SetConsoleCtrlHandler`](https://docs.microsoft.com/en-us/windows/console/setconsolectrlhandler) 来做。

<!--more-->

我们需要先定义一个 [`HandlerRoutine`](https://docs.microsoft.com/zh-cn/windows/console/handlerroutine) 的回调函数。

```cpp
bool bQuit = false;
BOOL CALLBACK HandleCtrlEvent(DWORD dwCtrlType)
{
    switch (dwCtrlType)
    {
        case CTRL_C_EVENT: // 按下 Ctrl + C
        case CTRL_LOGOFF_EVENT: // 注销系统
        case CTRL_SHUTDOWN_EVENT: // 关闭系统
        case CTRL_CLOSE_EVENT: // 按下关闭按钮
        default:
            // do something to release
            bQuit = true;
            return false;
    }
	return TRUE;
}
```

这个回调函数会传入一个 `DWORD` 的值，表示其接收到的事件。具体定义可以参考 [Microsoft 文档](
https://docs.microsoft.com/zh-cn/windows/console/handlerroutine#parameters)。

这里判断了四个与程序关闭相关的事件，然后做对应是释放操作。这里我们操作一个 bool 变量，控制循环的退出。

```cpp
int main()
{
    SetConsoleCtrlHandler(HandleCtrlEvent, TRUE);
    while(bQuit);
}
```

如果是 C# 程序，则可以通过 `DllImport` 导入，以及定义一个托管类型。
```csharp
[DllImport("Kernel32")]
private static extern bool SetConsoleCtrlHandler(EventHandler HandlerRoutine, bool Add);

private delegate bool EventHandler(uint dwCtrlType);
```

然后就可以使用了：

```csharp
class Program
{
    static EventHandler _handler;
    static bool _quit = false;

    private static bool Handler(uint sig)
    {
        switch (sig)
        {
            case 0:
            case 2:
            case 5:
            case 6:
            default:
                _quit = true;
                return false;
        }
    }

    static void Main(string[] args)
    {
        _handler += new EventHandler(Handler);
        SetConsoleCtrlHandler(_handler, true);
        while (!_quit);
    }
}
```