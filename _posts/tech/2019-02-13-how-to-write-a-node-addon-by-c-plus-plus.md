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

一个 Node C++ 扩展，包含三个部分：

1. 实际功能代码；
2. 扩展入口代码（addon.cc）；
3. 编译配置（binding.gyp）；

这里我们以 `ObjectWrap` 的方式开发一个 Windows ini C++ 扩展为例子作为说明。下面为目录结构：
```
├─inc
|   ├─win-ini.h
│   └─win-ini.cpp
├─src
|   ├─ini.h
|   ├─ini.cc
│   └─addon.cc
└─binding.gyp
```
相关代码可以查看 [Github](https://github.com/imlinhanchao/node-addon-windows-ini)

### 1. 编写 binding.gyp

`binding.gyp` 相当于一个 makefile，用于定义项目的依赖项，编译的 C++ 文件等等。详细文档可以查看 [GYP Document](https://gyp.gsrc.io/docs/UserDocumentation.md)。

在 C++ Node 扩展的项目里，必须设置 `target_name`（代表编译后的文件名），`sources`（代表需包含编译的文件）。此外还可以设置 `include_dirs` 头文件的包含文件夹。如果有需要包含动态链接库，还需要设置 `library_dirs`。

那么，对于我们的示例项目，可以如下配置：

```python
{
  "targets": [{
    "target_name": "ini",
    "sources": [ "src/addon.cc", "src/ini.cc", "inc/win-ini.cpp", "inc/common.cpp" ],
    "include_dirs": [
        "inc",
    ]
  }]
}
```