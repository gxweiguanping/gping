---
title: lambda表达式语法糖
tags: java
categories: java
cover: https://gitee.com/studentgitee/note-picture/raw/master/b8014a90f603738d4a50b23f67037457f819ecb1.jpg
---
![image-20230210143018927](https://gitee.com/studentgitee/note-picture/raw/master/image-20230210143018927.png)

# Java 8 新特性Stream流

- Java 8 API添加了一个新的抽象称为流Stream，可以让你以一种声明的方式处理数据。

- Stream 使用一种类似用 SQL 语句从数据库查询数据的直观方式来提供一种对 Java 集合运算和表达的高阶抽象。

- Stream API可以极大提高Java程序员的生产力，让程序员写出高效率、干净、简洁的代码。

- 这种风格将要处理的元素集合看作一种流， 流在管道中传输， 并且可以在管道的节点上进行处理， 比如筛选， 排序，聚合等。

- 元素流在管道中经过中间操作（intermediate operation）的处理，最后由最终操作(terminal operation)得到前面处理的结果。

以上的流程转换为 Java 代码为：

```java
List<Integer> transactionsIds = widgets.stream()
.filter(b -> b.getColor() == RED)
.sorted((x,y) -> x.getWeight() - y.getWeight())
.mapToInt(Widget::getWeight).sum(); 
```

## 什么是 Stream？

Stream（流）是一个来自数据源的元素队列并支持聚合操作

- **元素是特定类型的对象，形成一个队列。 Java中的Stream并不会存储元素，而是按需计算。**

- **数据源** 流的来源。 可以是集合，数组，I/O channel， 产生器generator 等。

- **聚合操作** 类似SQL语句一样的操作， 比如filter, map, reduce, find, match, sorted等。

和以前的Collection操作不同， Stream操作还有两个基础的特征：

- **Pipelining** : 中间操作都会返回流对象本身。 这样多个操作可以串联成一个管道， 如同流式风格（fluent style）。 这样做可以对操作进行优化， 比如延迟执行(laziness)和短路( short-circuiting)。

- **内部迭代** ： 以前对集合遍历都是通过Iterator或者For-Each的方式, 显式的在集合外部进行迭代， 这叫做外部迭代。 Stream提供了内部迭代的方式， 通过访问者模式(Visitor)实现。

## 生成流

在 Java 8 中, 集合接口有两个方法来生成流：

- **stream()** − 为集合创建串行流。

- **parallelStream()** − 为集合创建并行流。

```java
List<String> strings = Arrays.asList("abc", "", "bc", "efg", "abcd","", "jkl");

List<String> filtered = strings.stream().
    filter(string -> !string.isEmpty()).collect(Collectors.toList());
```

## forEach

Stream 提供了新的方法 **forEach** 来迭代流中的每个数据。以下代码片段使用 forEach 输出了10个随机数：

```
Random random = new Random(); random.ints().limit(10).forEach(System.out::println); 
```

## map

map 方法用于映射每个元素到对应的结果， **接收一个函数作为参数，该函数会被应用到每个元素上，并将其映射成一个新的元素。** 以下代码片段使用 map 输出了元素对应的平方数：

```
List<Integer> numbers = Arrays.asList(3, 2, 2, 3, 7, 3, 5); // 获取对应的平方数 

List<Integer> squaresList = numbers.stream().map( i -> i*i).distinct().collect(Collectors.toList()); 

List<String> list = Arrays.asList("a,b,c", "1,2,3");//将每个元素转成一个新的且不带逗号的元素

Stream<String> s1 = list.stream().map(s -> s.replaceAll(",",""));

s1.forEach(System.out::println); // abc  123 
```

## filter

filter 方法用于通过设置的条件过滤出元素。以下代码片段使用 filter 方法过滤出空字符串：

```
List<String>strings = Arrays.asList("abc","","bc","efg","abcd","","jkl"); // 获取空字符串的数量 

long count = strings.stream().filter(string -> string.isEmpty()).count(); 
```

## limit

limit 方法用于获取指定数量的流。 以下代码片段使用 limit 方法打印出 10 条数据：

```
Random random = new Random();
random.ints().limit(10).forEach(System.out::println); 
```

## skip(n)：

跳过n元素，配合limit(n)可实现分页

## **distinct** ：

通过流中元素的 hashCode() 和 equals() 去除重复元素

## sorted

- **sorted 方法用于对流进行排序。**

- **sorted()：自然排序。**

- **sorted(Comparator com)：定制排序，自定义Comparator排序器 。**

```
// 自然排序
Random random = new Random();random.ints().limit(10).sorted().forEach(System.out::println); 

// 定制排序，String 类自身已实现Compareable接口
List<String> list = Arrays.asList("aa", "ff", "dd");
list.stream().sorted().forEach(System.out::println);// aa dd ff
```

###  按照某个实体的某个字段排序

```
# 按照Danswer字段进行升序
List<SklExamRecoedDTO> collect = list.stream()
.sorted((Comparator.comparing(SklExamRecoedDTO::getDanswer)))
.collect(Collectors.toList());

# 按照Danswer字段进行降序
List<SklExamRecoedDTO> collect = list.stream()
.sorted((Comparator.comparing(SklExamRecoedDTO::getDanswer).reversed()))
.collect(Collectors.toList());
```

### 自定义排序

```
Student s1 = new Student("aa", 10);
Student s2 = new Student("bb", 20);
Student s3 = new Student("aa", 30);
Student s4 = new Student("dd", 40);
List<Student> studentList = Arrays.asList(s1, s2, s3, s4);//自定义排序：先按姓名升序，姓名相同则按年龄升序
studentList.stream().sorted(
        (o1, o2) -> {
            if (o1.getName().equals(o2.getName())) {
                return o1.getAge() - o2.getAge();
            } else {
                return o1.getName().compareTo(o2.getName());
            }
        }
).forEach(System.out::println);// 当前类new一个Comparator，只在当前对象生效
List<Integer> list = Arrays.asList(7, 6, 9, 4, 11, 6);		// 自定义排序		
Optional<Integer> max2 = list.stream().max(new Comparator<Integer>() {			
              @Override			
              public int compare(Integer o1, Integer o2) {return o1.compareTo(o2);}		
              }
           );
System.out.println("自定义排序的最大值：" + max2.get());
```

### 并行（parallel）程序

parallelStream 是流并行处理程序的代替方法。**在高并发的场景下不要使用**

**并行流就是把一个内容分成多个数据块，并用不同的线程分成多个数据块，并用不同的线程分别处理每个数据块的流。**

以下实例我们使用 parallelStream 来输出空字符串的数量：

```
List<String> strings = Arrays.asList("abc", "", "bc", "efg", "abcd","", "jkl");// 获取空字符串的数量
long count = strings.parallelStream().filter(string -> string.isEmpty()).count(); 
```

我们可以很容易的在顺序运行和并行直接切换。

## Collectors

Collectors 类实现了很多归约操作，例如将流转换成集合和聚合元素。Collectors 可用于返回列表或字符串：

```
List<String>strings = Arrays.asList("abc", "", "bc", "efg", "abcd","", "jkl");
List<String> filtered = strings.stream().filter(string -> !string.isEmpty()).collect(Collectors.toList());

System.out.println("筛选列表: " + filtered);
String mergedString = strings.stream().filter(string -> !string.isEmpty()).collect(Collectors.joining(","));

System.out.println("合并字符串: " + mergedString); 

// collectors操作
Student s1 = new Student("aa", 10,1);
Student s2 = new Student("bb", 20,2);
Student s3 = new Student("cc", 10,3);
List<Student> list = Arrays.asList(s1, s2, s3); //装成list
List<Integer> ageList = list.stream().map(Student::getAge).collect(Collectors.toList()); // [10, 20, 10] 

//转成list
setSet<Integer> ageSet = list.stream().map(Student::getAge).collect(Collectors.toSet()); // [20, 10] //转成set

//转成map,注:key不能相同，否则报错
Map<String, Integer> studentMap = list
         .stream()
         .collect(Collectors.toMap(Student::getName, Student::getAge)); // {cc=10, bb=20, aa=10}
         
//字符串分隔符连接
String joinName = list.stream().map(Student::getName) .collect(Collectors.joining(",", "(", ")")); // (aa,bb,cc) //聚合操作

//1.学生总数Long count = list.stream().collect(Collectors.counting()); // 3

//2.最大年龄 (最小的minBy同理)Integer maxAge = list.stream()
                                  .map(Student::getAge)
                                  .collect(Collectors.maxBy(Integer::compare)).get(); // 20
                                  
//3.所有人的年龄Integer sumAge = list.stream().collect(Collectors.summingInt(Student::getAge)); // 40

//4.平均年龄Double averageAge = list.stream().collect(Collectors.averagingDouble(Student::getAge)); 
// 13.333333333333334 

```

### groupingBy分组

```
// 按照Student的age字段进行分组
Map<Integer, List<Student>> ageMap = list.stream().collect(Collectors.groupingBy(Student::getAge));

//多重分组,先根据类型分再根据年龄分
Map<Integer, Map<Integer, List<Student>>> typeAgeMap=list.stream()                      
.collect(Collectors.groupingBy(Student::getType,Collectors.groupingBy(Student::getAge)));
```

### reducing规约

```
Integer allAge = list.stream().map(Student::getAge).collect(Collectors.reducing(Integer::sum)).get(); 
```

### partitioningBy分区

```
//分成两部分，一部分大于10岁，一部分小于等于10岁
Map<Boolean, List<Student>> partMap = list.stream().collect(Collectors.partitioningBy(v -> v.getAge() > 10));
```



## 统计

另外，一些产生统计结果的收集器也非常有用。它们主要用于int、double、long等基本类型上，它们可以用来产生类似如下的统计结果。

```
List<Integer> numbers = Arrays.asList(3, 2, 2, 3, 7, 3, 5);
IntSummaryStatistics stats = numbers.stream().mapToInt((x) -> x).summaryStatistics();

System.out.println("列表中最大的数 : " + stats.getMax());
System.out.println("列表中最小的数 : " + stats.getMin());
System.out.println("所有数之和 : " + stats.getSum());
System.out.println("平均数 : " + stats.getAverage()); 
```

# lambda简述

1、Lambda 表达式 ： 在 Java 8 语言中引入的一种新的语法元素和操作符 。 这个操作符为 -> ，该操作符被称为 Lambda 操作符 操作符或 箭头操作符 。 它将 Lambda 分为两个部分 ：

2 、 左侧 ： 指定了 Lambda 表达式需要的 参数列表

3 、 右侧 ： 指定了 Lambda 体 体 ， 是抽象方法的实现逻辑 ， 也即 Lambda表达式要执行的功能 

## lambda表达式语法糖

**语法格式一：无参，无返回值**

```
Runnable r1 =()-> {System.out.println("Hello Lambda!");};
```

**语法格式二：Lambda 需要一个参数，但是没有返回值。**

```
Consumer<String> con = (String str)-> {System.out.println(str);};
```

**语法格式三：数据类型可以省略，因为可由编译器推断得出，称为“类型推断”**

```
Consumer<String> con = (str) -> {System.out.println(str);}
```

**语法格式四：Lambda 若只需要一个参数时，参数的小括号可以省略**

```
Consumer<String> con = str ->{System.out.println(str);
```

**语法格式五：Lambda 需要两个或以上的参数，多条执行语句，并且可以有返回值**

```
Comparator<Integer> com= (x,y)->{System.out.printIn("实现函数式接口方法! ");return Integer.compare(x,y);}
```

**语法格式六：当 Lambda 体只有一条语句时，return与大括号若有，都可以省略**

```
Comparator<Integer> com = (y)-> Integer.compare(x, y);
```

## 方法引用

**1 、什么时候使用方法引用：**当要传递给 Lambda 体的操作，已经有实现的方法了，可以使用方法引用 ！ 方法引用可以看是

Lambda 表达式深层次的表达。换句话说，方法引用就是Lambda表达式，也就是函数式接口的一个实例，通过方法的名字来指向

一个方法，可以认为是Lambda表达式的一个语法糖。

**2 、使用 要求：**

a 、 实现接口的抽象方法的参数列表和返回值类型，必须与方法引用的

b 、 方法的参数列表和返回值类型保持一致！

**3 、使用 语法格式**：使用操作符 格式：使用操作符 :: 将类( 或对象) 与 与 方法名分隔开来。

如下三种主要使用情况：

a 、对象 对象:: 实例方法名

b 、类 类:: 静态方法名

c 、类 类::

**4、举例**

```
例如.对象:实例方法名
Consumer<String> con =(×)-> System.out.printIn(x);
等同于
Consumer<String> con2 =system.out::println;

例如.类:静态方法名
Comparator<Integer> com = (x,y)->Integer.compare(x, y);
Comparator<Integer> com1 =Integer::compare;
int value = com.compare(12,32);

例如.类::实例方法名
BiPredicate<String, String> bp =(x,y)-> x.equals(y);
等同于
BiPredicate<String,String> bp1 =string::equals;
boolean flag = bp1.test("hello","hi");

注意,当函数式接口方法的第一个参数是需要引用方法的调用者.并且第二个参数是需要引用方法的参数(或无参数)时. ClassName::methodName
```

****

# Optional正确的使用姿势

> Opitonal类就是Java提供的为了解决大家平时判断对象是否为空用，通常会用 `null!=obj `这样的方式存在的判断，从而令人头疼导致空指针异常，同Optional的存在可以让代码更加简单，可读性跟高，代码写起来更高效

1、判断对象是否为空，空则new一个对象返回

```
Student student = new Student();
if (null == student){
    return "student为null";
}
return student;
```

```
Student student = new Student();
return Optional.ofNullable(student).orElse(new Student());
```