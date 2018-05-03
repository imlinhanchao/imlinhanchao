---
author: hancel.lin
date: 2018-05-02
title: 使用 node-ffi 构建 Electron 和 C++ Library 混合桌面应用
tags:
    - electron
    - Node.js
category: tech
status: publish
layout: post
guid: urn:uuid:3F82F5AE-652E-4CFE-996B-F3E7B90646E2
---
使用 node-ffi 可以让 Node.js 调用 C++ 的 Library 。在 Windows 下是 `dll` ，在 Mac OS 下是 `dylib` ，Linux 则是 `so` 。node-ffi 加载 Library 是有限制的，只能处理 C 风格的 Library 。也就是函数要被放在 `extern "C"` 里。

安装 node-ffi 对于不同操作系统，会有不同的环境要求。具体可以参看：https://github.com/nodejs/node-gyp#installation
<!--more-->
## for electron 编译
而对于 electron ，需要对 node-ffi 重新编译。我们安装 electron-rebuild 和 electron-prebuilt 进行编译。
```shell
npm i --save-dev electron-rebuild
npm i -g electron-prebuilt
```
设定环境变量：
```bash
# Electron 的版本
export npm_config_target=1.8.4
# 要构建的 electron 类型.
export npm_config_arch=x64
export npm_config_target_arch=x64
# Electron 下载地址头，可以用镜像.
export npm_config_disturl=https://atom.io/download/electron
# 告诉 node-pre-gyp 是为 Electron 编译.
export npm_config_runtime=electron
# 告诉 node-pre-gyp 从源码编译.
export npm_config_build_from_source=true
# 下载的缓存路径.
HOME=~/.electron-gyp npm install
```
然后 rebuild：
```shell
./node_modules/.bin/electron-rebuild -e /usr/local/lib/node_modules/electron-prebuilt
```
`-e` 是本地 electron-prebuilt 的绝对路径。

## 载入 Library
假设有 Library 函数如下：
```cpp
int add(int n1, int n2);
int div(int d1, int d2, int* r);
```
那么 node-ffi 应该这样调用：
```javascript
var ref = require("ref");
var ffi = require("ffi");

var intPtr = ref.refType(ref.types.int); // 创建一个 int 指针类型

var lib = ffi.Library('mylib', {
  "add": [ 'int', [ 'int', 'int' ] ],
  "div": [ 'int', [ 'int', 'int', intPtr ] ]
});

let sum = lib.add(1, 2);
console.log(`1 + 2 = ${sum}`);
let remainder = ref.alloc(ref.types.int, 0);
let quotient = lib.div(10, 3, remainder);
console.log(`10 ÷ 3 = ${quotient} ...... ${remainder.deref()}`);

```
关于 `ffi` 的其他类型，可以参考：https://github.com/ffi/ffi/wiki/Types

# 数组参数的调用
对于参数有包含数组的函数，如：
```cpp
int analysis(int number, int factor[]);
```
我们则需要使用到 `ref-array` 来创建一个数组类型加载函数：
```javascript
var ffi = require("ffi");
var ArrayType = require('ref-array');

var IntArray = ArrayType(ref.types.int);
var lib = ffi.Library('mylib', {
  "analysis": [ 'int', [ 'int', IntArray ] ]
});

let data = [];
data.length = 100;

var factors = new IntArray(data);
lib.analysis(32, factors);

console.log(`The factors of 32 are ${factors.join(',')}`);

```

## 回调函数的使用
有些 C++ Library 会包含有回调函数作为参数的调用。比如：
```cpp
typedef void (*ioCallback) (int uVID, int uPID);
int device_listen(ioCallback MatchingCallback, ioCallback RemovalCallback);
```

node-ffi 对此有专门的用于生成回调函数参数的方法 `Callback`，示例：
```javascript
var ffi = require("ffi");
var lib = ffi.Library('mylib', {
  'device_listen': ['int', ['pointer', 'pointer']],
});

let matchCallback = ffi.Callback('void', ['int', 'int'], (vid, pid) => {
    console.log('match device, VID: ', vid, ', PID: ', pid)
});

let removeCallback = ffi.Callback('void', ['int', 'int'], (vid, pid) => {
    console.log('remove device, VID: ', vid, ', PID: ', pid)
});

lib.device_listen(matchCallback, removeCallback);
```
这个地方有个坑，如果你回调函数是用于持续监听，在程序运行过程中随时可能被调用的话（比如监听设备插入拔出），可能会在程序启动一段时间后，执行回调时引起程序崩溃退出。

这是因为一段时间后，回调函数被垃圾回收了。这里可以在程序最后添加：
```javascript
process.on('exit', function() {
    matchCallback;
    removeCallback;
});
```
这样在程序退出前都会保持引用，就不会被垃圾回收了。


参考资料：
https://github.com/electron/electron/blob/master/docs/tutorial/using-native-node-modules.md
https://github.com/node-ffi/node-ffi/wiki/Node-FFI-Tutorial
https://gist.github.com/ryosuzuki/186958bf1abb0492f626
https://github.com/node-ffi/node-ffi/issues/84



