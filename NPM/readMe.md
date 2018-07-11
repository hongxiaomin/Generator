# npm 模块安装机制简介

npm是Node的模块管理器。

## npm install
`npm install`命令用来安装模块到`node_modules`目录。
```
npm install <packageName>
```
安装之前,`npm install`会先检查，`node_modules`目录之中是否已经存在指定模块。如果存在，就不再重新安装了，即使远程仓库已经有了一个新版本，也是如此。

如果你希望，一个模块不管是否安装过，npm都要强制重新安装，可以使用`-f`或`--force`参数。
```
$ npm install <packageName> --force
```
## npm update
如果想更新已安装模块，就要用到`npm update`命令。

```
$ npm update <packageName>
```
它会先到远程仓库查询最新版本，然后查询本地版本。如果本地版本不存在，或者远程版本较新，就会安装。

## registry
`npm update`命令怎么知道每个模块的最新版本呢？
npm 模块仓库提供了一个查询服务，叫做registry。以npmjs.org为例，它的查询服务网址是`https://registry.npmjs.org/`。
这个网址后面跟上模块名，就会得到一个JSON对象，里面是该模块所有版本的信息。比如，访问`https://registry.npmjs.org/react`，就会看到react模块所有版本的信息。

它跟下面命令的效果是一样的。
```
$ npm view react
$ npm info react
$ npm show react
$ npm v react
```

registry网址的模块名后面，还可以跟上版本号或者标签，用来查询某个具体版本的信息。比如，访问https://registry.npmjs.org/react/v0.14.6，就可以看到React的0.14.6版。

返回的JSON对象里面，有一个`dist.tarball`属性，是该版本压缩包的网址。
```
dist:{
    shasum:'2a57c2cf8747b483759ad8de0fa47fb0c5cf5c6a',
    tarball:'http://registry.npmjs.org/react/-/react-0.14.6.tgz'
}
```
到这个网址下载压缩包，在本地解压，就得到了模块的源码。`npm install`和`npm update`命令，都是通过这种方式安装模块的。

## 缓存目录
`npm install`或`npm update`命令，从registry下载压缩包之后，都存放在本地缓存目录。
这个缓存目录，在Linux或Mac默认是用户主目录下`.npm`目录，在Windows默认是`AppData/npm-cache`。通过配置命令，可以查看这个目录的具体位置。
```
$ npm config get cache
$ HOME/.npm
$ npm cache ls
```
你会看到里面存放着大量的模块，存储结构是`{cache}/{name}/{version}`.
每个模块的每个版本，都有一个自己的子目录，里面是代码的压缩包`package.tgz`文件，以及一个描述文件`package/package.json`。

除此之外，还会生成一个`{cache}/{hostname}/{path}/.cache.json`文件。这个文件保存的是，所有版本的信息，以及该模块最近修改的时间和最新一次请求时服务器返回的ETag.

对于一些不是很关键的操作（比如`npm search`或`npm view`）,npm 会先查看`.cache.json`里面的模块最近更新的时间，跟当前时间的差距，是不是在可接受的范围之内。如果是的，就不再向远程仓库发出请求，而是直接返回`.cache.json`的数据。

`.npm`目录保存着大量文件，清空它的命令如下：
```
$ rm -rf ~/.npm/*
或者
$ npm cache clean
```

## 模块的安装过程
1. 发出npm install 命令
2. npm 向registry 查询模块压缩包的网址
3. 下载压缩包，存放在/.npm目录
4. 解压压缩包到当前项目的node_modules目录


