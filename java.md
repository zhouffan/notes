### 1、Java8的两个重大改变，Lambda，Stream API

Stream有如下三个操作步骤：

#### **一、创建Stream**

从一个数据源，如集合、数组中获取流。

eg：personList.stream()

​		Arrays.stream(words)

#### **二、中间操作**

一个操作的中间链，对数据源的数据进行操作。

- filter：接收Lambda，从流中排除某些操作；

- limit：截断流，使其元素不超过给定对象

  ```java
  //从Person列表中取出两个女性。
  List<Person> collect1 = personList.stream().filter(person -> person.getSex() == 'F').limit(2).collect(Collectors.toList());
  ```

- skip(n)：跳过元素，返回一个扔掉了前n个元素的流，若流中元素不足n个，则返回一个空流，与limit(n)互补

  ```java
  //从Person列表中从第2个女性开始，取出所有的女性。
  List<Person> collect2 = personList.stream().filter(person -> person.getSex() == 'F').skip(2).collect(Collectors.toList());
  ```

- distinct：筛选，通过流所生成元素的hashCode()和equals()去除重复元素。

  ```java
  //重写equal和hashcode
  @Override
  public boolean equals(Object o) {
      if (this == o) return true;
      if (o == null || getClass() != o.getClass()) return false;
      Person person = (Person) o;
      return name.equals(person.name);
  }
  @Override
  public int hashCode() {
      return Objects.hash(name);
  }
  
  
  
  //男性中有两个李康，去除掉了一个重复的。
  List<Person> collect = personList.stream().filter(person -> person.getSex() == 'M').distinct().collect(Collectors.toList());
  System.out.println("collect = " + collect);
  ```

  

- map--接收Lambda，将元素转换成其他形式或提取信息。接收一个函数作为参数，该函数会被应用到每个元素上，并将其映射成一个新的元素。

  ```java
  //我们用一个PersonCountry类来接收所有的国家信息：
  personList.stream().map(person -> {
      String country = person.getCountry();
      return new PersonCountry(country);
  }).forEach(System.out::println);
  ```

- flatMap--接收一个函数作为参数，将流中的每个值都换成另一个流，然后把所有流连接成一个流

  ```java
  // [H, e, l, l, o, W, o, r, l, d]
  String[] words = new String[]{"Hello","World"};  
  List<String> collect6 = Arrays.stream(words).flatMap(s -> Stream.of(s.split(""))).collect(Collectors.toList())
  ```

  

- sorted()--自然排序(Comparable)

- sorted(Comparator com)--定制排序（Comparator）

  ```java
  //sort
  List<Person> collect7 = personList.stream().sorted((s1, s2) -> s1.getName().compareTo(s2.getName())).collect(Collectors.toList());
  ```

  注意：**flatmap和map的区别**：一对一，  一对多。map返回一个对象，flatmap返回一个stream。



#### **三、终止操作**

一个终止操作，执行中间操作链，并产生结果。

- allMatch--检查是否匹配所有元素

  ```java
  //判断personList中的人是否都是成年人
  boolean b = personList.stream().allMatch(person -> person.getAge() >= 18);
  ```

- anyMatch--检查是否至少匹配一个元素

- noneMatch--检查是否没有匹配所有元素

- findFirst--返回第一个元素

- findAny--返回当前流中的任意元素

- count--返回流中元素的总个数

- max--返回流中最大值

- min--返回流中最小值

  ```java
  //年龄最大的人信息
  Optional<Person> min = personList.stream().min((p1, p2) -> p1.getAge().compareTo(p2.getAge()));
  ```



### 2、 Optional





### 3、contentType

Internet Media Type即互联网媒体类型，也叫做MIME类型，使用两部分标识符来确定一个类型。

**Content-Type中常见的媒体格式类型**

- 以text开头的媒体格式类型：

  text/html： HTML格式。

  text/plain：纯文本格式。

  text/xml： XML格式。

- 以image开头的媒体格式类型：

  image/gif：gif图片格式。

  image/jpeg：jpg图片格式。

  image/png：png图片格式。

- 以application开头的媒体格式类型：

  application/xhtml+xml：XHTML格式。

  application/xml： XML数据格式。

  application/atom+xml：Atom XML聚合格式 。

  application/json： JSON数据格式。

  application/pdf：pdf格式 。

  application/msword： Word文档格式。

  application/octet-stream： 二进制流数据（如常见的文件下载）。

  application/x-www-form-urlencoded： <form encType="">中默认的encType，form表单数据被编码为key/value格式发送到服务器（表单默认的提交数据的格式）。

- 上传文件之时使用的：

  multipart/form-data ： 需要在表单中进行文件上传时，就需要使用该格式。

  

![img](https://images2018.cnblogs.com/blog/602164/201809/602164-20180907153055007-849088541.png)

@pathVariable     get====>all

@requestParam   post/put/delete/pathch===>form-data/x-www-form-urlencoded

​								get===> all

@requestBody      post/put/delete/pathch===>json

```java
@RequestMapping(value ="/md5" ,produces = "application/json;charset=UTF-8")
	public String md5(@RequestBody ParamsBean bean) { 
 	System.out.println(bean.getAppKey());
 	return "md5";
}
```

### 4、线程池

[Java线程池的内部原理](https://blog.csdn.net/heihaozi/article/details/102882698?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-2.nonecase&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-2.nonecase)

![img](https://img-blog.csdnimg.cn/20191103152757811.jpg#pic_center)





#### 4.1 线程-死锁

```java
import java.util.Date;
 
public class LockTest {
   public static String obj1 = "obj1";
   public static String obj2 = "obj2";
   public static void main(String[] args) {
      LockA la = new LockA();
      new Thread(la).start();
      LockB lb = new LockB();
      new Thread(lb).start();
   }
}
class LockA implements Runnable{
   public void run() {
      try {
         System.out.println(new Date().toString() + " LockA 开始执行");
         while(true){
            synchronized (LockTest.obj1) {
               System.out.println(new Date().toString() + " LockA 锁住 obj1");
               Thread.sleep(3000); // 此处等待是给B能锁住机会
               synchronized (LockTest.obj2) {
                  System.out.println(new Date().toString() + " LockA 锁住 obj2");
                  Thread.sleep(60 * 1000); // 为测试，占用了就不放
               }
            }
         }
      } catch (Exception e) {
         e.printStackTrace();
      }
   }
}
class LockB implements Runnable{
   public void run() {
      try {
         System.out.println(new Date().toString() + " LockB 开始执行");
         while(true){
            synchronized (LockTest.obj2) {
               System.out.println(new Date().toString() + " LockB 锁住 obj2");
               Thread.sleep(3000); // 此处等待是给A能锁住机会
               synchronized (LockTest.obj1) {
                  System.out.println(new Date().toString() + " LockB 锁住 obj1");
                  Thread.sleep(60 * 1000); // 为测试，占用了就不放
               }
            }
         }
      } catch (Exception e) {
         e.printStackTrace();
      }
   }
}

//结果======>  都不能进入到第二个 synchronized中去，因为第二个锁对象 被两个线程的第一个锁对象给持有了，没有释放锁
Tue May 05 10:51:06 CST 2015 LockB 开始执行
Tue May 05 10:51:06 CST 2015 LockA 开始执行
Tue May 05 10:51:06 CST 2015 LockB 锁住 obj2
Tue May 05 10:51:06 CST 2015 LockA 锁住 obj1
```

https://github.com/gdutxiaoxu/Android_interview/blob/master/Android%E5%9F%BA%E7%A1%80/Android%E9%9D%A2%E8%AF%95%E5%BF%85%E5%A4%87-%E7%BA%BF%E7%A8%8B.md





**9、 volatile关键字在Java中有什么作用？** 当我们使用volatile关键字去修饰变量的时候，所以线程都会直接读取该变量并且不缓存它。这就确保了**线程读取到的变量是同内存中是一致**的。

**10、同步方法和同步块，哪个是更好的选择？**

同步块是更好的选择，因为它不会锁住整个对象（当然你也可以让它锁住整个对象）。**同步方法会锁住整个对象**，哪怕这个类中有多个不相关联的同步块，这通常会导致他们停止执行并需要等待获得这个对象上的锁。





### 5、自定义注解

[Java实现自定义注解](https://blog.csdn.net/zt15732625878/article/details/100061528)

[Java 自定义注解及使用场景](https://www.jianshu.com/p/a7bedc771204)

```java
@Target({ElementType.FIELD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Info {
      String value() default "tracy";
      boolean isDelete();
}

@Data
@Builder
// 为Person类配置了刚刚定义的注解@Info
@Info(isDelete = true)
public class Person { 
    /**
     * 姓名
     */
    private String name; 
}

public class AnnotationTest {
    public static void main(String[] args) {
        try {
            //获取Person的Class对象
            Person person = Person.builder().build();
            Class clazz = person.getClass();
            //判断person对象上是否有Info注解
            if (clazz.isAnnotationPresent(Info.class)) {
                System.out.println("Person类上配置了Info注解！");
                //获取该对象上Info类型的注解
                Info infoAnno = (Info) clazz.getAnnotation(Info.class);
                System.out.println("person.name :" + infoAnno.value() + ",person.isDelete:" + infoAnno.isDelete());
            } else {
                System.out.println("Person类上没有配置Info注解！");
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

**反射是否真的会让你的程序性能降低?**

1.反射大概比直接调用慢50~100倍，但是需要你在执行100万遍的时候才会有所感觉
2.判断一个函数的性能，你需要把这个函数执行100万遍甚至1000万遍
3.如果你只是偶尔调用一下反射，请忘记反射带来的性能影响
4.如果你需要大量调用反射，请考虑缓存。
5.你的编程的思想才是限制你程序性能的最主要的因素





### 6、链式写法

```java
public class MyBuilder {
    private String username;
    private String password;
    public static MyBuilder builder(){
        return new MyBuilder();
    }
    public String getUsername() {
        return username;
    }
    public MyBuilder setUsername(String username) {
        this.username = username;
        return this;
    }
    public String getPassword() {
        return password;
    }
    public MyBuilder setPassword(String password) {
        this.password = password;
        return this;
    }

    public static void main(String[] args) {
        MyBuilder myBuilder = MyBuilder.builder()
                    .setPassword("1")
                    .setPassword("1");
    }
}
// 自带get、set 不改变
public class MyBuilder2 {
    private String username;
    private String password;
    //自带get、set 不改变
    public String getUsername() {
        return username;
    }
    public void setUsername(String username) {
        this.username = username;
    }
    public String getPassword() {
        return password;
    }
    public void setPassword(String password) {
        this.password = password;
    }
    //与builder参数关联
    public MyBuilder2(Builder builder) {
        this.username = builder.username;
        this.password = builder.password;
    }
    //构建内部类
    public static Builder builder(){
        return new Builder();
    }
    //******扩展创建对象******
    public static class Builder{
    private String username;
    private String password;
    public String getUsername() {
        return username;
    }
    public Builder setUsername(String username) {
        this.username = username;
        return this;
    }
    public String getPassword() {
        return password;
    }
    public Builder setPassword(String password) {
        this.password = password;
        return this;
    }
    public MyBuilder2 build(){
        return new MyBuilder2(this);
    }
    }
    @Override
    public String toString() {
        return "MyBuilder2{" +
        "username='" + username + '\'' +
        ", password='" + password + '\'' +
        '}';
    }
    public static void main(String[] args) {
        // MyBuilder2 build1 = new MyBuilder2.Builder().setPassword("").setPassword("").build();
        MyBuilder2 build = MyBuilder2.builder() //构建内部类
        .setPassword("1") //设置内部类对象的值
        .setUsername("2")//设置内部类对象的值
        .build(); //创建外部实体类，并将内部类对象传给实体类
        System.out.print(build);
    }
}
```





### 7、IO流，NIO

https://blog.csdn.net/forezp/article/details/88414741



#### 7.1 同步异步、阻塞非阻塞



### 8、引用分类

**强引用**

> 强引用是使用最普遍的引用。一个对象具有强引用，则在`GC`发生时，该对象将不会回收。当**Jvm虚拟机**内存空间不足时，虚拟机会抛出`OutOfMemoryError`错误，不会回收具有强引用的对象来解决内存不足的问题。

**软引用**

> 当一个对象**只有**软引用，若虚拟机内存空间足够，垃圾回收器就不会回收该对象； 若内存空间不足，下次`GC`时这些只有软引用对象将被回收。若垃圾回收器没有回收它，该对象就可以被程序使用。软引用可用来实现内存敏感的高速缓存。
>
> 在创建软引用实例时，可以传入一个引用队列（`ReferenceQueue`）将该软引用与引用队列关联，这样当软引用所引用的对象被垃圾回收器回收前，**Jvm虚拟机**会把这个软引用加入到与之关联的引用队列中。

**弱引用**

> 弱引用与软引用的区别在于：只具有弱引用的对象拥有更短暂的生命周期。在`GC`发生时，若一个对象只有弱引用，不管虚拟机内存空间是否足够，都会回收它的内存。
>
> 在创建弱引用实例时，可以传入一个引用队列（`ReferenceQueue`）将该软引用与引用队列关联，这样当软引用所引用的对象被垃圾回收器回收前，**Jvm虚拟机**就会把这个软引用加入到与之关联的引用队列中。

**虚引用**

> 虚引用并不会决定对象的生命周期。如果一个对象仅持有虚引用，那么它就和没有任何引用一样，在任何时候都可能被垃圾回收器回收。
>
> 在创建虚引用实例时，可以传入一个引用队列（`ReferenceQueue`）将该软引用与引用队列关联，这样当软引用所引用的对象被垃圾回收器回收前，**Jvm虚拟机**就会把这个软引用加入到与之关联的引用队列中。

**软引用**、**弱引用**、**虚引用**的构造方法均可以传入一个`ReferenceQueue`与之关联。在引用所指的对象被回收后，引用（`reference`)本身将会被加入到`ReferenceQueue`之中，此时引用所引用的对象`reference.get()`已被回收 (**`reference`此时不为`null`，`reference.get()`此时为null**)。

所以，在一个**非强引用**所引用的对象回收时，如果引用`reference`没有被加入到被关联的`ReferenceQueue`中，则表示还有引用所引用的对象还没有被回收。如果判断一个对象的**非强引用**本该出现在`ReferenceQueue`中，实际上却没有出现，则表示该对象发送内存泄漏。













