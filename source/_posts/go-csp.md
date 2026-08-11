title: Go并发模型
date: '2026-07-28 18:16:04'
updated: '2026-07-28 18:16:07'
tags:
  - go
categories:
  - 每日一记
---
Go语言的Channel通信机制基于通信顺序进程 CSP（Communicating Sequential Processes） 并发模型。
它的核心哲学是：“不要通过共享内存来通信，而应该通过通信来共享内存”。

Channel 是一种特殊的并发安全队列，遵循 FIFO（先进先出） 规则。它主要分为两种类型：
- 无缓冲通道（Unbuffered Channel）：同步通信。发送方和接收方必须同时准备好，否则会阻塞。
- 有缓冲通道（Buffered Channel）：异步通信。只要缓冲区未满，发送方就不会阻塞；只要缓冲区不空，接收方就不会阻塞。

Channel 在底层是一个环形队列，其核心结构包含以下几个关键部分：
- qcount：当前通道中的元素个数。
- dataqsiz：环形队列的长度（缓冲区大小）。
- buf：指向底层环形队列的指针（仅对有缓冲通道有效）。
- sendq：Channel 的发送操作处理到的位置。
- recvq：Channel 的接收操作处理到的位置。
- lock：互斥锁，保障 Channel 并发操作的安全。

Channel 支持发送（ch <-）、接收（<- ch）和关闭（close）操作，使用场景包括：
- 数据传递：在不同的 Goroutine 间安全地传递数据。
- 信号通知：利用 chan struct{}（空结构体不占用内存）作为信号管道，通知其他 Goroutine 退出或开始执行。
- 超时控制：结合 select 语句和 time.After() 实现超时退出。
- 广播机制：关闭 Channel 可以向所有接收方发送广播信号，所有等待该 Channel 的 Goroutine 将立即解除阻塞。

接收数据：
```go
i <- ch
i, ok <- ch
```
> 可通过ok变量判断通道是否已关闭。

关闭Channel：
当 Channel 是一个空指针或者已经被关闭时，执行关闭操作运行时都会直接崩溃并抛出异常。