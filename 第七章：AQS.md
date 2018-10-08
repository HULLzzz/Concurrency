# AQS同步组件
## countDownLatch
实现类似计数器的功能，例如有一个任务A，他要执行4个线程之后才能执行，就需要用到countdownlatch
```java
@Slf4j
public class countDownLatch {
    private final static int threadCount = 200;

    public static void main (String[] args) throws Exception {
        //线程池用于调用多个线程
        ExecutorService exec = Executors.newCachedThreadPool();
        final CountDownLatch countDownLatch = new CountDownLatch(threadCount);

        for (int i = 0;i<threadCount;i++){
            final int threadNum = i;
            exec.execute(()->{
                try{
                    test(threadNum);
                }catch (Exception e){
                    log.error("exception",e);
                }finally{
                    countDownLatch.countDown(); //每个线程执行完需要调用countDown进行-1操作
                }
            });
        }
        countDownLatch.await(); //await保证方法之前的线程全部执行完
        //可以设置时间，在规定时间（这里约束的时间是线程中的方法的时间）内，没有执行完的线程不再执行       //countDownLatch.await(10,TimeUnit.MICROSECONDS);
        log.info("finish");
    }

    private static void test(int threadNum) throws Exception{
        Thread.sleep(100);
        log.info("{}",threadNum);
        Thread.sleep(100);
    }

}
```

## Semaphore
使用场景：并发访问控制：数据库的连接数是20，但是链接请求有200，会出现达不到连接数的错误，这时候可以使用semaphore
```java
package com.mmall.concurrency.example.aqs;

import lombok.extern.slf4j.Slf4j;

import java.util.concurrent.CountDownLatch;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Semaphore;

/**
 * @Auther: Think
 * @Date: 2018/10/8 13:12
 * @Description:
 */
@Slf4j
public class semaphoreEx {
    private final static int threadCount = 20;

    public static void main (String[] args) throws Exception {
        //线程池用于调用多个线程
        ExecutorService exec = Executors.newCachedThreadPool();
        final Semaphore semaphore = new Semaphore(3); //允许的并发数
        for (int i = 0;i<threadCount;i++){
            final int threadNum = i;
            exec.execute(()->{
                try{
                    semaphore.acquire();//获取一个许可
                    test(threadNum);
                    semaphore.release();//释放一个许可
                }catch (Exception e){
                    log.error("exception",e);
                }
            });
        }
         log.info("finish");
    }

    private static void test(int threadNum) throws Exception{
        Thread.sleep(10000);
        log.info("{}",threadNum);
        Thread.sleep(100);
    }
}
```

## cyclicBarrier
多线程分组计算

## reentrantLock
reentrantlock独有的功能：  
* 可判定是公平锁还是非公平锁，而syn只能是非公平锁（公平锁：先等待的线程先获得锁）
* 提供了condition类，可以实现线程同步，分组唤醒需要的唤醒的线程，syn只能随机唤醒一个线程或者是唤醒全部线程
* 提供能中断等待锁的线程的机制，持有锁线程长时间不释放锁的时候，等待线程可选择放弃等待 tryLock(long timeout, TimeUnit unit)
reentrantlock是实现了lock接口,syn不会造成死锁，但是其他的锁在操作不当的情况下（没有unlock），可能会造成死锁

## reentrantReadWriteLock
实现读写分离，但是写的时候需要等待读锁被释放，如果读的操作太多的话，会时写操作陷入长时间阻塞状态

## condition
```java
package com.mmall.concurrency.example.lock;

import lombok.extern.slf4j.Slf4j;

import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.ReentrantLock;

@Slf4j
public class LockExample6 {

    public static void main(String[] args) {
        ReentrantLock reentrantLock = new ReentrantLock();
        Condition condition = reentrantLock.newCondition();

        new Thread(() -> {
            try {
                reentrantLock.lock();
                log.info("wait signal"); // 1
                condition.await();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            log.info("get signal"); // 4
            reentrantLock.unlock();
        }).start();

        new Thread(() -> {
            reentrantLock.lock();
            log.info("get lock"); // 2
            try {
                Thread.sleep(3000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            condition.signalAll();
            log.info("send signal ~ "); // 3
            reentrantLock.unlock();
        }).start();
    }
}
```
执行完1之后，调用了condition.await()，相当于将锁资源释放，线程2此时获得了锁，执行2，3，线程2的锁释放之后，唤醒线程1，则继续执行4


