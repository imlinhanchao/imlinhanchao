---
title: My Appliction
layout: page
---

<style>
.post ul{
    list-style: none;
    padding: 0;
    flex-wrap: wrap;
    display: flex;
    justify-content: space-around;
}
.post ul>li {
    width: 20%;
    position: relative;
    text-align: center;
    margin: 2%;
    min-width: 130px;
}
.post ul>li img {
    width: 60%;
    margin: 20%;
}
.post ul>li strong {
    font-weight: normal;
    position: absolute;
    width: 200px;
    background: rgba(254, 252, 253, .9);
    border: 1px solid #5d5c5f;
    padding: 1em;
    display: none;
    top: 50%;
    left: 50%;
    z-index: 2;
}
.post ul>li:hover strong {
    display: block;
}
.post ul>li>a {
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
    display: inline-block;
    width: 100%;
    font-size: .8em;
}
.post ul>li br {
    display: none;
}
</style>

- ![code-snippet](/media/files/app/code-snippet.png)  
  [Coding Snippet](https://code-snippet.cn)  
  **Snippet 代码分享网站。类似 Gist，但加入了多文件在线编译运行，Web 代码在线预览的功能。**
- ![google-translate](/media/files/app/google-translate.png)  
  [Google Translate 扩展](https://marketplace.visualstudio.com/items?itemName=hancel.google-translate)  
  **基于 Google 翻译(cn)，无需科学上网，无需 API Key 的翻译扩展。**
- ![serialport](/media/files/app/serialport.png)  
  [Serial Port Helper](https://marketplace.visualstudio.com/items?itemName=hancel.serialport-helper)  
  **一个可以在 VSCode 调试串口通信的扩展，支持 RX/TX。**
- ![English](/media/files/app/eng.png)  
  [自考英语查询学习](https://eng.sxisa.com)  
  **自考英语二单词学习查询网站，支持笔记功能。**
- ![markdown-image](/media/files/app/markdown-image.png)  
  [Markdown Image 扩展](https://marketplace.visualstudio.com/items?itemName=hancel.markdown-image)  
  **一个用于方便的在 Markdown 中插入图片的扩展，支持将图片存放在本地或第三方的图床或对象存储。**
- ![weixin-code](/media/files/app/weixin-code.png)  
  [微信公众号代码高亮扩展](https://chrome.google.com/webstore/detail/kbiedhbfjcadjlajanccenpiicgdbfaf)  
  **用于在微信公众平台文章编辑时插入带高亮格式代码，支持样式更改，行内代码与二次编辑。**
- ![fishpi](media/files/app/fishpi.png)  
  [摸鱼派聊天室](https://marketplace.visualstudio.com/items?itemName=hancel.pwl-chat)  
  **基于摸鱼打工人社区[摸鱼派](https://fishpi.cn/)的开放 API 开发的聊天室扩展，可以在里面边写 Bug 边愉快地吹水摸鱼。**
- ![Resource](/media/files/app/res.png)  
  [资源站](https://res.sxisa.com)  
  **一个蒐集各种软件/影视/电子书籍资源下载的网站。**
- ![Invitation Card Maker](media/files/app/love.png)  
  [婚礼邀请函制作工具](http://marry.git.hancel.org/)  
  **婚礼邀请函制作工具，支持批量制作，设定好变量即可。**
- ![Librejo](media/files/app/librejo.png)  
  [Librejo 图书馆](https://librejo.cn/)  
  **个人图书馆网站，可以管理自己的图书与做笔记。**
- ![mofish](media/files/app/mofish.jpg)  
  [摸鱼大闯关](https://p.hancel.org/)  
  **一个网页解谜网站，共 54 关。**
- ![storageEditor](media/files/app/storageditor.png)  
  [Storage Editor](https://chrome.google.com/webstore/detail/lpmmcjhefcghagdhnpbodfdamfmlicfn)  
  **一个用于编辑浏览器 LocalStorage 的 Chrome 扩展。**
- ![Maze](media/files/app/maze.png)  
  [迷宫 Maze](https://maze.hancel.org/)  
  **一个网页迷宫游戏，你需要凭直觉找到出口。**
- ![Sticky Notes](media/files/app/sticky.png)  
  [Sticky Notes](https://github.com/imlinhanchao/sticky_notes)  
  **一个 Windows 桌面应用，可以用来记录你的工作事项，然后钉在桌面上，随时可以查看修改。**
- ![Vue 国际化开发助手](media/files/app/vue-i18n.png)  
  [Vue 国际化开发助手](https://marketplace.visualstudio.com/items?itemName=hancel.front-i18n)  
  **VSCode 扩展，快速为中文 Vue 项目添加国际化支持。**



<script>
    Array.from(document.querySelectorAll('.post ul>li')).map(e => e.onmouseover = (ev) => {
        let target = ev.target
        if (ev.target.nodeName.toLowerCase() != 'li') target = target.parentNode;
        target.querySelector('strong').style.left = (ev.clientX - target.offsetLeft + 10) + 'px'
        target.querySelector('strong').style.top = (ev.clientY - target.offsetTop + 10) + 'px'
    }, false)
</script>