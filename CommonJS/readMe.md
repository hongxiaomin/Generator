# CommonJS 和 AMD /CMD

为什么模块很重要？
因为有了模块，我们就可以更方便地使用别人的代码，想要什么功能，就加载什么模块。但是，前提是必须以同样的方式编写模块。

JS中的模块规范（CommonJS，AMD，CMD），

### 一、CommonJS
CommonJS API定义很多普通应用程序（主要指非浏览器的应用）使用的API。

在兼容CommonJS的系统中，你可以使用JavaScript开发以下程序：
1. 服务器端JavaScript应用程序
2. 命令行工具
3. 图形界面应用程序
4. 混合应用程序
NodeJS是CommonJS规范的实现。webpack也是以CommonJS的形式来书写。

在CommonJS中，有一个全局性方法require()，用于加载模块。

CommonJS定义的模块分为：{模块引用(require)} {模块定义(exports)} {模块标识(module)}

require()用来引入外部模块；
exports对象用于导出当前模块的方法或变量，唯一的导出口；
module对象就代表模块本身。

#### CommonJS原理
浏览器不兼容CommonJS的根本原因，在于缺少四个nodeJS环境的变量。
* module
* exports
* require 
* global

只要能够提供这四个变量，浏览器就能加载CommonJS模块。
### Browserify 的实现
*Browserify*是目前最常用的CommonJS格式转换的工具。

例子：main.js模块加载foo.js模块
```
//foo.js
module.exports = function(x){
    console.log(x);
}

//main.js
var foo = require('./foo');
foo('Hi');
```
使用下面的命令，就能将main.js转为浏览器可用的格式

```
browserify main.js > compiled.js
```
Browserify 到底做了什么？安装一下**browser-unpack**就能看清楚了。
```
$ npm install browser-unpack -g
```
然后，将前面生成的compiled.js解包
```
$ browser-unpack < compiled.js

[
    {
        "id":1,
        "source":"module.exports = function(x){\n console.log(x);\n};",
        "deps":{}
    },
    {
        "id":2,
        "source":"var foo = require(\"./foo\");\nfoo(\"Hi\");",
        "deps":{"./foo":1},
        "entry":true
    }
]
```
可以看到，browserify将所有模块放入一个数组，id属性是模块的编号，source属性是模块的源码，deps属性是模块的依赖。

因为main.js里面加载了foo.js，所以deps属性就指定./foo对应1号模块。执行的时候，浏览器遇到require('./foo')语句，就自动执行1号模块的source属性，并将执行后的module.exports属性值输出。

### Tiny Browser Require
虽然Browserify很强大，但不能在浏览器里操作，有时就很不方便。

根据*mocha*的内部实现，做一个纯浏览器的CommonJS模块加载器tiny-browser-require 。完全不需要命令行，直接放进浏览器即可，所有代码只有30多行。

它的逻辑非常简单，就是把模块读入数组，加载路径就是模块的id。

```
function require(p){
    var path = require.resolve(p);
    var mod = require.modules[path];
    if(!mod) throw new Error('failed to require " ' +p+'"');
    if(!mod.exports){
        mod.exports = {};
        mod.call(mod.exports,mod,mod.exports,require.relative(path));
    }
    return mod.exports;
}

require.modules={};

require.resolve = function(path){
    var orig = path;
    var reg = path + '.js';
    var index = path + '/index.js';
    return require.modules[reg]&&reg || require.modules[index]&&index || orig;
}

require.register = function(path,fn){
    require.modules[path]=fn;
}

require.relative = function(parent){
    return function(p){
        if('.' != p.charAt(0)) return require(p);
        var path = parent.split('/');
        var segs = p.split('/');
        path.pop();

        for(var i = 0;i<segs.length;i++){
            var seg = segs[i];
            if('..' == seg) path.pop();
            else if('.'!=seg) path.push(seg);
        }

        return require(path.join('/'));
    }
}
```
## 二、AMD
基于CommonJS规范的nodeJS出来以后，服务端的模块概念已经形成，很自然地，大家就想要客户端模块。而且最好两者能够兼容，一个模块不用修改，在服务器和浏览器都可以运行。但是，由于一个重大的局限，是得CommonJS规范不适用于浏览器环境。还是上面的代码，如果在浏览器中运行，会有一个很大的问题。
```
var math = require('math');
math.add(2,3);
```

第二行math.add(2,3)，在第一行require('math')之后运行，因此必须等math.js加载完成。也就是说，如果加载时间很长，整个应用就会停在那里等。**您会注意到require是同步的。**

这对服务器端不是一个问题，因为所有的模块都存放在本地硬盘，可以同步加载完成，等待时间就是硬盘的读取时间。但是，对于浏览器，这却是一个很大的问题，因为模块都放在服务器端，等待时间取决于网速的快慢，可能要等很长时间，浏览器处于“假死”状态。

因此，浏览器端的模块，不能采用“同步加载”(synchronous)，只能采用"异步加载"(asynchronous)。这就是AMD规范诞生的背景。

CommonJS是主要为了JS在后端的表现制定的，它是不适合前端的，AMD（异步模块定义）出现了，它就主要为前端JS的表现制定规范。

**AMD**是“Asynchronous Module Definition”的缩写，意思就是“异步模块定义”。它采用异步方式加载模块，模块的加载不影响它后面语句的运行。所有依赖这个模块的语句，都定义在一个回调函数中，等到加载完成之后，这个回调函数才会运行。

AMD也采用require()语句加载模块，但是不同于CommonJS，它要求两个参数：
```
require([module],callback);
```
第一个参数[module]，是一个数组，里面的成员就是要加载的模块；第二个参数callbcak，则是加载成功之后的回调函数。如果将前面的代码改写成AMD形式，就是下面这样：
```
require(['math'],function(math){
    math.add(2,3);
});
```
math.add()与math模块加载不是同步的，浏览器不会发生假死。所以很显然，AMD比较适合浏览器环境。目前有两个JavaScript库实现了AMD规范：*require.js*和*curl.js*.

**RequreJS就是实现了AMD规范的**

require.js的诞生，就是为了解决这两个问题：
1. 实现JS文件的异步加载，避免网页失去响应；
2. 管理模块之间的依赖性，便于代码的编写和维护。

## 三、CMD
CMD即Common Module Definition通用模块定义。

seajs,就是遵循CMD规范。
语法如下：
```
define(id?,deps?,factory);
```
**CMD理念**
1. 一个文件一个模块，所以经常就用文件名作为模块id
2. CMD推崇依赖就近，所以一般不在define的参数中写依赖，在factory中写。

factory有三个参数：
```
function(require,exports,module)
``` 
实例
```
define(function(require,exports,module){
    var random = require('../random.js');
    random.doSomething();
    
    var int = require('../int.js');
    int.doSomething();
})

```

**AMD与CMD最大的区别**
1. 对于依赖的模块，AMD是提前执行，CMD是延迟执行。
2. CMD推崇依赖就近，AMD推崇依赖前置；
3. AMD的API默认是一个当多个用，CMD的API严格区分，推崇职责单一。




