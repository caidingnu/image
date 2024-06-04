## 区别 一

## 本质属性

synchronized 是java的一个关键字，ReentrantLock是一个类

## 区别 二

### 用法不同

`synchronized 可用来修饰实例方法、静态方法和代码块，而 ReentrantLock 只能用在代码块上。`

### synchronized

##### synchronized 修饰代码块：

```java
public void exec() {
    synchronized (this) {
        // 执行业务逻辑
    }
}
```

```java
public void exec() {
    synchronized (Demo.class) {
        // 执行业务逻辑
    }
}
```



##### synchronized 修饰静态方法

```java
public static synchronized void exec() {
    
        // 执行业务逻辑

}
```

##### synchronized 修饰实例方法

```java
public synchronized void exec() {
    
        // 执行业务逻辑

}
```



#### ReentrantLock 

ReentrantLock 在使用之前需要先创建 ReentrantLock 对象，然后使用 lock 方法进行加锁，使用完之后再调用 unlock 方法释放锁，具体使用如下：

```java
public class LockDemo {
    // 创建锁对象
    private final ReentrantLock lock = new ReentrantLock();
    public void method() {
        // 加锁操作
        lock.lock();
        try {
            // 执行业务逻辑
        } finally {
            // 释放锁
            lock.unlock();
        }
    }
}
```

### 区别 三

### 获取锁和释放锁方式不同

**synchronized 会自动加锁和释放锁**，当进入 synchronized 修饰的代码块之后会自动加锁，当离开 synchronized 的代码段之后会自动释放锁

```java
 public void aa() {
        synchronized (this) {  //加锁
    
        } //释放锁
    }
```



**而 ReentrantLock 需要手动加锁和释放锁**，如下图所示：

```java
   /**
     * @author 蔡定努
     */
    ReentrantLock reentrantLock = new ReentrantLock();

    public void aa() {
        try {
            reentrantLock.lock(); // 加锁
            //     业务逻辑

        } finally {
            reentrantLock.unlock(); // 释放
        }
    }
```



### 区别 四

### 锁类型不同

**synchronized 属于非公平锁，而 ReentrantLock 既可以是公平锁也可以是非公平锁。**默认情况下 ReentrantLock 为非公平锁

```java

    ReentrantLock reentrantLock = new ReentrantLock(true); //公平锁
        ReentrantLock reentrantLock = new ReentrantLock(false); //非公平锁
```



### 区别五

### 响应中断不同

**ReentrantLock 可以使用 lockInterruptibly 获取锁并响应中断指令，而 synchronized 不能响应中断，也就是如果发生了死锁，使用 synchronized 会一直等待下去，而使用 ReentrantLock 可以响应中断并释放锁，从而解决死锁的问题**，比如以下 ReentrantLock 响应中断的示例：

```java
import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;
 
public class ReentrantLockInterrupt {
    static Lock lockA = new ReentrantLock();
    static Lock lockB = new ReentrantLock();
    public static void main(String[] args) throws InterruptedException {
        // 线程 1：先获取 lockA 再获取 lockB
        Thread t1 = new Thread(() -> {
            try {
                // 先获取 LockA
                lockA.lockInterruptibly();
                // 休眠 10 毫秒
                TimeUnit.MILLISECONDS.sleep(100);
                // 获取 LockB
                lockB.lockInterruptibly();
            } catch (InterruptedException e) {
                System.out.println("响应中断指令");
            } finally {
                // 释放锁
                lockA.unlock();
                lockB.unlock();
                System.out.println("线程 1 执行完成。");
            }
        });
        // 线程 2：先获取 lockB 再获取 lockA
        Thread t2 = new Thread(() -> {
            try {
                // 先获取 LockB
                lockB.lockInterruptibly();
                // 休眠 10 毫秒
                TimeUnit.MILLISECONDS.sleep(100);
                // 获取 LockA
                lockA.lockInterruptibly();
            } catch (InterruptedException e) {
                System.out.println("响应中断指令");
            } finally {
                // 释放锁
                lockB.unlock();
                lockA.unlock();
                System.out.println("线程 2 执行完成。");
            }
        });
        t1.start();
        t2.start();
        TimeUnit.SECONDS.sleep(1);
        // 线程1：执行中断
        t1.interrupt();
    }
}
```

以上程序的执行结果如下所示：![](https://gitee.com/caidingnu/blog_image/raw/master/img/04b592e004921d2d2ec607c91443a6ba.png)

### 区别 六

### 底层实现原理不同

**synchronized 是 JVM 层面通过监视器（Monitor）实现的，在对象头中进行计数，每次获取到所都会+1，直到该计数器为0，表示释放锁， synchronized 通过监视器实现，可通过观察编译后的字节码得出结论，如下图所示：![](https://gitee.com/caidingnu/blog_image/raw/master/img/396896a1bb1ae2d4b26e99bcb7f1f7b0.png)其中 monitorenter 表示进入监视器，相当于加锁操作，而 monitorexit 表示退出监视器，相当于释放锁的操作。

ReentrantLock 是通过 AQS（AbstractQueuedSynchronizer）程序级别的 API 实现，可通过观察 ReentrantLock 的源码得出结论，核心实现源码如下：![](https://gitee.com/caidingnu/blog_image/raw/master/img/112936f19984e66bd9c7cb13fc4b8e8f.png)

### 总结

不管synchronized 和还是ReentrantLock 都是 Java 中的可重入锁，总体上的区别如下

1.  用法不同：synchronized 可以用来修饰普通方法、静态方法和代码块，而 ReentrantLock 只能用于代码块。
    
2.  获取锁和释放锁的机制不同：synchronized 是自动加锁和释放锁的，而 ReentrantLock 需要手动加锁和释放锁。
    
3.  锁类型不同：synchronized 是非公平锁，而 ReentrantLock 默认为非公平锁，也可以手动指定为公平锁。
    
4.  响应中断不同：ReentrantLock 可以响应中断，解决[死锁](https://so.csdn.net/so/search?q=%E6%AD%BB%E9%94%81&spm=1001.2101.3001.7020)的问题，而 synchronized 不能响应中断。
    
5.  底层实现不同：synchronized 是 JVM 层面通过监视器实现的，而 ReentrantLock 是基于 AQS 实现的。
    


