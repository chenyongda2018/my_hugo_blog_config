---
title: "Glide学习笔记"
date: 2019-11-14T20:13:19+08:00
tags:
- Android
- Glide
toc: true
categories:
- Android
---  

# 一.使用

## 1.配置依赖
在 `app` 层 `build.gradle` 中添加依赖 : 

```xml
dependencies {
    ...
    implementation 'com.github.bumptech.glide:glide:4.9.0'
    annotationProcessor 'com.github.bumptech.glide:compiler:4.9.0'
    annotationProcessor 'androidx.annotation:annotation:1.1.0'
    ...
}
```

添加必要权限 : 

```xml
<uses-permission android:name="android.permission.INTERNET"/>
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
```


## 2.初步上手

代码 : 

```java
Glide.with(context) 
    .asBitmap() //把动图当作静止图片处理,也就是只显示gif图的第一帧
    .load("https://pic2.zhimg.com/v2-c4970ee756c55333b7b871c5b617d9ed_b.gif")//图片url
    .placeholder(R.mipmap.ic_launcher)  //占位符,在加载图片完成之前显示的图片
    .error(R.drawable.ic_launcher_background) //加载失败时显示的图片
    .fitCenter() //当图片长宽大于ImageView时,缩放图片
    .fallback(R.mipmap.ic_launcher) //图片url为Null时显示的图片
    .into(image1); // 放入到ImageView中
```

效果图 :

<img src="http://ww1.sinaimg.cn/large/9c347cably1g8va6421ggj20dc0npq3i.jpg" alt="image.png" style="zoom:67%;" />



🎈Tips : fitCenter()与CenterCrop()区别 : 

- fitCenter() : 效果如上图,缩放原图至ImageView大小。
- CenterCrop():如下图,保持原图不变，只根据ImageView大小显示对应部分。

<img src="http://ww1.sinaimg.cn/large/9c347cably1g8va7anfvcj20dc0npab0.jpg" alt="image.png" style="zoom:67%;" />


## 3.配合RecyclerView显示图片列表

1.需要加载的图片url数据源

```java
String[] mImageList = {
            "https://gss0.bdstatic.com/94o3dSag_xI4khGkpoWK1HF6hhy/baike/crop%3D0%2C231%2C438%" +
                    "2C219%3BeWH%3D800%2C400/sign=b9c61bb42b3fb80e189e3b970be1031c/d50735fae6cd7b89267e2d06052442a7d9330e20.jpg",
            "https://s3.ifanr.com/wp-content/uploads/2018/02/https_2F2Fhk.hypebeast.com2Ffiles2F20182F012Fdragon-ball-super-ending-anime-details-01-1.jpg",
            "https://gss2.bdstatic.com/-fo3dSag_xI4khGkpoWK1HF6hhy/baike/s%3D220/sign=999dccfe8082b90139adc431438ca97e/a1ec08fa513d26970f8fef845ffbb2fb4216d88f.jpg",
            "http://img5.mtime.cn/CMS/News/2019/05/28/223631.52706895_620X620.jpg",
            "http://newsimg.5054399.com/uploads/userup/1902/2F95FQ953.jpg",
            "http://x0.ifengimg.com/cmpp/2019_37/63ef72f4f79be26_size61_w1278_h716.jpg",
            "https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcQJehCL30Zj-jAI8xxZ1eQI6mf9DuqaC6LXEOzF-Li8-Y1YlSnJ&s",
            "https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcRTzaN89LyHy1IzuibIhUdvJjEiCdrn8R84BnUi4O6hlPtmJZWe&s",
            "https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcQGFNu4QL1YIazFeV-hVtIv686UNFqJwjReiObeCmAZiFvl7CHJ&s",
            "https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcQyMlTlY90LK_A_nbv8egC6yZkhvd4WmAHOoMzwVuGaH8VJKQga&s",
            "https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcSNfpqnbeGqjIObnVpLipvUve0QGjT3PANsVoAEsKSvvmFdozMD&s",
            "https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcTjTT47tCkv6wy93-JIno1Egv-0QQQ5rvKmSTZ9gZ5Jd64VPY2o&s",
            "https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcQGFNu4QL1YIazFeV-hVtIv686UNFqJwjReiObeCmAZiFvl7CHJ&s",
            };
```

2.item的布局

```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="wrap_content">

    <ImageView
        android:id="@+id/imageView"
        android:layout_width="wrap_content"
        android:layout_height="150dp"
        android:layout_margin="10dp"
        android:layout_weight="1"
        tools:srcCompat="@tools:sample/avatars" />
</LinearLayout>
```

3.adapter

```java
public class ImageListAdapter extends RecyclerView.Adapter<ImageListAdapter.ViewHolder> {

    private Context mContext;
    private String[] mImageList;
    private LayoutInflater mInflater;

    RequestOptions mOptions = new RequestOptions()
            .placeholder(R.mipmap.ic_launcher)
            .error(R.drawable.ic_launcher_background);

    public ImageListAdapter(Context mContext, String[] mImageList) {
        this.mContext = mContext;
        this.mImageList = mImageList;
        mInflater = LayoutInflater.from(mContext);
    }

    public class ViewHolder extends RecyclerView.ViewHolder {
        public ImageView imageView;

        public ViewHolder(@NonNull View itemView) {
            super(itemView);
            imageView = (ImageView) itemView.findViewById(R.id.imageView);
        }
    }


    @NonNull
    @Override
    public ImageListAdapter.ViewHolder onCreateViewHolder(@NonNull ViewGroup parent, int viewType) {
       View view = mInflater.inflate(R.layout.main_activity_list_item,parent,false);
       return new ViewHolder(view);
    }

    @Override
    public void onBindViewHolder(@NonNull ImageListAdapter.ViewHolder holder, int position) {
        Glide.with(mContext)
                .load(mImageList[position])
                .apply(mOptions)
                .into(holder.imageView);
    }

    @Override
    public int getItemCount() {
        return mImageList.length;
    }
}
```

4. 在activity中绑定Recyclerview和数据源

```java
RecyclerView.LayoutManager linearLayoutManager = new LinearLayoutManager(getApplicationContext());
ImageListAdapter adapter = new ImageListAdapter(this,mImageList);

mRecyclerView = findViewById(R.id.recyclerView);
mRecyclerView.setLayoutManager(linearLayoutManager);
mRecyclerView.setAdapter(adapter);
```

效果图:

<img src="https://tva1.sinaimg.cn/large/008i3skNly1gugf8e90stj60dc0npjrl02.jpg" alt="Video_20191106_024957_705.gif" style="zoom:67%;" />


## 5.缓存

- 内存缓存 : 

通过 skipMemoryCache(true)可以设置不进行内存缓存     




- 磁盘缓存 : 

diskCacheStrategy(DiskCacheStrategy)
参数中有四个选项 : 


  - DiskCacheStrategy.ALL : 既缓存原图也缓存处理图
  - DiskCacheStrategy.NONE : 啥都不缓存 = 禁用磁盘缓存
  - DiskCacheStrategy.SOURCE : 只缓存原图
  - DiskCacheStrategy.RESULT : 只缓存处理图

- BitmapPool : 避免反复创建同样尺寸的bitmap而维护的一个bitmap池.

🎈Tips :Glide的读取缓存顺序是 : 先从内存中读取缓存,若没有再从磁盘中读取,若没有再从load中加载。

- 设置Glide只从缓存中加载图片，否则加载失败。

```java
Glide.with(fragment)
  .load(url)
  .onlyRetrieveFromCache(true)
  .into(imageView);
```


- 清理内存缓存

```java
Glide.get(context).clearMemory(); //需要在主线程运行
```

- 清理磁盘缓存(需要在子线程运行)

```java
new AsyncTask<Void, Void, Void> {
  @Override
  protected Void doInBackground(Void... params) {
    Glide.get(applicationContext).clearDiskCache();
    return null;
  }
}
```


- Glide对资源的重用机制

  

Glide通过 `引用计数法` 来判断一个资源是否正在被使用 :

  - 当一个资源通过 `into()` 方法加载时,它的 `引用计数` 就会+1
  - 当承载资源A的 `View` 或者 `Target` 调用了 `clear()` 或者又调用了 `into()` 加载了别的资源时, A资源的 `引用计数` -1。

当一个资源的 计数为0时，就会被释放，Glide可以重用该资源。





## 6.使用缩略图(Thumbnail)

1.当你已经有图片的缩略图的资源时,可以直接用url/uri加载

```java
GlideApp.with(mContext)
    .asDrawable()
    .load(原图url)
    .thumbnail(Glide.with(mContext)
               .load(缩略图url))
    .into(holder.imageView);
```

2.没有缩略图的资源时，也可以直接传入想要缩小的倍数

```java
Glide.with(mContext)
    .asDrawable()
    .load(mImageList[position])
    .thumbnail(0.1f)  //缩略图大小为原来的 1/10
    .into(holder.imageView);
```


## 7.Target

Target对象用来保存每一个Glide请求。如:

```java
Target<Drawable> target = Glide.with(mContext)
    .asDrawable()
    .load(R.drawable.ic_launcher_foreground)
    .into(holder.imageView);
```

这样做的目的是可以有效的对glide请求进行管理，取消请求，清理资源。

- 清理之前的请求 : 

```java
Target<Drawable> target = Glide.with(mContext)
    .asDrawable()
    .load(R.drawable.ic_launcher_foreground)
    .into(holder.imageView);

Glide.with(fragment).clear(target);     //清理之前的加载,节省资源
```

 当使用这个target再发出新的加载请求时,glide也会对上一次的加载进行清理，释放资源。

```java
Target<Drawable> target = Glide.with(mContext)
    .asDrawable()
    .load(R.drawable.ic_launcher_foreground)
    .into(holder.imageView);

Glide.with(mContext)
    .load(mImageList[position])
    .fitCenter()
    .into(target);
```



 

## 8.过渡动画(Transitions)

根据官方文档描述 , 过渡动画只能作用于一个请求中的(不能作用于先后2个不同的请求) : 

- 从占位符->原图的过程
- 从缩略图->原图的过程

示例:
可以发现占位图到原图之间的渐变效果
<img src="http://ww1.sinaimg.cn/large/9c347cably1g8vaat8gfrg20c00lchdv.gif" alt="Video_20191108_030848_174.gif" style="zoom:67%;" />

```java
Glide.with(mContext)
    .load(mImageList[position])
    .transition(withCrossFade(600))  //设置渐变的持续时长600ms
    .placeholder(R.drawable.ic_launcher_foreground)
    .centerCrop()
    .into(holder.imageView);
```


## 9.自定义Module

通过自定义module可以方便我们根据项目的实际需求来定制化glide。

1. 首先新建一个类继承 `AppGlideModule` ,并用 `@GlideModule` 注解:

```java
/**
 * 通过继承AppGlideModule来自定义Glide
 * @author chenyongda
 */
@GlideModule
public class MyGlideApp extends AppGlideModule {
}
```

2. 再重新 `ReBuild` 一下项目 ,将代码中之前的 `Glide.with()` 换成 `GlideApp.with()` :

```java
GlideApp.with(mContext)
    .load(mImageList[position])
    .placeholder(R.mipmap.ic_launcher)
    .error(R.drawable.ic_launcher_background)
    .into(holder.imageView);
```

3. 在applyIOptions()中进行自定义配置:

比如,我们想让glide默认情况下不进行任何缓存。

```java
 @Override
public void applyOptions(@NonNull Context context, @NonNull GlideBuilder builder) {
    //不缓存图片
    RequestOptions options = new RequestOptions()
        //不进行内存缓存
        .skipMemoryCache(true)
        //不进行磁盘缓存
        .diskCacheStrategy(DiskCacheStrategy.NONE);
    builder.setDefaultRequestOptions(options); //设置为默认配置
}
```


在 `applyOptions()` 函数中还可以对缓存根据需要进行自定义

自定义内存缓存：

最大缓存20m的图片
```java
@Override
  public void applyOptions(Context context, GlideBuilder builder) {
    int memoryCacheSizeBytes = 1024 * 1024 * 20; // 20mb
    builder.setMemoryCache(new LruResourceCache(memoryCacheSizeBytes));
  }
```






# 二.源码解析

## 1.Glide.with() : Glide的生命周期管理

进入Glide.with()方法 :

```java
@NonNull
  public static RequestManager with(@NonNull Context context) {
    return getRetriever(context).get(context);
  }
```

首先看一下 `getRetriever()` 方法:

```java
@NonNull
  private static RequestManagerRetriever getRetriever(@Nullable Context context) {
    // Context could be null for other reasons (ie the user passes in null), but in practice it will
    // only occur due to errors with the Fragment lifecycle.
    Preconditions.checkNotNull(
        context,
        "You cannot start a load on a not yet attached View or a Fragment where getActivity() "
            + "returns null (which usually occurs when getActivity() is called before the Fragment "
            + "is attached or after the Fragment is destroyed).");
    return Glide.get(context).getRequestManagerRetriever();
  }
```

通过注释了解到 :这个方法先对context进行判空,也就是当view还没加载时或者fragment还没与activity建立联系时。

看一下 `Glide.get(context)` 方法 : 

```java
  /**
   * Get the singleton.
   *
   * @return the singleton
   */
  @NonNull
  public static Glide get(@NonNull Context context) {
    if (glide == null) {
      synchronized (Glide.class) {
        if (glide == null) {
          checkAndInitializeGlide(context);
        }
      }
    }

    return glide;
  }
```

这是一个DCL双重检验锁的单例模式 。

`checkAndInitializeGlide()` 方法 :  

```java
private static void checkAndInitializeGlide(@NonNull Context context) {
    // In the thread running initGlide(), one or more classes may call Glide.get(context).
    // Without this check, those calls could trigger infinite recursion.
    if (isInitializing) {
        throw new IllegalStateException("You cannot call Glide.get() in registerComponents(),"
                                        + " use the provided Glide instance instead");
    }
    isInitializing = true;
    initializeGlide(context);
    isInitializing = false;
}
```

这个方法用来保证glide在同一时刻只能有一个线程在初始化它。

RequestManagerRetriever : 
这个类是 Glide 的一个辅助类，用来创建一个新的 `RequestManager` 对象或者获得一个已存在的 acitivity/fragment , 那 `RequestManager`将请求与activity/fragment等组件建立联系。

再看一下 RequestMangerRetriever类的get()方法 : 

```java
  @NonNull
  public RequestManager get(@NonNull Context context) {
    if (context == null) {
      throw new IllegalArgumentException("You cannot start a load on a null Context");
    } else if (Util.isOnMainThread() && !(context instanceof Application)) {
      if (context instanceof FragmentActivity) {
        return get((FragmentActivity) context);
      } else if (context instanceof Activity) {
        return get((Activity) context);
      } else if (context instanceof ContextWrapper) {
        return get(((ContextWrapper) context).getBaseContext());
      }
    }

    return getApplicationManager(context);
  }
```

这个方法通过将我们传入的 context ，转换为与 application/activity/fragment建立联系的 `ReuqestManger` 类 ,加入我们传入的是 activity,点进第 9 行代码 :

```java
 @NonNull
  public RequestManager get(@NonNull Activity activity) {
    if (Util.isOnBackgroundThread()) {
      //当我们在子线程用glide,glide会把请求与整个app的生命周期建立联系.
      return get(activity.getApplicationContext());
    } else {
      assertNotDestroyed(activity);
      //用于创建fragment
      android.app.FragmentManager fm = activity.getFragmentManager(); 
      return fragmentGet(
          activity, fm, /*parentHint=*/ null, isActivityVisible(activity));
    }
  }
```

点进第8行的 `fragmentGet()` :

```java
private RequestManager fragmentGet(@NonNull Context context,
      @NonNull android.app.FragmentManager fm,
      @Nullable android.app.Fragment parentHint,
      boolean isParentVisible) {
    //创建一个隐藏的Fragment
    RequestManagerFragment current = getRequestManagerFragment(fm, parentHint, isParentVisible);
    RequestManager requestManager = current.getRequestManager();
    //如果获得的RequestManager为空的话就创建一个。
    if (requestManager == null) {
      Glide glide = Glide.get(context);
      requestManager =
          factory.build(
              glide, current.getGlideLifecycle(), current.getRequestManagerTreeNode(), context);
      	//把RequestManager赋给Fragment
        current.setRequestManager(requestManager);
    }
    return requestManager;
  }
```

到这里可以发现 Glide与activity建立联系的关键在于:

1. 创建了一个无UI的Fragment 
1. 创建了一个 `RequestManager` ,并通过 `current.getGlideLifecycle()` 将fragment的 `ActivityFragmentLifecycle` 成员传给了 `RequestManager` 。
1. 无UI的Fragment通过 `setRequestManager()` 与 `RequestManager` 建立联系。

第12行代码中创建 `RequestManager` 对象时 调用了 `RequestManager` 的构造方法 : 

构造方法的22行调用了`ActivityFragmentLifecycle` 的 addListener()方法将 `RequestManager` 添加到自己的监听器列表中。

```java
  RequestManager(
      Glide glide,
      Lifecycle lifecycle,
      RequestManagerTreeNode treeNode,
      RequestTracker requestTracker,
      ConnectivityMonitorFactory factory,
      Context context) {
    this.glide = glide;
    this.lifecycle = lifecycle;
    this.treeNode = treeNode;
    this.requestTracker = requestTracker;
    this.context = context;

    connectivityMonitor =
        factory.build(
            context.getApplicationContext(),
            new RequestManagerConnectivityListener(requestTracker));

    if (Util.isOnBackgroundThread()) {
      mainHandler.post(addSelfToLifecycle);
    } else {
      //将 RequestManager 添加到自己的监听器列表中。
      lifecycle.addListener(this);
    }
    lifecycle.addListener(connectivityMonitor);

    defaultRequestListeners =
        new CopyOnWriteArrayList<>(glide.getGlideContext().getDefaultRequestListeners());
    setRequestOptions(glide.getGlideContext().getDefaultRequestOptions());

    glide.registerRequestManager(this);
  }
```

为了看清楚Glide是如何与组件activity/fragment建立生命周期联系，我们先从刚刚的无UI的Fragment->
`RequestManagerFragment` 这个类的生命周期看起 :

```java
public class RequestManagerFragment extends Fragment {
    
  private static final String TAG = "SupportRMFragment";
  private final ActivityFragmentLifecycle lifecycle;
    
  @Override
  public void onStart() {
    super.onStart();
    lifecycle.onStart();
  }

  @Override
  public void onStop() {
    super.onStop();
    lifecycle.onStop();
  }

  @Override
  public void onDestroy() {
    super.onDestroy();
    lifecycle.onDestroy();
    unregisterFragmentWithRoot();
  }
    	
}
```

这个fragment中的生命周期方法中,又调用了对应的 `ActivityFragmentLifecycle` 的方法 : 

```java
class ActivityFragmentLifecycle implements Lifecycle {
  
    private final Set<LifecycleListener> lifecycleListeners =
      Collections.newSetFromMap(new WeakHashMap<LifecycleListener, Boolean>());
  private boolean isStarted;
  private boolean isDestroyed
  
  @Override
  public void addListener(@NonNull LifecycleListener listener) {
    lifecycleListeners.add(listener);

    if (isDestroyed) {
      listener.onDestroy();
    } else if (isStarted) {
      listener.onStart();
    } else {
      listener.onStop();
    }
  }

  @Override
  public void removeListener(@NonNull LifecycleListener listener) {
    lifecycleListeners.remove(listener);
  }

  void onStart() {
    isStarted = true;
    for (LifecycleListener lifecycleListener : Util.getSnapshot(lifecycleListeners)) {
      lifecycleListener.onStart();
    }
  }

  void onStop() {
    isStarted = false;
    for (LifecycleListener lifecycleListener : Util.getSnapshot(lifecycleListeners)) {
      lifecycleListener.onStop();
    }
  }

  void onDestroy() {
    isDestroyed = true;
    for (LifecycleListener lifecycleListener : Util.getSnapshot(lifecycleListeners)) {
      lifecycleListener.onDestroy();
    }
  }      

}
```

`ActivityFragmentLifecycle` 中有一个 `Set<LifecycleListener>` 类型的集合,而 `RequestManager` 也正好实现了 `LifecycleListener` 这个接口 ,看一下 `RequestManager` 中的 `onStart()` 方法 : 

```java
  @Override
  public synchronized void onStart() {
    //处理请求
    resumeRequests();
    targetTracker.onStart();
  }

  @Override
  public synchronized void onStop() {
    pauseRequests();
    targetTracker.onStop();
  }

  public synchronized void resumeRequests() {
    requestTracker.resumeRequests();
  }

...
```

![](https://cdn.nlark.com/yuque/0/2019/webp/334982/1573460183383-c4cb601a-7b5e-4fa5-8f61-60ac75603667.webp#align=left&display=inline&height=238&originHeight=238&originWidth=1078&search=&size=0&status=done&width=1078)
所以现在可以看出,Glide与组件的生命周期相关联的方法是基于类似链式的结构。

1. Activity的生命周期会引起无UI的Fragment的生命周期变化.
1. Fragment的生命周期会会调用自己成员变量 `ActivityFragmentLifecycle` 中对应的onXXX()方法.
1. `ActivityFragmentLifecycle` 的onXXX()方法内部又调用了 `RequestManager` 的方法。


## 2.load(url) : 
在load()这一环节需要我们传入需要加载的图片的链接.

```java
  @Override
  public RequestBuilder<Drawable> load(@Nullable String string) {
    return asDrawable().load(string);
  }
```

load方法默认调用了 `asDrawable()` 方法 : 

方法主要作用是创建一个加载 `Drawable` 资源类型的 `RequestBuilder` 。
```java
  public RequestBuilder<Drawable> asDrawable() {
    return as(Drawable.class);
  }

  public <ResourceType> RequestBuilder<ResourceType> as(
      @NonNull Class<ResourceType> resourceClass) {
    return new RequestBuilder<>(glide, this, resourceClass, context);
  }
```

查看一下 `load()` 方法 : 

将String字符串赋予model变量
```java
  public RequestBuilder<TranscodeType> load(@Nullable String string) {
    return loadGeneric(string);
  }

  private RequestBuilder<TranscodeType> loadGeneric(@Nullable Object model) {
    this.model = model;
    isModelSet = true;//这个变量表示已经调用过load(),在into()中会再次用到此变量。
    return this;
  }
```


而这个model变量将在后面的 `into()` 中使用。


## 3.into()

```java
public ViewTarget<ImageView, TranscodeType> into(@NonNull ImageView view) {
    //判断当前是否是在主线程,因为这里涉及到改变UI
    Util.assertMainThread();
    //检查ImageView是否为空
    Preconditions.checkNotNull(view);

    BaseRequestOptions<?> requestOptions = this;
    if (!requestOptions.isTransformationSet()
        && requestOptions.isTransformationAllowed()
        && view.getScaleType() != null) {
      // Clone in this method so that if we use this RequestBuilder to load into a View and then
      // into a different target, we don't retain the transformation applied based on the previous
      // View's scale type.
      switch (view.getScaleType()) {
        case CENTER_CROP:
          requestOptions = requestOptions.clone().optionalCenterCrop();
          break;
        case CENTER_INSIDE:
          requestOptions = requestOptions.clone().optionalCenterInside();
          break;
        case FIT_CENTER:
        case FIT_START:
        case FIT_END:
          requestOptions = requestOptions.clone().optionalFitCenter();
          break;
        case FIT_XY:
          requestOptions = requestOptions.clone().optionalCenterInside();
          break;
        case CENTER:
        case MATRIX:
        default:
          // Do nothing.
      }
    }

    return into(
        //由ImageView构建出Target
        glideContext.buildImageViewTarget(view, transcodeClass),
        /*targetListener=*/ null,
        requestOptions,
        Executors.mainThreadExecutor());
  }
```

这里的into()方法只是根据ImageView的比例类型，对 `RequestOptions` 进行配置。

36行的into()方法中用 `GlideContext` 的 `buildImageViewTarget()` 用 ImageView 构建出 Target.

看一下这个方法 : 

```java
  @NonNull
  public <X> ViewTarget<ImageView, X> buildImageViewTarget(
      @NonNull ImageView imageView, @NonNull Class<X> transcodeClass) {
    return imageViewTargetFactory.buildTarget(imageView, transcodeClass);
  }

public class ImageViewTargetFactory {
  @NonNull
  @SuppressWarnings("unchecked")
  public <Z> ViewTarget<ImageView, Z> buildTarget(@NonNull ImageView view,
      @NonNull Class<Z> clazz) {
    if (Bitmap.class.equals(clazz)) {      //Bitmap类型的ViewTarget
      return (ViewTarget<ImageView, Z>) new BitmapImageViewTarget(view);
    } else if (Drawable.class.isAssignableFrom(clazz)) {          //Drawable类型的ViewTarget
      return (ViewTarget<ImageView, Z>) new DrawableImageViewTarget(view);
    } else {
      throw new IllegalArgumentException(
          "Unhandled class: " + clazz + ", try .as*(Class).transcode(ResourceTranscoder)");
    }
  }
}
```

可以看出 `ImageViewTargetFactory` 这个类就是用来把 `ImageView` 转化为2种(Bitmap,Drawable) 类型的 ViewTarget的。

>  而我们之前在分析 `load()` 方法时看到默认情况下把资源当作 Drawable 类型处理 , 所以一般情况下都是生成的 `DrawableImageViewTarget` 。
>
> ```java
>   @Override
>   public RequestBuilder<Drawable> load(@Nullable String string) {
>     return asDrawable().load(string);
>   }
> ```
>
> 




继续深入之前的 into() 方法。


```java
  private <Y extends Target<TranscodeType>> Y into(
      @NonNull Y target, //把创建的Target传进来
      @Nullable RequestListener<TranscodeType> targetListener,
      BaseRequestOptions<?> options,
      Executor callbackExecutor) {
    //Target不能为空
    Preconditions.checkNotNull(target);
    // isModelSet在load()被调用过才会为true
    if (!isModelSet) {
      throw new IllegalArgumentException("You must call #load() before calling #into()");
    }
	//创建Request对象 , 稍后解析
    Request request = buildRequest(target, targetListener, options, callbackExecutor);
	//获取此ImageView的之前的请求对象
    Request previous = target.getRequest();
    
    //如果此次的请求和之前的请求相同并且启用了内存缓存
    if (request.isEquivalentTo(previous)
        && !isSkipMemoryCacheWithCompletePreviousRequest(options, previous)) {
      //回收请求资源
      request.recycle();
     //若上一次的请求没有在加载，则再执行这个请求
      if (!Preconditions.checkNotNull(previous).isRunning()) {
        
        previous.begin();
      }
      return target;
    }
	//若这次请求与上次不同,则取消上一次请求
    requestManager.clear(target);
    target.setRequest(request);
    requestManager.track(target, request);

    return target;
  }
```



