# futureTask
* 使用场景  
利用FutureTask和ExecutorService，可以用多线程的方式提交计算任务，主线程继续执行其他任务，当主线程需要子线程的计算结果时，在异步获取子线程的执行结果。
```java
package com.mmall.concurrency.example.aqs;

import lombok.extern.slf4j.Slf4j;

import java.util.concurrent.*;


/**
 * @Auther: Think
 * @Date: 2018/10/8 15:59
 * @Description:
 */
@Slf4j
public class FutureEx {
    static class MyCallable implements Callable<String> {
        @Override
        public String call() throws Exception {
            log.info("do something in callable");
            Thread.sleep(5000);
            return "Done";
        }
    }

    public static void main(String[] args) throws Exception {
        ExecutorService executorService = Executors.newCachedThreadPool();
        Future<String> future = executorService.submit(new MyCallable());
        log.info("do something in main");
        Thread.sleep(1000);
        String result = future.get();
        log.info("result:{}",result);
    }
}
```

# Fork/Join框架
分解：当一个任务需要分解为多个小任务的时候，在框架中执行这样的小任务。合并：在主任务中等待子任务执行完成

# ArrayBlockingQueue和LinkedBlockingQueue
ArrayBlockingQueue和LinkedBlockingQueue的区别：
* 队列中锁的实现不同  
  ArrayBlockingQueue实现的队列中的锁是没有分离的，即生产和消费用的是同一个锁，LinkedBlockingQueue实现的队列中的锁是分离的，即生产用的是putLock，消费是takeLock，这也就意味着linked能够高效处理高并发。
* 在生产或消费时操作不同  
  ArrayBlockingQueue实现的队列中在生产和消费的时候，是直接将枚举对象插入或移除的；LinkedBlockingQueue实现的队列中在生产和消费的时候，需要把枚举对象转换为Node<E>进行插入或移除，会影响性能
* 队列大小初始化方式不同  
  ArrayBlockingQueue实现的队列中必须指定队列的大小；LinkedBlockingQueue实现的队列中可以不指定队列的大小，但是默认是Integer.MAX_VALUE

## 内部方法解析
* put take 方法
生产者将信息发送至队列中（put），直到达到队列的上限值，达到上限值后队列被阻塞，负责消费的线程在队列中消费对象，直到队列为空，队列为空的时候消费线程阻塞。take: 获取并移除此队列的头部，在元素变得可用之前一直等待 。queue的长度 == 0 的时候，一直阻塞。
* offer
offer: 将指定元素插入此队列中（如果立即可行且不会违反容量限制），成功时返回 true，如果当前没有可用的空间，则返回 false，不会抛异常。

