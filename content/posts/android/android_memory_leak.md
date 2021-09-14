---
title: "Android内存泄漏探究"
date: 2021-09-14T15:26:58+08:00
---  


## 一.什么是内存泄漏

`内存泄漏`（英语：Memory leak）是在计算机科学中，由于疏忽或错误造成程序未能释放已经不再使用的内存。                                                                                                                 — — — 维基百科

![Untitled](https://tva1.sinaimg.cn/large/008i3skNly1gugdlcd1bij60i20b4wet02.jpg)

## 二.内存泄漏的影响

使得应用程序容易发生 `OOM`

- Android系统为每个应用分配的内存有限，若程序的发生的内存泄漏较多,会导致所需的内存超过系统所给的限额。

最终:OOM → Crash

![Untitled 1.png](http://ww1.sinaimg.cn/large/002Rm80Hly1gug71nxsrxj609l0gywgb02.jpg)

## 三. GC Roots

对于使用可达性分析的垃圾回收算法来说，GC roots是一个比较特别的存在，它用来帮助GC判断哪些对象可以被回收，如果这个对象被是GC roots 或者被GC roots引用就不会被回收。

![Untitled 2.png](http://ww1.sinaimg.cn/large/002Rm80Hly1gug71un4dgj61060j4tck02.jpg)

哪些可以作为GC roots?

- **Class :** 被系统ClassLoader加载的class,自定义的ClassLoader加载的class不是GC roots, 注意:静态变量是属于类的。
- Thread: 处于活动状态的线程.
- Stack Local: 方法中的变量和参数.
- JNI Local: JNI方法中的变量和参数.
- JNI Global: 全局的JNI reference,也就是JNI中全局创建的引用.
- Monitor Used: 同步的对象,例如被 `Synchronized`  锁住的对象.
- Held by JVM: 取决于各个JVM的实现,系统的ClassLoader和JVM本身会用到的一些对象.

在开发中最常见的就是方法中的变量作为GC roots,如下代码:

![Untitled 3.png](http://ww1.sinaimg.cn/large/002Rm80Hly1gug727nsgnj60w809sq6102.jpg)

我们创建了80M大小的内存区域并用变量 b指向这块区域，然后我们调用gc。

垃圾回收器首先进行了一次Minor GC，log中可以看到年轻代从3932K 降到了 560K，然后对象b转移到了老年代。

随后垃圾回收器又进行了一次Full GC,老年代中的内存从81.9M 降到 439K, 也就是我们创建的80M内存被正常回收了。

再看下面一段代码

![Untitled 4.png](http://ww1.sinaimg.cn/large/002Rm80Hly1gug72cmnapj60wf0c178002.jpg)

可以看到在Full GC时,分配的80M内存并没有被回收。

原因: ByteArray对象被作为GC roots的变量b持有，无法被回收

## 四.内存泄漏 in Android

内存泄漏的根本原因在于对象始终被GC roots引用，或者本身作为GC roots当不在使用时没有正确置空或释放。

持有Activity的GC roots的引用链如下图

![Untitled 5.png](http://ww1.sinaimg.cn/large/002Rm80Hly1gug72kz4v7j619c0tcgvm02.jpg)

Activity被置空的时机:

在onDestory()方法执行时,ActivityThread通过handleDestroyActivity()方法将 Activity置空

![Untitled 6.png](http://ww1.sinaimg.cn/large/002Rm80Hly1gug72r24vdj60yy1jc4em02.jpg)

那么Activity被引用者置空后，对GC roots就不可达，可以正常被回收。

![Untitled 7.png](http://ww1.sinaimg.cn/large/002Rm80Hly1gug757r0xdj60n20ergos02.jpg)

### 1.Handler使用不规范

内部类/匿名内部类 隐式持有外部类的引用

![Untitled 8.png](http://ww1.sinaimg.cn/large/002Rm80Hly1gug75fnp1fj61360zcal102.jpg)

![Untitled 9.png](http://ww1.sinaimg.cn/large/002Rm80Hly1gug75jx49xj61a613cgzm02.jpg)


Activity → Handler → Message → MessageQueue → Looper()，有Looper()存在于整个应用的生命周期，所以，如果Activity销毁时，Handler发送的消息还未得到Looper()处理，将导致Activity泄漏

![Untitled 10.png](http://ww1.sinaimg.cn/large/002Rm80Hly1gug75qiqg5j60lv0eignk02.jpg)


> 内部类静态化 + WeakReference

- 若垃圾收集器发现一个对象是弱可达的(weakly reachable)， 会清除所有指向该
对象的弱引用
1. 通过static将内部类作为独立于外部类的Class
2. 通过弱引用持有Activity,不耽误Activity回收

![Untitled 11.png](http://ww1.sinaimg.cn/large/002Rm80Hly1gug7hxhnawj60z00wcqdu02.jpg)

1. 及时移除未处理的消息以及callback

消除message对Handler的引用

![Untitled 12.png](http://ww1.sinaimg.cn/large/002Rm80Hly1gug7i4py4mj618w0vcwoo02.jpg)

### 2.匿名内部类

- AsyncTask
- Thread
- 匿名接口回调

匿名内部类与内部类都会隐式持有外部类引用

### 3.单例模式

单例模式的生命周期 = 整个app运行时,

若需要引用context，请使用applicationContext

### 4.注册的监听器未注销

- EventBus

```java
class MainActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        EventBus.getInstance().register(this)
    }
    
}
```

在Activity销毁时需unregister(this)

- SensorManger

```java
void registerListener() {
       SensorManager sensorManager = (SensorManager) getSystemService(SENSOR_SERVICE);
       Sensor sensor = sensorManager.getDefaultSensor(Sensor.TYPE_ALL);
       sensorManager.registerListener(this, sensor, SensorManager.SENSOR_DELAY_FASTEST);
}

View smButton = findViewById(R.id.sm_button);
smButton.setOnClickListener(new View.OnClickListener() {
    @Override public void onClick(View v) {
        registerListener();
        nextActivity();
    }
});
```

在Activity销毁时需SensorManger.unregisterListener(this)


- ContentObserver

    ```java
    getContentResolver().registerContentObserver(
                SOME_URI, 
                true, 
                yourObserver);

    getContentResolver().unregisterContentObserver(yourObserver);
    ```

### 5. 资源对象未及时关闭

- file
- cursor
- bitmap
- ...

## 六.内存泄漏分析工具



- Android Studio Monitor

- LeakCanary

![Untitled 13.png](http://ww1.sinaimg.cn/large/002Rm80Hly1gug7iaq883j621w1yc7wh02.jpg)


