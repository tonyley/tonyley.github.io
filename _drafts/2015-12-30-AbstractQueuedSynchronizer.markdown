---
layout: post
title: "AbstractQueuedSynchronizer"
categories: Java Concurrent
---
AbstractQueuedSynchronizer(AQS)
1. AQS主要概念
1) 状态: 使用volatile修饰的int型值保存状态，AQS通过系列API保证其操作的原子性。
2) AQS队列: 使用Node作为节点的双向链表，保存获取状态失败的线程。
3) Node: AQS队列节点对象
4) 阻塞算法: AQS采用CLH(Craig, Landin, and Hagersten)自旋算法使获取状态失败的线程处于阻塞状态。
5) ConditionObject: 为子类提供一个java.util.concurrent.locks.Condition实现类
6) 模式: AQS分两套API，一套是排他模式（exclusive mode），另外一套为共享模式（shared mode）

2. Node梳理
waitStatus 状态   CANCELLED     取消
                  SIGNAL        阻塞
                  CONDITION     Condition队列中
                  PROPAGATE     
                  0             已释放
prev       前节点
next       下一个节点
thread     线程对象    节点构造时初始化，节点释放时置为null
nextWaiter Condition队列节点

3. 排他模式（exclusive mode），又称独占模式
子类实现函数：
boolean tryAcquire(int arg)     true if successful. Upon success, this object has been acquired    
boolean tryRelease (int arg)    true if this object is now in a fully released state, so that any waiting threads may attempt to acquire; and false otherwise.
boolean isHeldExclusively ()    是否为当前线程锁定

子类调用函数：
开始同步：
acquire:
acquireInterruptibly
tryAcquireNanos

结束同步
release:

acquire：
1. 调用tryAcquire，如果获取成功结束acquire。如果失败继续以下操作。
2. 添加一个EXCLUSIVE模式的Node到AQS队列尾。
3. 


AQS阻塞：
1. AQS采用CLH自旋锁的方式使当前线程处于阻塞状态
2. 具体如下：
a. 判断是否为第一个可使用的节点，如果是则再次调用tryAcquire，如果获取成功结束自旋并将当前节点设置为头节点。
b. 如果上面a执行失败，则判断该节点是否应该阻塞，如果应该阻塞，则阻塞该节点对应的线程。

3. 自旋算法
   a. 检查自己是否为第一个有效节点。
   b. 判断自己是否应该被阻塞。在AQS中，节点根据前一节点的状态判断自己是否应该被阻塞。如果前一节点状态为SIGNAL，那么该节点应该被阻塞。如果前一节点状态为CANCELLED状态，循环删除当前节点的前面所有处于CANCELLED状态的节点，直到前面节点不为CANCELLED状态。如果前一节点的状态不为SIGNAL也不为CANCELLED，则将前一节点状态设置为SIGNAL。

4. 释放算法
   a. AQS队列是否可释放（不为空，状态不为0）
   b. 如果AQS队列头节点状态为可释放状态，则修改其状态为0
   c. 头节点的next节点是否为可释放状态（不为空，状态为可释放），如果可释放，则唤醒（unpark）头节点的next节点。如果不可释放，则从队列尾开始依次向前寻找可释放的节点并唤醒(unpark) (为什么没修改唤醒节点的状态及从队列中删除)


5. 取消算法


共享模式（shared mode）

ReentrantLock分析
lock:
tryAcquire 获取锁成功，继续执行，如果获取锁失败则加入等待队列等待





References:
CLH自旋锁： http://coderbee.net/index.php/concurrent/20131115/577 