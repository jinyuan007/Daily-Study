# Dubbo 中的Registry

# 类结构
![img.png](img.png)
- RegistryService: 抽象了注册中心服务的能力
  - register() 方法和 unregister() 方法分别表示注册和取消注册一个 URL。
  - subscribe() 方法和 unsubscribe() 方法分别表示订阅和取消订阅一个 URL。订阅成功之后，当订阅的数据发生变化时，注册中心会主动通知第二个参数指定的 NotifyListener 对象，NotifyListener 接口中定义的 notify() 方法就是用来接收该通知的。
  - lookup() 方法能够查询符合条件的注册数据，它与 subscribe() 方法有一定的区别，subscribe() 方法采用的是 push 模式，lookup() 方法采用的是 pull 模式。

- Node:在 Dubbo 中，一般使用 Node 这个接口来抽象节点的概念。Node不仅可以表示 Provider 和 Consumer 节点，还可以表示注册中心节点
  Node 是 Dubbo 对“可运行、可判断状态、可销毁对象”的统一生命周期抽象，是框架进行资源管理和优雅关闭的基础接口。
  - getUrl() 方法返回表示当前节点的 URL；
  - isAvailable() 检测当前节点是否可用；
  - destroy() 方法负责销毁当前节点并释放底层资源;

- Registry:Dubbo本地的注册中心客户端，封装了和远程注册中心的操作继承了 RegistryService 接口和 Node 接口， ，它表示的就是一个拥有注册中心能力的节点. 

- RegistryFactory:创建Registry对象的工厂类,官方提供一个抽象类AbstractRegistryFactory,该抽象类具备所有Registry的维护，同时在创建一个Registry的时候会修改URL的值
，去掉一些和连接、唯一性无关的参数，新增一个URL的参数，例如在PATH上新增RegistryService标识这个URL是一个注册中心服务的URL

- AbstractRegistry：官方提供的Registry的抽象实现类，
  - 为了减轻注册中心组件的压力，AbstractRegistry 会把当前节点订阅的 URL 信息缓存到本地的 Properties 文件中
  - 在Provider暴露的接口有变化时，会通知到对应的订阅方
- FailbackRegistry：继承AbstractRegistry，提供重试能力 模板方法，覆盖了 AbstractRegistry 中 register()/unregister()、subscribe()/unsubscribe() 以及 notify() 这五个核心方法，结合前面介绍的时间轮，实现失败重试的能力；真正与服务发现组件的交互能力则是放到了 doRegister()/doUnregister()、doSubscribe()/doUnsubscribe() 以及 doNotify() 这五个抽象方法中，由具体子类实现。这是典型的模板方法模式的应用。
