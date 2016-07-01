---
layout: blog_item
title: 'HandlerThread源码分析'
date: 2016-07-01 10:00:00 +0800
author: christiein
categories: [原创]
---

# 1. HandlerThread是什么

经常都会使用到HandlerThread，看了它源码以后发现，其实HandlerThread是Thread的子类，因此HandlerThread其实是一个线程，只不过其内部帮你实现了一个Looper的循环而已。

# 2. HandlerThread怎么用

HandlerThread的用法也是相当简单的。

1. 创建实例对象
2. 启动HandlerThread线程
3. 构建循环消息处理机制

下面是示例代码：

```java
HandlerThread handlerThread = new HandlerThread("demo_handler_thread");
handlerThread.start();
Handler handler = new Handler(handlerThread.getLooper()) {
    @Override
    public boolean handleMessage(Message msg) {
        //实现自己的消息处理
        return true;
    }
};
```

也可以这样使用：

```java
HandlerThread handlerThread = new HandlerThread("demo_handler_thread");
handlerThread.start();
Handler handler = new Handler(handlerThread.getLooper(), new Handler.Callback() {
        @Override
        public boolean handleMessage(Message msg) {
            //实现自己的消息处理
            return true;
        }
});
```

# 3. HandlerThread的源码

打开HandlerThread的源码，你会发现它的源码真的很简单。首先是3个成员变量。

```java
public class HandlerThread extends Thread {
    int mPriority;// 线程优先级
    int mTid = -1;// 线程id
    Looper mLooper;// 与线程关联的Looper

    public HandlerThread(String name) {// 提供个名字，方便debug
        super(name);
        mPriority = Process.THREAD_PRIORITY_DEFAULT;// 没提供，则使用默认优先级
    }
    
    /**
     * Constructs a HandlerThread.
     * @param name
     * @param priority The priority to run the thread at. The value supplied must be from 
     * {@link android.os.Process} and not from java.lang.Thread.
     */
    public HandlerThread(String name, int priority) {
        super(name);
        mPriority = priority;// 使用用户提供的优先级，基于linux优先级，取值在[-20,19]之间
    }

    ...
}
```

<div class="note info">
  <h5>注意</h5>
  <p>值得注意的是这里的priority是基于Linux的优先级的，而不是Java Thread类里的MIN_PRIORITY，NORM_PRIORITY，MAX_PRIORITY之类，请注意区分。</p>
</div>

HandlerThrad的run方法

```java
public class HandlerThread extends Thread {
    ...
    /**
     * Call back method that can be explicitly overridden if needed to execute some
     * setup before Looper loops.
     */
    protected void onLooperPrepared() {
    }

    @Override
    public void run() {
        //获得当前线程的id
        mTid = Process.myTid();
        //准备循环条件
        Looper.prepare();
        //持有锁机制来获得当前线程的Looper对象
        synchronized (this) {
            mLooper = Looper.myLooper();
            //发出通知，当前线程已经创建mLooper对象成功，这里主要是通知getLooper方法中的wait
            notifyAll();
        }
        //设置当前线程的优先级
        Process.setThreadPriority(mPriority);
        //该方法实现体是空的，子类可以实现该方法，作用就是在线程循环之前做一些准备工作，当然子类也可以不实现。
        onLooperPrepared();
        //启动loop
        Looper.loop();
        mTid = -1;
    }
    ....
}
```

getLooper获得当前线程的Looper对象

```java
public class HandlerThread extends Thread {
    ...
    /**
     * This method returns the Looper associated with this thread. If this thread not been started
     * or for any reason is isAlive() returns false, this method will return null. If this thread 
     * has been started, this method will block until the looper has been initialized.  
     * @return The looper.
     */
    public Looper getLooper() {
        //如果线程不是存活的，则直接返回null
        if (!isAlive()) {
            return null;
        }
        
        // If the thread has been started, wait until the looper has been created.
        //如果线程已经启动，但是Looper还未创建的话，就等待，知道Looper创建成功
        synchronized (this) {
            while (isAlive() && mLooper == null) {
                try {
                    wait();
                } catch (InterruptedException e) {
                }
            }
        }
        return mLooper;
    }
```

该方法主要作用是获得当前HandlerThread线程中的mLooper对象。

首先判断当前线程是否存活，如果不是存活的，这直接返回null。其次如果当前线程存活的，在判断线程的成员变量mLooper是否为null，如果为null，说明当前线程已经创建成功，但是还没来得及创建Looper对象，因此，这里会调用wait方法去等待，当run方法中的notifyAll方法调用之后通知当前线程的wait方法等待结束，跳出循环，获得mLooper对象的值。

**总结**：在获得mLooper对象的时候存在一个同步的问题，只有当线程创建成功并且Looper对象也创建成功之后才能获得mLooper的值。这里等待方法wait和run方法中的notifyAll方法共同完成同步问题。

quit结束当前线程的循环

```java
    /**
     * Quits the handler thread's looper.
     * <p>
     * Causes the handler thread's looper to terminate without processing any
     * more messages in the message queue.
     * </p><p>
     * Any attempt to post messages to the queue after the looper is asked to quit will fail.
     * For example, the {@link Handler#sendMessage(Message)} method will return false.
     * </p><p class="note">
     * Using this method may be unsafe because some messages may not be delivered
     * before the looper terminates.  Consider using {@link #quitSafely} instead to ensure
     * that all pending work is completed in an orderly manner.
     * </p>
     *
     * @return True if the looper looper has been asked to quit or false if the
     * thread had not yet started running.
     *
     * @see #quitSafely
     */
    public boolean quit() {
        Looper looper = getLooper();
        if (looper != null) {
            looper.quit();
            return true;
        }
        return false;
    }

    /**
     * Quits the handler thread's looper safely.
     * <p>
     * Causes the handler thread's looper to terminate as soon as all remaining messages
     * in the message queue that are already due to be delivered have been handled.
     * Pending delayed messages with due times in the future will not be delivered.
     * </p><p>
     * Any attempt to post messages to the queue after the looper is asked to quit will fail.
     * For example, the {@link Handler#sendMessage(Message)} method will return false.
     * </p><p>
     * If the thread has not been started or has finished (that is if
     * {@link #getLooper} returns null), then false is returned.
     * Otherwise the looper is asked to quit and true is returned.
     * </p>
     *
     * @return True if the looper looper has been asked to quit or false if the
     * thread had not yet started running.
     */
    //安全退出循环
    public boolean quitSafely() {
        Looper looper = getLooper();
        if (looper != null) {
            looper.quitSafely();
            return true;
        }
        return false;
    }

    /**
     * Returns the identifier of this thread. See Process.myTid().
     */
    public int getThreadId() {
        return mTid;
    }
}
```

分析：以上有两种让当前线程退出循环的方法，一种是安全的，一中是不安全的。至于两者有什么区别? quitSafely方法效率比quit方法标率低一点，但是安全。具体选择哪种就要看具体项目了。

# 总结

1. HandlerThread适用于构建循环线程。
2. 在创建Handler作为HandlerThread线程消息执行者的时候必须调用start方法之后，因为创建Handler需要的Looper参数是从HandlerThread类中获得，而Looper对象的赋值又是在HandlerThread的run方法中创建。

