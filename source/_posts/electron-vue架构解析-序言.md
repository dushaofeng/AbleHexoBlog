---
title: electron-vue架构解析-序言
date: 2018-06-08 16:41:45
tags:
 - electron
 - vue
 - electron-vue
 - 前端
clearReading: true
thumbnailImage: post_head.jpg
thumbnailImagePosition: left
autoThumbnailImage: true
metaAlignment: center
coverImage: post_bg.jpg
coverCaption: ""
coverMeta: in
coverSize: partial
comments: false
meta: false
actions: false
---
<!-- toc -->

本系列文章将介绍[electron-vue](https://github.com/SimulatedGREG/electron-vue)前端框架的作用、结构、使用方法。

electron-vue是SimulatedGREG基于[vue-cli](https://github.com/vuejs/vue-cli)搭建的Vue+Webpack+Electron脚手架，可以用来开发跨PC平台的应用，源码地址在[这里](https://github.com/SimulatedGREG/electron-vue)。
其主要功能/特色包括：
 - 主进程和渲染进程配置文件分离
 - 代码热更新
 - 详细的Log输出
 - 除了必备的Electron、Vue、Webpack等插件外，还可以一键配置：Axios\Vue-router\Vuex\Eslint\等插件

由于其自带的[说明文件](https://simulatedgreg.gitbooks.io/electron-vue/content/en/)仅仅说明了该项目的概要、使用方法，并没有整个结构的解释，本系列文章就来从源码角度分析这个脚手架如何管理代码、如何分离编译环境、如何进行热更新等问题。

# 1.下载架构模板

由于electron-vue基于**vue-cli**进行了二次封装，因此在使用之前，需要先安装vue-cli的脚手架：

```javascript
npm install -g vue-cli
```

然后初始化electron-vue的项目：

```javascript
vue init simulatedgreg/electron-vue my-project
```

然后在项目根目录下使用npm或yarn安装依赖即可使用：

```javascript
cd my-project
yarn # or npm install
yarn run dev # or npm run dev
```

# 2.代码结构
我们先来看一下原始版的代码结构：
图图图
从这个结构中可以看到该框架下的文件大致可以分为四个部分：
 - webpack配置文件
 - 生成目录&依赖目录
 - 源码目录
 - 全局配置文件

接下来的几个小节我们分别来介绍他们的作用。
