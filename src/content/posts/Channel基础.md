---
title: Channel的小细节
published: 2025-04-16
description: Channel的小细节
image: ./cover.jpg
tags: [易错, go]
category: 后端
draft: false
pinned: false
lang: zh-CN      # 仅当文章语言与 config.ts 中的站点语言不同时设置


---

------

# 4.1 什么是CSP？

CSP 是一种并发模型，它主张用“消息传递”代替“共享内存”。Go 的 goroutine 和 channel 就是 CSP 的一个现代实现。在 Go 中，多个 goroutine 不需要竞争共享数据，而是通过 channel 来传递数据，从而减少锁竞争，提高并发安全性

你可以把它想象成：

- 每个人都坐在自己的房间里工作（goroutine）
- 房间之间用纸条传话（channel）
- 没有人抢同一份笔记本（共享内存）

自然就不会打架（锁争用）。

# 4.2 Channel的底层实现原理是怎样的？

Go 的 channel 底层核心是一个结构体：

```
type hchan struct {
    qcount   uint           // 队列中已有元素数量
    dataqsiz uint           // 环形队列容量
    buf      unsafe.Pointer // 环形队列
    sendx    uint           // 下一个发送位置
    recvx    uint           // 下一个接收位置

    recvq    waitq          // 接收阻塞队列（goroutines）
    sendq    waitq          // 发送阻塞队列（goroutines）

    lock     mutex          // 全局互斥锁
}
```

整个底层实现围绕**三个关键部件：**

1. **环形队列（buffer） → 存数据**
2. **发送阻塞队列（sendq） → 存等待发的 goroutine**
3. **接收阻塞队列（recvq） → 存等待收的 goroutine**

Go 的 channel 底层是一个带锁的结构，包含一个可选的环形缓冲区（buf）、两个 goroutine 等待队列（sendq 和 recvq）以及用于并发安全的互斥锁。

发送和接收操作都会根据当前 buffer 和等待队列的情况，决定是直接配对、放入缓冲区还是阻塞挂起。无缓冲 channel 通过直接交换数据实现同步。close 会唤醒所有接收方。整个机制保证了 CSP 模型的安全通信。

## 发送

```md
lock(chan)

如果 recvq 不为空：(可以理解为接收的人不为空)
    - 有 goroutine 在等数据
    - 直接把数据交给等待的接收者
    - 立即唤醒 goroutine（不进缓冲区）

否则如果 buffer 还有空间：
    - 把数据放入环形队列的 sendx 位置
    - sendx++（循环增长）

否则（recvq 空 & buffer 满）：（也就是发送方自己会被挂起，直到有人接收）
    - 发送方 goroutine 挂起
    - goroutine 加入 sendq 队列
    - unlock(chan)
    - 等待被唤醒

unlock(chan)
```

# 4.3 向channel发送数据的过程是怎样的？

**向 channel 发送数据的过程本质是：**

1. **优先唤醒接收者（直接把值交给对方）**
2. **否则尝试写入 buffer**
3. **失败则把发送方挂到 sendq 等待**

Channel 内部用锁保证安全，使用环形缓冲区存储，recvq/sendq 用于阻塞等待。

# 4.4 从Channel读取数据的过程是怎样的？

**接收数据时，channel 会按照优先级做三件事：**

1. **优先读 buffer**
2. **buffer 空，则直接向 sendq 中的发送者拿数据**
3. **否则阻塞当前 goroutine，加入 recvq**

关闭 channel 且 buffer 空时，接收返回零值并且 ok=false。

# 4.5 从一个已关闭Channel仍能读出数据吗？

关闭 channel 只表示不能再发送，但不会影响 buffer 数据的读取。读完 buffer 后再读，会返回零值和 ok=false。

> **channel 的 buffer 是存储空间，并不靠发送者来维持读操作。**

发送者只是往 buffer 放东西。所以关闭后还可以获取数据

# 4.6 Channel在什么情况下会引起内存泄漏？

当 channel 的发送或接收一侧没有对应方、或 channel 不关闭、或 buffer 满导致 goroutine 永远阻塞时，会发生内存泄漏。

# 4.7 关闭Channel会产生异常吗？

**关闭 channel 本身不会产生异常，但以下操作会 panic：**

- 重复关闭
- 向已关闭 channel 发送数据
- 关闭 nil channel

**而从已关闭 channel 读取是安全的，会读到已缓冲的数据，之后读到类型零值和 `ok=f`**

# 4.8 往一个关闭的Channel写入数据会发生什么？

向已关闭的 channel 写入数据会立刻 panic，因为关闭表示发送端生命周期结束，只允许继续接收不允许再发送。这样可以防止数据竞争和逻辑错误。

# 4.9 什么是select？

select 是 Go 的 channel 多路复用机制，用来同时等待多个 channel 的发送或接收。
 如果多个 case 都准备好了，select 会随机选择一个执行；
 如果没有 case 就绪，可以用 default 实现非阻塞，也可以结合 time.After 实现超时控制。

# 4.10 select的执行机制是怎样的？

select 会对所有 case 的 channel 操作进行一次检查：如果有 case 立即就绪，就随机选择一个执行；如果都不就绪且有 default，则执行 default；否则 select 会将当前 goroutine 挂起，加入对应 channel 的等待队列，一旦某个 channel 就绪，就把 goroutine 唤醒并继续执行该 case。

# 4.11 select的实现原理是怎样的？

select 会把所有 case 随机打乱并依次检查是否就绪；如果没有 case 就绪且没有 default，则将当前 goroutine 注册到所有相关 channel 的等待队列中，然后挂起。当任意一个 channel 变得可操作（可读/可写/关闭）时，runtime 唤醒该 goroutine，select 再次检查所有 case，并随机选择一个已就绪的 case 执行。执行完成后退出 select。