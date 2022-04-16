---
title: "Kotlin Lamdba"
date: 2022-04-16T15:48:16+08:00
draft: true
tags: [Kotlin]
---


# Kotlin lambda  

Kotlin中，充斥着各种各样的`Lambda` 表达式，这是Kotlin最方便的特性之一

了解Kotlin 中的`lambda`，首先得知道Kotlin中的`高阶函数`  

---


## 1.高阶函数  


在Java中，如果有一个a方法，要去调用b方法，那么在里面直接调用即可。

```java
int a() {
  return b(1);
}
a();
```

接着,如果我不想把调用b方法的参数写死，希望动态设置方法b的参数。

```java
int a(int param) {
  return b(param);
}
a(1);
a(2);
```

这些在Java中很轻松就能做到，不过...
如果我们想动态设置的不是方法参数，而是方法本身呢，比如在方法a内有一处对别的方法的调用，这个方法可能是方法b,方法c,方法d...,该方法的参数类型是int,返回值类型也是int。方法a在执行时，具体需要调用哪个方法，能否动态设置? 也就是说能否将一个方法作为参数传给a?  



通过接口

```java
public interface Wrapper {
  int method(int param);
}
```

将这个接口类型`Wrapper` 作为方法a的参数
```java
int a(Wrapper wrapper) {
  return wrapper.method(1);
}
```

```java
a(wrapper1);
a(wrapper2);
```


举个我们常见的例子

在Android里View的点击事件
```java
public class View {
  OnClickListener mOnClickListener;
  ...
  public void onTouchEvent(MotionEvent e) {
    ...
    mOnClickListener.onClick(this);
    ...
  }
}
```

`OnClickListener`就是一个接口，点击事件的内容全在`onClick()`方法里  

```java
public interface OnClickListener {
  void onClick(View v);
}
```

我们在给View设置点击事件时，传的就是一个OnClickListener,本质上这样传递的是一个稍后会被调用的`onClick()`方法,一般称之为`回调`,不过由于Java中不允许直接传递方法，所以需要用接口包装一下。

```java
OnClickListener listener1 = new OnClickListener() {
  @Override
  void onClick(View v) {
    doSomething();
  }
};
view.setOnClickListener(listener1);
```



在Kotlin里，函数是可以作为一个参数被传递的 :  

```kotlin
fun setOnClickListener(onClick: (View) -> Unit) {
  this.onClick = onClick
}
                         👇传入一个匿名方法，参数类型时View,无返回值                        
view.setOnClickListener(fun(v: View): Unit) {
  doSomething()
})
```



这种 `参数或者返回值是函数`的函数 在Kotlin中称之为`高阶函数(Higher-Order-Functions)`   



## 2. Lambda表达式

在刚刚的代码中，传入的匿名函数还能进一步简化，写成`Lambda`表达式的形式

```kotlin
                         👇简化后的匿名方法                       
view.setOnClickListener( { v: View ->
  doSomething()
} )
```

Kotlin允许如果`Lambda` 是最后一个参数时，`Lambda`可以写在方法()的外面： 

```kotlin
view.setOnClickListener() { v: View ->
  doSomething()
}
```

如果该`Lambda` 是唯一的参数，则方法()可以省略

```kotlin
view.setOnClickListener { v: View ->
  doSomething()
}
```

进一步，如果该`Lambda` 只有一个参数，该参数还可以省略不写, Kotlin为这种唯一参数提供了默认的名字`it`

```kotlin
view.setOnClickListener { 
  doSomething()
  it.visibility = GONE
}
```



## 3. 为什么Kotlin中函数可以作为参数被传递?  

将刚才的代码，反编译为Java:  
```java
                                      👇实际传递的是FunctionN类型的对象
public static final String a(@NotNull Function1 funParam) {
      Intrinsics.checkNotNullParameter(funParam, "funParam");
      return (String)funParam.invoke(1);
}
```

```kotlin
interface FunctionN<out R> : Function<R>, FunctionBase<R> {

    operator fun invoke(vararg args: Any?): R

    override val arity: Int
}
```



> 所以，之所以Kotlin中方法可以被当作参数 进行传递，是因为它实际一个FunctionN类型的实实在在的对象



对应关系:  

| 表达式                           | FunctionN                        |
| ----------------------------- | -------------------------------- |
| (String) -> Unit              | Function1<String,Int>            |
| (String,Int) -> Double        | Function2<String,Int,Double>     |
| ...                           | ...                              |
| (参数1类型,参数2类型..参数n类型) -> 返回值类型 | FunctionN<参数1类型,...,参数n类型,返回值类型> |  


## 4.inline内联函数

集合操作符 filter 可以定义规则对集合进行过滤  
```kotlin
listOf(1,2,3,4,5,6).filter {it > 2}
```
看一下 filter 方法的声明
```kotiln
public inline fun <T> Iterable<T>.filter(predicate: (T) -> Boolean): List<T> {
    return filterTo(ArrayList<T>(), predicate)
}
```

```kotlin
public inline fun <T, C : MutableCollection<in T>> Iterable<T>.filterTo(destination: C, predicate: (T) -> Boolean): C {
    for (element in this) if (predicate(element)) destination.add(element)
    return destination
}
```  

filter方法接收一个 函数类型的参数，并且返回值是boolean，符合使用lambda的场景，上面我们说过，lamdba是实现了FunctionN接口的对象。
如果集合里有成百上千个元素，那每次经过filter 都会生成匿名对象，是一个不容忽视的开销,引起性能问题。
所以,Kotlin中引入了内联函数的概念:

> 内联函数回把函数的实现直接拷贝的调用处,避免创建匿名内部类。  


```kotlin
inline fun inlined(getString: () -> String) = println(getString()) 
 
fun notInlined(getString: () -> String) = println(getString())

fun test() {
    var testVar = "Test"

    notInlined { testVar }

    inlined { testVar }
}
```  

编译后的java代码:  

```java
public static final void test() {
    final ObjectRef testVar = new ObjectRef();
    testVar.element = "Test Variable";
    
    // notInlined:
    //每个闭包 lamdba 都是一个对象会额外分配内存，影响运行时间
    notInlined((Function0)(new Function0(0) {
        public Object invoke() {
            return this.invoke();
        }
    
        @NotNull
        public final String invoke() {
            return (String)testVar.element;
        }
    }));

    // inlined:会直接将方法体复制到调用处，
    String var3 = (String)testVar.element;
    System.out.println(var3);
}
```  

> 在 kotlin 中，因为出现了大量的带 lamdba的 高阶函数 -- 「高阶函数是将函数用作参数或返回值的函数」，使得越来越多的地方出现 函数参数 不断传递的现象，每一个函数参数都会被编译成一个对象， 使得内存分配（对于函数对象和类）和虚拟调用会增加运行时间开销，通过内联化 lambda 表达式可以消除这类的开销。

