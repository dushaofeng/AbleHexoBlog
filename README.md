这是使用Hexo搭建的私人博客

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

# 写文章
