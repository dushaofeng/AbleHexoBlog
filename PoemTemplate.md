**这是文章的模板**

---
title: 2018-02-05-测试文章
subtitle: 子标题
date: 2018-02-06 10:13:19
tags:
    - Android
    - 前端
clearReading: true
thumbnailImage: thumbnail_image.jpg
thumbnailImagePosition: bottom
autoThumbnailImage: true
metaAlignment: center
coverImage: cover_image.jpg
coverCaption: "图片说明"
coverMeta: in
coverSize: partial
comments: false
meta: false
actions: false
---

这里是文章的概览，显示在主页缩略内容上面
<!-- more -->

这里是自动生成的文章目录
<!-- toc -->


>CSDN的发布流程烂到家了，只能自己动手搭建技术博客

# 这是大标题

## 这是二级标题

## 这里有个本地图片
![](title_image.jpg)

## 这里有个网络图片
![](http://ww3.sinaimg.cn/large/006tNc79gw1fb0neee6mlj30dw0aldgf.jpg)

## 文章结束

**配置解释**

 - `tags`
     >定义该文章的标签，定义之后可以在分类里面查看自动建立的索引

 - `thumbnailImage`
     >首页的文章标题旁边图片

 - `thumbnailImagePosition`
     >首页的文章图片位置

 - `coverImage`
     >文章打开时顶部的封面图片

 - `<!-- more -->`
     >这个标志之前的内容将会自动生成首页的概览

 - `<!-- toc -->`
     >这个标志的位置将会自动生成文章目录


**thumbnailImage加载不出来**
如果thumbnailImage中设置的本地图片加载不出来，可以设置_config文件的url-->/，即可正常加载
