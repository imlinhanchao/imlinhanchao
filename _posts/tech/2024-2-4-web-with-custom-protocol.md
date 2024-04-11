---
author: hancel.lin
date: 2024-2-4
title: Web 自定义协议实践
tags: 
    - web
    - custom protocol
    - app
category: tech
layout: post
guid: urn:uuid:17d463b5-0d74-4576-8ebc-eacd869847d9
---

自定义协议可用于在网页打开桌面应用程序。自定义协议有别于常见的 http、https、ftp 等协议，它是由开发者自己定义的。
<!-- more -->
## 协议注册

在 Windows 中，要注册一个自定义协议，需要在注册表中添加一些项目：

```reg
Windows Registry Editor Version 5.00

[HKEY_CURRENT_USER\Software\Classes\myprotocol]
@="URL:myprotocol"
"URL Protocol"=""

[HKEY_CURRENT_USER\Software\Classes\myprotocol\shell]

[HKEY_CURRENT_USER\Software\Classes\myprotocol\shell\open]

[HKEY_CURRENT_USER\Software\Classes\myprotocol\shell\open\command]
@="\"你的程序绝对路径\" \"%1\""
```

在上面的代码中，`myprotocol` 是自定义的协议名，你可以自己定义。`你的程序绝对路径` 是你的程序的绝对路径，`%1` 是网页传递过来的参数。比如网页的链接是 `myprotocol://hello`，那么 `你的程序绝对路径` 就会接收到 `myprotocol://hello` 这个参数。

这样，当网页中有一个链接是 `myprotocol://hello` 时，点击这个链接就会打开你的程序，并传递 `myprotocol://hello` 这个参数。

## 协议检测

在网页中，可以使用 `a` 标签来定义一个自定义协议的链接，点击就会打开你的程序并传递参数：

```html
<a href="myprotocol://hello">打开我的程序</a>
```

但是，如果你还没有定义这个协议，点击这个链接是没有任何效果的。那么我们就可以使用 [custom-protocol-check](https://www.npmjs.com/package/custom-protocol-check) 这个库来检测是否已经定义了这个协议。

```ts
import customProtocolCheck from "custom-protocol-check";

customProtocolCheck(
  "myprotocol://params",
  () => {
    if(!window.confirm("XXXXXX is not installed. Do you want to install it?")) return;
    window.open('https://example.com/XXXXXX.exe', '_blank');
  },
  () => {
    console.log("Custom protocol found and opened the file successfully.");
  }, 5000
);
```

这样，当用户点击这个链接时，如果没有定义这个协议，就会弹出一个提示框，询问用户是否要下载你的程序。

至此，我们就完成了一个自定义协议的定义和使用。

那么，如果我们想要更多高级的工程化的功能，启动的应用程序应该如何和 Web 页面进行配合呢？

下面，我们将通过一个 C# WinForm 实例，介绍自定义协议的 Web 与桌面应用程序配合的最佳实践。

## Web 与 App 通信

启动应用程序后，应用程序可以通过 Web 的协议链接，获取到 Web 传递过来的参数，但是这样的通信是单向的，如果我们希望 Web 页面和应用程序之间进行双向通信，就需要使用 WebSocket。

我们可以在应用程序内启动一个 WebSocket 服务，然后在 Web 页面中通过 WebSocket 连接到这个服务，这样就可以实现双向通信。

我们可以使用 Fleck 来实现 Websocket Server，这里是 [示例代码](https://github.com/imlinhanchao/custom-protocol-app/blob/master/ProtocolApp/WebSocketService.cs) 。

```csharp
server = new WebSocketServer("ws://127.0.0.1:" + Port);
server.Start(socket =>
{
    socket.OnOpen = () => Logger.Log.Info("WebSocket Open!");
    socket.OnClose = () => Logger.Log.Info("WebSocket Close!");
    socket.OnError = (err) => Logger.Log.Info("WebSocket Error: " + err.Message);
    socket.OnMessage = (message => {
        object msg;
        try
        {
            msg = new JavaScriptSerializer().DeserializeObject(message);
        }
        catch (Exception)
        {
            msg = message;
        }
        action(new MessageCall()
        {
            type = WebSocketMessageType.Text,
            message = msg,
            client = socket,
        });
    });
    socket.OnBinary = (message => {
        action(new MessageCall()
        {
            type = WebSocketMessageType.Binary,
            binary = message,
            client = socket,
        });
    });
});
```

这里监听了 WebSocket 的字符串消息和二进制消息，然后通过 `action` 回调函数将消息数据向外传递。其中 client 就是连接的客户端，可以通过 client 发送消息。

而 Web 那边，我们可以使用 WebSocket 连接到这个服务，[示例代码](https://github.com/imlinhanchao/custom-protocol-app/blob/master/docs/js/main.js#L27)：

```ts
function connectSocket(onMessage, port) {
  return new Promise((resolve) => {
    const ws = new WebSocket(`ws://localhost:${port}`);
    ws.onopen = () => {
      console.log("Socket is connected.");
      ws.onmessage = (message) => {
        onMessage(message.data);
      };
      ws.onclose = () => {
        console.log("Socket is closed.");
      }
      resolve(ws);
    };
    ws.onclose = () => {
      // retry after 1s
      setTimeout(() => connectSocket(onMessage, port), 1000);
    };
  });
}
```

因为启动程序是需要时间的，我们无法确定合适程序完成启动，所以这里使用了 setTimeout 来重试连接，如果连接失败，就会在 1s 后重试。

这样，就建立了一个 Web 页面和应用程序之间的双向通信信道。

## 通信协议

有了通信信道，我们就可以定义一套通信协议，用于 Web 页面和应用程序之间的通信。比如使用 JSON，定义信息结构如下：

```json
{
  "command": "download",
  "data": {
    "url": "https://file.fishpi.cn/2023/02/blob-e6f2be62.png",
    "local": "C:\\Protocol"
  }
}
```

用 `command` 字段来表示命令，`data` 字段来表示数据。这样，我们就可以在 Web 页面中发送一个下载文件的命令，应用程序就可以接收到这个命令，并执行下载文件到指定的目录。

然后，应用程序那边，就可以通过 WebSocket 来接收这个命令，然后执行下载文件的操作，[示例代码](https://github.com/imlinhanchao/custom-protocol-app/blob/master/ProtocolApp/Form2.cs#L61)：

```csharp
// 启动 WebSocket 服务
private void ServiceStart(int Port)
{
    Logger.Log.Info("Start WebSocket in Port " + Port);
    WebSocketService.Start(call =>
    {
        try
        {
            wsClient = call.client;
            if (call.type == WebSocketMessageType.Text)
            {
                MessageExecute((Dictionary<string, object>)call.message);
            }
        }
        catch (Exception ex)
        {
            Logger.Log.Fatal("Websocket Message Error: " + ex.Message);
            throw ex;
        }
    }, Port.ToString());
}

// 执行命令
private void MessageExecute(Dictionary<string, object> message)
{
    string command = message["command"] as string;
    object data = message.Keys.Contains("data") ? message["data"] : null;
    switch (command)
    {
        case "Ping":
            SendData("Ping", "Pong");
            break;
        case "Download":
            Download((data as Dictionary<string, object>)["url"] as string, (data as Dictionary<string, object>)["local"] as string);
            break;
        case "Close":
            Application.Exit();
            break;
    }
}
```

这样，我们就完成了一个自定义协议的 Web 与桌面应用程序的配合。

## 程序路由

如果我们要更进一步，我们可以在应用程序中定义一套路由系统，用于处理不同的协议地址，比如 `myprotocol://form1` 就打开一个窗口，`myprotocol://form2` 就打开另一个窗口。还可以接收 QueryString 参数，比如 `myprotocol://form2?port=12345`。

那么，我们就可以在程序中定义一个路由解析器，用于解析协议地址，然后执行对应的操作。这里有一个 [示例代码](https://github.com/imlinhanchao/custom-protocol-app/blob/master/ProtocolApp/Route.cs) 。具体的实现下回再说，感兴趣的可以先看看代码。

## 打包发布

最后，我们可以使用 [Inno Setup](https://jrsoftware.org/isinfo.php) 来打包发布我们的应用程序，这是一个免费的 Windows 安装程序制作工具，可以用于打包发布我们的应用程序。

这里是一个 [示例脚本](https://github.com/imlinhanchao/custom-protocol-app/blob/master/setup.iss)

然后，你可以在这里：https://protocol.git.hancel.org/ 查看到 Web 实例。

GitHub 仓库：https://github.com/imlinhanchao/custom-protocol-app
