---
author: hancel.lin
date: 2016-11-09
title: Docker学习笔记（一）
tags: 
    - linux
    - docker
    - 容器
category: tech
status: publish
layout: post
guid: urn:uuid:0fe39afe-d1c4-4efc-b678-186232e68227
---
使用树莓派作为实验机。

## 安装docker

```bash
curl -sSL get.docker.com | sudo sh
```

## 开启容器

安装成功，check版本：
```bash
sudo docker info
```
check docker服务状态
```bash
sudo service docker status
```
开启一个容器
```bash
sudo docker run -i -t resin/rpi-raspbian /bin/bash
```
检查有什么容器，及其状态
```bash
sudo docker ps -a
```
开启一个容器，并命名为`hancelDocker`，开启后会自动进去**bash**，`exit`指令可退出，但会导致容器停止！
```bash
sudo docker run --name hancelDocker -i -t resin/rpi-raspbian /bin/bash
```
<!--more-->
## 管理容器

启动指定名称容器
```bash
sudo docker start hancelDocker
```
进入指定名称容器，使用`exit`指令退出，**会导致容器停止**！
```bash
sudo docker attach hancelDocker
```
启动一个后台容器，执行括号内shell脚本
```bash
sudo docker run --name Helloworld -d resin/rpi-raspbian /bin/bash -c "while true; do echo hello world; sleep 1; done"
```
删除容器
```bash
sudo docker rm Helloworld
```
查看容器最后几行输出
```bash
sudo docker logs Helloworld
```
持续查看容器输出
```bash
sudo docker logs -f Helloworld
```
持续查看容器输出并显示时间戳
```bash
sudo docker logs -ft Helloworld
```
查看容器内进程
```bash
sudo docker top hancelDocker
```
在执行容器内执行指令，此处为创建文件
```bash
sudo docker exec -d hancelDocker touch /root/first.conf
```
启动容器bash并进入，执行`exit`可退出，同时容器**不会**停止！
```bash
sudo docker exec -t -i hancelDocker /bin/bash
```
停止容器
```bash
sudo docker stop Helloworld
```
查看容器详细信息
```bash
sudo docker inspect hancelDocker
```
查看容器指定栏位详细信息
```bash
sudo docker inspect --format='{{.State.Running}}' hancelDocker
```
```bash
sudo docker inspect --format='{{.Name}} {{.State.Running}}' hancelDocker
```
同时查看多个容器的相同信息
```bash
sudo docker inspect --format='{{.Name}} {{.State.Running}}' hancelDocker Helloworld
```
