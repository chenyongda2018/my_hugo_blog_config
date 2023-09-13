
---
title: "对象回收判定与垃圾回收算法 JVM学习笔记[1]"
date: 2020-01-13T20:21:09+08:00
draft: false
toc: false
comments: false
categories:
- Java
tags:
- Java
- JVM
---


<a name="p3BkG"></a>
# 本章要探究的问题 :

GC在回收内存时 :

1. 怎么判断哪些内存需要回收 ？
1. 什么时候回收？

 

在几个线程私有的运行时区域:<br />![image.png](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/6/17/16b64d8a41ca08fb~tplv-t2oaga2asx-image.image)

- 虚拟机栈
- 程序计数器
- 本地方法栈

它们的内存分配和回收**大多**都具有确定性，随着线程的创建而产生，随着线程的停止而被回收。栈帧中的内存大小基本在类的结构确定下来时就已知。

而在线程共有的 `Java堆(Heap)` 和 `方法区(Class(Method) Area)` 这两个区域则不同：

![image.png](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/6/17/16b64d8a42c57ecf~tplv-t2oaga2asx-image.image)


比如，一个接口有不同的实现类(类的信息在方法区中)，这几个实现类的内存大小肯定不一，没法在运行前就已知需要多大的内存，只有在运行期间才知道创建的对象的大小。


---

<a name="WasDg"></a>
## 
<a name="uulb2"></a>
## 一，哪些内存需要回收？

在知道哪些内存需要回收之前，我们要知道怎么判断一个对象是否还存活，当它不再存活时，就回收它。<br />而 `引用计数算法` 就是用来判断对象是否存活的一个算法。


<a name="NdERZ"></a>
### 1，引用计数算法（Reference Counting）

算法描述：给对象添加一个引用计数器，当有一个地方引用了它，计数器+1，当引用失效，计数器-1，在任何时刻，计数器为0时此对象将不能再被使用。

引用计数法在大多数情况下表现都不错，也有被很多公司采用的应用案例。但是在JVM中并没有采用这种算法，原因是：**无法解决对象之间存在相互引用的问题**。

```java
public class Person {
    Object instance = null;

    public static void main(String[] args) {
        Person a = new Person();
        Person b = new Person();
        
        a.instance = b;
        b.instance = a;
        
        a = null;
        b = null;// 正常情况下在这里GC就会把a,b回收掉
    }
}
```

正常情况下在执行11-12行代码时，JVM的GC会把a,b两个对象回收，但是在引用计数算法的情况下：

- 执行 `a=null` 时,a的引用计数器值为1，因为b对象在引用它。
- 执行 `b=null` 时,b的引用计数器值为1，因为a对象在引用它。

<a name="viwVk"></a>
### 2，可达性分析(Reachability Analysis)算法

在Java语言中是通过可达性分析来判断对象是否存活。  <br />算法描述 : 通过一系列的 `GC Roots` 作为起始点，从这些起始点开始向下搜索，能搜索的到的对象说明其可用，不会被GC回收掉，搜索所走过的路径称为 `引用链(Reference Chain)` 。相反，如果一个对象没有到达GC Roots的路径，则说明它不可用，被判定为可被GC回收的对象。

![image.png](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/6/17/16b64d8a4381d473~tplv-t2oaga2asx-image.image) 

如图 : 1区域的对象虽然互相关联，但是它们不可到达GC Roots,所以他们会被回收掉，而2区域的对象与GC Roots之间是有可到达路径的，所以它们不会被回收。 

<a name="yd7PO"></a>
#### 什么是GC Roots ? 

- 虚拟机栈(栈帧中的本地变量表)中引用的对象
- 方法区中类的静态属性引用的对象
- 方法区中常量引用的对象
- 本地方法栈中JNI(Native方法)引用的对象

这些都可作为GC Roots.


<a name="AJIwj"></a>
### 3，什么是引用（Reference）
我们在上面的 `引用计数算法` 和 `可达性分析` 中，都提到了 对象之间的`引用` 关系。

在Java1.2之前,关于 `引用` 的定义 : 
> 如果 `reference` 类型的数据存储的数值代表的是另一块内存的起始地址，就说这块内存代表一个引用。


JDK1,2之后,又引入了 `强引用` ， `软引用` ， `弱引用` ， `虚引用` ,这四个概念，并且这四种表现的引用关系越来越弱。

- 强引用(Strong Reference) :


例:
```
Object o = new Object();
```
 <br />只要强引用还在，GC永远不会回收掉被引用的对象。

- 软引用(Soft Reference) : 

有用，但非必须，在将要发生内存溢出时，会把 `软引用` 的对象回收掉，如果内存依然不够用，则抛出OOM异常。

- 弱引用(Weak Reference):

非必需对象，只要GC发生了垃圾回收，不管此时内存是否充足， `弱引用` 的对象都会被回收掉。

- 虚引用(Phantom Reference):
  - 最弱的引用关系
  - 无法通过虚引用构造市实例。
  - 唯一的作用就是在虚引用关联的对象被GC回收掉时，可以接受到一个信号。



<a name="fyTai"></a>
### 4，如何判断一个对象可回收(已死)？
一个对象仅仅通过上面说的可达性分析看它没有与GC ROOTS关联来判定这个对象是否可被回收是不够的。

一个对象要经过下面一段判断过程来判断它是否要被回收(建议收藏(*^__^*) 嘻嘻……):

![image.png](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/6/17/16b64d8a43f0e816~tplv-t2oaga2asx-image.image)
<a name="SNeCq"></a>
## 
<a name="jbN1k"></a>
### 5，方法区的回收
上面我们说的是存在于Java堆中的对象的回收，但其实在方法区还要回收以下东西：

<a name="4Zsz2"></a>
#### ① 回收废弃常量
假如常量池中有一个字符串 "abc" ,但是系统中没有一个String 对象指向它，也就是这个常量没有被引用，<br />当GC在回收时会回收此字面量。

<a name="nYR2X"></a>
#### ② 回收废弃的类(无用的类)

1. 该类的实例都已被回收，Java堆中不存在任何该类的实例。
1. 加载该类的ClassLoader已经被回收。
1. 该类的Java.lang.class对象没有被引用(在反射中会被用到这点)。

<a name="wKctE"></a>
#### ③ 方法区的回收策略:
GC在回收方法区时会采用一下2种方式:

- 标记-整理
- 标记-清除

---

<a name="Yy3DP"></a>
<br>
## 二，如何回收？

GC在回收内存时会采用多种垃圾收集算法，这些算法各有优劣。

<a name="KzuKK"></a>
### 1.标记-清除(Mark-Sweep)算法

此算法是最基础也是最古老的垃圾回收算法，该算法主要经过2个过程
<a name="kJmgg"></a>
#### ① 算法描述

1. 标记阶段:经过[如何判断一个对象可被回收](https://www.yuque.com/henudada/kir653/hl39eo#fyTai)所述，对可被回收的对象进行标记。
1. 清除阶段:将被标记的对象统一回收。

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/6/17/16b64d8a44d49bd2~tplv-t2oaga2asx-image.image)

<a name="6DmOn"></a>
#### ② 算法缺陷

1. 效率问题:此种算法标记和清除的效率都不高。
1. 标记清除后产生大量不连续的内存空间碎片。


<a name="vxPl5"></a>
### 2.复制(Copying)算法

复制算法针对效率问题进行了优化，它将内存区域划分为2块，每次只使用其中一块。

- 活动区域
- 空闲区域

<a name="16w2x"></a>
#### ① 算法描述
![image.png](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/6/17/16b64d8a46f2181b~tplv-t2oaga2asx-image.image)

如图:

- 回收前 : 内存被划分为左右两侧区域，右侧为空闲区域，暂时不使用它
- 回收时 : 将左侧要被回收的部分(黑色) 回收掉，然后将4个存活对象(淡灰色)移动到右侧的空闲区域，并且做了2件事
  - 将移动到空闲区域的存活对象按内存地址进行排列。
  - 将存活对象指向的旧地址指向新内存地址。
- 回收后 : 原先的右侧空闲区域变为活动区域，左侧的活动区域变为空闲区域。

左右两侧的区域状态在每一次回收后都来回转换...


<a name="iL3q8"></a>
#### ② 算法缺陷

1. 很显然，这种算法浪费了一般内存。
1. 当活动区域的100%的对象都还在活跃，那么在回收时需要将全部的对象复制到右侧的空闲区域，此时的效率就很低。

<a name="u5Sfy"></a>
#### ③ 算法应用
IBM公司经研究表明，Java堆新生代种的对象98%是 '朝生夕死' 的对象，比如临时变量等作用域很少的对象。<br />所以现在的虚拟机并不会按照 1:1的比例划分两个区域。

![image.png](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/6/17/16b64d8a9103bd34~tplv-t2oaga2asx-image.image)

现在的JVM虚拟机中，将新生代划分为一块 `Eden` 区域，和2块较小的 `Survivor` 区域(from ,to区)。<br />每次使用Eden区和1块Survivor区(from区)最为活动区域，当发生内存回收时，将这2块内存中的存活对象复制到另一块Survivor区(to区)。

在 `HotSpot` 虚拟机中，Eden区和Survivor的划分是: 8: 1，这样，活动区域占新生代的 (8+1)/10 *100% = 90%，只有10%的内存浪费。

**老年代** : 当将存活对象从活动区域(Eden,from) 复制到 to区时，如果to区不够用，则将剩下的存活对象放到**老年代。**<br />**<br />**
<a name="UPb85"></a>
### 3.标记-整理算法

标记-整理主要运用于老年代中。

<a name="6VV9R"></a>
#### ① 算法描述 

![image.png](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/6/17/16b64d8aa34e6440~tplv-t2oaga2asx-image.image)

此算法与**标记-清除算法**类似，也是经历2个阶段:

1. 标记阶段:此阶段于标记-清除中的标记阶段相同，都是标记出要回收的对象。
1. 整理阶段:把所有存活的对像，按内存地址排列移动到内存区域的一端，将端边界以外的区域进行回收。

<a name="BfH5m"></a>
#### ② 算法应用
由于老年代的特点，对象的存活率较高，没有额外的空闲区域，所以 老年代适用 标记-清除和标记-整理算法。


<br>
<br>
<br>
<br>
<br>
<br>
<br>
(完~)

_**Reference**:_<br />[深入理解Java虚拟机](https://book.douban.com/subject/24722612/) 

![image.png](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/6/17/16b64d8aa7475751~tplv-t2oaga2asx-image.image)