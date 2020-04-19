---
layout: post
title:  "更新 Ghost 1.5.0"
date:   2017-08-09 02:17:42 +0800
---
几周前我从[鸟哥](https://rebornix.com)那听说 ghost 终于发布了1.0版本，据说新版的编辑器很赞，我在本地跑了一下，短暂的试用得出结论：确实很赞，就决定抽时间把服务器更新，从做出这个决定，到真正开始实施的这短短几周时间里，ghost 已经迅速更新到了1.5 .... 有句古话说的好：

>time is money



之前服务器上 ghost 的部署已经被我 [docker](https://huntersnk.com/20170122/) 化，原本每次更新只要简单的```docker pull ghost:latest```

但是1.0的正式版相比之前的0.x，变化还蛮多的，工作目录、配置文件、theme 里一些方法的调用等等都改了，所以我还是花了几个小时重新写和测试 Dockerfile,docker-compose,config.production.json 等等，准备好文件后真正实施部署也就5分钟，另外我还用 [qn-store](https://github.com/minwe/qn-store) 七牛云存储来替换了 ghost 本地的图片存储，配合新版的编辑器，在网页端写 blog 和发图片彻底无痛无缝，简直完美。



![01](https://dn-huntersnk.qbox.me/2017/08/09/01.jpg)

上传图片



![02](https://dn-huntersnk.qbox.me/2017/08/09/02.jpg)



![03](https://dn-huntersnk.qbox.me/2017/08/09/03.jpg)

直接生成七牛的链接



上传后直接传至七牛，并返回七牛的链接，不占用服务器空间。

冲着这一点，今后也要多更新 blog (一个新 flag 冉冉立起）



周末更新 Deploying Ghost With Docker-Compose 2.0<br><br><br>