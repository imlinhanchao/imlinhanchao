---
author: hancel.lin
date: 2024-1-11
title: 捉鬼记（二）—— 神秘的部分接口 404
tags: 
    - NGINX
    - 接口
    - 反向代理
category: tech
layout: post
guid: urn:uuid:f9d90d1b-8e3d-44b6-b8c0-d246f94e4e28
---

> 前言：这是捉鬼记系列的第二篇，如果你还没看过第一篇，可以看看：[捉鬼记（一）—— ERR_INSUFFICIENT_RESOURCES](/2023/12/25/ERR_INSUFFICIENT_RESOURCES.html)

又是某日周五，本来已经收拾好了东西准备下班，路过同事的时候，瞥了一眼，就被拦下了。

同事遇到一个问题，他的系统有很多接口，但是其中一个接口，访问会返回 404。接口是通过 NGINX 反向代理的，但是代理的后端原始接口，使用 APIFox 请求却完全正常。

<!--more-->

他 404 的接口是 `/api/A/B/D`，而代理的配置是到 `/api/A`，但是 `/api/A/C` 本身是可以访问的，只有 `/api/A/B/D` 会返回 404。下面是他接口代理的配置：

```nginx 
location /api/A/ {
  rewrite /api/A/(.*) /$1 break;
  proxy_pass http://192.168.0.1:12345;
  proxy_redirect off;
  proxy_set_header X-Real-IP $remote_addr;
  proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  add_header 'Access-Control-Allow-Origin' '*';
  add_header 'Access-Control-Allow-Credentials' 'true';
  add_header 'Access-Control-Allow-Methods' 'GET,POST,OPTIONS';
}
```

看了配置，感觉没有任何问题，很常规的写法。然后我向上看，发现了一段：

```
server {
  listen 9000;
  server_name localhost;
```

这是才知道，他这个系统并没有部署在我们常规的前端服务器，而是另外一个，然后前端服务器再代理这个服务器。于是我又去找到了这段代理的配置。

```nginx
location /system_name/ {
  rewrite /system_name/(.*) /$1 break;
  proxy_pass http://192.168.0.100:9000;
}
```

额，同样没啥毛病。于是我就在前端服务器上，直接请求了一下 `/system_name/api/A/B`，结果竟然是正常的！这时候我就有点懵了，为什么我直接请求是正常的，但是通过代理就不行呢？

也就是问题是出在上面这段配置？但是这段配置也没啥问题啊，而且其他接口都是正常的，只有这个接口不行。

**一切的配置都是正常的，但却导致一个不正常的结果，这必然是有一个尚未被发现的异常配置！**

于是我在前端服务器上的 NGINX 配置下执行了一句命令：

```bash
grep -R system_name .
```

这段命令会在当前目录下，递归搜索 `system_name`，并打印出所有包含这个字符串的文件名和内容。结果，我发现了：

```
./system_name.conf:  location /system_name/ {
./other_system_name.conf:    location /system_name/api/A/B {
```

啊？竟然找到了两个系统的代理配置。打开 `other_system_name.conf`，发现 `/system_name/api/A/B` 竟然是代理到了另外一个系统的接口上，这下谜题就解开了。然后我又执行了一下 `ls -l other_system_name.conf`，发现这个文件的创建者是另一个同事，罪魁祸首抓住了！误我下班！

然后我让同事去找他，让他去确认一下修改的缘由。然后我就下班迎接我的周末去了。