---
title: electron-vue架构解析-开发环境启动流程分析
date: 2018-06-08 16:47:52
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

这一节我们来看开发环境的启动流程。
该框架主要修改是对开发环境的优化，包括了于开发环境的配置文件隔离，主进程和渲染进程配置文件隔离，编译过程提示等功能，因此这一节内容才是整个框架的核心。
<!-- more -->

我们从开发人员用到的启动命令说起。

从package中我们看到启动命令就是：

```javascript
"dev": "node .electron-vue/dev-runner.js",
```

也就是在终端使用`npm run dev`就可以了，而这个命令对应的脚本就是dev-runner.js。

```javascript
function init() {
    greeting();

    Promise.all([startRenderer(), startMain()])
        .then(() => {
            startElectron();
        })
        .catch(err => {
            console.error(err);
        });
}
```

这是脚本运行时执行的唯一方法，其中的greeting()就是在终端输出一个Log，如图所示：
![](欢迎的Log.png)

然后做了三个操作分别启动了渲染进程、主进程和Electron：
 - startRenderer()
 - startMain()
 - startElectron()

下面我们逐个来看具体的启动流程。

# 1.渲染进程启动过程分析

```javascript
function startRenderer() {
    return new Promise((resolve, reject) => {
        //加载webpack配置文件
        rendererConfig.entry.renderer = [path.join(__dirname, 'dev-client')].concat(rendererConfig.entry.renderer);

        //创建webpack
        const compiler = webpack(rendererConfig);
        //创建webpack-hot-middleware
        hotMiddleware = webpackHotMiddleware(compiler, {
            log: false,
            heartbeat: 2500
        });

        //编译状态监控
        compiler.plugin('compilation', compilation => {
            compilation.plugin('html-webpack-plugin-after-emit', (data, cb) => {
                hotMiddleware.publish({action: 'reload'});
                cb();
            });
        });

        compiler.plugin('done', stats => {
            logStats('Renderer', stats);
        });

        //创建webpack-dev-server
        const server = new WebpackDevServer(
            compiler,
            {
                contentBase: path.join(__dirname, '../'),
                quiet: true,
                before(app, ctx) {
                    app.use(hotMiddleware);
                    ctx.middleware.waitUntilValid(() => {
                        resolve();
                    });
                }
            }
        );

        server.listen(9080);
    });
}
```

在这个方法里，共完成了三个操作：
 - 创建webpack对象
 - 利用webpack对象来创建WebpackDevServer对象
 - 监听webpack编译过程

我们分别来看这三个操作的具体情况。

## 1.1.渲染进程**创建webpack对象**

webpack对象创建时使用的配置文件是我们分析的重点。我们来看webpack的配置文件，也就是rendererConfig变量：

```javascript
rendererConfig.entry.renderer = [path.join(__dirname, 'dev-client')].concat(rendererConfig.entry.renderer);
```

这说明webpack的配置文件来自于两个文件：dev-client模块和rendererConfig.entry.renderer变量。
这里看一下源码就知道，concat方法连接而成的数组中包含了两个元素，一个是"dev-client"，另一个是根目录的"../src/renderer/main.js"文件，也就是说，webpack的entry参数实际上是这种形式：

```javascript
entry: {
    renderer: ['dev-client',path.join(__dirname, '../src/renderer/main.js')]
},
```

再来看一下output参数内容（在webpack.renderer.config.js）：

```javascript
output: {
    filename: '[name].js',
    libraryTarget: 'commonjs2',
    path: path.join(__dirname, '../dist/electron')
},
```

这种用法包含了三个信息：
 - webpack将会把dev-client**模块**和main.js**文件**同时打包进output指定的文件中
 - dev-client是一个模块，根据源码查找，这个模块就是dev-client.js文件
 - entry内容以key--value形式定义，那么output中的name变量就是entry中的key

综合来说，这种用法的作用就是：同时把dev-client.js和main.js文件打包，输出到根目录下的/dist/electron/render.js文件中。
这个信息非常重要，后面要用到。
接下来我们先来解决这两个文件的作用。

### 渲染进程的dev-client配置文件

这个文件内容很简单，直接贴出源码：

```javascript
const hotClient = require('webpack-hot-middleware/client?noInfo=true&reload=true');
//注册webpack-hot-middleware监听器
hotClient.subscribe(event => {
    //这里只处理了Main进程发送的"compiling"的事件，实际上在Render进程中还发送了"reload"的消息
    if (event.action === 'compiling') {
      ...
      <div id="dev-client">
        Compiling Main Process...
      </div>
    `;
    }
});
```

我们直接来说作用，他负责在编译过程中，在界面上显示“Compiling Main Process...”的提示语。
效果如下：
![](render编译log.png)
至于他如何检测编译过程，我们稍后来说。

### 渲染进程的webpack.renderer.config配置文件

接下来我们来看渲染进程的主配置文件的主要内容：

```javascript
//将vue模块列为白名单
let whiteListedModules = ['vue'];
let rendererConfig = {
    //指定sourcemap方式
    devtool: '#cheap-module-eval-source-map',
    entry: {
        renderer: path.join(__dirname, '../src/renderer/main.js')
    },
    externals: [
        //编译白名单
        ...Object.keys(dependencies || {}).filter(d => !whiteListedModules.includes(d))
    ],
    module: {
        rules: [
            {
                test: /\.css$/,
                use: ExtractTextPlugin.extract({
                    fallback: 'style-loader',
                    use: 'css-loader'
                })
            },
            {
                test: /\.html$/,
                use: 'vue-html-loader'
            },
            {
                test: /\.js$/,
                use: 'babel-loader',
                exclude: /node_modules/
            },
            {
                test: /\.node$/,
                use: 'node-loader'
            },
            {
                test: /\.vue$/,
                use: {
                    loader: 'vue-loader',
                    options: {
                        extractCSS: process.env.NODE_ENV === 'production',
                        loaders: {
                            sass: 'vue-style-loader!css-loader!sass-loader?indentedSyntax=1',
                            scss: 'vue-style-loader!css-loader!sass-loader'
                        }
                    }
                }
            },
            {
                test: /\.(png|jpe?g|gif|svg)(\?.*)?$/,
                use: {
                    loader: 'url-loader',
                    query: {
                        limit: 10000,
                        name: 'imgs/[name]--[folder].[ext]'
                    }
                }
            },
            {
                test: /\.(mp4|webm|ogg|mp3|wav|flac|aac)(\?.*)?$/,
                loader: 'url-loader',
                options: {
                    limit: 10000,
                    name: 'media/[name]--[folder].[ext]'
                }
            },
            {
                test: /\.(woff2?|eot|ttf|otf)(\?.*)?$/,
                use: {
                    loader: 'url-loader',
                    query: {
                        limit: 10000,
                        name: 'fonts/[name]--[folder].[ext]'
                    }
                }
            }
        ]
    },
    node: {
        //根据版本信息确定__dirname和__filename的行为
        __dirname: process.env.NODE_ENV !== 'production',
        __filename: process.env.NODE_ENV !== 'production'
    },
    plugins: [
        //css文件分离
        new ExtractTextPlugin('styles.css'),
        //自动生成html首页
        new HtmlWebpackPlugin({
            filename: 'index.html',
            template: path.resolve(__dirname, '../src/index.ejs'),
            minify: {
                collapseWhitespace: true,
                removeAttributeQuotes: true,
                removeComments: true
            },
            nodeModules: process.env.NODE_ENV !== 'production'
                ? path.resolve(__dirname, '../node_modules')
                : false
        }),
        //热更新模块
        new webpack.HotModuleReplacementPlugin(),
        //在编译出现错误时，使用 NoEmitOnErrorsPlugin 来跳过输出阶段
        new webpack.NoEmitOnErrorsPlugin()
    ],
    output: {
        filename: '[name].js',
        libraryTarget: 'commonjs2',
        path: path.join(__dirname, '../dist/electron')
    },
    resolve: {
        alias: {
            //在代码中使用@代表renderer目录
            '@': path.join(__dirname, '../src/renderer'),
            //精确指定vue特指vue.esm.js文件
            'vue$': 'vue/dist/vue.esm.js'
        },
        extensions: ['.js', '.vue', '.json', '.css', '.node']
    },
    //指定编译为 Electron 渲染进程
    target: 'electron-renderer'
};
```

这个配置文件的主要配置项都给出了注释，作用很容易理解，这里不再详细介绍，webpack更多配置可以查看[这里](https://webpack.docschina.org/configuration/)。

## 1.2.渲染进程利用webpack对象来创建WebpackDevServer对象

我们先来区分一下三个概念：
 - webpack
    > webpack本身只是打包工具，不具备热更新功能

 - [webpack-hot-middleware](https://github.com/webpack/webpack-dev-middleware)
    > An express-style development middleware for use with webpack bundles and allows for serving of the files emitted from webpack. This should be used for development only.
    > The middleware installs itself as a webpack plugin, and listens for compiler events.
    他是一个运行于**内存中**的（非常重要的概念，作用稍后会提到）一个文件系统，可以检测文件改动自动编译（到内存中）。

 - [webpack-dev-server](https://github.com/webpack/webpack-dev-server)
    > Use webpack with a development server that provides live reloading. This should be used for development only.  It uses webpack-dev-middleware under the hood, which provides fast in-memory access to the webpack assets.
    他是利用webpack-hot-middleware搭建了Express的服务器，从而实现实时刷新的功能。

了解了这三个概念之后，我们来看具体如何创建WebpackDevServer对象，其实很简单，把我们刚才创建的webpack作为参数来初始化WebpackDevServer即可：

```javascript
//创建webpackHotMiddleware
hotMiddleware = webpackHotMiddleware(compiler, {
    log: false,
    heartbeat: 2500
});

//创建WebpackDevServer
const server = new WebpackDevServer(
    //以webpack对象作为参数
    compiler,
    {
        contentBase: path.join(__dirname, '../'),
        quiet: true,
        before(app, ctx) {
            //使用webpackHotMiddleware
            app.use(hotMiddleware);
            ctx.middleware.waitUntilValid(() => {
                resolve();
            });
        }
    }
);

//服务器运行在9080端口
server.listen(9080);
```

这个创建过程很清晰，需要注意的是，webpackHotMiddleware的heartbeat的参数作用并不是检测文件的频率，而是保持服务器链接存活的心跳频率：
>heartbeat - How often to send heartbeat updates to the client to keep the connection alive. Should be less than the client's timeout setting - usually set to half its value.

## 1.3.渲染进程监听webpack编译过程

经过以上的步骤，渲染进程的初始化就基本上结束了，但是我们看到在创建过程中有两个“小插曲”：

```javascript
compiler.plugin('compilation', compilation => {
    compilation.plugin('html-webpack-plugin-after-emit', (data, cb) => {
        hotMiddleware.publish({action: 'reload'});
        cb();
    });
});

compiler.plugin('done', stats => {
    logStats('Renderer', stats);
});
```

其中第二个logStats作用就是在终端屏幕上输出编译过程（后面我们在看main进程编译过程也会输出到终端中），如图所示：
![](render编译log.png)
而第一个hotMiddleware.publish()作用是什么呢？
这其实是一个钩子函数，检测webpack的编译状态，把其中的`html-webpack-plugin-after-emit`状态，发布到webpackHotMiddleware中。
>我们可以检测多种状态，比如
 - html-webpack-plugin-before-html-processing
 - html-webpack-plugin-after-html-processing
 - html-webpack-plugin-after-emit

然后我们就可以在渲染进程中检测到这一状态。
然后还记得之前的dev-client模块吗？我们再来看一下源码：

```javascript
const hotClient = require('webpack-hot-middleware/client?noInfo=true&reload=true');
//注册webpack-hot-middleware监听器
hotClient.subscribe(event => {
    //这里只处理了Main进程发送的"compiling"的事件，实际上在Render进程中还发送了"reload"的消息
    if (event.action === 'compiling') {
      ...
      <div id="dev-client">
        Compiling Main Process...
      </div>
    `;
    }
});
```

也就是说，在整个webpack-hot-middleware编译过程中发送的编译消息，将会在界面展示一个提示框提示开发者，编译器正在进行的工作。
从这一点来看也发现，该框架的作者对细节的把握多么的重视。

# 2.主进程过程分析

前面分析了渲染进程的启动过程，现在来看主进程的启动过程。

```javascript
function startMain() {
    return new Promise((resolve, reject) => {
        mainConfig.entry.main = [path.join(__dirname, '../src/main/index.dev.js')].concat(mainConfig.entry.main);
        //创建主进程的webpack
        const compiler = webpack(mainConfig);

        compiler.plugin('watch-run', (compilation, done) => {
            //向webpack-hot-middleware发布"compiling"的消息，用于页面显示
            hotMiddleware.publish({action: 'compiling'});
            done();
        });

        compiler.watch({}, (err, stats) => {
            if (err) {
                return;
            }
            ...
            if (electronProcess && electronProcess.kill) {
                //主进程文件发生改变，重启Electron
                manualRestart = true;
                process.kill(electronProcess.pid);
                electronProcess = null;
                startElectron();

                setTimeout(() => {
                    manualRestart = false;
                }, 5000);
            }

            resolve();
        });
    });
}
```

主进程的启动和渲染进程非常相似，共经历了三个步骤：
 - 创建webpack
 - 实时发布hotMiddleware状态（用于页面展示编译过程）
 - 主进程的代码"热更新"

下面我们也来分别介绍这三个过程。

## 1.1.主进程的webpack创建过程

webpack创建的关键在于配置文件，和渲染进程一样，主进程也有两个配置文件：
 - /src/main/index.dev.js
 - /.electron-vue/webpack.main.config.js

下面分别介绍他们内容。

### 主进程的index.dev.js配置文件

我们直接来看这个文件内容。

```javascript
process.env.NODE_ENV = 'development'

//安装`electron-debug`工具
require('electron-debug')({ showDevTools: true })

//安装Vue的一个chrome开发工具`vue-devtools`
require('electron').app.on('ready', () => {
  let installExtension = require('electron-devtools-installer')
  installExtension.default(installExtension.VUEJS_DEVTOOLS)
    .then(() => {})
    .catch(err => {
      console.log('Unable to install `vue-devtools`: \n', err)
    })
})
...
```

这个配置文件的作用就是安装了`electron-debug`和`vue-devtools`两个工具，其中`vue-devtools`工具因为网络原因无法安装，可以自己手动安装。

### 主进程的webpack.main.config.js配置文件

这个文件主要配置项如下：

```javascript
let mainConfig = {
    entry: {
        main: path.join(__dirname, '../src/main/index.js')
    },
    externals: [
        ...Object.keys(dependencies || {})
    ],
    module: {
        rules: [
            {
                test: /\.js$/,
                use: 'babel-loader',
                exclude: /node_modules/
            },
            {
                test: /\.node$/,
                use: 'node-loader'
            }
        ]
    },
    node: {
        __dirname: process.env.NODE_ENV !== 'production',
        __filename: process.env.NODE_ENV !== 'production'
    },
    output: {
        filename: '[name].js',
        libraryTarget: 'commonjs2',
        path: path.join(__dirname, '../dist/electron')
    },
    plugins: [
        new webpack.NoEmitOnErrorsPlugin()
    ],
    resolve: {
        extensions: ['.js', '.json', '.node']
    },
    //编译为 Electron 主进程
    target: 'electron-main'
};
```

他和渲染进程最大不同就是主进程处理的文件很有限，他不会（不需要）处理vue、图片、css、html等文件类型。

## 2.2.主进程的编译过程跟踪

这里和渲染进程的跟踪稍有不同，我们对比一下：
**渲染进程**

```javascript
compiler.plugin('compilation', compilation => {
    compilation.plugin('html-webpack-plugin-after-emit', (data, cb) => {
        hotMiddleware.publish({action: 'reload'});
        cb();
    });
});
```

**主进程**

```javascript
compiler.plugin('watch-run', (compilation, done) => {
    logStats('Main', chalk.white.bold('compiling...'));
    hotMiddleware.publish({action: 'compiling'});
    done();
});
```

从这里发现，他们分别监听了编译的不同事件。渲染进程监听的是`compilation`，而主进程监听的是`watch-run`。
>这里的"watch-run"只会出现在webpack3.X版本中，在webpack4.x版本上，"watch-run"事件被修改为了"watchRun"事件。
>webpack还支持的一些事件包括：
 - afterPlugins
 - afterResolvers
 - environment
 - afterEnvironment
 - beforeRun
 - run
 - beforeCompile
 - afterCompile

从webpack说明文档看出，"compilation"会在编译器创建时触发：
>Runs a plugin after a compilation has been created

"watchRun"也是会在编译器创建之后被触发，不同的是，这个这个事件只有在"watch mode"模式下才会生效：
>Executes a plugin during watch mode after a new compilation is triggered but before the compilation is actually started.

由于渲染进程使用了`WebpackDevServer`的热更新，因此可以检测`compilation`事件来跟踪事件。
主进程在初始化过程中，由于没有使用`WebpackDevServer`，而是开启了`watch`模式，所以可以检测到这个事件。

## 2.3.主进程的代码"热更新"

主进程没有使用WebpackDevServer的方式自动更新界面，而是通过webpack的watch模式，不断重启Electron实现的：

```javascript
compiler.watch({}, (err, stats) => {
    if (err) {
        return;
    }

    if (electronProcess && electronProcess.kill) {
        manualRestart = true;
        process.kill(electronProcess.pid);
        electronProcess = null;
        //重启Electron
        startElectron();

        setTimeout(() => {
            manualRestart = false;
        }, 5000);
    }

    resolve();
});
```

# 3.Electron过程分析

前面两节分析了主进程和渲染进程的创建流程，在调试模式命令的最后一步就是开启Electron：

```javascript
function init() {
    Promise.all([startRenderer(), startMain()])
        .then(() => {
            startElectron();
        })
        .catch(err => {
        });
}
```

下面我们来看如何开启Electron：

```javascript
function startElectron() {
    electronProcess = spawn(electron, ['--inspect=5858', '.']);
    ...
}
```

原来是通过node的spawn方法运行了electron，并传递了两个参数。
两个参数分别代表打开5858的调试端口和electron的运行目录（也就是当前目录）。
至此，调试环境运行时的流程就介绍完了，我们用一张流程图来归纳一下这个过程：
![](流程.png)


