# Zookeeper
# 集群架构
三种类型的节点：
- Leader : 负责读写
- Follower:只负责读，如果有写请求会转发到Leader
- Observer:只回复读，不支持写

# Watcher
通过注册Watcher来实现监听节点的状态变更，特性：
- 主动推送给监听客户端
- watch是一次性的，一次通知后就会被销毁
- watch的通知在节点更新数据之前
- 保持状态更新的顺序和通知的顺序一直

# 消息广播
- Leader节点会针对每一个Follower和Observer 节点维护一个队列
- 在收到一个写请求时，会创建自增生成一个zid，创建一个proposal提案（包含这个zid）可以根据zid的大小来保证
事务的写入顺序。
- Leader节点会把这个提案放到队列中等待发送，所有队列都需要保证一致，才能控制所有节点的事务
一致，每次发送都需要等待队列中的前一个发送结束才能发送
- Follower和Observer节点收到消息之后，会写到本地事务日志，然后ACK Leader节点
- Leader收到超过半数以上的ACK之后，就会在创建一个COMMIT消息，走相同的流程发送
```java
队列:
proposal(zxid=1)
proposal(zxid=2)
commit(zxid=1)
commit(zxid=2)
```
- 最终节点收到消息之后会提交本地事务