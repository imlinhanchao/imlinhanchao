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

所有的 `.cc` 和 `.cpp` 文件都需要加入到 `sources` 里。

### 2. 编写转接类
假设你已经有一个 C++ 的 Class，例如 [Ini](https://github.com/imlinhanchao/node-addon-windows-ini/blob/master/inc/win-ini.cpp)。那么我们要将其转为一个 Node 的 C++ 扩展，最简单的方式是做一个类型的转接。将 v8 的类型转为 C++ 的类型。以如下 Class 为例：
```cpp
class Ini
{
public:
	Ini(const TCHAR* pszIniFile/*full path*/);
    
    void SetFile(const TCHAR* pszIniFile/*full path*/);

	bool GetAllKeys(TCHAR* pszAppName, TCHAR* pszKeys[], int& nCount);
	bool GetAllApps(TCHAR* pszAppName[], int& nCount);

	// read ini
	bool Read(TCHAR*  pszAppName, TCHAR*  pszKeyName, TCHAR*   pszValue, int nMaxSize=1024);
	
	// write ini
	bool Write(TCHAR*  pszAppName, TCHAR*  pszKeyName, TCHAR*  pszValue);

	bool Remove(TCHAR* pszAppName, TCHAR* pszKeyName);
	bool Remove(TCHAR* pszAppName);

private:
	TCHAR m_szIni[MAX_PATH];
};
```

那么这里主要有三个地方要注意，1. 构造函数；2. 函数形参；3. 返回值。把这三者搞定就可以了。这里我们基于 `ObjectWrap`来写这个类。
```cpp
class IniNode : public ObjectWrap {
public:
    // 初始化类
    static void Init(Isolate* isolate);
    // 类入口
    static void NewInstance(const FunctionCallbackInfo<Value>& args);

private:
    // 构造函数
    explicit IniNode(Local<String> value);
    ~IniNode();
    
    // 构造函数转化
    static void New(const FunctionCallbackInfo<Value>& args);

    // 类模板
    static Persistent<Function> constructor;

    // ini 路径
    String::Utf8Value m_szIni;

    // 非 Node C++ 扩展 ini 类实例
    Ini m_ini;
}
```

首先我们把构造函数写一下，基本上把变量初始化，然后给到我们自己的 ini 类直接初始化即可：

```cpp
IniNode::IniNode(Local<String> value) 
    : m_szIni(value), m_ini(*m_szIni)
{
}

```

然后，我们要编写初始化函数，用来设定类模板，此部分会影响到 C++ 扩展的对外接口：
```cpp
void IniNode::Init(Isolate* isolate)
{
    // 创建类模板
    Local<FunctionTemplate> tpl = FunctionTemplate::New(isolate, New);
    // 设置类名
    tpl->SetClassName(String::NewFromUtf8(isolate, "IniNode"));
    // 设置类对外暴露的函数或成员计数
    tpl->InstanceTemplate()->SetInternalFieldCount(1);
    // 设置对外函数，名字为 getPath
    NODE_SET_PROTOTYPE_METHOD(tpl, "setPath", setPath);
    // 设置类模板
    constructor.Reset(isolate, tpl->GetFunction());
}
```

`New` 是 Node 实际调用的接口，通过 `New` 我们生成一个类实例。如果构造函数有参数，我们还需要取得参数。返回参数就是实例对象。

```cpp
void IniNode::New(const FunctionCallbackInfo<Value>& args)
{
    // 获取参数
    Local<String> str = args[0]->IsString() ? Local<String>::Cast(args[0]) : String::NewFromUtf8(Isolate::GetCurrent(), "");
    // 实例化对象
    IniNode* obj = new IniNode(str);
    // 将对象存入 this
    obj->Wrap(args.This());
    // 返回 this
    args.GetReturnValue().Set(args.This());
}
```

这样当我们在 Node.js 里面引用扩展的似乎，就可以用一个变量接收实例了：
```javascript
const ini = require('ini')
let ini_obj = ini('your_ini_path')
```

这样基础的结构就搭出来了，然后我们要实现一下 `Init` 里的 `setPath` 接口。我们需要定义两个 `setPath` 的函数。

```cpp
    const char* setPath(const char* szPath);
private:
    static void setPath(const FunctionCallbackInfo<Value>& args);
```

私有的 `setPath` 就是 `Init` 里实际设置的接口，Node 调用函数时，会实际调用这个函数。`FunctionCallbackInfo` 这是包含了函数调用的一些上下文信息，包含入参和返回值。

```cpp
void IniNode::setPath(const FunctionCallbackInfo<Value>& args)
{
    Isolate* isolate = args.GetIsolate();

    // 获取第一个入参
    String::Utf8Value path(Local<String>::Cast(args[0]));

    // 获取对象指针
    IniNode* obj = ObjectWrap::Unwrap<IniNode>(args.Holder());

    // 调用实际的 setPath 并返回执行结果，返回值类型为 String
    args.GetReturnValue().Set(String::NewFromUtf8(isolate, obj->setPath(*path)));
}
```
而实际的 `setPath` 这只要转接一下即可。

```cpp
const char* IniNode::setPath(const char* szPath)
{
    m_ini.SetFile(szPath);
    return szPath;
}
```

其他的函数也是类似的做法。

### 3. 编写入口 addon.cc

编写完成类之后，就可以写入口了。编写一个入口初始化函数：
```cpp
void InitAll(Local<Object> exports, Local<Object> module)
{
    // 初始化类模板
    IniNode::Init(exports->GetIsolate());
    // 导出扩展入口
    NODE_SET_METHOD(module, "exports", IniNode::NewInstance);
}
```

然后用宏定义 `NODE_MODULE` 设置即可：

```cpp
NODE_MODULE(addon1, InitAll)
```

如此，就完成了一个 Node C++ 扩展的编写。

### 4. 编译生成与调用

依次执行两条指令：
```bash
# 生成对应平台的工程项目
node-gyp configure
# 编译生成 node C++ 扩展
node-gyp build
```
编译成功后，就可以在 `build/Release` 里找到一个 `.node` 的后缀名的文件。这就是 Node C++ 扩展，可以用相对路径的方式 `require` 之后使用。例如：

```javascript
const ini = require('ini')
let ini_obj = ini('your_ini_path')
let ini_path = ini_obj.setPath('new_ini_path')
console.log(ini_path) // new_ini_path
```