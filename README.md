这是使用Hexo搭建的私人博客，请严格按照以下顺序进行操作

# Hexo搭建步骤
hexo init XXX
cd XXX
npm install
hexo s

# 导入tranquilpeak主题
 - 下载tranquilpeak的主题包（不要用git的方式clone，否则里面的node_modules无法安装）
 - 将主题包解压到themes文件夹下面，重命名为tranquilpeak
 - 如果主题里面包含了node_modules文件夹，则**不需要**执行npm install
 - 修改根目录下的_config文件，将theme变量设置为tranquilpeak

**上面操作完成之后，先不要运行hexo s，还有一些必要的配置完成之后才可以**

# 配置项

## 根目录的_config文件配置

 - subtitle、description、author信息
 - language如果使用中文的话，要用themes\tranquilpeak\languages目录中的语言文件名称，比如中文的话，要用zh-cn，**注意大小写**
 - post_asset_folder-->true，在推送文章时，才会将资源文件推送服务器
 - relative_link-->false

### 启用RSS订阅功能

 - npm install hexo-generator-feed --save
 - 配置文件添加如下配置
```
feed:
    type: atom
    path: atom.xml
    limit: 20
```
 
## tranquilpeak目录的_config文件配置

 - 侧边栏sidebar的定制，可以删除、调整某个链接
 - Header配置，Header是显示文章时右上角的小图标，可以制定其图标或者作用
 - Author配置，可以配置作者的邮箱信息
 - Author的picture是头像，可以使用外链，或者把文件放在主题themes\tranquilpeak\source\assets\images文件夹内，使用时直接使用图片名称即可，如Photo.jpg
 - Author的工作、个人简介信息要去themes\tranquilpeak\languages下面当前语言的文件里面的author项中配置

# 运行项目

 hexo s 运行之后，打开浏览器的http://127.0.0.1:4000/地址即可，记得要点击地址栏左侧的提示，禁用该地址的cookie，否则会在更新配置后看不到更新

# 其他配置



# 写文章
