### java_集合

#### 集合框架

+ 集合类存放于 java.util 包中，主要有 *set（集）、list（列表，包括 queue）、map（映射）* 等类型的集合！

+ 集合框架基础接口：

  **Collection **：*是 List、Set、Queue 等集合接口的基础接口* ！

  **Map**：*是映射表的基础接口* ！

   Iterable ：迭代器接口，是 Collection 的父接口，用于迭代遍历 Collection 中的元素！

  <img src="image\集合类图.jpg"  />

+ 集合工具类：

  **Arrays、Collections**

#### Collection

<img src="image\Collection类图.png" style="zoom: 67%;" />

##### *Set*

###### HashSet：散列集

+ **基于 HashMap 实现，底层是 哈希表**！是无序集！

+ HashSet 的元素是 HashMap 的 key，value 为全局的 Object 常量 ！

+ 通过 HashMap 能够 *支持快速查找*，但是*不会维护元素插入的顺序* ！

  ```java
  public class HashSet<E> extends AbstractSet<E> implements Set<E>, Cloneable, java.io.Serializable {
      static final long serialVersionUID = -5024744406713321676L;

      // 基于 hashmap 实现 [ 集 ]
      private transient HashMap<E,Object> map;

      // 当作 hashmap 中的 假value
      private static final Object PRESENT = new Object();

      // 构造空集：底层 hashmap 实例默认容量未 16，填装因子为 0.75
      public HashSet() {
          map = new HashMap<>();
      }
      // ... ...
  }
  ```


###### TreeSet：树集

+ **默认基于 TreeMap 实现，底层是 红黑树**。是**有序集**，支持 有序性操作，例如根据一个范围查找元素

+ TreeSet 的元素是 TreeMap 的 key，value 为全局的 Object 常量 ！

+ *TreeSet 查找的时间复杂度为 O(logN)*，*HashSet 的查找时间复杂度为 O(1)*：查询单个元素上，**HashSet 更快，但 TreeSet 支持有序操作**！

  ```java
  public class TreeSet<E> extends AbstractSet<E> implements NavigableSet<E>, Cloneable, java.io.Serializable{
      // 基于 navigable map 实现 [ 有序集 ]
      private transient NavigableMap<E,Object> m;

      // 当作 navigable map 中的 假value
      private static final Object PRESENT = new Object();

      // Constructs a set backed by the specified navigable map.
      TreeSet(NavigableMap<E,Object> m) {
          this.m = m;
      }
      /*
       · 构造一个空树集：默认使用 tree map 实现！
       	所有添加到 set 中的对象，都必须要实现 Comparable 接口，并实现 compareTo() 方法，保证元素之间相互可比较
       	String、Integer 等已经实现了 该接口，可以直接使用！
       	而自定义的类，必须要要是实现上述接口与方法！
      */
      public TreeSet() {
          this(new TreeMap<>());
      }
      // ... ...
  }
  ```


###### LinkedHashSet

+ 继承于 HashSet，但是，底层专门使用 LinkedHashMap 实现！

+ **LinkedHashSet 具有 HashMap 的查找效率，并且，其内部通过 双端队列 维护元素的插入顺序** ！

+ *【补充】LinkedHashMap 的实质，是在 HashMap 的基础上，重定义了 HashMap 的节点，仅仅在节点中添加了 before、after 前后节点的引用，使用 HashMap 存储的同时，达到双端队列的效果！*

+ LinkedHashSet 实现上只有 4 个构造方法，它继承于 HashSet，所有操作与 HashSet 相同，使用时，调用的是父类 HashSet 的接口！

  ```java
  // LinkedHashSet 调用的是 HashSet 的构造方法，并与 HashSet 共用所有操作！
  public class LinkedHashSet<E> extends HashSet<E> implements Set<E>, Cloneable, java.io.Serializable {

      private static final long serialVersionUID = -2851667679971038690L;

      public LinkedHashSet(int initialCapacity, float loadFactor) {
          super(initialCapacity, loadFactor, true);
      }
      public LinkedHashSet(int initialCapacity) {
          super(initialCapacity, .75f, true);
      }
      // 空构造器
      public LinkedHashSet() {
          super(16, .75f, true);
      }
      public LinkedHashSet(Collection<? extends E> c) {
          super(Math.max(2*c.size(), 11), .75f, true);
          addAll(c);
      }
  }

  // HashSet 专门为 LinkedHashSet 提供的构造器！
  public class HashSet<E> extends AbstractSet<E> implements Set<E>, Cloneable, java.io.Serializable{
      static final long serialVersionUID = -5024744406713321676L;

      private transient HashMap<E,Object> map;

      // Dummy value to associate with an Object in the backing Map
      private static final Object PRESENT = new Object();
      // ... ...

      // 该构造器为 包私有，只能呗 LinkedHashSet 使用！
      HashSet(int initialCapacity, float loadFactor, boolean dummy) {
          // LinkedHashMap 继承于 HashMap
          map = new LinkedHashMap<>(initialCapacity, loadFactor);
      }
  }
  ```

##### *List*

###### ArrayList

+ ArrayList 基于**动态数组** 实现，支持**随机访问、扩容、缩容**！
+ *增删慢：O(N) ，查询快：O(1)*

###### Vector

+ 类似于 ArrayList ，它是 *线程安全* 的 ArrayList

###### LinkedList

+ 是基于 链表/结点 的结构实现的**双向链表**！
+ *增删快：O(1) ，查询慢（只能顺序访问）：O(N)*
+ 继承了 Deque 接口，支持 首、尾 结点的数据操作，因此还能作为 **队列、双向队列、栈 **使用！

##### *Queue*

###### LinkedList

+ 继承了 Deque 接口，支持首尾结点的数据操作，可以作为**普通队列**，也可以作为**双向队列**！

###### PriorityQueue

+ 基于 **堆** 实现，最大堆、最小堆实现**优先权队列**！

  ```
  初始建堆：O(n)
  修改后堆化：O(nlogn)
  ```


#### Map

<img src="image\Map类图.png" style="zoom:67%;" />

##### TreeMap

+ 基于 **红黑树** 实现！

##### HashMap

+ 基于 哈希表 实现！
+ **jdk1.8 之前，哈希表 使用的是 数组链表 的结构，jdk1.8 开始，使用了 数组+链表/红黑树 的结构！**

##### LinkedHashMap

+ 继承于 HashMap，并在 HashMap 的继承上，重写了 结点结构，仅仅为结点新增了 前后继结点的应用，使得 LinkedHashMap 在*以 HashMap 结构存储的基础上，达到 双向链表 的效果*！

  [LinkedHashMap 详解](https://www.jianshu.com/p/8f4f58b4b8ab)

+ 使用双向链表维护了 数据插入时的顺序，可以指定Map 维护插入顺序 或者 操作顺序（默认为 插入顺序）

+ 同时，LinkedHashMap 还*实现了 LRU 功能* ！

  [LRU 算法](https://mp.weixin.qq.com/s/KgXE195lmX0fUrqLBMWbqA)

##### HashTable

+ 是线程安全的 HashMap，但是性能低！
+ 需要使用 HashMap 线程安全时，应该使用 ConcurrentHashMap！

#### 容器中的设计模式

##### 迭代器 模式

+ 迭代器 模式

  它为遍历容器提供了统一的接口，通过统一的接口，开发人员能够轻松的遍历容器中的元素，而不必关注不同容器具体的实现方式！

+ **实现方式**：

  Java 中容器的基础接口是 Collection，它继承了 Iterable 接口，接口方法 iterator 用于创建一个 Iterator对象，这个对象就是一个迭代器对象！

  Iterator 是一个 接口，它为遍历容器提供了统一的规范，所有实现了 Collection 的容器都会依据自身的特点，聚合一个 Iterator 接口的实现类，并通过实现 Iterable 的接口方法，将该类的对象作为迭代器对象，提供给开发人员！

  于是，开发者就只需要通过 Iterator 对象，访问所有容器中的元素，而不必关心这些容器的实现方式！

  ```java
  // Iterable 接口
  public interface Iterable<T> {
      // Returns an iterator over elements of type {@code T}.
      Iterator<T> iterator();
      // ... ...
  }

  // Collection 容器基础接口：继承了 Iterable 接口
  public interface Collection<E> extends Iterable<E> {
      // ... ...
  }

  // Iterator 接口：遍历容器的规范接口
  public interface Iterator<E> {
      boolean hasNext();
      E next();

      default void remove() {
          throw new UnsupportedOperationException("remove");
      }

      default void forEachRemaining(Consumer<? super E> action) {
          Objects.requireNonNull(action);
          while (hasNext())
              action.accept(next());
      }
  }
  ```

+ *为什么容器不直接继承 Iterator 接口？* *而是通过聚合的方式呢？*

  我个人的理解是这样的，直接继承 Iterator 接口，那么遍历容器的任务就会交给容器完成！这就导致容器必须要实现迭代器，并且是紧耦合的！

  而以聚合的方式实现，容器就只需要关注管理数据的任务，而迭代容器就交由 Iterator 的实现类完成，各司其职！

+ jdk1.5 之后，可以*使用 foreach 方法来遍历 Iterable 接口的实现类对象* ！

  **for each 迭代容器时，会自动调用 Iterator 中的 next、hasNext 方法**

  ```java
  List<String> list=new ArrayList<>(1);
  list.add("a");
  list.add("b");
  for(String item:list){
      // 会调用 iterator 迭代器的方法 next、hasNext
      System.out.println(item);
  }
  ```

##### 适配器 模式

+ 适配器 模式

  *将一个接口转换为用户所希望的接口！适配器模式，通过增加一个新类，作为 适配器类，解决接口冲突的问题，使得原本没有关系的类，可以协同工作！*

+ java.util.Arrays # **asList() 就使用了适配器模式**，**将 对象数组类型 转换为 List 类型**

  ```java
  // 注意 asList 是 泛型的 可变长参数：所以不能使用基本类型
  public static <T> List<T> asList(T... a) {
      // ArrayList 是工具类 Arrays 中自定义的类：相当于 适配器类！
      return new ArrayList<>(a);
  }

  // 适配器类
  private static class ArrayList<E> extends AbstractList<E>{

      // 适配者：将 泛型数组 转换为 ArrayList
      // 使用 final 修饰，表示该对象不可指向其他引用！
      private final E[] a;

      // 泛型数组 是 用户提供的，即：用户提供 适配者
      ArrayList(E[] array) {
          // 保证数组构造的数组不为 null，否则，抛出 NullPointException 异常
          a = Objects.requireNonNull(array);
      }
      // 以下是 通过泛型数组 实现的一些列的 ArrayList 接口！
      // ... ...
  }
  ```

+ 注意：

  1. 适配器类 使用了 泛型数组 作为 原始数组对象的引用，因此，**原始数组不能为 基本类型！**

  2. **适配器类并没有实现 add/remove/clear 方法**，当调用这些方法时会**抛出 UnsupportedOperationException 异常**！

  3. 该私有集合类，**底层依旧使用的数组实现**，并且该数组引用就是所要转换的数组引用。

     适配器模式 ：只转换接口，后台数据仍然是原数组！

+ asList 使用方式

  ```java
  // 方式 1
  Integer[] arr={1,2,3,4};
  List list=Arrays.asList(arr);

  // 方式 2
  List list=Arrays.asList(1,2,3,4);

  // 错误方式：不能使用 基本类型！应该使用 包装类！
  int[] arr={1,2,3,4}
  List list=Arrays.asList(arr);
  ```

#### 源码分析

##### ArrayList

+ ArrayList 是动态数组，底层是 Object 数组实现！

  它能够根据数组的容量自动扩容，扩容时，新容量为当前容量的 1.5 倍！

  扩容实际上是创建一个新数组，再通过 System.arraycopy 方法，实现深拷贝！

######  扩容（扩容机制）

+ 初始化

  + **初始容量为 10，如果创建 ArrayList 时，不指定或者指定容量为0，则 在第一次添加元素时，容量默认为 10！**
  + 容量最大为 **2^31-1**

+ 扩容

  ```java
  // 参数 minCapacity 传入的是 即将插入元素后，当前数组的总长度！
  private int newCapacity(int minCapacity) {
      int oldCapacity = elementData.length;
      
      // 扩容 1.5 倍
      int newCapacity = oldCapacity + (oldCapacity >> 1);
      
      if (newCapacity - minCapacity <= 0) {
          // 如果 为空数组，则 将默认扩容的容量作为此次扩容的容量返回
          if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA)
              // DEFAULT_CAPACITY ：默认扩容为 10
              return Math.max(DEFAULT_CAPACITY, minCapacity);
          // overflow：超出int类型可表示的范围
          // 所以 ArrayList 的容量最大为 2^32-1（MAX_VALUE = 0x7fffffff）
          if (minCapacity < 0)
              throw new OutOfMemoryError();
          return minCapacity;
      }
      // 如果新 分配的空间 大于 MAX_ARRAY_SIZE（0x7fffffff-8）
      // 但未溢出时，为其分配 最大空间 MAX_VALUE（0x7fffffff）
      return (newCapacity - MAX_ARRAY_SIZE <= 0) ? newCapacity : hugeCapacity(minCapacity);
  }
  // --------------------------------------------------------------------------
  // 当确定扩容的容量后，ArrayList 调用 Arrays.copyOf()方法将数据从原数组中拷贝到新的数组中，该方法底层采用System.arraycopy()方法进行进行数组的深拷贝！

  // --------------------------------------------------------------------------
  // ArrayList 线程不安全，与之对应的是 Vector 是线程安全的，其内部使用 synchorized 保证单个修改操作的原子性！
  ```

+ 缩容

  1. trimToSize() 方法：对外提供缩容接口！

     ```
     · 通过 trimToSize 方法将 ArrayList 的容量缩小为当前元素的个数，以减少 ArrayList 的内存占用。
     ```

  2. remove() 方法：自己内部缩容

     ```java
     public E remove(int index) {
         Objects.checkIndex(index, size);

         modCount++;
         E oldValue = elementData(index);

         int numMoved = size - index - 1;
         if (numMoved > 0){
             // 使用 System.arraycopy 方法，将结点之后的数据，都往前移动一位
             System.arraycopy(elementData, index+1, elementData, index, numMoved);
         }
         // 至此，数组最后一位就是多余的！
         // ArrayList 将其置空，帮助 GC 回收，同时将数组大小 -1 ！
         elementData[--size] = null; 

         return oldValue;
     }
     ```

##### Vector

+ 扩容 2 倍

+ 直接在方法上加锁，实现线程安全性。

  与之对应的就是 Collections.synchronizedArrayList() 实现的是 方法中的同步块！

##### CopyOnWriteArrayList

+ 写时复制，也是线程安全的，是一种乐观的同步措施，读取写入都不加锁，在修改/写入 时，拷贝一个新数组进行修改，再替换旧数组，不会阻塞线程！
+ 缺点：无法保证实时性，读写线程可能读取到旧数据！

##### LinkedList

+ LinkedList 实现了 List 和 Queue 接口，是双向链表的实现
+ 它提供了 头部以及尾部的 插入和删除 方法，使得 LinkedList 既能用作 栈也能用作队列！

###### 链表 特点

+ 物理上非连续，非顺序的储存结构，结点之间通过引用相关联
+ 一个链表上包含1个或者多个结点，每个结点至少有一个`数据域`和一个`引用域`
+ 链表的优点：数据增删快，不需要修改其余结点（数组在增加或者删除某个元素时需要移动其余元素）
+ 链表的缺点：数据查询慢，需要`O(n)`（数组通过下标访问元素：只需要`O(1)`）
+ 详见：[Java-链表](https://www.cnblogs.com/782687539-nanfu/p/10333031.html)

###### ArrayList vs LinkedList

```java
// --------------------------------------------------------------------------
// ArrayList
public class ArrayList<E>
    extends AbstractList<E>
    implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{...}

// --------------------------------------------------------------------------
// LinkedList
public class LinkedList<E>
    extends AbstractSequentialList<E>
    implements List<E>, Deque<E>, Cloneable, java.io.Serializable
{...}

/*
实现上，LinkedList 实现了 Deque 接口，使得其可以作为 双端队列使用
ArrayList 内部使用了 动态数组 的数据结构保存数据：支持动态扩容、缩容
LinkedList 内部使用 链表/结点 的数据结构保存数据

操作上：
		ArrayList				LinkedList
增删	   慢：需要移动其他元素		快：O(1)
查询	   快：O(1)：下标查询		   慢：O(n)：需要遍历各个结点

ArrayList 
	动态扩容，缩容是一项内存优化策略，也是它的缺点，因为每次容量的增减，都会涉及到整个数组的深拷贝！
	支持对元素进行快速的随机访问！
	适合随机查找和遍历，不适合插入和删除元素
LinkedList
	适合动态的插入和删除元素，不适合随机访问和遍历！
	它实现了 Deque<T>双端队列 接口，具有有专门用来操作表头和表尾元素的结点，所以非常适合当作 堆、栈、队列、双向队列使用！
*/
```

##### HashMap

###### HashMap 介绍

```java
/*
HashMap 底层采用 Hash 表实现，也就是 数组+链表的 实现方式，在 jdk1.8 开始，当链表达到一定的长度时，就会转换成 红黑树！
	当 哈希桶 中元素的个数 大于等于 8 时，就会自动转换为 红黑树！
	当 哈希桶 中元素的个数 小于等于 6 时，就会自动由红黑树转换为 链表！

Hash表 是一种 结合了 数组和链表 各自优势的数据结构，它寻址容易元素查找方便，元素增删快！

jdk1.8 以前：HashMap内部的 Hash 表=数组+链表
jdk1.8 开始：HashMap内部的 Hash 表=数组+链表/红黑树

为什么使用 红黑树替换链表？
*/
```

###### hash表 拉链法

+ 平均访问次数：

  ```
  若 一共有 N 个元素，M 个 entry，则装载因子为 a=N/M ，即每个 entry 的平均长度，而根据顺序查找表，不得出查找失败需要 a+1
  ```

  详见：[Hash查找中拉链法查找失败的平均探查次数1+a的证明](https://www.xuebuyuan.com/836410.html)

+ 另外还有 *开放定址法* 散列的方式！

###### equals vs hashCode

+ 如果重新定义了 equals 就要重新定义 hashCode方法，保证 equals 方法与 hashCode 方法的**语义相同**！

  即：如果 *a.equals.(b) = true* 则 *a.hashCode() = b.hashCode()* 成立！

  ```
   · equals 和 hashCode 方法都是 Object 提供的方法！
  	equals ：比较两个 对象的引用！
  	hashCode ：默认为该对象的存储地址！

   · 如果重写了 equals 方法，那么 equals 判断相等的两个 对象，可能并不是同一个对象，因此，导致 equals 和原始的 hashCode 的语义不统一！
  ```

+ **为什么要语义相同**：

  + hashCode 方法通常用于 将元素存入 hash 表时，计算元素的 hash码 ，从而获取 元素所在桶的位置

    *hash%(桶数-1)*

    ```
    一个好的hash函数，可以将元素均匀的哈希到整个 hash表中！
    ```

  + 当存入到 hash 表的元素增多时，多个元素会被 哈希到同一个桶中，则认为这些元素可能相等，对于未 hash 到同一个桶中的元素，则肯定不相等！

    ```
    为保证同一个桶中的元素都不相等，则在插入元素的时候，需要利用 equals 方法将待存入的元素与桶内的元素相比较，若 equals 方法结果相等，则表示桶内的元素存在与待插入的元素相等！

    equals 方法保证了 同一个桶中的 元素 不相等，hashcode 保证了相等的元素一定会被 hash 到同一个桶中！
    ```

+ 注意：

  1. 对象相等则hashCode一定相等；

  2. hashCode相等对象未必相等

     hashCode 是所有 java 对象的固有方法，如果不重载的话， 返回的实际上是该对象在 jvm 的堆上的内存地址，而不同对象的内存地址肯定不同，所以这个 hashCode 也就肯定不同了。 

     如果重载了的话，由于采用的算法的问题，有可能导致两个不同对象的hashCode相同

###### 初始化

+ 默认初始化的**容量 大小为 16**，默认**填装因子为 0.75**，初始化 **扩容阈值为 capacity*loadFactor=12**

  ```java
  // 默认的初始化容量：16，必须是 2的幂次
  static final int DEFAULT_INITIAL_CAPACITY = 1 << 4;

  public HashMap() {
      // 初始化容量为16，填装因子 0.75，但是一个 空 map
      this.loadFactor = DEFAULT_LOAD_FACTOR;
  }
  ```

+ 为什么 容量必须是 2 的幂次？

  + 能使用 二进制位运算 高效寻址，从而代替十进制下的取模运算!

    ```
     · hachCode(key) & (capacity -1)
    	当 capacity 为 2的幂次 时，(capacity -1) 二进制全为 1
    	key 的hash值 与之求与，便等效于十进制下的 取模 ！
    ```

  + 使得元素 分布更加均匀！

    ```
    2 的幂次，能够在 hash 时充分利用 元素哈希码的 后 n 位，以元素的 后n位 哈希码决定 hash 到的桶的位置！
    ```

###### 元素插入

+ jdk1.8 以前：数据的插入采用 “ *头插法*  ”
+ jdk1.8 开始：数据的插入采用 “ *尾插法*  ”

###### 头插 - 线程不安全

+ 元素总是插入到 Hash桶 的头部，由于每个 Hash桶 都是 链表，所以头插法，只需要替换 Hash桶 中的 头结点即可！（这是因为 代码作者认为，新插入的值被查找的可能性较大，以提升效率）

+ 缺点：

  ```
  在多线程环境下，头插法容易造成两个元素之间的循环引用，使得其他线程访问到这两个元素时，造成 死循环 。
  这是因为，扩容前后，同一个桶中的元素的相对位置是相反的，很可能在 扩容rehash 期间，由其他线程的影响导致，两个元素之间相互引用！
  ```

###### 尾插

+ 元素总是插入到 Hash桶的 尾部：扩容前后能保证结点之间的相对位置！

###### 插入

```java
/*
插入：
	如果存在，直接返回
	如果不存在，插入新结点，判断是否需要扩容！
判断存在：
	通过 key 的hash值找到对应的 hash桶！
	首先：判断 hash桶 首结点 是否为目标结点！
	否则：判断 hash桶 首结点 是否为 树结点
		如果是树结点，进入树，遍历插入！
	否则：说明 hash桶 为链表结构！
		遍历该链表！
		如果 链表中 目标结点存在：直接返回！
		如果 链表中 目标结点不存在：插入！同时判断该链表是否需要树化（8个元素即需要树化：红黑树）
扩容：
	 如果为新结点：插入元素：如果此时 HahMap 中元素的实际大小 > 界限threshold值：扩容！
*/

final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
               boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    if ((tab = table) == null || (n = tab.length) == 0)
        // 初始化扩容！
        n = (tab = resize()).length;
    if ((p = tab[i = (n - 1) & hash]) == null)
		tab[i] = newNode(hash, key, value, null);
    else {
        Node<K,V> e; K k;
        if (p.hash == hash && ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;
        else if (p instanceof TreeNode)
            // 说明：桶 是红黑树（桶的头结点 是 TreeNode 的实例）
            // 桶已经被树化，此时需要在这颗树种插入结点！
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        else {
            // 说明：桶 还是链表！
            // 循环遍历 hash桶：插入结点，如果结点已经存在，则返回
            for (int binCount = 0; ; ++binCount) {
                // 判断当前结点是否为尾结点
                if ((e = p.next) == null) {
                    // 尾结点 插入新元素！
                    p.next = newNode(hash, key, value, null);
                    
                    // 插入后，判断 桶的大小 是否达到 8，如果达到
                    if (binCount >= TREEIFY_THRESHOLD - 1)
                        // 将桶 树化（红黑树）！
                        treeifyBin(tab, hash);
                    break;
                }
                // 判断当前结点 是否为所要插入的元素
                if (e.hash == hash && ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                p = e;
            }
        }
        // HashMap 中存在 结点 ：直接返回：不需要判断扩容！
        if (e != null) {
            V oldValue = e.value;
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            afterNodeAccess(e);
            return oldValue;
        }
    }
    ++modCount;
    // 说明 该结点 为新结点：判断是否需要扩容！
    if (++size > threshold)
        resize();
    afterNodeInsertion(evict);
    return null;
}
```

##### 树化

+ [HashMap 树化](https://www.cnblogs.com/finite/p/8251587.html)

###### 寻址

+ **`hachCode(key) & (capacity -1)` 高效位运算取代了十进制的取模！**

+ 取模寻址：保证了只要 key 的hashcode本身分布均匀，Hash算法的结果就是均匀的！

###### 扩容

+  **`capacity << 1`扩大两倍！**

+  **何时扩容**：HashMap 中**元素的总个数** > **capacity*loadFactory**  ：扩容！

```java
final Node<K, V>[] resize() {
    Node<K, V>[] oldTab = table;
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    int oldThr = threshold;
    int newCap, newThr = 0;
    if (oldCap > 0) {
        if (oldCap >= MAXIMUM_CAPACITY) {
            // 桶数量 的最大限制：Integer.MAX_VALUE
            threshold = Integer.MAX_VALUE;
            return oldTab;
        } else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                oldCap >= DEFAULT_INITIAL_CAPACITY)
            // 新桶总数大小：扩容两倍 -------------------------------------------
            newThr = oldThr << 1; 
    } else if (oldThr > 0)
        // initial capacity was placed in threshold：初始化设置
        newCap = oldThr;
    else {
        // zero initial threshold signifies using defaults：初始化设置
        newCap = DEFAULT_INITIAL_CAPACITY;
        newThr = (int) (DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    }
    if (newThr == 0) {
        // 新阈值
        float ft = (float) newCap * loadFactor;
        // 最大阈值为 2^32-1
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float) MAXIMUM_CAPACITY ?
                (int) ft : Integer.MAX_VALUE);
    }
    // 设置阈值
    threshold = newThr;
    @SuppressWarnings({"rawtypes", "unchecked"})
    Node<K, V>[] newTab = (Node<K, V>[]) new Node[newCap];
    table = newTab;
    if (oldTab != null) {
        // 遍历每个 hash桶 对根据新的 hash规则 调整元素
        for (int j = 0; j < oldCap; ++j) {
            Node<K, V> e;
            if ((e = oldTab[j]) != null) {
                oldTab[j] = null;
                if (e.next == null)
                    newTab[e.hash & (newCap - 1)] = e;
                else if (e instanceof TreeNode)
                    // 对 树 结点调整！
                    ((TreeNode<K, V>) e).split(this, newTab, j, oldCap);
                else { 
                    // 对链表结点 调整：该桶内的元素，根据新的规则：要么还在该桶内，要么在 oldBinIndex+oldCap 所在桶内！
                    // preserve order
                    Node<K, V> loHead = null, loTail = null;
                    Node<K, V> hiHead = null, hiTail = null;
                    Node<K, V> next;
                    do {
                        next = e.next;
                        if ((e.hash & oldCap) == 0) {
                            if (loTail == null)
                                loHead = e;
                            else
                                loTail.next = e;
                            loTail = e;
                        } else {
                            if (hiTail == null)
                                hiHead = e;
                            else
                                hiTail.next = e;
                            hiTail = e;
                        }
                    } while ((e = next) != null);
                    if (loTail != null) {
                        // 原结点就在原桶内
                        loTail.next = null;
                        newTab[j] = loHead;
                    }
                    if (hiTail != null) {
                        // 原结点在 新桶内
                        hiTail.next = null;
                        newTab[j + oldCap] = hiHead;
                    }
                }
            }
        }
    }
    return newTab;
}
```

###### 扩容相关问题

1. **为什么 扩容>=8 而 缩容<=6 ？**

   中间间隔一个 7 ，是为了防止 HashMap 频繁的 树化和非树化 的转换，即：在 桶容量为 8 时，频繁的 插入删除 操作，就会导致频繁的 树化和非树化！

2. **为什么 会用到 红黑树？**

   当 哈希冲突 比较大时，多个元素哈希到同一个桶中，那么，如果使用链表的话，桶的查询就是 O(N)，而使用 红黑树的话，查询时间就是 O(logN)

   当桶的容量达到 8 时树化，是为了性能优化！

3. **为什么 不直接使用 红黑树，而要等到 8 呢？**

   因为，红黑树需要自平衡，如果达不到定义的规则，那么就需要变色+旋转！

   当元素个数较少时，旋转和变色 其实是没有必要的，直接使用 链表也能在很短的时间内查询，这就免去了 自平衡所带来的 性能消耗！

###### 线程安全性

+ HashMap 是线程不安全的 ：更新丢失

  它并没有保证每个对数据的修改操作是原子性的，多线程场景下，无法保证同一线程上一秒 put 的数据，下一秒 get 还能得到！

+ 在 jdk 1.7 中使用了头插法，在扩容时，并发的线程会导致 桶中的元素 引用倒置，形成循环引用的场景，而如果下一个线程访问到这个元素，那么将进入 死循环 ！


###### HashMap 的 key

+ **如果让你实现一个自定义的class作为HashMap的key该如何实现？**
  + 重写hashcode和equals方法注意什么
    + 两个对象想等，hashCode值一定相等
    + 两个对象不等，hashCode值可能相等
    + hashcode相等，两个对象不一定相等
    + hashcode不等，两个对象一定不等
  + 如何设计一个不变类
    + 类添加final修饰符，保证类不被继承。
    + 保证所有成员变量必须私有，并且加上final修饰
    + 不提供改变成员变量的方法，包括setter
    + 通过构造器初始化所有成员，进行深拷贝(deep copy)
    + 在getter方法中，不要直接返回对象本身，而是克隆对象，并返回对象的拷贝

##### TreeMap

+ 基于红黑树实现，内部有序，存取时间复杂度都是 O(logn)

+ 有序：

  ```
  1. key 实现 Comparator 接口
  2. 构造 treemap 时传入，通过自定义 Comparator 接口是实现类，或者通过方法引用的方式，传入可使用的compare 方法
  ```

##### LinkedHashMap

+ [LRU 算法详解](https://mp.weixin.qq.com/s/KgXE195lmX0fUrqLBMWbqA)