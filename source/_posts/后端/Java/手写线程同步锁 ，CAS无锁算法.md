---
title: 手写线程同步锁 ，CAS无锁算法
date: 2019-11-14 12:00:00
author: azheng
img: /medias/banner/5.jpg
cover: false
categories: 后端
tags:
  - Java
  - 并发
---

类似于ReentrantLock（重入锁）
```java
import lombok.AccessLevel;
import lombok.Getter;
import lombok.Setter;
import lombok.extern.slf4j.Slf4j;
import sun.misc.Unsafe;

import java.lang.reflect.Field;
import java.util.concurrent.ConcurrentLinkedQueue;
import java.util.concurrent.locks.LockSupport;


/**
 * 线程同步器
 * 核心使用cas无锁算法 ，并自旋循环获取锁，且使用LockSupport进行线程阻塞和唤醒
 * @author azheng
 * @since 2019/11/21
 */
@Getter
@Setter
@Slf4j
public class ZujiLock {
    private volatile int state = 0;
    private Thread lockHolder;
    @Setter(AccessLevel.NONE)
    @Getter(AccessLevel.NONE)
    private final ConcurrentLinkedQueue<Thread> waiters = new ConcurrentLinkedQueue<>();

    private boolean acquire() {
        int s = getState();
        Thread current = Thread.currentThread();
        if(s==0){
            if((waiters.size() == 0 || current == waiters.peek()) &&
                compareAndSwapState(0,1)){
                setLockHolder(current);
                return true;
            }
        }
        return false;
    }

    public void lock(){
        if(acquire()){
            return;
        }
        //获取锁失败进行排队
        Thread current = Thread.currentThread();
        waiters.add(current);
        //自旋 循环获取锁，并且进行阻塞，等待上一个线程唤醒。
        while (true){
            if(waiters.peek() == current && acquire()){
                waiters.poll();
                return;
            }
            LockSupport.park();//阻塞
        }
    }

    public void unlock(){
        if(Thread.currentThread() != getLockHolder()){
            throw new RuntimeException("lockHolder is not current");
        }
        int state = getState();
        if(compareAndSwapState(1,0)){
            //释放锁并唤醒下一个线程
            setLockHolder(null);
            Thread current = waiters.peek();
            if(current != null){
                LockSupport.unpark(current);
            }
        }
    }
    public final boolean compareAndSwapState(int except, int update){
        return unsafe.compareAndSwapInt(this,stateOffset,except,update);
    }

    private static final Unsafe unsafe = initUnsafe();
    private static final long stateOffset = initStateOffset();

    private static long initStateOffset() {
        try {
            return unsafe.objectFieldOffset(ZujiLock.class.getDeclaredField("state"));
        } catch (NoSuchFieldException e) {
            e.printStackTrace();
            log.error(e.getMessage(),e);
            return 0;
        }
    }
    private static Unsafe initUnsafe(){
        try {
            Field f = Unsafe.class.getDeclaredField("theUnsafe");
            f.setAccessible(true);
            return (Unsafe) f.get(null);
        } catch (Exception e) {
            e.printStackTrace();
            log.error(e.getMessage(),e);
            return null;
        }
    }
}
```
测试代码

```java
@RestController
@RequestMapping("/test")
@Slf4j
public class TestController {

    private volatile Integer stock = 5;

    private ZujiLock zujiLock = new ZujiLock();

    @GetMapping("/order")
    public Object order() {
        zujiLock.lock();
        if(stock > 0){
            stock = stock - 1;
            log.info("-----------下单成功，剩余库存为："+stock);
        }else{
            log.info("-----------下单失败，已售罄!");
        }
        zujiLock.unlock();
        return "成功";
    }
}
```
