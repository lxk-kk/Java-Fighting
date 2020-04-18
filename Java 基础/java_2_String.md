#### String

##### 字符串 编码

+ **UTF-16 编码**

  ```
   · Unicode 是一种编码机制，拥有自己的 Unicode 字符集，每个字符也称为 Unicode 字符！
   · UTF-16 编码采用不同长度的编码 表示 所有的 Unicode码点（码点：编码表中的代码值）！
   	Unicode 码点可以分为 17 个代码等级，第一个等级称为 基本多语言级别，剩余的 16 中级别中，包含了一些 辅助字符！
   	基本多语言级别中，每个字符采用 16bit 表示，通常被称为一个 代码单元；
   	其余级别中，每个 辅助字符 采用一对连续的 代码单元 进行编码！
  ```

+ **Java 字符串**

  ```
   · Java 字符串就是 Unicode 字符序列，也就是 char 序列！它没有内置的 字符串类型，而是在标准 Java 类库中提供了一个 预定义类 String。（预定义类：已经定义好的类，会被 JVM 虚拟机自动加载，不需要显示导入）
   
   · Java 中，char 类型 描述了 UTF-16编码 中的一个代码单元！
   	因此，char 类型占用 2 个字节！
   	
   · 我们常说的字符，并不是指 char，而指的是 Unicode字符！
   	大多数常用字符，使用一个 代码单元 就能表示；
   	而 辅助字符需要一对 代码单元 才能表示，即两个 char 才能够表示一个 辅助字符！
  ```

+ *外码 vs 内码*

  ```java
  /*
   · 内码：指程序内部使用的字符编码，即char、String类型再内部使用的内部编码！
   	外码：指程序与外部交互时使用的字符编码！
  
   · 外码编码不同，字符和字节的换算不同，几种常见的编码换算如下：
   	1. ASCII编码是单字节编码，只有英文字符，不能编码汉字。
  	2. GBK编码1个英文字符是1个字节，一个汉字是是2个字节。
  	3. UTF-8编码1个英文字符是1个字节，一个汉字是3个字节。
  	4. Unicode编码1个英文字符是2个字节，一个汉字是2个字节。
  	
   · Java规定了字符的内码要用UTF-16编码，一个字符是2个字节，并规定字符串是UTF-16 code unit的序列。
  */
  ```

  + [详见：内码 vs 外码](https://www.cnblogs.com/Xieyang-blog/p/9401999.html)

##### String 类

###### 不可变

+ 不可变

  ```java
  /*
   · String 类中没有提供用于修改 字符串的方法，所以在 java 文档中将 String类对象 称为不可变字符串！
   · 要修改一个字符串，只会创建一个新字符串，并在新字符串上修改，返回新字符串！
   
   · 不可变字符串的优点：编译器可以让字符串共享！
   	即可视为：所有字符串存放在公共的存储池中，字符串变量指向存储池中相应的位置，如果复制一个字符串变量，原始字符串变量和复制的字符串变量共享池中相同字符串！
  */
  ```

###### 编码

+ 介绍

  ```
   · 字符串在 String 中都是以 byte 数组的 形式存储的！
   	String 会按照字符串的 内容，自动选择 UTF-16 或者 LATIN1 对字符进行编码！
   	1. UTF-16：两个字节代表一个 字符！
   	2. LATIN1： 一个字节代表一个字符！ 
   
   · 如果 字符串中的字符 都可以使用一个 字节 表示那么将会使用 LATIN1 编码方式，使用 1 个字节编码字符！
   	这样就能够减少 byte[] 占用多余的内存空间！
   	
   	否则，使用 UTF-16 编码，2个字节 编码一个字符！
  ```

+ 源码

  ```java
  // 源码
  public final class String
      implements java.io.Serializable, Comparable<String>, CharSequence {
      @Stable
      private final byte[] value;
  
      static final byte LATIN1 = 0;
      static final byte UTF16  = 1;
  
      public int length() {
          return value.length >> coder();
      }
      // COMPACT_STRINGS 是压缩字符串的标识，默认为 true！
      byte coder() {
          return COMPACT_STRINGS ? coder : UTF16;
      }
      // ...
      
      public static String valueOf(char c) {
          // 如果 一个字节能够表示字符，则使用 LATIN1，折叠/压缩 空间！
          if (COMPACT_STRINGS && StringLatin1.canEncode(c)) {
              return new String(StringLatin1.toBytes(c), LATIN1);
          }
          // 否则使用 UTF-16 
          return new String(StringUTF16.toBytes(c), UTF16);
      }
  }
  ```

###### 字符 API

+ API

  ```java
  str.length() // 返回 采用UTF-16 编码 表示的给定字符串所需的 【 代码单元数量 】 ！
  str.codePointCount(0,str.length()) // 得到实际的 字符 长度（码点数量）！
  str.charAt(int i) // 返回位置 i 的代码单元（可能是一个完整 Unicode字符 ，可能是 辅助字符 的一部分）
  str.codePointAt(int i) // 获取 第i个索引处的 码点（如果是辅助字符，则会自动结合下一个代码单元，生成该辅助字符的码点）
  
  Character.isSupplementaryCodePoint(int codePoint) // 判断码点是否为辅助字符
  Character.isSurrogate(char ch) // 判断代码单元是否为辅助字符的后部分
  Character.toChars(int codePoint) // 将码点转换为 char[] 
  ```

  

##### 字符串 反转

+ 注意：

  ```
  字符串反转，不能直接操作字符 char，需要考虑到 辅助字符！（除非字符串中都是基本字符）
  ```

+ 正确示例：

  1. **StringBuilder 的 reverse 方法**

     ```java
     // StringBuilder 类的 reverse 方法：不用人工处理辅助字符的码点问题！
     public static String reverse_1(String string) {
         StringBuilder builder = new StringBuilder(string);
         return builder.reverse().toString();
     }
     ```

  2. **String 码点 API 正序遍历**

     ```java
     // 字符串遍历码点：正序遍历！
     public static String reverse_2(String string) {
         String result = "";
         for (int i = 0; i < string.length(); ++i) {
             // 获取 字符串中 第i个索引处的码点
             // 如果是辅助单元，则将连续获取两个代码单元获得码点）
             int codePoint = string.codePointAt(i);
             if (Character.isSupplementaryCodePoint(codePoint)) {
                 // 如果是辅助字符，则跳过下一个代码单元
                 i++;
             }
             // Character.toChars(str) 将码点转换为字符（返回的是字符数组）
             result = String.valueOf(Character.toChars(codePoint)) + result;
         }
         return result;
     }
     ```

  3. **String 码点 API 逆序遍历**

     ```java
     // 字符串遍历码点：逆序遍历！
     public static String reverse_3(String string) {
         String result = "";
         for (int i = string.length() - 1; i >= 0; i--) {
             // 判断 第i个 代码单元 是否是辅助字符中的代码单元
             if (Character.isSurrogate(string.charAt(i))) {
                 // 如果是 则跳到前该辅助字符的第一个 代码单元
                 i--;
             }
             // 获取 字符串中 第i个索引处 的码点 
             // 如果是辅助单元，则将连续获取两个代码单元获得码点）
             int codePoint = string.codePointAt(i);
             result += String.valueOf(Character.toChars(codePoint));
         }
         return result;
     }
     ```

##### 可变 字符串

+ **不可变 String 弊端**

  ```
   · 由于 String 类型不可变，因此对 String 的修改，都会返回一个新的 String 对象！
   
   · 因此，在拼接字符串 的过程中，不断的创建 新的字符串，会程序降低执行效率！
   	同时，新字符串创建之后，就会丢弃拼接之前的旧字符串对象，这使得，内存中存在大量无用的“中间字符串”，占用内存空间！
   	
   · 为了解决这种 问题，出现了 可变的字符串序列：StringBuffer 及 StringBuilder
  ```

  ![](image\String.png)

+ **StringBuilder & StringBuffer**

  ```
   · StringBuilder StringBuffer 都是可变字符串序列，对字符序列的修改，不会生成新字符串对象！
   	其内部维护的是 byte[]，对字符串的修改，就是对数组的修改！
  ```

  *byte[] 扩容* ：**默认长度=16  默认扩容=max( 2*原长+2, append的字符串长度+原字符串长度 )** 

  ```
   · StringBuilder StringBuffer 默认的字符串长度是 16，避免频繁的扩容！
   	在 append 字符序列时，如果 byte[] 长度不足，则需要扩容，默认是 2*字符串原长 + 2，如果此长度不满足需求的话，就会采用 所要 append的字符串长度+原字符串长度！
   	
   	注意：这里讨论的是字符串的长度，不是 byte[] 的具体长度，byte[] 的具体长度还得看使用的是 UTF-16 编码，还是使用了 LATIN1 进行编码！
  ```

  ![](image\String & StringBuilder & StringBuffer.png)

+ **StringBuilder vs StringBuffer**

  ```
   · StringBuilder 与 StringBuffer 的区别主要在于，StringBuffer 是线程安全的，它在每个 字符序列的修改方法上都加上了 synchronized 锁！
   
   · 因此，无锁竞争 的环境下，可以使用 StringBuilder，因为避免了 无谓的 加锁解锁，执行速度更快！
   	存在锁竞争的环境下：使用 StringBuffer，能够保证字符串并发修改的 线程安全性！ 	
  ```

##### 字符串 存储

###### 方法区

+ 方法区是线程共享的内存区域。用于存放被 java 虚拟机加载的 *类型信息*、*常量*、*静态变量*、*即时编译器编译后的代码缓存* 等数据。

+ 《 Java虚拟机规范 》中规定，当 方法区无法满足新的 内存需求时，将会抛出 OOM 异常！

+ **运行时常量池**

  + class 文件常量池 在 编译期生成，用于存放类相关的 *字面量和符号应用* ；

    而在类加载之后，*类相关的字面量、符号引用 以及 由部分符号引用翻译出的直接引用* 将会被加载到 运行时常量池中！

  + 运行时常量池 vs class文件常量池

    *运行常量池具备动态性* ：java 语言并不要求只能在编译期生成常量，即，并非预置入 Class 文件常量池 的内容才能进入 运行时常量池，运行期生成的常量 也能 存储在运行时常量池中！

    例如：String 类的 intern() 方法！

  + 运行时常量池 是 方法区的一部分，受到方法区内存的限制，当常量池无法再申请到足够的内存时会抛出 OOM 异常！

+ HotSpot 中的方法区

  + HotSpot 将永久代扩展到 方法区，使得 垃圾回收器能够像管理 java 堆一样管理这片区域！

  + jdk1.7 开始，HotSpot 将方法区中的 字符串常量池、静态变量 移到堆中！

    ```java
    /*
     · 原因：
     	HotSpot 使用永久代管理 方法区，方法区的大小固定，由 XX:MaxPermSize 参数设定，当方法区无法满足新的内存需求时，就会触发 GC 甚至 抛出OOM！
     	字符串常量池 是 运行时常量池 的一部分，用于存放池中全局唯一的 字符串！由于运行时常量池具备动态性，能够存储运行期生成的字符串常量，例如：String 的 intern 方法生成的字符串常量！所以，当程序中动态生成的字符串常量较多，导致常量池内存不够时，会抛出 OOM异常！
    
     	将字符串常量池放入堆中，为字符串常量提供了更大的内存空间，同时缓解了方法区的内存消耗！
    */
    ```

  + jdk1.8 开始，HotSpot 废弃方法区，将剩余的内容（主要为类型信息）移动到元空间中！

    ```java
    /*
     · 元空间并不在 JVM 管辖的内存中，它属于本地内存，原则上只会受限制于操作系统的大小！（32位/64位 操作系统可用的虚拟内存的大小）。
     
     · 内存分配
     	1. 可以通过 -XX:MaxMetasapceSize 设定元空间的内存大小，当内存不够时，会抛出 OOM 错误！
     	2. 如果不指定元空间内存大小，本地内存就是 元空间内存的上限！
     	3. 元空间会忽略 PermSize 和 MaxPermSize 参数!
     	
     · 元空间 & 类加载器
    	元空间为每个类加载器 分配了一个 “块”空间，该块空间与类加载器相关联，用于存储 该类加载器 所加载的所有类元信息！不同块空间，大小不同！
    	由于元空间只存储类加载后的类元数据，其生命周期与对应的类加载器一致，当不再使用某个类型时，不需要单独回收，而是所属的类加载器不再使用时，会将类加载器对应的整个块空间回收！
     	这充分利用了 Java 规范的特点：类及相关元数据的生命周期 与 类加载器一致！
     
     · 为什么废除永久代？（永久代缺点）
     	1. 永久代的大小在启动时就固定，并且没有设定标准，很难进行调优！
     	2. 永久代的数据位置 可能会随着 每一次的 full GC 而移动，而元空间中的数据不会发生移动！
       3. 元空间的内存回收过程没有重定位和压缩过程！当类加载器不再存活时，回收与之相关的整个 块空间 即可，元空间省略了 GC 扫描 和 压缩的时间！
    */
    ```

    >[java8 的元空间](https://www.iteye.com/blog/aoyouzi-2243929)
    >[Metaspace 介绍](https://www.cnblogs.com/williamjie/p/9558094.html)
    >[java 的永久代去哪儿](https://www.infoq.cn/article/Java-PERMGEN-Removed/)
    >
    >[JVM 移除永久代](https://www.cnblogs.com/gccbuaa/p/7233916.html)

###### 字符串常量池

+ 字符串常量池 中的 字符串

  ```
   · 池中只保留一份 “字符串”：意为 池中要么是表示该字符串的对象！要么是表示该字符串的引用！
  	如果是 “字符串”对像，则是在 池中创建的对象
  	如果是 “字符串”的引用，则不是在池中创建，而是在堆中创建的对象，在池中保存了该对象的引用！
  ```

+ 字符串创建规则：

  + **双引号("")**

    ```
     · 在字符串常量池中 创建一个字符串对象，如果池中已经存在了该字符串，则直接返回！
    ```

  +  **new 关键字**

    ```
     · 任何时刻，都会在堆中创建一个字符串对象！
    
     · 示例：new String("1"); 会在堆中创建字符串对象，如果常量池中没有，则还会在池中创建！
    ```

  + **str.intern() 方法**

    ```
    · 如果 str 所代表的 字符串在 常量池中存在，则直接返回：堆中对象的引用 或 常量池中的字符串的引用
    · 如果 str 代表的字符串在常量池中不存在，则创建：
     	 jdk1.7 之前，直接在 常量池 中创建字符串，并返回该字符串的引用！
     	 jdk1.7 开始，直接在 常量池 中创建 堆中该字符串对象的引用（常量池就在堆中），并返回！
    ```

+ *代码示例*

  ```java
  /*
   · jdk1.7 之前：无论何时，字符串常量池中的对象与堆中的对象都不相等(分别在堆、字符串常量池中创建)
   · jdk1.7 开始：字符串常量池中的字符串可能与堆中的相同，因为字符串常量池中可能是指向堆中对象的引用!
   
   · 测试环境：jdk1.8
  */
  public static void main(String[] args) {
      System.out.println("----------------------------------");
      // 在堆中创建，池中没有，在池中创建！
      String b = new String("aa");
      // 池中已经存在！
      b.intern();
      // 池中已有，直接返回！
      String a = "aa";
      System.out.println(b.intern() == a); 	// true 
      System.out.println(b.intern() == b);	// false
      
      System.out.println("----------------------------------");
      // 池中没有！创建“1a”
      String s2 = "1a";
      // 堆中创建，池中已有，不用创建！
      String s1 = new String("1a");
      System.out.println(s1.intern() == s2);		// true
      System.out.println(s1 == s1.intern());		// false 
      
      System.out.println("----------------------------------");
      // 池中没有"123"，只在堆中创建！
      String s3 = "1" + "2" + new String("3");
      // 池中创建s3对象的引用！
      s3.intern();
      // 池中已存在：直接返回该对象引用！（即为s3的引用）
      String t3 = "123";
      System.out.println(s3 == s3.intern()); 		// true
      System.out.println(s3.intern() == t3);		// true
      System.out.println(s3 == t3);				// true
  
      System.out.println("----------------------------------");
      // 只在堆中创建，而 池中不创建"1234"
      String s4 = "12" + new String("34");
      // 池中没有"1234"，创建 "1234" 对象
      String t4 = "1234";
      // 池中已存在 "1234"，是 t4 创建的，直接返回 t4 引用
      s4.intern();
      System.out.println(s4 == s4.intern()); 		// false
      System.out.println(s4.intern() == t4);		// true
      System.out.println(s4 == t4);				// false
  }
  ```

+ 相关资料
  + [阿里面试官：字符串在JVM中如何存放？](https://mp.weixin.qq.com/s/zibKAhoTfIvBp6GcN6KGsg)
  + [你真的了解String类的intern()方法吗](https://blog.csdn.net/weixin_30828379/article/details/99526088)
  + [String中intern方法的作用](https://blog.csdn.net/guoxiaolongonly/article/details/80425548)
  + [java中的常量池：字符串常量池、运行时常量池、class常量池](https://www.cnblogs.com/shoshana-kong/p/11249311.html)

