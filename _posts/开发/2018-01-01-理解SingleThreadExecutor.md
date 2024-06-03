---
title: 理解 SingleThreadExecutor
categories: 开发
toc: true
permalink: /java-single-thread-executor.html
---

SingleThreadExecutor的含义不仅仅是说只有一个线程的线程池，它能**保证**任何时候都只有一个任务被执行，且能**保证**多个提交的任务按照提交的先后顺序被执行，本文分析了SingleThreadExecutor的实现原理。

## 源码

查看Executors中几种线程池的实现，可以看到newSingleThreadExecutor和其他线程池的实现不同，其他线程池直接返回ThreadPoolExecutor的实例，而newSingleThreadExecutor则返回FinalizableDelegatedExecutorService的实例，源码如下。

```java
public static ExecutorService newSingleThreadExecutor() {
    return new FinalizableDelegatedExecutorService
        (new ThreadPoolExecutor(1, 1,
                                0L, TimeUnit.MILLISECONDS,
                                new LinkedBlockingQueue<Runnable>()));
}
```

## 单个核心线程

SingleThreadExecutor的本质是一个核心线程数和最大线程数都是1的线程池，并且不允许线程超时，多个提交的任务被按顺序放到一个无界队列中。由于只有一个执行线程，所以在任何时候都只有一个任务被执行，多个任务按提交的先后顺序被执行，如果这个唯一运行的线程在执行期间因为某些原因被关闭，那么会创建一个新的线程来取代它执行接下来的任务。

## 禁止重新配置线程池

FinalizableDelegatedExecutorService是DelegatedExecutorService的子类，而DelegatedExecutorService是一个代理类，它实现了ExecutorService接口的所有方法，实现方法是委托给持有的ExecutorService。通过增加这一层代理，使得ThreadPoolExecutor的set方法不会对外暴露，从而确保返回的线程池不能被重新配置，也就保证了newSingleThreadExecutor永远只有一个线程执行。DelegatedExecutorService的代码如下：

```java
static class DelegatedExecutorService extends AbstractExecutorService {
    private final ExecutorService e;
    DelegatedExecutorService(ExecutorService executor) { e = executor; }
    public void execute(Runnable command) { e.execute(command); }
    public void shutdown() { e.shutdown(); }
    ......
}
```

## 线程池的关闭

由于SingleThreadExecutor核心线程池总是1，并且默认不允许核心线程池超时，也不允许重新配置线程池，那么一旦忘记关闭线程池，程序将被挂起，无法自己结束。这和ThreadPoolExecutor语义不一致。

ThreadPoolExecutor的注释中提到，如果线程池在程序中不再被引用，**并且**该线程池中没有存活的线程时，线程池将自动被关闭。对于ThreadPoolExecutor，用户可以通过设置合适的存活时间、设置核心线程数为0或者设置allowCoreThreadTimeOut(true)来确保线程池在忘记调用shutdown时仍能被回收.

SingleThreadExecutor返回的线程池不允许重新配置，所以FinalizableDelegatedExecutorService中通过实现finalize方法，调用线程的shutdown方法关闭线程，这样便可以在线程池不被引用时自动被关闭。

```java
static class FinalizableDelegatedExecutorService
    extends DelegatedExecutorService {
    FinalizableDelegatedExecutorService(ExecutorService executor) {
        super(executor);
    }
    protected void finalize() {
        super.shutdown();
    }
}
```

## 反例

如果还不理解上面的做法，那么只需看看下面用FixedThreadPool实现的单线程线程池就知道了。因为es可以被强制转换为ThreadPoolExecutor，所以无法确保只有一个任务被执行，而且由于没有调用shutdown，下面的程序是无法结束的。

```java
public static void main(String[] args) throws Exception {
    Runnable r = new Runnable() {
        @Override
        public void run() {
            System.out.println("www.hiwzc.com");
        }
    };

    ExecutorService es = Executors.newFixedThreadPool(1);
    es.submit(r);

    // 能够强制转换成ThreadPoolExecutor并重新配置
    ThreadPoolExecutor executor = (ThreadPoolExecutor) es;
    executor.setCorePoolSize(2);

    // 无法关闭线程并结束程序
    es = null;
    System.gc();
}
```

但如果使用SingleThreadExecutor则没有问题，代码如下：

```java
public static void main(String[] args) throws Exception {
    Runnable r = new Runnable() {
        @Override
        public void run() {
            System.out.println("www.hiwzc.com");
        }
    };

    ExecutorService es = Executors.newSingleThreadExecutor();
    es.submit(r);

    // 无法强制转换成ThreadPoolExecutor，抛ClassCastException异常
    // ThreadPoolExecutor executor = (ThreadPoolExecutor) es;
    // executor.setCorePoolSize(2);

    // 自动关闭线程，程序结束
    es = null;
    System.gc();
}
```
