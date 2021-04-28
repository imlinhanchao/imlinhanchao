---
author: hancel.lin
date: 2017-12-26
title: 基于 Coding WebHook 自动部署 Node.js 应用
tags: 
    - 部署
    - Coding
    - Node.js
    - WebHook
category: tech
status: publish
layout: post
guid: urn:uuid:01ac9922-1de8-47e9-8bf3-a3070738afca
---

因为平时常常使用 Node.js 写网站，为了方便部署写了这个基于 [Coding](https://coding.net/) WebHook 的自动部署应用：https://imlinhanchao.coding.net/public/deploy_node/deploy_node/git
<!--more-->

## 部署服务端
1. 将此 Node.js 项目部署在你的服务器上，端口号默认为3000，假设访问网址为：`http://domain.com:3000`。
2. 将 script 目录下的 deploy-node 加入到系统命令中。
3. 需安装 `forever` npm 包，所有 Node.js 项目都将通过 `forever` 管理。

## 自动部署设置
1. 在 Coding 对应项目设置中，将服务器的 SSH 公钥添加到部署密钥。
2. 执行
```bash
# deploy-node 项目名(英文) git-ssh地址 分支名 启动服务文件
deploy-node fresh git@git.coding.net:user/project.git master ./bin/www
```
3. 部署完成后，会自动开启 log 监控。此时可以在网站操作监控 log 。确认没有问题后按下 `Ctrl + C` 退出监控。
4. 在项目设置的 WebHook 添加 URL：http://domain.com:3000/webhook，若需 token 在服务端的 config.json 配置；

后续只要将代码提交到 master 分支，就会自动同步部署在服务器了。

## 后续开发计划
1. 实现只需添加 WebHook 地址，即可实现部署，无需到服务器操作。
2. 自动更新 Nginx 配置，分配子域名。
3. 在项目中可动态查看管理的应用状态与 Log。
4. 可动态切换应用分支。
5. 同个应用可部署不同分支，比如测试分支。

本文参与 [Coding 征文计划](https://coding.net/wow/stories/)