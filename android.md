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

![img](https://upload-images.jianshu.io/upload_images/6988326-58819360d6526e55.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

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















