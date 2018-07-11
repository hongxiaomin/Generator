# webpack
一个常见的`webpack`配置文件
```
const webpack = require('webpack');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const ExtractTextPlugin = require('extract-text-webpack-plugin');

module.exports={
    entry:__dirname+"/app/main.js",//唯一入口文件
    output:{
        path:__dirname+"/build",
        filename:"bundle-[hash].js"
    },
    devtool:'none',
    devServer:{
        contentBase:"./public",//本地服务器所加载的页面所在的目录
        historyApiFallback:true,//不跳转
        inline:true,
        hot:true
    },
    module:{
        rules:[
            {
                test:/(\.jsx|\.js)$/,
                use:{
                    loader:"babel-loader"
                },
                exclude:/node_modules/
            },
            {
                test:/\.css$/,
                use:ExtractTextPlugin.extract({
                    fallback:"style-loader",
                    use:[{
                        fallback:"css-loader",
                        options:{
                            modules:true,
                            localIdentName:'[name]_[local]--[hash:base64:5]'
                        }
                    },{
                        loader:"postcss-loader"
                    }]
                })
            }
        ]
    },
    plugins:[
        new webpack.BannerPlugin('版权所有，翻版必究'),
        new HtmlWebpackPlugin({
            template:__dirname+"/app/index.tmpl.html"
        }),
        new webpack.optimize.OccurrenceOrderPlugin(),
        new webpack.optimize.UglifyJsPlugin(),
        new ExtractTextPlugin("style.css")
    ]
};

```
## 什么是webpack，为什么要使用它？
**webpack**可以看做是**模块打包机**：它做的事情是，分析你的项目结构，找到JavaScript模块以及其他的一些浏览器不能直接运行的拓展语言（Scss,TypeScript等），并将其转换和打包为适合的格式供浏览器使用。

## webpack和Grunt以及Gulp相比有什么特性
其实webpack和另外两个并没有太多的可比性，Gulp/Grunt是一种能够优化前端的开发流程的工具，而webpack是一种模块化的解决方案，webpack的优点使得webpack在很多场景下可以替代Gulp、Grunt类的工具。

Grunt和Gulp的工作方式：在一个配置文件中，指明对某些文件进行类似编译，组合，压缩等任务的具体步骤，工具之后可以自动替你完成这些任务。

webpack的工作方式是：把你的项目当做一个整体，通过一个给定的主文件（如：index.js），webpack将从这个文件开始找到你的项目的多有依赖，使用loaders处理它们，最后打包为一个（或多个）浏览器可以识别的JavaScript文件。

webpack的处理速度更快更直接，能打包更多不同类型的文件。

##开始使用webpack
### 安装
```
//全局安装
npm install -g webpack
、、安装到项目目录
npm install --save-dev webpack
```

**正式使用webpack前的准备**
1. 在文件中创建package.json文件，这是一个标准的npm说明文件，里面包括当前项目的依赖模块，自定义的脚本任务等。在终端使用`npm init`命令可以自动创建这个package.json文件。

```
npm init
```
2. package.json文件已经就绪，在项目中安装Webpack作为依赖包
```
npm install --save-dev webpack
```

通过简单的配置，`webpack`就可以在打包时为我们生成的`source maps`，这为我们提供了一种对应编译文件和源文件的方法，使得编译后的代码可读性更高，也更容易调试。
在`webpack`的配置文件中配置`source maps`，需要配置`devtool`，它有以下四种不同的配置选项，各具优缺点，描述如下：
* *source-map* 在一个单独的文件中产生一个完整且功能完全的文件。这个文件具有最好的source map，但是它会减慢打包速度；
* *cheap-module-source-map* 在一个单独的文件中生成一个不带列映射的map，不带列映射提高了打包速度，但是也使得浏览器开发者工具只能对应到具体的行，不能对应到具体的列（符号），会对调试造成不便；
* *eval-source-map* 使用eval打包源文件模块，在同一个文件中生成干净的完整的source map。这个选项可以在不影响构建速度的前提下生成完整的sourcemap，但是对打包后输出的JS文件的执行具有性能和安全的隐患。在开发阶段这是一个非常好的选项，在生产阶段则一定不要启用这个选项；
* *cheap-module-eval-source-map* 这是在打包文件时最快的生成source map的方法，生成的Source Map 会和打包后的JavaScript文件同行显示，没有列映射，和eval-source-map选项具有相似的缺点；

对小到中型的项目中，`eval-source-map`是一个很好的选项，再次强调你只应该开发阶段使用它


### 使用webpack构建本地服务器
想不想让你的浏览器监听你的代码的修改，并自动刷新显示修改后的结果，其实Webpack提供一个可选的本地开发服务器，这个本地服务器基于node.js构建，可以实现你想要的这些功能，不过它是一个单独的组件，在webpack中进行配置之前需要单独安装它作为项目依赖
```
npm install --save-dev webpack-dev-server
```
devserver作为webpack配置选项中的一项，以下是它的一些配置选项
* *contentBase* 默认webpack-dev-server会为根文件夹提供本地服务器，如果想为另外一个目录下的文件提供本地服务器，应该在这里设置其所在目录（本例设置到“public"目录）
* *port* 设置默认监听端口，如果省略，默认为”8080“
* *inline* 设置为true，当源文件改变时会自动刷新页面
* *historyApiFallback* 在开发单页应用时非常有用，它依赖于HTML5 history API，如果设置为true，所有的跳转将指向index.html

```
module.exports={
    devtool:"eval-source-map",
    entry:__dirname+"/app/main.js",
    output:{
        path:__dirname+"/public",
        filename:"bundle.js"
    },
    devServer:{
        contentBase:"./public",
        historyApiFallback:true,
        inline:true
    }
}
```
### Loaders

通过使用不同的`loader`，`webpack`有能力调用外部的脚本或工具，实现对不同格式的文件的处理

`Loaders`需要单独安装并且需要在`webpack.config.js`中的`modules`关键字下进行配置，
Loaders的配置包括以下几方面：
* test:一个用以匹配loaders所处理文件的拓展名的正则表达式（必须）
* loader:loader的名称（必须）
* include/exclude:手动添加必须处理的文件（文件夹）或屏蔽不需要处理的文件（文件夹）（可选）；
* query:为loaders提供额外的设置选项（可选）
```
module.exports={
    devtool:"eval-source-map",
    entry:__dirname+"/app/main.js",
    output:{
        path:__dirname+"/public",
        filename:"bundle.js"
    },
    devServer:{
        contentBase:"./public",
        historyApiFallback:true,
        inline:true
    },
    module:{
        rules:[
            {
                test:/(\.jsx|\.js)$/,
                use:{
                    loader:"babel-loader",
                    options:{
                        presets:[
                            "env","react"
                        ]
                    }
                },
                exclude:/node_module/
            }
        ]
    }
}
```

### Babel 
Babel其实可以完全在 `webpack.config.js` 中进行配置，但是考虑到`babel`具有非常多的配置选项，在单一的`webpack.config.js`文件中进行配置往往使得这个文件显得太复杂，因此一些开发者支持把babel的配置选项放在一个单独的名为 `".babelrc"` 的配置文件中。
**webpack会自动调用.babelrc里的babel配置选项**

```
//webpack.config.js
module.exports={
    devtool:"eval-source-map",
    entry:__dirname+"/app/main.js",
    output:{
        path:__dirname+"/public",
        filename:"bundle.js"
    },
    devServer:{
        contentBase:"./public",
        historyApiFallback:true,
        inline:true
    },
    module:{
        rules:[
            {
                test:/(\.jsx|\.js)$/,
                use:{
                    loader:"babel-loader",
                },
                exclude:/node_module/
            }
        ]
    }
}


//.babelrc
{
    "preset":["react","env"]
}
```
### 一切皆模块
Webpack有一个不可不说的优点，它把所有的文件都都当做模块处理，JavaScript代码，CSS和fonts以及图片等等通过合适的loader都可以被处理。

### CSS
webpack提供两个工具处理样式表，`css-loader` 和 `style-loader`，二者处理的任务不同，`css-loader`使你能够使用类似`@import` 和 `url(...)`的方法实现 `require()`的功能,`style-loader`将所有的计算后的样式加入页面中，二者组合在一起使你能够把样式表嵌入webpack打包后的JS文件中。

被称为CSS modules的技术意在把JS的模块化思想带入CSS中来，通过CSS模块，所有的类名，动画名默认都只作用于当前模块。Webpack对CSS模块化提供了非常好的支持，只需要在CSS loader中进行简单配置即可，然后就可以直接把CSS的类名传递到组件的代码中，这样做有效避免了全局污染。具体的代码如下
```
module.exports={
    devtool:"eval-source-map",
    entry:__dirname+"/app/main.js",
    output:{
        path:__dirname+"/public",
        filename:"bundle.js"
    },
    module:{
        rules:[
            {
                test:/\.css$/,
                use:[
                    {
                        loader:"style-loader"
                    },{
                        loader:"css-loader",
                        options:{
                            modules:true,
                            localIdentName:'[name]__[local]--[hash:base64:5]'
                        }
                    }
                ]
            }
        ]
    }
}
```
### CSS预处理器
`Sass` 和 `Less` 之类的预处理器是对原生CSS的拓展，它们允许你使用类似于`variables`, `nesting`, `mixins`, `inheritance`等不存在于CSS中的特性来写CSS，CSS预处理器可以这些特殊类型的语句转化为浏览器可识别的CSS语句，


### 插件（Plugins）
插件（Plugins）是用来拓展Webpack功能的，它们会在整个构建过程中生效，执行相关的任务。
Loaders和Plugins常常被弄混，但是他们其实是完全不同的东西，可以这么来说，loaders是在打包构建过程中用来处理源文件的（JSX，Scss，Less..），一次处理一个，插件并不直接操作单个文件，它直接对整个构建过程其作用。
Webpack有很多内置插件，同时也有很多第三方插件，可以让我们完成更加丰富的功能。

####　使用插件的方法
要使用某个插件，我们需要通过npm安装它，然后要做的就是在webpack配置中的plugins关键字部分添加该插件的一个实例（plugins是一个数组）继续上面的例子，我们添加了一个给打包后代码添加版权声明的插件。
```
const webpack = require('webpack');
module.exports={
    devtool:"eval-source-map",
    entry:__dirname+"/app/main.js",
    output:{
        path:__dirname+"/public",
        filename:"bundle.js"
    },
    plugins:[
        new webpack.BannerPlugin('版权所有，翻版必究')
    ]
}
```
推荐几个常用的插件
* **HtmlWebpackPlugin**
    这个插件的作用是依据一个简单的index.html模板，生成一个自动引用你打包后的JS文件的新index.html。这在每次生成的js文件名称不同时非常有用（比如添加了hash值）。
    安装
    ```
    npm install --save-dev html-webpack-plugin
    ```
    这个插件自动完成了我们之前手动做的一些事情，在正式使用之前需要对一直以来的项目结构做一些更改：
    1. 移除public文件夹，利用此插件，index.html文件会自动生成，此外CSS已经通过前面的操作打包到JS中了。
    2. 在app目录下，创建一个index.tmpl.html文件模板，这个模板包含title等必须元素，在编译过程中，插件会依据此模板生成最终的html页面，会自动添加所依赖的 css, js，favicon等文件，index.tmpl.html中的模板源代码如下：
    ```
    <!DOCTYPE html>
        <html lang="en">
            <head>
                <meta charset="utf-8">
                <title>Webpack Sample Project</title>
            </head>
            <body>
                <div id="root"></div>
            </body>
        </html>
    ```
    3. 更新webpack的配置文件，方法同上,新建一个build文件夹用来存放最终的输出文件
    ```
    const webpack = require('webpack');
    const HtmlWebpackPlugin = require('html-webpack-plugin');
    module.exports={
        devtool:"eval-source-map",
        entry:__dirname+"/app/main.js",
        output:{
            path:__dirname+"/build",
            filename:"bundle.js"
        },
        devServer:{
            contentBase:"./build",
            historyApiFallback:true,
            inline:true
        },
        module:{
            rules:[
                {
                    test:/(\.jsx|\.js)$/,
                    use:{
                        loader:"babel-loader",
                    },
                    exclude:/node_module/
                },
                {
                    test:/\.css$/,
                    use:[
                        {
                            loader:"style-loader"
                        },{
                            loader:"css-loader",
                            options:{
                                modules:true,
                            }
                        },{
                            loader:"postcss-loader"
                        }
                    ]
                }
            ]
        },
        plugins:[
            new webpack.BannerPlugin('版权所有，翻版必究'),
            new HtmlWebpackPlugin({
                template:__dirname+"/app/index.tmpl.html"
            })
        ]
    }
    ```
* **Hot Module Replacement**
    `Hot Module Replacement`（HMR）也是webpack里很有用的一个插件，它允许你在修改组件代码后，自动刷新实时预览修改后的效果。
    在webpack中实现HMR也很简单，只需要做两项配置
    1. 在webpack配置文件中添加HMR插件；
    2. 在Webpack Dev Server中添加“hot”参数；
    HMR是一个webpack插件，它让你能浏览器中实时观察模块修改后的效果，但是如果你想让它工作，需要对模块进行额外的配额；
    ```
    const webpack = require('webpack');
    const HtmlWebpackPlugin = require('html-webpack-plugin');
    module.exports={
        devtool:"eval-source-map",
        entry:__dirname+"/app/main.js",
        output:{
            path:__dirname+"/build",
            filename:"bundle.js"
        },
        devServer:{
            contentBase:"./build",
            historyApiFallback:true,
            inline:true,
            hot:true
        },
        plugins:[
            new webpack.BannerPlugin('版权所有，翻版必究'),
            new HtmlWebpackPlugin({
                template:__dirname+"/app/index.tmpl.html"
            }),
            new webpack.HotModuleReplacementPlugin(),
        ]
    }
    ```
* **优化插件**
    webpack提供了一些在发布阶段非常有用的优化插件，它们大多来自于webpack社区，可以通过npm安装，通过以下插件可以完成产品发布阶段所需的功能
    - OccurenceOrderPlugin :为组件分配ID，通过这个插件webpack可以分析和优先考虑使用最多的模块，并为它们分配最小的ID
    - UglifyJsPlugin：压缩JS代码；
    - ExtractTextPlugin：分离CSS和JS文件
    OccurenceOrder 和 UglifyJS plugins 都是内置插件，你需要做的只是安装其它非内置插件
    ```
    npm install --save-dev extract-text-webpack-plugin
    ```
### 缓存
缓存无处不在，使用缓存的最好方法是保证你的文件名和文件内容是匹配的（内容改变，名称相应改变）
webpack可以把一个哈希值添加到打包的文件名中，使用方法如下,添加特殊的字符串混合体（[name], [id] and [hash]）到输出文件名前
```
const webpack = require('webpack');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const ExtractTextPlugin = require('extract-text-webpack-plugin');

module.exports = {
..
    output: {
        path: __dirname + "/build",
        filename: "bundle-[hash].js"
    },
   ...
};
```
### 去除build文件中的残余文件
添加了`hash`之后，会导致改变文件内容后重新打包时，文件名不同而内容越来越多，因此这里介绍另外一个很好用的插件`clean-webpack-plugin`。
安装：
```
cnpm install clean-webpack-plugin --save-dev
```
使用：
引入`clean-webpack-plugin`插件后在配置文件的`plugins`中做相应配置即可：
```
const CleanWebpackPlugin = require("clean-webpack-plugin");
  plugins: [
    ...// 这里是之前配置的其它各种插件
    new CleanWebpackPlugin('build/*.*', {
      root: __dirname,
      verbose: true,
      dry: false
  })
  ]
```