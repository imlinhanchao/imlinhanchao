---
author: hancel.lin
date: 2017-01-02
title: 维基百科镜像制作全解析
tags: 
	- 维基百科
	- 反向代理
	- Nginx
category: tech
status: publish
layout: post
guid: urn:uuid:27cfd837-1192-4cc4-8ad4-a16d2e4ed84d
---

维基百科（Wikipedia）是一个自由的网络百科全书，包含了全球各地各个知识领域的各色百科知识。

![维基百科](/media/files/how-to-make-wiki-mirror/0.png)

但是，由于某些不可名的原因，目前中文维基百科被**关键字屏蔽**和**DNS污染**无法正常访问。这里有一个已经制作好的维基百科的镜像：w.hancel.net，通过这个镜像，你同样可以访问到相同的中文维基百科的内容，去了解，认识这个世界。

接下来，我们将来讲讲这个镜像的制作方式，以备某日我们提供的镜像失联你可以自行制作。

## 准备

首先，你需要先准备：

1. 在香港/台湾或国外的VPS；
2. VPS上安装Linux系统；
3. 系统内安装Nginx；
4. Nginx带有*ngx_http_substitutions_filter_module*插件；
<!--more-->
## 原理

镜像实际上是对维基百科的反向代理，通过VPS作为代理，代替用户到真实的维基百科服务器拉取资源然后返回。

![反向代理](/media/files/how-to-make-wiki-mirror/1.png)

对于所有反向代理网站的制作，我们必须解决几个问题：

1. 反向代理域名；
2. 替换的网页内容；

对于中文维基百科，我们需要反向代理（下面简称反代）三个域名：

1. zh.wikipedia.org	（主域名）
2. zh.m.wikipedia.org （手机域名）
3. upload.wikimedia.org （图片资源域名）

而**替换内容**主要是为了将原页面的链接替换为我们服务器的域名，这样才能实现无缝访问。

## 域名反代

首先，我们来试一下基本的反代：

```nginx
server {
	server_name w.domain.com;
	listen 80;
	location / {
		proxy_pass https://zh.wikipedia.org;
	}
}
```

在Nginx的配置档里加入上面的配置，把`domain.com`更换为你自己的服务器域名。然后重启Nginx服务。就完成最基本的维基百科反代了。

`proxy_pass` 是Nginx服务器`ngx_http_proxy_module`模块的参数，表示设置一个被代理的服务器协议与地址。在这里我们要反代的是 _https_ 协议的中文维基百科主域名 _zh.wikipedia.org_

访问`w.domain.com`，就会看到维基百科的中文首页。但是，图片全部都是无法显示的。

## 资源反代

那么，接下来我们就再加入维基百科**图片资源域名**的反代。

```nginx
server {
	server_name up.w.domain.com;
	listen 80;
	location / {
		proxy_pass https://upload.wikimedia.org;
	}
}
```

记得每次修改配置后，都要重启Nginx服务。

这时我们随意选择页面上一张图片，右键**复制图片地址**，比如：
https://upload.wikimedia.org/wikipedia/commons/thumb/6/6a/CelegansGoldsteinLabUNC.jpg/190px-CelegansGoldsteinLabUNC.jpg

将**https**改为**http**，将**upload.wikimedia.org**改为**up.w.domain.com**，然后打开：
http://up.w.domain.com/wikipedia/commons/thumb/6/6a/CelegansGoldsteinLabUNC.jpg/190px-CelegansGoldsteinLabUNC.jpg

就会发现，图片可以显示了。

![图片的反代](/media/files/how-to-make-wiki-mirror/2.png)

## 地址替换

但是这样还是不够的，我们还需要将网页上所有图片地址和链接的域名更换为我们的域名。这就需要使用到Nginx的**ngx_http_substitutions_filter_module**模块，这个模块默认安装是没有的，需要自行编译安装进去。方法请自行Google。

若你已经安装，就把配置修改如下：

```nginx
server {
	server_name w.domain.com;
	listen 80;
	location / {
		proxy_pass https://zh.wikipedia.org;
	}
	
	subs_filter_types text/css text/html text/xml text/javascript application/javascript application/json;
	subs_filter https:// http://;
	subs_filter //zh.wikipedia.org //w.domain.com;
	subs_filter upload.wikimedia.org up.w.domain.com;
}
```

这里新增了两个参数：

1. subs_filter_types：设置需要参与替换的文件mimeType。
2. subs_filter：设置替换文本的规则。这里只是单纯的文本替换，末尾加上`r`参数，还可以支援正则表达式替换。

然而，你打开页面就会发现，图片的地址根本没有被换掉，是什么配置写错了吗？

替换的代码没有任何问题，问题出在**zh.wikipedia.org**本身。

打开Chrome开发者工具，选择Network。重新刷新页面，就可以看到页面加载过程的所有请求。选择第一个请求，在**Response Headers**里可以看到网页的**Content-Encoding**是gzip。也就是服务器请求到的内容是压缩的，所以自然无法直接做文本替换了。

![gzip](/media/files/how-to-make-wiki-mirror/3.png)

那么我们可以在**proxy_pass**后加上一句：

```nginx
proxy_set_header Accept-Encoding '';
```

设置请求头，加上`Accept-Encoding ''`，这样维基百科的服务器就会返回文本的内容而不是gzip的了。访问一下，果然所有的图片都可以访问了~

等等，好像还缺点什么，用手机访问试试？

咦！为什么不行？！仔细看看，域名又跳回去到维基百科原域名了。原来，维基百科还有手机专用的域名： _zh.m.wikipedia.org_ 。

用同样的方式，加上手机域名的反代：

```nginx
server {
	server_name m.w.domain.com;
	listen 80;
	location / {
		proxy_pass https://zh.m.wikipedia.org;
		proxy_set_header Accept-Encoding '';
	}
	
	subs_filter_types text/css text/html text/xml text/javascript application/javascript application/json;
	subs_filter https:// http://;
	subs_filter //zh.m.wikipedia.org //m.w.domain.com;
	subs_filter upload.wikimedia.org up.w.domain.com;
}
```

然后，主域名的反代也要修改一下：
```nginx
server {
	server_name w.domain.com;
	listen 80;
	location / {
		proxy_pass https://zh.wikipedia.org;
		proxy_set_header Accept-Encoding '';
		# 将跳转请求的地址替换为我们的地址
		proxy_redirect https://zh.m.wikipedia.org/ http://m.w.domain.com/;
	}
	
	subs_filter_types text/css text/html text/xml text/javascript application/javascript application/json;
	subs_filter https:// http://;
	subs_filter //zh.wikipedia.org //w.domain.com;
	subs_filter upload.wikimedia.org up.w.domain.com;
}
```

重启服务后，再访问就可以了：

![手机的反代](/media/files/how-to-make-wiki-mirror/4.png)

至此，中文维基百科的反代就完成了。以后就可以畅通无阻地~~开车~~访问人类目前最大知识宝库了。另外，全语言的维基百科镜像已经开源到[Github](https://github.com/imlinhanchao/ngx_proxy_wiki)。

## 预告

但是，这样的车还是没有安全带的。为了安全，我们还要给网站上`https`。怎么让镜像支援`https`呢？且听下回分解。


