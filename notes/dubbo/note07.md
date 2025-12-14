# Netty
## 线程模型

- Channel,但是一个Channel只能运行在一个EventLoop上,运行模型是这样的
```javascript
while(true) {
    //调用JDK底层 selector 阻塞等待I/O有数据
    selector.select();
    //处理每个就绪的selectKey
    //OP_ACCEPT（Boss EventLoop）：如果是建立连接，那就创建对应的channel,绑定
    //OP_READ（Worker EventLoop）：如果是socket读入数据，那就找到对应的channel->pipeline.fireChannelRead,触发入站
    //OP_WRITE（Worker EventLoop）:如果是在用户任务中handle没写完的数据，在这里继续写
    processSelectedKeys();
    //运行用户线程提交到该Loop上的任务,
    runAllTasks();
}
```
- EventLoop(主要介绍的是NioEventLoop):是Chanel运行的的地方，底层其实就是一个循环线程，一个EventLoop可以绑定对个
- EventLoopGroup:多个EventLoop组成一组，在 Netty 服务器端中，会有 BossEventLoopGroup 和 WorkerEventLoopGroup 两个 NioEventLoopGroup。
BossEventLoopGroup只用于处理连接创建，不执行任务和数据读写。WorkerEventLoopGroup主要负责数据读写和任务执行
- Channel:是netty对于网络连接的抽象，封装了建立连接,I/O读写操作
- Inbound/Outbound: 入站（Inbound）事件和出站（Outbound）事件,
- ChannelPipeline&ChannelHandler ：ChannelPipeline 之上可以注册多个 ChannelHandler（ChannelInboundHandler 或 ChannelOutboundHandler）

