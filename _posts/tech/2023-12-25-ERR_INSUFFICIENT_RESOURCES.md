---
author: hancel.lin
date: 2023-12-25
title: 捉鬼记（一）—— ERR_INSUFFICIENT_RESOURCES
tags: 
    - 前端
    - Chrome
    - vue
category: tech
layout: post
guid: urn:uuid:3dc738ec-a097-48ef-b06a-1355b080fffd
---

代码写多了，难免撞到鬼。鬼本是不存在的，只是代码多屎山堆积，幽径曲折，一时之间往往难以窥得全貌，遂以撞鬼。须知代码之下，皆是逻辑。故鬼怪作祟，多是自己或前人挖坑罢了。

此捉鬼记，记录日常撞见鬼故事与最后破除之法，以便鱼友共勉。

<!--more-->

某周五，一个西安的同事突然报告，系统某些页面进不去了！一开始以为是后端又 Bug，所以马上模拟同事身份登录系统，看看是不是他的数据出了问题导致接口报错。

![](/media/files/ERR_INSUFFICIENT_RESOURCES/493cbeb91f569b5241f815c975237d7ed971f279702c90ed08b8f4c8a645ff89.jpg)  


结果，进去之后一切功能竟然完全正常，没有任何问题！

于是请他录屏看看发生什么。拿到视频一看，坏了，好像是前端问题！在他点击了某个页面路由时，顶端loading 条正常跑过，但界面却没有任何变化。看一下后台日志，没有收到任何接口请求。浏览器也是最新版本，基本排除了兼容性问题。用户补充了一个信息，以前也出现过，但后面自己好了。

![](/media/files/ERR_INSUFFICIENT_RESOURCES/05996da79b4cc738925039ae77a979a032a5780a294e8b3179e44bb1455837f3.jpg)  


这种时候，就只能看看用户端有没有什么 js 报错了。但是由于公司信息安全控制严格，根本没办法给用户远程看问题，只好通过开会议共享屏幕的方式来看。结果连上去卡到不行，只看到初始画面，根本看不到啥（伏笔）。

这种时候只好空口指挥用户打开浏览器开发者工具，这里就推荐让用户按  Ctrl + Shift + J 。这样可以省略很多步骤，而不是按 F12，很多用户如果是笔记本，可能他就会疑惑：为什么要我调屏幕亮度？

最后终于拿到报错信息：

```
Failed to load resource:net: :ERR_ INSUFFICIENT RESOURCES
```

很多这样的报错，全是在请求页面内的 Js 分片文件。也就是进入路由后，组件的 js 文件根本请求不到。但是，这个错误又是啥呢？从来都没见过啊。

![](/media/files/ERR_INSUFFICIENT_RESOURCES/5bb3fb8901df0e3f16b5cd871058a6f56a33a970281661de7e512f1ce8772c83.jpg)  


正在我还在努力 Google 查这是个啥错误时，用户突然和我说，好了！系统又正常了！我连忙问他做了啥，他说刚刚开了开发者工具后，加上开会议，电脑卡到动不了。他就重启了，然后就好了……

这时候我也找到关于这个错误的解释：Chrome 无法在短时间内处理大量请求。渲染器请求过多会导致内存耗尽并最终导致浏览器崩溃。

于是我在本地测试了一下，在进入那个页面时，瞬间产生了 600 个请求！简直恐怖，赶紧去看看项目的代码。在项目里面找到当初写的一段代码：

```typescript
const modules = import.meta.globEager('./*/index.vue');
const components = {};
for (const path in modules) {
  const p = path.split('/')[1];
  components[p] = modules[path].default;
}

export default components;
```

这个项目有个动态组件，会根据后端接口返回的内容在一个目录内载入一个与之相关的组件，结果为了把这个目录内的组件都打包载入，写了这段代码。当初写的时候，这个目录内只有 10 几个组件，结果随着项目的发展，现在已经有了几百个组件！于是，就导致了在进入这个页面的瞬间，浏览器发起了 600多个请求获取组件代码与其依赖。对于很多用户其实还好，但是遇上一些还在用破电脑的用户，就瞬间可能导致资源不足，最终请求失败。

在这段代码中，使用了 `import.meta.globEager` ，会把组件同步方式载入。当使用了这个 ts 时，就会将所有组件一次性载入。因此，需要使用 vue 的异步组件的载入方式，这里就要使用到 `defineAsyncComponent`来做载入。最后，代码修改为：

```typescript
import { defineAsyncComponent } from 'vue';

const modules = import.meta.glob('./*/index.vue');
const components = {};
for (const path in modules) {
  const p = path.split('/')[1];
  components[p] = defineAsyncComponent(modules[path]);
}

export default components;
```

问题解决！
