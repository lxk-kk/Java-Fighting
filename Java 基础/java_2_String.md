#### String知识点

##### 不可变

```java
/*
 · String 类中没有提供用于修改 字符串的方法，所以在 java 文档中将 String类对象 称为不可变字符串！
 · 要修改一个字符串，只会创建一个新字符串，并在新字符串上修改，返回新字符串！
 
 · 不可变字符串的优点：编译器可以让字符串共享！
 	即可视为：所有字符串存放在公共的存储池中，字符串变量指向存储池中相应的位置，如果复制一个字符串变量，原始字符串变量和复制的字符串变量共享池中相同字符串！

*/
```



##### new String("abc")到底创建了几个对象

```xml
<!--
【方法区与元空间】

方法区：线程共享，用于存放已被虚拟机加载的类信息、常量、静态变量、即时编译器（JIT）编译后的代码数据！

jdk 1.6 HotSpot虚拟机使用“永久代 PermGen”实现方法区（虽然HotSpot的方法区常常被称为“永久代”，但是二者并不等价，其他厂商的虚拟机没有“永久代”），它同 堆 一样是线程共享的运行时数据区，使用永久代实现方法区，方法区中的内存回收同堆中的内存回收没有什么区别，这样实现就可以像管理 堆 内存一样管理这部分内存（使用垃圾回收器回收），但是由于方法区的内存大小固定，当方法区内存占用达到 XX:MaxPermSize 设定的大小时就会导致内存溢出（OOM异常）

jdk 1.7 及之后版本的JVM，便将 字符串常量池 从方法区中移除，并在堆中开辟 一块新的区域给 字符串常量池
	为什么会移到堆中？永久代的垃圾回收和老年代的垃圾回收是绑定的，一旦其中有一个区域被占满，这两个区域都要进行垃圾回收，当通过 -XX:MaxPerSize 设置了方法区的内存空间后，一旦类的元数据超过了设定的大小，程序就会耗尽并出现内存溢出OOM！将字符串常量池放在 永久代中，会导致一系列的性能问题和内存溢出问题！
	一旦常量池中大量使用 intern 是会直接产生java.lang.OutOfMemoryError: PermGen space错误的！

jdk 1.8 彻底消除了方法区，取而代之的是 元空间（MetaSpace）！
元空间中只存放类的元数据信息！其余信息被移动到堆中！
元空间与方法区最大的区别在于，元空间并不存在于虚拟机中，而是使用的本地内存！其内存原则上只受制于本地内存的大小（容量取决于是32位的或是64位的操作系统的可用虚拟内存大小）。
	1、可以通过 -XX:MaxMetaspaceSize 参数为元空间设置内存大小，占用空间达到内存大小的限制时，抛出OOM异常（这种OOM是由元空间内存分配失败由JVM抛出的）！
	2、也可以不指定元空间的内存大小：此时元空间会在运行时根据需要动态调整所需要的内存的大小，于是它的上限便是本地内存的大小！
	3、忽略 PermSize 和 MaxPermSize 参数

jdk 1.8 将类元数据放到本地内存的元空间中，将常量池、静态变量放到了堆中！

元空间充分利用了 Java 语言规范中的优点：类及相关的元数据的生命周期与类加载器一致！元空间 为每个类加载器 分配了一个以 “块” 为数据结构的储存空间，“块”与类加载器相关联，该类加载器所加载的所有类的类元数据都会被存放到该“块”中！不同块之间，大小不同！
元数据的存储地址固定！
由于元空间中只储存类加载后的类元数据，其生命周期与对应的类加载器一致，所以并不需要对其进行垃圾收回，当某个类加载器不再存活时，会将整个对应的块空间收回！
【元空间相关】详见：
	https://www.iteye.com/blog/aoyouzi-2243929
	https://www.cnblogs.com/williamjie/p/9558094.html
	https://www.infoq.cn/article/Java-PERMGEN-Removed/
	https://www.cnblogs.com/gccbuaa/p/7233916.html

	为什么要移除永久代?
	1、它的大小是在启动时固定好的，很难进行调优! -XX:MaxPermSize 设置无标准！
	2、永久代中的元数据可能会随着 每一次 Full GC 发生而进行移动，而元空间中的元数据对象不会移动！
	3、元空间的回收过程没有重定位和压缩过程，元空间的内存管理由 “元空间虚拟机” 完成，在方法区中对于类的元数据，我们需要使用不同的垃圾回收器处理，当类加载器不再存活时，回收与之相关联的整个空间即可！元空间省掉了GC扫描和压缩的时间！
-->
```

```xml
<!--
【JVM中的常量池：字符串常量池、class常量池、运行时常量池】
详见：https://www.cnblogs.com/shoshana-kong/p/11249311.html
-->
```

```java
/*
下文的 池：字符串常量池！

池中只保留一份 “字符串”：意为 池中要么是表示该字符串的对象！要么是表示该字符串的引用！
	如果是 “字符串”对像，则是在 池中创建的对象
	如果是 “字符串”的引用，则不是在池中创建，而是在堆中创建的对象，只是 在池中保存了该对象的引用！

出现""，就会在字符串常量池中创建一个字符串对象，当该引号所表示的字符串已存在，就直接返回该字符串对象!
通过new 创建的字符串会在堆中创建一个对象!
	所以 new String("1"); 会在堆中创建，如果 池中没有，则还会在池中创建！
	
String的intern()方法，判断当前字符串对象所表示的字符串在字符串常量池中是否存在，如果不存在，则创建，否则无操作(在jdk1.6中，该方法创建的是字符串对象，而在jdk1.7及之后，由于字符串常量池已移动到堆中，并没有必要新创建字符串对象，而是创建堆中该对象的引用!)

结论：
对于jdk1.6，无论何时，字符串常量池中的对象与堆中的对象都不相等(一个在堆中创建，一个在字符串常量池中创建)
jdk1.7及之后，字符串常量池中的字符串可能与堆中的相同，因为字符串常量池中可能是指向堆中对象的引用!
*/
    
public class Test {
    public static void main(String[] args) {
        // 池中只保留一份 “字符串”：意为 池中要么是表示该字符串的对象！要么是表示该字符串的引用！
        // 如果是 “字符串”对像，则是在 池中创建的对象
        // 如果是 “字符串”的引用，则不是在池中创建，而是在堆中创建的对象，只是 在池中保存了该对象的引用！

        System.out.println("----------------------------------");
        // 在堆中创建，池中没有，在池中创建！
        String b = new String("aa");
        // 池中已经存在！
        b.intern();
        // 池中已有，直接返回！
        String a = "aa";
        // true : a 和 intern 是直接返回池中已经存在的字符串！就是 new String("1") 在堆中创建的同时，在 池中创建的对象！
        System.out.println(b.intern() == a);
        // false ：堆中创建的和池中创建的不一样！
        System.out.println(b.intern() == b);
        System.out.println("----------------------------------");

        // 池中没有！创建“1a”
        String s2 = "1a";
        // 堆中创建，池中已有，不用创建！
        String s1 = new String("1a");
        // true : 都是 创建 s2 时的对象
        System.out.println(s1.intern() == s2);
        // false : s2 是 池中的对象，调用 s1.intern 时已经存在，直接返回即可，而 s1 是堆中的对象，两者不同！
        System.out.println(s1 == s1.intern());
        System.out.println("----------------------------------");

        // 池中没有"123"，只在堆中创建！
        String s3 = "1" + "2" + new String("3");
        // 池中创建s3对象的引用！
        s3.intern();
        // 池中已存在：直接返回该对象引用！（即为s3的引用）
        String t3 = "123";
        // true ：上述 调用 intern 方法前，池中不存在 "123" ，所以将 s3 的引用放入池中！所以两者引用的对象相同！
        System.out.println(s3 == s3.intern());
        // true : 创建 t3 时，池中已存在"123"，它是 s3 的引用！如上述 s3.intern 方法也是 s3 的引用！
        System.out.println(s3.intern() == t3);
        // true : t3 引用指向的就是 s3
        System.out.println(s3 == t3);
        System.out.println("----------------------------------");

        // 只在堆中创建，而 池中不创建"1234"
        String s4 = "12" + new String("34");
        // 池中没有"1234"，创建 "1234" 对象
        String t4 = "1234";
        // 池中已存在 "1234"，是 t4 创建的，直接返回 t4 引用
        s4.intern();
        // false ：如上述，s4.intern 是 t4 的引用 ，而非 s4 的引用
        System.out.println(s4 == s4.intern());
        // true : 如上述，s4.intern 是 t4 的引用
        System.out.println(s4.intern() == t4);
        // false : s4 是最先开始在 堆中创建的对象，t4 是后来在池中创建的对象，两者不同！
        System.out.println(s4 == t4);
    }
}

```

+ [阿里面试官：字符串在JVM中如何存放？](https://mp.weixin.qq.com/s/zibKAhoTfIvBp6GcN6KGsg)
+ [你真的了解String类的intern()方法吗](https://blog.csdn.net/weixin_30828379/article/details/99526088)
+ [String中intern方法的作用](https://blog.csdn.net/guoxiaolongonly/article/details/80425548)
+ [java中的常量池：字符串常量池、运行时常量池、class常量池](https://www.cnblogs.com/shoshana-kong/p/11249311.html)

##### Java 一个字符占多少字节，为什么？Java字符的编码

```java
/*
UTF-16编码（一个 代码单元 为 16bit 2个字节）采用不同长度的编码方式 表示所有的Unicode码点（码点：指与一个编码表中的某个字符对应的代码值），在Unicode标准中，将码点分成17个等级（code plan），第一等级便是 “基本多语言级别” ，其余16个级别的码点中，则包含了一些“辅助字符”。
在基本多语言级别中，每个字符使用16位表示（2个字节），通常被称为代码单元（code unit）；辅助字符采用 一对连续的代码单元 进行编码。

java 字符串由 char 值序列组成，char 类型是采用 UTF-16 编码表示 Unicode码点 的一个代码单元！大多数常用Unicode字符使用一个代码单元就能表示，而辅助字符需要一对代码单元才才能表示（即两个char组成：一个码点由两个代码单元组成）！
*/
```

```java
/*
String 类

str.length()：返回 采用UTF-16 编码 表示的给定字符串所需的 【 代码单元数量 】 ！
str.codePointCount(0,str.length())：得到实际的 字符 长度（码点数量）！

str.charAt(int i)：返回位置 i 的代码单元（可能是一个完整 Unicode字符 ，可能是 辅助字符 的一部分）
str.codePointAt(int i)：获取 第i个索引处的 码点（如果是辅助字符，则会自动结合下一个代码单元，生成该辅助字符的码点）

Character.isSupplementaryCodePoint(int codePoint)：判断码点是否为辅助字符
Character.isSurrogate(char ch)：判断代码单元是否为辅助字符的后部分
Character.toChars(int codePoint)：将码点转换为 char[] 
*/

// 如果想要正序遍历一个字符串，并遍历其所有码点！
void iterateString(String str){
    for(int i=0;i<str.length();++i){
        int codePoint=str.codePointAt(i);
        if(Character.isSupplementaryCodePoint(codePoint)){
            i++;
        }
        System.out.println(String.valueOf(Character.toChars(codePoint)));
    }
}

// 如果想要逆序遍历一个字符串，并遍历器所有码点
void interateStringDesc(String str){
    for(int i=str.length()-1;i>=0;--i){
        if(Character.isSurrogate(str.charAt(i))){
            i--;
        }
        int codePoint=str.codePointAt(i);
        System.out.println(String.valueOf(Character.toChars(codePoint)));
    }
}
```

##### 字符串反转

```java
/*
注意：字符串反转：不能直接操作字符char！必须要考虑辅助字符（由两个代码单元构成：两个char类型构成一个Unicode字符）
*/

// StringBuilder 类的 reverse 方法：不用人工处理辅助字符的码点问题！
public static String reverse_1(String string) {
    StringBuilder builder = new StringBuilder(string);
    return builder.reverse().toString();
}

// 字符串遍历码点：正序遍历！
public static String reverse_2(String string) {
    String result = "";
    for (int i = 0; i < string.length(); ++i) {
        // 获取 字符串中 第i个索引处的码点（如果是辅助单元，则将连续获取两个代码单元获得码点）
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

// 字符串遍历码点：逆序遍历！
public static String reverse_3(String string) {
    String result = "";
    for (int i = string.length() - 1; i >= 0; i--) {
        // 判断 第i个 代码单元 是否是辅助字符中的代码单元
        if (Character.isSurrogate(string.charAt(i))) {
            // 如果是 则跳到前该辅助字符的第一个 代码单元
            i--;
        }
        // 获取 字符串中 第i个索引处 的码点（如果是辅助单元，则将连续获取两个代码单元获得码点）
        int codePoint = string.codePointAt(i);
        result += String.valueOf(Character.toChars(codePoint));
    }
    return result;
}
// 递归？
```

```java
/*
Java规定了字符的内码要用UTF-16编码，一个字符是2个字节，并规定字符串是UTF-16 code unit的序列。
外码字符所占字节取决于具体编码。字符和字节是不一样的。

内码：指程序内部使用的字符编码，即char、String类型再内部使用的内部编码！
外码：指程序与外部交互时使用的字符编码！

外码编码不同，字符和字节的换算不同，几种常见的编码换算如下：

ASCII编码是单字节编码，只有英文字符，不能编码汉字。
GBK编码1个英文字符是1个字节，一个汉字是是2个字节。
UTF-8编码1个英文字符是1个字节，一个汉字是3个字节。
Unicode编码1个英文字符是2个字节，一个汉字是2个字节。

详见：https://www.cnblogs.com/Xieyang-blog/p/9401999.html
*/
```

##### String.length()：代码单元的长度

```java
// 源码
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
    /**
     * The value is used for character storage.
     *
     * @implNote This field is trusted by the VM, and is a subject to
     * constant folding if String instance is constant. Overwriting this
     * field after construction will cause problems.
     *
     * Additionally, it is marked with {@link Stable} to trust the contents
     * of the array. No other facility in JDK provides this functionality (yet).
     * {@link Stable} is safe here, because value is never null.
     */
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
}

/*
如上：字符串 在String内部都是以 byte 数组的形式储存
 根据 储存值的不一样，String 会先择 UTF16 和 LATIN1 两种不同的编码方式对字符储存！
 1、 UTF16 编码方式：两个字节代表一个字符！
 2、 LATIN1 编码方式：一个字节代表一个字符！
*/
```

##### 空字符串哪种情况下会报异常：

+ 示例：[Gson解析空字符串发生异常的处理方法](https://www.jb51.net/article/87159.htm)

