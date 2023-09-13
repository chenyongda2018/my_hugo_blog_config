
---
title: "Android消息机制全面解析"
date: 2019-04-19T20:27:16+08:00
draft: false
toc: false
comments: false
categories:
- Android
tags:
- Android
- Handler
---


## (1),Android消息机制概述 

Android中的消息机制主要指 **Handler的运行机制** 以及 **MessageQueue，Looper的工作过程** ，三者相互协作，保证着消息的接收，发送，处理，执行。

![ ](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/4/28/16a645eb49972a0d~tplv-t2oaga2asx-image.image)       

> ​                                                            图片来自郭神的《第一行代码》 

先简单的介绍一下 Android 中 消息机制大家庭的主要成员 :  

- **Handler** : 是Android消息机制的上层接口，最为大家常用，相当于Android消息机制的入口，我们通过使用`Handler`发送消息来引起消息机制的循环。通常用于:在子线程执行完耗时任务完后，更新UI。

- **MessageQueue** : 存储 **消息(Message)** 对象的消息队列，实则是单链表结构.

- **Looper** : 用于无限的从 `MessageQueue`中取出消息，相当于消息的永动机，如果有新的消息，则处理执行，若没有，则就一直等待，堵塞。Looper** 所在的线程是 创建 `Handler` 时所在的线程。

  主线程创建Handler时，会自动创建一个Looper,**但是子线程并不会自动创建Looper** 

- **ThreadLocal** : 在每个线程互不干扰的存储，提供数据，以此来获取当前线程的`Looper`

- **ActivityThread** : Android 的主线程，也叫UI线程，主线程被创建时 自动初始化主线程的 `Looper`对象。



#### 问题 : 大家都知道只有在UI线程才能对UI元素进行操作，在子线程更改UI就会报错，为什么？

看完[《Android艺术开发探索》](<https://book.douban.com/subject/26599538/>) 这本书的第10章之后我也才明白

1. Android中的UI控件不是线程安全的，如果在子线程中也能修改UI元素，那多线程的时，共同访问同一个UI元素，就会导致这个UI元素处于我们不可预知的状态，这个线程让它往左一点，那个线程让它往右一点，UI该听谁的，好tm乱。。 干脆我就只听主线程的把。



#### 问题 : 那为什么不通过对访问UI控件的子线程加上锁机制呢 ？ 

这个很简单了，如果为不同的线程访问同一UI元素加上锁机制，那我们程序员写相关代码的时候会变得超级麻烦。。。 改个UI还得考虑它是不是已经被别的线程占用了，被占用了，还得让那个线程释放锁。。。线程再多一点的话，大大地加大了程序员地工作量.

而且加上锁机制无疑会由于线程堵塞地原因降低访问UI的效率，帧率降低，体验也会不友好。

让UI元素只能再主线程访问就会省下很多事，创建一个`Handler`就行了。



下面从整体概述一下 消息机制的整个工作过程 : 

1. `Handler `创建时会采用当前线程的 `Looper`来构建内部的消息循环系统，如果Handler在子线程，则一开始是没有`Looper`对象的(解决方法稍后介绍)，主线程`ActivityThread`默认有一个Looper。

2. `Handler`创建完毕，通过 `post`方法传入`Runnable`对象，或者通过`sendMessage(Message msg)`发送消息。  

   `post()`方法里也是通过调用`send()`实现的

3. `send()`方法被调用后，调用 MessageQueue的enqueueMessage()方法将消息发送到消息队列中，等待被处理。

4. `Looper`对象运行在`Handler`所在的线程，从`MessageQueue消息队列`中不断地取出消息，处理，所以业务逻辑(通常是更新UI)就运行在Looper的线程中。





### 

接下来从局部来分析消息机制的每个成员。





## (2),ThreadLocal 工作原理

#### 1, 	什么是ThreadLocal？

ThreadLocal是一个线程内部的数据存储类，通过它可以在指定的线程中获得存储数据，获得数据，线程之间的ThreadLocal相互独立，且无法获得另一个线程的TheadLocal.

- 相对整个程序来说，每个线程的ThreadLocal是局部变量。

- 相对一个线程来说，线程内的ThreadLocal是线程的全局变量

ThreadLocal是一个泛型类，可以存储任意类型的对象。

示例:  

```java
public class ThreadLocalTest {

	public static void main(String[] args) {
		
		ThreadLocal<Boolean> mThreadLocal = new ThreadLocal<Boolean>();
		mThreadLocal.set(true);
		System.out.println("#Main Thread : ThreadLocal " + mThreadLocal.get());
		
		
		new Thread( new Runnable() {
			
			@Override
			public void run() {
				mThreadLocal.set(false);
				System.out.println("#1 Thread : ThreadLocal " + mThreadLocal.get());
			}
		}).start();
		
		new Thread( new Runnable() {
			
			@Override
			public void run() {
				System.out.println("#2 Thread : ThreadLocal " + mThreadLocal.get());
			}
		}).start();

	}

}
```

我们在主线程创建一个 泛型为`Boolean`的ThreadLocal，并`.set(True)`,然后在第一个子线程中`.set(False)`，在第二个子线程中不做修改，直接打印。 可以看到，在不同的线程中获得的值也不同。



输出 : 

```
#Main Thread : ThreadLocal true
#1 Thread : ThreadLocal false
#2 Thread : ThreadLocal null
```



#### 2，ThreadLocal的实现原理

首先每个线程内部都维护着一个`ThreadLocalMap`对象

*Thread.Java*

```java
/* ThreadLocal values pertaining to this thread. This map is maintained
 * by the ThreadLocal class. */
ThreadLocal.ThreadLocalMap threadLocals = null;
```

这个ThraedLocalMap 与Map类似，一个线程内可以有多个ThreadLocal类型变量，所以通过`ThreadLocalMap <ThreadLocal<?> key, Object value>`.保存着多个<ThreadLocal  ,  任意类型对象>键值对。

看一下ThreadLocal的`set()`方法实现 :    

```java
/**
     * Sets the current thread's copy of this thread-local variable
     * to the specified value.  Most subclasses will have no need to
     * override this method, relying solely on the {@link #initialValue}
     * method to set the values of thread-locals.
     *
     * @param value the value to be stored in the current thread's copy of
     *        this thread-local.
     */
    public void set(T value) {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null)
            map.set(this, value);
        else
            createMap(t, value);
    }
```



 先是获得当前线程的`ThreadLocalMap`对象，map.set(this,value)  设置了我这个ThreadLocal存储的值.



`get()`方法实现 : 

```java
/**
     * Returns the value in the current thread's copy of this
     * thread-local variable.  If the variable has no value for the
     * current thread, it is first initialized to the value returned
     * by an invocation of the {@link #initialValue} method.
     *
     * @return the current thread's value of this thread-local
     */
    public T get() {
        Thread t = Thread.currentThread();//获得当前线程
        ThreadLocalMap map = getMap(t);//根据根据获得它的ThreadLocalMap
        if (map != null) {
            ThreadLocalMap.Entry e = map.getEntry(this);//获得<k,v>键值对
            if (e != null) {
                @SuppressWarnings("unchecked")
                T result = (T)e.value;//通过<k,v>获得值
                return result;
            }
        }
        return setInitialValue();
    }
```







#### 3，ThreadLocal的使用场景

一般，当某些数据是以线程为作用域，并且不同的线程具有不同的数据副本时，可以考虑用`ThreadLocal`

###### 场景1:   

对于`Handler`,它想要获得当前线程的`Looper`，并且`Looper`的作用域就是当前的线程，不同的线程具有不同的`Looper`对象，这时可以使用ThreadLocal。

###### 场景2: 

复杂逻辑下的对象的传递，如果想要一个对象贯穿着整个线程的执行过程，可采用Threadlocal让此对象作为该线程的全局对象。







## (3),MessageQueue的工作原理

以**单链表**的形式，存储着`Handler`发送过来的消息，再来一张图加深印象  

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/4/28/16a645eb4989cbbf~tplv-t2oaga2asx-image.image)

主要包含两个操作:

- 通过`enqueueMessage(Message msg,long when)`，像队列插入一个消息,这里为了节省篇幅，就不上源码，贴上源码连接,[MessageQueue.enqueueMessage()](<http://androidxref.com/6.0.0_r1/xref/frameworks/base/core/java/android/os/MessageQueue.java#enqueueMessage>)
- 通过`next()`从无限循环队列中取出消息，并从消息队列中删除。[MessageQueue.next()](<http://androidxref.com/6.0.0_r1/xref/frameworks/base/core/java/android/os/MessageQueue.java#next>)

虽然它叫做`消息队列`，但内部其实是以单链表的结构存储，有利于插入，删除的操作。







## (4),Looper的工作原理

它的主要作用就是 不停地从消息队列中 查看是否有新的消息，如果有新的消息就会立刻处理，没有消息就会堵塞。

持有`MessageQueue`的引用，并且会在构造方法中初始化  

```java
private Looper(boolean quitAllowed) {
    mQueue = new MessageQueue(quitAllowed);
    mThread = Thread.currentThread();
}
```

#### 问题: 如何在子线程创建它的Looper对象 ? 

前面说到主线程自己会创建一个`Looper`对象，所以我们在主线程使用`Handler`的时候直接创建就可以了。

但是在子线程使用Handler的话，就需要我们手动创建`Looper`了，

示例:  

```java
new Thread() {
    @Override
    public void run() {
        Looper.prepare();
        Handler handler = new Handler();
        Looper.loop();
    }
}.start();
```

`prepare()`源码如下:  

```java
 /** Initialize the current thread as a looper.
77      * This gives you a chance to create handlers that then reference
78      * this looper, before actually starting the loop. Be sure to call
79      * {@link #loop()} after calling this method, and end it by calling
80      * {@link #quit()}.
81      */
82    public static void prepare() {
83        prepare(true);
84    }
85
86    private static void prepare(boolean quitAllowed) {
87        if (sThreadLocal.get() != null) {
88            throw new RuntimeException("Only one Looper may be created per thread");
89        }
90        sThreadLocal.set(new Looper(quitAllowed));
91    }
```

可以看到最终是调用了 此 Looper 所在线程的 **ThreadLocal.set()**方法，存了一个Looper对象进去。



除了`prepare()`，还有一些其他方法，我们也需要知道  

- **loop()** : 启动消息循环，，只有当Looper调用了loop()之后，整个消息循环才活了起来

- **prepareMainLooper()** : 给主线程创建Looper对象

- **getMainLooper()** : 获得主线程的Looper对象

- **quit()** : 通知消息队列，直接退出消息循环，不等待当前正在处理的消息执行完，quit之后，再向消息队列中发送新的消息就会失败( Handler的send()方法就会返回false )

  ```java
   public void quit() {
       mQueue.quit(false);
   }
  ```

  

- **quitSafety()** : 通过消息队列，不再接收新的消息，等当前的消息队列中的消息处理完就退出。

  ```java
   public void quitSafely() {
          mQueue.quit(true);
   }    
  ```



下面分析`loop()`的实现:  

```java
/**
119     * Run the message queue in this thread. Be sure to call
120     * {@link #quit()} to end the loop.
121     */
122    public static void loop() {
123        final Looper me = myLooper();
124        if (me == null) {
125            throw new RuntimeException("No Looper; Looper.prepare() wasn't called on                                            this thread.");
126        }
127        final MessageQueue queue = me.mQueue;
128
129        ...//省略部分代码
133		  //从这里开启无限循环，直到 没有消息
134        for (;;) {
135            Message msg = queue.next(); // might block
136            if (msg == null) {
137                // No message indicates that the message queue is quitting.
138                return;
139            }
140
141            // This must be in a local variable, in case a UI event sets the logger
142            Printer logging = me.mLogging;
143            if (logging != null) {
144                logging.println(">>>>> Dispatching to " + msg.target + " " +
145                        msg.callback + ": " + msg.what);
146            }
147
148            msg.target.dispatchMessage(msg);
149			   ...//省略部分代码
166        }
167    }
```

在 for 循环里 : 

1. 通过queue.next()一直读取新的消息，如果没有消息 则退出循环。
2. 接下来，` msg.target.dispatchMessage(msg);`，target是发送此消息的 Hander对像，通知Handler调用`dispatchMessage()`来接收消息。





## (5),Handler的工作原理

Handler的主要工作就是 发送消息，接收消息。

发送消息的方式有post(),send(),不过post()方法最后还是调用的send()方法



- **发送消息的过程**: 

  send类型的发送消息方法有很多，并且是嵌套的

  *sendMessage()*

  ```java
  public final boolean sendMessage(Message msg) {
          return sendMessageDelayed(msg, 0);
  }
  ```

  *sendMessageDelayed()*

  ```java
  public final boolean sendMessageDelayed(Message msg, long delayMillis) {
      if (delayMillis < 0) {
          delayMillis = 0;
      }
      return sendMessageAtTime(msg, SystemClock.uptimeMillis() + delayMillis);
  }
  ```

   *sendMessageAtTime()*  

  ```java
   public boolean sendMessageAtTime(Message msg, long uptimeMillis) {
         MessageQueue queue = mQueue;
  		if (queue == null) {
           RuntimeException e = new RuntimeException(
                      this + " sendMessageAtTime() called with no mQueue");
              Log.w("Looper", e.getMessage(), e);
              return false;
          }
          return enqueueMessage(queue, msg, uptimeMillis);
  }
  ```

  三种send的发送消息方式，最后都会通过`enqueueMessage()`来通知消息队列 插入这条新的消息。

  *Handler.enqueueMessage*

  ```java
  private boolean enqueueMessage(MessageQueue queue, Message msg, long uptimeMillis) {
        msg.target = this;
        if (mAsynchronous) {
             msg.setAsynchronous(true);
        }
        return queue.enqueueMessage(msg, uptimeMillis);//调用消息队列的enqueueMessage()
  }
  ```

  

- **接收消息的过程**

  接收消息由`dispatchMessage(Message msg)`为入口

  *dispatchMessage()*

  ```java
  public void dispatchMessage(Message msg) {
         if (msg.callback != null) {
              handleCallback(msg);
          } else {
              if (mCallback != null) {
                  if (mCallback.handleMessage(msg)) {
                     return;
                  }
              }
              handleMessage(msg);
         }
   }
  ```

  这里的`callback`是我们调用 post(Runnable runnalbe) 时传入的`Runnable`对象，如果我们传入了`Runnable`对象

  就会执行`Runnable的`run方法:  

  ```java
  private static void handleCallback(Message message) {
          message.callback.run();
  }
  ```

  如果没有通过`post`传`Runnable`，就会看创建Handler时的构造方法中有没有传`Runnable`参数，传了的话由`mCallback`存储。

  这个`mCallback`是Handler内部的一个接口

  ```java
  public interface Callback {
          public boolean handleMessage(Message msg);
  }
  ```

  如果构造Handler时也没有传`Runnable`对象，最终会执行`handleMessage(msg)`,这个 方法就是我们创建handler时重写的handleMessage()方法.

  ![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/4/28/16a645eb4972d252~tplv-t2oaga2asx-image.image)









------

参考资料: [Android艺术开发探索](<https://book.douban.com/subject/26599538/>)

<https://www.cnblogs.com/luxiaoxun/p/8744826.html>





(完~)

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/4/28/16a645eb499d0c8f~tplv-t2oaga2asx-image.image)