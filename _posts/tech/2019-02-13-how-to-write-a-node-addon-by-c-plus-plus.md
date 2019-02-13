---
author: hancel.lin
date: 2019-02-13
title: 如何写一个 Node C++ 扩展
tags:
    - node
    - C++
    - node-gyp
category: tech
status: publish
layout: post
guid: urn:uuid:4afb4d9c-9808-488e-bb08-044b612db0e0
---

想要开发一个 Node 的 C++ 扩展，首推死月大神的[《Node.js：来一打 C++ 扩展》](https://book.douban.com/subject/30247892/)。钻研了一个多月，记录一些心得如下：
<!--more-->
## 开发环境

首先自然的 [Node](http://nodejs.org)，推荐安装 [nvm](https://github.com/creationix/nvm)，这样就可以随时切换不同的 Node 版本。另外，为了编译后的扩展在 32 位系统也能使用，最好是安装 32 bit 的 Node。nvm 的指令可以指定，如：`nvm install 10.15.1 32`。

然后就是安装 [node-gyp](https://github.com/nodejs/node-gyp/)，根据其[安装教程](https://github.com/nodejs/node-gyp/#on-windows)，Windows 下除了安装 node-gyp ：`npm i -g node-gyp`，还需要在**管理员**权限下安装 windows-build-tools ：`npm install --global --production windows-build-tools`。

安装完成之后，应该启动时（在 CMD 执行 node-gyp）可能会报错误：
> if not defined npm_config_node_gyp (node "......\node_modules\node-gyp\bin\node-gyp.js"  )  else (node ""  )
> internal/modules/cjs/loader.js:583
>    throw err;
>    ^
>
> Error: Cannot find module '......\npm\node_modules\node_modules\node-gyp\bin\node-gyp.js'

这个错误只要设置一下 `npm_config_node_gyp` 的环境变量就可以了。值就是`node-gyp.js`的路径，通常在 Node 安装目录下的 `\node_modules\npm\node_modules\node-gyp\bin\node-gyp.js`。到 Windows 的环境变量新增一下就行。

## 开发流程

待续...
