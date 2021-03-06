# MVC 模式
* 如何设计一个程序的结构，这是一门专门的学问，叫做“架构模式”（architectural pattern）,属于编程的方法论。MVC模式就是架构模式的一种。
* **MVC是三个单词的首字母缩写，他们是Model（模型）、View（视图）和Controller（控制）。**这个模式认为，程序不论简单或复杂，从结构上看，都可以分成三层。
    1. 最上面的一层，是直接面向最终用户的“视图层”（View）。它是提供给用户的操作界面，是程序的外壳。
    2. 最底下的一层，是核心的“数据层”（Model），也就是程序需要操作的数据或信息。
    3. 中间的一层，就是“控制层”（Controller），它负责根据用户从“视图层”输入的指令，选取“数据层”中的数据，然后对其进行相应的操作，产生最终的结果。
    这三层是紧密联系在一起的，但又是互相独立的，每一层内部的变化不影响其他层。每一层对外都提供接口（Interface），供上面一层调用。这样一来，软件就可以实现模块化，修改外观或者变更数据都不用修改其他层，大大方便了维护和升级。
