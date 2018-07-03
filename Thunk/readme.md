# Thunk 函数的含义和用法
## 一、参数的求值策略
Thunk函数早在上个世纪60年代就诞生了。

那时，编程语言刚刚起步，计算机学家还在研究，编译器怎么写比较好。**一个争论的焦点是“求值策略”，即函数的参数到底应该何时求职。**
```
var x = 1;
function f(m){
    return m*2;
}
f(x+5);
```
上面代码先定义函数f,然后向它传入表达式x+5。请问，这个表达式应该何时求值？

一种意见是*传值调用*（call by value），即在进入函数体之前，就计算x+5的值（等于6），再将这个值传入函数f。
```
f(x+5);
//传值调用时，等同于
f(6);
```
另一种意见是*“传名调用”*（call by name），即直接将表达式x+5传入函数体，只在用到它的时候求值。
```
f(x+5);
//传名调用时，等同于
(x+5)*2
```
**传值调用和传名调用，哪一种比较好？回答是各有利弊。**传值调用比较简单，但是对参数求值的时候，实际上还没用到这个参数，有可能造成性能损失。
```
function f(a,b){
    return b;
}
f(3*x*x-2*x-1,x);
```
上面代码中，函数f的第一个参数是一个复杂的表达式，但是函数体内根本没用到。对这个参数求值，实际上是不必要的。

因此，有一些计算机学家倾向于“传名调用”，即只在执行时求值。

## 二、Thunk函数的含义
编译器的“传名调用”实现，往往是将参数放到一个临时函数之中，再将这个临时函数传入函数体。这个临时函数就叫做Thunk函数。
```
function f(m){
    return m*2;
}
f(x+5);
//等同于
var thunk = function(){
    return x+5;
}

function f(thunk){
    return thunk()*2;
}
```
上面代码中，函数f的参数x+5被一个函数替换了。凡是用到原参数的地方，对Thunk函数求值即可。
**这就是Thunk函数的定义，它是“传名调用”的一种实现策略，用来替换某个表达式。**

## 三、JavaScript语言的Thunk函数
JavaScript语言是传值调用，它的Thunk函数含义有所不同。**在JavaScript语言中，Thunk函数替换的不是表达式，而是多参数函数，将其替换成单参数的版本，且只接受回调函数作为参数**
```
//正常版本的readFile（多参数版本）
fs.readFile(fileName,callback);

//Thunk 版本的readFile（单参数版本）
var readFileThunk = Thunk(fileName);
readFileThunk(callback);

var Thunk = function(fileName){
    return function(callback){
        return fs.readFile(fileName,callback);
    }
}
```
上面代码中，fs模块的readFile方法是一个多参数函数，两个参数分别为文件名和回调函数。经过转换器处理，它变成了一个单参数函数，只接受回调函数作为参数。这个单参数版本，就叫做Thunk函数。

任何函数，只要参数有回调函数，就能写成Thunk函数的形式。下面是一个简单的Thunk函数转换器。
```
var Thunk = function(fn){
    return function(){
        var args = Array.prototype.slice.call(arguments);
        return function(callback){
            args.push(callback);
            return fn.apply(this.args);
        }
    }
}
```
使用上面的转换器，生成fs.readFile的Thunk函数
```
var readFileThunk = Thunk(fs.readFile);
readFileThunk(fileA)(callback);
```

## 四、Thunkify 模块
生产环境的转换器，建议使用Thunkify模块

首先是安装。
```
$ npm install thunkify
```
使用方式如下。
```
var thunkify = require('thunkify');
var fs = require('fs');
var read = thunkify(fs.readFile);
read('package.json')(function(err,str){
    //...
})
```
Thunkify 的源码与上一节那个简单的转换器非常像。
```
function thunkify (fn){
    return function(){
        var args = new Array(arguments.length);
        var ctx = this;
        for(var i=0;i<args.length;++i){
            args[i]=arguments[i];
        }
        return function (done){
            var called;
            args.push(function(){
                if(called) return;
                called = true;
                done.apply(null,arguments);
            });
            try {
                fn.apply(ctx,args);
            }catch(err){
                done(err);
            }
        }
    }
}
```
它的源码主要多了一个检查机制，变量called确保回调函数只运行一次。这样的设计与下文的Generator函数相关。


## 五、Generator函数的流程管理
以读取文件为例。下面的Generator函数封装了两个异步操作。
```
var fs = require('fs');
var thunkify = require('thunkify');
var readFile = thunkify(fs.readFile);
var gen = function* (){
    var r1 = yield readFile('/etc/fstab');
    console.log(r1.toString());
    var r2 = yield readFile('/etc/shells');
    console.log(r2.toString());
};

```
上面代码中，yield命令用于将程序执行权移出Generator函数，那么就需要一种方法，将执行权再交给Generator函数。

这种方法就是Thunk函数，因为它可以在回调函数里，将执行权交还给Generator函数。为了便于理解，我们先看如何手动执行上面这个Generator函数。
```
var g = gen();
var r1 = g.next();
r1.value(function(err,data){
    if(err) throw err;
    var r2 = g.next(data);
    r2.value(function(err,data){
        if(err) throw err;
        g.next(data);
    })
});
```
上面代码中，变量g是Generator函数的内部指针，表示目前执行到哪一步。next方法负责将指针移动到下一步，并返回该步的信息（value属性和done属性）。

仔细查看上面的代码，可以发现Generator函数的执行过程，其实是将同一个回调函数，反复传入next方法的value属性。这使得我们可以用递归来自完成这个过程。

## 六、Thunk函数的自动流程管理
Thunk函数真正的威力，在于可以自动执行Generator函数。下面就是一个基于Thunk函数的Generator执行器。
```
function run(fn){
    var gen = fn();
    function next(err,data){
        var result = gen.next(data);
        if(result.done) return;
        result.value(next)
    }
    next();
}

run(gen);
```

上面代码的run函数，就是一个Generator函数的自动执行器。内部的next函数就是Thunk的回调函数。next函数先将指针移到Generator函数的下一步（gen.next方法），然后判断Generator函数是否结束（result.done属性），如果没结束，就将next函数再传入Thunk函数（result.value属性），否则就直接退出。

有了这个执行器，执行Generator函数方便多了。不管有多少个异步操作，直接传入run函数即可。当然，前提是每一个异步操作，都要是Thunk函数，也就是说，跟在yield命令后面的必须是Thunk函数。

```
var gen = function* (){
    var f1 = yield readFile('fileA');
    var f2 = yield readFile('fileB');
    //...
    var fn = yield readFile('fileN');
};

run(gen);
```
上面代码中，函数gen封装了n个异步的读取文件操作，只要执行run函数，这些操作就会自动完成。这样一来，异步操作不仅可以写得像同步操作，而且一行代码就可以执行。

Thunk函数并不是Generator函数自动执行的唯一方案。因为自动执行的关键是，必须有一种机制，自动控制Generator函数的流程，接受和交还程序的执行权。回调函数可以做到这一点，Promise对象也可以做到这一点。

