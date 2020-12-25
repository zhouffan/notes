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
| `implementation`      | `compile`  | Gradle 会将依赖项添加到编译类路径，并将依赖项打包到构建输出。不过，当您的模块配置 `implementation` 依赖项时，会让 Gradle 了解您不希望该模块在编译时将该依赖项泄露给其他模块。也就是说，其他模块只有在运行时才能使用该依赖项。使用此依赖项配置代替 `api` 或 `compile`（已弃用）可以**显著缩短构建时间**，因为这样可以减少构建系统需要重新编译的模块数。例如，如果 `implementation` 依赖项更改了其 API，Gradle 只会重新编译该依赖项以及直接依赖于它的模块。大多数应用和测试模块都应使用此配置。 |
| `api`                 | `compile`  | Gradle 会将依赖项添加到编译类路径和构建输出。当一个模块包含 `api` 依赖项时，会让 Gradle 了解该模块要以传递方式将该依赖项导出到其他模块，以便这些模块在运行时和编译时都可以使用该依赖项。此配置的行为类似于 `compile`（现已弃用），但使用它时应格外小心，只能对您需要以传递方式导出到其他上游消费者的依赖项使用它。这是因为，如果 `api` 依赖项更改了其外部 API，Gradle 会在编译时重新编译所有有权访问该依赖项的模块。因此，拥有大量的 `api` 依赖项会显著增加构建时间。除非要将依赖项的 API 公开给单独的模块，否则库模块应改用 `implementation` 依赖项。 |
| `compileOnly`         | `provided` | Gradle 只会将依赖项添加到编译类路径（也就是说，不会将其添加到构建输出）。如果您创建 Android 模块时在编译期间需要相应依赖项，但它在运行时可有可无，此配置会很有用。如果您使用此配置，那么您的库模块必须包含一个运行时条件，用于检查是否提供了相应依赖项，然后适当地改变该模块的行为，以使该模块在未提供相应依赖项的情况下仍可正常运行。这样做不会添加不重要的瞬时依赖项，因而有助于减小最终 APK 的大小。此配置的行为类似于 `provided`（现已弃用）。**注意**：您不能将 `compileOnly` 配置与 AAR 依赖项配合使用。 |
| `runtimeOnly`         | `apk`      | Gradle 只会将依赖项添加到构建输出，以便在运行时使用。也就是说，不会将其添加到编译类路径。此配置的行为类似于 `apk`（现已弃用）。 |
| `annotationProcessor` | `compile`  | 如需添加对作为注释处理器的库的依赖关系，您必须使用 `annotationProcessor` 配置将其添加到注释处理器类路径。这是因为，使用此配置可以将编译类路径与注释处理器类路径分开，从而提高构建性能。如果 Gradle 在编译类路径上找到注释处理器，则会禁用[避免编译](https://docs.gradle.org/current/userguide/java_plugin.html#sec:java_compile_avoidance)功能，这样会对构建时间产生负面影响（Gradle 5.0 及更高版本会忽略在编译类路径上找到的注释处理器）。如果 JAR 文件包含以下文件，则 Android Gradle 插件会假定依赖项是注释处理器： `META-INF/services/javax.annotation.processing.Processor`。如果插件检测到编译类路径上包含注释处理器，则会生成构建错误。 |
| `lintChecks`          |            | 使用此配置可以添加您希望 Gradle 在构建项目时执行的 lint 检查。**注意**：使用 Android Gradle 插件 3.4.0 及更高版本时，此依赖项配置不再将 lint 检查打包在 Android 库项目中。如需将 lint 检查依赖项包含在 AAR 库中，请使用下面介绍的 `lintPublish` 配置。 |
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

[Android：手把手带你 深入读懂 Retrofit 2.0 源码](https://www.jianshu.com/p/0c055ad46b6c)     ************

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





### [4. glide官方](https://muyangmin.github.io/glide-docs-cn/#glide-v4-android-english-tip)

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



### 7、gradle打包及自定义config.gradle/xx.properties



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



#### 8.1 定义

​		RxAndroid是RxJava在Android上的一个扩展，大牛JakeWharton的项目。据说和Retorfit、OkHttp组合起来使用，效果不是一般的好。而且用它似乎可以**完全替代eventBus**和OTTO。

​		Rx(Reactive Extensions)是一个库，用来**处理事件和异步任务**，在很多语言上都有实现，RxJava是Rx在Java上的实现。简单来说，RxJava就是处理异步的一个库，最基本是基于观察者模式来实现的。通过Obserable和Observer的机制，实现所谓**响应式的编程**体验。
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

[【Android】自定义无限循环的LayoutManager](https://juejin.cn/post/6909363022980972552)







### 11、Android相关基础

[Android的自定义View-基础知识-坐标系](https://juejin.cn/post/6909347047887880199)

