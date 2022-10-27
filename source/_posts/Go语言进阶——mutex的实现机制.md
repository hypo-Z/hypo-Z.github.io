---
title: Go语言进阶——mutex的实现机制
author: hypo
img: medias/featureimages/70.jpeg
top: false
cover: false
toc: true
mathjax: false
date: 2022-10-27 10:56:10
coverImg:
password:
summary: 互斥锁是并发程序中对共享资源进行访问控制的主要手段，对此Go语言提供了非常简单易用的Mutex，Mutex为一结构体类型，对外暴露两个方法Lock()和Unlock()分别用于加锁和解锁。
categories: Golang
tags:
- Golang
- 进阶
- 笔记
---
# mutex的实现机制

> 互斥锁是并发程序中对共享资源进行访问控制的主要手段，对此Go语言提供了非常简单易用的Mutex，Mutex为一结构体类型，对外暴露两个方法Lock()和Unlock()分别用于加锁和解锁。
>
> Mutex使用起来非常方便，但其内部实现却复杂得多，这包括Mutex的几种状态。另外，我们也想探究一下Mutex重复解锁引起panic的原因。
>
> 按照惯例，本节内容从源码入手，提取出实现原理，又不会过分纠结于实现细节。

## Mutex数据结构

源码包`src/sync/mutex.go:Mutex`定义了互斥锁的数据结构：

```go
type Mutex struct {
    state int32
    sema  uint32
}
```

- Mutex.state表示互斥锁的状态，比如是否被锁定等。
- Mutex.sema表示信号量，协程阻塞等待该信号量，解锁的协程释放信号量从而唤醒等待信号量的协程。

我们看到Mutex.state是32位的整型变量，内部实现时把该变量分成四份，用于记录Mutex的四种状态。

下图展示Mutex的内存布局：

![null](https://topgoer.cn/uploads/gozhuanjia/images/m_45a91868c2c9d5dc2617e9fda0e46049_r.png)

- Locked: 表示该Mutex是否已被锁定，0：没有锁定 1：已被锁定。
- Woken: 表示是否有协程已被唤醒，0：没有协程唤醒 1：已有协程唤醒，正在加锁过程中。
- Starving：表示该Mutex是否处于饥饿状态，0：没有饥饿 1：饥饿状态，说明有协程阻塞了超过1ms。
- Waiter: 表示阻塞等待锁的协程个数，协程解锁时根据此值来判断是否需要释放信号量。

协程之间抢锁实际上是抢给Locked赋值的权利，能给Locked域置1，就说明抢锁成功。抢不到的话就阻塞等待Mutex.sema信号量，一旦持有锁的协程解锁，等待的协程会依次被唤醒。

Woken和Starving主要用于控制协程间的抢锁过程，后面再进行了解。

## 加解锁过程

### 简单加锁

假定当前只有一个协程在加锁，没有其他协程干扰，那么过程如下图所示：

![null](https://topgoer.cn/uploads/gozhuanjia/images/m_4b43555a5440890c948680aab58e982f_r.png)

加锁过程会去判断Locked标志位是否为0，如果是0则把Locked位置1，代表加锁成功。从上图可见，加锁成功后，只是Locked位置1，其他状态位没发生变化。

### 加锁被阻塞

假定加锁时，锁已被其他协程占用了，此时加锁过程如下图所示：

![null](https://topgoer.cn/uploads/gozhuanjia/images/m_7009a47d6e8acb7e7b9c421ad1fece22_r.png)

从上图可看到，当协程B对一个已被占用的锁再次加锁时，Waiter计数器增加了1，此时协程B将被阻塞，直到Locked值变为0后才会被唤醒。

### 简单解锁

假定解锁时，没有其他协程阻塞，此时解锁过程如下图所示：

![null](https://topgoer.cn/uploads/gozhuanjia/images/m_6f4885510e5659f0615f17e6a5b89d2f_r.png)

由于没有其他协程阻塞等待加锁，所以此时解锁时只需要把Locked位置为0即可，不需要释放信号量。

### 解锁并唤醒协程

假定解锁时，有1个或多个协程阻塞，此时解锁过程如下图所示：

![null](https://topgoer.cn/uploads/gozhuanjia/images/m_f45d385c3f7bacc272878bbfd6d48182_r.png)

协程A解锁过程分为两个步骤，一是把Locked位置0，二是查看到Waiter>0，所以释放一个信号量，唤醒一个阻塞的协程，被唤醒的协程B把Locked位置1，于是协程B获得锁。

## 自旋过程

加锁时，如果当前Locked位为1，说明该锁当前由其他协程持有，尝试加锁的协程并不是马上转入阻塞，而是会持续的探测Locked位是否变为0，这个过程即为自旋过程。

自旋时间很短，但如果在自旋过程中发现锁已被释放，那么协程可以立即获取锁。此时即便有协程被唤醒也无法获取锁，只能再次阻塞。

自旋的好处是，当加锁失败时不必立即转入阻塞，有一定机会获取到锁，这样可以避免协程的切换。

### 什么是自旋

自旋对应于CPU的”PAUSE”指令，CPU对该指令什么都不做，相当于CPU空转，对程序而言相当于sleep了一小段时间，时间非常短，当前实现是30个时钟周期。

自旋过程中会持续探测Locked是否变为0，连续两次探测间隔就是执行这些PAUSE指令，它不同于sleep，不需要将协程转为睡眠状态。

### 自旋条件

加锁时程序会自动判断是否可以自旋，无限制的自旋将会给CPU带来巨大压力，所以判断是否可以自旋就很重要了。

自旋必须满足以下所有条件：

- 自旋次数要足够小，通常为4，即自旋最多4次
- CPU核数要大于1，否则自旋没有意义，因为此时不可能有其他协程释放锁
- 协程调度机制中的Process数量要大于1，比如使用GOMAXPROCS()将处理器设置为1就不能启用自旋
- 协程调度机制中的可运行队列必须为空，否则会延迟协程调度

可见，自旋的条件是很苛刻的，总而言之就是不忙的时候才会启用自旋。

### 自旋的优势

自旋的优势是更充分的利用CPU，尽量避免协程切换。因为当前申请加锁的协程拥有CPU，如果经过短时间的自旋可以获得锁，当前协程可以继续运行，不必进入阻塞状态。

### 自旋的问题

如果自旋过程中获得锁，那么之前被阻塞的协程将无法获得锁，如果加锁的协程特别多，每次都通过自旋获得锁，那么之前被阻塞的进程将很难获得锁，从而进入饥饿状态。

为了避免协程长时间无法获取锁，自1.8版本以来增加了一个状态，即Mutex的Starving状态。这个状态下不会自旋，一旦有协程释放锁，那么一定会唤醒一个协程并成功加锁。

## Mutex模式

前面分析加锁和解锁过程中只关注了Waiter和Locked位的变化，现在我们看一下Starving位的作用。

每个Mutex都有两个模式，称为Normal和Starving。下面分别说明这两个模式。

### normal模式

默认情况下，Mutex的模式为normal。

该模式下，协程如果加锁不成功不会立即转入阻塞排队，而是判断是否满足自旋的条件，如果满足则会启动自旋过程，尝试抢锁。

### starvation模式

自旋过程中能抢到锁，一定意味着同一时刻有协程释放了锁，我们知道释放锁时如果发现有阻塞等待的协程，还会释放一个信号量来唤醒一个等待协程，被唤醒的协程得到CPU后开始运行，此时发现锁已被抢占了，自己只好再次阻塞，不过阻塞前会判断自上次阻塞到本次阻塞经过了多长时间，如果超过1ms的话，会将Mutex标记为”饥饿”模式，然后再阻塞。

处于饥饿模式下，不会启动自旋过程，也即一旦有协程释放了锁，那么一定会唤醒协程，被唤醒的协程将会成功获取锁，同时也会把等待计数减1。

## Woken状态

Woken状态用于加锁和解锁过程的通信，举个例子，同一时刻，两个协程一个在加锁，一个在解锁，在加锁的协程可能在自旋过程中，此时把Woken标记为1，用于通知解锁协程不必释放信号量了，好比在说：你只管解锁好了，不必释放信号量，我马上就拿到锁了。

## 源码实现（源码包`src/sync/mutex.go`）

### 加锁实现（***重点***）

1. 首先如果当前锁处于初始化状态就直接用 CAS 方法尝试获取锁，这是**_ Fast Path_**

2. 如果失败就进入**_ Slow Path_**

    1. 会首先判断当前能不能进入自旋状态，如果可以就进入自旋，最多自旋 4 次

    2. 自旋完成之后，就会去计算当前的锁的状态

    3. 然后尝试通过 CAS 获取锁

    4. 如果没有获取到就调用 `runtime_SemacquireMutex` 方法休眠当前 goroutine 并且尝试获取信号量

    5. goroutine 被唤醒之后会先判断当前是否处在饥饿状态，（如果当前 goroutine 超过 1ms 都没有获取到锁就会进饥饿模式） 1. 如果处在饥饿状态就会获得互斥锁，如果等待队列中只存在当前 Goroutine，互斥锁还会从饥饿模式中退出 1. 如果不在，就会设置唤醒和饥饿标记、重置迭代次数并重新执行获取锁的循环

       CAS 方法在这里指的是 `atomic.CompareAndSwapInt32(addr, old, new) bool` 方法，这个方法会先比较传入的地址的值是否是 old，如果是的话就尝试赋新值，如果不是的话就直接返回 false，返回 true 时表示赋值成功 饥饿模式是 Go 1.9 版本之后引入的优化，用于解决公平性的问题[10]

回味一下上面看到的流程图，我们来看看互斥锁是如何加锁的

```go
func (m *Mutex) Lock() {
	// Fast path: grab unlocked mutex.
	if atomic.CompareAndSwapInt32(&m.state, 0, mutexLocked) {
		return
	}
	// Slow path (outlined so that the fast path can be inlined)
	m.lockSlow()
}
```

- 当我们调用 `Lock` 方法的时候，会先尝试走 Fast Path，也就是如果当前互斥锁如果处于未加锁的状态，尝试加锁，只要加锁成功就直接返回
- 否则的话就进入 slow path

```go
func (m *Mutex) lockSlow() {
	var waitStartTime int64 // 等待时间
	starving := false // 是否处于饥饿状态
	awoke := false // 是否处于唤醒状态
	iter := 0 // 自旋迭代次数
	old := m.state
	for {
		// Don't spin in starvation mode, ownership is handed off to waiters
		// so we won't be able to acquire the mutex anyway.
		if old&(mutexLocked|mutexStarving) == mutexLocked && runtime_canSpin(iter) {
			// Active spinning makes sense.
			// Try to set mutexWoken flag to inform Unlock
			// to not wake other blocked goroutines.
			if !awoke && old&mutexWoken == 0 && old>>mutexWaiterShift != 0 &&
				atomic.CompareAndSwapInt32(&m.state, old, old|mutexWoken) {
				awoke = true
			}
			runtime_doSpin()
			iter++
			old = m.state
			continue
		}
```

在 `lockSlow` 方法中我们可以看到，有一个大的 for 循环，不断的尝试去获取互斥锁，在循环的内部，第一步就是判断能否自旋状态。
进入自旋状态的判断比较苛刻，具体需要满足什么条件呢？ `runtime_canSpin` 源码见下方

- 当前互斥锁的状态是非饥饿状态，并且已经被锁定了
- 自旋次数不超过 4 次
- cpu 个数大于一，必须要是多核 cpu
- 当前正在执行当中，并且队列空闲的 p 的个数大于等于一

```go
// Active spinning for sync.Mutex.
//go:linkname sync_runtime_canSpin sync.runtime_canSpin
//go:nosplit
func sync_runtime_canSpin(i int) bool {
	if i >= active_spin || ncpu <= 1 || gomaxprocs <= int32(sched.npidle+sched.nmspinning)+1 {
		return false
	}
	if p := getg().m.p.ptr(); !runqempty(p) {
		return false
	}
	return true
}
```

如果可以进入自旋状态之后就会调用 `runtime_doSpin` 方法进入自旋， `doSpin` 方法会调用 `procyield(30)` 执行三十次 `PAUSE` 指令

```shell
TEXT runtime·procyield(SB),NOSPLIT,$0-0
	MOVL	cycles+0(FP), AX
again:
	PAUSE
	SUBL	$1, AX
	JNZ	again
	RET
```

为什么使用 PAUSE 指令呢？ PAUSE 指令会告诉 CPU 我当前处于处于自旋状态，这时候 CPU 会针对性的做一些优化，并且在执行这个指令的时候 CPU 会降低自己的功耗，减少能源消耗

```go
if !awoke && old&mutexWoken == 0 && old>>mutexWaiterShift != 0 &&
	atomic.CompareAndSwapInt32(&m.state, old, old|mutexWoken) {
	awoke = true
}
```

在自旋的过程中会尝试设置 `mutexWoken` 来通知解锁，从而避免唤醒其他已经休眠的 `goroutine` 在自旋模式下，当前的 `goroutine` 就能更快的获取到锁

```go
new := old
// Don't try to acquire starving mutex, new arriving goroutines must queue.
if old&mutexStarving == 0 {
	new |= mutexLocked
}
if old&(mutexLocked|mutexStarving) != 0 {
	new += 1 << mutexWaiterShift
}
// The current goroutine switches mutex to starvation mode.
// But if the mutex is currently unlocked, don't do the switch.
// Unlock expects that starving mutex has waiters, which will not
// be true in this case.
if starving && old&mutexLocked != 0 {
	new |= mutexStarving
}
if awoke {
	// The goroutine has been woken from sleep,
	// so we need to reset the flag in either case.
	if new&mutexWoken == 0 {
		throw("sync: inconsistent mutex state")
	}
	new &^= mutexWoken
}
```

自旋结束之后就会去计算当前互斥锁的状态，如果当前处在饥饿模式下则不会去请求锁，而是会将当前 goroutine 放到队列的末端

```go
if atomic.CompareAndSwapInt32(&m.state, old, new) {
    if old&(mutexLocked|mutexStarving) == 0 {
        break // locked the mutex with CAS
    }
    // If we were already waiting before, queue at the front of the queue.
    queueLifo := waitStartTime != 0
    if waitStartTime == 0 {
        waitStartTime = runtime_nanotime()
    }
    runtime_SemacquireMutex(&m.sema, queueLifo, 1)
    starving = starving || runtime_nanotime()-waitStartTime > starvationThresholdNs
    old = m.state
    if old&mutexStarving != 0 {
        // If this goroutine was woken and mutex is in starvation mode,
        // ownership was handed off to us but mutex is in somewhat
        // inconsistent state: mutexLocked is not set and we are still
        // accounted as waiter. Fix that.
        if old&(mutexLocked|mutexWoken) != 0 || old>>mutexWaiterShift == 0 {
            throw("sync: inconsistent mutex state")
        }
        delta := int32(mutexLocked - 1<<mutexWaiterShift)
        if !starving || old>>mutexWaiterShift == 1 {
            // Exit starvation mode.
            // Critical to do it here and consider wait time.
            // Starvation mode is so inefficient, that two goroutines
            // can go lock-step infinitely once they switch mutex
            // to starvation mode.
            delta -= mutexStarving
        }
        atomic.AddInt32(&m.state, delta)
        break
    }
    awoke = true
    iter = 0
}
```

状态计算完成之后就会尝试使用 CAS 操作获取锁，如果获取成功就会直接退出循环 如果获取失败，则会调用 `runtime_SemacquireMutex(&m.sema, queueLifo, 1)` 方法保证锁不会同时被两个 goroutine 获取。`runtime_SemacquireMutex` 方法的主要作用是:

- 不断调用尝试获取锁
- 休眠当前 goroutine
- 等待信号量，唤醒 goroutine

goroutine 被唤醒之后就会去判断当前是否处于饥饿模式，如果当前等待超过 `1ms` 就会进入饥饿模式

- 饥饿模式下：会获得互斥锁，如果等待队列中只存在当前 Goroutine，互斥锁还会从饥饿模式中退出
- 正常模式下：会设置唤醒和饥饿标记、重置迭代次数并重新执行获取锁的循环

### 解锁

和加锁比解锁就很简单了，直接看注释就好

```go
// 解锁一个没有锁定的互斥量会报运行时错误
// 解锁没有绑定关系，可以一个 goroutine 锁定，另外一个 goroutine 解锁
func (m *Mutex) Unlock() {
	// Fast path: 直接尝试设置 state 的值，进行解锁
	new := atomic.AddInt32(&m.state, -mutexLocked)
    // 如果减去了 mutexLocked 的值之后不为零就会进入慢速通道，这说明有可能失败了，或者是还有其他的 goroutine 等着
	if new != 0 {
		// Outlined slow path to allow inlining the fast path.
		// To hide unlockSlow during tracing we skip one extra frame when tracing GoUnblock.
		m.unlockSlow(new)
	}
}

func (m *Mutex) unlockSlow(new int32) {
    // 解锁一个没有锁定的互斥量会报运行时错误
	if (new+mutexLocked)&mutexLocked == 0 {
		throw("sync: unlock of unlocked mutex")
	}
    // 判断是否处于饥饿模式
	if new&mutexStarving == 0 {
        // 正常模式
		old := new
		for {
			// 如果当前没有等待者.或者 goroutine 已经被唤醒或者是处于锁定状态了，就直接返回
			if old>>mutexWaiterShift == 0 || old&(mutexLocked|mutexWoken|mutexStarving) != 0 {
				return
			}
			// 唤醒等待者并且移交锁的控制权
			new = (old - 1<<mutexWaiterShift) | mutexWoken
			if atomic.CompareAndSwapInt32(&m.state, old, new) {
				runtime_Semrelease(&m.sema, false, 1)
				return
			}
			old = m.state
		}
	} else {
		// 饥饿模式，走 handoff 流程，直接将锁交给下一个等待的 goroutine，注意这个时候不会从饥饿模式中退出
		runtime_Semrelease(&m.sema, true, 1)
	}
}
```

## 为什么重复解锁要panic

可能你会想，为什么Go不能实现得更健壮些，多次执行Unlock()也不要panic？

仔细想想Unlock的逻辑就可以理解，这实际上很难做到。Unlock过程分为将Locked置为0，然后判断Waiter值，如果值>0，则释放信号量。

如果多次Unlock()，那么可能每次都释放一个信号量，这样会唤醒多个协程，多个协程唤醒后会继续在Lock()的逻辑里抢锁，势必会增加Lock()实现的复杂度，也会引起不必要的协程切换。

## 编程Tips

### 使用defer避免死锁

加锁后立即使用defer对其解锁，可以有效的避免死锁。

### 加锁和解锁应该成对出现

加锁和解锁最好出现在同一个层次的代码块中，比如同一个函数。

重复解锁会引起panic，应避免这种操作的可能性。



参考文章：

[Go专家编程](https://topgoer.cn/docs/gozhuanjia/gozhuanjiamutex)

[mohuishou的博客](https://lailin.xyz/post/go-training-week3-sync.html)