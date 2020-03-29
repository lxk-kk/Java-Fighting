#### Java 8 新特性

##### Lambda

1. 介绍

   + Lambda 表达式是一个可传递的代码块，它允许把函数作为一个方法的参数，它是匿名内部类的简化！

2. 作用

   + 为 函数式接口（只有一个抽象方法的接口）提供一个具体实现了该函数式接口的对象。

3. 语法

   1. `( 参数 ) -> 表达式` 或者 `( 参数 ) -> { 代码块 }`
   2. 即使 lambda 表达式没有参数，也需要提供 参数的空括号：`() ->...`

4. 解析

   + lambda 表达式有三个部分：
     1. 一个代码块
     2. 参数
     3. 自由变量的值（指：非参数并且也不在代码块中定义的变量）
   + `代码块`与`自由变量值`称为`“闭包”`：lambda 表达式就是 闭包 ！

5. 规则：

   1. lambda 表达式中捕获的变量（自由变量）必须是`最终变量：被 final 修饰`

      ```java
      /*
      局部变量是储存在栈上的，而栈上的内容在当前线程执行完成之后就会被GC回收！
      
      lambda 表达式会被交给另一个线程执行，而不是其所在方法的执行线程，所以，当所在方法的执行线程执行完毕后，执行 lambda的线程 需要时使用到 lambda 外部的局部变量时会出错（因为该外部的局部变量已经被回收）！
      
      鉴于上述原因：会将该局部变量拷贝一份到 lambda 表达式内部，为了保证数据的一致性，强制使用 final 修饰！
      */
      ```

   2. 如果可以推到出一个 lambda 表达式的`参数类型`，则可以忽略其参数类型

      ```java
      Comparator<String> comp=(first,second)-> first.length()-second.length();
      // 等同于
      Compatator<String> comp=(String first,String second)-> first.length()-second.length();
      ```

   3. 无需指定 lambda 表达式的`返回类型`，lambda 表达式的返回值类型总是会由上下文推导得到。

      ```java
      (String first,String second)-> first.length()-second.length();
      // 该 lamdba 表达式可以在 需要 int类型结果的上下文中使用。
      ```

   4. lambda 的 `代码块 与 嵌套块?` 有相同的作用域时，同样适用` 命名冲突和遮蔽`的有关规定 ？

   5. lambda 表达式中声明一个与 局部变量 同名的`参数 或者 局部变量`是不合法的！

      ```java
      int test_1=0;
      int test_2=0;
      Comparator<String> comp=(test_1,second)->{
          String test_2="";
          return test_1.length()-second.length();
      }
      /*
      这里将会有两个错误：
      1. lambda 参数 test_1 与 局部变量 test_1 同名！
      2. lambda 内部的局部变量 test_2 与 局部变量 test_2 同名！
      */
      ```

   6. lambda 表达式 中不能拥有同名的局部变量！

   7. lambda 表达式中使用 this 关键字时，该 this 指的是 创建这个 lambda 表达式 的方法的this参数！

      ```java
      public class Main{
          public void test(){
              ActionListener listener = event ->{
                  System.out.println(this.toString());
                  // ...
              }
          }
      }
      //这里的 this 关键字 this 是 test 方法中的 隐式参数this，而不是 ActionListener 的 listener
      ```

6. 使用场景

   + lambda 表达式的重点就是`延迟执行、以后再执行`！

7. 注意

   + 不能将 lambda表达式 赋给类型为 Object 的变量，因为 Object 不是一个函数式接口！

   + 如果一个 lambda 表达式只在某些分支返回一个值，而在另一分支不返回值 —— 不合法！

     ```java
     (int x)->{ 
         if(x>0) 
             return 1;
     }
     // 不合法：只在 x>0 的分支具有返回值！上下文不能正确推导出 lambda的正确的返回类型（我的猜想）！
     ```

##### 方法引用

1. 介绍

   + 类似于 lambda 表达式，方法引用不能独立存在，总是会转换为函数式接口的实例！ 
   + lambda 需要重新定义 函数式接口的方法，而方法引用则是直接引用现有 `Java类 或者 对象实例` 的 `方法或者 构造器`，方法引用使得语言更加紧凑简洁，减少了代码的冗余。

2. 语法

   + 对象 :: 实例方法`object::instanceMethod`

     ```java
     // 方法引用等价于提供方法参数的 lambda 表达式。即：
         System.out::println;
         // 等价于
         x->System.out.println(x);
     ```

   + 类 :: 静态方法`Class::staticMethod`

     ```java
     // 方法引用等价于提供方法参数的 lambda 表达式。即：
     	Math::pow;
     	// 等价于
     	(x,y)->Math.pow(x,y);
     ```

   + 类 :: 实例方法`Class::instanceMethod`

     ```java
     // 该方法引用 使得 方法的第一个参数会成为方法的调用者！
     	String::compareToIgnoreCase;
     	// 等价于
     	(x,y)->x.compareToIgnoreCase(y);
     ```

3. 规则

   + 可以在 方法引用中使用`this、super`等关键字

     ```java
     this::equals;
     // 等价于
     x->this.equals(x);
     // this与super同样是针对 使用该函数式编程的类而言的！
     
     super::instanceMethod;
     // 使用 super 作为 目标（调用者），它会调用给定实例方法的超类版本！如下：
     
     class Greeter{
         public void greet(){
             System.out.println("Super Hello World");
         }
     }
     class TimedGreeter extends Greeter{
         public void greet(){
             Time t=new Timer(1000,super::greet());
             t.start();
         }
     }
     
     // TimedGreeter.greet方法开始执行时，会构造一个 Timer ，他会在 每次定时器滴答时执行super::greet方法，这个方法会调用 超类的 greet 方法！
     ```

4. 构造器引用

   + 它与方法引用类似，只不过 方法名 变为指定的`new`。例如：`Person::new`

   + 当有多个构造器时，使用哪个构造器，将由上下文决定！

   + 可以用`数组类型`建立 构造器引用

     ```java
     int[]::new;
     // 等价于
     x->new int[x];
     ```

   + java有一个限制，不能实例化`泛型的类型变量`与`泛型数组`。即：不能构造`泛型T`的数组、对象 以及` 泛型T`的class对象。

     ```markdown
     即：
         new T[n] 表达式会产生错误！
         new T(...) 表达式产生错误！
         T.class 表达式产生错误！
         因为由于 泛型的类型擦除 它会被修改为 new Object[n]、new Object(...)、Object.class
     解决办法（建议查找资料！）：
     	1. 使用 具体类的构造器引用
     		Person[]::new
     		Person::new
     	2. 使用传统的 反射
     		Array.newInstance ：构造泛型对象数组
     		Class.newInstance ：构造泛型对象
     ```

##### 接口的默认方法

1. 默认方法就是在接口中就已经实现的方法，是非抽象方法，它是接口方法的一个默认实现，必须使用`default`修饰符标记！

2. 在 Java 8 中可以把所有的接口方法都声明为 `默认方法`，这些默认实现什么都不做！这样的话，实现这个接口，就只需要`覆盖（重写）`真正想要实现的默认方法即可，并不需要实现所有的接口方法！

3. 默认方法的重要用法：`接口演化`。

   例如：Java 8 中为 `Collection `类添加了一个`默认方法 stream `方法获取`集合的 流对象`，如果`stream` 不是默认方法，那么所有 Java 8之前实现了 `Collection `接口的类都将无法运行，因为他们没有实现 `stream`方法！

4. 方法冲突问题：

   + 如果一个类实现了多个接口，并且这些接口都实现了`相同的默认方法(方法签名相同)`，则该类需要解决默认冲突问题！

     ```java
     // 如下 两个 接口均实现了默认方法：drink()
     public interface GreenTea{
         default void drink(){
             System.out.println("drink green tea!");
         }
     }
     public interface RedTea{
         default void drink(){
             System.out.println("drink red tea!");
         }
     }
     ```

     ```java
     // 现在 若有一个 类 同时实现这两个接口！
     public class KkTea implements GreenTea,RedTea{
     }
     // 编译Error：java: 类 KkTea 从类型 RedTea 和 GreenTea 中继承了 drink() 的不相关默认值
     //------------------------------------------------------------------------------------
     // 解决冲突：
     public class KkTea implements GreenTea,RedTea{
         // 方案1：子类中必须创建重写这个具有 相同方法签名的方法
         @Override
         public void drink(){
             System.out.println("drink my own tea!");
         }
         
         // 方案2：如果子类中需要 复用 某个接口的 默认方法：则通过 接口.super 的方法指定调用即可！
         @Override
         public void drink(){
             GreenTea.super.drink();
         }
     }
     ```

   + 如果 多个接口中实现了`相同方法名的默认方法`，但是`方法签名不同`，则子类不需要做处理，子类中这些方法共存，因为他们的方法签名不同（重载），本质上是不同的方法

   + 如果 多个接口中 存在`只有返回类型不同`的（默认）方法，则`编译报错`！

     ```java
     public interface interface_1 {
         default int print(){
             return 1;
         }
     }
     public interface interface_2 {
         default void print() {
         }
     }
     
     public class Test implements interface_1, interface_2 {
         public static void main(String[] args){
         }
     }
     // 编译Error：java: 类型test.interface_2和test.interface_1不兼容; 两者都定义了print(), 但却带有不相关的返回类型
     
     // 实现多个接口，若不同的接口中存在 仅返回值类型不同的方法（无论是默认的、抽象的还是两者混合的），都无法成功编译！应该通过设计避免这种情况的发生！但是这个规则，对于接口中的 静态方法 不适用！
     ```

##### 接口的静态方法

+ Java 8 中可以为接口定义一个或者多个 静态方法，其用法和 类的静态方法一样，它属于接口，不属于任何的实现类，该方法`不可被继承/实现`。

+ 使用时 `接口.staticMethod()`即可！


##### Steam API

1. 简介

   + Java 8 中新增加的包 ：`java.tuil.stream`，把真正的`函数式编程`风格引入到`Java`中，并且可以以一种`声明式`的方式处理数据。
   + `Stream`将`集合元素`看作是一种流，在`管道中传输`，并且可以在管道的节点上进行处理：筛选、排序、聚合等等。集合元素在管道种经过各种中间操作的处理，由最终的操作得到前面处理的结果！
   + Stream（流） 是一个来自数据源的元素队列，并且支持聚合操作！
     + 元素是特定类型的对象，形成一个队列。Java中的`Stream并不会储存元素，而是按需计算`！
     + 数据源：是流的来源：可以是 集合、数组、I/O Channel、产生器 generator 等。
     + 聚合操作：类似于 SQL 语句一样的操作：比如：filter、map、reduce、find、match、sorted等

2. Stream 特征：

   + Pipelining ：中间操作都会返回流对象本身。这样多个操作都可串联形成一个通道，如同流式风格。这种机制可以对各种操作进行优化，比如延迟执行和短路等！

   + 内部迭代：
     + 以前对 集合遍历 都是通过 Iterator 或者 For-Each的方式，显式地在集合外部进行迭代，称为外部迭代！
     + Stream 提供了内部迭代的方式，通过`访问者模式 Visitor `实现

3. Java 8 中，集合接口生成流对象的两个方法：

   1. collection.stream()：为集合创建`串行流`
   2. colleciton.parallelStream()：为集合创建`并行流`

4. 注意：

   Stream是`快速失败 fast-fail`的，相对于普通集合的快速失败，Stream会`将方法执行完毕，再抛出异常`，而普通集合如果检测到`ModCount!=ExceptModCount`就立刻抛出异常！

   + `快速失败：通过迭代器或者ForEach遍历集合时，如果遍历过程中对集合对象的内容进行 增、删则会抛出 Concurrent Modification Exception 异常！`

     ```java
     /*
     如何解决 快速失败！
     1、使用 Iterator 迭代器，iterator 迭代器中的 remove 方法将不会产生快速失败，并且修改后的结果在后续遍历中可见！
     2、使用简单的 for 循环！
     */
     ```

   + `安全失败：出现修改后，不会抛出 Concurrent Modification Exception 异常`

   + 延伸

     java.util包下的所有集合类都是快速失败的！

     java.util.concurrent包下的所有类都是安全失败的：ConcurrentHashMap弱一致性的表现！

     [详见：快速失败&安全失败](https://blog.csdn.net/rexct392358928/article/details/51773481)

     [强一致性、顺序一致性、弱一致性和共识](https://blog.csdn.net/chao2016/article/details/81149674)

     再延伸：分布式下的一致性、CAP原理...

   + Stream 面试：[Stream # foreach 方法摸底三问](https://www.cnblogs.com/coderxiaohei/p/11955787.html)

5. Stream 详解：

   [Java 8 Stream 用法示例](https://www.runoob.com/java/java8-streams.html)

   [Stream原理：源码分析](https://blog.csdn.net/dage960815/article/details/100036578)

##### Optional

1. `Optional`类是Java 8 为了解决`null`值判断问题，借鉴`google guava`类库的 `Optional`类而引入的一个同名`Optional`类，使用`Optional`类可以避免显示的`null`值判断（`null`的防御性检查），避免`null`导致的`NPE(NullPointException)`问题。

   ```java
   // 阿里巴巴 java 开发手册
   防止 NPE，是程序员的基本修养，注意 NPE 产生的场景：
       1）返回类型为基本数据类型，return 包装数据类型的对象时，自动拆箱有可能产生 NPE。
       // 反例：public int f() { return Integer 对象}， 如果为 null，自动解箱抛 NPE。
       2） 数据库的查询结果可能为 null。 
       3） 集合里的元素即使 isNotEmpty，取出的数据元素也可能为 null。
       4） 远程调用返回对象时，一律要求进行空指针判断，防止 NPE。 
       5） 对于 Session 中获取的数据，建议 NPE 检查，避免空指针。
       6） 级联调用 obj.getA().getB().getC(); 一连串调用，易产生 NPE。
   /* 解决：使用 JDK8 的 Optional 类来防止 NPE 问题 */
   ```

2. Optional 是一个元素（元素：Optional内部维护的一个 泛型T 的对象）可以为null的容器。

   ```java
   isPresent()	方法判断元素值是否为null，即判断值是否存在！
   ```

   [详见:Java 8 Optional](https://www.runoob.com/java/java8-optional-class.html)

   [Optional 解决 NPE 问题](https://blog.csdn.net/qq_27276045/article/details/102689086)

##### Date Time API

1. Java 8 通过发布新的 Date Time API（JSR 310）加强对日期与时间的处理。在旧版本的 java  中，日期时间 API存在诸多问题：
   + **非线程安全**：java.util.Date 是线程不安全的，所有的日期类都是可变的，这是 Java 日期最大的问题之一。
   + **设计差**：Java 的日期/时间类的定义并不一致，在 java.util 和 java.sql 的包中都有日期类，此处用于格式化和解析的类在 java.text 包中定义。java.util.Date 同时包含日期和时间，而 java.sql.Date仅仅包含了日期，所以将其纳入 java.sql 包并不合理。另外这两个类都有相同的名字——糟糕的设计。
   + **时区处理麻烦**：日期类并不提供国际化，没有时区支持，因此 java 引入了 java.util.Calendar 和 java.util.TimeZone 类，但他们同样存在上述问题！
   
2. [详见：Java8新特性-日期类](https://www.jianshu.com/p/cb9e04077201)

##### Base 64

+ [详见：Java8 Base64](https://www.runoob.com/java/java8-base64.html)

+ Base64 与 MD5

  ```java
  /*
  【Base64】 是一种编码格式，如同 UTF-8
  1、是一种采用 64 个字符 表示任意二进制数据的编码
  2、具备可逆性：可编码，可解码
  3、可以将图片等 二进制文件转换为 文本文件
  4、可以将 非ASCII字符 的数据转换为 ASCII字符，避免了比可见字符！
  5、使用场景：
  	1）通常用于处理文本数据的场合，表示、传输、存储一些二进制数据，包括MIME的电子邮件及XML的一些复杂数据，例如 http 请求/响应
  
  【MD5】 是一种加密算法
  1、使用的是 一种散列表的计算方式，具有弱碰撞性和高度离散型：将原始字符修改一点点，其加密后的字符串变化都会很大！
  2、具备不可逆性
  3、任意长度的明文字符串，加密后得到的密文字符串的长度是固定的！
  4、使用场景：
  	1）储存密码
  	2)下载文件时，比较文件前后的 md5 判断文件是否被修改
  	3）用 md5 进行数字签名：防止数字签名被篡改
  */
  ```

  