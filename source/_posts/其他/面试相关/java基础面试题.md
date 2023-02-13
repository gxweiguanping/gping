---
title: java基础面试题
tags: 面试题
categories: 面试题
cover: https://gitee.com/studentgitee/note-picture/raw/master/115.jpg
---
## Java有几种基本数据类型

> 有八种基本数据类型。一个字节等于8个bit位

| 数据类型 | 占用字节 |
| :------: | :------: |
|   byte   |    1     |
|  short   |    2     |
|   int    |    4     |
|   long   |    8     |
|  double  |    4     |
|  float   |    8     |
|   char   |    2     |
| boolean  |    4     |

## 为什么要有包装类型

> 术语：让基本类型也具有对象的特征
>
> 为了让基本类型也具有对象的特征，就出现了包装类型（如我们在使用集合类型Collection时就一定要使用包装类型而非基本类型）因为容器都是装object的，这是就需要这些基本类型的包装器类了。

| 基本类型 | 包装类型 |
| :------: | :------: |
|   byte   |   Byte   |
|  short   |  Short   |
|   int    | Integer  |
|   long   |   Long   |
|  double  |  Double  |
|  float   |  Float   |
|   char   |   Char   |
| boolean  | Boolean  |

```java
自动装箱: new Integer(6)
底层调用: Integer.valueOf(6)

自动拆箱: int i = new Integer(6)
底层调用: i.intValue()
```

## 基本类型和包装类型的区别

- 成员变量包装类型不赋值就是 `null` ，而基本类型有默认值且不是 `null`。
- 包装类型可用于泛型，而基本类型不可以。
- 基本数据类型的局部变量存放在 Java 虚拟机栈中的局部变量表中，基本数据类型的成员变量（未被 `static` 修饰 ）存放在 Java 虚拟机的堆中。包装类型属于对象类型，我们知道几乎所有对象实例都存在于堆中。
- 相比于对象类型， 基本数据类型占用的空间非常小。

**为什么说是几乎所有对象实例呢？** 这是因为 HotSpot 虚拟机引入了 JIT 优化之后，会对对象进行逃逸分析，如果发现某一个对象并没有逃逸到方法外部，那么就可能通过标量替换来实现栈上分配，而避免堆上分配内存

⚠️注意 ： **基本数据类型存放在栈中是一个常见的误区！** 基本数据类型的成员变量如果没有被 `static` 修饰的话（不建议这么使用，应该要使用基本数据类型对应的包装类型），就存放在堆中。

## 面向对象三大特征

> 面向对象的编程语言有封装、继承 、多态等3个主要的特征。

**封装**：简单的来说就是我将不想给别人看的数据，以及别人无需知道的内部细节， “锁起来” ，我们只留下一些入口，使其与外部发生联系。public、private、protected 等权限修饰符 在类的 同程度的 ”锁“ 决定了紧跟其后被定义的东西能够被谁使用。

**继承**：子类继承了父类后，仍然获取了父类中的所有属性和方法，但是由于对象的封装性影响，使得子类调用不了父类的属性和方法，类可以多层继承，但一个类不能同时继承多个类

**多态**：

- 多态使用的前提是要有继承关系，不然没意义

- 方法重写
- 多态不适合属性，只适合方法
- 多态性是一个运行时行为

```java
# 父类的引用指向子类的对象,编译是看左边的类型，执行看右边的类型
B extends A
A b = new B();
```

## 面向对象和面向过程的区别

两者的主要区别在于解决问题的方式不同：

- 面向过程把解决问题的过程拆成一个个方法，通过一个个方法的执行解决问题。
- 面向对象会先抽象出对象，然后用对象执行方法的方式解决问题。

另外，面向对象开发的程序一般更易维护、易复用、易扩展。

## 重载与重写的区别

**重载：两同一不同**

1、`两同`：同一个类中，同一个方法名称

2、`一不同`：方法的参数类型、参数个数、参数顺序不同、

3、重载跟方法的返回类型没有关系

**重写：子类继承父类，可以重写父类方法,方法的重写要遵循“两同两小一大”**

> 构造方法无法被重写

1、`两同`子类重写的方法必须和父类被重写的方法具有相同的`方法名称`、`参数列表`

2、子类重写的方法的返回值类型不能大于父类被重写的方法的返回值类型

- 如果父类的方法的返回值类型是void，那么子类的返回值类型也是void
- 如果父类的方法的返回值类型是基本类型（double），那么子类的返回值类型也是double
- 如果父类的方法的返回值类型A类型，那么子类的返回值类型是A类型或者A类型的子类

3、子类重写的方法使用的访问权限不能小于父类被重写的方法的访问权限

- 子类不能覆盖父类的private方法
- 比如父类的方法时public，那么子类在重写该方法的时候权限要比public小，比如default、protected、private

4、`一大`指的是子类抛出的异常不能大于父类被重写方法的异常

**二者之间的区别**

1. 声明方式不同：基本类型不使用new关键字，而包装类型需要使用new关键字来在堆中分配存储空间；
2. 存储方式及位置不同：基本类型是直接将变量值存储在栈中，而包装类型是将对象放在堆中，然后通过引用来使用；
3. 初始值不同：基本类型的初始值如int为0，boolean为false，而包装类型的初始值为null；
4. 使用方式不同：基本类型直接赋值直接使用就可以，而包装类型在集合如Collection、Map时会使用到；

## == 和 equals区别

**`==`** ：它的作用是判断两个对象的地址是不是相等。即，判断两个对象是不是同一个对象

- 对于基本数据类型`==`比较的是值
- 对于引用数据类型`==`比较的是内存地址

`equals`：分两种情况来理解

- 情况 1：类没有重写 equals() 方法。则通过 equals() 比较该类的两个对象时，等价于通过`==`比较这两个对象。
- 情况 2：类重写了 equals() 方法。一般，我们都重写 equals() 方法来比较两个对象的内容是否相等；若它们的内容相等，则返回 true (即，认为这两个对象相等)。

```java
/*我们来看看String重写的equals方法：它不止判断了内存地址，还增加了字符串是否相同的比较。*/
public boolean equals(Object anObject) {
    //判断内存地址是否相同
    if (this == anObject) {
        return true;
    }
    // 判断参数类型是否是String类型
    if (anObject instanceof String) {
        // 强转
        String anotherString = (String)anObject;
        int n = value.length;
        // 判断两个字符串长度是否相等
        if (n == anotherString.value.length) {
            char v1[] = value;
            char v2[] = anotherString.value;
            int i = 0;
            // 一一比较 字符是否相同
            while (n-- != 0) {
                if (v1[i] != v2[i])
                    return false;
                i++;
            }
            return true;
        }
    }
    return false;
}
```

## String和StringBuffer和StringBuilder区别

 **数据可变和不可变**

1、`String`底层使用一个不可变的字符数组`private final char value[];`所以它内容不可变。

2、`StringBuffer`和`StringBuilder`都继承了`AbstractStringBuilder`底层使用的是可变字符数组：`char[] value;`

**线程安全**

1、`StringBuilder`是线程不安全的，效率较高；而`StringBuffer`是线程安全的，效率较低。

```java
# StringBuilder没有加synchronized，而StringBuffer添加了synchronized锁
@Override
public synchronized StringBuffer append(Object obj) {
    toStringCache = null;
    super.append(String.valueOf(obj));
    return this;
}
@Override
public StringBuilder append(String str) {
    super.append(str);
    return this;
}
```

**操作速度**

> StringBuilder > StringBuffer > String

**什么时候适用**

1、操作少量的数据: 适用 String 

2、单线程操作字符串缓冲区下操作大量数据: 适用 StringBuilder 

3、 多线程操作字符串缓冲区下操作大量数据: 适用 StringBuff

## 接口和抽象类的区别

**设计上的区别：**

1、接口：自上而下，先定义接口约束，在去实现接口（对类进行约束）

2、抽象类：自下而上，已经有了很多类，发现他们有很多相同的共性，提取成抽象类（代码复用）

3、抽象类似对类本质的抽象，表达的是is a 的关系，比如BWM is a Car.由子类去实现

4、接口是对行为上的抽象，表达的是like a的关系，比如 Bird like a Aircraft(像飞行器一样)，但其实本质上是is a Bird,接口的核心是定义的行为

**使用场景：**

1、关注事物的本质，用抽象类

2、关注一个操作的时候，用接口

**语法上的区别：**

1、接口只有定义，不能有方法的实现，但java 1.8中可以定义default方法体，而抽象类可以有定义与实现，方法可在抽象类中实现

2、抽象类中可以包含非抽象的普通方法，接口中的所有方法必须都是抽象的，不能有非抽象的普通方法。

3、抽象类中可以包含静态方法，接口中不能包含静态方法

- 抽象类和接口中都可以包含静态成员变量，抽象类中的静态成员变量的访问类型可以任意，但接口中定义的变量只能是`public static final`类型，并且默认即为`public static final`类型

4、实现接口的关键字为`implements`，继承抽象类的关键字为`extends`。一个类可以实现多个接口，但一个类只能继承一个抽象类。所以，使用接口可以间接地实现多重继承

5、接口方法默认修饰符是 `public`，抽象方法可以有 `public`、`protected` 和` default` 这些修饰符（抽象方法就是为了被重写所以不能使用 private`关键字修饰！）。

## Java访问修饰符有哪些

> Java中的修饰符有public、private、protected 

|  修饰符   | 当前类 | 同包 | 子类 | 其他包 |
| :-------: | :----: | :--: | :--: | :----: |
|  private  |   √    |  ×   |  ×   |   ×    |
|  default  |   √    |  √   |  ×   |   ×    |
| protected |   √    |  √   |  √   |   ×    |
|  public   |   √    |  √   |  √   |   √    |

## static和final区别

| 关键词 | 修饰物 |                           影响                           |
| :----: | :----: | :------------------------------------------------------: |
| final  |  变量  |             分配到常量池中，程序不可改变其值             |
| final  |  方法  |                    子类中将不能被重写                    |
| final  |   类   |                        不能被继承                        |
| static |  变量  | 分配在内存堆上，引用都会指向这一个地址而不会重新分配内存 |
| static | 方法块 |                      虚拟机优先加载                      |
| static |   类   |             可以直接通过类来调用而不需要new              |

## 子父类代码执行顺序

> - 1.父类静态代码块！
> - 2.子类静态代码块！
> - 3.父类代码块！
> - 4.父类构造！
> - 5.子类代码块！
> - 6.子类构造！

## String s = "hello"和String s = new String("hello");区别

`String s = new String("hello");`可能创建两个对象也可能创建一个对象。如果常量池中有`hello`字符串常量的话，则仅仅在堆中创建一个对象。如果常量池中没有`hello`对象，则堆上和常量池都需要创建。

`String s = "hello"`这样创建的对象，JVM会直接检查字符串常量池是否已有"hello"字符串对象，如没有，就分配一个内存存放"hello"，如有了，则直接将字符串常量池中的地址返回给栈。(没有new，没有堆的操作)

## Java中的深拷贝和浅拷贝

> 回答：深拷贝和浅拷贝都是对象拷贝

`浅拷贝`：按位拷贝对象，它会创建一个新对象，这个对象有着原始对象属性值的一份精确贝。如果属性是基本类型，拷贝的就是基本类型的值；如果属性是内存地址（引用类型），拷贝的就是内存地址 ，因此如果其中一个对象改变了这个地址，就会影响到另一个对象。（浅拷贝仅仅复制所考虑的对象，而不复制它所引用的对象。）

上图： 两个引用 `student1 `和` student2` 指向不同的两个对象，但是两个引用 `student1`  和`student2 `中的两个 teacher 引用指向的是同一个对象，所以说明是 `浅拷贝 `

`深拷贝`：在拷贝引用类型成员变量时，为引用类型的数据成员另辟了一个独立的内存空间，实现真正内容上的拷贝。（深拷贝把要复制的对象所引用的对象都复制了一遍。）

上图：两个引用 `student1` 和 `student2` 指向不同的两个对象，两个引用 `student1` 和
`student2 `中的两个 teacher 引用指向的是两个对象，但对 teacher 对象的修改只能影响
student1 对象，所以说是 `深拷贝`

## Java中的异常体系

Java中的异常主要分为Error和Exception。

![image-20230212115226826](https://gitee.com/studentgitee/note-picture/raw/master/image-20230212115226826.png)

> **Error** 指Java程序运行错误，如果程序在启动时出现Error，则启动失败；如果程序运行过程中出现Error，则系统将退出程序。出现Error是系统的内部错误或资源耗尽，Error不能在程序运行过程中被动态处理，一旦出现Error，系统能做的只有记录错误的原因和安全终止。

> Exception 指 Java程序运行异常，在运行中的程序发生了程序员不期望发生的事情，可以被Java异常处理机制处理。Exception也是程序开发中异常处理的核心，可分为RuntimeException（运行时异常）和CheckedException（检查异常）

1、`RuntimeException`（运行时异常）：指在Java虚拟机正常运行期间抛出的异常RuntimeException可以被捕获并处理，如果出现此情况，我们需要抛出异常或者捕获并处理异常。常见的有：`NullPointerException`（空指针错误）、`ClassCastException`（类型转换错误）、`ArrayIndexOutOfBoundsException`（数组越界错误）等;

2、`CheckedException`（检查异常）：指在编译阶段Java编译器检查CheckedException异常，并强制程序捕获和处理此类异常，要求程序在可能出现异常的地方通过try catch语句块捕获异常并处理异常，如果不处理，则编译不通过。常见的有由于I/O错误导致的`IOException`、`SQLException`、`ClassNotFoundException`等。该类异常通常由于打开错误的文件、SQL语法错误、类不存等引起。

### 追问1：异常的处理方式？

回答：异常处理方式有抛出异常和使用try catch语句块捕获异常两种方式。

1、抛出异常：遇到异常时不进行具体的处理，直接将异常抛给调用者，让调用者自己根据情况处理。抛出异常的三种形式：throws、throw和系统自动抛出异常。其中throws作用在方法上，用于定义方法可能抛出的异常；throw作用在方法内，表示明确抛出一个异常。

2、使用try catch捕获并处理异常：使用try catch 捕获异常能够有针对性的处理每种可能出现的异常，并在捕获到异常后根据不同的情况做不同的处理。其使用过程比较简单：用try catch语句块将可能出现异常的代码抱起来即可。

### 追问2： try-catch-finally 如何使用

- `try`块 ： 用于捕获异常。其后可接零个或多个 `catch` 块，如果没有 `catch` 块，则必须跟一个 `finally` 块。
- `catch`块 ： 用于处理 try 捕获到的异常。
- `finally` 块 ： 无论是否捕获或处理异常，`finally` 块里的语句都会被执行。当在 `try` 块或 `catch` 块中遇到 `return` 语句时，`finally` 语句块将在方法返回之前被执行。

```java
try {
    System.out.println("Try to do something");
    throw new RuntimeException("RuntimeException");
} catch (Exception e) {
    System.out.println("Catch Exception -> " + e.getMessage());
} finally {
    System.out.println("Finally");
}
```

**⚠️注意：不要在 finally 语句块中使用 return!** 当 try 语句和 finally 语句中都有 return 语句时，try 语句块中的 return 语句会被忽略。这是因为 try 语句中的 return 返回值会先被暂存在一个本地变量中，当执行到 finally 语句中的 return 之后，这个本地变量的值就变为了 finally 语句中的 return 返回值。

### 追问3：finally 中的代码一定会执行吗？

不一定的！在某些情况下，finally 中的代码不会被执行。

就比如说 finally 之前虚拟机被终止运行的话，finally 中的代码就不会被执行。

```java
try {
    System.out.println("Try to do something");
    throw new RuntimeException("RuntimeException");
} catch (Exception e) {
    System.out.println("Catch Exception -> " + e.getMessage());
    // 终止当前正在运行的Java虚拟机
    System.exit(1);
} finally {
    System.out.println("Finally");
}
```

另外，在以下 2 种特殊情况下，`finally` 块的代码也不会被执行：

1. 程序所在的线程死亡。
2. 关闭 CPU。

## 什么是泛型？有什么作用？

**Java 泛型（Generics）** 是 JDK 5 中引入的一个新特性。使用泛型参数，可以增强代码的可读性以及稳定性。

编译器可以对泛型参数进行检测，并且通过泛型参数可以指定传入的对象类型。比如 `ArrayList<Persion> persons = new ArrayList<Persion>()` 这行代码就指明了该 `ArrayList` 对象只能传入 `Persion` 对象，如果传入其他类型的对象就会报错.

### 追问1：泛型的使用方式有哪几种

泛型一般有三种使用方式:**泛型类**、**泛型接口**、**泛型方法**。

**1、泛型类**

```java
//此处T可以随便写为任意标识，常见的如T、E、K、V等形式的参数常用于表示泛型
//在实例化泛型类时，必须指定T的具体类型
public class Generic<T>{

    private T key;

    public Generic(T key) {
        this.key = key;
    }

    public T getKey(){
        return key;
    }
}
```

如何实例化泛型类：

```java
Generic<Integer> genericInteger = new Generic<Integer>(123456);
```

**2、泛型接口**

```java
public interface Generator<T> {
    public T method();
}
```

实现泛型接口，不指定类型：

```java
class GeneratorImpl<T> implements Generator<T>{
    @Override
    public T method() {
        return null;
    }
}
```

实现泛型接口，指定类型：

```java
class GeneratorImpl<T> implements Generator<String>{
    @Override
    public String method() {
        return "hello";
    }
}
```

**3、泛型方法** ：

```java
   public static < E > void printArray( E[] inputArray )
   {
         for ( E element : inputArray ){
            System.out.printf( "%s ", element );
         }
         System.out.println();
    }
```

使用泛型方法：

```java
// 创建不同类型数组： Integer, Double 和 Character
Integer[] intArray = { 1, 2, 3 };
String[] stringArray = { "Hello", "World" };
printArray( intArray  );
printArray( stringArray  );
```

> 注意: `public static < E > void printArray( E[] inputArray )` 一般被称为静态泛型方法;在 java 中泛型只是一个占位符，必须在传递类型后才能使用。类在实例化时才能真正的传递类型参数，由于静态方法的加载先于类的实例化，也就是说类中的泛型还没有传递真正的类型参数，静态的方法的加载就已经完成了，所以静态泛型方法是没有办法使用类上声明的泛型的。只能使用自己声明的 `<E>`

### 追问2：泛型应用场景

- 自定义接口通用返回结果 `CommonResult<T>` 通过参数 `T` 可根据具体的返回类型动态指定结果的数据类型

### 追问3：什么是泛型的类型擦除

ArrayList<String>和ArrayList<Integer>在编译时是不同的类型，但是在编译完成后都被编译器简化成了ArrayList，这一现象，被称为泛型的类型擦除(Type Erasure)。泛型的本质是参数化类型，而类型擦除使得类型参数只存在于编译期，在运行时，jvm是并不知道泛型的存在的。

### 追问4：为什么要进行泛型类型擦除

> 类型擦除的主要目的是避免过多的创建类而造成的运行时的过度消耗

类型擦除分为两种：

1、无限制类型擦除

当类定义中的类型参数没有任何限制时，在类型擦除后，会被直接替换为Object。在下面的例子中，<T>中的类型参数T就全被替换为了Object(左侧为编译前的代码，右侧为通过字节码文件反编译得到的代码)：

![image-20220703122345900](https://gitee.com/studentgitee/note-picture/raw/master/image-20220703122345900.png)

2、有限制类型擦除

当类定义中的类型参数存在限制时，在类型擦除中替换为类型参数的上界或者下界。下面的代码中，经过擦除后T被替换成了Integer：

![image-20220703122428664](https://gitee.com/studentgitee/note-picture/raw/master/image-20220703122428664.png)

## 何为反射

> 反射之所以被称为框架的灵魂，主要是因为它赋予了我们在运行时分析类以及执行类中方法的能力。通过反射你可以获取任意一个类的所有属性和方法。

### 反射的应用场景

日常使用的 Spring中使用很多的反射机制，为什么你使用 Spring 的时候 ，一个`@Component`注解就声明了一个类为 Spring Bean 呢？为什么你通过一个 `@Value`注解就读取到配置文件中的值呢？究竟是怎么起作用的呢？这些都是因为你可以基于反射分析类，然后获取到类/属性/方法/方法的参数上的注解。你获取到注解之后，就可以做进一步的处理。

### 获取java类字节码的方法

1、Class.class

2、object.getClass()

3、Class.forName("")

- **JDBC 的数据库的连接**

在JDBC 的操作中，如果要想进行数据库的连接，则必须按照以上的几步完成

1. 通过Class.forName()加载数据库的驱动程序 （通过反射加载，前提是引入相关了Jar包）
2. 通过 DriverManager 类进行数据库的连接，连接的时候要输入数据库的连接地址、用户名、密码
3. 通过Connection 接口接收连接

```
public class ConnectionJDBC {  
  
    /** 
     * @param args 
     */  
    //驱动程序就是之前在classpath中配置的JDBC的驱动程序的JAR 包中  
    public static final String DBDRIVER = "com.mysql.jdbc.Driver";  
    //连接地址是由各个数据库生产商单独提供的，所以需要单独记住  
    public static final String DBURL = "jdbc:mysql://localhost:3306/test";  
    //连接数据库的用户名  
    public static final String DBUSER = "root";  
    //连接数据库的密码  
    public static final String DBPASS = "";  
      
      
    public static void main(String[] args) throws Exception {  
        Connection con = null; //表示数据库的连接对象  
        Class.forName(DBDRIVER); //1、使用CLASS 类加载驱动程序 ,反射机制的体现 
        con = DriverManager.getConnection(DBURL,DBUSER,DBPASS); //2、连接数据库  
        System.out.println(con);  
        con.close(); // 3、关闭数据库  
    }  
```

## java线程池

⚠️注意 ：线程池不允许使用Executors去创建，而是通过ThreadPoolExecutor的方式，这样的处理方式让写的同学

更加明确线程池的运行规则，规避资源耗尽的风险。`摘自阿里编码规约`。 说明：Executors各个方法的弊端：

1）newFixedThreadPool和newSingleThreadExecutor:
  主要问题是堆积的请求处理队列可能会耗费非常大的内存，甚至OOM。
2）newCachedThreadPool和newScheduledThreadPool:
  主要问题是线程数最大数是Integer.MAX_VALUE，可能会创建数量非常多的线程，甚至OOM。

```java
public ThreadPoolExecutor(    int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              RejectedExecutionHandler handler) {
        this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue,
             Executors.defaultThreadFactory(), handler);
    }
```

**corePoolSize & maximumPoolSize**

核心线程数（corePoolSize）和最大线程数（maximumPoolSize）是线程池中非常重要的两个概念。

1、当一个新任务被提交到池中，如果当前运行线程小于核心线程数（corePoolSize），即使当前有空闲线程，也会新建一个线程来处理新提交的任务；

2、如果当前运行线程数大于核心线程数（corePoolSize）并小于最大线程数（maximumPoolSize），只有当等待队列已满的情况下才会新建线程。

**keepAliveTime & unit**

keepAliveTime 为超过 corePoolSize 线程数量的线程最大空闲时间，unit 为时间单位。

**等待队列（workQueue）**

任何阻塞队列（BlockingQueue）都可以用来转移或保存提交的任务，线程池大小和阻塞队列相互约束线程池：

1、如果运行线程数小于`corePoolSize`，提交新任务时就会新建一个线程来运行；

2、如果运行线程数大于或等于`corePoolSize`，新提交的任务就会入列等待；如果队列已满，并且运行线程数小于`maximumPoolSize`，也将会新建一个线程来运行；

3、如果线程数大于`maximumPoolSize`，新提交的任务将会根据**拒绝策略**来处理。

下面来看一下三种通用的入队策略：

1、**直接传递**：通过 SynchronousQueue 直接把任务传递给线程。如果当前没可用线程，尝试入队操作会失败，然后再创建一个新的线程。当处理可能具有内部依赖性的请求时，该策略会避免请求被锁定。直接传递通常需要无界的最大线程数（maximumPoolSize），避免拒绝新提交的任务。当任务持续到达的平均速度超过可处理的速度时，可能导致线程的无限增长。

2、**无界队列**：使用无界队列（如 LinkedBlockingQueue）作为等待队列，当所有的核心线程都在处理任务时， 新提交的任务都会进入队列等待。因此，不会有大于 corePoolSize 的线程会被创建（maximumPoolSize 也将失去作用）。这种策略适合每个任务都完全独立于其他任务的情况；例如网站服务器。这种类型的等待队列可以使瞬间爆发的高频请求变得平滑。当任务持续到达的平均速度超过可处理速度时，可能导致等待队列无限增长。

3、**有界队列**：当使用有限的最大线程数时，有界队列（如 ArrayBlockingQueue）可以防止资源耗尽，但是难以调整和控制。队列大小和线程池大小可以相互作用：使用大的队列和小的线程数可以减少CPU使用率、系统资源和上下文切换的开销，但是会导致吞吐量变低，如果任务频繁地阻塞（例如被I/O限制），系统就能为更多的线程调度执行时间。使用小的队列通常需要更多的线程数，这样可以最大化CPU使用率，但可能会需要更大的调度开销，从而降低吞吐量。

**拒绝策略**

当线程池已经关闭或达到饱和（最大线程和队列都已满）状态时，新提交的任务将会被拒绝。 ThreadPoolExecutor 定义了四种拒绝策略：

1、**AbortPolicy**：默认策略，在需要拒绝任务时抛出RejectedExecutionException；

2、**CallerRunsPolicy**：直接在 execute 方法的调用线程中运行被拒绝的任务，如果线程池已经关闭，任务将被丢弃；

3、**DiscardPolicy**：直接丢弃任务；

4、**DiscardOldestPolicy**：丢弃队列中等待时间最长的任务，并执行当前提交的任务，如果线程池已经关闭，任务将被丢弃。

我们也可以自定义拒绝策略，只需要实现 RejectedExecutionHandler； 需要注意的是，拒绝策略的运行需要指定线程池和队列的容量。

## java注解

> Java注解又称Java标注，是在 JDK5 时引入的新特性，注解（也被称为元数据）。

Java注解分类

1. 标准注解，包括@Override、@Deprecated、@SuppressWarnings等，使用这些注解后编译器就会进行检查。
2. 元注解，注解是用于定义注解的注解，包括@Retention、@Target、@Inherited、@Documented、@Repeatable 等。元注解也是Java自带的标准注解，只不过用于修饰注解，比较特殊。
3. 自定义注解，用户可以根据自己的需求定义注解。

**标准注解**

|       注解名称       |                           功能描述                           |
| :------------------: | :----------------------------------------------------------: |
|      @Override       | 检查该方法是否是重写方法，如果发现其父类，或者是引用的接口中并没有该方法时，会报编译错误 |
|     @Deprecated      |          标记过时方法，如果使用该方法，会报编译警告          |
|  @SuppressWarnings   |              指示编译器去忽略注释解中声明的警告              |
| @Functionallnterface |           java8支持，标识一个匿名函数或函数式接口            |

**元注解**

@Retention用来定义该注解在哪一个级别可用，在源代码中(SOURCE)、类文件中(CLASS)或者运行时(RUNTIME)

```java
@Documented@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.ANNOTATION_TYPE)
public @interface Retention {
  RetentionPolicy value();
}
public enum RetentionPolicy {
  //此注解类型的信息只会记录在源文件中，编译时将被编译器丢弃，也就是说
  //不会保存在编译好的类信息中
  SOURCE,
  //编译器将注解记录在类文件中，但不会加载到JVM中。如果一个注解声明没指定范围，则系统
  //默认值就是Class
  CLASS,
  //注解信息会保留在源文件、类文件中，在执行的时也加载到Java的JVM中，因此可以反射性的读取。
  RUNTIME
}
```

@Documented 生成文档信息的时候保留注解，对类作辅助说明

```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.ANNOTATION_TYPE})
public @interface Documented {
}
```

@Target：用于描述注解的使用范围（即：被描述的注解可以用在什么地方）

```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.ANNOTATION_TYPE)
public @interface Target {
    ElementType[] value();
}
```

|   Target类型    |                作用                |
| :-------------: | :--------------------------------: |
|      TYPE       | 应用于类、接口(包括注解类型)、枚举 |
|      FIELD      |          应用于字段或属性          |
|    PARAMETER    |          应用于方法的参数          |
|     METHOD      |             应用于方法             |
|   CONSTRUCTOR   |           应用于构造函数           |
| LOCAL_VARIABLE  |           应用于局部变量           |
| ANNOTATION_TYPE |         应用于一个注解类型         |
|     PACKAGE     |              应用于包              |
| TYPE_PARAMETER  |           应用于类型变量           |
|    TYPE_USE     |     应用于任何使用类型的语句中     |
|     MODULE      |                                    |



![](java集合面试题.assets/java-collection-hierarchy.77b66a51.png)



## ArrayList 扩容机制

先从 ArrayList 的构造函数说起

### (JDK8)ArrayList 有三种，构造方法

```
 /**
     * 默认初始容量大小
     */
    private static final int DEFAULT_CAPACITY = 10;


    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

    /**
     *默认构造函数，使用初始容量10构造一个空列表(无参数构造)
     */
    public ArrayList() {
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
    }

    /**
     * 带初始容量参数的构造函数。（用户自己指定容量）
     */
    public ArrayList(int initialCapacity) {
        if (initialCapacity > 0) {//初始容量大于0
            //创建initialCapacity大小的数组
            this.elementData = new Object[initialCapacity];
        } else if (initialCapacity == 0) {//初始容量等于0
            //创建空数组
            this.elementData = EMPTY_ELEMENTDATA;
        } else {//初始容量小于0，抛出异常
            throw new IllegalArgumentException("Illegal Capacity: "+initialCapacity);
        }
    }

   /**
    *构造包含指定collection元素的列表，这些元素利用该集合的迭代器按顺序返回
    *如果指定的集合为null，throws NullPointerException。
    */
     public ArrayList(Collection<? extends E> c) {
        elementData = c.toArray();
        if ((size = elementData.length) != 0) {
            // c.toArray might (incorrectly) not return Object[] (see 6260652)
            if (elementData.getClass() != Object[].class)
                elementData = Arrays.copyOf(elementData, size, Object[].class);
        } else {
            // replace with empty array.
            this.elementData = EMPTY_ELEMENTDATA;
        }
    }
```

> **以无参数构造方法创建 `ArrayList` 时，实际上初始化赋值的是一个空数组。当真正对数组进行添加元素操作时，才真正分配容量。即向数组中添加第一个元素时，数组容量扩为 10。**

### add方法

```java
  /**
     * 将指定的元素追加到此列表的末尾。
  */
    public boolean add(E e) {
        //添加元素之前，先调用ensureCapacityInternal方法
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        //这里看到ArrayList添加元素的实质就相当于为数组赋值
        elementData[size++] = e;
        return true;
    }
```

### ensureCapacityInternal方法

```
//得到最小扩容量
private void ensureCapacityInternal(int minCapacity) {
    if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
       // 获取默认的容量和传入参数的较大值，第一次进入minCapacity = 10
        minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
    }

    ensureExplicitCapacity(minCapacity);
}
```

> **当 要 add 进第 1 个元素时，minCapacity 为 1，在 Math.max()方法比较后，minCapacity 为 10。**

### ensureExplicitCapacity方法

```
// 判断是否需要扩容
private void ensureExplicitCapacity(int minCapacity) {
    modCount++;
    // overflow-conscious code
    if (minCapacity - elementData.length > 0)
    //   //调用grow方法进行扩容，调用此方法代表已经开始扩容了
        grow(minCapacity);
}
```

- 当我们要 add 进第 1 个元素到 ArrayList 时，elementData.length 为 0 （因为还是一个空的 list），因为执行了 `ensureCapacityInternal()` 方法 ，所以 minCapacity 此时为 10。此时，`minCapacity - elementData.length > 0`成立，所以会进入 `grow(minCapacity)` 方法。
- 当 add 第 2 个元素时，minCapacity 为 2，此时 e lementData.length(容量)在添加第一个元素后扩容成 10 了。此时，`minCapacity - elementData.length > 0` 不成立，所以不会进入 （执行）`grow(minCapacity)` 方法。
- 添加第 3、4···到第 10 个元素时，依然不会执行 grow 方法，数组容量都为 10。

直到添加第 11 个元素，minCapacity(为 11)比 elementData.length（为 10）要大。进入 grow 方法进行扩容

### grow扩容方法

```
/**
* 要分配的最大数组大小
*/
private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

private void grow(int minCapacity) {
    // overflow-conscious code
    // 原来的长度
    int oldCapacity = elementData.length;
//将oldCapacity 右移一位，其效果相当于oldCapacity /2，
//我们知道位运算的速度远远快于整除运算，整句运算式的结果就是将新容量更新为旧容量的1.5倍，
    int newCapacity = oldCapacity + (oldCapacity >> 1);
//然后检查新容量是否大于最小需要容量，若还是小于最小需要容量，那么就把最小需要容量当作数组的新容量，
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
// 如果新容量大于 MAX_ARRAY_SIZE,进入(执行) `hugeCapacity()` 方法来比较 minCapacity 和 MAX_ARRAY_SIZE，
//如果minCapacity大于最大容量，则新容量则为`Integer.MAX_VALUE`，否则，新容量大小则为MAX_ARRAY_SIZE 即为 `Integer.MAX_VALUE - 8`。
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    // minCapacity is usually close to size, so this is a win:
    elementData = Arrays.copyOf(elementData, newCapacity);
}
```

**int newCapacity = oldCapacity + (oldCapacity >> 1),所以 ArrayList 每次扩容之后容量都会变为原来的 1.5 倍左右（oldCapacity 为偶数就是 1.5 倍，否则是 1.5 倍左右）！** 奇偶不同，比如 ：10+10/2 = 15, 33+33/2=49。如果是奇数的话会丢掉小数.

> ">>"（移位运算符）：>>1 右移一位相当于除 2，右移 n 位相当于除以 2 的 n 次方。这里 oldCapacity 明显右移了 1 位所以相当于 oldCapacity /2。对于大数据的 2 进制运算,位移运算符比那些普通运算符的运算要快很多,因为程序仅仅移动一下而已,不去计算,这样提高了效率,节省了资源

- 当 add 第 1 个元素时，oldCapacity 为 0，经比较后第一个 if 判断成立，newCapacity = minCapacity(为 10)。但是第二个 if 判断不会成立，即 newCapacity 不比 MAX_ARRAY_SIZE 大，则不会进入 `hugeCapacity` 方法。数组容量为 10，add 方法中 return true,size 增为 1。
- 当 add 第 11 个元素进入 grow 方法时，newCapacity 为 15，比 minCapacity（为 11）大，第一个 if 判断不成立。新容量没有大于数组最大 size，不会进入 hugeCapacity 方法。数组容量扩为 15，add 方法中 return true,size 增为 11。

**这里补充一点比较重要，但是容易被忽视掉的知识点：**

- java 中的 `length`属性是针对数组说的,比如说你声明了一个数组,想知道这个数组的长度则用到了 length 这个属性.
- java 中的 `length()` 方法是针对字符串说的,如果想看这个字符串的长度则用到 `length()` 这个方法.
- java 中的 `size()` 方法是针对泛型集合说的,如果想看这个泛型有多少个元素,就调用此方法来查看
- 以此类推······

## ArrayList 与 LinkedList 区别

1. **是否保证线程安全：** `ArrayList` 和 `LinkedList` 都是不同步的，也就是不保证线程安全；

2. **底层数据结构：** `ArrayList` 底层使用的是 **`Object` 数组**；`LinkedList` 底层使用的是 **双向链表** 数据结构（JDK1.6 之前为循环链表，JDK1.7 取消了循环。注意双向链表和双向循环链表的区别，下面有介绍到！）

3. 插入和删除是否受元素位置的影响：

   - `ArrayList` 采用数组存储，所以插入和删除元素的时间复杂度受元素位置的影响。 比如：执行`add(E e)`方法的时候， `ArrayList` 会默认在将指定的元素追加到此列表的末尾，这种情况时间复杂度就是 O(1)。但是如果要在指定位置 i 插入和删除元素的话（`add(int index, E element)`）时间复杂度就为 O(n-i)。因为在进行上述操作的时候集合中第 i 和第 i 个元素之后的(n-i)个元素都要执行向后位/向前移一位的操作。

   - `LinkedList` 采用链表存储，所以，如果是在头尾插入或者删除元素不受元素位置的影响（`add(E e)`、`addFirst(E e)`、`addLast(E e)`、`removeFirst()` 、 `removeLast()`），时间复杂度为 O(1)，如果是要在指定位置 `i` 插入和删除元素的话（`add(int index, E element)`，`remove(Object o)`）， 时间复杂度为 O(n) ，因为需要先移动到指定位置再插入。

4. **是否支持快速随机访问：** `LinkedList` 不支持高效的随机元素访问，而 `ArrayList` 支持。快速随机访问就是通过元素的序号快速获取元素对象(对应于`get(int index)`方法)。

5. **内存空间占用：** `ArrayList` 的空 间浪费主要体现在在 list 列表的结尾会预留一定的容量空间，而 LinkedList 的空间花费则体现在它的每一个元素都需要消耗比 ArrayList 更多的空间（因为要存放直接后继和直接前驱以及数据）。

另外，不要下意识地认为 `LinkedList` 作为链表就最适合元素增删的场景。我在上面也说了，`LinkedList` 仅仅在头尾插入或者删除元素的时候时间复杂度近似 O(1)，其他情况增删元素的时间复杂度都是 O(n) 。

## HashMap的底层实现

JDK1.8 之前

JDK1.8 之前 `HashMap` 底层是 **数组和链表** 结合在一起使用也就是 **链表散列**，采用头插法。**HashMap 通过 key 的 hashCode 经过扰动函数处理过后得到 hash 值，然后通过 (n - 1) & hash 判断当前元素存放的位置（这里的 n 指的是数组的长度），如果当前位置存在元素的话，就判断该元素与要存入的元素的 hash 值以及 key 是否相同，如果相同的话，直接覆盖，不相同就通过拉链法解决冲突。**

**所谓扰动函数指的就是 HashMap 的 hash 方法。使用 hash 方法也就是扰动函数是为了防止一些实现比较差的 hashCode() 方法 换句话说使用扰动函数之后可以减少碰撞。**

**JDK 1.8 HashMap 的 hash 方法源码:**

JDK 1.8 的 hash 方法 相比于 JDK 1.7 hash 方法更加简化，但是原理不变。

```java
    static final int hash(Object key) {
      int h;
      // key.hashCode()：返回散列值也就是hashcode
      // ^ ：按位异或
      // >>>:无符号右移，忽略符号位，空位都以0补齐
      return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
  }
```

对比一下 JDK1.7 的 HashMap 的 hash 方法源码.

```java
static int hash(int h) {
    // This function ensures that hashCodes that differ only by
    // constant multiples at each bit position have a bounded
    // number of collisions (approximately 8 at default load factor).

    h ^= (h >>> 20) ^ (h >>> 12);
    return h ^ (h >>> 7) ^ (h >>> 4);
}
```

相比于 JDK1.8 的 hash 方法 ，JDK 1.7 的 hash 方法的性能会稍差一点点，因为毕竟扰动了 4 次。

所谓 **“拉链法”** 就是：将链表和数组相结合。也就是说创建一个链表数组，数组中每一格就是一个链表。若遇到哈希冲突，则将冲突的值加到链表中即可。

![jdk1.7_hashmap.05c0a661](https://gitee.com/studentgitee/note-picture/raw/master/jdk1.7_hashmap.05c0a661.png)

**JDK1.8 之后**

相比于之前的版本， JDK1.8 之后在解决哈希冲突时有了较大的变化，采用尾插法进行插入元素，当链表长度大于阈值（默认为 8）（将链表转换成红黑树前会判断，如果当前数组的长度小于 64，那么会选择先进行数组扩容，而不是转换为红黑树）时，将链表转化为红黑树，以减少搜索时间。

![jdk1.8之后的内部结构-HashMap](https://gitee.com/studentgitee/note-picture/raw/master/jdk1.8_hashmap.ae1eae18.png)

> TreeMap、TreeSet 以及 JDK1.8 之后的 HashMap 底层都用到了红黑树。红黑树就是为了解决二叉查找树的缺陷，因为二叉查找树在某些情况下会退化成一个线性结构。

## HashMap的put方法

![hashmap-put](https://gitee.com/studentgitee/note-picture/raw/master/hashmap-put.png)

1、put(key, value)中直接调用了内部的putVal方法，并且先对key进行了hash操作；

2、putVal方法中，先检查HashMap数据结构中的索引数组表是否位空，如果是的话则进行一次resize扩容操作；

3、以HashMap索引数组表的长度减一与key的hash值进行与运算，得出在数组中的索引，如果索引指定的位置值为空，则新建一个k-v的新节点；

4、如果不满足的3的条件，说明索引指定的数组位置的已经存在内容，这个时候称之碰撞出现；

5、在上面判断流程走完之后，计算HashMap全局的modCount值，以便对外部并发的迭代操作提供修改的Fail-fast判断提供依据，于此同时增加map中的记录数，并判断记录数是否触及容量扩充的阈值，触及则进行一轮resize操作；

6、在步骤4中出现碰撞情况时，从步骤7开始展开新一轮逻辑判断和处理；

7、判断key索引到的节点（暂且称作被碰撞节点）的hash、key是否和当前待插入节点（新节点）的一致，如果是一致的话，则先保存记录下该节点；如果新旧节点的内容不一致时，则再看被碰撞节点是否是树（TreeNode）类型，如果是树类型的话，则按照树的操作去追加新节点内容；如果被碰撞节点不是树类型，则说明当前发生的碰撞在链表中（此时链表尚未转为红黑树），此时进入一轮循环处理逻辑中；

8、循环中，先判断被碰撞节点的后继节点是否为空，为空则将新节点作为后继节点，作为后继节点之后并判断当前链表长度是否超过最大允许链表长度8，如果大于的话，需要进行一轮是否转树的操作；如果在一开始后继节点不为空，则先判断后继节点是否与新节点相同，相同的话就记录并跳出循环；如果两个条件判断都满足则继续循环，直至进入某一个条件判断然后跳出循环；

9、步骤8中转树的操作treeifyBin，如果map的索引表为空或者当前索引表长度还小于64（最大转红黑树的索引数组表长度），那么进行resize操作就行了；否则，如果被碰撞节点不为空，那么就顺着被碰撞节点这条树往后新增该新节点；

10、最后，回到那个被记住的被碰撞节点，如果它不为空，默认情况下，新节点的值将会替换被碰撞节点的值，同时返回被碰撞节点的值（V）。

## HashMap源码中在计算hash值的时候为什么要右移16位

> 尽量扰动计算的出来的hash值，减少哈希冲突。

```java
public class Hash {
    public static void main(String[] args) {
        for (int i = 1; i < 11; i++) {
            Integer a = 65536 * i + 311156;
            System.out.println(hash(a) % 16);
        }

    }
    static final int hash(Object key) {
        int h;
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    }
}
```

⚠️注意 ： 扰动函数只有在hashCode值大于65536的时候效果才是最显著的，因为需要右移16位，所以hashCode值转换成二进制的位置要大于16位，也就是2的16次方（65536），如果hashCode太小，扰动的效果不大，因为hashCode跟hashCode右移16位进行疑惑时，将会得到它本身。

补充知识点：

| 操作符 |                             描述                             |              例子              |
| :----: | :----------------------------------------------------------: | :----------------------------: |
|   ＆   |            如果相对应位都是1，则结果为1，否则为0             | （A＆B），得到12，即0000 1100  |
|   \|   |           如果相对应位都是 0，则结果为 0，否则为 1           | （A \| B）得到61，即 0011 1101 |
|   ^    |            如果相对应位值相同，则结果为0，否则为1            | （A ^ B）得到49，即 0011 0001  |
|   〜   |     按位取反运算符翻转操作数的每一位，即0变成1，1变成0。     |  （〜A）得到-61，即1100 0011   |
|   <<   |     按位左移运算符。左操作数按位左移右操作数指定的位数。     |  A << 2得到240，即 1111 0000   |
|   >>   |     按位右移运算符。左操作数按位右移右操作数指定的位数。     |      A >> 2得到15即 1111       |
|  >>>   | 按位右移补零操作符。左操作数的值按右操作数指定的位数右移，移动得到的空位以零填充。 |     A>>>2得到15即0000 1111     |

## HashMap扩容的时候为什么是2的n次幂

为了能让 HashMap 存取高效，尽量较少碰撞，也就是要尽量把数据分配均匀。我们上面也讲到了过了，Hash 值的范围值-2147483648 到 2147483647，前后加起来大概 40 亿的映射空间，只要哈希函数映射得比较均匀松散，一般应用是很难出现碰撞的。但问题是一个 40 亿长度的数组，内存是放不下的。所以这个散列值是不能直接拿来用的。用之前还要先做对数组的长度取模运算，得到的余数才能用来要存放的位置也就是对应的数组下标。这个数组下标的计算方法是“ `(n - 1) & hash`”。（n 代表数组长度）。这也就解释了 HashMap 的长度为什么是 2 的幂次方。

**这个算法应该如何设计呢？**

我们首先可能会想到采用%取余的操作来实现。但是，重点来了：**“取余(%)操作中如果除数是 2 的幂次则等价于与其除数减一的与(&)操作（也就是说 hash%length==hash&(length-1)的前提是 length 是 2 的 n 次方；）。”** 并且 **采用二进制位操作 &，相对于%能够提高运算效率，这就解释了 HashMap 的长度为什么是 2 的幂次方。**

## HashMap线程不安全的原因

jdk1.8的hashMap

```java
//实际存储的key-value键值对的个数
transient int size;
//阈值，当table == {}时，该值为初始容量（初始容量默认为16）；当table被填充了，也就是为table分配内存空间后，threshold一般为 capacity*loadFactory。HashMap在进行扩容时需要参考threshold，后面会详细谈到
int threshold;
//负载因子，代表了table的填充度有多少，默认是0.75
final float loadFactor;
//用于快速失败，由于HashMap非线程安全，在对HashMap进行迭代时，如果期间其他线程的参与导致HashMap的结构发生变化了（比如put，remove等操作），需要抛出异常ConcurrentModificationException
transient int modCount;
```

```java
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
        //如果没有hash碰撞则直接插入元素
        if ((p = tab[i = (n - 1) & hash]) == null) 
            tab[i] = newNode(hash, key, value, null);
        else {
            Node<K,V> e; K k;
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
            else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else {
                for (int binCount = 0; ; ++binCount) {
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;
        // 当size（已存放元素个数） > thrshold（阀值），进行扩容
        resize();
        if (++size > threshold)
            resize();
        afterNodeInsertion(evict);
        return null;
    }
```

这是jdk1.8中HashMap中put操作的主函数， 注意第6行代码，如果没有hash碰撞则会直接插入元素。如果线程A和线程B同时进行put操作，刚好这两条不同的数据hash值一样，并且该位置数据为null，所以这线程A、B都会进入第6行代码中。

假设一种情况，线程A进入后还未进行数据插入时挂起，而线程B正常执行，从而正常插入数据，然后线程A获取CPU时间片，此时线程A不用再进行hash判断了，问题出现：线程A会把线程B插入的数据给覆盖，发生线程不安全。

## HashMap 和 Hashtable 的区别

1. **线程是否安全：** `HashMap` 是非线程安全的，`Hashtable` 是线程安全的,因为 `Hashtable` 内部的方法基本都经过`synchronized` 修饰。（如果你要保证线程安全的话就使用 `ConcurrentHashMap` 吧！）；
2. **效率：** 因为线程安全的问题，`HashMap` 要比 `Hashtable` 效率高一点。另外，`Hashtable` 基本被淘汰，不要在代码中使用它；
3. **对 Null key 和 Null value 的支持：** `HashMap` 可以存储 null 的 key 和 value，但 null 作为键只能有一个，null 作为值可以有多个；Hashtable 不允许有 null 键和 null 值，否则会抛出 `NullPointerException`。
4. **初始容量大小和每次扩充容量大小的不同 ：** ① 创建时如果不指定容量初始值，`Hashtable` 默认的初始大小为 11，之后每次扩充，容量变为原来的 2n+1。`HashMap` 默认的初始化大小为 16。之后每次扩充，容量变为原来的 2 倍。② 创建时如果给定了容量初始值，那么 Hashtable 会直接使用你给定的大小，而 `HashMap` 会将其扩充为 2 的幂次方大小（`HashMap` 中的`tableSizeFor()`方法保证，下面给出了源代码）。也就是说 `HashMap` 总是使用 2 的幂作为哈希表的大小,后面会介绍到为什么是 2 的幂次方。
5. **底层数据结构：** JDK1.8 以后的 `HashMap` 在解决哈希冲突时有了较大的变化，当链表长度大于阈值（默认为 8）（将链表转换成红黑树前会判断，如果当前数组的长度小于 64，那么会选择先进行数组扩容，而不是转换为红黑树）时，将链表转化为红黑树，以减少搜索时间。Hashtable 没有这样的机制。

## HashMap和TreeMap的区别

1、`TreeMap` 和`HashMap` 都继承自`AbstractMap` ，但是需要注意的是`TreeMap`它还实现了`NavigableMap`接口和`SortedMap` 接口。

![image-20220703220032649](https://gitee.com/studentgitee/note-picture/raw/master/image-20220703220032649.png)

2、HashMap和TreeMap都是线程不安全的

3、HashMap是通过hash值进行快速查找的；HashMap中的元素是没有顺序的；TreeMap中
所有的元素都是有某一固定顺序的，如果需要得到一个有序的结果，就应该使用TreeMap；

