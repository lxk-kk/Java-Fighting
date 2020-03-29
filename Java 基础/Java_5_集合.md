#### ArrayList



##### ArrayList 扩容次数（扩容机制）

+ 初始化

  + 创建ArrayList对象时可指定初始化容量大小，新建指定容量大小的数据并返回！
  + 如果 不指定 或者 指定容量为0 ：返回空数组（size为零）！

+ 扩容

  ```java
  /**
   * Returns a capacity at least as large as the given minimum capacity.
   * Returns the current capacity increased by 50% if that suffices.
   * Will not return a capacity greater than MAX_ARRAY_SIZE unless
   * the given minimum capacity is greater than MAX_ARRAY_SIZE.
   *
   * @param minCapacity the desired minimum capacity
   * @throws OutOfMemoryError if minCapacity is less than zero
   *
   * 参数 minCapacity 传入的是 当前数组中的元素总个数！
   */
  private int newCapacity(int minCapacity) {
      int oldCapacity = elementData.length;
      
      // 扩容 1.5 倍
      int newCapacity = oldCapacity + (oldCapacity >> 1);
      
      if (newCapacity - minCapacity <= 0) {
          // 如果 为空数组，则 将默认扩容的容量作为此次扩容的容量返回
          if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA)
              // DEFAULT_CAPACITY ：默认扩容为 10
              return Math.max(DEFAULT_CAPACITY, minCapacity);
           // overflow：超出int类型可表示的范围：所以 ArrayList 的容量最大为 2^32-1（MAX_VALUE = 0x7fffffff）
          if (minCapacity < 0)
              throw new OutOfMemoryError();
          return minCapacity;
      }
      // 如果新 分配的空间 大于 MAX_ARRAY_SIZE（0x7fffffff-8）但未溢出时，为其分配 最大空间 MAX_VALUE（0x7fffffff）
      return (newCapacity - MAX_ARRAY_SIZE <= 0)
              ? newCapacity
              : hugeCapacity(minCapacity);
  }
  
  
  // --------------------------------------------------------------------------
  // 当确定扩容的容量后，ArrayList 调用 Arrays.copyOf()方法将数据从原数组中拷贝到新的数组中，该方法底层采用System.arraycopy()方法进行进行数组的深拷贝！
  
  // --------------------------------------------------------------------------
  // ArrayList 的缩容机制：通过 trimToSize 方法将 ArrayList 的容量缩小为当前元素的个数，以减少 ArrayList 的内存占用。
  
  // --------------------------------------------------------------------------
  // ArrayList 线程不安全，与之对应的是 Vector 是线程安全的，其内部使用 synchorized 保证单个修改操作的原子性！
  
  ```

#### 链表

##### 链表的特点

+ 物理上非连续，非顺序的储存结构，结点之间通过引用相关联
+ 一个链表上包含1个或者多个结点，每个结点至少有一个`数据域`和一个`引用域`
+ 链表的优点：数据增删快，不需要修改其余结点（数组在增加或者删除某个元素时需要移动其余元素）
+ 链表的缺点：数据查询慢，需要`O(n)`（数组通过下标访问元素：只需要`O(1)`）
+ 详见：[Java-链表](https://www.cnblogs.com/782687539-nanfu/p/10333031.html)

#### ArrayList 与 LinkedList 的区别（x4）

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

#### arrays.asList是否使用了适配器模式

```java
/*
Arrays.asList(T...a)使用了适配器模式，它可以很好的将数组转换为 集合List ，其返回的集合类是 Arrays 的私有内部类 ArrayList【源码如下】。
该类并没有实现 add/remove/clear 方法，当调用这些方法时会出现 UnsupportedOperationException 异常！
该私有集合类，底层依旧使用的数组实现，并且该数组引用就是所要转换的数组引用
这体现的是 【 适配器模式 ：只转换接口，后台数据仍然是原数组！】

正是由于上述模式：对 返回列表ArrayList 的更改将 “直接写” 到原数组中：因为是同一引用！
*/

// -----------------------------------------------------------------------------------------------
// Arrays.asList(T... a)方法
public static <T> List<T> asList(T... a) {
    return new ArrayList<>(a);
}

// -----------------------------------------------------------------------------------------------
//  Arrays 内部私有类 ArrayList 
private static class ArrayList<E> extends AbstractList<E>
        implements RandomAccess, java.io.Serializable
{
    private static final long serialVersionUID = -2764017481108945198L;
    // 使用 final 修饰，表示该a不可指向其他引用！
    private final E[] a;

    ArrayList(E[] array) {
        // 检测 所要转换的数组是否为 null，如果为 null requireNonNull方法将会抛出 NullPointerException 异常，如果不为 null 则直接将传入的数组 引用赋值给 ArrayList 底层的数组！
        a = Objects.requireNonNull(array);
    }

    @Override
    public int size() {...}
    @Override
    public Object[] toArray() {...}
    @Override
    @SuppressWarnings("unchecked")
    public <T> T[] toArray(T[] a) {...}
    @Override
    public E get(int index) {...}
    @Override
    public E set(int index, E element) {...}
    @Override
    public int indexOf(Object o) {
        E[] a = this.a;
        if (o == null) {
            for (int i = 0; i < a.length; i++)
                // 允许存放 null 元素
                if (a[i] == null)
                    return i;
        } else {
            for (int i = 0; i < a.length; i++)
                if (o.equals(a[i]))
                    return i;
        }
        return -1;
    }

    @Override
    public boolean contains(Object o) {...}
    @Override
    public Spliterator<E> spliterator() {...}
    @Override
    public void forEach(Consumer<? super E> action) {...}

    @Override
    public void replaceAll(UnaryOperator<E> operator) {
        Objects.requireNonNull(operator);
        E[] a = this.a;
        for (int i = 0; i < a.length; i++) {
            a[i] = operator.apply(a[i]);
        }
    }

    @Override
    public void sort(Comparator<? super E> c) {
        Arrays.sort(a, c);
    }

    @Override
    public Iterator<E> iterator() {
        return new ArrayItr<>(a);
    }
}
```

#### HashMap

##### hash表拉链法平均访问次数

```java
/*
若 一共有 N 个元素，M 个 entry，则装载因子为 a=N/M ，即每个 entry 的平均长度，而根据顺序查找表，不得出查找失败需要 a+1
*/
```

详见：[Hash查找中拉链法查找失败的平均探查次数1+a的证明](https://www.xuebuyuan.com/836410.html)

##### HashMap 实现

```java
/*
HashMap 底层采用 Hash 表实现！

Hash表 是一种 结合了 数组和链表 各自极端优势的数据结构，它寻址容易元素查找方便，元素增删快！

jdk1.8 以前：HashMap内部的 Hash 表=数组+链表
jdk1.8 开始：HashMap内部的 Hash 表=数组+链表/红黑树

为什么使用 红黑树替换链表？
*/
```

##### 寻址：`hachCode(key) & (capacity -1)` 高效位运算取代了十进制的取模！

+ 取模寻址：保证了只要 key 的hashcode本身分不均匀，Hash算法的结果就是均匀的！

##### 初始化

+ 默认初始化的容量 大小为 16，默认填装因子为 0.75，初始化 扩容阈值为 capacity*loadFactor=12

  ```java
  /**
   * The default initial capacity - MUST be a power of two.
   * 默认的初始化容量：16
   */
  static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16
  
  /**
   * Constructs an empty {@code HashMap} with the default initial capacity
   * (16) and the default load factor (0.75).
   */
  public HashMap() {
      // 这是个空的map，size为零，只不过容量为16
      this.loadFactor = DEFAULT_LOAD_FACTOR;
  }
  ```

+ 为什么 容量必须是 2 的幂次？

  ```java
  /*
  这是为了能使用 二进制位运算 高效寻址，从而代替十进制下的取模运算!
  寻址为：
  	hachCode(key) & (capacity -1)
  	当 capacity 为 2的幂次 时，(capacity -1) 二进制全为 1
  	key 的hash值 与之求与，便等效于十进制下的 取模 ！
  */
  ```

##### 元素插入

```java
/*
jdk1.8 以前：数据的插入采用 “ 头插法 ”
jdk1.8 开始：数据的插入采用 “ 尾插法 ”
*/
```

###### 头插法

+ 元素总是插入到 Hash桶 的头部，由于每个 Hash桶 都是 链表，所以头插法，只需要替换 Hash桶 中的 头结点即可！（这是因为 代码作者认为，新插入的值被查找的可能性较大，以提升效率）

+ 缺点：

  扩容前后无法会改变 结点 之间的相对位置，可能导致 结点之间的 循环引用：形成 环 ！

  ```java
  /*
  扩容前：
  	某个 hash桶 中有A、B结点，并且A->B
  扩容后：
  	由于 采用的是 头插入法，并且 A结点最先被调整，B结点最后被调整，若A、B两个结点被调整到同一个桶中，并且相邻时，B结点将会称为A结点的头结点，此时会出现B->A，由于扩容前 A->B，所以此时出现 A<=>B 相互引用的状态！当HashMap后面的操作需要遍历该hash桶，并且需要经过A、B结点时，悲剧发生：Infinite Loop 无限循环
  */
  ```

###### 尾插法

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
		如果是树结点，进入树中遍历插入！
	否则：说明 hash桶 为链表结构！
		遍历该链表！
		如果 链表中 目标结点存在：直接返回！
		如果 链表中 目标结点不存在：插入！同时判断该链表是否需要树化（8个元素即需要树化：红黑树）
扩容：
	 如果为新结点：插入元素：如果此时 HahMap 中元素的实际大小达到 界限threshold值：扩容！
*/

/**
 * Implements Map.put and related methods
 *
 * @param hash hash for key
 * @param key the key
 * @param value the value to put
 * @param onlyIfAbsent if true, don't change existing value
 * @param evict if false, the table is in creation mode.
 * @return previous value, or null if none
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

##### 扩容： `capacity << 1`扩大两倍！

何时扩容：HashMap 中元素的总个数 > capacity*loadFactory  ：扩容！

```java
/**
 * Initializes or doubles table size.  If null, allocates in
 * accord with initial capacity target held in field threshold.
 * Otherwise, because we are using power-of-two expansion, the
 * elements from each bin must either stay at same index, or move
 * with a power of two offset in the new table.
 *
 * @return the table
 */
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
            // double threshold：扩大两倍
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
        float ft = (float) newCap * loadFactor;
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float) MAXIMUM_CAPACITY ?
                (int) ft : Integer.MAX_VALUE);
    }
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

##### 为什么重写 equals 方法时需要重写 hashcode 方法？

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
  
  建议使用 null 安全的 Objects.equals、ObjectS.hashCode！
  Objects.hash 是一个可变参数的方法，它会根据传入的参数，对各个参数调用 Objects.hashCode 方法，并组合这些 哈希码 ！
  如果使用数组类型，可以使用静态的 Arrays.hashCode 方法计算该数组的 hash码，该hash码由各个元素的 hash码组成。
  
  如果重新定义了equals方法，就必须重新定义hashcode方法，以便开发人员可以将对象插入到散列表中！
  equals 方法的语意 和 hashCode 方法的语意 必须一致！
  	如果 x.equals(y) 为 true 那么 x.hashCode()==y.hashCode() 为 true ！
  */
  ```

##### HashMap 线程安全性

+ HashMap 是线程不安全的！

  它并没有保证每个对数据的修改操作是原子性的，多线程场景下，无法保证同一线程上一秒 put 的数据，下一秒 get 还能得到！

+ 对于多线程场景，可以使用线程安全的 HashTable 、 CurrentHashMap 代替，或者使用 Collections 为集合提供的线程安全的集合：synchronizedXXX。

  synchronizedXXX 内部使用 加锁的方式实现线程安全，HashTable 内部直接对方法 使用 Synchronized加锁 修饰，线程的并发度低，性能较差，所以一般使用 CurrentHashMap 代替 HashTable ！