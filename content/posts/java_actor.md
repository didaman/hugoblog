+++
date = '2025-09-05T16:25:45+08:00'
draft = false
title = 'Java_actor'
+++

# Java并发编程-Actor模式

Actor 模式是一种并发编程模型，它将每个并发实体视为一个独立的 Actor。

- 每个 Actor 都有自己的状态和行为，并且通过异步消息传递进行通信。
- Actor 之间不会共享状态，避免了传统并发编程中因共享状态带来的锁竞争、死锁等问题。

关键组件：

- **消息队列**：每个 Actor 都有一个自己的消息队列，用于接收其他 Actor 发送过来的消息。
- **行为逻辑**：Actor 根据接收到的消息执行相应的行为逻辑，向其他 Actor 发送消息，这一过程是非阻塞的。
- **创建与生命周期**：Actor 可以创建新的 Actor，新创建的 Actor 同样拥有自己的消息队列和行为逻辑。

## 实现Actor模式

简单Actor实现  SimpleActor.java

```java
package actor;

import java.util.concurrent.*;

public abstract class SimpleActor {
    // 线程安全的队列
    private final BlockingQueue<Object> mailbox = new LinkedBlockingQueue<>();
    private final ExecutorService executor = Executors.newSingleThreadExecutor();
    private volatile boolean running = true;
    // 通过构造函数，创建Actor或子类时自动调用。
    public SimpleActor() {
        // 异步提交任务。 创建一个线程，并执行messageLoop方法。
        executor.submit(this::messageLoop);
    }

    // 发送消息。 tell 方法将消息放入邮箱队列中，实现异步发送
    public void tell(Object message) {
        if (running) {
            mailbox.offer(message); // 非阻塞地将消息加入队列
        }
    }

    // 消息处理循环
    private void messageLoop() {
        while (running) {
            try {
                Object message = mailbox.poll(1, TimeUnit.SECONDS);
                if (message != null) {
                    receive(message);
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
                break;
            }
        }
    }

    // 子类实现具体的消息处理逻辑
    protected abstract void receive(Object message);

    // 停止Actor
    public void stop() {
        running = false;
        executor.shutdown();
    }
}
```

计数器Actor示例 CounterActor.java

```java
package actor;

import java.util.concurrent.CompletableFuture;

public class CounterActor extends SimpleActor {
    private int count = 0;

    // 消息类型
    public static class Increment {}
    public static class Decrement {}
    public static class GetCount {
        private final CompletableFuture<Integer> future;

        public GetCount(CompletableFuture<Integer> future) {
            this.future = future;
        }

        public CompletableFuture<Integer> getFuture() {
            return future;
        }
    }

    @Override
    protected void receive(Object message) {
        if (message instanceof Increment) {
            count++;
            System.out.println("计数器递增，当前值: " + count);
        } else if (message instanceof Decrement) {
            count--;
            System.out.println("计数器递减，当前值: " + count);
        } else if (message instanceof GetCount) {
            GetCount getCount = (GetCount) message;
            getCount.getFuture().complete(count);
        }
    }

    // 便捷方法
    public void increment() {
        tell(new Increment());
    }

    public void decrement() {
        tell(new Decrement());
    }

    public CompletableFuture<Integer> getCount() {
        CompletableFuture<Integer> future = new CompletableFuture<>();
        tell(new GetCount(future));
        return future;
    }
}
```



使用示例 ActorExample.java

```java
package actor;

import java.util.concurrent.*;

public class ActorExample {
    public static void main(String[] args) throws Exception {
        CounterActor counter = new CounterActor();

        // 并发操作计数器
        ExecutorService executor = Executors.newFixedThreadPool(10);

        // 提交多个递增任务
        for (int i = 0; i < 100; i++) {
            executor.submit(counter::increment);
        }

        // 提交多个递减任务
        for (int i = 0; i < 50; i++) {
            executor.submit(counter::decrement);
        }

        // 等待一段时间让所有消息处理完成
        Thread.sleep(1000);

        // 获取最终计数
        CompletableFuture<Integer> finalCount = counter.getCount();
        System.out.println("最终计数: " + finalCount.get());

        // 清理资源
        counter.stop();
        executor.shutdown();
    }
}
```



# 应用场景

## 分布式系统

**负载均衡**

在分布式系统中，不同的服务器可以看作是不同的 Actor。Actor模式还支持动态扩缩容，当系统负载增加时可以动态创建新的Worker Actor，负载降低时可以停止部分Actor释放资源。同时，Actor的监督机制能够提供良好的容错能力，当某个Worker Actor出现故障时，监督者可以重启它或者将流量转移到其他健康的Actor上。

**数据一致性维护**

多个数据节点 Actor 可以通过消息传递来同步数据。比如在分布式数据库中，当一个节点的数据发生变化时，该节点 Actor 向其他节点 Actor 发送更新消息，其他节点 Actor 接收到消息后进行相应的数据更新，从而维护数据的一致性。

## 事件驱动系统

**游戏开发**

在游戏中，每个游戏对象（如角色、道具等）可以看作是一个 Actor。

例如，当一个角色 Actor 接收到 “受到攻击” 的消息时，它会根据自身的状态（如生命值、防御力等）执行相应的行为，如减少生命值、播放受击动画等。同时，它还可能向其他相关的 Actor（如攻击者 Actor、队友 Actor 等）发送消息，通知它们发生了相应的事件。

**图形用户界面（GUI）开发**

在 GUI 应用中，每个组件（如按钮、文本框等）可以看作是一个 Actor。当用户点击按钮时，按钮 Actor 接收到点击消息，然后执行相应的操作，如触发业务逻辑、更新界面等。并且，按钮 Actor 可以向其他相关的组件 Actor 发送消息，实现组件之间的交互。

## 高并发计算

**并行计算任务**

在进行大规模数据处理或复杂计算时，可以将任务分解为多个子任务，每个子任务由一个 Actor 来处理。例如，在大数据分析中，一个数据处理 Actor 可以接收数据块消息，对其进行特定的计算操作（如统计、过滤等），然后将处理结果发送给其他 Actor 进行进一步的汇总或分析。这样可以充分利用多核 CPU 的优势，提高计算效率。

# 并发编程框架

**Akka**是一个基于Actor模型的并发框架，作用是简化高并发、快速构建分布式应用。

比如负载均衡场景可以设计架构：

1. 创建一个LoadBalancerActor作为主控节点，负责接收客户端请求；
2. 创建多个WorkerActor代表后端服务实例，每个Worker维护自己的连接池和状态信息；
3. LoadBalancerActor根据负载均衡算法（轮询、权重、最少连接等）选择合适的WorkerActor来处理请求。

```java
import akka.actor.AbstractActor;
import akka.actor.ActorRef;
import akka.actor.Props;

public class HelloActor extends AbstractActor {
    @Override
    public Receive createReceive() {
        return receiveBuilder()
              .match(String.class, message -> {
                    System.out.println("Received message: " + message);
                    // 可以在这里向其他Actor发送消息
                })
              .build();
    }

    public static Props props() {
        return Props.create(HelloActor.class);
    }
}
```

