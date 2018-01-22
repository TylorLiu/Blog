---
title: 倒腾Hexo博客全过程
date: 2018-01-22 08:42:47
tags: Hexo
category: blog

---
长久以来在一些博客网站上零零散散写过几篇博客，但是缺乏交流，也没有什么归属感，一直没有足够的劲头坚持下来。后来，在网上看到各路大牛已经独立建站，了解了大致过程，虽心向往之，却力有未逮。直到偶然了解到Hexo+GitHub可以发布个人博客，才决定付诸实践。最后根据潘柏信同学的[博文](http://baixin.io/2015/08/HEXO%E6%90%AD%E5%BB%BA%E4%B8%AA%E4%BA%BA%E5%8D%9A%E5%AE%A2/)，以及网上的一些资源，完成博客的搭建和使用。

---
经过这两天的摸索，我总结了一个简单的套路，follow me!


## **环境准备**
### Node.js
Node.js[官网](https://nodejs.org/)下载
### Git
用来把本地内容提交到github，我是直接装了[GitHub Desktop](https://desktop.github.com/)
### GitHub
[GitHub](https://github.com/)账号一个
#### 建立Repository
建立与你用户名对应的仓库，仓库名必须为【your_user_name.github.io】，用来发布博客
### Hexo
直接参考[官网](https://hexo.io/)
##### 安装

```
npm install hexo-cli -g
```
#### 生成项目到指定目录blog
```
hexo init blog
cd blog
npm install
```
#### 启动项目
```
hexo server
```
得到:

	INFO  Hexo is running at http://localhost:4000/. Press Ctrl+C to stop.

此时，打开 http://localhost:4000/ 就可以看到Hexo默认的博客了。

#### 发布到GitHub
修改blog目录下 _config.yml中的deploy配置

	vim _config.yml

翻到最下面，改成 (your_user_name替换成你的github用户名)

	deploy:
	  type: git
	  repository: https://github.com/your_user_name/your_user_name.github.io.git
	  branch: master

安装HEXO的git插件

	npm install hexo-deployer-git--save

然后就可以愉快的发布了

	hexo deploy

　去你的页面看看吧，http://your_user_name.github.io/
  以后每次修改后部署，可按以下三步来进行。

	hexo clean
	hexo generate
	hexo deploy
   for short

	hexo clean
	hexo g -d

## 博客建设
### 主题
网上有大量Hexo主题，怎么获取不再细说，但是怎么用？
 主题放到blog/themes下，修改_config.yml中theme

    theme: newtheme

 顺便可以改下

    title:
    subtitle:
    description:
    author:

然后，hexo s 运行看下效果
### 更新维护
为了方便维护，我把blog文件也传到Github上，最后决定用统一用IDEA管理。
- Markdown插件编辑博客
- 然后用Terminal窗口执行Hexo命令发布
- 确认效果后，用git插件提交。

