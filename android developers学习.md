### 1、jetpack架构

#### 1.1 视图绑定

##### 1.1.1 特点

- 在大多数情况下，视图绑定会替代 `findViewById`

- ```groovy
  android {
  	viewBinding {
              enabled = true
      }
  }
  ```

- ```xml
  //忽略某个布局文件
  <LinearLayout
              ...
              tools:viewBindingIgnore="true" >
          ...
  </LinearLayout>
  ```

- 系统会通过以下方式生成绑定类的名称：将 XML 文件的名称转换为驼峰式大小写，并在末尾添加“Binding”一词。

- 每个绑定类还包含一个 `getRoot()` 方法，用于为相应布局文件的根视图提供直接引用。

  ```java
  //视图view和activity产生关联
  mainBinding = ActivityMainBinding.inflate(getLayoutInflater());
  setContentView(mainBinding.getRoot());
  ```

- Fragment 的存在时间比其视图长。请务必在 Fragment 的 onDestroyView() 方法中清除对绑定类实例的所有引用。

  ```java
  private ResultProfileBinding binding;
      @Override
      public View onCreateView (LayoutInflater inflater,
                                ViewGroup container,
                                Bundle savedInstanceState) {
          binding = ResultProfileBinding.inflate(inflater, container, false);
          View view = binding.getRoot();
          return view;
      }
      @Override
      public void onDestroyView() {
          super.onDestroyView();
          binding = null;
      }
  ```

##### 1.1.2 与 findViewById 的区别

  与使用 `findViewById` 相比，视图绑定具有一些很显著的优点：

  - **Null 安全**：由于视图绑定会创建对视图的直接引用，因此不存在因视图 ID 无效而引发 Null 指针异常的风险。此外，如果视图仅出现在布局的某些配置中，则绑定类中包含其引用的字段会使用 `@Nullable` 标记。
  - **类型安全**：每个绑定类中的字段均具有与它们在 XML 文件中引用的视图相匹配的类型。这意味着不存在发生类转换异常的风险。

#####       1.1.3 与数据绑定的对比

视图绑定和[数据绑定](https://developer.android.com/topic/libraries/data-binding)均会生成可用于直接引用视图的绑定类。视图绑定旨在处理更简单的用例，与数据绑定相比，具有以下优势：

- **更快的编译速度**：视图绑定不需要处理注释，因此编译时间更短。
- **易于使用**：视图绑定不需要特别标记的 XML 布局文件，因此在应用中采用速度更快。在模块中启用视图绑定后，它会自动应用于该模块的所有布局。

反过来，与数据绑定相比，视图绑定也具有以下限制：

- 视图绑定不支持[布局变量或布局表达式](https://developer.android.com/topic/libraries/data-binding/expressions)，因此不能用于直接在 XML 布局文件中声明动态界面内容。
- 视图绑定不支持[双向数据绑定](https://developer.android.com/topic/libraries/data-binding/two-way)。

考虑到这些因素，在某些情况下，最好在项目中同时使用视图绑定和数据绑定。您可以在需要高级功能的布局中使用数据绑定，而在不需要高级功能的布局中使用视图绑定。



#### 1.2 数据绑定

以申明方式将 可观察数据绑定到界面元素。将布局中的界面组件绑定到应用中的数据源

```groovy
android { 
        dataBinding {
            enabled = true
        }
}
```



- ```xml
  <layout xmlns:android="http://schemas.android.com/apk/res/android">
      <data>
          <variable
              name="user"
              type="com.amt.test_jetpack.User" />
      </data>
      <androidx.constraintlayout.widget.ConstraintLayout 
          android:layout_width="match_parent"
          android:layout_height="match_parent" > 
          <TextView
              android:id="@+id/tv1"
              android:layout_width="wrap_content"
              android:layout_height="wrap_content"
              <!--lastName访问的是 user的 set方法 或者 直接属性 -->
              android:text="@{String.valueOf(user.firstName)}" /> 
      </androidx.constraintlayout.widget.ConstraintLayout>
  </layout>
  
  // databind中自动生成全部 variable 的set get方法，以供xml使用
  User user = new User(111,"222");
  mainBinding.setUser(user);
  ```

- **注意**：布局表达式应**保持精简**，因为它们无法进行单元测试，并且拥有的 IDE 支持也有限。为了简化布局表达式，可以使用自定义[绑定适配器](https://developer.android.com/topic/libraries/data-binding/binding-adapters)。

- ```java
  //activity 使用
  mainBinding = DataBindingUtil.setContentView(this, R.layout.activity_main);
  //or
  mainBinding = ActivityMainBinding.inflate(getLayoutInflater());
  setContentView(mainBinding.getRoot());
  ```

- ```java
  //在 Fragment、ListView 或 RecyclerView 适配器中使用数据绑定项
  ListItemBinding binding = ListItemBinding.inflate(layoutInflater, viewGroup, false);
  // or
  ListItemBinding binding = DataBindingUtil.inflate(layoutInflater, R.layout.list_item, viewGroup, false);
  ```

- 比较运算符 `== > < >= <=`（请注意，`<` 需要转义为 &lt;）

- ```xml
  //判断 左边运算数不是 null
  android:text="@{user.displayName ?? user.lastName}"
  //or
  android:text="@{user.displayName != null ? user.displayName : user.lastName}"
  
  ```

- **注意**：要使 XML 不含语法错误，您必须转义 `<` 字符。例如：不要写成 `List<String>` 形式，而是必须写成 `List<String>`。

  ```xml
  <data>
          <import type="android.util.SparseArray"/>
          <import type="java.util.Map"/>
          <import type="java.util.List"/>
          <variable name="list" type="List&lt;String>"/>
          <variable name="sparse" type="SparseArray&lt;String>"/>
          <variable name="map" type="Map&lt;String, String>"/>
          <variable name="index" type="int"/>
          <variable name="key" type="String"/>
      </data> 
  
      android:text="@{list[index]}" 
      android:text="@{sparse[index]}" 
      android:text="@{map[key]}"
  ```

- ====================点击事件  ----   容易遗漏此处=====================

  ```java
  mainBinding = ActivityMainBinding.inflate(getLayoutInflater());
  //注意创建监听器
  mainBinding.setHandlers(new MyHandlers());
  
  
  
  android:onClick="@{(view) -> handlers.onClickFriend(view)}"
  
  public class MyHandlers {
      public void onClickFriend(View view) {
          Toast.makeText(view.getContext(), "onClickFriend", Toast.LENGTH_SHORT).show();
      }
  }
  ```

- ```xml
  <LinearLayout
             android:orientation="vertical"
             android:layout_width="match_parent"
             android:layout_height="match_parent">
             <include layout="@layout/name"
                 bind:user="@{user}"/>
             <include layout="@layout/contact"
                 bind:user="@{user}"/>
  </LinearLayout>
  
  <merge><!-- Doesn't work   减少include的最父层级-->
             <include layout="@layout/name"
                 bind:user="@{user}"/>
             <include layout="@layout/contact"
                 bind:user="@{user}"/>
  </merge>
  ```



#### 1.3 lifecycles

##### 1.3.1 定义

生命周期感知型组件可执行操作来响应另一个组件（如 Activity 和 Fragment）的生命周期状态的变化。这些组件有助于您写出更有条理且往往更精简的代码，这样的代码更易于维护。

[`androidx.lifecycle`](https://developer.android.com/reference/androidx/lifecycle/package-summary) 软件包提供了可用于构建生命周期感知型组件的类和接口

- 添加观察者

  ```java
  getLifecycle().addObserver(new MyObserver());
  ```

- 生命周期

 ```java
  public class MyObserver implements LifecycleObserver {
          @OnLifecycleEvent(Lifecycle.Event.ON_RESUME)
          public void connectListener() {
              Log.i("=====》","111111111111");
          }
          @OnLifecycleEvent(Lifecycle.Event.ON_PAUSE)
          public void disconnectListener() {
              Log.i("=====》","222222222222");
          }
      }
 ```

  



##### 1.3.2生命周期感知型组件的最佳做法

- 使界面控制器（Activity 和 Fragment）尽可能保持精简。它们不应试图获取自己的数据，而应使用 [`ViewModel`](https://developer.android.com/reference/androidx/lifecycle/ViewModel) 执行此操作，并观察 [`LiveData`](https://developer.android.com/reference/androidx/lifecycle/LiveData) 对象以将更改体现到视图中。

- 设法编写数据驱动型界面，对于此类界面，界面控制器的责任是随着数据更改而更新视图，或者将用户操作通知给 [`ViewModel`](https://developer.android.com/reference/androidx/lifecycle/ViewModel)。

- 将数据逻辑放在 [`ViewModel`](https://developer.android.com/reference/androidx/lifecycle/ViewModel) 类中。[`ViewModel`](https://developer.android.com/reference/androidx/lifecycle/ViewModel) 应充当界面控制器与应用其余部分之间的连接器。不过要注意，[`ViewModel`](https://developer.android.com/reference/androidx/lifecycle/ViewModel) 不负责获取数据（例如，从网络获取）。[`ViewModel`](https://developer.android.com/reference/androidx/lifecycle/ViewModel) 应调用相应的组件来获取数据，然后将结果提供给界面控制器。

- 使用 [Data Binding](https://developer.android.com/topic/libraries/data-binding) 在视图与界面控制器之间维持干净的接口。这样一来，您可以使视图更具声明性，并尽量减少需要在 Activity 和 Fragment 中编写的更新代码。如果您更愿意使用 Java 编程语言执行此操作，请使用诸如 [Butter Knife](http://jakewharton.github.io/butterknife/) 之类的库，以避免样板代码并实现更好的抽象化。

- 如果界面很复杂，不妨考虑创建 [presenter](http://www.gwtproject.org/articles/mvp-architecture.html#presenter) 类来处理界面的修改。这可能是一项艰巨的任务，但这样做可使界面组件更易于测试。

- 避免在 [`ViewModel`](https://developer.android.com/reference/androidx/lifecycle/ViewModel) 中引用 `View` 或 `Activity` 上下文。如果 `ViewModel` 存在的时间比 Activity 更长（在配置更改的情况下），Activity 将泄露并且不会由垃圾回收器妥善处置。

- 使用 [Kotlin 协程](https://developer.android.com/topic/libraries/architecture/coroutines)管理长时间运行的任务和其他可以异步运行的操作。

  

##### 1.3.3生命周期感知型组件的用例

生命周期感知型组件可使您在各种情况下更轻松地管理生命周期。下面列举几个例子：

- 在粗粒度和细粒度位置更新之间切换。使用生命周期感知型组件可在位置应用可见时启用细粒度位置更新，并在应用位于后台时切换到粗粒度更新。借助生命周期感知型组件 [`LiveData`](https://developer.android.com/reference/androidx/lifecycle/LiveData)，应用可以在用户使用位置发生变化时自动更新界面。
- 停止和开始视频缓冲。使用生命周期感知型组件可尽快开始视频缓冲，但会推迟播放，直到应用完全启动。此外，应用销毁后，您还可以使用生命周期感知型组件终止缓冲。
- 开始和停止网络连接。借助生命周期感知型组件，可在应用位于前台时启用网络数据的实时更新（流式传输），并在应用进入后台时自动暂停。
- 暂停和恢复动画可绘制资源。借助生命周期感知型组件，可在应用位于后台时暂停动画可绘制资源，并在应用位于前台后恢复可绘制资源。





#### 1.4 LiveData



#### 1.5 Navigation



#### 1.6 Paging



#### 1.7 Room



#### 1.8 ViewModel



#### 1.9 WorkManager











