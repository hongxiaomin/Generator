# require()源码解读

## 一、require()的基本用法
当Node遇到require(x)时，按下面的顺序处理。
1. 如果X是内置模块(比如require('http'))
    * 返回该模块
    * 不再继续执行
2. 如果X以“./”或者"/"或者“../”开头
    * 根据X所在的父模块，确定X的绝对路径。
    * 将X当成文件，依次查找下面文件，只要其中有一个存在，就返回该文件，不再继续执行。
    ```
        X
        X.js
        X.json
        X.node
    ```
    * 将X当成目录，依次查找下面文件，只要其中有一个存在，就返回该文件，不再继续执行。
    ```
        X/package.json(main字段)
        X/index.js
        X/index.json
        X/index.node
    ```
3. 如果X不带路径
    * 根据X所在的父模块，确定X可能的安装目录。
    * 依次在每个目录中，将X当成文件名或目录名加载。
4. 抛出“not found”

例：
当前脚本文件/home/ry/projects/foo.js 执行了require('bar')，这属于上面的第三种情况。Node内部运行过程如下。

首先，确定X的绝对路径可能是下面这些位置，依次搜素每一个目录。
```
/home/ry/projects/node_modules/bar
/home/ry/node_modules/bar
/home/node_modules/bar
/node_modules/bar
```
搜索时，Node先将bar当成文件名，依次尝试加载下面这些文件，只要有一个成功就返回。
```
bar
bar.js
bar.json
bar.node
```
如果都不成功，说明bar可能是目录名，于是依次尝试加载下面这些文件
```
bar/package.json
bar/index.js
bar/index.json
bar/index.node
```
如果在所有目录中，都无法找到bar对应的文件或目录，就抛出一个错误。

## 二、Module构造函数
```
function Module(id,parent){
    this.id=id;
    this.exports={};
    this.parent=parent;
    this.filename=null;
    this.loaded = false;
    this.children =[];
}

module.exports = Module;
var module = new Module(filename,parent);
```
上面代码中，Node定义了一个构造函数Module，所有的模块都是Module的实例。可以看到，当前模块（module.js）也是Module的一个实例。

每个实例都有自己的属性。下面通过一个例子，看看这些属性的值是什么。新建一个脚本文件a.js。
```
//a.js
console.log('module.id:',module.id);
console.log('module.exports:',module.exports);
console.log('module.parent:',module.parent);
console.log('module.filename:',module.filename);
console.log('module.loaded:',module.loaded);
console.log('module.children:',module.children);
console.log('module.paths:',module.paths);
```
运行这个脚本
```
$ node a.js

module.id: .
module.exports: {}
module.parent: null
module.filename: /home/ruanyf/tmp/a.js
module.loaded:false
module.children: []
module.paths: [
    '/home/ruanyf/tmp/node_modules',
    '/home/ruanyf/node_modules',
    '/home/node_modules',
    '/node_modules'
]

```
可以看到，如果没有父模块，直接调用当前模块，parent属性就是null，id属性就是一个点。filename属性是模块的绝对路径，path属性是一个数组，包含了模块可能的位置。另外，输出这些内容时，模块还没有全部加载，所以loaded属性为false。

新建另一个脚本文件b.js，让其调用a.js。
```
//b.js
var a = require('./a.js');
```
运行b.js
```
$ node b.js

module.id: /home/ruanyf/tmp/a.js
module.exports: {}
module.parent: {object}
module.filename: /home/ruanyf/tmp/a.js
module.loaded:false
module.children: []
module.paths: [
    '/home/ruanyf/tmp/node_modules',
    '/home/ruanyf/node_modules',
    '/home/node_modules',
    '/node_modules'
]

```
上面代码中，由于a.js被b.js调用，所以parent属性指向b.js模块，id属性和filename属性一致，都是模块的绝对路径。

## 三、模块实例的require方法
每个模块实例都有一个require方法。
```
Module.prototype.require = function(path){
    return Module._load(path,this);
}
```
由此可知，require并不是全局性命令，而是每个模块提供的一个内部方法，也就是说，只有在模块内部才能使用require命令（唯一的例外是REPL环境）。另外，require其实内部调用Module._load方法。

下面来看 Module._load的源码。
```
Module._load = function(request,parent,isMain){
    //计算绝对路径
    var filename = Module._resolveFilename(request,parent);

    //第一步：如果有缓存，取出缓存
    var cacheModule = Module._cache(filename);
    if(cacheModule){
        return cacheModule.exports;
    }

    //第二步：是否为内置模块
    if(NativeModule.exists(filename)){
        return NativeModule.require(filename);
    }

    //第三步：生成模块实例，存入缓存
    var module = new Module(filename,parent);
    Module._cache[filename]=module;

    //第四步：加载模块
    try{
        module.load(filename);
        hadException=false;
    }finally{
        if(hadException){
            delete Module._cache[filename];
        }
    }
    //第五步：输出模块的exports属性
    return module.exports;
}
```
上面代码中，首先解析出模块的绝对路径（filename），以它作为模块的识别符。然后，如果模块已经在缓存中，就从缓存取出；如果不在缓存中，就加载模块。

因此，Module._load 的关键步骤是两个。
```
Module._resolveFilename():确定模块的绝对路径
Module.load():加载模块
```

## 四、模块的绝对路径
下面是Module._resolveFilename 方法的源码
```
Module._resolveFilename = function(request,parent){
    //第一步：如果是内置模块，不含路径返回
    if(NativeModule.exists(request)){
        return request;
    }

    //第二步：确定所有可能的路径
    var resolveModule = Module._resolveLookupPaths(request,parent);
    var id = resolveModule[0];
    var paths = resolveModule[1];

    //第三步：确定哪一个路径为真
    var filename = Module._findPath(request,paths);
    if(!filename){
        var err = new Error("Cannot find module "+request);
        err.code = 'MODULE_NOT_FOUND';
        throw err;
    }
    return filename;
}
```
上面代码中，在Module.resolveFilename方法内部，又调用了两个方法Module.resolveLookupPaths()和Module._findPath(),前者用来列出可能的路径，后者用来确认哪一个路径为真。

有时在项目代码中，需要调用模块的绝对路径，那么除了module.filename，Node还提供一个require.resolve方法，供外部调用，用于从模块名取到绝对路径。

## 五、加载模块
有了模块的绝对路径，就可以加载该模块了。下面是module.load方法的源码。
```
Module.prototype.load = function(filename){
    var extension = path.extname(filename)||'.js';
    if(!Module._extensions[extension]) extension = '.js';
    Module._extensions[extension](this,filename);
    this.loaded=true;
}
```

