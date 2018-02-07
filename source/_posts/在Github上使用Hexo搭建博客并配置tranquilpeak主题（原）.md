---
title: 在Github上使用Hexo搭建博客并配置tranquilpeak主题（原）
date: 2018-02-07 14:06:25
tags:
 - Hexo
 - Github
 - tranquilpeak
 - 教程
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

该文章将会引导大家使用Hexo搭建私人博客，并配置tranquilpeak主题，最终效果[如此](https://dushaofeng.github.io/)，请**严格按照**以下顺序进行操作。
<!-- more -->

文章目录
<!-- toc -->

# 1.创建Hexo工程
cmd下以此执行以下命令
```
hexo init XXX
cd XXX
npm install
hexo s
```
然后打开浏览器的`http://127.0.0.1:4000/`地址即可看到默认的Hexo页面。

# 2.导入tranquilpeak主题
 - 下载tranquilpeak的[主题包](https://github.com/LouisBarranqueiro/hexo-theme-tranquilpeak/blob/master/docs/user.md)
    >不要用git的方式clone，否则里面的node_modules无法安装
 - 将主题包解压到Hexo项目的themes文件夹下面，重命名为`tranquilpeak`
 - 主题里面包含了node_modules文件夹，**不需要**执行`npm install`
 - 修改**根目录**下的**_config.yml**文件，将theme变量设置为tranquilpeak

**上面操作完成之后，先不要运行hexo s，还有一些必要的配置完成之后才可以**
# 3.必要配置项

## 3.1.根目录的_config文件配置

 - subtitle、description、author信息
 - language信息，要用themes\tranquilpeak\languages目录中的语言文件名称，比如中文的话，要用zh-cn，**注意大小写**
 - post_asset_folder-->true，在推送文章时，才会将资源文件推送服务器
 - relative_link-->false

 
## 3.2.tranquilpeak目录的_config文件配置

 - 侧边栏sidebar的定制，可以删除、调整某个链接
 - Header配置，Header是显示文章时右上角的小图标，可以制定其图标或者作用
 - Author配置，可以配置作者的邮箱信息
 - Author的picture是头像，可以使用外链，或者把文件放在主题themes\tranquilpeak\source\assets\images文件夹内，使用时直接使用图片名称即可，如Photo.jpg
 - Author的工作、个人简介信息要去themes\tranquilpeak\languages下面当前语言的文件里面的author项中配置

# 4.运行项目

```
hexo s
```
运行之后，刷新浏览器的http://127.0.0.1:4000/地址即可，记得要点击地址栏左侧的提示，禁用该地址的cookie，否则会在更新配置后看不到更新。

# 5.主题config的其他配置

 - sidebar_behavior
    >配置项可以改变侧边栏的动作，包括侧边栏大小、是否自动隐藏等，配置选项为1--6，可以自己尝试配一下

 - clear_reading
    >读文件时是否显示侧边栏

 - cover_image:
    >博客全局的背影图片，图片资源放在主题的\themes\tranquilpeak\source\assets\images目录，配置项直接使用图片文件名称，比如common_bg.jpg

 - author_links:
    >配置侧边栏里面的个人链接(区别于侧边栏的内容菜单，比如首页、分类、归档等)，比如添加知乎、github、微博等链接，包括标题、链接地址、图标等
 
 - 修改侧边栏的所有菜单、链接名称
    >如果要修改侧边栏的显示名称，需要到语言文件中修改相应中文即可

# 6.启用RSS订阅功能

项目根目录执行：
`npm install hexo-generator-feed --save`
在根目录的config中添加如下配置
```
feed:
    type: atom
    path: atom.xml
    limit: 20
```

# 7.启用侧边栏的"分类"菜单
 - hexo new page "all-categories"
 - 修改根目录的source/all-categories/index.md文件，将内容替换如下，包括三个"---"符号哦
```
    title: "all-categories"
    layout: "all-categories"
    comments: false
```

# 8.启用侧边栏的"标签"菜单
 - hexo new page "all-tags"
 - 替换根目录的source/all-tags/index.md中的内容如下：

```
    title: "all-tags"
    layout: "all-tags"
    comments: false
```

# 9.启用侧边栏的"归档"菜单
 - hexo new page "all-archives"
 - 替换根目录的source/all-archives/index.md中的内容如下：
```
    title: "all-archives"
    layout: "all-archives"
    comments: false
```

# 10.写文章

 - 还是把自带的那个hello-world.md文章删掉吧，太丑了
 - hexo new "XXX" 新建一篇文章

>一定看清，这里的new命令后面直接跟上带日期的文章名称，并且没有page参数!

上面的命令将会在工程的`source\_posts`文件夹下面生成一个文件和相同名称的文件夹，其中的文件就是博客内容，文件夹用来放置当前文章所用的一些资源，比如图片等。
编辑文章内容如下：

```
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

```

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

 - `metaAlignment`
    >查看文章时，文章标题的排列，居左还是居中

 - `coverMeta`
    >文章的标题是在文章图片背景的内部，还是底部
    
 - `coverSize`
    >文章背景大小，full为全屏，partial为60%

**thumbnailImage加载不出来**
如果thumbnailImage中设置的本地图片加载不出来，可以设置_config文件的url-->/，即可正常加载

# 11.推送Github
 - 安装推送工具
    >npm install hexo-deployer-git –save

 - 配置根目录的_config文件中的github地址
```
deploy:
  type: git
  repository: git@github.com:dushaofeng/dushaofeng.github.io.git
  branch: master
```

 - 生成静态页面并验证
```
hexo g
hexo s
```

 - 推送到github
    >hexo d

 - **等待15分钟**后刷新github地址即可

**提示**
这里的github推送地址和当前Hexo项目地址是分开的，也就是说，github.io的地址上面是没有hexo源码的，只有生成的静态页面。

