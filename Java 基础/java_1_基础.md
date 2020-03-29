### 【 java_基础  】

#### Java 与 C++

```java
/*
 · java 存粹是面向对象的语言，所有类都继承于 Object；
 	而 C++ 为了兼容 c ，也支持面向过程
 	
 · java 通过虚拟机实现跨平台特性！java 字节码文件可以在任意部署了 JVM 的平台上执行！
 	c++ 是平台相关的

· java 是解释型与编译型并存的语言，源程序生成的字节码文件，会被 jvm 会解释执行；并且，在运行期，对于“热点代码”，JVM 会将其编译为 本地机器码，提高了热点代码的执行效率，热点代码便不需要被频繁的解释！
 	C++ 是编译型语言，源程序经过编译和链接后 直接生成可执行的 二进制代码，因此 C++ 效率更高！
 	【热点代码】某个代码块、方法 运行的特别频繁！
 	
 · java 没有指针，它只有对象的引用，可以理解为 C++ 中安全的指针（因为不会直接操作内存？有 GC 管理内存？）
 	c++ 和 c都具有指针，c++ 的引用表示的是变量的别名！使用 指针可能会引起系统安全问题（可以直接操作内存地址？）
 
 · java 有 垃圾回收器 自动回收垃圾，并且不需要显示申请空间！
 	C++ 需要开发人员自己管理内存分配（包括资源的申请和释放），c++通常会将 需要释放资源的代码放到析构函数中，而 java 中只有拯救自己的 finalize() 方法！
 	
 · java 中只有的单继承
 	C++ 支持多重继承，进而会引出虚基类等概念！
 
 · java 不支持运算符重载！
 	c++ 中支持运算符重载！

*/
```



#### 为什么要面向对象

```java
/*
面向过程：分析需求所需要的步骤，通过函数一步一步实现这步骤，依次调用即为面向过程

面向对象：将整个需求按照其特点、功能划分，将这些特点及功能封装在一起即组成一个对象，封装成对象不是为了完成某个步骤。对象是事物的抽象描述，它描述的是抽象事物总体的行为特征！

为什么要面向对象？
因为面向过程有诸多问题：
1、软件的可重用性差：相似的代码总被复制粘贴！
2、软件的可维护性差：代码可读性差，维护成本高！
3、面向过程开发的软件，不能满足用户需求的变化：因为面向过程往往使用结构化方法开发，使得功能一层一层的往下切分，当需求变化时，代码层级之间变化较大！

面向对象的优点：
1、代码可重用性高。例如：继承
2、易于维护：程序可读性高！
3、易于扩展：存在 继承、封装、多态等特性，可以设计处高内聚、低耦合的系统结构，使得系统灵活，易于扩展！
4、更安全：访问控制修饰符的存在，使得数据可对外开放，可对外不可见，更加安全！
*/
```

#### 面向对象的特性

##### 封装

把描述一个对象的属性和行为封装成一个类，把该对象所拥有的具体的业务逻辑功能实现封装成一个方法。封装通过访问控制修饰符，实现私有化属性，公有化方法，保证数据安全性！

##### 继承

即`is-a`关系，描述类的特殊与一般的关系！特殊的类为子类，一般的类为父类，子类拥有父类所有的属性和行为，它实现了代码复用性！

```java
/*
 · java 中不支持多重继承，即一个类只能有一个父类存在（Object 是所有类的超类）
 · java 构造方法不可继承，如果构造函数被 private 则该类也不可被继承！
 · java 中类的初始化顺序
 	1、初始化 父类中的 静态成员变量 和 静态代码块
 	2、初始化 子类中的 静态成员变量 和 静态代码块
 	
 	3、初始化 父类中的 普通成员变量 和 代码块，再执行 父类的构造方法（如果父类构造方法为 private 则初始化子类的时候不可以被执行）
 	4、初始化 子类中的 普通成员变量 和 代码块，再执行 子类的构造方法
 
 · 子类拥有的特点：
 	1、子类拥有对父类 非private 的属性和方法
 	2、子类可以添加自己的方法和属性，即对父类进行扩展
 	3、子类可以重新定义父类的方法，即方法的覆盖和重写
*/
```

###### 重载和重写

```java
/*
方法的名称和参数列表称为方法的签名！

【重载】 
	同一个类中，方法名称相同，但参数列表不同的方法，就是互为重载方法!对于重载的方法，其返回类型不作为判断标准，即不能有两个 方法签名相同 但 返回类型不同 的方法出现在同一个类中！它描述的是行为相似但具体实现方法有差异！
	
【重写（覆盖）】
	描述同一个行为时，对子类而言，父类的方法实现不适用，或者父类未实现该方法，需要自定义方法实现，于是将继承的父类的方法进行重写/覆盖！
	如果一个子类中定义了一个与父类 【 签名相同的方法 】，那么子类中的该方法就覆盖了超类中签名相同的方法！返回类型不是签名的一部分，但是在覆盖方法时，一定要保证返回类型的兼容性，即子类的返回类型应为超类该方法返回类型的子类型！ 
*/
```

##### 多态

多态 指 程序中定义的`引用类型变量` 在编译器` 无法确定其引用的具体类型`，也`无法确定其调用的具体方法`！而是在运行期才能确定其引用的具体对象，以及所要调用具体方法。

```java
/*
 · 多态的三种实现方式
 	1、通过子类对父类的覆盖实现！
 	2、通过在一个类中对方法的重载来实现！
 	3、通过将子类作为父类对象来实现！
*/
```

#### 抽象类 vs 接口

##### 抽象类

```java
/*
 · 抽象类：abstract 修饰
 	类在继承层次中很通用，它用于声明抽象行为，行为的具体实现交由 不同的 子类完成！它相对普通类的继承而言，不需要实现它的任何实例，它只作为顶层的通用规范！
 · 注意：
 	1、包含一个或者多个抽象方法的类本身必须被声明为抽象类！
 	2、抽象类除了声明抽象方法外，还可以包含具体的数据和具体的方法！
 	3、类即使不包含抽象方法，也可以将其声明为抽象类！
 	4、抽象类不能被实例化！
 	5、可以定义一个抽象类的 对象变量，但是它只能引用非抽象子类的对象！

*/
```

##### 接口

```java
/*
 · 接口：interface 修饰
 	接口技术主要用于描述类的功能，而并不给出每个功能的具体实现，其作用抽象类类似！
 · 注意：
 	1、一个类允许实现多个接口！也可以被子类扩展！
 	2、接口不是类，不能通过 new 运算符实例化一个接口！
 	3、可以定义一个 接口变量，但它只能引用其实现类的对象！
 	4、接口中的变量必须是 static final 的常量，不能是实例域！
 	5、jdk 8 之前，接口中不能实现方法，也不能包含静态方法，jdk 8 开始，可以在 接口中实现默认方法和静态方法！
 	6、接口中的方法自动会被设置成 public，接口中的常量自动被设为 public static final
 	
 · 有关 【接口的 静态方法、默认方法】详见 《 java 8 》这一章节！
*/
```

##### 接口 vs 抽象类

```java
/*
 · 区别
	1、抽象类只能单继承，而一个类可以实现多个接口！
 	2、抽象类中可以包含实例方法、实例域，甚至可以没有抽象方法；接口中只能包含 静态域，并且 jdk 8之前，接口中不能实现任何方法，jdk 8 开始，加入默认方法（default）、静态方法（static）！

· 如何选择
	根据需求，如果只需要定义一些抽象方法，而不需要其他额外的具体方法或者变量，则可以选择接口！否则，选择抽象类，因为抽象类中可以实现 实例域、实例方法！
*/
```

#### 泛型

+ 详见 ： 《 java_泛型 》章节

+ [泛型详解](https://blog.csdn.net/stypace/article/details/42102567)
+ [泛型面试题汇总](https://www.cnblogs.com/huajiezh/p/6411123.html)

#### java 元注解

##### 注解介绍

```java
/*
 · 注解是 绑定到 程序源代码元素 的元数据！可以代替繁杂的配置文件，简化开发！
 · 典型用例：
 	1、向编译器传递信息 —— 使用注解，编译器可以检测错误或者抑制警告
 	2、编译时 和 部署时 处理 —— 软件工具可以处理注解并生成代码、生成配置配置文件等！
 	3、运行时 处理 —— 可以在运行时检查注解，以自定义程序的行为！
*/
```

##### java 中有哪几种元注解

​		 java 中提供了 4 个元注解，元注解的作用是负责注解 其他的注解！

+ **@Target**

  ```java
  /*
   · 说明注解所修饰的对象的范围
  */
  @Documented
  @Retention(RetentionPolicy.RUNTIME)
  @Target(ElementType.ANNOTATION_TYPE)
  public @interface Target {
      /**
       * @return an array of the kinds of elements an annotation type can be applied to
       */
      ElementType[] value();
  }
  
  public enum ElementType {
      /*类、接口（包括注解类型）、枚举*/  /* 域（包括 枚举、常量）*/	/*方法*/
      TYPE,								 FIELD,						  METHOD,
      /*参数*/		/*构造器*/		/*本地变量*/		/*注解类型*/		/*包*/		/*类型参数*/
      PARAMETER,  CONSTRUCTOR, 	LOCAL_VARIABLE, 	ANNOTATION_TYPE,	PACKAGE,	TYPE_PARAMETER,
      TYPE_USE,	MODULE
  }
  
  // 示例：
  @Target({ElementType.TYPE, ElementType.METHOD})
  public @interface MyAnn{ /* @MyAnn 注解只能标注在 类、接口、枚举、方法上 */}
  ```

+ **@Retention（保留策略）**

  ```java
  /*
   · 定义了该注解保留时间的长短
  */
  public @interface Retention {
      /**
       * @return the retention policy
       */
      RetentionPolicy value();
  }
  public enum RetentionPolicy {
      // 在 源文件中 保留：只在源文件中有效，编译时被忽略
      SOURCE,
      // 在 class文件中 保留：源文件被编译，生成并保留到 class 文件！在 jvm 运行时不可见
      CLASS,
      // 在 运行时 保留：（运行时有效）运行时可见
      RUNTIME
  }
  
  ```

+ **@Documented**

  ```java
  /*
   · 用于描述其他类型的 annotation 应该被作为 被标注的程序的 公共API，注解会被 javadoc 此类的工具文档化
   · 它是个 标记注解，没有成员
  */
  public @interface Documented { }
  ```

+ **@Inherited**

  ```java
  /*
   · 阐述某个被标记的类型是被继承的
   · 如果 使用 一个被 @Inherited 修饰的 annotation 类型 标记一个类，则说明，该 annotation 会被用于该 类的子类！ 
   · 是一个标记注解，没有成员
  */
  public @interface Inherited {}
  ```

##### 注解定义

```java
/*
 · 注解使用 @interface ，而不是 class、enum、interface
*/
public @interface MyAnnotation{}

/*
 · 注解 可以定义属性
 · 定义注解的属性，形式上类似于定义方法！
 	当注解指定属性后，在使用注解时，就必须要给属性赋值，除非定义时通过 default 关键字给定默认值
*/
public @interface MyAnnotation{
    String strValue();
    int intValue();
}
// 使用
@MyAnnotation(strValue="test",intValue=1)
public class MyTestClass{}
```

##### 注解深入：注解处理器

```java
/*
 · 当定义一个注解后，还需要一个注解处理器来执行注解的内部逻辑，注解处理器定义了注解的处理逻辑，涉及到反射机制和线程机制！！！
*/
```



[详见：java 注解 面试](https://www.jianshu.com/p/53cad116d619)

#### 创建数组

+ 简介

  数组是一种数据结构，它储存同一类型数值的集合，通过一个整型的下标可以访问数组中的每一个值

+ 语法

  1. 声明数组

     声明数组变量时，需要指出数组类型（数据元素类型 + []）和数组变量名`int [] a;`

  2. 定义/创建数组：可以使用 new 运算符创建数组，且`必须指定数组的维:即数组的长度，它不要求是常量，可以是变量 可以是0 `！

     ```java
     int k=1;
     int[] a1 = {1, 2, 3};        // ok ：创建数组的同时，时赋予数组初值：简写方式：不需要 new 运算符
     int[][] b1 = {{1}, {3}};     // ok : 简写方式
     int[] b2 = new int[]{1, 2};  // ok : 创建数组的同时，初始化数组
     int[] a2 = new int[k];       // ok : k 可以是 常量、变量、0 ：Java中允许数组的长度为 0
     
     a1 = new int[]{1};           // ok : 初始化一个 匿名数组，这种形式可以在 不创建新变量的情况下 重新初始化 一个旧的数组！
     
     int[] a4 = new int[];        // Error: java: 缺少数组维
     ```

  3. 数组（默认）初始化

     ```java
     /*
     创建 一个 数字 数组：所有元素初始化为 0				    例如：int [] a=new int[n];
     创建 一个 boolean 数组：所有元素会被初始化为 false  		 例如：boolean [] a=new boolean[n];
     创建 一个 对象 数组：所有对象元素会被初始化为 特殊值 null   例如：Object [] a=new Object[n];
     */
     ```

#### equals、hashCode关系

```java
/*
如果重新定义了 equals 就要重新定义 hashCode方法，保证 equals 方法与 hashCode 方法的语义相同！
即：如果 a.equals.(b) = true 则 a.hashCode() = b.hashCode() 成立！

hashCode 方法通常用于 将元素存入 hash 表时，计算元素的 hash码 ，从而获取 元素所在桶的位置（将hash码 对 桶的总数 取余）！一个好的hash函数，可以将元素散步到整个hash表中！当存入到 hash 表的元素增多时，会出现多个元素分布到同一个桶中，则认为这些元素可能相等，对于未hash到同一个桶中的元素，则肯定不相等！为保证同一个桶中的元素都不相等，则在存入元素的时候，需要利用 equals 方法将待存入的元素与桶内的元素相比较，若 equals 方法结果相等，则表示桶内的元素存在与待插入的元素相等！
*/
```

#### 包装类 缓存范围

```xml
<!--
包装类：
	自动装箱规范要求：boolean、ASSIC码值小于 127 的char、值介于-128~127的 int 和 short 都被包装到固定的对象中，-128~127 的 byte！
	double、long 的包装类没有缓存！ 

【补充】在 java 中 byte 类型是 8 位带符号的二进制数，取值范围为 [ -128 ~ 127 ] ，即占 一个 字节8位 ！

Java 在包装类的内部都为上述的基本类型设置了相应的缓存！
	在自动装箱时，如果值的范围符合上述要求，则都会返回固定的对象！
	但是：通过 new 创建的 包装类，则不会使用到 缓存的对象！

【补充】
boolean  --	Boolean      1 字节 [或者] 4 字节
byte     -- Byte         1 字节 8 bit
char     -- Character    2 字节 16 bit
short    -- Short        2 字节 16 bit
int      -- Integer      4 字节 32 bit
float    -- Float        4 字节 32 bit
double   -- Double       8 字节 64 bit
long     -- Long         8 字节 64 bit

【解析 boolean 在 java 中是多少字节！】
1、在 javase 的官方文档中提到：boolean 基本类型 的大小 并没有精确定义！
2、在 java 虚拟机规范中提到：java 虚拟机中没有任何可供 boolean 值 专用的字节码指令，java语言表达式 所操作的 boolean 值，在编译之后都会使用 java 虚拟机中的 int 数据类型代替！所以此时 boolean 类型为 4 个字节！
3、java 虚拟机直接支持 boolean 类型的数组，JVM 可以通过 指令创建 boolean 数组，并且该数组的 访问和修改 与 byte类型数组 共用 boload 和 bastore 指令，由于 byte 是一个字节，所以此时 boolean 类型为 1 个字节！
【总结】：boolean 类型在数组情况下 为 1个字节，在单个 boolean 时，是4个字节！
-->
```



```java
// 源码：这些缓存类都是内部类！
// Double、Long 没有内部缓存

// Integer 缓存
private static class IntegerCache {
    static final int low = -128;
    static final int high;
    static final Integer cache[];

    static {
        // high value may be configured by property
        int h = 127;
        /*
             ... ...
             通过在虚拟机中配置 high 的大小，设置 h
             VM.getSavedProperty("java.lang.Integer.IntegerCache.high")
            */
        high = h;

        cache = new Integer[(high - low) + 1];
        int j = low;
        for(int k = 0; k < cache.length; k++)
            cache[k] = new Integer(j++);
        // range [-128, 127] must be interned (JLS7 5.1.7)
        assert IntegerCache.high >= 127;
    }
    private IntegerCache() {}
}

// Character 缓存
private static class CharacterCache {
    private CharacterCache(){}

    static final Character cache[] = new Character[127 + 1];

    static {
        for (int i = 0; i < cache.length; i++)
            // 通过 ASCII 转换 ：将 0~127 的整型数转换为 char 字符
            cache[i] = new Character((char)i);
    }
}

// Short 缓存
private static class ShortCache {
    private ShortCache(){}
    static final Short cache[] = new Short[-(-128) + 127 + 1];
    static {
        for(int i = 0; i < cache.length; i++)
            cache[i] = new Short((short)(i - 128));
    }
}

// Boolean 固定对象
public static final Boolean TRUE = new Boolean(true);
public static final Boolean FALSE = new Boolean(false);

// Byte 缓存
private static class ByteCache {
    private ByteCache(){}

    static final Byte cache[] = new Byte[-(-128) + 127 + 1];

    static {
        for(int i = 0; i < cache.length; i++)
            cache[i] = new Byte((byte)(i - 128));
    }
}
```

+ 相关知识：

  [Java 中的 boolean 占用几个字节](https://mp.weixin.qq.com/s/FLY__y7qHmovG_ACc1ekEg)

  [JAVA包装类的缓存范围](https://www.cnblogs.com/javatech/p/3650460.html)

  [java中int与char之间的互相转化](https://www.cnblogs.com/liulaolaiu/p/11744410.html)

#### 异常 与 捕获异常

##### try、catch、finally

+ 必须在 try 之后添加 catch 或者 finally 块
+ try 后面 可同时添加 catch 和 finally ，但是至少要 有其中一个 块
+ 如果同时使用 catch 与 finally ，则 必须要在 catch 之后使用 finally
+ 可以 使用多个 catch 块，但每次只会匹配一个 catch，匹配完成之后 就会退出 当前的 try-catch-finally
+ try-catch-finally 可嵌套
+ 可在 try-catch-finally中通过 throw 重新抛出异常！
+ 如果添加了 finally 块，则以下情况将不会执行 finally
  1. JVM过早终止：在 try 或者 catch 中调用 Ststem.exit(0)：它将终止整个JVM
  2. finally 中抛出一个为处理异常！
  3. 计算机 断电、失火、遭遇病毒攻击

##### try、catch、finally 与 return

+ 详见：[finally语句到底是在return之前还是之后执行](https://mp.weixin.qq.com/s/5Ez6_I4T4eOOf3SRMk_cSA)

  ```java
  /*
  
  【try代码块中无异常时】
  1、 若try中有return，finally中没有return，则，finally代码块在 return 执行之后，最终返回之前执行！
  （如果 try代码块中为 return method(); 则会等该 method() 方法执行后，最终执行 方法中的return前，执行finally ）
  
  2、 若try中有return，finally中没有return，则，原来的返回值可能因为 finally 中的修改而改变，也可能不会改变！这关系到 java 中的对象传递始终是值传递的概念！【可以通过引用修改对象的内容，但是不能通过改变对象引用，达到修改对象的目的】若finally代码块中，对返回的对象，通过引用修改了对象的内容，则返回值受影响，若只是改变原引用所引用的对象，那对返回类型毫无影响！
  
  3、 若 try、finally代码块中都有 return 时，finally 中的 return 将会覆盖 try 中的 return。
  
  【try代码块中出现异常】
  1、 try 在 return 之前发生异常并杯捕获时，将不会再执行 try 中的 return
  
  2、 若 catch、finally 中都存在 return，则这与上述 未发生异常时 try、finally 中都有 return 执行过程一样！
  */
  ```


##### java 异常

+ [Java 异常面试题](https://blog.csdn.net/qq_36523638/article/details/79363652?utm_source=copy)

###### Java 异常结构

```java
/*
所有异常都是由 Throwable  继承而来
  Throwable
  |
  | ------> Error	：非受查异常
  |
  | ------> Exception
  			   |
  			   | ------> RuntimeException	：非受查异常
  			   |
  			   | ------> IOException		：受查异常

Java 将 异常分为 受查异常 和 非受查异常：
	【 非受查异常 】不强制在程序中 捕获或抛出 的异常，通常认为是由 程序产生的、人为可控的（RuntimeException）以及 不可控的（Error）。人为可控的，例如因为 null 抛出的 空指针异常！不可控的，例如因为内存泄漏导致的 OOM 或者 StackOverFlow 等错误！
	【 受查异常 】程序本身没有问题，但是出现了像 I/O 错误这类问题产生的异常！编译器会检查是否为 程序中出现的所有 受查异常 提供异常处理器（捕获异常、抛出异常等处理）。
	
	受查异常：这类异常，并不是由于程序的错误导致的异常，而是通常由于系统资源请求出现错误导致的异常，为保证程序的健壮性，将这类异常声明为 受查异常，强制程序员对该类异常进行处理，避免了程序因这类异常而中断运行，同时也避免了因为请求资源失败，而出现资源未关闭等问题的出现！ 
*/
```

######  NullPointerException  VS  ArrayIndexOutOfBoundException 

```java
/*
答：
	NullPointerException 是 空指针异常，就是 NPE 问题，这是由于程序代码对 null 处理不仔细引起的，这种异常人为可控！
	ArrayIndexOutOfBoundException 是 java 中数组越界的引发的异常，同样是由于 代码处理不完善引起，这中异常人为可控！java 中 数组有固定的大小限制，而c里面的数组没有大小限制，所以c里面不会抛出类似于 ArrayIndexOutOfBoundException 的异常。
	
【总结】 共同点：都继承于 RuntimeException，属于非受查异常，可以避免！
*/
```

###### 出现 OOM 如何搞定

```java
/*
涉及 内存泄漏、JVM调优、调试方面的技术 ———— 面试加分！
*/
```

######  空引用异常的判断

```xml
<!--
null 是 Java 中的一个关键字，不能写成 NULL、Null，只能是 null
null 是所有引用类型的默认值，他表示该引用没有指向任何一个实际的对象！
当通过该 引用 调用类的方法时，就会出现 NullPointerException 异常 —— NPE 问题。
    
使用 对象前首先确保该对象是否为 null ，做出必要的处理，否则程序直接崩溃！
    1、 Objects.isNull(object);
    2、 在 Spring 中使用 验证注解 @NonNull 标记该属性不可为 null
如果程序中出现 NullPointerException 则应该从打印的堆栈结果中定位到出现该异常的地方！
	3、java 8 为了更好的解决和避免常见的 NPE 问题，引入了Optional 类，从而优雅的接解决了 NEP 问题！它并不是对 null 关键字的代替，而是提供一种更加优雅的 判断 null 的方式！
	Optional 类 允许元素的值为 null ，同时可以优雅的解决 NPE 问题，它避免了 嵌套大量的 if-else 代码块！

【注意】
	Optional 是一个 final 类，没有实现任何接口，Optional 不能序列化（因为没有实现 Serializable 接口），不能作为类的字段，所以，当我们在利用该类包装定义类的属性时，如果我们定义的类有序列化的要求，会出现问题！ 
-->
```

+ [Optional 解决 NPE 问题](https://blog.csdn.net/qq_27276045/article/details/102689086)

