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

[`LiveData`](https://developer.android.com/reference/androidx/lifecycle/LiveData) 是一种可观察的数据存储器类。具有生命周期感知能力。

如果观察者（由 [`Observer`](https://developer.android.com/reference/androidx/lifecycle/Observer) 类表示）的生命周期处于 [`STARTED`](https://developer.android.com/reference/androidx/lifecycle/Lifecycle.State#STARTED) 或 [`RESUMED`](https://developer.android.com/reference/androidx/lifecycle/Lifecycle.State#RESUMED) 状态，则使活跃状态。LiveData只通知活跃状态的观察者。

可以放心地观察 [`LiveData`](https://developer.android.com/reference/androidx/lifecycle/LiveData) 对象而不必担心泄露（当 Activity 和 Fragment 的生命周期被销毁时，系统会立即退订它们）。

有以下优势：

- **确保界面符合数据状态** ：LiveData 遵循观察者模式。当生命周期状态发生变化时，LiveData 会通知 [`Observer`](https://developer.android.com/reference/androidx/lifecycle/Observer) 对象。您可以整合代码以在这些 `Observer` 对象中更新界面。观察者可以在每次发生更改时更新界面，而不是在每次应用数据发生更改时更新界面。
- **不会发生内存泄漏**： 观察者会绑定到 [`Lifecycle`](https://developer.android.com/reference/androidx/lifecycle/Lifecycle) 对象，并在其关联的生命周期遭到销毁后进行自我清理。
- **不会因 Activity 停止而导致崩溃**：如果观察者的生命周期处于非活跃状态（如返回栈中的 Activity），则它不会接收任何 LiveData 事件。
- **不再需要手动处理生命周期**：界面组件只是观察相关数据，不会停止或恢复观察。LiveData 将自动管理所有这些操作，因为它在观察时可以感知相关的生命周期状态变化。
- **数据始终保持最新状态**：如果生命周期变为非活跃状态，它会在**再次**变为活跃状态时接收最新的数据。例如，曾经在后台的 Activity 会在返回前台后立即接收最新的数据。
- **适当的配置更改**：如果由于配置更改（如设备旋转）而重新创建了 Activity 或 Fragment，它会立即接收最新的可用数据。
- **共享资源**：您可以使用单一实例模式扩展 [`LiveData`](https://developer.android.com/reference/androidx/lifecycle/LiveData) 对象以封装系统服务，以便在应用中共享它们。`LiveData` 对象连接到系统服务一次，然后需要相应资源的任何观察者只需观察 `LiveData` 对象。

LiveData 是一种可用于任何数据的封装容器，其中包括可实现 `Collections` 的对象，如 `List`。[`LiveData`](https://developer.android.com/reference/androidx/lifecycle/LiveData) 对象通常存储在 [`ViewModel`](https://developer.android.com/reference/androidx/lifecycle/ViewModel) 对象中，并可通过 getter 方法进行访问

**注意**：请确保用于更新界面的 `LiveData` 对象存储在 `ViewModel` 对象中，而不是将其存储在 Activity 或 Fragment 中，原因如下：

- 避免 Activity 和 Fragment 过于庞大。现在，这些界面控制器负责显示数据，但不负责存储数据状态。

- 将 `LiveData` 实例与特定的 Activity 或 Fragment 实例分离开，并使 `LiveData` 对象在配置更改后继续存在。

```java
private MutableLiveData<List<User>> users;
public LiveData<List<User>> getUsers() {
    if (users == null) {
        users = new MutableLiveData<List<User>>();
        loadUsers();
    }
    return users;
}
```



 

#### 1.5 ViewModel 

[`ViewModel`](https://developer.android.com/reference/androidx/lifecycle/ViewModel) 类旨在以注重生命周期的方式存储和管理界面相关的数据。[`ViewModel`](https://developer.android.com/reference/androidx/lifecycle/ViewModel) 类让数据可在发生屏幕旋转等配置更改后继续留存。

Activity 和 Fragment 之类的界面控制器主要用于显示界面数据、对用户操作做出响应或处理操作系统通信（如权限请求）。如果要求界面控制器也负责从数据库或网络加载数据，那么会使类越发膨胀。

从界面控制器逻辑中分离出视图数据所有权的做法更易行且更高效。

架构组件为界面控制器提供了 [`ViewModel`](https://developer.android.com/reference/androidx/lifecycle/ViewModel) 辅助程序类，该类负责为界面准备数据。在配置更改期间会自动保留 [`ViewModel`](https://developer.android.com/reference/androidx/lifecycle/ViewModel) 对象，以便它们存储的数据立即可供下一个 Activity 或 Fragment 实例使用。例如，如果您需要在应用中显示用户列表，请确保将获取和保留该用户列表的责任分配给 [`ViewModel`](https://developer.android.com/reference/androidx/lifecycle/ViewModel)，而不是 Activity 或 Fragment。

```java
 public class MyViewModel extends ViewModel {
        private MutableLiveData<List<User>> users;
        public LiveData<List<User>> getUsers() {
            if (users == null) {
                users = new MutableLiveData<List<User>>();
                loadUsers();
            }
            return users;
        } 
        private void loadUsers() {
            // Do an asynchronous operation to fetch users.
        }
    }
```

```java
public class MyActivity extends AppCompatActivity {
        public void onCreate(Bundle savedInstanceState) {
            // Create a ViewModel the first time the system calls an activity's onCreate() method.
            // Re-created activities receive the same MyViewModel instance created by the first activity.

            MyViewModel model = new ViewModelProvider(this).get(MyViewModel.class);
            model.getUsers().observe(this, users -> {
                // update UI
            });
        }
    }
    
```

- Fragment 之间**共享**数据

  主 Fragment：从列表中选择一项； 从 Fragment：显示选定项的内容。不需要fragment和依赖activity之间的任何交互处理。

  ```java
  public class SharedViewModel extends ViewModel {
          private final MutableLiveData<Item> selected = new MutableLiveData<Item>();
  
          public void select(Item item) {
              selected.setValue(item);
          }
  
          public LiveData<Item> getSelected() {
              return selected;
          }
      }
  
      public class MasterFragment extends Fragment {
          private SharedViewModel model;
  
          public void onViewCreated(@NonNull View view, Bundle savedInstanceState) {
              super.onViewCreated(view, savedInstanceState);
              model = new ViewModelProvider(requireActivity()).get(SharedViewModel.class);
              itemSelector.setOnClickListener(item -> {
                  model.select(item);
              });
          }
      }
  
      public class DetailFragment extends Fragment {
  
          public void onViewCreated(@NonNull View view, Bundle savedInstanceState) {
              super.onViewCreated(view, savedInstanceState);
              SharedViewModel model = new ViewModelProvider(requireActivity()).get(SharedViewModel.class);
              model.getSelected().observe(getViewLifecycleOwner(), { item ->
                 // Update the UI.
              });
          }
      }
  ```

#### 1.6 Navigation





#### 1.7 Room



#### 1.8 Paging



#### 1.9 WorkManager



#### 2.0 Fragment 

设计 Fragment 时需要牢记的一个要点是，不要创建与特定 Activity 的强耦合。为此，您通常可以定义一个接口，将 Fragment 与其宿主 Activity 进行交互时需要使用的所有方式抽象化，然后宿主 Activity 实现该接口。

```java
 public class HeadlinesFragment extends ListFragment {
        ...
        OnHeadlineSelectedListener mHeadlineSelectedListener = null;

        /* Must be implemented by host activity */
        public interface OnHeadlineSelectedListener {
            public void onHeadlineSelected(int index);
        }
        ...

        public void setOnHeadlineSelectedListener(OnHeadlineSelectedListener listener) {
            mHeadlineSelectedListener = listener;
        }
     
     
     ...
        public void onItemClick(AdapterView<?> parent,
                                View view, int position, long id) {
            if (null != mHeadlineSelectedListener) {
                mHeadlineSelectedListener.onHeadlineSelected(position);
            }
        }
    }
```



### 2、应用基础知识

#### 2.1 应用基础知识

每个 Android 应用都处于各自的安全沙盒中，并受以下 Android 安全功能的保护：

- Android 操作系统是一种多用户 Linux 系统，其中的每个应用都是一个不同的用户；

- 默认情况下，系统会为每个应用分配一个唯一的 Linux 用户 ID（该 ID 仅由系统使用，应用并不知晓）。系统会为应用中的所有文件设置权限，使得只有分配给该应用的用户 ID 才能访问这些文件；

- 每个进程都拥有自己的虚拟机 (VM)，因此应用代码独立于其他应用而运行。

- 默认情况下，每个应用都在其自己的 Linux 进程内运行。Android 系统会在需要执行任何应用组件时启动该进程，然后当不再需要该进程或系统必须为其他应用恢复内存时，其便会关闭该进程。

Android 系统实现了*最小权限原则*。换言之，默认情况下，每个应用只能访问执行其工作所需的组件，而不能访问其他组件。不过，应用仍可通过一些途径与其他应用共享数据以及访问系统服务：

- 可以安排两个应用共享同一 Linux 用户 ID，在此情况下，二者便能访问彼此的文件。为节省系统资源，也可安排拥有相同用户 ID 的应用在同一 Linux 进程中运行，并共享同一 VM。应用还必须使用相同的证书进行签名。
- 应用可以请求访问设备数据（如用户的联系人、短信消息、可装载存储装置（SD 卡）、相机、蓝牙等）的权限。用户必须明确授予这些权限。

应用组件是 Android 应用的基本构建块。**每个组件都是一个入口点**，系统或用户可通过该入口点进入您的应用。应用组件类型：

- Activity

- 服务

  服务可能会在后台播放音乐或通过网络获取数据，但这不会阻断用户与 Activity 的交互。

  <u>**注意：**如果您的应用面向 Android 5.0（API 级别 21）或更高版本，请使用 `JobScheduler` 类来调度操作。JobScheduler 的优势在于，它能通过优化作业调度来降低功耗，以及使用 [Doze](https://developer.android.com/training/monitoring-device-state/doze-standby) API，从而达到省电目的。</u>

  <u>**注意：**如果您使用 Intent 来启动 `Service`，请使用[显式](https://developer.android.com/guide/components/intents-filters#Types) Intent 来确保应用的安全性。使用隐式 Intent 启动服务存在安全隐患，因为您无法确定哪些服务将响应 Intent，且用户无法看到哪些服务已启动。从 Android 5.0（API 级别 21）开始，如果使用隐式 Intent 调用 `bindService()`，系统会抛出异常。请勿为您的服务声明 Intent 过滤器。</u>

- 广播接收器

  许多广播均由系统发起，例如，通知屏幕已关闭、电池电量不足或已拍摄照片的广播。广播接收器更常见的用途只是作为通向其他组件的*通道*，旨在执行极少量的工作。例如，它可能会根据带 `JobScheduler` 的事件调度 `JobService` 来执行某项工作

- 内容提供程序

  *内容提供程序*管理一组共享的应用数据，您可以将这些数据存储在文件系统、SQLite 数据库、网络中或者您的应用可访问的任何其他持久化存储位置。

- 启动组件

  通过定义消息来启动特定组件（显式 Intent）或特定的组件*类型*（隐式 Intent）。

  每种组件都有不同的启动方法：

  - 如要启动 Activity，您可以向 `startActivity()` 或 `startActivityForResult()` 传递 `Intent`（当您想让 Activity 返回结果时），或者为其安排新任务。
  - 在 Android 5.0（API 级别 21）及更高版本中，您可以使用 `JobScheduler` 类来调度操作。对于早期 Android 版本，您可以通过向 `startService()` 传递 `Intent` 来启动服务（或对执行中的服务下达新指令）。您也可通过向将 `bindService()` 传递 `Intent` 来绑定到该服务。
  - 您可以通过向 `sendBroadcast()`、`sendOrderedBroadcast()` 或 `sendStickyBroadcast()` 等方法传递 `Intent` 来发起广播。
  - 您可以通过在 `ContentResolver` 上调用 `query()`，对内容提供程序执行查询。

#### 2.2 应用资源

| 目录        | 资源类型                                                     |
| :---------- | :----------------------------------------------------------- |
| `animator/` | 用于定义[属性动画](https://developer.android.com/guide/topics/graphics/prop-animation)的 XML 文件。 |
| `anim/`     | 用于定义[渐变动画](https://developer.android.com/guide/topics/graphics/view-animation#tween-animation)的 XML 文件。（属性动画也可保存在此目录中，但为了区分这两种类型，属性动画首选 `animator/` 目录。） |
| `color/`    | 用于定义颜色状态列表的 XML 文件。请参阅[颜色状态列表资源](https://developer.android.com/guide/topics/resources/color-list-resource) |
| `drawable/` | 位图文件（`.png`、`.9.png`、`.jpg`、`.gif`）或编译为以下可绘制对象资源子类型的 XML 文件：位图文件九宫格（可调整大小的位图）状态列表形状动画可绘制对象其他可绘制对象请参阅 [Drawable 资源](https://developer.android.com/guide/topics/resources/drawable-resource)。 |
| `mipmap/`   | 适用于不同启动器图标密度的可绘制对象文件。如需了解有关使用 `mipmap/` 文件夹管理启动器图标的详细信息，请参阅[管理项目概览](https://developer.android.com/tools/projects#mipmap)。 |
| `layout/`   | 用于定义用户界面布局的 XML 文件。请参阅[布局资源](https://developer.android.com/guide/topics/resources/layout-resource)。 |
| `menu/`     | 用于定义应用菜单（如选项菜单、上下文菜单或子菜单）的 XML 文件。请参阅[菜单资源](https://developer.android.com/guide/topics/resources/menu-resource)。 |
| `raw/`      | 需以原始形式保存的任意文件。如要使用原始 `InputStream` 打开这些资源，请使用资源 ID（即 `R.raw.*filename*`）调用 `Resources.openRawResource()`。但是，如需访问原始文件名和文件层次结构，则可以考虑将某些资源保存在 `assets/` 目录（而非 `res/raw/`）下。`assets/` 中的文件没有资源 ID，因此您只能使用 `AssetManager` 读取这些文件。 |
| `values/`   | 包含字符串、整型数和颜色等简单值的 XML 文件。其他 `res/` 子目录中的 XML 资源文件会根据 XML 文件名定义单个资源，而 `values/` 目录中的文件可描述多个资源。对于此目录中的文件，`<resources>` 元素的每个子元素均会定义一个资源。例如，`<string>` 元素会创建 `R.string` 资源，`<color>` 元素会创建 `R.color` 资源。由于每个资源均使用自己的 XML 元素进行定义，因此您可以随意命名文件，并在某个文件中放入不同的资源类型。但是，您可能需要将独特的资源类型放在不同的文件中，使其一目了然。例如，对于可在此目录中创建的资源，下面给出了相应的文件名约定：arrays.xml：资源数组（[类型数组](https://developer.android.com/guide/topics/resources/more-resources#TypedArray)）。colors.xml：[颜色值](https://developer.android.com/guide/topics/resources/more-resources#Color)。dimens.xml：[尺寸值](https://developer.android.com/guide/topics/resources/more-resources#Dimension)。strings.xml：[字符串值](https://developer.android.com/guide/topics/resources/string-resource)。styles.xml：[样式](https://developer.android.com/guide/topics/resources/style-resource)。请参阅[字符串资源](https://developer.android.com/guide/topics/resources/string-resource)、[样式资源](https://developer.android.com/guide/topics/resources/style-resource)和[更多资源类型](https://developer.android.com/guide/topics/resources/more-resources)。 |
| `xml/`      | 可在运行时通过调用 `Resources.getXML()` 读取的任意 XML 文件。各种 XML 配置文件（如[可搜索配置](https://developer.android.com/guide/topics/search/searchable-config)）都必须保存在此处。 |
| `font/`     | 带有扩展名的字体文件（如 `.ttf`、`.otf` 或 `.ttc`），或包含 `<font-family>` 元素的 XML 文件。如需详细了解作为资源的字体，请参阅 [XML 中的字体](https://developer.android.com/preview/features/fonts-in-xml)。 |

配置限定符名称

| 配置                 | 限定符值                                                     | 描述                                                         |
| :------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| MCC 和 MNC           | 示例： `mcc310` `mcc310-mnc004` `mcc208-mnc00` 等等          | 移动设备国家代码 (MCC)，（可选）后跟设备 SIM 卡中的移动设备网络代码 (MNC)。例如，`mcc310` 是指美国的任一运营商，`mcc310-mnc004` 是指美国的 Verizon 公司，`mcc208-mnc00` 是指法国的 Orange 公司。如果设备使用无线装置连接（GSM 手机），则 MCC 和 MNC 值均来自 SIM 卡。您也可以单独使用 MCC（例如，将国家/地区特定的合法资源加入应用）。如果只需根据语言指定，则改用*语言和地区*限定符（稍后进行介绍）。如果决定使用 MCC 和 MNC 限定符，请谨慎执行此操作并测试限定符是否按预期工作。另请参阅配置字段 `mcc` 和 `mnc`，二者分别表示当前的移动设备国家代码和移动设备网络代码。 |
| 语言和区域           | 示例： `en` `fr` `en-rUS` `fr-rFR` `fr-rCA` `b+en` `b+en+US` `b+es+419` | 语言通过由两个字母组成的 [ISO 639-1](http://www.loc.gov/standards/iso639-2/php/code_list.php) 语言代码进行定义，可以选择后跟两个字母组成的 [ISO 3166-1-alpha-2](https://www.iso.org/obp/ui/#iso:pub:PUB500001:en) 区域码（前缀用小写字母 `r`）。这些代码*不*区分大小写；`r` 前缀用于区分区域码。您不能单独指定区域。Android 7.0（API 级别 24）引入对 [BCP 47 语言标记](https://tools.ietf.org/html/bcp47)的支持，可供您用来限定特定语言和区域的资源。语言标记由一个或多个子标记序列组成，每个子标记都能优化或缩小由整体标记标识的语言范围。如需了解有关语言标记的详细信息，请参阅[用于标识语言的标记](https://tools.ietf.org/html/rfc5646)。如要使用 BCP 47 语言标记，请将 `b+` 和两个字母的 [ISO 639-1](http://www.loc.gov/standards/iso639-2/php/code_list.php) 语言代码连接；其后还可选择使用其他子标记，用 `+` 分隔即可。如果用户在系统设置中更改语言，则语言标记可能会在应用的生命周期中发生变更。如需了解运行时应用会因此受到何种影响，请参阅[处理运行时变更](https://developer.android.com/guide/topics/resources/runtime-changes)。有关针对其他语言本地化应用的完整指南，请参阅[本地化](https://developer.android.com/guide/topics/resources/localization)。另请参阅 `getLocales()` 方法，了解该方法提供的已定义语言区域列表。此列表包含主要的语言区域。 |
| 布局方向             | `ldrtl` `ldltr`                                              | 应用的布局方向。`ldrtl` 是指“布局方向从右到左”。`ldltr` 是指“布局方向从左到右”（默认的隐式值）。此配置适用于布局、可绘制资源或值等任何资源。例如，若要针对阿拉伯语提供某种特定布局，并针对任何其他“从右到左”的语言（如波斯语或希伯来语）提供某种通用布局，则可提供以下资源：`res/    layout/        main.xml (Default layout)    layout-ar/        main.xml (Specific layout for Arabic)    layout-ldrtl/        main.xml (Any "right-to-left" language, except                  for Arabic, because the "ar" language qualifier                  has a higher precedence.) `**请注意：**如要为应用启用从右到左的布局功能，则必须将 [`supportsRtl`](https://developer.android.com/guide/topics/manifest/application-element#supportsrtl) 设置为 `"true"`，并将 [`targetSdkVersion`](https://developer.android.com/guide/topics/manifest/uses-sdk-element#target) 设置为 17 或更高版本。*此项为 API 级别 17 中的新增配置。* |
| smallestWidth        | `sw<N>dp`  示例： `sw320dp` `sw600dp` `sw720dp` 等等         | 屏幕的基本尺寸，由可用屏幕区域的最小尺寸指定。具体而言，设备的 smallestWidth 是屏幕可用高度和宽度的最小尺寸（您也可将其视为屏幕的“最小可能宽度”）。无论屏幕的当前方向如何，您均可使用此限定符确保应用界面的可用宽度至少为 `<N>` dp。例如，如果布局要求屏幕区域的最小尺寸始终至少为 600dp，则可使用此限定符创建布局资源 `res/layout-sw600dp/`。仅当可用屏幕的最小尺寸至少为 600dp（无论 600dp 表示的边是用户所认为的高度还是宽度）时，系统才会使用这些资源。最小宽度为设备的固定屏幕尺寸特征；**即使屏幕方向发生变化，设备的最小宽度仍会保持不变**。使用最小宽度确定一般屏幕尺寸非常有用，因为宽度通常是设计布局时的驱动因素。界面经常会垂直滚动，但对其水平方向所需要的最小空间具有非常硬性的限制。可用宽度也是确定是否对手持式设备使用单窗格布局，或对平板电脑使用多窗格布局的关键因素。因此，您可能最关注每台设备上的最小可能宽度。设备的最小宽度会将屏幕装饰元素和系统界面考虑在内。例如，如果设备屏幕上的某些永久性界面元素沿着最小宽度轴占据空间，则系统会声明最小宽度小于实际屏幕尺寸，因为这些屏幕像素不适用于您的界面。以下是一些可用于常见屏幕尺寸的值：320，适用于屏幕配置如下的设备：240x320 ldpi（QVGA 手机）320x480 mdpi（手机）480x800 hdpi（高密度手机）480，适用于 480x800 mdpi 之类的屏幕（平板电脑/手机）。600，适用于 600x1024 mdpi 之类的屏幕（7 英寸平板电脑）。720，适用于 720x1280 mdpi 之类的屏幕（10 英寸平板电脑）。当应用为多个资源目录提供不同的 smallestWidth 限定符值时，系统会使用最接近（但未超出）设备 smallestWidth 的值。*此项为 API 级别 13 中的新增配置。*另请参阅 [`android:requiresSmallestWidthDp`](https://developer.android.com/guide/topics/manifest/supports-screens-element#requiresSmallest) 属性（声明与应用兼容的最小 smallestWidth）和 `smallestScreenWidthDp` 配置字段（存放设备的 smallestWidth 值）。如需了解有关不同屏幕设计和使用此限定符的详细信息，请参阅[支持多种屏幕](https://developer.android.com/guide/practices/screens_support)开发者指南。 |
| 可用宽度             | `w<N>dp`  示例： `w720dp` `w1024dp` 等等                     | 指定资源应使用的最小可用屏幕宽度（以 `dp` 为单位，由 `<N>` 值定义）。当屏幕方向在横向和纵向之间切换时，此配置值也会随之变化，以匹配当前的实际宽度。此功能往往有助于确定是否使用多窗格布局，因为即便在使用平板电脑设备时，您通常也不希望竖屏以横屏的方式使用多窗格布局。因此，您可以使用此功能指定布局所需的最小宽度，而无需同时使用屏幕尺寸和屏幕方向限定符。应用为此配置提供具有不同值的多个资源目录时，系统会使用最接近（但未超出）设备当前屏幕宽度的值。此处的值会考虑屏幕装饰元素，因此如果设备显示屏的左边缘或右边缘上有一些永久性 UI 元素，考虑到这些 UI 元素，同时为减少应用的可用空间，设备会使用小于实际屏幕尺寸的宽度值。*此项为 API 级别 13 中的新增配置。*另请参阅 `screenWidthDp` 配置字段，该字段存放当前屏幕宽度。如需了解有关不同屏幕设计和使用此限定符的详细信息，请参阅[支持多种屏幕](https://developer.android.com/guide/practices/screens_support)开发者指南。 |
| 可用高度             | `h<N>dp`  示例： `h720dp` `h1024dp` 等等                     | 指定资源应使用的最小可用屏幕高度（以“dp”为单位，由 `<N>` 值定义）。当屏幕方向在横向和纵向之间切换时，此配置值也会随之变化，以匹配当前的实际高度。对比使用此方式定义布局所需高度与使用 `w<N>dp` 定义所需宽度，二者均非常有用，且都无需同时使用屏幕尺寸和方向限定符。但大多数应用不需要此限定符，因为界面经常垂直滚动，所以高度需更有弹性，而宽度则应更固定。当应用为此配置提供具有不同值的多个资源目录时，系统会使用最接近（但未超出）设备当前屏幕高度的值。此处的值会考虑屏幕装饰元素，因此如果设备显示屏的上边缘或下边缘上有一些永久性 UI 元素，考虑到这些 UI 元素，同时为减少应用的可用空间，设备会使用小于实际屏幕尺寸的高度值。非固定的屏幕装饰元素（例如，全屏时可隐藏的手机状态栏）并*不*在考虑范围内，标题栏或操作栏等窗口装饰亦如此，因此应用必须准备好处理稍小于其指定值的空间。*此项为 API 级别 13 中的新增配置。*另请参阅 `screenHeightDp` 配置字段，该字段存放当前屏幕宽度。如需了解有关不同屏幕设计和使用此限定符的详细信息，请参阅[支持多种屏幕](https://developer.android.com/guide/practices/screens_support)开发者指南。 |
| 屏幕尺寸             | `small` `normal` `large` `xlarge`                            | `small`：尺寸类似于低密度 VGA 屏幕的屏幕。小屏幕的最小布局尺寸约为 320x426 dp。例如，QVGA 低密度屏幕和 VGA 高密度屏幕。`normal`：尺寸类似于中等密度 HVGA 屏幕的屏幕。标准屏幕的最小布局尺寸约为 320x470 dp。例如，WQVGA 低密度屏幕、HVGA 中等密度屏幕、WVGA 高密度屏幕。`large`：尺寸类似于中等密度 VGA 屏幕的屏幕。大屏幕的最小布局尺寸约为 480x640 dp。例如，VGA 和 WVGA 中等密度屏幕。`xlarge`：明显大于传统中等密度 HVGA 屏幕的屏幕。超大屏幕的最小布局尺寸约为 720x960 dp。在大多数情况下，屏幕超大的设备体积太大，不能放进口袋，最常见的是平板式设备。*此项为 API 级别 9 中的新增配置。***请注意：**使用尺寸限定符并不表示资源*仅*适用于该尺寸的屏幕。如果没有为备用资源提供最符合当前设备配置的限定符，则系统可能会使用其中[最匹配](https://developer.android.com/guide/topics/resources/providing-resources#BestMatch)的资源。**注意：**如果所有资源均使用*大于*当前屏幕的尺寸限定符，则系统**不**会使用这些资源，并且应用将在运行时崩溃（例如，如果所有布局资源均以 `xlarge` 限定符标记，但设备是标准尺寸的屏幕）。*此项为 API 级别 4 中的新增配置。*如需了解详细信息，请参阅[支持多种屏幕](https://developer.android.com/guide/practices/screens_support)。另请参阅 `screenLayout` 配置字段，该字段指示屏幕是小尺寸、标准尺寸还是大尺寸。 |
| 屏幕纵横比           | `long` `notlong`                                             | `long`：宽屏，如 WQVGA、WVGA、FWVGA`notlong`：非宽屏，如 QVGA、HVGA 和 VGA*此项为 API 级别 4 中新增配置。*此配置完全基于屏幕的纵横比（宽屏较宽），并且与屏幕方向无关。另请参阅 `screenLayout` 配置字段，该字段指示屏幕是否为宽屏。 |
| 圆形屏幕             | `round` `notround`                                           | `round`：圆形屏幕，例如圆形可穿戴式设备`notround`：方形屏幕，例如手机或平板电脑*此项为 API 级别 23 中的新增配置。*另请参阅 `isScreenRound()` 配置方法，该方法指示屏幕是否为圆形屏幕。 |
| 广色域               | `widecg` `nowidecg`                                          | {@code widecg}：显示广色域，如 Display P3 或 AdobeRGB{@code nowidecg}：显示窄色域，如 sRGB*此项为 API 级别 26 中的新增配置。*另请参阅 `isScreenWideColorGamut()` 配置方法，该方法指示屏幕是否具有广色域。 |
| 高动态范围 (HDR)     | `highdr` `lowdr`                                             | {@code highdr}：显示高动态范围{@code lowdr}：显示低/标准动态范围*此项为 API 级别 26 中的新增配置。*另请参阅 `isScreenHdr()` 配置方法，该方法指示屏幕是否具有 HDR 功能。 |
| 屏幕方向             | `port` `land`                                                | `port`：设备处于纵向（垂直）`land`：设备处于横向状态（水平）如果用户旋转屏幕，此配置可能会在应用生命周期中发生变化。如需了解这会在运行时期间给应用带来哪些影响，请参阅[处理运行时变更](https://developer.android.com/guide/topics/resources/runtime-changes)。另请参阅 `orientation` 配置字段，该字段指示当前的设备方向。 |
| 界面模式             | `car` `desk` `television` `appliance` `watch` `vrheadset`    | `car`：设备正在车载手机座上显示`desk`：设备正在桌面手机座上显示`television`：设备正在通过电视显示内容，通过将界面投影到离用户较远的大屏幕上，为用户提供“十英尺”体验。主要面向遥控交互或其他非触控式交互`appliance`：设备正在用作没有显示屏的装置`watch`：设备配有显示屏，并且可戴在手腕上`vrheadset`：设备正在通过虚拟现实耳机显示内容*此项为 API 级别 8 中的新增配置，API 13 中的新增电视配置，API 20 中的新增手表配置。*如需了解应用在设备插入基座或从中移除时的响应方式，请阅读[确定并监控插接状态和类型](https://developer.android.com/training/monitoring-device-state/docking-monitoring)。如果用户将设备插入基座，此配置可能会在应用生命周期中发生变化。您可以使用 `UiModeManager` 启用或禁用其中的部分模式。如需了解这会在运行时期间给应用带来哪些影响，请参阅[处理运行时变更](https://developer.android.com/guide/topics/resources/runtime-changes)。 |
| 夜间模式             | `night` `notnight`                                           | `night`：夜间`notnight`：白天*此项为 API 级别 8 中的新增配置。*如果夜间模式停留在自动模式（默认），此配置可能会在应用生命周期中发生变化。在此情况下，该模式会根据当天的时间进行调整。您可以使用 `UiModeManager` 启用或禁用此模式。如需了解这会在运行时期间给应用带来哪些影响，请参阅[处理运行时变更](https://developer.android.com/guide/topics/resources/runtime-changes)。 |
| 屏幕像素密度 (dpi)   | `ldpi` `mdpi` `hdpi` `xhdpi` `xxhdpi` `xxxhdpi` `nodpi` `tvdpi` `anydpi` `*nnn*dpi` | `ldpi`：低密度屏幕；约为 120dpi。`mdpi`：中等密度（传统 HVGA）屏幕；约为 160dpi。`hdpi`：高密度屏幕；约为 240dpi。`xhdpi`：超高密度屏幕；约为 320dpi。*此项为 API 级别 8 中的新增配置*`xxhdpi`：绝高密度屏幕；约为 480dpi。*此项为 API 级别 16 中的新增配置*`xxxhdpi`：极高密度屏幕使用（仅限启动器图标，请参阅*支持多种屏幕*中的[注释](https://developer.android.com/guide/practices/screens_support#xxxhdpi-note)）；约为 640dpi。*此项为 API 级别 18 中的新增配置*`nodpi`：可用于您不希望为匹配设备密度而进行缩放的位图资源。`tvdpi`：密度介于 mdpi 和 hdpi 之间的屏幕；约为 213dpi。此限定符并非指“基本”密度的屏幕。它主要用于电视，且大多数应用都不使用该密度 — 大多数应用只会使用 mdpi 和 hdpi 资源，而且系统将根据需要对这些资源进行缩放。*此项为 API 级别 13 中的新增配置*`anydpi`：此限定符适合所有屏幕密度，其优先级高于其他限定符。这非常适用于[矢量可绘制对象](https://developer.android.com/training/material/drawables#VectorDrawables)。*此项为 API 级别 21 中的新增配置*`*nnn*dpi`：用于表示非标准密度，其中 `*nnn*` 是正整数屏幕密度。此限定符不适用于大多数情况。使用标准密度存储分区，可显著减少因支持市场上各种设备屏幕密度而产生的开销。六个基本密度之间的缩放比为 3:4:6:8:12:16（忽略 tvdpi 密度）。因此，9x9 (ldpi) 位图相当于 12x12 (mdpi)、18x18 (hdpi)、24x24 (xhdpi) 位图，依此类推。如果您认为图像资源在电视或其他某些设备上的呈现效果不够好，进而想尝试使用 tvdpi 资源，则缩放系数应为 1.33*mdpi。例如，mdpi 屏幕的 100px x 100px 图像应相当于 tvdpi 屏幕的 133px x 133px 图像。**请注意：**使用密度限定符并不表示资源*仅*适用于该密度的屏幕。如果没有为备用资源提供最符合当前设备配置的限定符，则系统可能使用其中[最匹配](https://developer.android.com/guide/topics/resources/providing-resources#BestMatch)的资源。如需详细了解如何处理不同屏幕密度以及 Android 如何缩放位图以适应当前密度，请参阅[支持多种屏幕](https://developer.android.com/guide/practices/screens_support)。 |
| 触摸屏类型           | `notouch` `finger`                                           | `notouch`：设备没有触摸屏。`finger`：设备有一个专供用户通过手指直接进行交互的触摸屏。另请参阅 `touchscreen` 配置字段，该字段指示设备上的触摸屏类型。 |
| 键盘可用性           | `keysexposed` `keyshidden` `keyssoft`                        | `keysexposed`：设备拥有可用的键盘。如果设备启用了软键盘（不无可能），那么即使用户*未*找到硬键盘，或者该设备没有硬键盘，也可使用此限定符。如果未提供或已禁用软键盘，则只有在配备硬键盘的情况下才可使用此限定符。`keyshidden`：设备具有可用的硬键盘，但其处于隐藏状态，*且*设备*未*启用软键盘。`keyssoft`：设备已启用软键盘（无论是否可见）。如果您提供了 `keysexposed` 资源，但未提供 `keyssoft` 资源，则无论键盘是否可见，只要系统已启用软键盘，其便会使用 `keysexposed` 资源。如果用户打开硬键盘，此配置可能会在应用生命周期中发生变化。如需了解这会在运行时期间给应用带来哪些影响，请参阅[处理运行时变更](https://developer.android.com/guide/topics/resources/runtime-changes)。另请参阅配置字段 `hardKeyboardHidden` 和 `keyboardHidden`，二者分别指示硬键盘的可见性和任一键盘（包括软键盘）的可见性。 |
| 主要的文本输入法     | `nokeys` `qwerty` `12key`                                    | `nokeys`：设备没有用于文本输入的硬按键。`qwerty`：设备拥有标准硬键盘（无论是否对用户可见）。`12key`：设备拥有 12 键硬键盘（无论是否对用户可见）。另请参阅 `keyboard` 配置字段，该字段指示可用的主要文本输入法。 |
| 导航键可用性         | `navexposed` `navhidden`                                     | `navexposed`：导航键可供用户使用。`navhidden`：导航键不可用（例如，在密封盖子后面）。如果用户显示导航键，此配置可能会在应用生命周期中发生变化。如需了解这会在运行时期间给应用带来哪些影响，请参阅[处理运行时变更](https://developer.android.com/guide/topics/resources/runtime-changes)。另请参阅 `navigationHidden` 配置字段，该字段指示导航键是否处于隐藏状态。 |
| 主要的非触摸导航方法 | `nonav` `dpad` `trackball` `wheel`                           | `nonav`：除了使用触摸屏以外，设备没有其他导航设施。`dpad`：设备具有用于导航的方向键。`trackball`：设备具有用于导航的轨迹球。`wheel`：设备具有用于导航的方向盘（不常见）。另请参阅 `navigation` 配置字段，该字段指示可用的导航方法类型。 |
| 平台版本（API 级别） | 示例： `v3` `v4` `v7` 等等                                   | 设备支持的 API 级别。例如，`v1` 对应 API 级别 1（装有 Android 1.0 或更高版本系统的设备），`v4` 对应 API 级别 4（装有 Android 1.6 或更高版本系统的设备）。如需了解有关这些值的详细信息，请参阅 [Android API 级别](https://developer.android.com/guide/topics/manifest/uses-sdk-element#ApiLevels)文档。 |

[关于使用配置限定符名称的规则](https://developer.android.com/guide/topics/resources/providing-resources)：

- 您可以为单组资源指定多个限定符，并使用短划线分隔。例如，`drawable-en-rUS-land` 适用于屏幕方向为横向的美国英语设备。
- 这些限定符必须遵循 表 2  中列出的顺序。例如：
  - 错误：`drawable-hdpi-port/`
  - 正确：`drawable-port-hdpi/`
- 不能嵌套备用资源目录。例如，您的目录不能为 `res/drawable/drawable-en/`。
- 值不区分大小写。在处理之前，资源编译器会将目录名称转换为小写，以免不区分大小写的文件系统出现问题。名称中使用的所有大写字母只是为了便于认读。
- 每种限定符类型仅支持一个值。例如，若要对西班牙语和法语使用相同的可绘制对象文件，则您*不能*拥有名为 `drawable-rES-rFR/` 的目录，而是需要两个包含相应文件的资源目录，如 `drawable-rES/` 和 `drawable-rFR/`。然而，您实际无需在两处复制相同的文件。相反，您可以创建指向资源的别名。请参阅下面的[创建别名资源](https://developer.android.com/guide/topics/resources/providing-resources#AliasResources)。

**声明由 Activity 处理配置变更.**

`"orientation"` 值可在屏幕方向发生变更时阻止重启。

`"screenSize"` 值也可在屏幕方向发生变更时阻止重启，但仅适用于 Android 3.2（API 级别 13）及以上版本的系统。若想在应用中手动处理配置变更，您必须在 `android:configChanges` 属性中声明 `"orientation"` 和 `"screenSize"` 值。

`"keyboardHidden"` 值可在键盘可用性发生变更时阻止重启。

```xml
<activity android:name=".MyActivity"
          android:configChanges="orientation|keyboardHidden"
          android:label="@string/app_name">
```

```java
@Override
public void onConfigurationChanged(Configuration newConfig) {
    super.onConfigurationChanged(newConfig);

    // Checks the orientation of the screen
    if (newConfig.orientation == Configuration.ORIENTATION_LANDSCAPE) {
        Toast.makeText(this, "landscape", Toast.LENGTH_SHORT).show();
    } else if (newConfig.orientation == Configuration.ORIENTATION_PORTRAIT){
        Toast.makeText(this, "portrait", Toast.LENGTH_SHORT).show();
    }
}
```

#### [2.3 动画](https://developer.android.com/guide/topics/resources/animation-resource)

##### [属性动画](https://developer.android.com/guide/topics/resources/animation-resource#Property)

通过使用 `Animator` 在设定的时间段内修改对象的属性值来创建动画。

在 XML 中定义的动画，用于在设定的一段时间内修改目标对象的属性，例如背景颜色或 Alpha 值。

保存在 `res/animator/property_animator.xml` 的 XML 文件：

```xml
 <set android:ordering="sequentially">
        <set>
            <objectAnimator
                android:propertyName="x"
                android:duration="500"
                android:valueTo="400"
                android:valueType="intType"/>
            <objectAnimator
                android:propertyName="y"
                android:duration="500"
                android:valueTo="300"
                android:valueType="intType"/>
        </set>
        <objectAnimator
            android:propertyName="alpha"
            android:duration="500"
            android:valueTo="1f"/>
    </set>
```

```java
 AnimatorSet set = (AnimatorSet) AnimatorInflater.loadAnimator(myContext,
        R.animator.property_animator);
    set.setTarget(myObject);
    set.start();
```



##### [视图动画](https://developer.android.com/guide/topics/resources/animation-resource#View)

视图动画框架可支持补间动画和逐帧动画，两者都可以在 XML 中声明.

使用视图动画框架可以创建两种类型的动画：

- [补间动画](https://developer.android.com/guide/topics/resources/animation-resource#Tween)：通过使用 `Animation` 对单张图片执行一系列转换来创建动画.

  在 XML 中定义的动画，用于对图形执行旋转、淡出、移动和拉伸等转换。

  res/anim/*filename*.xml

  ```xml
   <?xml version="1.0" encoding="utf-8"?>
      <set xmlns:android="http://schemas.android.com/apk/res/android"
          android:interpolator="@[package:]anim/interpolator_resource"
          android:shareInterpolator=["true" | "false"] >
          <alpha
              android:fromAlpha="float"
              android:toAlpha="float" />
          <scale
              android:fromXScale="float"
              android:toXScale="float"
              android:fromYScale="float"
              android:toYScale="float"
              android:pivotX="float"
              android:pivotY="float" />
          <translate
              android:fromXDelta="float"
              android:toXDelta="float"
              android:fromYDelta="float"
              android:toYDelta="float" />
          <rotate
              android:fromDegrees="float"
              android:toDegrees="float"
              android:pivotX="float"
              android:pivotY="float" />
          <set>
              ...
          </set>
      </set>
  ```

  ```java
     ImageView image = (ImageView) findViewById(R.id.image);
      Animation hyperspaceJump = AnimationUtils.loadAnimation(this, R.anim.hyperspace_jump);
      image.startAnimation(hyperspaceJump);
  ```

  | 插值器类                           | 资源 ID                                            |
  | :--------------------------------- | :------------------------------------------------- |
  | `AccelerateDecelerateInterpolator` | `@android:anim/accelerate_decelerate_interpolator` |
  | `AccelerateInterpolator`           | `@android:anim/accelerate_interpolator`            |
  | `AnticipateInterpolator`           | `@android:anim/anticipate_interpolator`            |
  | `AnticipateOvershootInterpolator`  | `@android:anim/anticipate_overshoot_interpolator`  |
  | `BounceInterpolator`               | `@android:anim/bounce_interpolator`                |
  | `CycleInterpolator`                | `@android:anim/cycle_interpolator`                 |
  | `DecelerateInterpolator`           | `@android:anim/decelerate_interpolator`            |
  | `LinearInterpolator`               | `@android:anim/linear_interpolator`                |
  | `OvershootInterpolator`            | `@android:anim/overshoot_interpolator`             |

  

- [帧动画](https://developer.android.com/guide/topics/resources/animation-resource#Frame)：通过使用 `AnimationDrawable` 按顺序显示一系列图片来创建动画。

  `res/drawable/rocket.xml` **的 XML 文件：**

  ```xml
  <?xml version="1.0" encoding="utf-8"?>
      <animation-list xmlns:android="http://schemas.android.com/apk/res/android"
          android:oneshot="false">
          <item android:drawable="@drawable/rocket_thrust1" android:duration="200" />
          <item android:drawable="@drawable/rocket_thrust2" android:duration="200" />
          <item android:drawable="@drawable/rocket_thrust3" android:duration="200" />
      </animation-list>
  ```

  ```java
    ImageView rocketImage = (ImageView) findViewById(R.id.rocket_image);
      rocketImage.setBackgroundResource(R.drawable.rocket_thrust);
  
      rocketAnimation = rocketImage.getBackground();
      if (rocketAnimation instanceof Animatable) {
          ((Animatable)rocketAnimation).start();
      }
  ```



其他：

```xml
    <?xml version="1.0" encoding="utf-8"?>
    <selector xmlns:android="http://schemas.android.com/apk/res/android" >
        <item
            android:color="hex_color"
            android:state_pressed=["true" | "false"]
            android:state_focused=["true" | "false"]
            android:state_selected=["true" | "false"]
            android:state_checkable=["true" | "false"]
            android:state_checked=["true" | "false"]
            android:state_enabled=["true" | "false"]
            android:state_window_focused=["true" | "false"] />
    </selector>
```

字符串处理：

支持以下 HTML 元素：

- 粗体：<b>、<em>

- 斜体：<i>、<cite>、<dfn>

- 文本放大 25%：<big>

- 文本缩小 20%：<small>

- 设置字体属性：<font face=”font_family“ color=”hex_color”>。可能的字体系列示例包括 `monospace`、`serif` 和 `sans_serif`。

- 设置等宽字体系列：<tt>

- 删除线：<s>、<strike>、<del>

- 下划线：<u>

- 上标：<sup>

- 下标：<sub>

- 列表标记：<ul>、<li>

- 换行符：<br>

- 区隔标记：<div>

- CSS 样式：<span style=”color|background_color|text-decoration”>

- 段落：<p dir=”rtl | ltr” style=”…”>

  

```xml
<string name="welcome_messages">Hello, %1$s! You have %2$d new messages.</string>

String text = getString(R.string.welcome_messages, username, mailCount);

```

#### 2.4 [activity](https://developer.android.com/guide/topics/manifest/activity-element)：

```xml
<activity android:allowEmbedded=["true" | "false"]
          android:allowTaskReparenting=["true" | "false"]
          android:alwaysRetainTaskState=["true" | "false"]
          android:autoRemoveFromRecents=["true" | "false"]
          android:banner="drawable resource"
          android:clearTaskOnLaunch=["true" | "false"]
          android:colorMode=[ "hdr" | "wideColorGamut"]
          android:configChanges=["mcc", "mnc", "locale",
                                 "touchscreen", "keyboard", "keyboardHidden",
                                 "navigation", "screenLayout", "fontScale",
                                 "uiMode", "orientation", "density",
                                 "screenSize", "smallestScreenSize"]
          android:directBootAware=["true" | "false"]
          android:documentLaunchMode=["intoExisting" | "always" |
                                  "none" | "never"]
          android:enabled=["true" | "false"]
          android:excludeFromRecents=["true" | "false"]
          android:exported=["true" | "false"]
          android:finishOnTaskLaunch=["true" | "false"]
          android:hardwareAccelerated=["true" | "false"]
          android:icon="drawable resource"
          android:immersive=["true" | "false"]
          android:label="string resource"
          android:launchMode=["standard" | "singleTop" |
                              "singleTask" | "singleInstance"]
          android:lockTaskMode=["normal" | "never" |
                              "if_whitelisted" | "always"]
          android:maxRecents="integer"
          android:maxAspectRatio="float"
          android:multiprocess=["true" | "false"]
          android:name="string"
          android:noHistory=["true" | "false"]  
          android:parentActivityName="string" 
          android:persistableMode=["persistRootOnly" | 
                                   "persistAcrossReboots" | "persistNever"]
          android:permission="string"
          android:process="string"
          android:relinquishTaskIdentity=["true" | "false"]
          android:resizeableActivity=["true" | "false"]
          android:screenOrientation=["unspecified" | "behind" |
                                     "landscape" | "portrait" |
                                     "reverseLandscape" | "reversePortrait" |
                                     "sensorLandscape" | "sensorPortrait" |
                                     "userLandscape" | "userPortrait" |
                                     "sensor" | "fullSensor" | "nosensor" |
                                     "user" | "fullUser" | "locked"]
          android:showForAllUsers=["true" | "false"]
          android:stateNotNeeded=["true" | "false"]
          android:supportsPictureInPicture=["true" | "false"]
          android:taskAffinity="string"
          android:theme="resource or theme"
          android:uiOptions=["none" | "splitActionBarWhenNarrow"]
          android:windowSoftInputMode=["stateUnspecified",
                                       "stateUnchanged", "stateHidden",
                                       "stateAlwaysHidden", "stateVisible",
                                       "stateAlwaysVisible", "adjustUnspecified",
                                       "adjustResize", "adjustPan"] >   
    . . .
</activity>
```

#### 2.5 支持不同的像素密度

![img](https://developer.android.com/images/screens_support/densities-phone_2x.png)

   **图 1.** 尺寸相同的两个屏幕可能具有不同数量的像素

要在密度不同的屏幕上保留界面的可见尺寸，您必须使用**密度无关像素 (dp) **作为度量单位来设计界面。dp 是一个虚拟像素单位，1 dp 约等于中密度屏幕（160dpi；“基准”密度）上的 1 像素。对于其他每个密度，Android 会将此值转换为相应的实际像素数。

例如，考虑图 1 中的两部设备。如果将某个视图定义为“100px”宽，那么它在左侧设备上看起来要大得多。因此，您必须改用“100dp”来确保它在两个屏幕上看起来大小相同。



- 将 dp 单位转换为像素单位

在某些情况下，您需要以 `dp` 表示尺寸，然后将其转换为像素。dp 单位转换为屏幕像素很简单：

```
px = dp * (dpi / 160)
```

假设在某一应用中，用户的手指至少移动 16 像素之后，系统才会识别出滚动或滑动手势。在基线屏幕上，用户必须移动 `16 pixels / 160 dpi`（等于一英寸的 1/10 或 2.5 毫米），系统才会识别该手势。而在配备高密度显示屏 (240dpi) 的设备上，用户的手指必须至少移动 `16 pixels / 240 dpi`，相当于 1 英寸的 1/15（1.7 毫米）。此距离短得多，因此用户会感觉应用在该设备上更灵敏。

要解决此问题，必须在代码中以 `dp` 表示手势阈值，然后再转换为实际像素。例如：

```java
// The gesture threshold expressed in dp
    private static final float GESTURE_THRESHOLD_DP = 16.0f;

    // Get the screen's density scale
    final float scale = getResources().getDisplayMetrics().density;
    // Convert the dps to pixels, based on density scale
    mGestureThreshold = (int) (GESTURE_THRESHOLD_DP * scale + 0.5f);

    // Use mGestureThreshold as a distance in pixels...
    
```

`DisplayMetrics.density` 字段根据当前像素密度指定将 `dp` 单位转换为像素时所必须使用的缩放系数。在中密度屏幕上，`DisplayMetrics.density` 等于 1.0；在高密度屏幕上，它等于 1.5；在超高密度屏幕上，等于 2.0；在低密度屏幕上，等于 0.75。此数字是一个系数，用其乘以 `dp` 单位，即可得出当前屏幕的实际像素数。

![img](https://developer.android.com/images/screens_support/devices-density_2x.png)

| 密度限定符 | 说明                                                         |
| :--------- | :----------------------------------------------------------- |
| `ldpi`     | 适用于低密度 (ldpi) 屏幕 (~ 120dpi) 的资源。                 |
| `mdpi`     | 适用于中密度 (mdpi) 屏幕 (~ 160dpi) 的资源（这是基准密度）。 |
| `hdpi`     | 适用于高密度 (hdpi) 屏幕 (~ 240dpi) 的资源。                 |
| `xhdpi`    | 适用于加高 (xhdpi) 密度屏幕 (~ 320dpi) 的资源。              |
| `xxhdpi`   | 适用于超超高密度 (xxhdpi) 屏幕 (~ 480dpi) 的资源。           |
| `xxxhdpi`  | 适用于超超超高密度 (xxxhdpi) 屏幕 (~ 640dpi) 的资源。        |
| `nodpi`    | 适用于所有密度的资源。这些是与密度无关的资源。无论当前屏幕的密度是多少，系统都不会缩放以此限定符标记的资源。 |
| `tvdpi`    | 适用于密度介于 mdpi 和 hdpi 之间的屏幕（约 213dpi）的资源。这不属于“主要”密度组。它主要用于电视，而大多数应用都不需要它。对于大多数应用而言，提供 mdpi 和 hdpi 资源便已足够，系统将视情况对其进行缩放。如果您发现有必要提供 tvdpi 资源，应按一个系数来确定其大小，即 1.33*mdpi。例如，如果某张图片在 mdpi 屏幕上的大小为 100px x 100px，那么它在 tvdpi 屏幕上的大小应该为 133px x 133px。 |



### 3、Android TV

```xml
   <manifest>
        <uses-feature android:name="android.hardware.touchscreen"
                  android:required="false" />
        ...
    </manifest>
    
```

**注意**：您必须在应用清单中声明触摸屏并非必要条件（如本示例代码中所示），否则您的应用将不会出现在 TV 设备上的 Google Play 中。

使用 Android 的标准字号：

```xml
    <TextView
          android:id="@+id/atext"
          android:layout_width="wrap_content"
          android:layout_height="wrap_content"
          android:gravity="center_vertical"
          android:singleLine="true"
          android:textAppearance="?android:attr/textAppearanceMedium"/>
```

### 4、Activity

例如，假设您的应用想要使用一个名为 SocialApp 的应用在社交媒体上分享文章，则 SocialApp 本身必须定义调用它的应用所需具备的权限：

```xml
<manifest>
    <activity android:name="...."
       android:permission=”com.google.socialapp.permission.SHARE_POST”

    />
```

然后，为了能够调用 SocialApp，您的应用必须匹配 SocialApp 清单中设置的权限：

```xml
    <manifest>
       <uses-permission android:name="com.google.socialapp.permission.SHARE_POST" />
    </manifest>
```



#### 4.1 Activity 生命周期

- onCreate()

`setContentView()`

`onCreate()` 完成后，下一个回调将是 `onStart()`。

- onStart()

 Activity 进入“已开始”状态时， 对用户可见，**会非常快速**地完成，接着进入“已恢复”状态，系统将调用 `onResume()`。

例如，应用通过此方法来**初始化**维护界面的代码。

- onResume()

进入“已恢复”状态时来到前台，应用与用户互动的状态，应用会一直保持这种状态，**直到**某些事件发生，让**焦点远离应用**。此类事件包括接到来电、用户导航到另一个 Activity，或设备屏幕关闭。

从onPause()回来，初始化在 [`onPause()`](https://developer.android.com/reference/android/app/Activity#onPause()) 期间释放的组件。

例如启动相机预览。

`onResume()` 回调后面总是跟着 `onPause()` 回调。

```java
public class CameraComponent implements LifecycleObserver { 
    ... 
    @OnLifecycleEvent(Lifecycle.Event.ON_RESUME)
    public void initializeCamera() {
        if (camera == null) {
            getCamera();
        }
    } 
}
```

- onPause()

当 Activity 失去焦点并进入“已暂停”状态时，离开。 

`onPause()` 执行完毕后，下一个回调为 `onStop()`或 `onResume()`，具体取决于 Activity 进入“已暂停”状态后发生的情况。

​	例如：
​		如 onResume() 部分所述，某个事件会中断应用执行。这是最常见的情况。
​		在 Android 7.0（API 级别 24）或更高版本中，有多个应用在多窗口模式下运行。无论何时，都只有一个应用（窗口）可以拥有焦点，因此系统会暂停所有其他应用。
​		有新的半透明 Activity（例如对话框）处于开启状态。只要 Activity 仍然部分可见但并未处于焦点之中，它便会一直暂停。

使用 [`onPause()`](https://developer.android.com/reference/android/app/Activity#onPause()) 方法释放系统资源、传感器（例如 GPS）手柄

```java
public class JavaCameraComponent implements LifecycleObserver { 
    @OnLifecycleEvent(Lifecycle.Event.ON_PAUSE)
    public void releaseCamera() {
        if (camera != null) {
            camera.release();
            camera = null;
        }
    } 
}
```

- onStop()

进入“已停止”状态。对用户可见。

在 `onStop()` 方法中

1、应用应释放或调整在应用对用户不可见时的无用资源。例如，应用可以暂停动画效果，或从精确位置更新切换到粗略位置更新。使用 `onStop()` 而非 `onPause()` 可确保与界面相关的工作继续进行。

2、执行 CPU 相对密集的关闭操作。例如，如果您无法找到更合适的时机来将信息保存到数据库。 

系统调用的下一个回调将是 `onRestart()`（如果 Activity 重新与用户互动）或者 `onDestroy()`（如果 Activity 彻底终止）。

例如：当新启动的 Activity 覆盖整个屏幕时，可能会发生这种情况。

将草稿笔记内容保存到持久性存储空间中。

```java
@Override
protected void onStop() {
    // call the superclass method first
    super.onStop(); 
    // save the note's current draft, because the activity is stopping
    // and we want to be sure the current note progress isn't lost.
    ContentValues values = new ContentValues();
    values.put(NotePad.Notes.COLUMN_NAME_NOTE, getCurrentNoteText());
    values.put(NotePad.Notes.COLUMN_NAME_TITLE, getCurrentNoteTitle());

    // do this update in background on an AsyncQueryHandler or equivalent
    asyncQueryHandler.startUpdate (
            mToken,  // int token to correlate calls
            null,    // cookie, not used here
            uri,    // The URI for the note to update.
            values,  // The map of column names and new values to apply to them.
            null,    // No SELECT criteria are used.
            null     // No WHERE columns are used.
    );
}
```

- onRestart()

当处于“已停止”状态的 Activity 即将重启时，系统就会调用此回调。`onRestart()` 会从 Activity 停止时的状态恢复 Activity。

此回调后面总是跟着 `onStart()`。

- onDestroy()

系统会在销毁 Activity 之前调用此回调。

1. Activity 即将结束（由于用户彻底关闭 Activity 或由于系统为 Activity 调用 [`finish()`](https://developer.android.com/reference/android/app/Activity#finish())）
2. 由于配置变更（例如设备旋转或多窗口模式），系统暂时销毁 Activity

此回调是 Activity 接收的最后一个回调。通常，实现 `onDestroy()` 是为了确保在销毁 Activity 或包含该 Activity 的进程时释放该 Activity 的所有资源。

可以使用 [`isFinishing()`](https://developer.android.com/reference/android/app/Activity#isFinishing()) 方法区分这两种情况。



- 状态变化

可以单独使用 onSaveInstanceState() 使界面状态在**配置更改**和**系统启动的进程被终止**时保持不变。[`Bundle`](https://developer.android.com/reference/android/os/Bundle) 对象并不适合保留大量数据，因为它需要在主线程上进行序列化处理并占用系统进程内存。如需保存大量数据，您应组合使用持久性本地存储、[`onSaveInstanceState()`](https://developer.android.com/reference/android/app/Activity#onSaveInstanceState(android.os.Bundle)) 方法和 [`ViewModel`](https://developer.android.com/reference/androidx/lifecycle/ViewModel) 类来保存数据。

**注意**：当用户显式关闭 Activity 时，或者在其他情况下调用 `finish()` 时，系统不会调用 [`onSaveInstanceState()`](https://developer.android.com/reference/android/app/Activity#onSaveInstanceState(android.os.Bundle))。

```java
static final String STATE_SCORE = "playerScore";
static final String STATE_LEVEL = "playerLevel"; 
@Override
public void onSaveInstanceState(Bundle savedInstanceState) {
    // Save the user's current game state
    savedInstanceState.putInt(STATE_SCORE, currentScore);
    savedInstanceState.putInt(STATE_LEVEL, currentLevel); 
    // Always call the superclass so it can save the view hierarchy state
    super.onSaveInstanceState(savedInstanceState);
}
```

```java
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState); // Always call the superclass first

    // Check whether we're recreating a previously destroyed instance
    if (savedInstanceState != null) {
        // Restore value of members from saved state
        currentScore = savedInstanceState.getInt(STATE_SCORE);
        currentLevel = savedInstanceState.getInt(STATE_LEVEL);
    } else {
        // Probably initialize members with default values for a new instance
    }
    // ...
}
```

或者

```java
public void onRestoreInstanceState(Bundle savedInstanceState) {
    // Always call the superclass so it can restore the view hierarchy
    super.onRestoreInstanceState(savedInstanceState);

    // Restore state members from saved instance
    currentScore = savedInstanceState.getInt(STATE_SCORE);
    currentLevel = savedInstanceState.getInt(STATE_LEVEL);
}
```

#### 4.2 状态变更

有很多事件会触发配置更改。最显著的例子或许是**横屏和竖屏之间**的屏幕方向变化。其他情况，如语言或输入设备的改变等，也可能导致配置更改。

当配置发生更改时，Activity 会被销毁并重新创建。原始 Activity 实例将触发 [`onPause()`](https://developer.android.com/reference/android/app/Activity#onpause)、[`onStop()`](https://developer.android.com/reference/android/app/Activity#onstop) 和 [`onDestroy()`](https://developer.android.com/reference/android/app/Activity#ondestroy) 回调。系统将创建新的 Activity 实例，并触发 [`onCreate()`](https://developer.android.com/reference/android/app/Activity#onCreate(android.os.Bundle))、[`onStart()`](https://developer.android.com/reference/android/app/Activity#onstart) 和 [`onResume()`](https://developer.android.com/reference/android/app/Activity#onResume()) 回调。

如果有新的 Activity 或对话框出现在前台，夺取了焦点且完全覆盖了正在进行的 Activity，则被覆盖的 Activity 会失去焦点并进入“已停止”状态。然后，系统会快速地接连调用 `onPause()` 和 `onStop()`。

当被覆盖的 Activity 的同一实例返回到前台时，系统会对该 Activity 调用 `onRestart()`、`onStart()` 和 `onResume()`。如果被覆盖的 Activity 的新实例进入后台，则系统不会调用 onRestart()，而只会调用 `onStart()` 和 `onResume()`。

#### 4.3 [Fragment](https://developer.android.com/guide/components/fragments)

`inflate()` 方法带有三个参数：

- 您想要扩展的布局的资源 ID。

- 将作为扩展布局父项的 `ViewGroup`。传递 `container` 对系统向扩展布局的根视图（由其所属的父视图指定）应用布局参数具有重要意义。

- 指示是否应在扩展期间将扩展布局附加至 `ViewGroup`（第二个参数）的布尔值。（在本例中，此值为 false，因为系统已将扩展布局插入 `container`，而**传递 true 值会在最终布局中创建一个多余的视图组。**）

  ```java
  public static class ExampleFragment extends Fragment {
      @Override
      public View onCreateView(LayoutInflater inflater, ViewGroup container,
                               Bundle savedInstanceState) {
          // Inflate the layout for this fragment
          return inflater.inflate(R.layout.example_fragment, container, false);
      }
  }
  ```



```java
// Create new fragment and transaction
Fragment newFragment = new ExampleFragment();
FragmentTransaction transaction = getSupportFragmentManager().beginTransaction();

// Replace whatever is in the fragment_container view with this fragment,
// and add the transaction to the back stack
transaction.replace(R.id.fragment_container, newFragment);
transaction.addToBackStack(null);

// Commit the transaction
transaction.commit();
```

调用 `addToBackStack()`，您可以将替换事务保存到返回栈，以便用户能够通过按*返回*按钮撤消事务并回退到上一片段。

> 例如，如果某个新闻应用的 Activity 有两个片段，其中一个用于显示文章列表（片段 A），另一个用于显示文章（片段 B），则片段 A 必须在列表项被选定后告知 Activity，以便它告知片段 B 显示该文章。

```java
public static class FragmentA extends ListFragment {
    ...
    // Container Activity must implement this interface
    public interface OnArticleSelectedListener {
        public void onArticleSelected(Uri articleUri);
    }
    ...
    OnArticleSelectedListener listener;
    ...
    @Override
    public void onAttach(Context context) {
        super.onAttach(context);
        try {
            listener = (OnArticleSelectedListener) context;
        } catch (ClassCastException e) {
            throw new ClassCastException(context.toString() + " must implement OnArticleSelectedListener");
        }
    }
    
    @Override
    public void onListItemClick(ListView l, View v, int position, long id) {
        // Append the clicked item's row ID with the content provider Uri
        Uri noteUri = ContentUris.withAppendedId(ArticleColumns.CONTENT_URI, id);
        // Send the event and Uri to the host activity
        listener.onArticleSelected(noteUri);
    }
}
```

```
onAttach()
```

在片段已与 Activity 关联时进行调用（`Activity` 传递到此方法内）。

```
onCreateView()
```

调用它可创建与片段关联的视图层次结构。

```
onActivityCreated()
```

当 Activity 的 `onCreate()` 方法已返回时进行调用。

```
onDestroyView()
```

在移除与片段关联的视图层次结构时进行调用。

```
onDetach()
```

在取消片段与 Activity 的关联时进行调用。



- Fragment 之间传递数据

Fragment A 

```java
@Override
public void onCreate(@Nullable Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    getParentFragmentManager().setFragmentResultListener("key", this, new FragmentResultListener() {
        @Override
        public void onFragmentResult(@NonNull String key, @NonNull Bundle bundle) {
            // We use a String here, but any type that can be put in a Bundle is supported
            String result = bundle.getString("bundleKey");
            // Do something with the result...
        }
    });
}
```

Fragment B

```java
button.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View v) {
        Bundle result = new Bundle();
        result.putString("bundleKey", "result");
        getParentFragmentManager().setFragmentResult("requestKey", result);
    }
});
```

如需在 Fragment 之间通信，建议创建一个共享的 [`ViewModel`](https://developer.android.com/reference/androidx/lifecycle/ViewModel) 对象。两个 Fragment 都可以通过所在的 Activity 访问 ViewModel。Fragment 可在 ViewModel 内更新数据，如果使用 [`LiveData`](https://developer.android.com/reference/androidx/lifecycle/LiveData) 公开该数据，新状态会被推送至其他 Fragment（只要它正在从 ViewModel 观察 LiveData）。要了解如何实现这种通信机制，请参阅 [ViewModel 指南](https://developer.android.com/topic/libraries/architecture/viewmodel)中的“在 Fragment 之间共享数据”部分。



#### 4.4 Intent

**注意：**为了确保应用的安全性，启动 `Service` 时，请始终使用显式 Intent，且不要为服务声明 Intent 过滤器。使用隐式 Intent 启动服务存在安全隐患，因为您无法确定哪些服务将响应 Intent，且用户无法看到哪些服务已启动。从 Android 5.0（API 级别 21）开始，如果使用隐式 Intent 调用 `bindService()`，系统会抛出异常。

```java
// Create the text message with a string
Intent sendIntent = new Intent();
sendIntent.setAction(Intent.ACTION_SEND);
sendIntent.putExtra(Intent.EXTRA_TEXT, textMessage);
sendIntent.setType("text/plain");

// Verify that the intent will resolve to an activity
if (sendIntent.resolveActivity(getPackageManager()) != null) {
    startActivity(sendIntent);
}
```



##### 4.4.1相机

**请注意：**当您使用 `ACTION_IMAGE_CAPTURE` 拍摄照片时，相机可能还会在结果 `Intent` 中返回缩小尺寸的照片副本（缩略图），这个副本以 `Bitmap` 形式保存在名为 `"data"` 的 extra 字段中。

```java
static final int REQUEST_IMAGE_CAPTURE = 1;
static final Uri locationForPhotos;

public void capturePhoto(String targetFilename) {
    Intent intent = new Intent(MediaStore.ACTION_IMAGE_CAPTURE);
    intent.putExtra(MediaStore.EXTRA_OUTPUT,
            Uri.withAppendedPath(locationForPhotos, targetFilename));
    if (intent.resolveActivity(getPackageManager()) != null) {
        startActivityForResult(intent, REQUEST_IMAGE_CAPTURE);
    }
} 
@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
    if (requestCode == REQUEST_IMAGE_CAPTURE && resultCode == RESULT_OK) {
        Bitmap thumbnail = data.getParcelableExtra("data");
        // Do other work with full size photo saved in locationForPhotos
        ...
    }
}
```

### 5、界面

#### 5.1 使用 <merge> 标记

将此布局添加到其他布局中（使用 `<include/>` 标记）时，系统会忽略 `<merge>` 元素并直接在布局中放置两个按钮，以代替 `<include/>` 标记。

```xml
<merge xmlns:android="http://schemas.android.com/apk/res/android"> 
        <Button
            android:layout_width="fill_parent"
            android:layout_height="wrap_content"
            android:text="@string/add"/> 
        <Button
            android:layout_width="fill_parent"
            android:layout_height="wrap_content"
            android:text="@string/delete"/> 
</merge>
```

#### 5.2 [完全自定义的组件](https://developer.android.com/guide/topics/ui/custom-components)

| 类别                                      | 方法                                         | 说明                                                         |
| :---------------------------------------- | :------------------------------------------- | :----------------------------------------------------------- |
| 创建                                      | 构造函数                                     | 包含在从代码创建视图时调用的构造函数形式和在从布局文件扩充视图时调用的构造函数形式。第二种形式的构造函数应解析并应用布局文件中定义的任何属性。 |
| `onFinishInflate()```                     | 在视图及其所有子级都已从 XML 扩充之后调用。  |                                                              |
| 布局                                      | `onMeasure(int, int)```                      | 调用以确定此视图及其所有子级的大小要求。                     |
| `onLayout(boolean, int, int, int, int)``` | 在此视图应为其所有子级分配大小和位置时调用。 |                                                              |
| `onSizeChanged(int, int, int, int)```     | 在此视图的大小发生变化时调用。               |                                                              |
| 绘制                                      | `onDraw(Canvas)```                           | 在视图应渲染其内容时调用。                                   |
| 事件处理                                  | `onKeyDown(int, KeyEvent)```                 | 在发生新的按键事件时调用。                                   |
| `onKeyUp(int, KeyEvent)```                | 在发生松开按键事件时调用。                   |                                                              |
| `onTrackballEvent(MotionEvent)```         | 在发生轨迹球动作事件时调用。                 |                                                              |
| `onTouchEvent(MotionEvent)```             | 在发生触屏动作事件时调用。                   |                                                              |
| 焦点                                      | `onFocusChanged(boolean, int, Rect)```       | 在视图获得或失去焦点时调用。                                 |
| `onWindowFocusChanged(boolean)```         | 在包含视图的窗口获得或失去焦点时调用。       |                                                              |
| 附加                                      | `onAttachedToWindow()```                     | 在视图附加到窗口时调用。                                     |
| `onDetachedFromWindow()```                | 在视图与其窗口分离时调用。                   |                                                              |
| `onWindowVisibilityChanged(int)```        | 在包含视图的窗口的可见性发生变化时调用。     |                                                              |

**命名空间**：

不属于 `http://schemas.android.com/apk/res/android` 命名空间，而是属于 `http://schemas.android.com/apk/res/[your package name]`

如果视图类是**内部类**，您必须使用视图外部类的名称进一步限定。例如，`PieChart` 类有一个名为 `PieView` 的内部类。如需使用此类中的自定义属性，您需要使用 `com.example.customviews.charting.PieChart$PieView` 标记。

```xml
 <?xml version="1.0" encoding="utf-8"?>
    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"   xmlns:custom="http://schemas.android.com/apk/res/com.example.customviews">
     <com.example.customviews.charting.PieChart
         custom:showText="true"
         custom:labelPosition="left" />
    </LinearLayout>
```



- 需要绘制什么，由 `Canvas` 处理
- 如何绘制，由 `Paint` 处理。

```java
 private void init() {
       textPaint = new Paint(Paint.ANTI_ALIAS_FLAG);
       textPaint.setColor(textColor);
       if (textHeight == 0) {
           textHeight = textPaint.getTextSize();
       } else {
           textPaint.setTextSize(textHeight);
       }

       piePaint = new Paint(Paint.ANTI_ALIAS_FLAG);
       piePaint.setStyle(Paint.Style.FILL);
       piePaint.setTextSize(textHeight);

       shadowPaint = new Paint(0);
       shadowPaint.setColor(0xff101010);
       shadowPaint.setMaskFilter(new BlurMaskFilter(8, BlurMaskFilter.Blur.NORMAL));
```

- 手势识别

  ```java
   class MyListener extends GestureDetector.SimpleOnGestureListener {
         @Override
         public boolean onDown(MotionEvent e) {
             return true;
         }
      }
      detector = new GestureDetector(PieChart.this.getContext(), new MyListener());
  
  
      //具体处理
      @Override
      public boolean onTouchEvent(MotionEvent event) {
         boolean result = detector.onTouchEvent(event);
         if (!result) {
             if (event.getAction() == MotionEvent.ACTION_UP) {
                 stopScrolling();
                 result = true;
             }
         }
         return result;
      }
  ```



  - 视图高度

    要设置某个视图的默认（静止）高度，请在 XML 布局中使用 `android:elevation` 属性。如果要在 Activity 的代码中设置视图高度，请使用 `View.setElevation()` 方法。

    如果要设置视图转换，请使用 `View.setTranslationZ()` 方法。

```xml
<TextView
        android:id="@+id/myview"
        ...
        android:elevation="2dp"
        android:background="@drawable/myrect" />


<!-- res/drawable/myrect.xml -->
    <shape xmlns:android="http://schemas.android.com/apk/res/android"
           android:shape="rectangle">
        <solid android:color="#42000000" />
        <corners android:radius="5dp" />
    </shape>
```

#### 5.3 通知

从 Android 5.0 开始，通知可以显示在锁定屏幕上。

通知的设计由系统模板决定，您的应用只需要定义模板中各个部分的内容即可。通知的部分详情仅在展开后视图中显示。

![img](https://developer.android.com/images/ui/notifications/notification-callouts_2x.png)

**图 7.** 包含基本详情的通知



图 7 展示了通知最常见的部分，具体如下所示：

1. 小图标：必须提供，通过 `setSmallIcon()` 进行设置。

2. 应用名称：由系统提供。

3. 时间戳：由系统提供，但您可以通过 `setWhen()` 将其替换掉或者通过 `setShowWhen(false)` 将其隐藏。

4. 大图标：可选内容（通常仅用于联系人照片，请勿将其用于应用图标），通过 `setLargeIcon()` 进行设置。

5. 标题：可选内容，通过 `setContentTitle()` 进行设置。

6. 文本：可选内容，通过 `setContentText()` 进行设置。

   

**注意**：如果同一应用发出 4 条或更多条通知且未指定分组，则系统会自动将这些通知分为一组。

```java
        // Create an explicit intent for an Activity in your app
    Intent intent = new Intent(this, AlertDetails.class);
    intent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK | Intent.FLAG_ACTIVITY_CLEAR_TASK);
    PendingIntent pendingIntent = PendingIntent.getActivity(this, 0, intent, 0);

    NotificationCompat.Builder builder = new NotificationCompat.Builder(this, CHANNEL_ID)
            .setSmallIcon(R.drawable.notification_icon)
            .setContentTitle("My notification")
            .setContentText("Hello World!")
            .setPriority(NotificationCompat.PRIORITY_DEFAULT)
            // Set the intent that will fire when the user taps the notification
            .setContentIntent(pendingIntent)
        	.addAction(R.drawable.ic_snooze, getString(R.string.snooze),
                    snoozePendingIntent)
            .setAutoCancel(true);


	//显示通知
    NotificationManagerCompat notificationManager = NotificationManagerCompat.from(this);

    // notificationId is a unique int for each notification that you must define
    notificationManager.notify(notificationId, builder.build());
```

- 隐藏状态栏

```java
    View decorView = getWindow().getDecorView();
    // Hide the status bar.
    int uiOptions = View.SYSTEM_UI_FLAG_FULLSCREEN;
    decorView.setSystemUiVisibility(uiOptions);
    // Remember that you should never show the action bar if the
    // status bar is hidden, so hide that too if necessary.
    ActionBar actionBar = getActionBar();
    actionBar.hide();
    

   // Hide the status bar.
    window.decorView.systemUiVisibility = View.SYSTEM_UI_FLAG_FULLSCREEN
    // Remember that you should never show the action bar if the
    // status bar is hidden, so hide that too if necessary.
    actionBar?.hide()

```



- 隐藏导航栏

  ```java
  View decorView = getWindow().getDecorView();
      // Hide both the navigation bar and the status bar.
      // SYSTEM_UI_FLAG_FULLSCREEN is only available on Android 4.1 and higher, but as
      // a general rule, you should design your app to hide the status bar whenever you
      // hide the navigation bar.
      int uiOptions = View.SYSTEM_UI_FLAG_HIDE_NAVIGATION
                    | View.SYSTEM_UI_FLAG_FULLSCREEN;
      decorView.setSystemUiVisibility(uiOptions);
      
  ```

#### 5.4 拖放

用户使用拖拽手势（通常是长按视图对象）开始拖拽。

```java
    // Create a string for the ImageView label
    private static final String IMAGEVIEW_TAG = "icon bitmap"

    // Creates a new ImageView
    ImageView imageView = new ImageView(this);

    // Sets the bitmap for the ImageView from an icon bit map (defined elsewhere)
    imageView.setImageBitmap(iconBitmap);

    // Sets the tag
    imageView.setTag(IMAGEVIEW_TAG);

        ...

    // Sets a long click listener for the ImageView using an anonymous listener object that
    // implements the OnLongClickListener interface
    imageView.setOnLongClickListener(new View.OnLongClickListener() {

        // Defines the one method for the interface, which is called when the View is long-clicked
        public boolean onLongClick(View v) {

        // Create a new ClipData.
        // This is done in two steps to provide clarity. The convenience method
        // ClipData.newPlainText() can create a plain text ClipData in one step.

        // Create a new ClipData.Item from the ImageView object's tag
        ClipData.Item item = new ClipData.Item(v.getTag());

        // Create a new ClipData using the tag as a label, the plain text MIME type, and
        // the already-created item. This will create a new ClipDescription object within the
        // ClipData, and set its MIME type entry to "text/plain"
        ClipData dragData = new ClipData(
            v.getTag(),
            new String[] { ClipDescription.MIMETYPE_TEXT_PLAIN },
            item);

        // Instantiates the drag shadow builder.
        View.DragShadowBuilder myShadow = new MyDragShadowBuilder(imageView);

        // Starts the drag

                v.startDrag(dragData,  // the data to be dragged
                            myShadow,  // the drag shadow builder
                            null,      // no need to use local data
                            0          // flags (not currently used, set to 0)
                );

        }
    }
    
```

#### 5.5画中画

默认情况下，系统不会自动为应用提供画中画支持。要想在应用中支持画中画，您可以通过将 `android:supportsPictureInPicture` 和 `android:resizeableActivity` 设置为 `true`

```xml
    <activity android:name="VideoActivity"
        android:resizeableActivity="true"
        android:supportsPictureInPicture="true"
        android:configChanges=
            "screenSize|smallestScreenSize|screenLayout|orientation"
        ...
    
```

要进入画中画模式，Activity 必须调用 `enterPictureInPictureMode()`。



### 6、动画********

| 类/接口                            | 说明                                                         |
| :--------------------------------- | :----------------------------------------------------------- |
| `AccelerateDecelerateInterpolator` | 该插值器的变化率在开始和结束时缓慢但在中间会加快。           |
| `AccelerateInterpolator`           | 该插值器的变化率在开始时较为缓慢，然后会加快。               |
| `AnticipateInterpolator`           | 该插值器先反向变化，然后再急速正向变化。                     |
| `AnticipateOvershootInterpolator`  | 该插值器先反向变化，再急速正向变化，然后超过定位值，最后返回到最终值。 |
| `BounceInterpolator`               | 该插值器的变化会跳过结尾处。                                 |
| `CycleInterpolator`                | 该插值器的动画会在指定数量的周期内重复。                     |
| `DecelerateInterpolator`           | 该插值器的变化率开始很快，然后减速。                         |
| `LinearInterpolator`               | 该插值器的变化率恒定不变。                                   |
| `OvershootInterpolator`            | 该插值器会急速正向变化，再超出最终值，然后返回。             |
| `TimeInterpolator`                 | 该接口用于实现您自己的插值器。                               |

| 类               | 说明                                                         |
| :--------------- | :----------------------------------------------------------- |
| `ValueAnimator`  | 属性动画的主计时引擎，它也可计算要添加动画效果的属性的值。它具有计算动画值所需的所有核心功能，同时包含每个动画的计时详情、有关动画是否重复播放的信息、用于接收更新事件的监听器以及设置待评估自定义类型的功能。为属性添加动画效果分为两个步骤：计算添加动画效果之后的值，以及对要添加动画效果的对象和属性设置这些值。`ValueAnimator` 不会执行第二个步骤，因此，您必须监听由 `ValueAnimator` 计算的值的更新情况，并使用您自己的逻辑修改要添加动画效果的对象。如需了解详情，请参阅[使用 ValueAnimator 添加动画效果](https://developer.android.com/guide/topics/graphics/prop-animation#value-animator)部分。 |
| `ObjectAnimator` | `ValueAnimator` 的子类，用于设置目标对象和对象属性以添加动画效果。此类会在计算出动画的新值后相应地更新属性。在大多数情况下，您不妨使用 `ObjectAnimator`，因为它可以极大地简化对目标对象的值添加动画效果这一过程。不过，有时您需要直接使用 `ValueAnimator`，因为 `ObjectAnimator` 存在其他一些限制，例如要求目标对象具有特定的访问器方法。 |
| `AnimatorSet`    | 此类提供一种将动画分组在一起的机制，以使它们彼此相对运行。您可以将动画设置为一起播放、按顺序播放或者在指定的延迟时间后播放。如需了解详情，请参阅[使用 AnimatorSet 编排多个动画](https://developer.android.com/guide/topics/graphics/prop-animation#choreography)部分。 |

```java
    ObjectAnimator animation = ObjectAnimator.ofFloat(textView, "translationX", 100f);
    animation.setDuration(1000);
    animation.start();
```

- AnimatorSet 编排多个动画

```java
	AnimatorSet bouncer = new AnimatorSet();
    bouncer.play(bounceAnim).before(squashAnim1);
    bouncer.play(squashAnim1).with(squashAnim2);
    bouncer.play(squashAnim1).with(stretchAnim1);
    bouncer.play(squashAnim1).with(stretchAnim2);
    bouncer.play(bounceBackAnim).after(stretchAnim2);
    ValueAnimator fadeAnim = ObjectAnimator.ofFloat(newBall, "alpha", 1f, 0f);
    fadeAnim.setDuration(250);
    AnimatorSet animatorSet = new AnimatorSet();
    animatorSet.play(bouncer).before(fadeAnim);
    animatorSet.start();
```

- 为卡片翻转添加动画

```java
getSupportFragmentManager()
                    .beginTransaction()

                    // Replace the default fragment animations with animator resources
                    // representing rotations when switching to the back of the card, as
                    // well as animator resources representing rotations when flipping
                    // back to the front (e.g. when the system Back button is pressed).
                    .setCustomAnimations(
                            R.animator.card_flip_right_in,
                            R.animator.card_flip_right_out,
                            R.animator.card_flip_left_in,
                            R.animator.card_flip_left_out)

                    // Replace any fragments currently in the container view with a
                    // fragment representing the next page (indicated by the
                    // just-incremented currentPage variable).
                    .replace(R.id.container, new CardBackFragment())

                    // Add this transaction to the back stack, allowing users to press
                    // Back to get to the front of the card.
                    .addToBackStack(null)

                    // Commit the transaction.
                    .commit();
```

- 自动为布局更新添加动画

  ```xml
      <LinearLayout android:id="@+id/container"
          android:animateLayoutChanges="true"
          ...
      />
      
  ```



- Activity 过渡 API 适用于 Android 5.0 (API 21) 及更高版本。



### 7、视频

#### 唤醒锁定

为了确保您的 Service 在这些情况下能继续运行，您必须使用“唤醒锁定”。唤醒锁定可以告诉系统：您的应用正在使用一些即使在手机处于闲置状态时也应该可用的功能。

**注意**：您应始终谨慎使用唤醒锁定，并只使其保留必要的时长，因为它们会显著缩短设备的电池续航时间。

```java
    mediaPlayer = new MediaPlayer();
    // ... other initialization here ...
    mediaPlayer.setWakeMode(getApplicationContext(), PowerManager.PARTIAL_WAKE_LOCK);
```

#### 7.1 mediaRecorder 音频

```java
    package com.android.audiorecordtest;

    import android.Manifest;
    import android.content.Context;
    import android.content.pm.PackageManager;
    import android.media.MediaPlayer;
    import android.media.MediaRecorder;
    import android.os.Bundle;
    import android.support.annotation.NonNull;
    import android.support.v4.app.ActivityCompat;
    import android.support.v7.app.AppCompatActivity;
    import android.util.Log;
    import android.view.View;
    import android.view.ViewGroup;
    import android.widget.Button;
    import android.widget.LinearLayout;

    import java.io.IOException;

    public class AudioRecordTest extends AppCompatActivity {

        private static final String LOG_TAG = "AudioRecordTest";
        private static final int REQUEST_RECORD_AUDIO_PERMISSION = 200;
        private static String fileName = null;

        private RecordButton recordButton = null;
        private MediaRecorder recorder = null;

        private PlayButton   playButton = null;
        private MediaPlayer   player = null;

        // Requesting permission to RECORD_AUDIO
        private boolean permissionToRecordAccepted = false;
        private String [] permissions = {Manifest.permission.RECORD_AUDIO};

        @Override
        public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
            super.onRequestPermissionsResult(requestCode, permissions, grantResults);
            switch (requestCode){
                case REQUEST_RECORD_AUDIO_PERMISSION:
                    permissionToRecordAccepted  = grantResults[0] == PackageManager.PERMISSION_GRANTED;
                    break;
            }
            if (!permissionToRecordAccepted ) finish();

        }

        private void onRecord(boolean start) {
            if (start) {
                startRecording();
            } else {
                stopRecording();
            }
        }

        private void onPlay(boolean start) {
            if (start) {
                startPlaying();
            } else {
                stopPlaying();
            }
        }

        private void startPlaying() {
            player = new MediaPlayer();
            try {
                player.setDataSource(fileName);
                player.prepare();
                player.start();
            } catch (IOException e) {
                Log.e(LOG_TAG, "prepare() failed");
            }
        }

        private void stopPlaying() {
            player.release();
            player = null;
        }

        private void startRecording() {
            recorder = new MediaRecorder();
            recorder.setAudioSource(MediaRecorder.AudioSource.MIC);
            recorder.setOutputFormat(MediaRecorder.OutputFormat.THREE_GPP);
            recorder.setOutputFile(fileName);
            recorder.setAudioEncoder(MediaRecorder.AudioEncoder.AMR_NB);

            try {
                recorder.prepare();
            } catch (IOException e) {
                Log.e(LOG_TAG, "prepare() failed");
            }

            recorder.start();
        }

        private void stopRecording() {
            recorder.stop();
            recorder.release();
            recorder = null;
        }

        class RecordButton extends Button {
            boolean mStartRecording = true;

            OnClickListener clicker = new OnClickListener() {
                public void onClick(View v) {
                    onRecord(mStartRecording);
                    if (mStartRecording) {
                        setText("Stop recording");
                    } else {
                        setText("Start recording");
                    }
                    mStartRecording = !mStartRecording;
                }
            };

            public RecordButton(Context ctx) {
                super(ctx);
                setText("Start recording");
                setOnClickListener(clicker);
            }
        }

        class PlayButton extends Button {
            boolean mStartPlaying = true;

            OnClickListener clicker = new OnClickListener() {
                public void onClick(View v) {
                    onPlay(mStartPlaying);
                    if (mStartPlaying) {
                        setText("Stop playing");
                    } else {
                        setText("Start playing");
                    }
                    mStartPlaying = !mStartPlaying;
                }
            };

            public PlayButton(Context ctx) {
                super(ctx);
                setText("Start playing");
                setOnClickListener(clicker);
            }
        }

        @Override
        public void onCreate(Bundle icicle) {
            super.onCreate(icicle);

            // Record to the external cache directory for visibility
            fileName = getExternalCacheDir().getAbsolutePath();
            fileName += "/audiorecordtest.3gp";

            ActivityCompat.requestPermissions(this, permissions, REQUEST_RECORD_AUDIO_PERMISSION);

            LinearLayout ll = new LinearLayout(this);
            recordButton = new RecordButton(this);
            ll.addView(recordButton,
                    new LinearLayout.LayoutParams(
                            ViewGroup.LayoutParams.WRAP_CONTENT,
                            ViewGroup.LayoutParams.WRAP_CONTENT,
                            0));
            playButton = new PlayButton(this);
            ll.addView(playButton,
                    new LinearLayout.LayoutParams(
                            ViewGroup.LayoutParams.WRAP_CONTENT,
                            ViewGroup.LayoutParams.WRAP_CONTENT,
                            0));
            setContentView(ll);
        }

        @Override
        public void onStop() {
            super.onStop();
            if (recorder != null) {
                recorder.release();
                recorder = null;
            }

            if (player != null) {
                player.release();
                player = null;
            }
        }
    } 
```

### 8、service

[Service](https://developer.android.com/reference/android/app/Service)

服务默认使用应用的主线程， 需要创建线程。

[IntentService](https://developer.android.com/reference/android/app/IntentService)

- 是 `Service` 的子类，创建默认的工作线程。

- 处理完所有启动请求后停止服务，因此您永远不必调用 `stopSelf()`。

- 如要完成客户端提供的工作，请实现 `onHandleIntent()`
- 除 `onHandleIntent()` 之外，您无需从中调用超类的唯一方法就是 `onBind()`。只有在服务允许绑定时，您才需实现该方法。

```java
public class HelloIntentService extends IntentService {

  /**
   * A constructor is required, and must call the super <code><a href="/reference/android/app/IntentService.html#IntentService(java.lang.String)">IntentService(String)</a></code>
   * constructor with a name for the worker thread.
   */
  public HelloIntentService() {
      super("HelloIntentService");
  }

  /**
   * The IntentService calls this method from the default worker thread with
   * the intent that started the service. When this method returns, IntentService
   * stops the service, as appropriate.
   */
  @Override
  protected void onHandleIntent(Intent intent) {
      // Normally we would do some work here, like download a file.
      // For our sample, we just sleep for 5 seconds.
      try {
          Thread.sleep(5000);
      } catch (InterruptedException e) {
          // Restore interrupt status.
          Thread.currentThread().interrupt();
      }
  }
}
```

与上述使用 `IntentService` 的示例完全相同。

```java
public class HelloService extends Service {
  private Looper serviceLooper;
  private ServiceHandler serviceHandler;

  // Handler that receives messages from the thread
  private final class ServiceHandler extends Handler {
      public ServiceHandler(Looper looper) {
          super(looper);
      }
      @Override
      public void handleMessage(Message msg) {
          // Normally we would do some work here, like download a file.
          // For our sample, we just sleep for 5 seconds.
          try {
              Thread.sleep(5000);
          } catch (InterruptedException e) {
              // Restore interrupt status.
              Thread.currentThread().interrupt();
          }
          // Stop the service using the startId, so that we don't stop
          // the service in the middle of handling another job
          stopSelf(msg.arg1);
      }
  }

  @Override
  public void onCreate() {
    // Start up the thread running the service. Note that we create a
    // separate thread because the service normally runs in the process's
    // main thread, which we don't want to block. We also make it
    // background priority so CPU-intensive work doesn't disrupt our UI.
    HandlerThread thread = new HandlerThread("ServiceStartArguments",
            Process.THREAD_PRIORITY_BACKGROUND);
    thread.start();

    // Get the HandlerThread's Looper and use it for our Handler
    serviceLooper = thread.getLooper();
    serviceHandler = new ServiceHandler(serviceLooper);
  }

  @Override
  public int onStartCommand(Intent intent, int flags, int startId) {
      Toast.makeText(this, "service starting", Toast.LENGTH_SHORT).show();

      // For each start request, send a message to start a job and deliver the
      // start ID so we know which request we're stopping when we finish the job
      Message msg = serviceHandler.obtainMessage();
      msg.arg1 = startId;
      serviceHandler.sendMessage(msg);

      // If we get killed, after returning from here, restart
      return START_STICKY;
  }

  @Override
  public IBinder onBind(Intent intent) {
      // We don't provide binding, so return null
      return null;
  }

  @Override
  public void onDestroy() {
    Toast.makeText(this, "service done", Toast.LENGTH_SHORT).show();
  }
}
```

```
START_NOT_STICKY
```

如果系统在 `onStartCommand()` 返回后终止服务，则除非有待传递的挂起 Intent，否则系统*不会*重建服务。这是最安全的选项，可以避免在不必要时以及应用能够轻松重启所有未完成的作业时运行服务。

```
START_STICKY
```

如果系统在 `onStartCommand()` 返回后终止服务，则其会重建服务并调用 `onStartCommand()`，但*不会*重新传递最后一个 Intent。相反，除非有挂起 Intent 要启动服务，否则系统会调用包含空 Intent 的 `onStartCommand()`。在此情况下，系统会传递这些 Intent。此常量适用于不执行命令、但无限期运行并等待作业的媒体播放器（或类似服务）。

```
START_REDELIVER_INTENT
```

如果系统在 `onStartCommand()` 返回后终止服务，则其会重建服务，并通过传递给服务的最后一个 Intent 调用 `onStartCommand()`。所有挂起 Intent 均依次传递。此常量适用于主动执行应立即恢复的作业（例如下载文件）的服务。



- 前台服务

  例如，应将通过服务播放音乐的音乐播放器设置为在前台运行，因为用户会明确意识到其操作。状态栏中的通知可能表示正在播放的歌曲，并且其允许用户通过启动 Activity 与音乐播放器进行交互。同样，如果应用允许用户追踪其运行，则需通过前台服务来追踪用户的位置。

```java
Intent notificationIntent = new Intent(this, ExampleActivity.class);
PendingIntent pendingIntent =
        PendingIntent.getActivity(this, 0, notificationIntent, 0);

Notification notification =
          new Notification.Builder(this, CHANNEL_DEFAULT_IMPORTANCE)
    .setContentTitle(getText(R.string.notification_title))
    .setContentText(getText(R.string.notification_message))
    .setSmallIcon(R.drawable.icon)
    .setContentIntent(pendingIntent)
    .setTicker(getText(R.string.ticker_text))
    .build();

startForeground(ONGOING_NOTIFICATION_ID, notification);
```



```java
public class ExampleService extends Service {
    int startMode;       // indicates how to behave if the service is killed
    IBinder binder;      // interface for clients that bind
    boolean allowRebind; // indicates whether onRebind should be used

    @Override
    public void onCreate() {
        // The service is being created
    }
    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        // The service is starting, due to a call to startService()
        return mStartMode;
    }
    @Override
    public IBinder onBind(Intent intent) {
        // A client is binding to the service with bindService()
        return mBinder;
    }
    @Override
    public boolean onUnbind(Intent intent) {
        // All clients have unbound with unbindService()
        return mAllowRebind;
    }
    @Override
    public void onRebind(Intent intent) {
        // A client is binding to the service with bindService(),
        // after onUnbind() has already been called
    }
    @Override
    public void onDestroy() {
        // The service is no longer used and is being destroyed
    }
}
```

**注意：**与 Activity 生命周期回调方法不同，您*不*需要调用这些回调方法的超类实现。



### 9、后台任务-线程池



创建多个线程：使用 [`ExecutorService`](https://developer.android.com/reference/java/util/concurrent/ExecutorService) 接口，创建线程的成本很高，务必将 [`ExecutorService`](https://developer.android.com/reference/java/util/concurrent/ExecutorService) 的实例保存在 `Application` 类或[依赖项注入容器](https://developer.android.com/training/dependency-injection/manual)中。

```java
public class MyApplication extends Application {
    ExecutorService executorService = Executors.newFixedThreadPool(4);
}
```

```java
public class LoginRepository {
    ...
    private final Executor executor;

    public LoginRepository(LoginResponseParser responseParser, Executor executor) {
        this.responseParser = responseParser;
        this.executor = executor;
    }
    ...
     public void makeLoginRequest(final String jsonBody) {
        executor.execute(new Runnable() {
            @Override
            public void run() {
                Result<LoginResponse> ignoredResponse = makeSynchronousLoginRequest(jsonBody);
            }
        });
    }

    public Result<LoginResponse> makeSynchronousLoginRequest(String jsonBody) {
        ... // HttpURLConnection logic
    }
}
```



- 使屏幕保持开启状态

```java
    public class MainActivity extends Activity {
      @Override
      protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        getWindow().addFlags(WindowManager.LayoutParams.FLAG_KEEP_SCREEN_ON);
      }
    }
```













### [adb命令](https://developer.android.com/studio/command-line/adb#shellcommands)

使用 `adb` 触发一个 Intent

```
adb shell am start -a <ACTION> -t <MIME_TYPE> -d <DATA> \
  -e <EXTRA_NAME> <EXTRA_VALUE> -n <ACTIVITY>
  
adb shell am start -a android.intent.action.DIAL \
  -d tel:555-5555 -n org.example.MyApp/.MyActivity
```

#### 录制视频

```
$ adb shell
shell@ $ screenrecord --verbose /sdcard/demo.mp4
(press Control + C to stop)
shell@ $ exit
$ adb pull /sdcard/demo.mp4
```

#### 截取屏幕截图

```
$ adb shell
shell@ $ screencap /sdcard/screen.png
shell@ $ exit
$ adb pull /sdcard/screen.png
```

```
adb shell am start -a android.intent.action.VIEW
```























### [jetpack汇总](https://carsonho.blog.csdn.net/article/details/104243841)



Android Jetpack的组件主要分为四大类：

- 基础 - `Foundation`

  > 提供了最基础的底层功能，如向后兼容性、测试、开发语言Kotlin支持等。包含的组件库

- 架构 - `Architecture`

  > 帮助开发者设计稳健、可测试且易维护的应用

  1. Data Binding(数据绑定)：属于支持库可使用声明式将布局中的界面组件绑定到应用中的数据源
  2. Lifecycles：管理 Activity 和 Fragment 生命周期
  3. LiveData：是一个可观察的数据持有者类。与常规observable不同，LiveData是有生命周期感知的。
  4. Navigation：处理应用内导航所需的一切
  5. Paging：一次加载 or 按需加载 & 显示小块数据
  6. Room：帮助开发者更友好、流畅的访问SQLite数据库。
  7. ViewModel：以生命周期感知的方式存储和管理与UI相关的数据。
  8. WorkManager：调度预期将要运行的可延迟异步任务。（即便应用程序退出 or重启）

- 行为 - `Behavior`

  > 帮助应用与标准的 Android 服务（如通知、权限、分享和 Google 助理）相集成。包含组件库：

  1. 相机 - CameraX：简化相机应用的开发工作，可向后兼容至 Android 5.0（API 级别 21）
  2. 下载 - DownloadManager：可处理长时间运行的HTTP下载 & 超时重连
  3. 多媒体 - Media & playback：用于媒体播放 & 路由的向后兼容 API。
  4. 通知 - Notifications：提供向后兼容的通知 API，支持 Wear 和 Auto。
  5. 权限 - Permissions：用于检查和请求应用权限的兼容性 API。
  6. 偏好设置 - Preferences：提供了能够改变应用的功能和行为能力。
  7. 共享 - Sharing：提供适合应用操作栏的共享操作。
  8. 切片 - Slices：创建可在应用外部显示应用数据的灵活界面元素。

- 界面 - `UI`

  > 辅助绘制界面的View类 & 各种辅助组件，包括：

  1. 动画 - Animation & Transitions：提供各类内置动画，也可以自定义动画效果。
  2. 表情 - Emoji：使用户在未更新系统版本的情况下也可以使用表情符号。
  3. 布局 - Layout：xml书写的界面布局或者使用Compose完成的界面。
  4. 调试板 - Palette：从调色板中提取出有用的信息。

