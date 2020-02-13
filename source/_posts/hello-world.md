---
title: Hello World
date: 2016-11-19 11:14:28
tags: hexo
---
使用 [Hexo](https://hexo.io/) 来搭建自己的个人静态博客:

1. Markdown 写博客
2. Hexo 生成 HTML
3. Hexo 发布 HTML 到 Github Pages

> 既然是静态博客，那么就没有后台了，留言功能可以使用第三方的服务，例如畅言。



## Install Node

Hexo 需要 Node

- Mac 安装 Node，可以使用 Homebrew 安装: `brew install node`
- Windows 安装 Node，进入 https://nodejs.org/en/ 下载安装

> Node 就是 Node.js

### 使用 Node 的淘宝镜像

由于网络的问题，访问 Node 的默认仓库有可能会很慢，很多东西都下载不下来，可以使用淘宝的 Node 的镜像，命令行里执行

```
npm install -g cnpm --registry=https://registry.npm.taobao.org
```

即可，具体可参考 [http://npm.taobao.org](http://npm.taobao.org/)

## Install Github Client

Hexo 和 Github 一起使用就可以搭建一个免费的博客网站

- 如果没有安装 Git，需要安装一下
- 到 [https://desktop.github.com](https://desktop.github.com/) 下载 Github 客户端

## Install Hexo and initialize Pages

```
$ npm install hexo-cli -g
$ hexo init Blog
$ cd Blog
$ npm install
$ hexo server
```

## Quick Start

<!--more-->

### Create a new post

``` bash
$ hexo new "My New Post"
```

More info: [Writing](https://hexo.io/docs/writing.html)

### Run server

``` bash
$ hexo server
```

More info: [Server](https://hexo.io/docs/server.html)

### Generate static files

``` bash
$ hexo generate
```

More info: [Generating](https://hexo.io/docs/generating.html)

### Deploy to remote sites

``` bash
$ hexo deploy
```

More info: [Deployment](https://hexo.io/docs/deployment.html)
