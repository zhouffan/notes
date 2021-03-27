### [1. ARouter之基础应用篇](https://www.jianshu.com/p/a33029557b1f)

```
// 初始化
// 调试模式不是必须开启，但是为了防止有用户开启了InstantRun，但是
 // 忘了开调试模式，导致无法使用Demo，如果使用了InstantRun，必须在
 // 初始化之前开启调试模式，但是上线前需要关闭，InstantRun仅用于开
 // 发阶段，线上开启调试模式有安全风险，可以使用BuildConfig.DEBUG
 // 来区分环境
 ARouter.init(getApplication());
```

```java
@Route(path = "/test/activity_jnject", name = "测试用 Activity")
public class ActivityJnject extends AppCompatActivity {
    @Autowired(desc = "姓名")
    String name = "jack";

    @Autowired
    int age = 10;
    @Autowired
    int height = 175;
    @Autowired(name = "boy", required = true)
    boolean girl;
    @Autowired
    char ch = 'A';
    @Autowired
    float fl = 12.00f;
    @Autowired
    double dou = 12.01d;
    @Autowired
    TestSerializable ser;
    @Autowired
    TestParcelable pac;
    @Autowired
    TestObj obj;
    @Autowired
    List<TestObj> objList;
    @Autowired
    Map<String, List<TestObj>> map;
    private long high;
    @Autowired
    String url;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_jnject);
        ARouter.getInstance().inject(this);

        String params = String.format(
                "name=%s,\n age=%s, \n height=%s,\n girl=%s,\n high=%s,\n url=%s,\n ser=%s,\n pac=%s,\n obj=%s \n ch=%s \n fl = %s, \n dou = %s, \n objList=%s, \n map=%s",
                name,age, height, girl, high,
                url,  ser, pac, obj, ch, fl,  dou, objList,  map
        );

        ((TextView) findViewById(R.id.test)).setText("I am " + ActivityJnject.class.getName());
        ((TextView) findViewById(R.id.test2)).setText(params);

    }
}
```

```dart
//调用部分
 TestSerializable testSerializable = new TestSerializable("Titanic", 555);
                TestParcelable testParcelable = new TestParcelable("jack", 666);
                TestObj testObj = new TestObj("Rose", 777);
                List<TestObj> objList = new ArrayList<>();
                objList.add(testObj);
                Map<String, List<TestObj>> map = new HashMap<>();
                map.put("testMap", objList);
                ARouter.getInstance().build("/test/activity_jnject")
//                ARouter.getInstance().build("/test/activity1")
                        .withString("name", "老王")
                        .withInt("age", 18)
                        .withBoolean("boy", true)
                        .withLong("high", 180)
                        .withString("url", "https://a.b.c")
                        .withSerializable("ser", testSerializable)
                        .withParcelable("pac", testParcelable)
                        .withObject("obj", testObj)
                        .withObject("objList", objList)
                        .withObject("map", map)
                        .navigation();
```

### [2.api与implementation的区别](https://www.jianshu.com/p/8962d6ba936e)

| 新配置                | 已弃用配置 | 行为                                                         |
| :-------------------- | :--------- | :----------------------------------------------------------- |
| `implementation`      | `compile`  | Gradle 会将依赖项添加到编译类路径，并将依赖项打包到构建输出。**不希望该模块在编译时将该依赖项泄露**给其他模块。也就是说，其他模块只有在运行时才能使用该依赖项。使用此依赖项配置代替 `api` 或 `compile`（已弃用）可以**显著缩短构建时间**，因为这样可以减少构建系统需要重新编译的模块数。例如，如果 `implementation` 依赖项更改了其 API，Gradle 只会重新编译该依赖项以及直接依赖于它的模块。大多数应用和测试模块都应使用此配置。 |
| `api`                 | `compile`  | Gradle 会将依赖项添加到编译类路径和构建输出。当一个模块包含 `api` 依赖项时，会让 Gradle 了解该模块要以**传递方式**将该依赖项**导出到其他模块**，以便这些模块在**运行时和编译时**都可以使用该依赖项。此配置的行为类似于 `compile`（现已弃用），但使用它时应格外小心，只能对您需要以传递方式导出到其他上游消费者的依赖项使用它。这是因为，如果 `api` 依赖项更改了其外部 API，Gradle 会在编译时重新编译所有有权访问该依赖项的模块。因此，拥有大量的 `api` 依赖项会显著**增加构建时间**。除非要将依赖项的 API 公开给单独的模块，否则库模块应改用 `implementation` 依赖项。 |
| `compileOnly`         | `provided` | Gradle **只会**将依赖项**添加到编译类路径**（也就是说，不会将其添加到构建输出）。如果您创建 Android 模块时在编译期间需要相应依赖项，但它在运行时可有可无，此配置会很有用。如果您使用此配置，那么您的库模块必须包含一个运行时条件，用于检查是否提供了相应依赖项，然后适当地改变该模块的行为，以使该模块在未提供相应依赖项的情况下仍可正常运行。这样做不会添加不重要的瞬时依赖项，因而有助于减小最终 APK 的大小。此配置的行为类似于 `provided`（现已弃用）。**注意**：您不能将 `compileOnly` 配置与 AAR 依赖项配合使用。 |
| `runtimeOnly`         | `apk`      | Gradle **只会**将依赖项**添加到构建输出**，以便在运行时使用。也就是说，不会将其添加到编译类路径。此配置的行为类似于 `apk`（现已弃用）。 |
| `annotationProcessor` | `compile`  | 如需添加对作为**注释处理器的库**的依赖关系，您必须使用 `annotationProcessor` 配置将其添加到注释处理器类路径。这是因为，使用此配置可以将编译类路径与注释处理器类路径分开，从而提高构建性能。如果 Gradle 在编译类路径上找到注释处理器，则会禁用[避免编译](https://docs.gradle.org/current/userguide/java_plugin.html#sec:java_compile_avoidance)功能，这样会对构建时间产生负面影响（Gradle 5.0 及更高版本会忽略在编译类路径上找到的注释处理器）。如果 JAR 文件包含以下文件，则 Android Gradle 插件会假定依赖项是注释处理器： `META-INF/services/javax.annotation.processing.Processor`。如果插件检测到编译类路径上包含注释处理器，则会生成构建错误。 |
| `lintChecks`          |            | 使用此配置可以添加您希望 Gradle 在构建项目时**执行的 lint 检查**。**注意**：使用 Android Gradle 插件 3.4.0 及更高版本时，此依赖项配置不再将 lint 检查打包在 Android 库项目中。如需将 lint 检查依赖项包含在 AAR 库中，请使用下面介绍的 `lintPublish` 配置。 |
| `lintPublish`         |            | 在 Android 库项目中使用此配置可以添加您希望 Gradle 编译成 `lint.jar` 文件并打包在 AAR 中的 lint 检查。这会使得使用 AAR 的项目也应用这些 lint 检查。如果您之前使用 `lintChecks` 依赖项配置将 lint 检查包含在已发布的 AAR 中，则需要迁移这些依赖项以改用 `lintPublish` 配置。`dependencies {  // Executes lint checks from the ':checks' project  // at build time.  lintChecks project(':checks')  // Compiles lint checks from the ':checks-to-publish'  // into a lint.jar file and publishes it to your  // Android library.  lintPublish project(':checks-to-publish') }` |

![img](https://i.loli.net/2019/04/19/5cb8a51ddc485.png)



- api：是对外的，就应该公开。**模块之间方法调用传递**
- implementation：内部的实现，**不应该公开出来，就近模块才能调用**。  编译耗时短
- compile： 与api一样，无法隐藏自己使用的依赖
- compileOnly： 同provided，**编译有效**，不参与打包
- runtimeOnly：同apk， 编译不参与，只在**打包**时有效
- testImplementation： 同testCompile，在**单元测试和打包测试apk**时有效
- debugImplementation： 同debugCompile，在**debug**下有效
- releaseImplementation: 同releaseComile，在**release模式和打包release**下有效

### [2 Android 反编译](https://www.jianshu.com/p/e89be5c05a60)

1. [apktool](https://links.jianshu.com/go?to=https%3A%2F%2Fbitbucket.org%2FiBotPeaches%2Fapktool%2Fdownloads%2F)  作用：把apk文件反编译，取出资源
2. [dex2jar](https://links.jianshu.com/go?to=http%3A%2F%2Fsourceforge.net%2Fprojects%2Fdex2jar%2Ffiles%2F)  作用：把存有java内容的dex文件反编译（classes.dex转化成jar文件）
3. [jd-gui](https://links.jianshu.com/go?to=http%3A%2F%2Fjd.benow.ca%2F)  作用：查看APK中classes.dex转化成出的jar文件，即源码文件

 

### [3. retrofit2](https://www.jianshu.com/p/360768def285)

[官网](https://square.github.io/retrofit/)

[Android：手把手带你 深入读懂 Retrofit 2.0 源码](https://www.jianshu.com/p/0c055ad46b6c)     

https://www.jianshu.com/p/360768def285



**Retrofit的Url组合规则**

| BaseUrl                              | 和URL有关的注解中提供的值 | 最后结果                                 |
| ------------------------------------ | ------------------------- | ---------------------------------------- |
| http://localhost:4567/path/to/other/ | /post                     | http://localhost:4567/post               |
| http://localhost:4567/path/to/other/ | post                      | http://localhost:4567/path/to/other/post |
| http://localhost:4567/path/to/other/ | https://github.com/ikidou | https://github.com/ikidou                |



![img](https://upload-images.jianshu.io/upload_images/21236288-d35bc5858893bd41.png?imageMogr2/auto-orient/strip|imageView2/2/w/1106/format/webp)



@url  : 如果不包含scheme和host，则使用baseUrl。 如果@url值是“/xxxxxxx” ，斜线开头，只使用http:ip:port/   。[后面则路径则不会使用](https://blog.csdn.net/weijinqian0/article/details/78082925)

- **一、注解静态添加方式**

  ```
      @Headers({"Content_Type:application/json", "charset:UTF-8"})
      @POST("/config/getvalue")
      fun getConfig(@Body req: ConfigReq): Observable<BaseRespone<ConfigRes>>
  ```

- **二、注解动态添加方式**

  ```
  @POST("order/carry")
  fun takeOrder(
           @Header("groupID") groupID: String,
           @Body req: OperateOrderReq
      ): Observable<BaseRespone<Any>>
  ```

**三、通用拦截器方式**

```
//创建拦截器 
val interceptor = Interceptor { chain -> 
    val request = chain.request() 
    val requestBuilder = request.newBuilder() 
    val url = request.url() 
    val builder = url.newBuilder() 
    requestBuilder.url(builder.build()) 
        .method(request.method(), request.body()) 
        .addHeader("Content_Type", "application/json") 
        .addHeader("charset", "UTF-8") 
         chain.proceed(requestBuilder.build()) 
} 
//创建OKhttp 
val client = OkHttpClient.Builder() 
		.addInterceptor(interceptor) 
        .connectTimeout(10, TimeUnit.SECONDS) 
        .readTimeout(20, TimeUnit.SECONDS) 
        .retryOnConnectionFailure(false) 
        .build() 
//Retrofit实例化 
retrofit = Retrofit.Builder() 
    .baseUrl(BaseConstant.SERVICE_HOST) 
    .addConverterFactory(GsonConverterFactory.create()) 
    .addCallAdapterFactory(RxJava2CallAdapterFactory.create()) 
    .client(client)
    .build()
```





### 4. glide 图片加载

#### 4.1  [glide官方](https://muyangmin.github.io/glide-docs-cn/#glide-v4-android-english-tip)

- 当 [`Glide.with()`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/Glide.html#with-android.app.Fragment-) 中传入的 Activity 或 Fragment 实例销毁时，Glide 会自动取消加载并回收资源。

- 在 ListView 或 RecyclerView 中加载图片的代码和在单独的 View 中加载完全一样。Glide 已经自动处理了 View 的复用和请求的取消。对 url 进行 null 检验并不是必须的，如果 url 为 null，Glide 会清空 View 的内容

- 在Glide中，当你为一个 [ImageView](http://developer.android.com/reference/android/widget/ImageView.html) 开始加载时，Glide可能会自动应用 [FitCenter](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/load/resource/bitmap/FitCenter.html) 或 [CenterCrop](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/load/resource/bitmap/CenterCrop.html) ，这取决于view的 [ScaleType](http://developer.android.com/reference/android/widget/ImageView.ScaleType.html) 。如果 `scaleType` 是 `CENTER_CROP` , Glide 将会自动应用 `CenterCrop` 变换。如果 `scaleType` 为 `FIT_CENTER` 或 `CENTER_INSIDE` ，Glide会自动使用 `FitCenter` 变换。

- 使用 `Target.SIZE_ORIGINAL` 可能非常低效，或如果你的图片足够大可能引发 OOM 。

  

```
动画资源和定制目标
如果你只是要加载 GifDrawable，或任何其他资源类型到一个 View，你应该总是尽可能地使用 into(ImageView)。除了优雅的处理或新发起请求之外，Glide的大部分 ViewTarget 实现已经为您处理了 Drawable 动画。如果你确实必须使用定制的 ViewTarget，请确保继承自 ViewTarget 或在新请求开始之前和展示资源结束之后严格地清理从 into(Target) 返回的 Target。

如果你并非往 View 中加载图片，而直接使用 ViewTarget 或使用了定制的 Target 比如 SimpleTarget 且你正在加载一个动画资源例如 GifDrawable，你需要确保在 onResourceReady 中调用 start() 来启动这个动画：

Glide.with(fragment)
  .asGif()
  .load(url)
  .into(new SimpleTarget<>() {
    @Override
    public void onResourceReady(GifDrawable resource, Transition<GifDrawable> transition) {
      resource.start();
      // Set the resource wherever you need to use it.
    }
  });
如果你加载的是 Bitmap 或 GifDrawable，你可以判断这个可绘制对象是否实现了 Animatable：

Glide.with(fragment)
  .load(url)
  .into(new SimpleTarget<>() {
    @Override
    public void onResourceReady(Drawable resource, Transition<GifDrawable> transition) {
      if (resource instanceof Animatable) {
        resource.start();
      }
      // Set the resource wherever you need to use it.
    }
  });
```

#### 4.2 glide预加载  有问题？

https://blog.csdn.net/Ever69/article/details/104237329

```java 
 Glide.with(this).load(mUrl).diskCacheStrategy(DiskCacheStrategy.DATA).listener(object : RequestListener<Drawable> {
            override fun onLoadFailed(e: GlideException?, model: Any?, target: Target<Drawable>?, isFirstResource: Boolean): Boolean {
                Log.d("kkk", "预加载失败")
                return true
            }
 
            override fun onResourceReady(resource: Drawable?, model: Any?, target: Target<Drawable>?, dataSource: DataSource?, isFirstResource: Boolean): Boolean {
                Log.d("kkk", "预加载完成")
                return true
            }
        }).preload()
```

**diskCacheStrategy(DiskCacheStrategy.DATA)**

**preload()**

/*默认的策略是DiskCacheStrategy.AUTOMATIC 
DiskCacheStrategy有五个常量：
DiskCacheStrategy.**ALL** 使用DATA和RESOURCE缓存远程数据，仅使用RESOURCE来缓存本地数据。
DiskCacheStrategy.**NONE** 不使用磁盘缓存
DiskCacheStrategy.**DATA** 在资源**解码前**就将原始数据写入磁盘缓存
DiskCacheStrategy.**RESOURCE** 在资源**解码后**将数据写入磁盘缓存，即经过缩放等转换后的图片资源。
DiskCacheStrategy.**AUTOMATIC** 根据**原始图片**数据和资源编码策略来自动选择**磁盘缓存策略**。*/





### 5.打包jar 及 module引用aar（该module也需要打包成aar)

```
def jarName = "loadingAni1.0"
//Copy类型
task makeJar(type: Copy) {
 //删除存在的
  delete 'build/libs/' + jarName + ".jar"
  //设置拷贝的文件
  from("build/intermediates/aar_main_jar/release")
  //打进jar包后的文件目录,将classes.jar放入build/libs/目录下
  into('build/libs/')
  //要打包的jar文件
  include('classes.jar')
  //重命名
  rename('classes.jar', jarName + ".jar")
}
makeJar.dependsOn(build)
```

- module引用aar

  File -> New -> New Module -> Import .JAR/.AAR Package

  ```
  dependencies {
    implementation project(":imported_aar_module")
  }
  ```



### 6. [Gradle之apt, annotationProcessor和kapt](https://blog.csdn.net/l460133921/article/details/104908122)

APT(Annotation Processing Tool)，即 **注解处理工具**

annotationProcessor：注解处理器，是Gradle中 **内置的APT工具**

```
dependencies {
  annotationProcessor "com.alibaba:arouter-compiler:1.2.2"
}
```

kapt：Kotlin中不使用annotationProcessor，而是使用kapt

```
apply plugin: 'kotlin-kapt'
dependencies {
  kapt "com.alibaba:arouter-compiler:1.2.2"
}
```



### 7、gradle打包及自定义config.gradle/xx.properties-阿里云镜像



```groovy
repositories {
    //阿里云的镜像库
    maven {
        url "http://maven.aliyun.com/nexus/content/groups/public/"
    }
    jcenter()
    google()
}
```



```java
//gradle.properties文件
android.useDeprecatedNdk=true
MYAPP_RELEASE_STORE_FILE=app.keystore
MYAPP_RELEASE_KEY_ALIAS=hls
MYAPP_RELEASE_STORE_PASSWORD=123123

//build.gradle直接使用
signingConfigs {
        release {
            storeFile file(MYAPP_RELEASE_STORE_FILE)
            storePassword MYAPP_RELEASE_STORE_PASSWORD
            keyAlias MYAPP_RELEASE_KEY_ALIAS
            keyPassword MYAPP_RELEASE_KEY_PASSWORD
        }
}
```

```java
//根目录自定义keystore.properties
keyAlias=Key0
keyPassword=123456
keystore=../keystore/wanandroid.jks
storePassword=123456
    
//build.gradle使用
def keystoreFile = rootProject.file("keystore.properties")
def keystoreProperties = new Properties();
keystoreProperties.load(new FileInputStream(keystoreFile))

signingConfigs {
        release {
            keyAlias keystoreProperties['keyAlias']
            keyPassword keystoreProperties['keyPassword']
            storeFile file(keystoreProperties['keystore'])
            storePassword keystoreProperties['storePassword']
        }
}
```

```java
//根build.gradle引用自定义配置gradle文件
apply from: "config22.gradle"

buildscript {
    repositories {
        maven { url 'http://maven.aliyun.com/nexus/content/groups/public/' }
        google()
        jcenter() 
    }
    
    
//新建自定义文件config22.gradle名称
ext {
    android = [
            compileSdkVersion: 28,
            minSdkVersion    : 21,
            targetSdkVersion : 28,
            versionCode      : 1,
            versionName      : "1.1",
    ]
    libsVersion = [
            butterknife_version = "10.0.0",
            retrofit_version = "2.6.1",
            lifecycle_version = "2.2.0",
    ]

//最终module的build.gradle中使用  rootProject....
//该文件内部使用自己 butterknife           : "com.jakewharton:butterknife:$rootProject.butterknife_version"
ext {
    android = [
            compileSdkVersion: 28,
            minSdkVersion    : 21,
            targetSdkVersion : 28,
            versionCode      : 1,
            versionName      : "1.1",
    ]
    libsVersion = [
            butterknife_version = "10.0.0",
            retrofit_version = "2.6.1",
            lifecycle_version = "2.2.0",
    ]
    dependencies = [
            appcompat             : "androidx.appcompat:appcompat:1.0.0",
            arouterApi            : "com.alibaba:arouter-api:1.5.0",
            arouterCompiler       : "com.alibaba:arouter-compiler:1.2.2",
            butterknife           : "com.jakewharton:butterknife:$rootProject.butterknife_version",
            butterknifeCompiler   : "com.jakewharton:butterknife-compiler:$rootProject.butterknife_version",
```



### 8、RxJava RxAndroid

[给 Android 开发者的 RxJava 详解](https://gank.io/post/560e15be2dca930e00da1083#toc_31)

[RxJava 沉思录（一）：你认为 RxJava 真的好用吗？](https://juejin.cn/post/6844903670203547656)



`View` 是被观察者， `OnClickListener` 是观察者，二者通过 `setOnClickListener()` 方法达成订阅关系。

`Observable` (可观察者，即被观察者)

`Observer` (观察者)

`subscribe` (订阅)

事件

**在不指定线程的情况下， RxJava 遵循的是线程不变的原则**，即：在哪个线程调用 `subscribe()`，就在哪个线程生产事件；在哪个线程生产事件，就在哪个线程消费事件。如果需要切换线程，就需要用到 `Scheduler` （**调度器**）。

- `Schedulers.immediate()`: 直接在当前线程运行，相当于不指定线程。这是默认的 `Scheduler`。
- `Schedulers.newThread()`: 总是启用新线程，并在新线程执行操作。
- `Schedulers.io()`: I/O 操作（读写文件、读写数据库、网络信息交互等）所使用的 `Scheduler`。行为模式和 `newThread()` 差不多，区别在于 `io()` 的内部实现是是用一个无数量上限的线程池，可以重用空闲的线程，因此多数情况下 `io()` 比 `newThread()` 更有效率。不要把计算工作放在 `io()` 中，可以避免创建不必要的线程。
- `Schedulers.computation()`: 计算所使用的 `Scheduler`。这个计算指的是 CPU 密集型计算，即不会被 I/O 等操作限制性能的操作，例如图形的计算。这个 `Scheduler` 使用的固定的线程池，大小为 CPU 核数。不要把 I/O 操作放在 `computation()` 中，否则 I/O 操作的等待时间会浪费 CPU。
- 另外， Android 还有一个专用的 `AndroidSchedulers.mainThread()`，它指定的操作将在 Android 主线程运行。

**subscribeOn(...)**: `Observable.OnSubscribe` 被激活时所处的线程。或者叫做**事件产生的线程**。

**observeOn(...)**: 指定 `Subscriber` 所运行在的线程(回调在哪个线程中)。或者叫做**事件消费的线程**



**变换，就是将事件序列中的对象或整个序列进行加工处理，转换成不同的事件或事件序列。**

flat   flatmap



#### 8.1 定义

​		RxAndroid是RxJava在Android上的一个扩展，大牛JakeWharton的项目。据说和Retorfit、OkHttp组合起来使用，效果不是一般的好。而且用它似乎可以**完全替代eventBus**和OTTO。

​		Rx(Reactive Extensions)是一个库，用来**处理事件和异步任务**，在很多语言上都有实现，RxJava是Rx在Java上的实现。简单来说，RxJava就是处理异步的一个库，最基本是基于观察者模式来实现的。通过Obserable和Observer的机制，实现所谓**响应式的编程**体验。subscribeOn
　　Android的童鞋都知道，处理**异步事件**，现有的AsyncTask、**Handler**，不错的第三方事件总线EventBus、OTTO等等都可以处理。

​		最概括的两个字：**简洁**。而且当业务越繁琐越复杂时这一点就越显出优势——它能够保持简洁。它提供的各种功能强悍的操作符真的很强大。



#### 8.2 Observable：被观察者

**创建**（除了create方法外，还有just()和from()方法也可以创建Observable对象）：

```java
Observable<String> mObservable = Observable.create(new Observable.OnSubscribe<String>() {
            @Override
            public void call(Subscriber<? super String> subscriber) {

            }
}); 
```



#### 8.3 Subsriber：订阅者

```java
Subscriber<String> mTestSubscriber = new Subscriber<String>() {
            @Override
            public void onCompleted() {
            }
            @Override
            public void onError(Throwable e) {
            }
            @Override
            public void onNext(String s) {
            }
}; 
```



#### 8.4 Observer：观察者

Observer是一个接口，而Subscriber是它的一个抽象实现类。

public interface Observer<T> {}

public abstract class Subscriber<T> implements Observer<T>, Subscription {}

```java
Observer<String> mTestObsever = new Observer<String>() {
            @Override
            public void onCompleted() { 
         } 
​        @Override
​        public void onError(Throwable e) { 
​        } 
​        @Override
​        public void onNext(String s) { 
​        }
​    };
```

#### 8.5关联

```java
mObservable.subscribe(mTestSubscriber);
//or
mTestSubscriber.subscribe(mObservable);
```

**1、subscribeOn(Schedulers.io())  指定observable的 subscribeOn() 方法在后台线程运行。**

**2、observeOn(AndroidSchedulers.mainThread()) 指定 observer 的回调方法在主线程运行。**





### 9、**UI线程才能更新UI？** 

[Android UI 线程更新UI也会崩溃？？？](https://juejin.cn/post/6844904131962880008)

```java
void checkThread() {
    if (mThread != Thread.currentThread()) {
        throw new CalledFromWrongThreadException(
                "Only the original thread that created a view hierarchy can touch its views.");
    }
}
```

Only the original thread that created a view hierarchy can touch its views.
只有创建了视图层次结构的原始线程才能接触到它的视图

- 哪个现场创建的view，就只能在当前view显示更新等。
- 通常我们的ui都是在主线程中创建，所以才说UI线程更新ui。如果ui在子线程中创建，则...
- 如果跨线程通信，可借助handler。



### 10、recycleView从0到1



https://mp.weixin.qq.com/s/auphzaQF6_wJx6dGFY6niA : RecyclerView 有五虎上将

![图片](https://mmbiz.qpic.cn/mmbiz_png/v1LbPPWiaSt4iciasKiaPrFk69TMxh9DB0h55icAiciaoSlORib0U0FYWNRLZk5SZQQQxdMgZDW4D2NDThxD9aGVsE8UUw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



[【Android】自定义无限循环的LayoutManager](https://juejin.cn/post/6909363022980972552)



[如何实现 RecyclerView 的刷新分页](https://mp.weixin.qq.com/s/t1qiAggzR0MgxJXTB9mBAw)







### 11、Android相关基础

[Android的自定义View-基础知识-坐标系](https://juejin.cn/post/6909347047887880199)





### 12、UI适配

#### 12.1 ui优化

 **xml布局优化** 

在写xml布局文件的时候，我们要做的也有很多，比如： 
- 减少布局嵌套。多使用ViewStub、Merge、ConstraintLayout来代替。
- 优化开销。RelativeLayout和 使用weight的LinearLayout 开销比较大，建议使用ConstraintLayout，LinearLayout代替。

ViewStub：延迟加载

Merge： 取消层级

#### 12.2 屏幕适配

[骚年你的屏幕适配方式该升级了!-今日头条适配方案](https://segmentfault.com/a/1190000016079558)

[Android 目前稳定高效的UI适配方案](https://mp.weixin.qq.com/s/X-aL2vb4uEhqnLzU5wjc4Q)





```java
public static float applyDimension(int unit, float value,
                                       DisplayMetrics metrics)
    {
        switch (unit) {
        case COMPLEX_UNIT_PX:
            return value;
        case COMPLEX_UNIT_DIP:
            //公式 dp = px / density
            return value * metrics.density;
        case COMPLEX_UNIT_SP:
            return value * metrics.scaledDensity;
        case COMPLEX_UNIT_PT:
            return value * metrics.xdpi * (1.0f/72);
        case COMPLEX_UNIT_IN:
            return value * metrics.xdpi;
        case COMPLEX_UNIT_MM:
            return value * metrics.xdpi * (1.0f/25.4f);
        }
        return 0;
    }
```

https://segmentfault.com/a/1190000016079558

不管你在布局文件中填写的是什么单位，最后都会被转化为 **px**。

`今日头条适配方案`默认项目中只能 **以高或宽** 中的一个作为**基准**。





```java
DisplayMetrics{density=3.0, width=1080, height=2259, scaledDensity=3.0, xdpi=391.885, ydpi=391.026, densityDpi=480, noncompatWidthPixels=1080, noncompatHeightPixels=2259, noncompatDensity=3.0, noncompatDensityDpi=480, noncompatXdpi=391.885, noncompatYdpi=391.026} //华为手机
DisplayMetrics{density=1.5, width=1920, height=1080, scaledDensity=1.5, xdpi=159.895, ydpi=160.421}   //天邑盒子
DisplayMetrics{density=1.0, width=1280, height=720, scaledDensity=1.0, xdpi=160.157, ydpi=160.421}    //悦ME盒子
DisplayMetrics{density=1.5, width=1920, height=1080, scaledDensity=1.5, xdpi=240.0, ydpi=240.0}       //中兴盒子
int width = metric.widthPixels;  // 屏幕宽度（像素）
int height = metric.heightPixels;  // 屏幕高度（像素）
float density = metric.density;  // 屏幕密度（0.75 / 1.0 / 1.5）
int densityDpi = metric.densityDpi;  // 屏幕密度DPI（120 / 160 / 240）
		
//xml布局
android:layout_width="1908px"  ==>等效于  (1920px 换算成 1280dp)
android:layout_width="1272dp" 

//设置的是 px
view.setLayoutParams(new LinearLayout.LayoutParams(1910, 200));  
```



### 13、MVC/MVP/MVVM

https://mp.weixin.qq.com/s/h2i_17ChO9JsJvxq8tXwlA



![Jetpack MVVM 架构](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4fdef6223d184a1284479669b059f241~tplv-k3u1fbpfcp-watermark.image)

### 14、ViewPager2

与ViewPager相比，包括但不限于下列几点：

 1、不但支持水平方向翻页，还支持垂直方向翻页；

 2、支持RecyclerView.Adapter，允许调用适配器对象的notifyItem***方法，从而动态刷新某项视图；

 3、除了当前页，也支持展示左右两页的部分区域； 

4、支持在翻页过程中展示自定义的切换动画；

基础使用：https://cloud.tencent.com/developer/article/1650816

```java
implementation 'androidx.recyclerview:recyclerview:1.1.0'
implementation 'androidx.viewpager2:viewpager2:1.0.0'

<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">
    <!-- 标签布局TabLayout节点需要使用完整路径 -->
    <com.google.android.material.tabs.TabLayout
        android:id="@+id/tab_title"
        android:layout_width="match_parent"
        android:layout_height="wrap_content" />
    <!-- 二代翻页视图ViewPager2节点也需要使用完整路径 -->
    <androidx.viewpager2.widget.ViewPager2
        android:id="@+id/vp2_content"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1" />
</LinearLayout>
```





### 15、内存调优

[深入内存优化](https://www.jianshu.com/p/a38bfaaa9fac)

https://github.com/zhouffan/AndroidGuide/blob/master/android_opensource/6-LeakCanary%E6%89%A9%E5%B1%95%E9%98%85%E8%AF%BB.md

**内存泄露的本质是长周期对象持有了短周期对象的引用，导致短周期对象该被回收的时候无法被回收，从而导致内存泄露。**

**如果该对象由于代码错误或者其它原因导致迟迟无法被系统回收，此时就是发生了内存泄露。**

#### 1、Broadcast Receivers

如果在 Activity 中注册了 BroadcastReceiver 而忘记了 **unregister** 的话，BroadcastReceiver 就将一直持有对 Activity 的引用，即使 Activity 已经执行了 `onDestroy`

#### 2、Static Activity or View Reference

看下面的示例代码，将 TextView 声明为了静态变量（无论出于什么原因）。不管是直接还是间接通过静态变量引用了 Activity 或者 View，在 Activity 被销毁后都无法对其进行垃圾回收

**永远不要通过静态变量来引用 Activity、View 和 Context**

#### 3、Singleton Class Reference

看下面的例子，定义了一个 Singleton 类，该类需要传递 Context 以便从本地存储中获取一些文件

此时如果没有主动将 SingletonSampleClass 包含的 context 置空的话，就将导致内存泄露。那如何解决这个问题？

- 可以传递 ApplicationContext，而不是将 ActivityContext 传递给 singleton 类
- 如果真的必须使用 ActivityContext，那么当 Activity 被销毁的时候，需要确保传递将 singleton 类的 Context 设置为 null







### 16、Bitmap位图OOM

[Bitmap之位图采样和内存计算详解](https://mp.weixin.qq.com/s/vv7M5knOqF3YIx-NxD5D4g)

使用 bitmap.getConfig() 获取 Bitmap 的格式，这里是 **ARGB_8888** ，这种 Bitmap 格式下**一个像素点占 4 个字节**，所以要 x 4

drawable-**xxhdpi** 所代表的 density 为 **480**(density)，我的手机屏幕所代表的 density 是 480(targetDensity)



如下可以解释图片放在drawable-xxhdpi的密度越低的里面，加载图片越容易出现OOM，因为计算出来的图片大小越大。

```java
//targetDensity 手机固定的密度
scale = targetDensity / density
widthPix = originalWidth * scale
heightPix = orignalHeight * scale
Bitmap Memory = widthPix * scale * heightPix * scale * 4
    
//先采样然后计算采样后的内存，这里指定 inSampleSize 为200
inSampleSize = 200
scale = targetDensity / density = 480 / 480 = 1
widthPix = orignalScale * scale = 6000 / 200 * 1 = 30 
heightPix = orignalHeight * scale = 4000 / 200 * 1 = 20
Bitmap Memory =  widthPix * heightPix * 4
= 30 * 20 * 4 = 2400(Byte)
//---------------分割线-----xhdpi会成倍增加------------------------
//如果放在 drawable-xhdpi， density为 320
inSampleSize = 200
scale = targetDensity / density = 480 / 320
widthPix = orignalWidth * scale = 6000 / 200 * scale = 45
heightPix = orignalHeight * scale = 4000 / 200 * 480 / 320 = 30
Bitmap Memory =  widthPix * scale * heightPix * scale * 4
= 45 * 30 * 4 = 5400(Byte) 
   
//---------------分割线---------------------------------------
//一张大图 6000 x 4000 ,图片接近 12M------xxhdpi，请求宽高为100x100
inSampleSize = 4000 / 100 = 40
scale = targetDensity / density = 480 / 480 = 1
widthPix = orignalWidth * scale = 6000 / 40 * 1 = 150      
heightPix = orignalHeight * scale = 4000 / 40 * 1 = 100
BitmapMemory = widthPix * scale * heightPix * scale * 4 = 60000(Byte)
```



### 17、基础 



#### 17.1 AppCompat

[到底什么是AndroidX](https://blog.csdn.net/guolin_blog/article/details/97142065)

appcompat-v7指的是将库中提供的API向下兼容至API 7，也就是Android 2.1系统.

androidx:  对Android Support Library的一次升级。



[AppCompatActivity](https://mp.weixin.qq.com/s/nHGZmzTAioVH9FYEqR1c9Q)

其间接继承自Activity，之间还继承了其他Activity特色类,可以使得低版本上运行的Activity也能拥有ToolBar和暗黑主题等新功能。





#### 17.2 事件分发

安卓的事件分发大概会经历 Activity -> PhoneWindow -> DecorView -> ViewGroup -> View 的 dispatchTouchEvent

https://juejin.cn/post/6874589638925746190

1. 如何解决滑动冲突？

- 通过重写父类的 onInterceptTouchEvent 来拦截滑动事件
- 通过在子类中调用 parent.requestDisallowInterceptTouchEvent 来通知父类是否要拦截事件







#### 17.3 View绘制

[android 绘制流程之修改子View的绘制顺序](https://juejin.cn/post/6883323198918787080/)

[ViewGroup 默认顺序绘制子 View，如何修改？什么场景需要修改绘制顺序？](https://blog.csdn.net/plokmju88/article/details/107399102)

int getChildDrawingOrder（int childCount, int i）中

i           ====> 代表的是绘制的顺序，**第几个** 

返回值 ====> 代表 当前容器的 **第几个子View**。

```java
@Override
protected int getChildDrawingOrder(int childCount, int i) {
  View view = getLayoutManager().getFocusedChild();
  if (null == view) {
    return super.getChildDrawingOrder(childCount, i);
  }
  int position = indexOfChild(view);
  if (position < 0) {
    return super.getChildDrawingOrder(childCount, i);
  }
  //绘制顺序=====》跟最后一个互换
  //当绘制最后一个 view顺序时，取position焦点view
  if (i == childCount - 1) {
    return position;
  }
  //当前绘制的这个 跟焦点view下标相同，取最后一个view
  if (i == position) {
    return childCount - 1;
  }
  return super.getChildDrawingOrder(childCount, i);
}
```







[公共技术点之 View 绘制流程](https://a.codekk.com/detail/Android/lightSky/%E5%85%AC%E5%85%B1%E6%8A%80%E6%9C%AF%E7%82%B9%E4%B9%8B%20View%20%E7%BB%98%E5%88%B6%E6%B5%81%E7%A8%8B)

![image-20210207142554320](D:\Desktop\notes\view-draw.png)

#### 17.4 理解android:sharedUserId="android.uid.system"

https://blog.csdn.net/u012398902/article/details/52735980

​		通过Shared User id,拥有同一个User id的多个APK可以配置成运行在**同一个进程中**。那么把程序的UID配成android.uid.system，也就是要让程序运行在**系统进程**中，这样就有权限来修改系统时间了。 只是加入UID还不够，如果这时候安装APK的话发现无法安装，提示**签名不符**。

**目的： 跟系统在同一个进程中**



#### 17.5 异常

[OutOfMemoryError 可以被 try catch 吗？](https://juejin.cn/post/6874916707543187463)

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bd1a8299be0041f086449e1f105842b1~tplv-k3u1fbpfcp-zoom-1.image)



#### 17.6 view 监听事件

**setOnDragListener**

https://www.jianshu.com/p/37e72684412f -   Android之View拖拽效果

```java
bt_1.setOnLongClickListener(new View.OnLongClickListener() {
        @Override
        public boolean onLongClick(View v) {
            //设置震动反馈
            bt_1.performHapticFeedback(HapticFeedbackConstants.LONG_PRESS, HapticFeedbackConstants.FLAG_IGNORE_GLOBAL_SETTING);

            // 创建DragShadowBuilder，我把控件本身传进去
            View.DragShadowBuilder builder = new View.DragShadowBuilder(bt_1);
            // 剪切板数据，可以在DragEvent.ACTION_DROP方法的时候获取。
            ClipData data = ClipData.newPlainText("Label", "我是文本内容！");
            // 开始拖拽
            if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.N) {
                bt_1.startDragAndDrop(data, builder,bt_1, 0);
            }else{
                bt_1.startDrag(data, builder, bt_1, 0);
            }
            return true;
        }
    });


//
ll_demo.setOnDragListener(new View.OnDragListener() {
        @Override
        public boolean onDrag(View v, DragEvent event) {

            //获取事件
            int action = event.getAction();
            switch (action) {
                case DragEvent.ACTION_DRAG_STARTED:
                    Log.d("aaa", "开始拖拽");
                    break;
                case DragEvent.ACTION_DRAG_ENDED:
                    Log.d("aaa", "结束拖拽");
                    break;
                case DragEvent.ACTION_DRAG_ENTERED:
                    Log.d("aaa", "拖拽的view进入监听的view时");
                    break;
                case DragEvent.ACTION_DRAG_EXITED:
                    Log.d("aaa", "拖拽的view离开监听的view时");
                    ll_demo.setBackgroundColor(Color.parseColor("#303F9F"));
                    break;
                case DragEvent.ACTION_DRAG_LOCATION:
                    float x = event.getX();
                    float y = event.getY();
                    ll_demo.setBackgroundColor(Color.GRAY);
                    Log.i("aaa", "拖拽的view在监听view中的位置:x =" + x + ",y=" + y);
                    break;
                case DragEvent.ACTION_DROP:
                    Log.i("aaa", "释放拖拽的view");

                    if(event != null && event.getLocalState() != null){
                        View localState = (View) event.getLocalState();
                        FrameLayout.LayoutParams layoutParams = new FrameLayout.LayoutParams(ViewGroup.LayoutParams.WRAP_CONTENT, ViewGroup.LayoutParams.WRAP_CONTENT);
                        ((ViewGroup) localState.getParent()).removeView(localState);
                        ll_demo.addView(localState, layoutParams);
                    }
                    break;
            }
            return true;
        }
    });
```





**setOnHoverListener** - 悬停事件

View类现在支持“悬停”事件，通过对指针设备（如鼠标或其他设备驱动屏幕上的光标）支持，使得其用户交互更加丰富。

 悬停事件可以是下列操作之一：
ACTION_HOVER_ENTER
ACTION_HOVER_EXIT
ACTION_HOVER_MOVE

如果你在View.OnHoverListener中 onHover()处理了此事件，则应该返回真。 如果返回false，则悬停事件将被继续分派到它的父视图中。

可以通过android:state_hovered 和state_hovered属性状态列表提供不同的背景绘制来响应悬停事件。







### 18、Jetpack

[是让人耳目一新的 Jetpack MVVM 精讲](https://www.jianshu.com/p/85e528e8474b)