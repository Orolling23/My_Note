# Java线程间通信

https://redspider.gitbook.io/concurrent/di-yi-pian-ji-chu-pian/5  

## 锁与同步

Java中，锁的概念都是基于对象的，所以我们经常称它为对象锁。  

线程同步是线程之间按照一定顺序执行。为了达到线程同步，我们可以用锁来实现它。  

```
public class ObjectLock {
    private static Object lock = new Object();

    static class ThreadA implements Runnable {
        @Override
        public void run() {
            synchronized (lock) {
                for (int i = 0; i < 100; i++) {
                    System.out.println("Thread A " + i);
                }
            }
        }
    }

    static class ThreadB implements Runnable {
        @Override
        public void run() {
            synchronized (lock) {
                for (int i = 0; i < 100; i++) {
                    System.out.println("Thread B " + i);
                }
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        new Thread(new ThreadA()).start();
        Thread.sleep(10);
        new Thread(new ThreadB()).start();
    }
}
```

这里这个例子使用lock锁，让线程A执行完才执行线程B。  

### 场景

线程需要执行顺序，或者线程之间有共享资源并发写，都需要线程同步加锁。  

比如双重检验锁的单例模式  

## 等待/通知机制

基于Object类的wait()和notify(), notifyAll()方法来实现。  

比如上面的例子中，加入ThreadA调用lock.wait()让自己进入等待，那么此时lock锁被释放，ThreadB获得锁并执行。  

B在某一时刻，调用lock.notify()，通知其他线程被唤醒。但此时要注意，B并没有释放锁，而A只是被唤醒了，还没有从wait()方法上返回，返回还需要获取锁。所以A还需要等待B执行完毕，或者B主动调用wait()方法释放锁。  

```
public class WaitAndNotify {
    private static Object lock = new Object();

    static class ThreadA implements Runnable {
        @Override
        public void run() {
            synchronized (lock) {
                for (int i = 0; i < 5; i++) {
                    try {
                        System.out.println("ThreadA: " + i);
                        lock.notify();
                        lock.wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
                lock.notify();
            }
        }
    }

    static class ThreadB implements Runnable {
        @Override
        public void run() {
            synchronized (lock) {
                for (int i = 0; i < 5; i++) {
                    try {
                        System.out.println("ThreadB: " + i);
                        lock.notify();
                        lock.wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
                lock.notify();
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        new Thread(new ThreadA()).start();
        Thread.sleep(1000);
        new Thread(new ThreadB()).start();
    }
}

// 输出：
ThreadA: 0
ThreadB: 0
ThreadA: 1
ThreadB: 1
ThreadA: 2
ThreadB: 2
ThreadA: 3
ThreadB: 3
ThreadA: 4
ThreadB: 4
```

### 场景

等待通知机制适用于**生产者消费者模式中**。生产者消费者模式就是通过一个容器来解决生产者和消费者的强耦合，不直接通信，而是通过阻塞队列来通信。

一般就是资源有固定限制，生产者产生资源，资源放不下了就把自己挂起，直到有消费者消费了资源唤醒了自己；消费者消耗资源，资源消耗完了就把自己挂起，直到有生产者生产了资源唤醒了自己。  

* 数据库连接池/线程池等各种资源池

## 信号量

Java提供了类似信号量的类 Semaphore。也可以利用volatile关键字自己实现信号量通信。  

> volatile关键字保证内存可见性

```
public class Signal {
    private static volatile int signal = 0;

    static class ThreadA implements Runnable {
        @Override
        public void run() {
            while (signal < 5) {
                if (signal % 2 == 0) {
                    System.out.println("threadA: " + signal);
                    synchronized (this) {
                        signal++;
                    }
                }
            }
        }
    }

    static class ThreadB implements Runnable {
        @Override
        public void run() {
            while (signal < 5) {
                if (signal % 2 == 1) {
                    System.out.println("threadB: " + signal);
                    synchronized (this) {
                        signal = signal + 1;
                    }
                }
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        new Thread(new ThreadA()).start();
        Thread.sleep(1000);
        new Thread(new ThreadB()).start();
    }
}

// 输出：
threadA: 0
threadB: 1
threadA: 2
threadB: 3
threadA: 4
```

注意signal++非原子，所以需要上锁。  

### 场景

* 信号量可以作为锁，让某段代码执行之前先检查信号量
* 信号量可以作为资源计数，限制资源的使用数量。比如线程的并发数量

## join()

让某个线程等待其他线程执行完毕，比如主线程等待子线程执行完毕  

