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



### 2. Optional