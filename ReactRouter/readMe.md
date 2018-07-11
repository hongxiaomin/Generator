# React Router
通过管理URL，实现组件的切换和状态的变化，开发复杂的应用。

## 一、基本用法
React Router安装命令如下：
```
$ npm install -S react-router
```
使用时，路由器`Router`就是React的一个组价
```
import {Router} from 'react-router';
render(<Router/>,document.getElementById('app'));
```
`Router`组件本身只是一个容器，真正的路由要通过`Route`组件定义
```
import {Router,Route,hashHistory} from 'react-router';
render((
    <Router history={hashHistory}>
        <Route path="/" component={APP}/>
    </Router>
),document.getElementById('app'))
```

`Router`组件有一个参数`history`，它的值`hashHistory`表示，路由的切换由URL的hash变化决定，即URL的`#`部分发生变化。
`Route`组件定义了URL路径与组件的对应关系。可以同时使用多个`Route`组件。

## 二、嵌套路由
`Route`组件还可以嵌套。
```
 <Router history={hashHistory}>
    <Route path="/" component={APP}>
        <Route path="/repos" component={Repos}/>
        <Route path="/abuot" component={About}/>
    </Route>
</Router>
```
上面代码中，用户访问`/repos`时，会先加载`App`组件，然后在它的内部再加载`Repos`组件。

子路由也可以不写在`Router`组件里面，单独传入`Router`组件的routes属性。
```
let routes = <Route path="/" component={APP}>
        <Route path="/repos" component={Repos}/>
        <Route path="/abuot" component={About}/>
    </Route>;
    <Router routes={routes} history={browserHistory}/>
```
## 三、path属性
`Route`组件的`path`属性指定路由的匹配规则。这个属性是可以省略的，这样的话，不管路径是否匹配，总是会加载指定组件。

## 四、通配符
`path`属性可以使用通配符
```
<Route path="/hello/:name">
//匹配 /hello/michael
//匹配 /hello/ryan

<Route path="/hello(/:name)">
//匹配 /hello
//匹配 /hello/michael
//匹配 /hello/ryan

<Route path="/files/*.*">
//匹配 /files/hello.jpg
//匹配 /files/hello.html

<Route path="/files/*">
//匹配 /files/
//匹配 /files/a
//匹配 /files/a/b

<Route path="/**/*.jpg">
//匹配 /files/hello.jpg
//匹配 /files/path/to/file.jpg
```
通配符的规则如下：
1. paramName
    paramName匹配URL的一个部分，直到遇到下一个/、?、#为止。这个路径参数可以通过`this.props.params.paramName`取出。
2. ()
    ()表示URL的这个部分是可选的。
3. *
    *匹配任意字符，直到模式里面的下一个字符为止。匹配方式是非贪婪模式。
4. **
    ** 匹配任意字符，直到下一个/、?、#为止。匹配方式是贪婪模式。

path属性也可以使用相对路径（不以/开头），匹配时就会相对于父组件的路径。嵌套路由如果想摆脱这个规则，可以使用绝对路由。

路由匹配规则是从上到下执行，一旦发现匹配，就不再其余的规则。因此带参数的路径一般要写在路由规则的底部。

此外，URL的查询字符串`/foo?bar=baz`，可以用`this.props.location.query.bar`获取。

## 五、IndexRoute组件
```
<Router>
  <Route path="/" component={App}>
    <Route path="accounts" component={Accounts}/>
    <Route path="statements" component={Statements}/>
  </Route>
</Router>
```
上面代码中，访问根路径`/`,不会加载任何子组件。`IndexRoute`就是解决这个问题，显示指定`Home`是跟路由的子组件，即指定默认情况下加载的子组件。
`IndexRoute`组件没有路径参数`path`

## 六、Redirect组件
`<Redirect>`组件用于路由的跳转，即用户访问一个路由，会自动跳转到另一个路由。
```
<Route path="inbox" component={Inbox}>
  {/* 从 /inbox/messages/:id 跳转到 /messages/:id */}
  <Redirect from="messages/:id" to="/messages/:id" />
</Route>
```
现在访问`/inbox/messages/5`，会自动跳转到`/messages/5`。

## 七、IndexRedirect组件
`IndexRedirect`组件用于访问根路由的时候，将用户重定向到某个子组件。
```
<Route path="/" component={App}>
  <IndexRedirect to="/welcome" />
  <Route path="welcome" component={Welcome} />
  <Route path="about" component={About} />
</Route>
```
上面代码中，用户访问根路径时，将自动重定向到子组件`welcome`。

## 八、Link
`Link`组件用于取代`<a>`元素，生成一个链接，允许用户点击后跳转到另一个路由。他基本上就是`<a>`元素的React版本，可以接收Router的状态。
```
render() {
  return <div>
    <ul role="nav">
      <li><Link to="/about">About</Link></li>
      <li><Link to="/repos">Repos</Link></li>
    </ul>
  </div>
}
```
如果希望当前的路由与其他路由有不同样式，这时可以使用`Link`组件的`activeStyle`属性。
```
<Link to="/about" activeStyle={{color: 'red'}}>About</Link>
<Link to="/repos" activeStyle={{color: 'red'}}>Repos</Link>
```
上面代码中，当前页面的链接会红色显示。

另一种做法是，使用`activeClassName`指定当前路由的`Class`。
```
<Link to="/about" activeClassName="active">About</Link>
<Link to="/repos" activeClassName="active">Repos</Link>
```
在`Router`组件之外，导航到路由页面，可以使用浏览器的`History API`.

实际上，`IndexLink`就是对`Link`组件的`onlyActiveOnIndex`属性的包装。

## 九、history属性

`Router`组件的`history`属性，用来监听浏览器地址栏的变化，并将URL解析成一个地址对象，供 React Router 匹配。

`history`属性，一共可以设置三种值。
1. browserHistory
2. hashHistory
3. createMemoryHistory

如果设为`hashHistory`，路由将通过URL的`hash`部分（`#`）切换，URL的形式类似`example.com/#/some/path`。

如果设为`browserHistory`，浏览器的路由就不再通过`Hash`完成了，而显示正常的路径`example.com/some/path`，背后调用的是浏览器的`History API`。

但是，这种情况需要对`服务器改造`。否则用户直接向服务器请求某个子路由，会显示网页找不到的404错误。

如果开发服务器使用的是`webpack-dev-server`，加上`--history-api-fallback`参数就可以了。
```
$ webpack-dev-server --inline --content-base . --history-api-fallback

```
`createMemoryHistory`主要用于服务器渲染。它创建一个内存中的`history`对象，不与浏览器URL互动。
```
const history = createMemoryHistory(location)
```

## 十、 表单处理
`Link`组件用于正常的用户点击跳转，但是有时还需要表单跳转、点击按钮跳转等操作。这些情况怎么跟React Router对接呢？
* 第一种方法是使用`browserHistory.push`
```
import { browserHistory } from 'react-router'

// ...
  handleSubmit(event) {
    event.preventDefault()
    const userName = event.target.elements[0].value
    const repo = event.target.elements[1].value
    const path = `/repos/${userName}/${repo}`
    browserHistory.push(path)
  },
```
* 第二种方法是使用`context`对象。
```
export default React.createClass({

  // ask for `router` from context
  contextTypes: {
    router: React.PropTypes.object
  },

  handleSubmit(event) {
    // ...
    this.context.router.push(path)
  },
})
```

## 十一、 路由的钩子
每个路由都有`Enter`和`Leave`钩子，用户进入或离开该路由时触发。
```
<Route path="about" component={About} />
＜Route path="inbox" component={Inbox}>
  ＜Redirect from="messages/:id" to="/messages/:id" />
</Route>
```
上面的代码中，如果用户离开`/messages/:id`，进入`/about`时，会依次触发以下的钩子。
```
/messages/:id的onLeave
/inbox的onLeave
/about的onEnter
```
下面是一个例子，使用`onEnter`钩子替代`<Redirect>`组件。
```
<Route path="inbox" component={Inbox}>
  <Route
    path="messages/:id"
    onEnter={
      ({params}, replace) => replace(`/messages/${params.id}`)
    } 
  />
</Route>
```
`onEnter`钩子还可以用来做认证。
```
const requireAuth = (nextState, replace) => {
    if (!auth.isAdmin()) {
        // Redirect to Home page if not an Admin
        replace({ pathname: '/' })
    }
}
export const AdminRoutes = () => {
  return (
     <Route path="/admin" component={Admin} onEnter={requireAuth} />
  )
}
```
下面是一个高级应用，当用户离开一个路径的时候，跳出一个提示框，要求用户确认是否离开。
```
const Home = withRouter(
  React.createClass({
    componentDidMount() {
      this.props.router.setRouteLeaveHook(
        this.props.route, 
        this.routerWillLeave
      )
    },

    routerWillLeave(nextLocation) {
      // 返回 false 会继续停留当前页面，
      // 否则，返回一个字符串，会显示给用户，让其自己决定
      if (!this.state.isSaved)
        return '确认要离开？';
    },
  })
)
```
上面代码中，`setRouteLeaveHook`方法为`Leave`钩子指定`routerWillLeave`函数。该方法如果返回`false`，将阻止路由的切换，否则就返回一个字符串，提示用户决定是否要切换。


# React Router 4
RR4 采用单代码仓库模型架构（monorepo），这意味着这个仓库里面有若干个相互独立的包，分别是：
* react-router React Router 核心
* react-router-dom 用于DOM绑定的React Router
* react-router-native 用于React Native的React Router
* react-router-redux React Router和Redux的集成
* react-router-config 静态路由配置的小助手

react-router-dom比react-router多出`<Link> <BrowserRouter>`这样的DOM类组件。
我们只需要引用`react-router-dom`这个包就行了。如果搭配`redux`，还需要使用`react-router-redux`.

## 组件
### <BrowserRouter>
一个使用了HTML5 history API的高阶路由组件，保证UI界面和URL保持同步。此组件拥有以下属性：
* **basename:string**
  作用：为所有位置添加一个基准URL
  使用场景：假如需要把页面部署到服务器的二级目录，可以使用basename设置到此目录。
  ```
  <BrowserRouter basename="/minooo"/>
  ```
* **getUserConfirmation:func**
  作用：导航到此页面前执行的函数，默认使用window.confirm
  使用场景：当需要用户进入页面前执行什么操作时可用，不过一般用到的不多。
  ```
  const getConfirmation = (message,callback)=>{
    const allowTransition = window.confirm(message);
    callback(allowTransition);
  }

  <BrowserRouter getUserConfirmation={getConfirmation('Are you Sure?',yourCallBack)} />
  ```
* **forceRefresh:bool**
  作用：当浏览器不支持HTML5的history API时强制刷新新页面。
  使用场景：同上。
  ```
  const supportsHistory = 'pushState' in window.history
  <BrowserRouter forceRefresh={!supportsHistory} />
  ```
* **keyLength:number**
  作用：设置它里面路由的location.key的长度。默认是6.（key的作用：点击同一个链接时，每次该路由下的location.key都会改变，可以通过key的变化来刷新页面。）
  使用场景：按需设置。
  ```
  <BrowserRouter keyLength={12}>
  ```
* **children:node**
  作用：渲染唯一子元素。
  使用场景：作为一个React组件，天生自带children属性。
### <HashRouter> 
Hash history 不支持`location.key`和`location.state`。另外由于该技术只是用来支持旧版浏览器，因此更推荐使用BrowserRouter。

### <Route>
最基本的职责就是当页面的访问地址与Route上的path匹配时，就渲染出对应的UI界面。

* render methods 分别是：
  * <Route component>
  * <Route render>
  * <Route children>
**每种render method都有不同的应用场景，同一个<Route>应该只使用一种render method，大部分情况下使用component。**

* props分别是：
  * match
  * location
  * history
所有的render method无一例外都将被传入这些props。

* component
只有当访问地址和路由匹配时，一个React Component才会被渲染，此时此组件接受 route props(match,location,history).

当使用`component`时，router将使用`React.createElement`根据给定的component创建一个新的React元素。这意味着如果使用内联函数（inline function）传值给`component`将会产生不必要的重复装载。对于内联渲染（inline rendering），建议使用render prop.
```
<Route path="/user/:username" component={User}/>
const User = ({match})=>{
  return <h1>Hello {match.params.username}</h1>
}
```
* render:func
此方法适用于内联渲染，而且不会产生上文说的重复装载问题。
```
<Route path="/home" render={()=><h1>Home</h1>}>

//包装 组合
const FadingRoute = ({component:Component,...rest})=>(
  <Route {...rest} render={props=>(
    <FadeIn>
      <Component {...props}/>
    </FadeIn>
  )}/>
)

<FadingRoute path="/cool" component={Something}/>
```
* children:func
有时候你可能只想知道访问地址是否被匹配，然后改变下别的东西，而不仅仅是对应的页面。
```
<ul>
  <ListItemLink to="/somewhere" />
  <ListItemLink to="/somewhere-ele" />
</ul>
 
const ListItemLink = ({ to, ...rest }) => (
  <Route path={to} children={({ match }) => (
    <li className={match ? 'active' : ''}>
      <Link to={to} {...rest} />
    </li>
  )}
)
```
* path:string
任何可以被`path-to-regexp`解析的有效URL路径，如果不给path，那么路由将总是匹配。
* exact:bool
如果为true，path为'/one'的路由将不能匹配'/one/two'，反之，亦然。
* strict:bool
对路径末尾斜杠的匹配。如果为true。path为'/one/'将不能匹配'/one'但可以匹配'/one/two'。
**如果要确保路由没有末尾斜杠，那么strict和exact都必须同时为true**

### <Link>
为你的应用提供声明式，无障碍导航。
* to:string
作用：跳转到指定路径。
使用场景：如果只是单纯的跳转就直接用字符串形式的路径。
```
<Link to="/courses">
```
* to:object
作用：携带参数跳转到指定路径。
作用场景：比如你点击的这个链接将要跳转的页面需要展示此链接对应的内容，又比如这是个支付跳转，需要把商品的价格等信息传递过去。
```
<Link to={{
  pathname:'/course',
  search:'?sort=name',
  state:{price:18}
}}>
```
* replace:bool
为true时，点击链接后将使用新地址替换掉上一次访问的地址。
比如：依次访问'/one','/two','/three','four'这四个地址，如果回退，将依次回退至'/three','/two','/one',这符合我们的预期，假如我们把链接'/three'中的replace设置为true时。会依次退至'/three','/one'。

### <NavLink>
这是 <Link> 的特殊版，顾名思义这就是为页面导航准备的。因为导航需要有 “激活状态”。
* activeClassName: string
导航选中激活时候应用的样式名，默认样式名为 `active`
```
<NavLink
  to="/about"
  activeClassName="selected"
>MyBlog</NavLink>
```
* activeStyle: object
如果不想使用样式名就直接写style
```
<NavLink
  to="/about"
  activeStyle={{ color: 'green', fontWeight: 'bold' }}
>MyBlog</NavLink>
```
* exact: bool
若为 true，只有当访问地址严格匹配时激活样式才会应用

* strict: bool
若为 true，只有当访问地址后缀斜杠严格匹配（有或无）时激活样式才会应用

* isActive: func
决定导航是否激活，或者在导航激活时候做点别的事情。不管怎样，它不能决定对应页面是否可以渲染。


### <Switch>
只渲染出第一个与当前访问地址匹配的 <Route> 或 <Redirect>。
另外，<Switch> 对于转场动画也非常适用，因为被渲染的路由和前一个被渲染的路由处于同一个节点位置！

* children: node
<Switch> 下的子节点只能是 <Route> 或 <Redirect> 元素。只有与当前访问地址匹配的第一个子节点才会被渲染。<Route> 元素用它们的 path 属性匹配，<Redirect> 元素使用它们的 from 属性匹配。如果没有对应的 path 或 from，那么它们将匹配任何当前访问地址。

### <Redirect>
<Redirect> 渲染时将导航到一个新地址，这个新地址覆盖在访问历史信息里面的本该访问的那个地址。

* to: string
重定向的 URL 字符串

* to: object
重定向的 location 对象

* push: bool
若为真，重定向操作将会把新地址加入到访问历史记录里面，并且无法回退到前面的页面。

* from: string
需要匹配的将要被重定向路径。

### Prompt
当用户离开当前页面前做出一些提示。
* message: string
  当用户离开当前页面时，设置的提示信息。
  ```
  <Prompt message="确定要离开？" />
  ```
* message: func
当用户离开当前页面时，设置的回掉函数
```
<Prompt message={location => (
  `Are you sue you want to go to ${location.pathname}?` 
)} />
```
* when: bool
通过设置一定条件要决定是否启用 Prompt

## 对象和方法
### history
histoty 是 RR4 的两大重要依赖之一（另一个当然是 React 了），在不同的 javascript 环境中， history 以多种能够行驶实现了对会话（session）历史的管理。

我们会经常使用以下术语：
* "browser history" - history 在 DOM 上的实现，用于支持 HTML5 history API 的浏览器
* "hash history" - history 在 DOM 上的实现，用于旧版浏览器。
* "memory history" - history 在内存上的实现，用于测试或非 DOM 环境（例如 React Native）。

history 对象通常具有以下属性和方法：
* length: number 浏览历史堆栈中的条目数
* action: string 路由跳转到当前页面执行的动作，分为 PUSH, REPLACE, POP
* location: object 当前访问地址信息组成的对象，具有如下属性：
  * pathname: string URL路径
  * search: string URL中的查询字符串
  * hash: string URL的 hash 片段
  * state: string 例如执行 push(path, state) 操作时，`location 的 state 将被提供到堆栈信息里，state 只有在 browser 和 memory history 有效。
* push(path, [state]) 在历史堆栈信息里加入一个新条目。
* replace(path, [state]) 在历史堆栈信息里替换掉当前的条目
* go(n) 将 history 堆栈中的指针向前移动 n。
* goBack() 等同于 go(-1)
* goForward 等同于 go(1)
* block(prompt) 阻止跳转

history 对象是可变的，因为建议从 <Route> 的 prop 里来获取 location，而不是从 history.location 直接获取。这样可以保证 React 在生命周期中的钩子函数正常执行，例如以下代码：
```
class Comp extends React.Component {
  componentWillReceiveProps(nextProps) {
    // locationChanged
    const locationChanged = nextProps.location !== this.props.location
 
    // 错误方式，locationChanged 永远为 false，因为history 是可变的
    const locationChanged = nextProps.history.location !== this.props.history.location
  }
}
```
### location
location 是指你当前的位置，将要去的位置，或是之前所在的位置
```
{
  key: 'sdfad1'
  pathname: '/about',
  search: '?name=minooo'
  hash: '#sdfas',
  state: {
    price: 123
  }
}
```
在以下情境中可以获取 location 对象
* 在 `Route component` 中，以 this.props.location 获取
* 在 `Route render` 中，以 ({location}) => () 方式获取
* 在 `Route children` 中，以 ({location}) => () 方式获取
* 在 `withRouter` 中，以 this.props.location 的方式获取
location 对象不会发生改变，因此可以在生命周期的回调函数中使用 location 对象来查看当前页面的访问地址是否发生改变。

### match
match 对象包含了 <Route path> 如何与 URL 匹配的信息，具有以下属性：
* params: object 路径参数，通过解析 URL 中的动态部分获得键值对
* isExact: bool 为 true 时，整个 URL 都需要匹配
* path: string 用来匹配的路径模式，用于创建嵌套的 <Route>
* url: string URL 匹配的部分，用于嵌套的 <Link>
