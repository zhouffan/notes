### 1. Kotlin 相关链接

[两万六千字带你Kotlin入门](https://www.jianshu.com/p/f03ac08e4080)

[kotlin 开发文档](https://www.kotlincn.net/docs/reference/server-overview.html)





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







