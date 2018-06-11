---
title: electron-vue架构解析-生产环境打包过程分析
date: 2018-06-08 16:46:45
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

我们先从生产环境打包流程来分析。
<!-- more -->

从package.json文件入口来看打包命令和调用的脚本：

```javascript
  "scripts": {
    "build": "node .electron-vue/build.js",
    "build:darwin": "cross-env BUILD_TARGET=darwin node .electron-vue/build.js",
    "build:linux": "cross-env BUILD_TARGET=linux node .electron-vue/build.js",
    "build:mas": "cross-env BUILD_TARGET=mas node .electron-vue/build.js",
    "build:win32": "cross-env BUILD_TARGET=win32 node .electron-vue/build.js",
    "build:clean": "cross-env BUILD_TARGET=clean node .electron-vue/build.js",
    "build:web": "cross-env BUILD_TARGET=web node .electron-vue/build.js",
    ...
  },
```

这说明打包的入口文件为build.js，我们就从这个入口文件来分析打包的过程。

```javascript
//用到的配置文件
const buildConfig = require('./build.config')
const mainConfig = require('./webpack.main.config')
const rendererConfig = require('./webpack.renderer.config')

//除了`clean`和`web`的命令外，其他指令都会进行build()的操作：
if (process.env.BUILD_TARGET === 'clean') clean()
else if (process.env.BUILD_TARGET === 'web') web()
else build()


function build () {
  ...
  //构建app
  bundleApp()

  ...
  //打包主进程
  pack(mainConfig).then(result => {
  }).catch(err => {
  })

  ...
  //打包渲染进程
  pack(rendererConfig).then(result => {
  }).catch(err => {
  })
}
function bundleApp () {
  packager(buildConfig, (err, appPaths) => {
        ...
  })
}

function pack (config) {
  return new Promise((resolve, reject) => {
    webpack(config, (err, stats) => {
        ...
    })
  })
}
```

这个构建步骤中，分别进行了三个操作：
 - 使用buildConfig(也就是build.config.js)--构建app
 - 使用mainConfig(也就是webpack.main.config.js)--打包主进程
 - 使用rendererConfig(也就是webpack.renderer.config.js)--打包渲染进程

主进程和渲染进程使用的配置文件我们在稍后的开发流程中分析，这里主要看构建app用的配置文件，也就是build.config文件：
文件内容很简单，我们直接贴出源码：

```javascript
const path = require('path')
module.exports = {
    //目标架构和平台
    arch: 'x64',
    platform: process.env.BUILD_TARGET || 'all',
    //是否启用asar压缩打包
    asar: true,
    //打包目录
    dir: path.join(__dirname, '../'),
    //应用图标
    icon: path.join(__dirname, '../build/icons/icon'),
    ignore: /(^\/(src|test|\.[a-z]+|README|yarn|static|dist\/web))|\.gitkeep/,
    out: path.join(__dirname, '../build'),
    overwrite: true
}
```

这个文件内容很简单，用于electron-packager打包时读取，主要配置了最终生成exe文件的一些参数，包括：
 - arch
 - platform
    目标架构和平台
 - asar
    是否启用asar压缩打包
 - out
 - dir
    指定生成目录
 - icon
    图标
 - ignore
    忽略那些文件
 - overwrite
    覆盖模式打包

该文件中配置的参数其实都可以通过命令的形式实现：

```javascript
electron-packager ./app <name> --platform=win32 --arch=x64 --overwrite --ignore=dev-settings
```

写在配置文件中可以实现“傻瓜式”的打包目的。
这就是生产环境打包的过程。
下一节我们看开发环境的启动过程。
