---
author: hancel.lin
date: 2024-1-11
title: 捉鬼记（三）—— 调试时就正常
tags: 
    - element-plus
    - 焦点
    - tab
category: tech
layout: post
guid: urn:uuid:0211b44d-3394-40bf-a304-3e0825615613
---

> 前言：这是捉鬼记系列的第三篇，如果你还没看过第二篇，可以看看：[捉鬼记（二）—— 神秘的部分接口 404](/tech/2024/1/11/partial-interface-404.html)

这次终于不是周五了，某日早晨，坐我对面的同事突然探出头来，“林工，来帮忙看个问题呗~”。隐隐一些预感，鬼又来了。

他的问题是这样的，他的系统有一个页面，里面有一个 tab，因为某些特殊的需求，需要按下左右键切换前后的 tab。于是他使用了一个 hotkey 的库来做监听，然后在监听到左右键的时候，去修改 tab 的 `activeIndex`，`activeIndex` 是一个 `v-model` 绑定的变量。除了听过左右键切换，也可以直接点击 tab 切换，每次切换时，就会调用一个函数，用来改变 tab 下方动态载入的组件。这两个功能单独来说都是正常的，但是如果先点击 tab 切换，然后再使用左右键切换，就会出现问题，每次会切换两个 tab。

<!--more-->

然后诡异的地方来了，我打了断点在切换执行的函数，想看看是不是执行了两次。结果，问题消失了！我再次点击 tab 切换，然后使用左右键切换，结果也是正常的。我尝试刷新页面，问题也没有出现。但是，如果把断点去掉，问题又会出现！

仿佛这个 bug 像是长了眼睛一样，发现有人在盯着他，就躲起来了。

这样的话，就只好用打印调试法了，在切换执行的函数里面，打印出 `activeIndex`，结果发现，点击切换后再按左右键，console 打印了两次。

那么问题的关键，就是多出来的一次是谁调用的呢？可是现在我如果下断点，问题就不会出现了，那么我就没办法通过查看堆栈知道是谁调用的了。

于是，我就把 `console.log` 改成 `console.trace` 了，这样就可以打印出堆栈了。然后，就得到了这个：

```ts
(anonymous)               	@	index.vue:72
callWithErrorHandling     	@	runtime-core.esm-bundler.js:158
callWithAsyncErrorHandling	@	runtime-core.esm-bundler.js:166
job                       	@	runtime-core.esm-bundler.js:1756
callWithErrorHandling     	@	runtime-core.esm-bundler.js:158
flushJobs                 	@	runtime-core.esm-bundler.js:357
Promise.then(异步)		
queueFlush                	@	runtime-core.esm-bundler.js:270
queueJob                  	@	runtime-core.esm-bundler.js:264
scheduler                   @	runtime-core.esm-bundler.js:1778
triggerEffect             	@	reactivity.esm-bundler.js:373
triggerEffects            	@	reactivity.esm-bundler.js:363
triggerRefValue           	@	reactivity.esm-bundler.js:974
set value                 	@	reactivity.esm-bundler.js:1018
changeCurrentName         	@	tabs.tsx:98
setCurrentName            	@	tabs.tsx:110
await in setCurrentName(异步)		
handleTabClick              @	tabs.tsx:119
callWithErrorHandling     	@	runtime-core.esm-bundler.js:158
callWithAsyncErrorHandling	@	runtime-core.esm-bundler.js:166
emit                        @	runtime-core.esm-bundler.js:664
(anonymous)	                @	runtime-core.esm-bundler.js:7422
onClick	                    @	tab-nav.tsx:331
callWithErrorHandling       @	runtime-core.esm-bundler.js:158
callWithAsyncErrorHandling	@	runtime-core.esm-bundler.js:166
invoker	                    @	runtime-dom.esm-bundler.js:278
changeTab                   @	tab-nav.tsx:234
callWithErrorHandling       @	runtime-core.esm-bundler.js:158
callWithAsyncErrorHandling	@	runtime-core.esm-bundler.js:166
invoker                   	@	runtime-dom.esm-bundler.js:278
```

注意其中 `tab-nav.tsx` 中的几笔，这是 element-plus 的 tab 组件。于是我就点进去看看 ElementPlus 的源码，发现了这么一段：

```typescript
const changeTab = (e: KeyboardEvent) => {
      const code = e.code

      const { up, down, left, right } = EVENT_CODE
      if (![up, down, left, right].includes(code)) return

      // 左右上下键更换tab
      const tabList = Array.from(
        (e.currentTarget as HTMLDivElement).querySelectorAll<HTMLDivElement>(
          '[role=tab]:not(.is-disabled)'
        )
      )
      const currentIndex = tabList.indexOf(e.target as HTMLDivElement)

      let nextIndex: number
      if (code === left || code === up) {
        // left
        if (currentIndex === 0) {
          // first
          nextIndex = tabList.length - 1
        } else {
          nextIndex = currentIndex - 1
        }
      } else {
        // right
        if (currentIndex < tabList.length - 1) {
          // not last
          nextIndex = currentIndex + 1
        } else {
          nextIndex = 0
        }
      }
      tabList[nextIndex].focus({ preventScroll: true }) // 改变焦点元素
      tabList[nextIndex].click() // 选中下一个tab
      setFocus()
    }
```

这段代码的作用，就是监听 tab 的左右上下键，然后切换 tab。这函数就绑定在 `tab-nav` 的 `onKeydown` 上。

这样就真相大白了，因为点击时，会把焦点移动到 `tab-nav` 上，因此，导致他可以监听到 `keydown` 由此触发切换。而如果我打了断点，就会导致焦点不会移动到 `tab-nav` 上，因此，就不会触发 `keydown`，也就不会切换了。

得出这个猜测，我做了一个实验，点击 tab 切换后，鼠标在空白处再点击一下，再按左右键，果然就不会触发了。

于是，我再查了一下 ElementPlus 的文档，发现他并没有属性可以控制是否监听。因此，只能用一点点魔法来解决了。我们在切换的处理函数中，添加了这么一句：

```typescript
document.activeElement?.blur()
```

这样就可以把焦点移除了，就不会触发 ElementPlus 组件的 `keydown` 了。

由此，问题解决！