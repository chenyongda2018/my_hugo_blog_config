
---
title: "JVM内存分配机制与回收策略选择 JVM学习笔记[2]"
date: 2020-04-13T20:22:27+08:00
draft: false
toc: false
comments: false
categories:
- Java
tags:
- Java
- JVM
---



> Java体系中的自动内存管理主要包括了2个方面:
> 1. 自动地给对象分配内存。
> 1. 自动地回收分配给对象地内存。

本文也围绕这两个点展开
<a name="lEFV6"></a>
## 一. 内存分配规则

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/6/24/16b8975973c1114c~tplv-t2oaga2asx-image.image)

<a name="mTE4x"></a>
### 1.优先在Eden区分配
大多数情况下，JVM会在 `Eden` 区优先分配对象，如果 `Eden` 没有足够的空间，则进行一次 `Minor GC` 。通过参数 `-XX:+PrintGCDetails` 可以让虚拟机在进行垃圾回收时打印日志，方便我们看到回收前后的内存占用情况。


例: 假如现在内存大小指定如下:

- 新生代 ->10M
  - Eden区 -> 8M
  - from区 -> 1M
  - to 区 -> 1M
- 老年代 -> 10M

然后我们又先后在代码中创建4个对象:

```java
byte[] byte1 = new byte[2MB];
byte[] byte2 = new byte[2MB];
byte[] byte3 = new byte[2MB];
byte[] byte4 = new byte[4MB];
```


当创建完第三个对象后，Eden区已经用掉了6M的空间来存放 byte1,byte2,byte3 三个对象，再创建第四个对象时,Eden区加上一个from区已经放不下了，如先前所述，此时会触发一次 `MinorGC` ,将三个2MB的对象转移到老年代中，腾出Eden区的空间给 第四个对象。

所以，执行后的内存情况如下:

- 新生代 -> 10M 
  - Eden 区 -> 8M (剩余4M) 存放 byte4
  - from区  -> 1M
  - to区      -> 1M
- 老年代 -> 10M (剩余4M) 存放byte1,byte2,byte3。


<a name="afMtC"></a>
### 2.大对象直接进入老年代

通过 `-XX:PretentureSizeThreshold` 参数设置大于这个值的对象直接分配到老年代。

<a name="LLkbd"></a>
### 3.长期存活的对象进入老年代
> 怎么算是长期存活 ?


JVM给每个对象定义了一个 `对象年龄计数器` 。当对象一开始被分配到新生代Eden区，经过一次  `MniorGC` <br />后仍然存活，并且Survivor区能够容纳它，则此对象被转移到 `Survivor` 区，年龄变为1。<br />在 `Survivor` 区中的对象没熬过一次 `MniorGC` ,年龄就涨1，当年龄达到我们设定的年龄阈值(JVM默认设定15)时，就会进入老年代。15岁就已经步入老年....<br />年龄阈值可通过参数 `-XX:MaxTenuringThreshold = 指定值` 来设定。<br />

<a name="16QsE"></a>
#### 对对象年龄判定的优化

JVM中，如果**Survivor区**中的**相同年龄**的**所有对象的大小**总和大于**Survivor空间**的一半，那么这些同窗们就直接进入老年代。无须等到上面的年龄阈值。<br />
<br />

<a name="Hp1zy"></a>
## 二. 空间分配担保机制与回收策略(Mnior GC还是Full GC)
首先介绍一下 `Mnior GC`  和 `Full GC`  的区别:
> MniorGC :  发生在新生代的垃圾回收活动，由于新生代的Java对象 短命的特性，这种垃圾回收活动频繁，回收速度较快。(就像扫碎纸屑)
> Full GC : 发生在老年的垃圾回收活动，不过出现一次  Full GC,也会伴随着一次 MniorGC,由于老年代中的对象基本都是大对象，长命，所以Full GC的速度比Mnior GC 的速度慢10倍以上。(就像搬大石头)


新生代的垃圾回收算法采用的是 [复制算法](https://www.yuque.com/henudada/kir653/hl39eo#vxPl5) ,当进行一次 `Mnior GC` 时，会将新生代的活动区域( `Eden区` 和Survivor中的 `From区` )中的存活对象复制到 Survivor中的 `to区` ,如果 to 区的内存不足以放下这些对象，那么这时就需要老年代出马，进行分配担保机制,将放不下的对象放到老年代。<br />    所以，在进行 `Monior GC`  前，JVM会做以下流程的检查，以确认老年代是否能够放得下那些对象，来选择进行 ` Mnior GC`  还是  `Full GC` 。

![image.png](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/6/24/16b8975970bb9584~tplv-t2oaga2asx-image.image)







_Reference:_<br />[深入理解Java虚拟机](https://book.douban.com/subject/24722612/) 

![image.png](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/6/24/16b897598641ce69~tplv-t2oaga2asx-image.image)
