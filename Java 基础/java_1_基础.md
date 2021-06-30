### java_基础

#### 面向对象

##### 面向对象 vs 面向过程

+ **面向过程**

  ```
  分析 实现某个需求所需要的完整步骤，将各个步骤函数化，再依次调用这些函数完成需求！
  面向过程，针对的是 完成这个需求 的过程！
  ```

  *缺点：*

  ```
   · 代码的可复用性差：相似的代码总是会被 复制粘贴！
   · 软件的可维护性差，并且不能满足用户需求的变化：面向过程往往采用 结构化 方法开发，使得功能一层一层的往下切分，当需求变化时，代码的层级之间变化较大!
  ```

+ **面向对象**

  ```
  根据事物的总体特征，对其进行 功能划分，再将所有的 功能和数据 组织在一起，形成一个整体，抽象为一个对象，抽象成对象 并不是为了完成某个 行为，它描述的是 事物总体的行为特征！
  ```

  *特性：*

  ```
   · （抽象）封装、继承、多态
   	1. 通过 封装 保证了 数据的安全性：访问控制修饰符的存在，使得 属性私有，方法公有！
   	2. 通过 继承 实现了 代码的 可复用性，子类拥有父类的非私有方法和属性，并且能够根据自身特点重写方法！
   	3. 通过 多态 实现了 代码的 高度灵活性，能够编写出更加通用的代码，以适应需求的不断变化！
   	4. 正是由于对象的三大特性，实现了高内聚、低耦合 的系统结构，使得系统结构更加灵活，可维护性更强！
  ```

+ *缺点：*

  ```
  为了实现相同需求，需要维护对象的状态，性能比面向过程低！
  ```

##### Java vs C++

1. *面向对象 & 面向对象（兼容面向过程）*

   ```
    · Java 存粹是面向对象的语言，所有类都继承于 Object；
    · 而 C++ 为了兼容 c ，也支持面向过程
   ```

2. *解释型 & 编译型*

   ```
    · java 是解释型与编译型并存的语言，源程序生成的字节码文件，会被 jvm 解释执行；并且，在运行期，对于“热点代码”，JVM 会将其编译为 本地机器码，提高了热点代码的执行效率，热点代码就不需要被频繁的解释！（解释过程也是耗时的）
    	热点代码：某个代码块、方法 执行的特别频繁！
    	
    · C++ 是编译型语言，源程序经过编译和链接后 直接生成可执行的 二进制代码（本地机器码），因此 C++ 效率更高！
   ```

3. *平台无关 & 平台相关*

   ```
    · Java 虚拟机实现了跨平台的特性，编译后的 Java 字节码文件，可以在任意部署了 JVM 的平台上执行，JVM 会将字节码文件解释成操作系统相关的机器码！
    	每个操作系统都有自己相关的 JVM，而 字节码文件 格式是统一的！
    
    · C++ 与平台相关，主要在于，编译后的 C++ 直接生成可执行的 机器码，而不同的操作系统拥有自己的指令集，编译器是与指令集相关的，因此 编译后的机器码 也是和 指令集相关的，也就是平台相关！
   ```

   ```
    · 源程序 到 可执行程序：编辑 -> 编译 -> 汇编 -> 链接
    	编译最重要，可分为6个部分：词法分析、语法分析、语义分析、中间代码生成、目标代码生成 和 优化！
    	链接：静态链接、动态链接
    	
    · C++ 的静态链接、动态链接
    	静态链接：在编译期为目标程序 连接 所要依赖的库函数，并一起打包到可执行文件中。如果多个目标代码 依赖于 同一个库函数，则分别为这些目标程序 拷贝一份 函数放在内存中形成 静态库！
    	动态链接：在程序运行期，链接 所依赖的的库函数，如果多个目标程序 依赖于 同一个库函数，那么编译后内存中只会有 1 份库函数，不同的库函数就构成了 动态库（也称为 共享库）！
    	
    · Java 的静态链接、动态链接
    	静态链接：在类加载时，将字节码文件中的符号引用 转换为 直接引用！
    	动态链接：在程序运行期，将字节码文件中的符号引用 转换为 直接引用！
   ```

   + [C++ 静态链接、动态链接](https://blog.csdn.net/zstu_zlj/article/details/78951630)

4. *Java中的引用 & C++ 中的指针与引用*

   ```
    · Java 没有指针，它只有对象的引用，可以理解为 C++ 中安全的指针，虽然对象的引用就是对象的地址，但是它没有操作内存地址的特性，也就不会直接操作内存，因此是安全的！
    
    · c++ 和 c都具有指针，c++ 的引用表示的是变量的别名！使用 指针可能会引起系统安全问题（可以直接操作内存地址？）
   ```

5. *内存管理*

   ```
    · java 有 垃圾回收器 自动回收垃圾，并且不需要显示管理空间！
    
    · C++ 需要开发人员自己管理内存分配（包括资源的申请和释放），C++会在析构函数中 释放资源，而 java 中只有拯救自己的 finalize() 方法！
   ```

6. *继承*

   ```
   Java 只支持单继承！
   C++ 支持多重继承，进而会引出 虚基类！
   ```

7. *运算符重载*

   ```
   Java 不支持运算符重载！
   C++ 支持运算符重载！
   ```

##### 特性

###### 抽象

+ 定义

  ```
   · 概括出 事物的 总体特征，抽取出事物的 公共属性、行为，将这些 行为、属性 组织在一起，形成一个对象！
  ```

###### 封装

+ 定义

  ```
   · 将描述一个对象的 行为和属性 封装成一个类，并隐藏对象的属性 和 方法细节，仅仅对外提供接口！
   	通过访问控制修饰符，私有化属性，公有化方法，保证数据安全性！
  ```

###### 继承

+ 定义

  ```
   · 描述的是 类的特殊与一般 的关系！特殊的类为子类，一般的类为父类，子类拥有父类所有的非私有 属性和行为，并在父类的基础上，添加新的 属性和方法，以满足某个特定的需求！
   
   · 继承 实现了 代码的复用性。
  ```

+ **Java 继承**

  + Java 不支持多重继承，即一个类只能显示继承一个父类！（Object 是所有类的超类）

  + *不可被继承*

    ```
     · Java 构造方法 不可被继承！
     · 被 private 修饰的方法和属性 不可被继承！
     · 如果 构造方法都被声明为 private，那么 该类 不可被继承！
     · 被 final 修饰的类不可被继承！
     · 方法上的 Synchronized 锁不会被继承！
    ```

  + *类初始化顺序*

    ```
     1、初始化 父类中的 静态成员变量 和 静态代码块
     2、初始化 子类中的 静态成员变量 和 静态代码块
     3、初始化 父类中的 普通成员变量 和 代码块，再执行 父类的构造方法。
    	如果父类构造方法为 private 则初始化子类的时候不可以被执行。
     4、初始化 子类中的 普通成员变量 和 代码块，再执行 子类的构造方法
    ```

  + *子类特点*

    ```
     1、子类拥有对父类 非private 的属性和方法
     2、子类可以添加自己的方法和属性，即对父类进行扩展
     3、子类可以重新定义父类的方法，即方法的覆盖和重写
    ```

+ **重载和重写**
  + 方法签名：方法名称 + 参数列表

  + *重载*

    ```
     · 同一个类中，名称相同，但参数列表不同的方法，就互为重载方法！
     · 重载描述的是 行为相似但具体实现不同 的场景！
     · 对于重载的方法，其返回类型不作为判断标准，即不能有两个 方法签名相同 但 返回类型不同 的方法出现在同一个类中！
    ```

  + *重写/覆盖*

    ```
     · 子类中存在 与 父类方法 相同签名的方法，那就称为 子类的方法 重写/覆盖了 父类的方法！
     · 重写描述的是，父类的方法并没有具体实现 或者 父类的方法并不适用于子类 的场景！
     · 对于重写而言，返回类型不是签名的一部分，但是在覆盖方法时，一定要保证返回类型的兼容性，即子类的返回类型应为超类该方法返回类型的子类型！ （可协变性）
    ```

###### 多态

+ 多态

  ```
   · 多态 指 对象同时具有多种状态，但在特定的情况下，表现为某种特定的状态。
   	这使得，在编译期无法确定 对象引用 的具体类型，也 无法确定其调用的具体方法！而是在运行期，才能确定其引用的具体对象，以及所要调用具体方法。
   
   · 使得 编写的代码更加的通用，更加的灵活，以应对不同需求的变化！
  ```

+ *三种多态*

  ```
   1、继承
   2、子类 覆盖 父类
   3、父类对象 引用 子类对象！
   	Object str=new String();
  ```

+ 注意：

  **重载 与 多态无关**，重载是同一个类下的行为，面向过程的编程语言中就支持了 重载；在面向对象编程中，对于重载而言，对象状态是确定的！

#### 抽象类 vs 接口

##### 抽象类

+ 定义

  ```
   · 抽象类：abstract 关键字修饰
   	抽象类 具备高度的通用性，它会声明 抽象行为，行为的具体实现交由 子类 根据不同的应用场景完成！
   
   · 相对普通类而言，抽象类没有自己的 实例对象，它是一种 通用性 规范！
  ```

+ 注意

  ```
  1、包含一个或者多个抽象方法的类 必须 被声明为抽象类！
  2、抽象类除了声明抽象方法外，还可以定义 具体的属性 和 方法！
  3、类即使不包含抽象方法，也可以将其声明为抽象类！
  4、抽象类不能被实例化，可以定义一个抽象类的 对象变量，但是它只能引用非抽象子类的对象！
  ```

##### 接口

+ 定义

  ```
   · 接口：interface 关键字修饰
   	接口技术主要用于描述类的功能，而并不给出每个功能的具体实现，其作用抽象类相似！
  ```

+ 注意

  ```
  1、一个类允许实现多个接口！也可以被子类扩展！

  2、接口中的 域 必须是 static final 的常量，不能是实例域！
  3、jdk 8 之前，接口中不能实现方法，也不能包含静态方法，jdk 8 开始，可以在 接口中实现默认方法和静态方法！
  4、接口中的方法自动会被设置成 public，接口中的常量自动被设为 public static final

  5、接口不是类，不能被实例化，可以定义一个 接口变量，但它只能引用其实现类的对象！
  ```

##### 接口 vs 抽象类

+ 区别

  ```
   1. 抽象类 只支持 单继承，而 一个类可以实现多个接口，通过接口达到 多重继承的效果！
   2. 抽象类 具备 普通类的所有特性，甚至可以没有抽象方法！
  	接口中 只能包含 静态域，并且在 jdk1.8 之前，接口不能实现任何的方法，只能提供方法的声明！而在 jdk1.8 之后，才允许在接口中定义 默认方法 和 静态方法！
  ```

+ 如何选择

  ```
  根据需求，如果只需要定义一些 抽象方法，而不需要其他欸外的具体方法或者变量，则可以选择接口！
  如果需要 定义实例域 或者 实例方法，那么需要使用 抽象类！
  ```

#### 泛型

+ 详见 ： 《 java_泛型 》章节

+ [泛型详解](https://blog.csdn.net/stypace/article/details/42102567)
+ [泛型面试题汇总](https://www.cnblogs.com/huajiezh/p/6411123.html)

#### java 元注解

##### 注解

+ 注解定义

  ```
   · 注解 是 java 的元数据，用于标记 程序源代码！
   	注解可以在 编译期、类加载时期、运行期 被读取，并在不改变源程序逻辑的情况下，为源程序添加补充的信息，可以用于 代替复杂的配置文件，简化开发！
  ```

+ 各个时期示例：

  ```
   1、向编译器传递信息 —— 使用注解，编译器可以检测错误或者抑制警告
   2、编译时 和 部署时 处理 —— 软件工具可以处理注解并生成代码、生成配置配置文件等！
   3、运行时 处理 —— 可以在运行时检查注解，以自定义程序的行为！
  ```

##### java 中有哪几种元注解

​		 java 中提供了 4 个元注解，元注解的作用是负责注解 其他的注解！

+ **@Target ：注解的作用范围**

  ```java
  // 描述 注解 所注释的目标的范围
  @Target(ElementType.ANNOTATION_TYPE)
  @Documented
  @Retention(RetentionPolicy.RUNTIME)
  public @interface Target {
      /**
       * @return an array of the kinds of elements an annotation type can be applied to
       */
      ElementType[] value();
  }

  public enum ElementType {
      // 类、接口（包括注解类型）、枚举
      TYPE,
      // 域（包括 枚举、常量）
       FIELD,
      // 方法
      METHOD,
      // 参数
      PARAMETER,
      // 构造器
      CONSTRUCTOR,
      // 本地变量
      LOCAL_VARIABLE,
      // 注解类型
      ANNOTATION_TYPE,
      // 包
      PACKAGE,
      // 类型参数
      TYPE_PARAMETER,
      
      TYPE_USE,	
      MODULE
  }

  // 示例：
  @Target({ElementType.TYPE, ElementType.METHOD})
  public @interface MyAnn{ 
      // @MyAnn 注解只能标注在 类、接口、枚举、方法上
  }
  ```

+ **@Retention（保留策略：生效的时机）**

  ```java
  // 定义了该注解保留时间的长短
  public @interface Retention {
      // @return the retention policy
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

+ **@Documented：注解会被文档化**

  ```java
  /*
   · 用于描述其他类型的 annotation 应该被作为 被标注的程序的 公共API，注解会被 javadoc 此类的工具文档化
   · @Documneted 是一个标记注解，没有成员
  */
  public @interface Documented { }
  ```

+ **@Inherited：注解可以被 被标记的类继承**

  ```java
  /*
   · 阐述某个被标记的类型是被继承的
   · 如果 使用 一个被 @Inherited 标注的 注解 标注一个类，则说明，该 注解 会被用于该 类的子类！ 
   · @Inherited 是一个标记注解，没有成员
  */
  public @interface Inherited {}
  ```

##### 自定义注解

```java
// 注解使用 @interface ，而不是 class、enum、interface
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

  3. **数组（默认）初始化**

     + 创建 一个 **数值 数组：所有元素初始化为 0**

       `例如：int [] a=new int[n];`

     + 创建 一个 **boolean 数组：所有元素会被初始化为 false**

       `例如：boolean [] a=new boolean[n];`

     + 创建 一个 **对象 数组：所有对象元素会被初始化为 特殊值 null**

       `例如：Object [] a=new Object[n];`

#### equals & hashcode

+ 介绍

  equals 和 hashcode 一般和 hash表 一起出现！

  hashcode：用于计算 元素的 hash值：它关系到 hash碰撞的几率！

  equals：当出现 hash碰撞时，equals 方法用于判断存在 hash碰撞的元素是否为同一元素！

+ 重写 equals、hashcode

  ```java
  // Object 类下的 equals 方法：比较的是 两个对象是否为同一个引用
  public boolean equals(Object obj) {
      return (this == obj);
  }
  // Object 类下的 hashcode 方法
  @HotSpotIntrinsicCandidate
  public native int hashCode();

  /*
  hashcode 方法定义在 Object 类中，因此每个对象都有一个默认的 hash码，默认为 该对象的储存地址！然而对于字符串而言，它的哈希码是由其内容导出的！即：字符串内容相同的两个字符串对象其hash码相同。
  hashCode 方法应该返回一个整型数值（可以为负数），并合理地组合实例域的 哈希码 ，以便能够让各个不同的对象产生的 哈希码 更加均匀的分布，减少 hash 冲突！

  建议使用 null 安全的 Objects.equals、Objects.hashCode！
  Objects.hash 是一个可变参数的方法，它会根据传入的参数，对各个参数调用 Objects.hashCode 方法，并组合这些 哈希码 ！
  如果使用数组类型，可以使用静态的 Arrays.hashCode 方法计算该数组的 hash码，该hash码由各个元素的 hash码组成。

  如果重新定义了equals方法，就必须重新定义hashcode方法，以便开发人员可以将对象插入到散列表中！
  equals 方法的语意 和 hashCode 方法的语意 必须一致！
  	如果 x.equals(y) 为 true 那么 x.hashCode()==y.hashCode() 为 true ！
  */
  ```

#### 包装类 缓存

+ 相关知识：

  [Java 中的 boolean 占用几个字节](https://mp.weixin.qq.com/s/FLY__y7qHmovG_ACc1ekEg)

  [JAVA包装类的缓存范围](https://www.cnblogs.com/javatech/p/3650460.html)

  [java中int与char之间的互相转化](https://www.cnblogs.com/liulaolaiu/p/11744410.html)

##### 概述

+ **包装类缓存**

  根据自动装箱规范 规定：**boolean**、**byte**、**ASSIC码值小于 127 的 char**、**值介于 -128~127 的 int、short 在自动装箱时，都会被自动包装到固定的对象中，缓存对象中**！

  `在 java 中 byte(字节) 类型是 8位带符号的二进制数，取值范围为 -128 ~ 127！`

  **float、double、long 包装类没有缓存！**

+ 注意

  *只有在自动装箱时才会使用到 缓存，而 通过 new 关键字创建的包装类对象，不会使用到缓存对象！*

+ 8 种基本类型 vs 包装类

  | 基本数据类型  | 包装类       | 大小          |
  | ------- | --------- | ----------- |
  | boolean | Boolean   | 1 字节  、4 字节 |
  | byte    | Byte      | 1 字节        |
  | char    | Character | 2 字节        |
  | short   | Short     | 2 字节        |
  | int     | Integer   | 4 字节        |
  | float   | Float     | 4 字节（无缓存）   |
  | double  | Double    | 8 字节（无缓存）   |
  | long    | Long      | 8 字节（无缓存）   |

  *boolean 大小会根据使用场景确定：*

  ```
   · Java 中的 boolean 类型其实是没有固定大小的！
   
   · Java 虚拟机规范中提到，Java 虚拟机中没有任何可共 boolean 类型的专用的字节码指令，在 java 语言表达式中 操作的 boolean 类型，在编译之后都会使用 Java虚拟机 中的 int 类型代替，此时，意味着 boolean 为 4 个字节！
   · Java 虚拟机直接支持 boolean 类型的数组，JVM 通过指令创建 boolean 数组，并且该数组的访问和修改 与 byte类型的数组 通用 boload 和 bastore 指令，由于 byte 是一个字节的，此时，意味着 boolean 为 1 个字节！
   
   · 总结：boolean 类型在数组情况下是1个字节，单个 boolean 操作时，是 4 个字节！
  ```

##### 源码

+ 源码：这些缓存类都是内部类！
+ **Float、Double、Long 没有内部缓存**

###### 内部缓存 Boolean

```java
// Boolean 固定对象
public static final Boolean TRUE = new Boolean(true);
public static final Boolean FALSE = new Boolean(false);
```

###### 内部缓存类 Byte

```java
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

###### 内部缓存类 Character

```java
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
```

###### 内部缓存类 Short

```java
// Short 缓存
private static class ShortCache {
    private ShortCache(){}
    static final Short cache[] = new Short[-(-128) + 127 + 1];
    static {
        for(int i = 0; i < cache.length; i++)
            cache[i] = new Short((short)(i - 128));
    }
}
```

###### 内部缓存类 Integer

```java
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
```

#### 异常

##### 异常捕获规则

+ 必须在 try 之后添加 catch 或者 finally 块
+ try 后面 可同时添加 catch 和 finally ，但是至少要 有其中一个 块
+ 如果同时使用 catch 与 finally ，则 必须要在 catch 之后使用 finally
+ 可以 使用多个 catch 块，但每次只会匹配一个 catch，匹配完成之后 就会退出 当前的 try-catch-finally
+ try-catch-finally 可嵌套
+ 可在 try-catch-finally中通过 throw 重新抛出异常！
+ 如果添加了 finally 块，则**以下情况将不会执行 finally**
  1. **JVM过早终止**：*在 try 或者 catch 中调用 Ststem.exit(0)：它将终止整个JVM*
  2. **finally 中抛出一个未处理异常**！
  3. **计算机 断电、失火、遭遇病毒攻击**

##### 异常捕获 与 return

+ 场景1：*try 中 无异常*

  1. **try 中 return，finally 中没有 return**

     ```
      · finally 代码块 会在 return 执行之后，最终返回之前执行！
      	例如：try 中执行 return method()，那么 finally 会在 menthod() 方法执行之后，最终执行 方法中的 return 之前 被执行！
      	
      · 因此：try 中的返回值，可能会因为 finally 中的操作而被修改！
      	由于 Java 是 按值传递 的（传递的是对象的引用）。因此，如果在 finally 中，通过引用修改了对象的内容，那么返回值受影响，如果仅仅是改变 原引用所引用的对象，那么返回值无影响！
     ```

  2. **try 和 finally 代码块中 都 return**

     ```
      · finally 的返回值会覆盖 try 中的返回值！
     ```

+ 场景2：*try 中 出现异常*

  1. **如果存在 catch，则 catch 中 return 的执行情况 与 未发生异常时 try 中 return 的执行情况一致！**
  2. **如果不存在 catch，直接执行 finally**

+ [详见：finally语句到底是在return之前还是之后执行](https://mp.weixin.qq.com/s/5Ez6_I4T4eOOf3SRMk_cSA)

##### Java 异常

1. **异常层次结构**

   ```
    · 所有异常都是由 Throwable  继承而来，Java 异常被分为 受查异常 和 非受查异常！

     Throwable
     |
     | ------> Error	：非受查异常
     |
     | ------> Exception
     			   |
     			   | ------> RuntimeException	：非受查异常
     			   |
     			   | ------> IOException		：受查异常
   ```

2. **非受查异常**

   ```
    · 不强制在 程序 中捕获或者抛出的异常，通常认为是 由程序产生的、人为可控的异常，或者是不可控的 错误！
    
    · 可控异常：RuntimeException
    	例如：因为 null 抛出的 空指针异常！

    · 不可控错误：Error
    	例如：因为内存泄漏导致的 OOM，或者是 栈大小不足产生的 StackOverFlowError 等！
   ```

3. **受查异常**

   ```
    · 程序本身没有问题，通常由于 系统资源请求出现错误导致的异常！（例如：I/O错误）
    	编译器会检查是否为 程序中所出现的 受查异常 提供异常处理器，强制开发人员捕获异常并处理！

    · 这类异常，并不是由于程序错误产生的，为了保证程序的健壮性，将这类异常声明为 受查异常，强制开发人员对其进行处理，避免程序因为这类异常而中断运行，也避免了因为请求资源失败，而出现资源未关闭的问题的出现！
   ```

+ [Java 异常面试题](https://blog.csdn.net/qq_36523638/article/details/79363652?utm_source=copy)

##### NPE

+ **NPE**

  ```
   · null 是 Java 中的一个关键字，不能写成 NULL、Null，只能是 null
   	它是所有引用类型的 初始零值，表示 该引用没有执行任何一个实际的对象！
   	当通过该引用调用 对象的方法时，就会抛出 NPE
  ```

+ **NPE 判断**

  使用对象之前应该确保该对象是否为 null，做出必要的处理，否则程序会崩溃！

  ```
   · 判断 null 的方法
   	1. Objects.isNull(object);
   	2. 在 Spring 中，使用 JSR303 校验：@NotNull("...")
   	3. Optional 类：解决 NPE 问题
   	
   · Optional 类
   	Optional 是一个 final 类，没有实现任何接口，Optional 不能序列化（因为没有实现 Serializable 接口），不能作为类的字段，所以，当我们在利用该类包装定义类的属性时，如果我们定义的类有序列化的要求，会出现问题！
   	它是 Java 8 中的新特性，用来解决 NPE 问题，它不是对 null 关键字的代替，而是一种优雅的处理 null 的方式！它允许 元素的值为 null，能够避免嵌套大量的 if-else 代码块！ 
  ```

  + [Optional 解决 NPE 问题](https://blog.csdn.net/qq_27276045/article/details/102689086)

+ *NullPointerException*  vs  *ArrayIndexOutOfBoundException*

  两者都继承于 RuntimeException，**都属于非受查异常**，都可以避免！

  ```
   · NullPointerException 是 空指针异常，就是 NPE 问题
  	这是由于程序代码对 null 处理不仔细引起的，这种异常人为可控！

   · ArrayIndexOutOfBoundException 是 java 中数组越界的引发的异常
   	同样是由于 代码处理不完善引起，这种异常人为可控！
   	
   	java 中 数组有固定的大小限制，而c里面的数组没有大小限制，所以c里面不会抛出类似于 ArrayIndexOutOfBoundException 的异常。
  ```

#### JDK & JRE

+ JRE： Java Runtime Environment

  ```
  JRE顾名思义是java运行时环境，包含了java虚拟机，java基础类库。是使用java语言编写的程序运行所需要的软件环境，是提供给想运行java程序的用户使用的。
  ```

+ JDK：Java Development Kit 

  ```
  JDK顾名思义是java开发工具包，是程序员使用java语言编写java程序所需的开发工具包，是提供给程序员使用的。

  JDK包含了JRE，同时还包含了编译java源码的编译器javac，还包含了很多java程序调试和分析的工具：jconsole，jvisualvm等工具软件，还包含了java程序编写所需的文档和demo例子程序。
  ```
