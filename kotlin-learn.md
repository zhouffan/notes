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







