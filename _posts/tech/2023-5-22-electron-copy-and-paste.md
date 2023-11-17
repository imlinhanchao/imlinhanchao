---
author: hancel.lin
date: 2023-05-22
title: Electron 复制和粘贴图片实现详解
tags: 
    - electron
    - 剪贴板
    - 复制
    - 粘贴
category: tech
layout: post
guid: urn:uuid:ac655960-2185-4829-8506-9e8db4ab30dc
---

最近，[摸鱼派客户端](https://github.com/imlinhanchao/fishpi-desktop)增强了图片方面的功能，支持复制客户端上消息的图片和粘贴图片到消息框，之前消息框仅能支援图片文件的粘贴，而现在网页的图片和富文本的图片也都可以了。接下来将详细介绍这两个功能的具体实现：

## 粘贴图片

粘贴图片是几乎纯前端的功能，比如摸鱼派的编辑器就有实现，基本思路是监听 `Paste` 事件，然后对 `clipboardData` 解析处理。[<sup>[1]</sup>](https://developer.mozilla.org/zh-CN/docs/Web/API/Element/paste_event) 通常，会根据每个 clipboardData 的 item 的 type 处理，type 的值为 MIME 类型。那么，在这里我们需要处理两种情况，一种是 `image/xxxx`，另一种是 `text/html`。

<!--more-->

```javascript
document.addEventListener('paste', async (ev) => {
    // 获取 Clipboard Data 的项
    let fileList = [];
    let items = ev.clipboardData && ev.clipboardData.items;
    for (var i = 0; items && i < items.length; i++) {
        const item = items[i];
        // 图片文件
        if (item.type.startsWith('image')) {
            fileList.push(items[i].getAsFile());
        }
        // 富文本复制会变成 `text/html`
        if (items[i].type == 'text/html') {
            // 下面解析
            let files = await htmlGetImg(items[i])
            files = files || []
            // 下面解析
            files = files.map(f => f.startsWith('file:') ? constructFileFromLocalFileData(new LocalFileData(f.replace(/file:\/\/\//g, ''))) : f)
            fileList = fileList.concat(files);
        }
    }
    if (file.length == 0) return;
    // 将图片文件上传并插入编辑框
    await this.uploadImg({ target: { files: file } });
});
```

如果复制的是图片文件，就比较简单。直接将图片剪贴板 item 转为图片 `items[i].getAsFile()` 然后上传即可。

而复杂的在于富文本的处理，因为他是一串 HTML 代码，所以就需要解析处理。这里我们编写了一个  `htmlGetImg` 的函数来处理：

```javascript
htmlGetImg(item) {
     return new Promise((resolve) => {
          // 将剪贴板数据转为文本
          item.getAsString((html) => {
                // 使用正则解析出图片地址并返回
                resolve(html.match(/(?<=src=")([^"]+)/g))
          })
     });
 }
```

正常解析出图片地址后，只要转为 Markdown 格式，就可以插入到消息框中。但是，这里有一个坑，比如你在 QQ 里既包含文本，也包含图片的消息中，拉动选中图片后复制，再粘贴到摸鱼派的编辑框中，就会出现这种样子：` ![](file:///C:\Users\XXXXXXX\Tencent Files\XXXXX\Image\Group2\XR\{2\XR{2({Q@PASEGXI06VVTJSV.png)`

这是因为 QQ 消息使用 HTML 富文本渲染，而图片是引用的本地图片，因此造成解析出来的地址是本地图片的磁盘路径。对于网页，这个问题自然是无法解决。但是 Electron 可以访问文件系统，所以可以再进一步处理。这里使用了一个 `get-file-object-from-local-path` 的 node 库[<sup>[2]</sup>](https://www.npmjs.com/package/get-file-object-from-local-path)，可以将本地文件转为 File 对象：

```
files = files.map(f => 
        f.startsWith('file:') ? 
           constructFileFromLocalFileData(
               new LocalFileData(f.replace(/file:\/\/\//g, ''))
           ) : f
)
file = file.concat(files);
```

这里检查了路径的协议，如果是 `file://` 协议，则将 `file://` 移除，剩下的就是文件的磁盘路径了，然后使用库的 `LocalFileData` 生生成一个 `FileData` 再使用 `constructFileFromLocalFileData` 方法转为 Web API 的 `File` 对象即可。这样，如果是本地的图片，就会被上传再取得网页地址插入编辑框中。

## 复制图片

如果是网页，据我目前了解，无法将图片复制到剪贴板中，只有浏览器原生功能可以做到。而 Electron，则提供了一套 Clipboard API，可以精细操作剪贴板。[<sup>[3]</sup>](https://www.electronjs.org/zh/docs/latest/api/clipboard)

要做到复制图片，首先要拿到图片的地址或对象，因此，我将功能加入到图片的右键菜单中。后面有机会再单独讲一下 Electron 右键菜单的实现，这里暂且略过。总之，我们通过右键菜单的操作拿到了要复制的图片的 Image 标签对象。接着就可以来实现复制图片的功能。

```javascript
async copyImg(img) {
    // 获取图片的 buffer
    const imgBuffer = await urlToBuffer(img.src);
    // 获取图片的原始宽高
    const { naturalWidth: width, naturalHeight: height } = img;
    // 生成一串 img 标签 HTML 代码
    const htmlContent = `<img src="${img.src}" width="${width}" height="${height}" />`;
    // 写入剪切板
    clipboard.write({
        // 写入富文本
        html: htmlContent,
        // 写入图片文件
        image: nativeImage.createFromBuffer(imgBuffer, {
            width,
            height,
        }),
    });
}
```

这里分为三个步骤：

1. 获取图片 buffer；
2. 组建 img 标签 HTML
3. 写入剪贴板；

写入剪贴板包含了 image 和  html 两种格式，为什么写入图片要同时写入两种格式呢？这是经过实验得出的做法：如果使用 html 格式写入，则在画图，Photoshop 之类的图片处理工具是无法粘贴的，而如果只是使用 image 的格式，则会出现 Gif 图片无法写入粘贴，我们在网页上使用复制 GIF 图片其实也可以发现，粘贴后图片是不会动的。因此，为了解决这个问题，就分为了 html 和 image 两种格式同时写入。这样无论是静态图还是动态图，都可以正常复制，并且可以粘贴到其他软件上使用。

不过对于动图的复制，还是有些悬而未解的问题，复制的 GIF 只能用于粘贴在一些富文本的编辑软件，QQ 微信之类仍然无法粘贴。还在持续研究，待解决后再更新了~

> <sup>[1]</sup> [Web API 接口参考 Element: paste 事件](https://developer.mozilla.org/zh-CN/docs/Web/API/Element/paste_event)
> <sup>[2]</sup> [Node 库 get-file-object-from-local-path](https://www.npmjs.com/package/get-file-object-from-local-path)
> <sup>[3]</sup> [Electron Clipboard API](https://www.electronjs.org/zh/docs/latest/api/clipboard)
