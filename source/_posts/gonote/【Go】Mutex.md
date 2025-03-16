---
title: 【Go】锁
toc: true
date: 2025-03-03 00:00:00
tags: 
  - go
categories: 
  - Golang



---

点击阅读更多查看文章内容<!--more-->

## 基本原语 

Go 语言在 [`sync`](https://golang.org/pkg/sync/) 包中提供了用于同步的一些基本原语，包括常见的 [`sync.Mutex`](https://draveness.me/golang/tree/sync.Mutex)、[`sync.RWMutex`](https://draveness.me/golang/tree/sync.RWMutex)、[`sync.WaitGroup`](https://draveness.me/golang/tree/sync.WaitGroup)、[`sync.Once`](https://draveness.me/golang/tree/sync.Once) 和 [`sync.Cond`](https://draveness.me/golang/tree/sync.Cond)：

这些基本原语提供了较为基础的同步功能，但是它们是一种相对原始的同步机制，在多数情况下，我们都应该使用抽象层级更高的 Channel 实现同步。

---

## Mutex

Go 语言的 [`sync.Mutex`](https://draveness.me/golang/tree/sync.Mutex) 由两个字段 `state` 和 `sema` 组成。其中 `state` 表示当前互斥锁的状态，而 `sema` 是用于控制锁状态的信号量。

```go
type Mutex struct {
	state int32
	sema  uint32
}
```

上述两个加起来只占 8 字节空间的结构体表示了 Go 语言中的互斥锁。

互斥锁的状态比较复杂，如下图所示，最低三位分别表示 `mutexLocked`、`mutexWoken` 和 `mutexStarving`，剩下的位置用来表示当前有多少个 Goroutine 在等待互斥锁的释放：

![golang-mutex-state](https://img.draveness.me/2020-01-23-15797104328010-golang-mutex-state.png)

在默认情况下，互斥锁的所有状态位都是 0，`int32` 中的不同位分别表示了不同的状态：

- `mutexLocked` — 表示互斥锁的锁定状态；
- `mutexWoken` — 表示从正常模式被从唤醒；
- `mutexStarving` — 当前的互斥锁进入饥饿状态；
- `waitersCount` — 当前互斥锁上等待的 Goroutine 个数；

## 正常模式和饥饿模式 

**正常模式**

1. 竞争激烈时性能优先：
   - 在正常模式下，锁的获取是非公平的，新到达的 goroutine 可能会直接获取锁，而不需要等待。
2. 自旋等待：
   - 当锁被占用时，新到达的 goroutine 会自旋等待一段时间，尝试直接获取锁，而不是立即进入休眠状态。
3. 适用场景：
   - 适用于竞争不激烈或锁持有时间较短的场景，可以最大化性能。

**饥饿模式**

- 公平性优先：
  - 在饥饿模式下，锁的获取是公平的，等待时间最长的 goroutine 会优先获取锁。
- 防止长时间等待：
  - 当某个 goroutine 等待锁的时间超过一定阈值（默认为 1 毫秒），锁会进入饥饿模式。
- 适用场景：
  - 适用于竞争激烈或锁持有时间较长的场景，可以防止某些 goroutine 长时间等