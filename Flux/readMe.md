## Flux 是什么？
简单说，Flux是一种架构思想，专门解决软件的结构问题。

#### Flux将一个应用分成四个部分
* View： 视图层
* Action（动作）：视图层发出的消息（比如mouse click）
* Dispatcher(派发器)：用来接收Actions、执行回调函数
* Store（数据层）：用来存放应用的状态，一旦发生变动，就提醒Views要更新页面

上面的过程中，数据总是“单向流动”，任何相邻的部分都不会发生数据的“双向流动”。这保证了流程的清晰。

### View（第一部分）
### Action 
每个Action都是一个对象，包含一个*actionTye*属性（说明动作的类型）和一些其他属性（用来传递数据）。
### Dispatcher
Dispatcher 的作用是将Action派发到Store。你可以把它看作一个路由器，负责在View和Store之间，建立Action的正确传递路线。*注意，Dispatcher只能有一个，而且是全局的。*
**Dispatcher只是用来派发Action，不应该有其他逻辑。**

### Store
Store保存整个应用的状态。


