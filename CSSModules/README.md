# CSS Modules 
## 一、局部作用域
CSS的规则都是全局的，任何一个组件的样式规则，都对整个页面有效。
产生局部作用域的唯一方法，就是使用一个独一无二的class的名字，不会与其他选择器重名。这就是 CSS Modules 的做法。
下面是一个React组件`App.js`
```
import React from 'react';
import style from './App.css';
export default ()=>{
    return (
        <h1 className={style.title}>
            Hello World
        </h1>
    )
}
```
上面代码中，我们将样式文件`App.css`输入到`style`对象，然后引用`style.title`代表一个`class`。
```
.title{
    color:red;
}
```
构建工具会将类名`style.title`编译成一个哈希字符串。
```
<h1 class="_3zyde4l1yATCOkgn-DBWEL">Hello World</h1>
```
`App.css`也会同时被编译。
```
._3zyde4l1yATCOkgn-DBWEL {
  color: red;
}
```
这样一来，这个类名就变成独一无二了，只对App组件有效。

CSS Modules 提供各种插件，支持不同的构建工具。本文使用的是 Webpack 的`css-loader`插件，因为它对 CSS Modules 的支持最好，而且很容易使用。

下面是这个示例的`webpack.config.js`。
```
module.exports={
    entry:__dirname+'/index.js',
    output:{
        publicPath:'/',
        filename:'./bundle.js'
    },
    module:{
        loaders:[
            {
                test:/\.jsx?$/,
                exclude:/node_modules/,
                loader:'babel',
                query:{
                    presets:['es2015','react','stage-0']
                }
            },{
                test:/\.css$/,
                loader:"style-loader!css-loader?modules"
            }
        ]
    }
}
```
上面代码中，关键的一行是`style-loader!css-loader?modules`，它在`css-loader`后面加了一个查询参数`modules`，表示打开 CSS Modules 功能。

## 二、全局作用域
CSS　Modules 允许使用`:global(.className)`的语法，声明一个全局规则。凡是这样声明的`class`，都不会被编译成哈希字符串。

CSS Modules 还提供一种显式的局部作用域语法`:local(.className)`,等同于`.className`，所以上面的`App.css`也可以写成下面这样。
```
:local(.title){
    color:red;
}

:global(.title){
    color:green;
}
```
## 三、定制哈希类名
`css-loader`默认的哈希算法是`[hash:base64]`，这会将`.title`编译成`._3zyde4l1yATCOkgn-DBWEL`这样的字符串。

`webpack.config.js`里面可以定制哈希字符串的格式。
```
module:{
    loaders:[
        {
            test:/\.css$/,
            loader:"style-loader!css-loader?modules&localIdentName=[path][name]---[local]---[hash:base64:5]"
        }
    ]
}
```
你会发现`.title`被编译成了`demo03-components-App---title---GpMto`。

## 四、Class的组合
在 CSS Modules 中，一个选择器可以继承另一个选择器的规则，这称为"组合"（"composition"）。

在`App.css`中，让`.title`继承`.className `。
```
.className {
  background-color: blue;
}

.title {
  composes: className;
  color: red;
}
```
相应地， h1的class也会编译成<h1 class="demo03-components-App---title---XSDrm demo03-components-App---className---MWmot">。

## 五、输入其他模块
选择器也可以继承其他CSS文件里面的规则。
`another.css`
```
.className{
    background-color:blue;
}
```
`App.css`可以继承`another.css`里面的规则
```
.title{
    composes:className from './antoher.css';
    color:red;
}
```

## 六、输入变量
CSS Modules支持使用变量，不过需要安装`postcss`和`postcss-modules-values`.
```
$ npm install --save postcss-loader postcss-modules-values
```

把`postcss-loader`加入`webpak.config.js`.
```
var values = require('postcss-modules-values');
module.exports = {
    entry:__dirname+'/index.js',
    output:{
        publicPath:'/',
        filename:'bundle.js'
    },
    module:{
        loaders:[
            {
                test:/\.jsx?$/,
                exclude:/node_modules/,
                loader:'babel',
                query:{
                    presets:['es2015','stage-0','react']
                }
            },
            {
                test:/\.css$/,
                loader:"style-loader!css-loader?modules!postcss-loader"
            }
        ]
    },
    postcss:[
        values
    ]
}

```
接着，在`colors.css`里面定义变量。
```
@value blue:#0c77f8;
@value red:#ff0000;
@value green:#aaf200;
```
`App.css`可以引用这些变量。
```
@value colors:"./colors.css";
@value blue,red,green from colors;
.title{
    color:red;
    background-color:blue;
}
```
