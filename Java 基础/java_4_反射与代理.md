#### java_反射

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

#### 代理

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

  + 动态代理更加灵活，它是在运行期动态地 利用反射机制 实现指定的接口，生成代理类，创建代理对象！

##### 代理的优点

+ 通过代理，能够对外隐藏 真实角色，这非常适用于需要保证 真实角色 安全的场景！
+ 代理 使得 真实角色 的任务变得更加纯粹，它只需要关心自己的核心任务，一些公共的任务，可以交给代理完成！

#####  静态代理

+ 代理类的已经实现按照开发者的实现，运行时期，直接执行即可

###### 静态代理 缺点

1. 代理角色 和 真实角色 绑定，紧耦合，一个代理类只能够代理一个真实角色类，如果需要代理多个真实角色，则需要增加其他真实角色类的代理模块！

   **导致 随着 类 不断的增多，代理类也会不断的增多，并且具有相同代理功能的代理方法，不能复用，使得代码冗余！**

2. 如果 代理类和真实角色类之间的紧耦合，使得 牵一发而动全身！

   **如果 真实角色 增加或者删除 某些方法，那么代理类也需要跟着改变，不利于代码的维护 和 扩展。**

###### 静态代理 示例

+ 抽象角色

  ```java
  public interface AbstractRole {
      // 抽象行为
      void action();
  }
  ```

+ 真实角色

  ```java
  public class RealRole implements AbstractRole {
      @Override
      public void action() {
          System.out.println("hello i am real person");
      }
  }
  ```

+ 代理角色

  ```java
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
  ```

+ 顾客

  消费 代理角色 提供的服务

  ```java
  public class Client {
      public static void main(String[] args) {
          // 真实角色
          RealRole realRole = new RealRole();
          // 代理角色
          ProxyRole role = new ProxyRole(realRole);
          role.action();
      }
  }
  
  // ------------------------------------------------ 
  // 输出结果：
  i am proxy person, following is the real one!
  hello i am real person
  proxy end
  ```

##### 动态代理

+ **动态代理**

  所说的 "动态" 是针对 `使用 Java 代码实际编写了代理类的` “静态”代理 而言的！

  动态代理，能够在程序运行期间，根据给定的 类或者接口，动态生成代理类，并创建代理对象！

  ```
  动态代理一种 字节码增强技术，它根据指定类或者接口的 字节码，生成新的字节码，创建新的类！
  ```

+ **动态代理 优点：**

  1. 通过动态代理，不需要显示编写代理类！
  2. 实现了在原始类和接口 还未知的情况下，就能确定代理类的行为，使代理类 与 原始类/接口 解耦，增加了代理类的灵活性，以及代理行为的复用性！

+ *实现方式：*

  1. 基于 接口（实现） 的动态代理：JDK 动态代理
  2. 基于 类（继承） 的动态代理：CGLIB 动态代理

+ **JDK vs CGLIB**

  1. JDK 基于接口，代理类 会实现给定的` 一个或者多个`接口。

     CGLIB 基于类，代理类 会继承`某个给定的类`。因此，要求被代理的类 和 方法，不能声明为 final 

  2. JDK 在 调用处理器（InvocationHandler）中 对 被代理的类 做附加的增强行为！

     CGLIB 在 方法拦截器（MethodIntercept）中 对 被代理的类 做附加的增强行为！

  3. JDK 使用 Proxy.newProxyIntance 静态方法，创建代理对象！

     CGLIB 使用 Enhancer 实例，创建代理对象！

  4. JDK 做增强时，所有方法都会调用 同一个 调用处理器 做增强！

     CGLIB 能够 为代理类中的不同方法，注册不同的 方法拦截器！

     ```
     CGLIB 通过 enhancer 可以注册多个的 MethodIntercept，同时，通过 enhancer注册 回调过滤器（CallbackFilter） 可以决定方法使用 哪一个 MethodIntercept 做增强！
     ```

##### JDK 动态代理

###### JDK 代理

+ JDK 动态代理

  在运行时，通过实现一组给定的接口，并实现接口方法，达到代理的目的！

  通常使用在 编译期 无法确定 需要实现哪一个接口时使用！

+ 一个动态代理，一般代表 一类业务：被代理的所有 真实角色，通用同一个 代理增强 逻辑！

  一个动态代理，可以代表多个  AbstractRole 的实现类：实现多组接口！

###### 代理增强

+ 问：既然代理类需要对 原始类 做增强，那么，它是在哪儿做增强的呢？

  答：InvocationHandler（调用处理器）接口中的 invoke() 方法，正是实现 代理增强的地方！

+ InvocationHandler 接口方法 invoke()

  ```java
  public interface InvocationHandler {
      public Object invoke(Object proxy, Method method, Object[] args) throws Throwable;
      /*
       param1 ：proxy ：代理角色：用于调用 表示 代理实例 与 调用处理器相关联！
       paran2 ：method ：所要调用的方法 的 Method对象
       param3 ：args ：方法参数
      */
  }
  ```

+ **实现步骤：**

  1. 定义一个 *调用处理器（InvocationHandler）的接口实现类，并实现 invoke() 方法* ！
  2. 在 invoke() 方法中，**调用 Method 实例的 invoke() 方法**：*表示 原始类方法的调用* ！（反射）
  3. 在 invoke() 方法中，对原始类做 *附加 的代理行为*，增强原始类！

+ 调用处理器 的 invoke() 方法，会被 加入到 代理类中，代替原始类方法的调用！

###### 创建代理

+ 问：现在可以增强原始类了，那么，代理类又是怎么实现的？

  答：**java.lang.reflect.Proxy 的静态方法 newProxyInstance() 正是用来生成代理类，创建代理对象的！**

+ Proxy::newProxyInstance

  ```java
  public static Object newProxyInstance(
      ClassLoader loader,		// 类加载器（null表示使用默认的类加载器）
      Class<?>[] interfaces,	// Class 对象数组：需要实现的 接口数组
      InvocationHandler h)	// 调用处理器	：实现接口方法，并处理接口方法的调用！
      { ... }
  ```

+ **实现：**

  向 newProxyInstance 方法**绑定** *类加载器、所要实现的接口、调用处理器* ，**返回值即为 代理对象**！

  ```java
  MyProxy myProxy = (MyProxy)Proxy.newProxyInstance(
      realRole.getClass().getClassLoader(),
      new Class[]{ AbstractRole.class },
      roleInvocationHandler
  );
  ```

  **本质：**

  ```
   · newProxyInstance 方法会 获取给定的接口的 字节码，并根据 调用处理器，修改这些接口的字节码，生成新的 byte[]，最终通过 绑定好的 类加载器，加载新的 byte[]，生成代理类，最后创建代理对象！
  ```

###### JDK 代理示例

+ 抽象角色

  ```java
  public interface AbstractRole {
      // 抽象行为
      void action();
  }
  ```

+ 真实角色

  ```java
  public class RealRole implements AbstractRole {
      @Override
      public void action() {
          System.out.println("hello i am real person");
      }
  }
  ```

+ 调用处理器

  ```java
  // 一个调用处理器 代表了 一类业务！ 
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
          // 如果 method.invoke(proxy,args)：就会引发无限循环调用！
          // 因为：method.invoke 会调用 proxy.invoke，如果 proxy 为代理对象，那将无限调用！
          method.invoke(abstractRole, args);
          System.out.println("proxy end");
          return null;
      }
  }
  ```

+ 顾客

  ```java
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

##### CGLIG 动态代理

###### CGLIB 代理

+ CGLIB 动态代理，通过 继承被代理的类，并覆盖 该类的方法，实现 该类的代理！

  因为是 继承类 并且 需要覆盖方法，那就不能将 被代理的类或者方法 声明为 final！

###### 代理增强

+ 问：既然代理类需要对 原始类 做增强，那么，它是在哪儿做增强的呢？

  答：**MethodInterceptor 接口的 intercept() 方法，正是 增强原始类的地方！**

+ MethodIntercept 接口方法 intercept()

  ```java
  public interface MethodInterceptor extends Callback {
    Object intercept(Object var1, Method var2, Object[] var3, MethodProxy var4) throws Throwable;
  }
  ```

+ **实现步骤：**
  
  1. 定义一个 *MethodInterceptor接口的实现类，并实现接口方法 intercept()*  ！
  2. 在 intercept() 方法中，调用 MethodProxy 实例的 **invokeSuper() 方法**：*表示调用 原始类的 方法*。
  3. 在 intercept() 方法中，增加 *附加的代理行为，增强原始类*。

###### 创建代理

+ 问：现在可以增强原始类了，那么，代理类又是怎么实现的呢？

+ 答：**Enhancer 正是用来 生成代理类，创建代理对象的！**

+ **实现步骤：**

  1. 定义一个 Enhancer 实例 enhancer

     ```java
     Enhancer enhancer=new Enhancer();
     ```

  2. 为代理类绑定 父类（被代理的类 / 代理类的原始类）

     ```java
     enhancer.setSuperclass(Father.class);
     ```

  3. 为代理类绑定 增强代理的 MethodIntercept接口实现类

     ```java
     enhancer.setCallbacks( new Callback[]{ YourMethodIntercept.class } );
     /*
      · MethodIntercept 继承了 Callback 接口！
     */
     ```

  4. 创建 代理对象

     ```java
     Father sonProxy = (Father)enhancer.create();
     ```

###### CGLIB 代理示例

+ 原始类（父类、被代理的类）

  ```java
  class Person {
      public void eat() {
          System.out.println("我要开始吃饭咯...");
      }
  
      public void play() {
          System.out.println("我要出去玩耍了,,,");
      }
  }
  ```

+ 方法拦截器

  ```java
  // 第一个方法拦截器
  class MyApiInterceptor implements MethodInterceptor {
      @Override
      public Object intercept(Object obj, Method method, Object[] args, MethodProxy proxy) 
          throws Throwable {
          
          // 附加行为
          System.out.println("吃饭前我会先洗手");
          // 原始类方法调用
          Object result = proxy.invokeSuper(obj, args);
          System.out.println("吃完饭我会先休息会儿" );
          return result;
      }
  }
  // 第二个方法拦截器
  class MyApiInterceptorForPlay implements MethodInterceptor {
      @Override
      public Object intercept(Object obj, Method method, Object[] args, MethodProxy proxy) 
          throws Throwable {
          
          // 附加行为
          System.out.println("出去玩我会先带好玩具");
          // 原始类方法调用
          Object result = proxy.invokeSuper(obj, args);
          System.out.println("玩一个小时我就回家了" );
          return result;
      }
  }
  ```

+ 方法拦截器（回调） 的 选择器

  ```java
  class CallbackFilterImpl implements CallbackFilter {
      @Override
      public int accept(Method method) {
          if (method.getName().equals("play"))
              return 1;
          else
              return 0;
      }
  }
  ```

+ 顾客

  ```java
  public class TestCglib {
      public static void main(String[] args) {
          // 方法拦截器数组（回调数组）
          Callback[] callbacks = new Callback[] {
                  new MyApiInterceptor(), new MyApiInterceptorForPlay()
          };
          // --------------------------------------------------------------
          Enhancer enhancer = new Enhancer();
          // 绑定 所要代理的类
          enhancer.setSuperclass(Person.class); 
          // 绑定 方法拦截器
          enhancer.setCallbacks(callbacks);
          // 绑定 回调选择器
          enhancer.setCallbackFilter(new CallbackFilterImpl()); 
          // 生成代理类，创建代理对象
          Person person = (Person) enhancer.create();
          // --------------------------------------------------------------
          person.eat();
          System.out.println("eat then play ? ");
          person.play();
      }
  }
  ```