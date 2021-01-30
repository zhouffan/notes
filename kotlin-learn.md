### 1. Kotlin 相关链接

[两万六千字带你Kotlin入门](https://www.jianshu.com/p/f03ac08e4080)

[kotlin 开发文档](https://www.kotlincn.net/docs/reference/server-overview.html)



https://kaixue.io/author/rengwuxian/ 

```kotlin
//var 是 variable 的缩写， val 是 value 的缩写
--------------分割线----------------------------
//java的Object变成kotlin的Any
//kotlin中的object（小写）：创建一个类对象， 将class换成object
// 👇 class 替换成了 object
object A {
    val number: Int = 1
    fun method() {
        println("A.method()")
    }
} 
//调用
A.number/ A.method()
--------------分割线----------------------------
//匿名类， 将new 换成object
val listener = object: ViewPager.SimpleOnPageChangeListener() {
    override fun onPageSelected(position: Int) {
        // override
    }
}  
--------------分割线----------------------------
//companion 可以理解为伴随、伴生，表示修饰的对象和外部类绑定
class Sample { 
       👇
    companion object {
        val anotherString = "Another String"
    }
    companion object B { //companion 修饰时,B可以省略
        var c: Int = 0
    }
}
//调用
A.B.c
===> A.c // 👈 B 没了
--------------分割线----------------------------
//Kotlin 的常量必须声明在对象（包括伴生对象）或者「top-level 顶层」中，因为常量是静态的。
//Kotlin 新增了修饰常量的 const 关键字。
class Sample {
    companion object {
         👇                  // 👇
        const val CONST_NUMBER = 1
    }
} 
const val CONST_SECOND_NUMBER = 2
--------------分割线----------------------------
//toMutable*() 返回的是一个新建的集合，原有的集合还是不可变的，所以只能对函数返回的集合修改。
val strList = listOf("a", "b", "c")
            👇
strList.toMutableList()
val strSet = setOf("a", "b", "c")
            👇
strSet.toMutableSet()
val map = mapOf("key1" to 1, "key2" to 2, "key3" to 3, "key4" to 3)
         👇
map.toMutableMap()
--------------分割线----------------------------
//java   达到「钩子」的效果
public class User {
    String name;
    public String getName() {
        return this.name + " nb";
    }
    public void setName(String name) {
        this.name = "Cute " + name;
    }
}
//kotlin
class User {
    var name = "Mike" 
        get() {
            return field + " nb"
        } 
        set(value) {
            field = "Cute " + value
        }
}
// 类中的变量
class Kotlin {
  var name = "kaixue.io"
}
//等价于java代码
public final class Kotlin {
   @NotNull
   private String name = "kaixue.io"; 
   @NotNull
   public final String getName() {
      return this.name;
   } 
   public final void setName(@NotNull String name) {
      this.name = name;
   }
}
// val 定义的变量， get时依然可以修改
val name = "Mike"
get() {
   return field + " nb"
}

--------------分割线----------------------------
fun main() {
    var activity: Activity = NewActivity()
    if (activity is NewActivity) {
        // 👇的强转由于类型推断被省略了
        activity.action()
    }
}
fun main() {
    var activity: Activity = NewActivity()
    // 👇'(activity as? NewActivity)' 之后是一个可空类型的对象，所以，需要使用 '?.' 来调用
    (activity as? NewActivity)?.action()
}

--------------分割线----------------------------
//调用
class User constructor(var name: String) {
                                   // 👇  👇 直接调用主构造器
    constructor(name: String, id: Int) : this(name) {
    }
                                                // 👇 通过上一个次构造器，间接调用主构造器
    constructor(name: String, id: Int, age: Int) : this(name, id) {
    }
}
//主构造器被修饰为私有
class User private constructor(name: String) {
//           👆 主构造器被修饰为私有的，外部就无法调用该构造器
}
//
class User(var name: String) {
}
//====》 等价于：
class User(name: String) {
  var name: String = name
}

--------------分割线----------------------------函数嵌套
fun login(user: String, password: String, illegalStr: String) {
    // 验证 user 是否为空
    if (user.isEmpty()) {
        throw IllegalArgumentException(illegalStr)
    }
    // 验证 password 是否为空
    if (password.isEmpty()) {
        throw IllegalArgumentException(illegalStr)
    }
}
//====> 等价
fun login(user: String, password: String, illegalStr: String) {
    fun validate(value: String) {
        if (value.isEmpty()) {                   👇
            throw IllegalArgumentException(illegalStr)
        }
    }
    validate(user)
    validate(password)
}
// Kotlin 内置的 require 函数
fun login(user: String, password: String, illegalStr: String) {
    require(user.isNotEmpty()) { illegalStr }
    require(password.isNotEmpty()) { illegalStr }
}

--------------分割线----------------------------
when (x) { 
    1, 2 -> print("x == 1 or x == 2")
    in 1..10 -> print("x 在区间 1..10 中") 
    in listOf(1,2) -> print("x 在集合中")
    // not in
    !in 10..20 -> print("x 不在区间 10..20 中")
    is String -> true
    else -> print("else")
}
when { 
    str1.contains("a") -> print("字符串 str1 包含 a") 
    str2.length == 3 -> print("字符串 str2 的长度为 3")
}
--------------分割线----------------------------


--------------分割线----------------------------


--------------分割线----------------------------

--------------分割线----------------------------

--------------分割线----------------------------


--------------分割线----------------------------


--------------分割线----------------------------


--------------分割线----------------------------


--------------分割线----------------------------

--------------分割线----------------------------

```



### 2. kotlin常用函数

#### 2.1 let函数

it指代的是object本身，**一个 lambda函数块block 作为参数**，**T类型**：**it**，**返回值R表示block函数，在block方法里最后一行表达的就是返回值**

```kotlin
@kotlin.internal.InlineOnly
public inline fun <T, R> T.let(block: (T) -> R): R {
    contract {
        callsInPlace(block, InvocationKind.EXACTLY_ONCE)
    }
    return block(this)
}
```

```java
//java
if (!TextUtils.isEmpty(bacData.getColor())) {
	...省略代码
	mBac.setBackgroundColor(Color.parseColor(bacData.getColor()));
}

//kotlin， 结合?处理处理判空
bacData.color?.let{
	...省略代码
	mBac.setBackgroundColor(Color.parseColor(it))
}
```

```java
//java
tabLayouParams.setLayoutParams(params);
tabLayouParams.setDark2();
tabLayouParams.setTag(i);
tabLayouParams.setText(page.getTitle());
//kotlin，明确变量的使用范围，让代码简洁易读
tabLayouParams.let{
	it.layoutparams = params
	it.setDark2()
	it.tag = i
	it.text = page.title
}

//最后一行为返回值
object1 = object.let{
	it
}
var num = object.let{
	100
}
```

#### 2.2 also函数

also函数与let相似，不同的是，**let是以闭包的形式返回**，返回函数体内最后一行的值。而also函数返回的是this,就是传入对象T本身。

适用场景：跟let使用场景类似。

```kotlin
@kotlin.internal.InlineOnly
@SinceKotlin("1.1")
public inline fun <T> T.also(block: (T) -> Unit): T {
    contract {
        callsInPlace(block, InvocationKind.EXACTLY_ONCE)
    }
    block(this)
    return this
}
```

```kotlin
data class Data(val text:String,val num:Int)

....省略代码
var data = Data("zhangsan",18)
var data1: Data
//正常
data1 = data.also{
	println(it.text)
}

data1 = data.also{
	it
}
//提示报错
data1 = data.let{
	println(it.text)
}

data1 = data.let{
	it
}
```

#### 2.3 with函数

**参数一**：T类型的对象receiver；**参数二**：一个lambda函数块

**with跟let的相似的地方是函数块最后一行作为返回值**

适用场景：直接调用类的函数或者属性。

```kotlin
@kotlin.internal.InlineOnly
public inline fun <T, R> with(receiver: T, block: T.() -> R): R {
    contract {
        callsInPlace(block, InvocationKind.EXACTLY_ONCE)
    }
    return receiver.block()
}
```

```java
//java
mTvTitle.setText(data.getTitle());
mTvSku.setText("已选规格:" + data.getSpec());
mTvPrice.setText("¥" + data.getPrice());
...

//kotlin
with(data,{
	mTvTitle.text(title)
	mTvSku.text("已选规格$spec")
	mTvPrice.text("¥$price")
	...
})
//上面等效于下面
with(data){
	mTvTitle.text(title)
	mTvSku.text("已选规格$spec")
	mTvPrice.text("¥$price")
	...
}
```



#### 2.4 run函数

run函数是let跟with的结合体，接收一个lambda函数作为参数，以闭包形式返回，返回值为最后一行的值。

适用场景：适用场景跟let和with相似，即可像let一样判空处理，函数块里面可以跟with函数一样直接访问公共属性跟方法。

```kotlin
@kotlin.internal.InlineOnly
public inline fun <T, R> T.run(block: T.() -> R): R {
    contract {
        callsInPlace(block, InvocationKind.EXACTLY_ONCE)
    }
    return block()
}
```

```kotlin
with(data){
	mTvTitle.text(title)
	mTvSku.text("已选规格$spec")
	mTvPrice.text("¥$price")
	...
}
//即做了判空处理，也做到直接访问属性跟方法
data?.run{
	mTvTitle.text(title)
	mTvSku.text("已选规格$spec")
	mTvPrice.text("¥$price")
	...
}
```

#### 2.5 apply函数

返回的是this，返回T类型，就是我们使用的对象本身。

适用场景：我们在做数据业务处理的时候，会遇到多层级对象的判空处理。

```kotlin
@kotlin.internal.InlineOnly
public inline fun <T> T.apply(block: T.() -> Unit): T {
    contract {
        callsInPlace(block, InvocationKind.EXACTLY_ONCE)
    }
    block()
    return this
}
```

```kotlin
//java
if (result == null || result.getData()==null) {
	return;
}
Data data = result.getData();
if(data.getAddress() != null){
	...省略代码
}
...省略代码

// kotlin
result?.apply{
	//result 对象存在时的处理
}?.data?.apply{
	//data 对象存在时的处理
}?.address?.apply{	
	//addres 对象存在时的处理
}
```

#### 2.6 对比总结

**let**

// 作用1：使用it替代object对象去访问其公有的属性 & 方法 

// 作用2：判断object为null的操作

// 注：返回值 = 最后一行 / return的表达式

```
object?.let{//表示object不为null的条件下，才会去执行let函数体
   it.function1() 
   it.function2() 
}
```



------------------------------------------------------------

**also**

类似let函数，但区别在于返回值：

- let函数：返回值 = 最后一行 / return的表达式

- also函数：返回值 = 传入的对象的本身

--------------------------------------------------------------

**with**

省去对象名，直接调用方法名 / 属性即可 

```
// kotlin
val people = People("carson", 25)
with(people) {
    println("my name is $name, I am $age years old")
} 
```



---------------------------------------

**run**

`结合了let、with两个函数的作用`

省去对象名，直接调用方法名 / 属性即可

统一做判空处理 

```
val people = People("carson", 25)
people?.run{
    println("my name is $name, I am $age years old")
}
```

-------------------

**apply**

与run类似，区别在于返回值

- run函数返回最后一行的值 / 表达式
- apply函数返回传入的对象的本身 

![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy85NDQzNjUtY2MzMTNkODU4MTFhNzhmMi5wbmc?x-oss-process=image/format,png)



### 3、语法糖（for-in/is/as/===/?./!!.）

#### 3.1范围使用

```kotlin
//作用：闭区间，[1,10]   step：步长
for (i in 1..10 step 2){
    print(i)
}
//作用：开区间[1,10)   step：步长
for (i in 1 until 10){
    print(i)
}
//作用：倒序判断
for (i in 10 downTo 1){
    print(i)
}
//在...范围内
var i = 20
if (i !in 1..15){
    println("xxxx")
}else{
    println("yyyyy")
}
```

#### 3.2 类型检查 & 转换

```kotlin
//is判断类型， 并且类型自动转换 
var a: Any = 1
if(a !is Int){
    println("输出1：$a")
}else{
    println("输出2："+(a + 1))
}
when(a){
    is String-> print("$a----b")
    is Int-> print(a+1)
}
//as 强转   ?避免抛出异常
var str = null
var l = str as? Int
print(l)
```

#### 3.3 相等性判断

```kotlin
var u1 = User("z1", 1)
var u2 = User("z1", 1)
var u3 = u1
//结构相等：equals 或者 ==
if(u1 == u3){//true
    println("true")
}else{
    println("false")
}
//引用相等：===
if(u2 === u3){//false
    println("true")
}else{
    println("false")
}
```

#### 3.4 空安全

```kotlin
//Kotlin中，有两种情况导致NullPointerException
// 情况1：显式调用 throw NullPointerException()
if(true){
    throw NullPointerException("主动抛出")
}
// 情况2：使用!! 操作符
var u: User? = null
var age = u!!.age
print(age)
//===================注：String类型变量不能容纳null，若要容纳null，添加?
var b: String? = "b"
b = null
//===================注：安全调用操作符   .前加?
var a = u?.age
//链式调用
var a2 = a?.b?.c?.d
```





### 4 泛型 in /out 

https://kaixue.io/kotlin-generics/

- `? extends` 叫做「上界通配符」，可以使 Java 泛型具有「协变性 Covariance」

- `? super` 叫做「下界通配符」，可以使 Java 泛型具有「逆变性 Contravariance」

- 可以使用泛型通配符 `? extends` 来使泛型支持协变，但是「只能读取不能修改」，这里的修改仅指对泛型集合添加元素，如果是 `remove(int index)` 以及 `clear` 当然是可以的。

  ```java
  List<? extends TextView> textViews = new ArrayList<Button>();
  TextView textView = textViews.get(0); // 👈 get 可以
  textViews.add(textView);
  //             👆 add 会报错，no suitable method found for add(TextView)
  ```

- 可以使用泛型通配符 `? super` 来使泛型支持逆变，但是「只能修改不能读取」，这里说的不能读取是指不能按照泛型类型读取，你如果按照 `Object` 读出来再强转当然也是可以的。

  ```javascript
  List<? super Button> buttons = new ArrayList<TextView>();
  Object object = buttons.get(0); // 👈 get 出来的是 Object 类型
  Button button = ...
  buttons.add(button); // 👈 add 操作是可以的
  ```

- 使用关键字 `out` 来支持协变，等同于 Java 中的上界通配符 `? extends`。

- 使用关键字 `in` 来支持逆变，等同于 Java 中的下界通配符 `? super`。

- **`out`** 表示，我这个变量或者参数**只用来输出**，不用来输入，你只能读我不能写我；**`in`** 就反过来，表示它**只用来输入**，不用来输出，你只能写我不能读我。

```java
🏝️             👇
class Producer<out T> {
    fun produce(): T {
        ...
    }
}

val producer: Producer<TextView> = Producer<Button>() // 👈 这里不写 out 也不会报错
val producer: Producer<out TextView> = Producer<Button>() // 👈 out 可以但没必要
---------------------------------------------------
🏝️            👇
class Consumer<in T> {
    fun consume(t: T) {
        ...
    }
}

val consumer: Consumer<Button> = Consumer<TextView>() // 👈 这里不写 in 也不会报错
val consumer: Consumer<in Button> = Consumer<TextView>() // 👈 in 可以但没必要
------------------------------------------------------- ?
//Java 中单个 ? 号也能作为泛型通配符使用，相当于 ? extends Object。
//它在 Kotlin 中有等效的写法：* 号，相当于 out Any。
------------------------------------------------------- where
//extends多个父
class Monster<T extends Animal & Food>{ 
}
//where 多个父
🏝️                👇
class Monster<T> where T : Animal, T : Food
------------------------------------------------------- reified
//Java 中的泛型存在类型擦除的情况，任何在运行时需要知道泛型确切类型信息的操作都没法用了
//java
<T> void test(Object item) {
    if (item instanceof T) { // 👈 IDE 会提示错误，illegal generic type for instanceof
        System.out.println(item);
    }
}
//=====>解决   额外传递一个 Class<T> 类型的参数
<T> void test(Object item, Class<T> type) {
    if (type.isInstance(item)) { 
        System.out.println(item);
    }
}
//kotlin
fun <T> test(item: Any) {
    if (item is T) { // 👈 IDE 会提示错误，Cannot check for instance of erased type: T
        println(item)
    }
}
//=====>解决    inline/reified泛型实化
inline fun <reified T> test(item: Any) {
    if (item is T) { // 👈 这里就不会在提示错误了
        println(item)
    }
}

```



### 5 协程

 https://blog.csdn.net/qq_39969226/article/details/101058033

https://kaixue.io/kotlin-coroutines-1/

1.协程是什么？
线程框架。【更方便】

协程就是launch里面的代码。

2.挂起谁？
挂起协程。

launch创建的协程在执行到某一个suspend函数挂起函数的时候，这个协程会被suspend（被挂起）

3.从哪儿挂起？
从当前线程挂起。

这个协程从正在执行它的线程上脱离了。不是这个协程停下来了而是协程所在的线程从这行代码开始不再运行这个协程了。

线程和协程分2波走了。



- 协程在执行到有supend标记的函数的时候会被挂起，**<u>挂起和开启一个协程一样</u>**，就是**<u>切</u>**个线程。只不过挂起函数在执行完毕之后协程会自动的<u>**重新切回它原先的那个线程**</u>。挂起就是一个稍后会被自动切回来的线程切换。



```kotlin
// 方法一，使用 runBlocking 顶层函数
// 方法一通常适用于单元测试的场景，而业务开发中不会用到这种方法，因为它是线程阻塞的。
runBlocking {
    getImage(imageId)
}

// 方法二，使用 GlobalScope 单例对象
// 方法二和使用 runBlocking 的区别在于不会阻塞线程。但在 Android 开发中同样不推荐这种用法，因为它的生命周期会和 app 一致，且不能取消
//            👇 可以直接调用 launch 开启协程
GlobalScope.launch {
    getImage(imageId)
}

// 方法三，自行通过 CoroutineContext 创建一个 CoroutineScope 对象
// 方法三是比较推荐的使用方法，我们可以通过 context 参数去管理和控制协程的生命周期（这里的 context 和 Android 里的不是一个东西，是一个更通用的概念，会有一个 Android 平台的封装来配合使用）。
//                                    👇 需要一个类型为 CoroutineContext 的参数
val coroutineScope = CoroutineScope(context)
coroutineScope.launch {
    getImage(imageId)
}
//---------------------分割线------------------------------------
// 同时请求多个地址
coroutineScope.launch(Dispatchers.Main) {
    //            👇  async 函数之后再讲
    val avatar = async { api.getAvatar(user) }    // 获取用户头像
    val logo = async { api.getCompanyLogo(user) } // 获取用户所在公司的 logo
    val merged = suspendingMerge(avatar, logo)    // 合并结果
    //                  👆
    show(merged) // 更新 UI
}
//---------------------分割线------------------------------------
// 协程切换-----withContext:自动把线程切回去继续执行
coroutineScope.launch(Dispatchers.Main) {      // 👈 在 UI 线程开始
    val image = withContext(Dispatchers.IO) {  // 👈 切换到 IO 线程，并在执行完成后切回 UI 线程
        getImage(imageId)                      // 👈 将会运行在 IO 线程
    }
    avatarIv.setImageBitmap(image)             // 👈 回到 UI 线程更新 UI
} 
//---------------------分割线------------------------------------
// 「用同步的方式写异步的代码」-----自动切换线程
launch(Dispatchers.Main) {              // 👈 在 UI 线程开始
    val image = getImage(imageId)
    avatarIv.setImageBitmap(image)     // 👈 执行结束后，自动切换回 UI 线程
}
//                               👇
suspend fun getImage(imageId: Int) = withContext(Dispatchers.IO) {
    ...
}

//---------------------分割线------------------------------------
// Deferred ：延迟； 调用 Deferred.await() 得到结果
coroutineScope.launch(Dispatchers.Main) {
    //                      👇  async 函数启动新的协程
    val avatar: Deferred = async { api.getAvatar(user) }    // 获取用户头像
    val logo: Deferred = async { api.getCompanyLogo(user) } // 获取用户所在公司的 logo
    //            👇          👇 获取返回值
    show(avatar.await(), logo.await())                     // 更新 UI
}

//suspend 函数
//要求 suspend 函数只能在协程里或者另一个 suspend 函数里被调用，是为了要让协程能够在 suspend 函数切换线程之后再切回来.
//=========================
//通过 withContext 源码可以知道，它本身就是一个挂起函数，它接收一个 Dispatcher 参数，依赖这个 Dispatcher 参数的指示，你的协程被挂起，然后切到别的线程。
//所以这个 suspend，其实并不是起到把任何把协程挂起，或者说切换线程的作用。
//真正挂起协程这件事，是 Kotlin 的协程框架帮我们做的。
//所以我们想要自己写一个挂起函数，仅仅只加上 suspend 关键字是不行的，还需要函数内部直接或间接地调用到 Kotlin 协程框架自带的 suspend 函数才行。

===>总结：切换线程需要调用系统的 挂起函数（如 withContext、 delay）
//---------------------分割线------------------------------------
//什么时候需要自定义 suspend 函数=======>如果你的某个函数比较耗时，也就是要等的操作，那就把它写成 suspend 函数。这就是原则。
//具体该怎么写===========>给函数加上 suspend 关键字，然后在 withContext 把函数的内容包住就可以了。（withContext：把线程自动切走和切回。）


协程就是切线程；
挂起就是可以自动切回来的切线程；
挂起的非阻塞式指的是它能用看起来阻塞的代码写出非阻塞的操作
```











### 6 空字符串判断

| 値       | isEmpty  | isNotEmpty | isNullOrEmpty | isBlank  | isNotBlank | isNullOrBlank | orEmpty  |
| :------- | :------- | :--------- | :------------ | :------- | :--------- | :------------ | :------- |
| var=""   | **true** | *false*    | **true**      | **true** | *false*    | **true**      | 空文字列 |
| var=" "  | *false*  | **true**   | *false*       | **true** | *false*    | **true**      | スペース |
| var=null | *Error*  | *Error*    | **true**      | *Error*  | *Error*    | **true**      | 空文字列 |
| var="1"  | *false*  | **true**   | *false*       | *false*  | **true**   | *false*       | 1        |
| var="0"  | *false*  | **true**   | *false*       | *false*  | **true**   | *false*       | 0        |





### 示例对比

1. int[] / string[]

```java
//java
String[] fromColumns = {ContactsContract.Data.DISPLAY_NAME,   ContactsContract.CommonDataKinds.Phone.NUMBER};
int[] toViews = {R.id.display_name, R.id.phone_number};

//kotlin
val fromColumns = arrayOf(ContactsContract.Data.DISPLAY_NAME,      ContactsContract.CommonDataKinds.Phone.NUMBER)
val toViews = intArrayOf(R.id.display_name, R.id.phone_number)
```







