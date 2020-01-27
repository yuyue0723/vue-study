八、谈谈你对MVC、MVP和MVVM的理解
1. MVC模式
MVC分为Model(模型)，View(视图)，Controller(控制器)三个模块。View(视图层)完成前端的数据展示，Controller(控制层)是对数据的接收和触发事件的接收和传递，Model(模型层)则是对数据的存储和处理，再传递给视图层响应或者展示。
+ 优点
  - 耦合性低
  - 重用性高
  - 可维护性高
+ 缺点
  - 没有明确的定义
  - 不适合小型，中等规模的应用程序
  - 增加系统结构和实现的复杂性
  - 视图和控制器间的过于紧密的连接
  - 视图对模型数据的低效率访问

2. MVP模式
MVP分为Model(模型)，View(视图)，Presenter(表示器)三部分组成。MVP模式主要是针对Android的MVC模式的升级版本，MVP与MVC最不同的一点是M与V是不直接关联的，也就是Model与View不存在直接关系，这两者之间间隔着的是Presenter层，其负责调控View与Model之间的间接交互。
+ 优点
  - 降低耦合度
  - 模块职责划分明显
  - 利于测试驱动开发
  - 代码复用
  - 隐藏数据
  - 代码灵活性
+ 缺点
  - 视图的渲染放在Presenter中，所以视图和Presenter的交互会过于频繁。如果Presenter过多地渲染视图，往往会使得它与特定的视图的联系过于紧密。

3.MVVM模式
MVVM分为Model(数据层)，ViewController/View(展示层)，ViewModel(数据模型)。MVVM模式主要是减轻Controller层或者View层的压力，实现更加清晰化代码。通过对ViewModel层的封装：封装业务逻辑处理，封装网络处理，封装数据缓存等，让逻辑处理分离出来，并且不需要处理Model数据，使得Controller层或者View层结构简单，条理清晰。

+ 优点
  - 双向绑定时，当Model变化时，ViewModel会自动更新，View也会自动变化。
  - View的功能进一步的强化，具有控制的部分功能。
  - 控制器的功能大部分移动到View上处理，大大的对控制器进行了瘦身。
+ 缺点
  - 数据绑定使得Bug很难被调试
  - 数据双向绑定不利于代码重用
  - 大的模块，model很大，不利于内存的释放。

MVVM与MVP区别：MVVM模式将Presenter改名为View Model，基本上与MVP模式完全一致，唯一的区别是它采用双向数据绑定：View的变动，自动反应在View Model，反之亦然，这样开发者就不用处理接收事件和View更新的工作，框架已经帮你做好了。
