#### 【java_反射】

##### 什么是反射

+ 在程序运行时，对于任意一个类，都能获取其类信息，对一个任意一个对象，都能调用它的任意一个方法和属性 的程序，称为反射！即能够动态操作java 代码，获取类信息，调用对象方法的程序！

##### 反射 的 作用

+ 在运行时 判断 任意一个类 所具有的 成员变量（类变量、实例变量） 和 方法
+ 在运行时 构造一个类的对象
+ 在运行时 判断任意一个对象所属的类
+ 在运行时 调用任意对象的方法（生成动态代理）

##### 反射 的 应用、优/缺点

```java
/*
【 应用 】

 · 反射应用广泛，atomic 包中的 对象的属性修改类型原子类(AtomicReferenceFieldUpdater) 就是利用了反射机制。
 · 动态代理也使用了反射机制！
 · 反射最重要的用途就是用于开发各种通用框架，例如 Spring 的 xml 配置文件。
*/


/*
【 优点 】
 · 可扩展性：应用程序可以利用 类全限定名 创建可扩展对象的实例，通过这样，使用来自外部的用户自定义的类！
 
 · 类浏览器 和 可视化开发环境：一个类浏览器需要可以枚举类的成员；可视化开发环境（如IDE）可以利用反射中可用的类型信息收益，以辅助开发者编写代码
 
 · 调试器 和 测试工具：调试器需要能够检查一个类里的私有成员；测试工具可以利用反射来自动调用类里定义的 可被发现的 API 定义，以确保一组测试中有较高的代码覆盖率！
*/


/*
【 缺点 】：能不使用反射时就不使用反射！

 · 性能开销：
 	反射涉及了动态类型的解析，JVM 无法对这些代码进行优化。因此，反射的操作效率 要比 非反射操作 低很多。
 	应该避免在 经常被执行的代码 或者 性能要求很高的程序中使用反射，除非非用不可！
 
 · 内部暴露：
 	反射调用方法时可以忽略权限检查，它能访问正常情况下不允许的操作（例如：私有属性、方法的访问），因此使用反射，可能会破坏面向对象的 封装特性，而导致安全问题！

 · 安全限制：
 	使用反射技术必须在一个没有安全限制的环境中执行，如果一个程序必须在有安全限制的环境中运行，如 Applet，就会出现问题！
*/
```

##### 反射 相关类

+ *java.lang.Class*：表示类相关，用于获取`类的相关信息`！

  每个类都有对应的 Class 对象，它是 java源程序经过编译后，在 class 字节码文件 进行类加载 时的 的加载阶段 生成，类加载的类信息被存放到方法区，而 同时会在 java程序中创建一个对应的 Class 对象，表示方法区中该类信息的入口！

+ *java.lang.reflect.Field*：表示类的成员变量相关，用于获取`实例变量和静态变量`

+ *java.lang.reflect.Method*：表示方法相关：用于获取类中的`方法参数`和`方法类型`

+ *java.lang.reflect.Constructor*：表示构造器相关：用于获取构造器的`相关参数和类型`等

#####  获取 Class 对象

1. 通过 `类型名称.class`来获取类对象

   ```java
   /*
    · 如果 T 是 任意的 java 类型（甚至 void 关键字），则 T.class 将代表该类型的 Class 对象。
   
    · 注意：一个 Class 对象实际上表示的是一个类型，而这个类型未必一定是一种类：
   */
   Class voidClass = void.class;	// void 并不是一种类
   Class intClass = int.class;		// int	并不是类
   Class arrayClass = Double[].class;	// Double[]	类数组
   Class randomClass = Random.class;	// Random 的 Class 对象
   ```

2. 通过 `对象.getClass()`来获取类对象

3. 通过 类名加载类 `Class.forName()`：只要名称就能获得到 class

   ```java
   /*
    · 调用静态方法 forName 获取类名对应的 Class 对象
    · 该方法只有在 className 是类名或者是接口名时才能够执行，并且会抛出 checked exception ：ClassNotFoundException
   */
   
   /*
    · 程序启动时，包含 main 方法的类会被加载，它会加载所有需要的类，这些被加载的类又要加载他们需要的类！
    · spring boot 启动类？
   */
   ```

##### 反射 之 类的实例

```java
/*
 · jvm 位每个 类型 都管理了一个 Class 对象，因此可以使用 "=="，实现两个类对象的比较！
 · 判断对象是否为 某个类的实例！
 	Class 中的实例方法：isInstance
*/
/*
 	obj：所要判断的对象
 		 若 obj 传入的是 基本类型的变量，则会自动装箱成包装类！
 */
public native boolean isInstance(Object obj);

// 示例 isInstance：输出结果为 true
public static void main(String[] args) {
    Class classa = Number.class;
    int a = 1;
    System.out.println(classa.isInstance(a));
}


/*
 · 获取指定 类的实例（两种方法）
 	1、Class 中的实例方法：newInstance()
 	  调用 类的默认构造器：无参构造器，如果没有默认构造器，则抛出异常！
 	  
 	  注意：class.newInstance 在 java 9 中被废弃：因为异常抛出的问题（会躲过编译器检查？）
 	  推荐使用：clazz.getDeclaredConstructor().newInstance()（如下）

 	2、Constructor 中的实例方法：newInstance(Object[] args)	
 	  调用 类的有参构造器调用，args 即为传入的参数！
*/

// 获取 String 类实例：方式 1 
Class class=String.class;
Object str=class.newInstance();
// 获取 String 类实例，方式 2 
Class class=String.class;
Construnctor constructor=class.getConstructor(String.class);
Objest str=constructor.newInstance("this method can carry parameters");
```

##### 反射 之 数组实例

```java
/*
 · java.lang.reflect.Array 反射类：能够获取获取数组信息，并操作数组，设置值，获取值，通过反射动态创建新数组等等！
*/
public static Object newInstance(Class<?> componentType, int length)
    throws NegativeArraySizeException {
    // 调用 如下的 newArray 方法
    return newArray(componentType, length);
}
@HotSpotIntrinsicCandidate
private static native Object newArray(Class<?> componentType, int length)
    throws NegativeArraySizeException;
/*
 · newInstance 方法：
 		param1 = 数组元素类型 Class 对象
		param2 = 新数组长度
		
		返回 Object ，需要强制类型转换
 · 能实现通用的 数组 扩/缩容：不仅适用于对象，还适用于 基本类型！
*/

private Object resize(Object array, int length) {
    // 1、获取数组 的 class 对象
    Class arrayClass = array.getClass();
    // 2、判断是否是一个数组
    if (!arrayClass.isArray()) {
        return null;
    }
    // 3、获取数组的元素类型
    Class elementType = arrayClass.getComponentType();
    // 4、通过反射创建新数组：初始时刻 都为 null
    Object newArray = Array.newInstance(elementType, length);
    // 利用反射获取 array 的数组长度！
    int oldLength = Array.getLength(array);
    // 5、复制值：为赋值的元素依旧为 null
    System.arraycopy(array, 0, newArray, 0, Math.min(oldLength, length));
    // 返回 Object 类型：Object 是所有类的超类，当然也是 数组的超类！
    return newArray;
}

/*
【 优势 1 】
 · 注意：可以将 子类 赋值给超类，但是 若一开始就是一个超类，则不能赋值给子类
 · 所以：上述 传入的 Type[] 可以转换为 Object，之后又通过反射获取 Type
 · 而：下面这种 resize 就是不正确的
【 优势 2 】
 · 可以将 Type[] 传递给 Object，所以尽管 type 是基本类型，也同样是适用！
*/

private Object[] resize(Object[] array, int length) {
    Object[] newArray = new Object[length];
    int oldLength = array.length;
    System.arraycopy(array, 0, newArray, 0, Math.min(oldLength, length));
    return newArray;
}
/*
 · 上面这种 resize 方式：可以将 Type[] 暂时传递给 Object[] array
 	但是 newArray 一开始就是 Object[] ，将其返回 将不能转换为 Type[] !
   只能在遍历 Object[] 数组时，根据元素类型强制转换！
 · 对于 Object[] 类型，只能将 对象数组 赋值给它，然而基本数据类型的数组，却无能为力！
*/
```

##### 反射 之 类的方法

```java
/*
 · Class 类中的实例方法：获取某个 Class 对象的方法
*/

/*
 1、getDeclaredMethods 方法：返回仅在 类 或者 接口声明的所有方法，包括公共、保护、默认访问（包） 以及 私有方法！但是不包括继承的方法！
*/
public Method[] getDeclaredMethods() throws SecurityException {}

/*
 2、getMethods 方法：返回某个类中所有的公共方法（public），包括继承的方法！
*/
public Method[] getMethods() throws SecurityException {}


/*
 3、getMethod 方法：根据参数，返回一个指定的方法：param1=方法名	param2=参数类型数组（可变长参数）
*/
public Method getMethod(String name, Class<?>... parameterTypes){}
```

##### 反射 之 类字段属性

```java
/*
 · Class 类中的实例方法：获取类的成员变量
*/


/*
 1、getField 方法：根据字段名称，访问公有（public）成员变量，包括继承字段！
*/
public Field getField(String name) {}

/*
 2、getDeclaredField 方法：根据字段名称，访问类中声明的成员变量，但不能继承字段！
*/
public Field getDeclaredField(String name){}

/*
 3、getFields 方法：返回类中所有的公有（public）成员变量，包括继承的字段！
*/
public Field[] getFields() throws SecurityException {}

/*s
 4、getDeclaredFields 方法：返回类中所有声明的成员变量，但不包括继承字段！
*/
public Field[] getDeclaredFields() throws SecurityException {}
```

##### 反射 之 方法调用

```java
/*
 · Method 类：invoke 方法，它允许调用 包装在当前 Method 对象中的方法 
*/
public Object invoke(Object obj, Object... args){}
/*
 · 参数：
 	param1 obj：执行该方法的目标对象（即方法的调用者）；若方法为 静态方法，则可省略，设置为 null
 	param2 args：方法的参数（如果没有就传递一个 null）
 
 · 注意：
 	如果返回类型是基本类型，invoke 方法会返回其包装器类型！

 · 应用
	在动态代理中将会被用到，动态代理中，每一个代理对象都需要有一个 调用处理器 与之关联，在 调用处理器中需要处理代理方法的调用！正是通过 Method 的实例方法 invoke！
*/

// 示例：
public class Solution {
    private int value = 0;

    private Solution(int value) {
        this.value = value;
    }

    public static void main(String[] args) throws NoSuchMethodException {
        Solution[] a = new Solution[3];
        a[0] = new Solution(1);
        a[1] = new Solution(2);
        a[2] = new Solution(3);

        // 获取 Class 对象
        Class<Solution> sol = Solution.class;
        try {
            // 获取类中声明的方法：resize
            // 提供参数类型：Object.class 与 int.class
            Method resize = sol.getDeclaredMethod("resize", Object.class, int.class);
            // 调用方法：由于是静态方法，调用者 填写 null：
            //填写参数 a、6；最后 返回值 Object 类型
            Solution[] newArray = (Solution[]) resize.invoke(null, a, 6);
            for (Solution aNewArray : newArray) {
                if (aNewArray == null) {
                    System.out.print(null + " ");
                } else {
                    System.out.print(aNewArray.value + " ");
                }
            }
        } catch (IllegalAccessException | InvocationTargetException e) {
            e.printStackTrace();
        }
    }

    private static Object resize(Object array, int length) {
        // 获取数组 的 class 对象
        Class arrayClass = array.getClass();
        // 判断是否是一个数组
        if (!arrayClass.isArray()) {
            return null;
        }
        // 获取数组的元素类型
        Class elementType = arrayClass.getComponentType();
        // 通过反射创建新数组：初始时刻 都为 null
        Object newArray = Array.newInstance(elementType, length);
        int oldLength = Array.getLength(array);
        // 复制值：为赋值的元素依旧为 null
        System.arraycopy(array, 0, newArray, 0, Math.min(oldLength, length));
        // 返回 Object 类型：Object 是所有类的超类，当然也是 数组的超类！
        return newArray;
    }
}
```

+ [深入理解 java 反射【源码解析】](https://www.sczyh30.com/posts/Java/java-reflection-2/)

#### 【代理】

##### 介绍

+ 代理：是一种结构型设计模式，其目的在于为其他 类的对象 提供一个中间代理，避免 该类对象被直接访问，同时还能为该 类对象 添加其他附加的业务！代理可以为其代理的对象，添加公共业务的功能，例如：记录日志、安全保障、信息拦截与过滤等等！

+ 代理 相关的 四个角色：

  1. `抽象角色（AbstractSubject）`：使用 接口、抽象类实现 —— 用于声明通用行为

  2. `真实角色（RealSubject）`：继承于抽象角色，实现自己的行为 —— 被代理的角色

  3. `代理角色（Proxy）`：继承于抽象角色 —— 代理真实角色，并在真实角色的行为基础上，符加其他行为

     根据`代理角色`的`实现方式`，可以分为*动态代理、静态代理*

  4. `顾客（Client）`：消费 代理者 提供的服务！

  ![](image\代理.png)

+ *动态代理*  和 *静态代理* ：两者都实现代理工作，区别在于实现方式的不同，如下：

  + 静态代理是事前先编写好代理的程序，经编译后，在运行期执行代理角色！

  + 动态代理更加灵活，它是在运行期动态地 利用反射机制 实现指定的接口，并创建代理对象！

##### 代理的优点

```java
/*
 · 代理的优点：
 	1、使得 真实角色 处理业务更加纯粹，不用再关注一些公共的事情，将公共业务，交给代理完成！
 	2、公共业务发生改变或者需要修改时，代码的改动变得更加集中和方便，只需要再代理类中修改即可！

*/
```



#####  静态代理

+ 代理类的已经实现按照开发者的实现，运行时期，直接执行即可

```java
/*
 · 静态代理的缺点：
	1、代理角色 与 真实角色 绑定，是紧耦合的关系，说明一个代理类 只能代理一个 真实角色类，如果需要代理多个，则需要增加其他真实角色类的代理模块！类会随着业务的变更不断的增多，不利于代码的维护与扩展！
	
	2、如果需要增加 抽象角色 中的接口方法，那么，所有的代理类都需要进行相应的增加，不利于代码的扩展！
*/

/*
 · 代码示例：
*/

// 抽象角色
public interface AbstractRole {
    /**
     * 抽象行为
     */
    void action();
}
// 真实角色
public class RealRole implements AbstractRole {
    @Override
    public void action() {
        System.out.println("hello i am real person");
    }
}

// 代理角色
public class ProxyRole implements AbstractRole {
    // 封装真实角色（写死：紧耦合：一个代理类只能代理一个真实角色类）
    private RealRole realRole;

    ProxyRole(RealRole role) {
        this.realRole = role;
    }

    // 在真实角色的行为上，添加自己的附属行为（打个广告啥的）
    @Override
    public void action() {
        System.out.println("i am proxy person, following is the real one!");
        realRole.action();
        System.out.println("proxy end");
    }
}

// 顾客：消费 代理角色 提供的服务
public class Client {
    public static void main(String[] args) {
        // 真实角色
        RealRole realRole = new RealRole();
        // 代理角色
        ProxyRole role = new ProxyRole(realRole);
        role.action();
    }
}
// 输出结果：
i am proxy person, following is the real one!
hello i am real person
proxy end
```



##### 动态代理

###### 介绍

```java
/*
· 动态代理：
 	在运行期，利用反射机制，实现抽象角色接口，动态的创建代理对象！
 	动态代理的所扮演的角色和静态代理一样，但是它比静态代理更加灵活通用！

· 动态代理的实现方式：
 	1、基于接口的实现方式：	
 		jdk 动态代理
 		
 	2、基于类的实现方式：
 		cglib 动态代理
*/
```

###### JDK 的 动态代理

```java
/*
 · 代理类 java.util.reflect.Proxy 
 · 作用：
 	利用动态代理，可以在程序运行时创建一个 实现了一组给定 [ 接口 ] 的代理类对象！
 	这种功能只有在编译时无法确定需要实现哪个接口时才有必要使用！
 	
 · 一个动态代理，一般代表一类业务
 	一个动态代理，可以代表多个 AbstractRole 的实现类
*/
```

###### 代理类 之 方法实现

```java
/*
 · 由上述，代理类需要对 真实角色 进行代理，那就需要在内部处理 真实角色的方法调用，还需要处理公共业务，这是如何实现的呢？
 	答：
		通过 调用处理器（invocation hendler） 实现！
		调用处理器 是 实现了 InvocationHandler接口 的类对象，该接口只声明了一个 invoke 方法。
		我们正是要在 invoke 方法中，处理 真实角色的 方法的调用，并且 实现公共业务的处理！

 · 无论何时调用代理对象的方法，调用处理器的 invoke 方法都会被调用，处理 真实角色方法调用的同时，附加公共业务！
*/

// InvocationHandler 接口中只有 invoke 一个方法！
public interface InvocationHandler {
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable;
    /*
     param1 ：proxy ：代理角色：用于调用 表示 代理实例 与 调用处理器相关联！
     paran2 ：method ：所要调用的方法 的 Method对象
     param3 ：args ：方法参数
     
     通过这 3 个参数，就能利用反射机制，调用指定的方法：Object result=method.invoke(obj,args);
    */
}
```

###### 代理类 之 创建代理对象

```java
/*
 · 由上述，代理类会在运行时，动态生成 代理对象，那是怎么实现的？
   答：
   	使用 Proxy 的静态方法 newProxyInstance 创建代理对象
   	newProxyInstance 方法返回指定接口的代理对象，每个代理实例都要关联一个 调用处理程序，因为 对 被代理角色的所有操作，都在调用处理器中完成！因此，方法调用会分派给指定的调用处理程序！
*/
public static Object newProxyInstance(
    ClassLoader loader,		// 类加载器（null表示使用默认的类加载器）
    Class<?>[] interfaces,	// Class 对象数组：需要实现的 接口数组
    InvocationHandler h) {	// 调用处理器	：实现接口方法，并处理接口方法的调用！
}
```

###### 动态代理  之 代码示例

```java
/*
 · 代码示例：
*/

// 抽象角色
public interface AbstractRole {
    /**
     * 抽象行为
     */
    void action();
}
// 真实角色
public class RealRole implements AbstractRole {
    @Override
    public void action() {
        System.out.println("hello i am real person");
    }
}


// 实现自己的 invocation handler：是通用代理的实现：能代理多个 AbstractRole 的实现类！
public class DynamicProxyInvocationHandler implements InvocationHandler {
    private AbstractRole abstractRole;

    DynamicProxyInvocationHandler(AbstractRole role) {
        this.abstractRole = role;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("hello, this is dynamic proxy executing ... ... ");
        System.out.println("Following is the real one!");
        // 反射：注意：这里应该填入被代理的真实角色的对象，而不是 代理角色的对象
        // 如果 method.invoke(proxy,args)：就会引发无限循环调用：他会调用 proxy.infoke，就是本方法，然后一直循环调用！
        method.invoke(abstractRole, args);
        System.out.println("proxy end");
        return null;
    }
}

// Client
public class Client {
    public static void main(String[] args) {
        // 真实角色
        RealRole realRole = new RealRole();
        // 调用处理器
        DynamicProxyInvocationHandler handler = new DynamicProxyInvocationHandler(realRole);
        // 动态生成代理类对象
        AbstractRole proxy = (AbstractRole) Proxy.newProxyInstance(
            realRole.getClass().getClassLoader(), 
            new Class[]{AbstractRole.class},
            handler
        );
        // 代理执行
        proxy.action();
    }
}

```

######  动态代理 之 典型应用

+ aop：spring 的切面编程
+ [详见：SpringAOP 源码入手 面向切面编程](https://www.jianshu.com/p/7638a236b8d9)