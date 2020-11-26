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

[属性动画](https://developer.android.com/guide/topics/resources/animation-resource#Property)

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



[视图动画](https://developer.android.com/guide/topics/resources/animation-resource#View)

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

