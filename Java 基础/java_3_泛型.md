#### 【泛型】

##### java 中的 泛型 是什么？使用 泛型 有什么好处？

```java
/*
泛型时 jdk1.5 的新特性，用于解决 类型泛化的问题！

在 1.5 以前，java 语言还没有出现 泛型，只能通过 继承和类型强制转化 实现类型泛化，这使得 在编译期间，编译器无法检查出 强制转换是否成功！程序员仅是依赖这个保障类型泛化的正确性，则 ClassCastException 风险将转嫁到 程序的运行期！
*/
```

##### java 的泛型是如何工作的？什么是 类型擦除？

```java
/*
java 的泛型是通过 类型擦除 实现的！
（ 类型检查发生在 编译期 ）

编译器在 编译时擦除了所有的类型信息，类型参数只会在 源程序代码中呈现，在编译后的字节码中，泛型会被替换成原始类型（raw type），并且在相应的地方插入强制转换代码！因此对于 运行期的 java 语言来说，ArrayList<Integer> 与 ArrayList<String> 就是同一个类！

java 通过 这种类型擦除的方式实现泛型 ，实际上是 伪泛型！对于 C# 来说，泛型则是“真实泛型”，不论是在 源程序中还是在 编译后的代码中，类型参数（编译后编程占位符） 都是真实存在的！
*/
```

##### 什么是 泛型中的 限定通配符 和 非限定通配符？

```java
/*
1、限定通配符 用于 允许类型参数 变化，并限定其类型变化：泛型类型必须使用 限定内的类型进行初始化，否则编译出错！
    1）子类型通配符	List<? extends T>
        确保类型必须是 T 的子类型
    2）超类型通配符	List< ? super T>
        确保类型必须是 T 的超类型
2、无限定通配符：List<?> 
	表示元素是一种特定但是未知的类型的列表！
	可以使用 任意类型代替！
	
3、List<?> 与 原始类型的区别？
	本质在于：
	
【注意】
1、"," 用于分隔 类型变量 	"?" 用于分隔 限定类型
2、使用 通配符的 泛型，类型擦除后，原始类型将使用 第一个限定的类型变量替换，如果是无限顶通配符，则使用 Object 替换！
*/
```

##### 如何编写一个泛型方法，并让它能接受 泛型参数并返回泛型类型？

```java
// 定义 普通类中的泛型方法
public <K,V> V put(K key,V value){
    return cache.put(key,value)
}
public <K extends Student, V extends Teacher> V put(K key,V value){
    return cache.put(key,value)
}
```

##### 如何编写泛型类：使用泛型编写带有参数的类

```java
public class Student<T>{
    private T name;
    public T getName(T name){
        return name;
    }
}
```

##### 编写一段 泛型程序 实现 LRU缓存

```java
/*
LinkedHashMap 可以用来实现 固定大小的 LRU 缓存，当 LRU 缓存已经满了，他会把最老的键值移除出缓存。
LinkedHashMap 中的 removeEldestEntry() 方法会被 put() 以及 putAll() 调用来删除最老的键值对。
*/
```

##### 可以把 List\<String> 传递给一个  接受 List\<Object> 参数的方法吗？

```java
/*
不能，List<String> 与 List<Object> 之间没有任何关系，他们的公共父类是 Object ！
*/
```

##### Array 可以用泛型吗？

```java
/*
Array 不支持泛型，建议使用 List 代替 Array，因为 List 可以在编译器提供类型安全的保证，而 Array 不能！
*/
```

##### <?> 与 raw type 的区别

```java
/*
编译器会对 带有带参数的类型进行 类型检查，而不会对原始类型进行类型检查！

<?>
进行类型检查！
所以不能够通过一个 List<?> 的引用，往该list 中添加任何除 null（因为 null 可以赋值给任意对象） 以外的元素，这是因为 无法确定 List<?> 中的元素类型到底是什么，无法保证类型安全！
	List<?> a -> { a.add(new Object()); } ----- compilation error!
	
raw type
不会进行类型检查！
	List a->{ a.add(new Object()); } ----- compilation warn!

raw type 不推荐使用，它的出现只是为了兼容性！
*/
```

##### < Object > 与 raw type 的区别

```java
/*
<Object>
进行类型检查！
以 Object 作为元素类型，告诉编译器该声明 可以 接受任意类型的元素！
所以 List<Object> list=new ArrayList<String>(); ----- compliation error !

raw type
不会进行类型检查！
可以将任意带类型参数的对象，传递给 原始类型！
所以 List list=new ArrayList<String>(); ----- compliation success!
*/
```

##### < Object > 与 < ? >  的区别 

```java
/*
< Object >
类型检查！
以 Object 作为元素类型，告诉编译器该声明 可以 接受任意类型的元素！
所以 List<Object> list=new ArrayList<String>(); ----- compliation error !

<?>
类型检查！
List<?> 表示 List 是未知的，但是其元素是某一个特定类型的！
所以：List<?> list=new ArrayList<String>(); ----- compliation success!
*/
```

##### \<String> 与 raw type 的区别

```java
/*
<String>
带参数类型是安全的，其类型安全由编译器保证！
所以对于List<String> ：不能将 除了String类型以外的其他类型元素加入该 list 中，编译器会报错。正因为此，从该 list 中获取元素时，直接就是 String 类型的，不需要 强制类型转换。 

raw type
原始类型是不安全的！
所以对于List : 可以将任意类型的元素 加入该 list 中！
正因此：从该 list 中获取元素时，由于不确定类型，所以需要使用 强制类型转换！
*/
```



