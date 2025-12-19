# Dubbo 传输层  NIO框架

# ChannelHandler
抽象了在一次通信过程中，能够执行的handler

# EndPoint接口 
一个已经建立通信能力的对端节点

# Channel接口 extent Channel
抽象了一个逻辑的通信

# AbstractPeer  implements Endpoint, ChannelHandler
是EndPoint的下一层抽象，AbstractPeer新增一些EndPoint需要管理的东西，
例如 URL的维护、连接的开关 以及通信中调用的ChannelHandler

# AbstractEndpoint extends AbstractPeer implements Resetable 
AbstractPeer的下一层抽象，新增了EndPoint需要关注的编解码、超时时间、连接时间的维护
同时实现了Resetable，具备了重启的功能

# AbstractServer extends AbstractEndpoint implements RemotingServer
对服务端的抽象，实现了服务端的一些公共逻辑
- localAddress 服务端的本地地址
- bindAddress 绑定的地址
- accepts 最大连接数
- executorRepository 负责管理线程池
- executor 当前Server关联的线程池
抽象了doOpen() doClose()等等方法 子类实现具体逻辑

# NettyServer extend AbstractServer
基于Netty的Server实现。
- doOpen（）会初始化Netty的BootStrap 同时创建两个EventLoopGroup :Boss和Worker
- 注册Netty的ChannelHandler
  - 编码 委托给实际注册的编解码codec类来处理，根据URL来获取的
  - 解码 委托给实际注册的编解码codec类来处理，根据URL来获取的
  - 心跳处理 netty原生支持的一个ChannelHandler
  - 包装原始注册的Dubbo的ChannelHandler逻辑，需要包装心跳处理Handler,多消息处理Handler以及消息派发Handler,其中消息派发Handler的是最核心的，
    Dispatcher实现了SPI接口，不同的实现会创建不同的DispatcherHandler进行消息分发，例如AllChannelHandler会通过URL找到与之关联的线程池，然后把Channel封装成任务提交给线程池去执行，这样实际的业务逻辑不会阻塞Netty的IO线程

# Client同理
