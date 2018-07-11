#Promise

## 含义
Promise，就是一个容器，里面保存着某个未来才会结束的事件（通常是一个异步操作）的结果。从语法上说，Promise是一个对象，从它可以获取异步操作的消息。

`promise`对象有以下两个特点：
1. 对象的状态不受外界影响。`Promise`对象代表一个异步操作，有三种状态：pending（进行中）、fulfilled(已成功)和rejected（已失败）。只有异步操作的结果，可以决定当前是哪一种状态，任何其他操作都无法改变这个状态。这也是Promise这个名字的由来，它的英语意思就是“承诺”，表示其他手段无法改变。

2. 一旦状态改变，就不会再变，任何时候都可以得到这个结果。`Promise`对象的状态改变，只有两种可能：从`pending`变为`fulfilled`和从`pending`变为`rejected`。只要这两种情况发生，状态就凝固了，不会再变了，会一直保持这个结果，这时就称为resolved（已定型）。如果改变已经发生了，你再对promise对象添加回调函数，也会立即得到这个结果。

## 基本用法
```
const promise = new Promise((resolve,reject)=>{
    if(/* 异步操作成功 */){
        resolve(value);
    }else{
        reject(error);
    }
});
```
`Promsie`构造函数接受一个函数作为参数，该函数的两个参数分别是`resolve`和`reject`。它们是两个函数，由JavaScript引擎提供，不用自己部署。

`resolve`函数的作用是，将`Promise`对象的状态从“未完成”变成“成功”（即从pending变为resolved），在异步操作成功时调用，并将异步操作的结果，作为参数传递出去；
`reject`函数的作用是，将`Promise`对象的状态从“未完成”变为"失败"（即从pending变为rejected），在异步操作失败时调用，并将异步操作报出的错误，作为参数传递出去。


```
promise.then((value)=>{
    //success
},(error)=>{
    //failure
})
```
`then`方法可以接受两个回调函数作为参数。第一个回调函数是`Promise`对象的状态变为`resolved`时调用，第二个回调函数是`Promise`对象的状态变为`rejected`时调用。其中，第二个函数是可选的，不一定要提供。这两个函数都接受Promise对象传出的值作为参数。
```
function timeout(ms){
    return new Promise((resolve,reject)=>{
        setTimeout(resolve,ms,'done');
    });
}

timeout(100).then((value)=>{
    console.log(value);
})
```

下面是异步加载图片的例子
```
function loadImageAsync(url){
    return new Promise(function(resolve,reject){
        const image = new Image();
        image.onload=function(){
            resolve(image);
        };

        image.onerror=function(){
            reject(new Error(`Could not load image at ${url}`));
        }

        image.src = url;
    })
}
```

下面是一个用`Promise`对象实现的Ajax操作的例子
```
const getJSON = function(url){
    const promise = new Promise(function(resolve,reject){
        const handler = function(){
            if(this.readyState!==4){
                return;
            }
            if(this.state===200){
                resolve(this.response);
            }else{
                reject(new Error(this.statusText));
            }
        };

        const client = new XMLHttpRequest();
        client.open("GET",url);
        client.onreadystatechange = handler;
        client.responseType = "json";
        client.setRequestHeader("Accept","application/json");
        client.send();
    });
    return promise;
};

getJSON("/posts.json").then((json)=>{
    console.log(`Content:${json}`);
},(error)=>{
    console.log(`出错了${error}`);
})

```

### Promise.prototype.then()
`then`方法返回的是一个新的`Promise`实例（注意，不是原来那个Promise实例）。因此可以采用链式写法，即`then`方法后面再调用另一个`then`方法。

### Promise.prototype.catch()
用于指定发生错误时的回调函数。
then方法指定的回调函数，如果运行中抛出错误，也会被catch方法捕获。

如果Promise状态已经变成resolved，再抛出错误时无效的。
```
const promise = new Promise((resolve,reject)=>{
    resolve('OK');
    throw new Error('test');
});
promise.then(value=>{
    console.log(value);
}).catch(error=>{
    console.log(error);
})
//ok
```

### Promise.prototype.finally()
`finally`方法用于指定不管Promise对象最后状态如何，都会执行的操作。
```
promise.then(result=>{
    console.log(result)
}).catch(error)=>{
    console.log(error)
}.finally(()=>{
    console.log('finally');
})
```
`finally`方法的回调函数不接受任何参数，这意味着没有办法知道，前面的 Promise 状态到底是`fulfilled`还是`rejected`。这表明，`finally`方法里面的操作，应该是与状态无关的，不依赖于 Promise 的执行结果。

### Promise.all()
用于将多个Promise实例，包装成一个新的Promise实例。
```
const p = Promise.all([p1,p2,p3]);
```
`Promise.all`方法接受一个数组作为参数，`p1`,`p2`,`p3`都是Promise实例，如果不是，就会先调用`Promise.resolve`方法，将参数转为Promise实例，再进一步处理。
`p`的状态由`p1`、`p2`、`p3`决定，分成两种情况。
（1）只有`p1`、`p2`、`p3`的状态都变成`fulfilled`，p的状态才会变成`fulfilled`，此时`p1`、`p2`、`p3`的返回值组成一个数组，传递给`p`的回调函数。

（2）只要`p1`、`p2`、`p3`之中有一个被`rejected`，`p`的状态就变成`rejected`，此时第一个被`reject`的实例的返回值，会传递给p的回调函数。


下面是一个具体的例子。
```
// 生成一个Promise对象的数组
const promises = [2, 3, 5, 7, 11, 13].map(function (id) {
  return getJSON('/post/' + id + ".json");
});

Promise.all(promises).then(function (posts) {
  // ...
}).catch(function(reason){
  // ...
});
```
上面代码中，`promises`是包含 6 个 Promise 实例的数组，只有这 6 个实例的状态都变成`fulfilled`，或者其中有一个变为`rejected`，才会调用Promise.all方法后面的回调函数。
### Promise.race()
`Promise.race`方法同样是将多个 Promise 实例，包装成一个新的 Promise 实例。

### Promise.resolve()
将现有对象转为 Promise 对象

`Promise.resolve`方法的参数分成四种情况。
* （1）参数是一个 Promise 实例
    如果参数是 Promise 实例，那么Promise.resolve将不做任何修改、原封不动地返回这个实例。
* （2）参数是一个thenable对象
    thenable对象指的是具有then方法的对象
* （3）参数不是具有then方法的对象，或根本就不是对象
    如果参数是一个原始值，或者是一个不具有then方法的对象，则Promise.resolve方法返回一个新的 Promise 对象，状态为resolved。
* （4）不带有任何参数
    Promise.resolve方法允许调用时不带参数，直接返回一个resolved状态的 Promise 对象。


### Promise.reject()
`Promise.reject(reason)`方法也会返回一个新的 Promise 实例，该实例的状态为`rejected`。
