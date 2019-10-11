---
title: 条件变量
date: 2019-03-05 17:52:09
tags:
 - C++
---

## 使用场景
在多线程编程中，经常会出现多个消费者同时等待、竞争资源的场景。利用条件变量以及互斥锁的机制，能优雅地解决这个问题。

<!--more-->

## 简单实例
```c++
std::condition_variable cv;
std::mutex mux;
int resource = 0;

//消费者竞争资源
void consumer()
{
    std::unique_lock<std::mutex> lk(mux);  //上锁
    while(resource == 0)    
        // 3.抢到锁的消费者，到4获取资源
        // 6.没有抢到锁的消费者在锁被释放后唤醒，判断资源已经没有了，回到1继续等待（见下方一节虚假唤醒）
    {
        // 1.解锁，等待资源
        cv.wait(lk);   
        // 2.唤醒后，重新竞争互斥锁
    } 
        // 4.抢到互斥锁的消费者，获取资源
    --resource;
}       //5.局部变量释放，解锁
```

```c++
//生产者生产资源
void producer()
{
    std::unique_lock<std::mutex> lk(mux);  //上锁
    ++resource;         //产生资源
    cv.notify_all();    //唤醒消费者
}   //解锁
```
## 虚假唤醒

在消费者的操作中，使用while循环判断资源是为了防止虚假唤醒，在多个消费者的场景下，如果执行notify_all，抢到互斥锁的消费者会率先被唤醒，此时其他消费者仍在等待状态，当取走资源后，互斥锁被释放（unique_lock析构时自动释放），此时其他消费者被唤醒，因此要再次判断资源是否可用，否则为虚假唤醒，重新进入等待的状态。

使用wait_for/wait_until也能很解决虚假唤醒的问题，他能指定一个判断条件，当条件为真时，才会被真正唤醒，同时还能设置超时时间，防止无限期等待。

```c++
//消费者竞争资源
void consumer()
{
    std::unique_lock<std::mutex> lk(mux);  //上锁
    if(cv.wait_for(lk, 
                std::chrono::milliseconds{1000},
                []{return i == 1}))
    {
        --resource;
    }
    else
    {
        std::out << " timed out" << std::endl;
    }
    
}//解锁
```
通过在判断函数里面加打印可以发现，实际上在notify_all的时候，每个消费者都会调用一次比较函数，查看资源是否可用（这里面也隐含了一次解锁和上锁的操作），如果资源不可用，则继续等待，效果与前面轮询的效果一致。

*对锁的操作规律：在开始等待的时候，锁被释放，收到通知时，重新竞争锁*