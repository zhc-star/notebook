学习规划:
视频40%
博客30%
源码20%
总结输出10%


第一阶段目标(预计三周):
达到面试官问: `ReentrantLock`, 可以从源码层面讲清楚.

---

# 第一周:
```
线程
⬇️
CAS
⬇️
volatile
⬇️
AQS
⬇️
ReentrantLock
```
注意💡: 别发散!!!

---

## 1 什么是CAS
搞懂`AtomicInteger`为什么线程安全
能回答:
```
什么是CAS
ABA问题是什么
为什么比synchronized快
CAS失败怎么办
```

## 2 什么是volatile
搞懂`volatile int flag`到底解决什么问题
重点: 
```
可见性
有序性
不能保证原子性
```

## 3 AQS总体结构
不要看源码细节, 只看世界观.
理解:
```
state
CLH队列
park
unpark
```
达到这个程度:
	AQS本质就是state + FIFO等待队列 
就够了

今晚学习完, 回复:
```
CAS学完
volatile学完
AQS学完
```

准备几个问题: 
```
为什么CAS需要循环?
volatile为什么不能保证++安全
AQS为什么要维护队列
```
如果答不出来, 继续补, 答出来进入下一阶段.


# 第二周

```
ReentrantLock
CountDownLatch
Semaphore
ThreadPoolExecutor
ConcurrentHashMap
```


# 第三周

```
线程池源码
AQS源码
ConcurrentHashMap源码
```

拼多多面试过手写线程池, 重点!!!