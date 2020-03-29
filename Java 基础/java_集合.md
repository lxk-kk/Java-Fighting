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





#### Map

<img src="image\Map类图.png" style="zoom:67%;" />

##### TreeMap

+ 基于 **红黑树** 实现！

##### HashMap

+ 基于 哈希表 实现！
+ **jdk1.8 之前，哈希表 使用的是 数组链表 的结构，jdk1.8 开始，使用了 数组+链表/红黑树 的结构！**

##### LinkedHashMap

+ 继承于 HashMap，并在 HashMap 的继承上，重写了 结点结构，仅仅为结点新增了 前后继结点的应用，使得 LinkedHashMap 在*以 HashMap 结构存储的基础上，达到 双向链表 的效果*！

+ 使用双向链表维护了 数据插入时的顺序。

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

  将一个接口转换为用户所希望的接口！适配器模式，通过增加一个新类，作为 适配器类，解决接口冲突的问题，使得原本没有关系的类，可以协同工作！

+ java.util.Arrays # **asList() 就使用了适配器模式**，**将数组类型转换为 List 类型**

  ```java
  // 注意 asList 是 泛型的 可变长参数：所以不能使用基本类型
  public static <T> List<T> asList(T... a) {
      // ArrayList 是工具类 Arrays 中自定义的类
      // 相当于 适配器类！
      return new ArrayList<>(a);
  }
  
  // 适配器类
  private static class ArrayList<E> extends AbstractList<E>{
  
      // 适配者：将 泛型数组 转换为 ArrayList
      private final E[] a;
  
      // 泛型数组 为 用户提供！
      // 即：用户提供 适配者
      ArrayList(E[] array) {
          a = Objects.requireNonNull(array);
      }
      // 以下是 通过泛型数组 实现的一些列的 ArrayList 接口！
      // ... ...
  }
  ```

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

##### Vector

##### CopyOnWriteArrayList

##### LinkedList

##### HashMap

##### LinkedHashMap

[LRU 算法详解](https://mp.weixin.qq.com/s/KgXE195lmX0fUrqLBMWbqA)