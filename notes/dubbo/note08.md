# DemoRpc框架

Client发起请求核心思路：


- 用户通过Consumer获取到一个服务实例,同时开启服务发现
- 通过动态代理代理该服务接口，内部会构造一个RPC的RpcClient,client内部封装了netty Client.同时和远程
服务建立连接，该代理对象会维护该连接。
- 后续业务调用该对象的方法时，会通过连接发起远程调用,同时会维护一个请求的id，然后封装一个Response,构造一个全局Map
来映射id和Response.Response中会包含一个Promise对象。后续调用方就通过Promise.get()来阻塞。同时这个请求的id也会作为
远程调用的header的一个参数。
- 同时代理对象还需要关注registry的信息，比如建立连接的时候，应该从注册中心获取实际地址。远程服务有变动需要调整等等
- 后续服务端返回后,netty的EventLoop会触发调用对应的channel,然后找到对应的入站handler执行，其中我们注册的一个handler
会反序列化请求，然后拿到header里的id，然后找到对应的Response，写入返回值,同时Promise.setSuccess()
- 后续代理对象的阻塞就结束，返回执行结果



Service处理请求核心思路：
-  构造一个RpcService，内部封装了nettyClient，注册一个自定义的Handler，同时需要把该服务注册到注册中心。
-  当一个请求达到服务端，会通过EventLoop走到自定义的Handler。这handler会获取请求，然后提交异步执行
-  同时会把ChannelHandlerContext一并传给业务方，业务方在执行完之后会通过ChannelHandlerContext.ctx.writeAndFlush
写入执行的结果，因为是外部线程写入的，所有会被EventLoop封装为一个task放到队列等待执行。执行完之后数据被写入
socket。
- 执行结束