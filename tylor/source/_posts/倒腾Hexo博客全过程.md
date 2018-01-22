---
title: 倒腾Hexo博客全过程
date: 2018-01-22 08:02:26
tags: Hexo
category: Blog
---
长久以来在一些博客网站上零零散散写过几篇博客，但是缺乏交流，也没有什么归属感，一直没有足够的劲头坚持下来。后来，在网上看到各路大牛已经独立建站，了解了大致过程，虽心向往之，却力有未逮。直到偶然了解到Hexo+GitHub可以发布个人博客，才决定付诸实践。最后根据潘柏信同学的博文，以及网上的一些资源，完成博客的搭建和使用。

大致过程：

环境准备

Node.js

Node.js官网下载

Git

用来把本地内容提交到github，我是直接装了GitHub Desktop

GitHub

GitHub账号一个，用来发布博客

Hexo

直接参考官网

安装

npm install hexo-cli -g
生成项目到指定目录blog

hexo init blog
cd blog
npm install
启动项目

hexo server
