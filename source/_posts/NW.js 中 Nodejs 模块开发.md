---
updated: '2017-05-26 08:00:00'
categories: code
excerpt: 'NW.js 是一个基于 Nodejs 和 Chromium 的框架, 可以用 HTML5 和 Javascript 方便的开发桌面应用, 其中 Javascript 中也能使用 Nodejs 的模块, 包括第三方的 C++ 模块.'
date: '2017-05-26 08:00:00'
tags:
  - C++
urlname: nwjs-cxx
title: NW.js 中 Nodejs 模块开发
---

NW.js 是一个基于 Nodejs 和 Chromium 的框架, 可以用 HTML5 和 Javascript 方便的开发桌面应用, 其中 Javascript 中也能使用 Nodejs 的模块, 包括第三方的 C++ 模块.


本文简要记录 NW.js 中 Node 模块开发过程, 其实整个过程和 Nodejs 模块开发类似, 不过需要用 nw-gyp 进行工程的编译.


# 准备工作


## 安装 NodeJS


选择 LTS 版本 [https://nodejs.org/dist/v6.10.2/node-v6.10.2-x64.msi](https://nodejs.org/dist/v6.10.2/node-v6.10.2-x64.msi)


配置 cnpm


```text
install -g cnpm -- registry=https://registry.npm.taobao.org

```


## 下载 nwjs


选择 LTS 版本，可支持 Windows XP [https://dl.nwjs.io/v0.14.7](https://dl.nwjs.io/v0.14.7) 下载 SDK 版 [https://dl.nwjs.io/v0.14.7/nwjs-sdk-v0.14.7-win-ia32.zip](https://dl.nwjs.io/v0.14.7/nwjs-sdk-v0.14.7-win-ia32.zip) SDK 版本与 Release 版本的区别： - SDK 版本有 DevTools 的支持


## 安装编译环境


```text
cnpm install --global --production windows-build-tools

```


这条命令会下载安装 Python 2.7 和 Visual C++ Build Tool 也可手动安装 Python 2.7 和 visualcppbuildtools_full.exe


## 安装 nw-gyp


cpm install -g nw-gyp


# 开发


## 创建工程


先创建项目目录, 比如 hello 创建 `binding.gyp` 文件,此文件定义了 gyp 生成工程的各参数, 内容如下


```text
{
  'targets': [
    {
      'target_name': 'hello',
      'sources': [
        'src/hello.cc'
      ],
      'dependencies': [
      ]
    }
  ]
}

```


然后创建 `src\hello.cc`:


```text
#include <node.h>
#include <v8.h>

void Method(const v8::FunctionCallbackInfo<v8::Value>& args) {
  v8::Isolate* isolate = args.GetIsolate();
  v8::HandleScope scope(isolate);
  args.GetReturnValue().Set(v8::String::NewFromUtf8(isolate, "bar"));
}

void init(v8::Local<v8::Object> exports) {
  NODE_SET_METHOD(exports, "foo", Method);
}

NODE_MODULE(binding, init)

```


以上代码定义了 hello 模块中的 foo 方法, 起返回值为 “bar”


运行以下命令创建工程, 工程文件将创建在 build 文件夹中


```text
nw-gyp configure --target=0.14.7 --arch=ia32

```


其中 target 为 nwjs 的版本, 运行 nw.exe 可以看到 arch 为要编译的目标平台


## 编译


运行以下命令编译工程


```text
nw-gyp build --target=0.14.7 --arch=ia32

```


如果编译成功,则会在 build/release 中生成 hello.node, 将此文件复制到 nwjs 项目的 app\node_modules\ 中, 可以 js 文件中调用:


```text
var hello = require("nello");
hello.foo();

```


为了避免每次编译后都手动复制的麻烦, 可以将复制操作编写在 `binding.gyp` 中


```text
{
  'targets': [
    {
      'target_name': 'hello',
      'sources': [
        'src/hello.cc'
      ],
      'dependencies': [
      ]
    },
    ,
    {
      "target_name": "copy_binary",
      "dependencies": ["hello"],
      "type": "none",
      "copies": [
        {
          "destination": "../nwjs-sdk/app/node_modules/",
          "files": [
            "./build/Release/hello.node"
          ]
        }
      ]
    }
  ]
}

```


这样在每次 build 之后可以自动将最新的 .node 文件复制到 nwjs 项目中


## 回调函数


刚才我们的示例代码中定义的 foo 函数是同步的, 调用后直接返回了执行结果.


实际的应用中很多场合需要在 Node 模块中实现异步操作, javascript 通过回调函数的方式获取操作结果


以下是在 Node 模块中实现异步操作的简单示例:


```text
#include <node.h>
#include <v8.h>
#include <uv.h>

//
struct CRequestHello
{
  Persistent<Function> callback;
};

//
void HelloThread(uv_work_t * req)
{
  CRequestHello * request = (CRequestHello *)(req->data);
  Sleep(1000 * 10);
}

//
void EnrollDone(uv_work_t* req, int status)
{
  Isolate * isolate = Isolate::GetCurrent();
  v8::HandleScope handleScope(isolate);

  const unsigned argc = 1;
  Local<Object> argv_0 = Object::New(isolate);
  Local<Value> argv[argc] = { argv_0 };

  CRequestHello * request_data = (CRequestHello *)(req->data);

  Local<Function>::New(isolate, request_data->callback)->
        Call(isolate->GetCurrentContext()->Global(), argc, argv);
}

//
void MethodHello(const FunctionCallbackInfo<Value>& args)
{
  Isolate * isolate = Isolate::GetCurrent();

  if(!args[0]->IsFunction())
  {
    isolate->ThrowException(
        String::NewFromUtf8(isolate, "argument must be a function")
    );
    return;
  }

  Local<Function> callback = Local<Function>::Cast(args[0]);
  CRequestHello * request = new CRequestHello;
  request->callback = callback;
  uv_work_t * work = new uv_work_t();
  work->data = request;

  uv_queue_work(uv_default_loop(), work, HelloThread, HelloDone);
  args.GetReturnValue().Set(Undefined(isolate));
}

//
void init(Local<Object> exports) {
  NODE_SET_METHOD(exports, "hello", MethodHello);
}

//
NODE_MODULE(binding, init)

```


其核心思路是通过 uv_queue_work 创建一个线程来执行耗时的操作 HelloThread, 当线程执行结束后在主线程中执行同步方法 HelloDone, 这时通过回调函数将操作结果返回到 Javascript.


有时我们还希望在一个耗时操作过程中能通过回调函数多次获取操作的执行过程, 比如文件下载进度, 在 Node 模块中的实现方法是通过 uv_async_init 初始化线程同步函数, 然后在线程中通过 uv_async_send 发送同步消息到主线程.

