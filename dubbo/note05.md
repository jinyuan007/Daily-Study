Dubbo 时间轮

```java
public interface Timer {
    //接受一个TimerTask任务，返回一个Timeout句柄,同时Timeout也是时间轮卡槽中链表上的节点
    Timeout newTimeout(TimerTask task, long delay, TimeUnit unit);
    Set<Timeout> stop();
    boolean isStop();
}
```
# 核心执行逻辑
- HashedWheelTimer会维护:
  - 卡槽列表
  - 指针 + 掩码
  - 任务缓冲队列
  - 取消任务缓冲列表
  - Worker线程：每次循环都会计算下一个指针需要等待多久，然后线程休眠对应时间后唤醒，然后计算得到需要调度的卡槽，再调度卡槽中任务之前会处理两个缓冲队列，然后遍历卡槽链表上的节点依次执行。

- HashedWheelBucket会维护：
  - 链表的首位节点
  - expireTimeouts()方法，执行链表上的任务

- HashedWheelTimeout会维护：
  - 所在卡槽
  - 前驱节点和后继节点
  - 当前任务的周次次数remainingRounds：针对套圈任务的情况

