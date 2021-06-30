
### Spring

#### Spring 概念

+ **Spring 是一个轻量级的 IOC 和 AOP 容器框架，它为 Java 应用程序提供基础服务，使得 使开发者专注于业务开发，简化了 企业应用程序的开发！**

+ 优点：

  1. Spring 的 *依赖注入（IOC/DI）*将对象之间的依赖关系交给了 容器处理，实现各个组件之间的低耦合性！

  2. Spring 的 *面向切面编程（AOP）*将应用程序中 通用的/公共的 任务 从核心业务中抽离出来，实现了 公共业务 的复用性 和 可维护性，同时也使得 开发人员可以专注于 核心业务的开发！

  3. Spring *集成了多种优秀的框架*，提供对 Struts、Hibernate、MyBatis、Quartz 等框架的直接支持，代码的侵入性低！

     ![](image\Spring 支持的框架.png)

+ 主要模块

  ​		<img src="image\Spring 核心模块.webp" style="zoom: 67%;" />

  + *Spring Core*

    ```
     · 是 Spring 的核心容器，主要提供 IOC 依赖注入服务！
     	核心容器的核心组件是 BeanFactory，它是工厂模式的实现，BeanFactory 使用 控制反转（IOC）将应用程序的 配置和依赖性规范 与 实际的应用程序代码分离！
     	
     	Spring 容器通过 配置文件或者注解 描述对象之间的依赖关系，自动完成对象的创建和对象外部资源的注入！
    ```

  + *Spring Context*

    ```
     · 为 Spring 应用程序提供 的上下文信息，还提供了企业级功能（JNDI、电子邮件、定时任务等）
    ```

  + *Spring AOP* 

    ```
     · 提供 面向切面编程 的服务
    ```

  + *Spring DAO*（Data Access Object）

    ```
     · 对 JDBC 进行抽象封装，简化了数据访问。
     · 提供了 统一的异常体系，用于管理不同数据库厂商抛出的异常信息，向业务层 屏蔽了 不同持久化技术的异常差异！
    ```

  + *Spring ORM*

    ```
     · 对现有的 ORM 持久层框架提供支持，包括：Hibernate、MyBatis、JDO 等
     	这些 ORM 框架都遵从 Spring 的 通用事务 和 DAO异常体系！
    ```

  + *Spring Web*

    ```
     · 它基于 应用程序上下文（spring context）模块之上，为创建 Web 应用程序 提供 上下文信息！
    ```

  + *Spring MVC*

    ```
     · 提供面向 Web 应用程序的 Model-View-Controller 实现
    ```

#### Spring IOC/DI

##### IOC/DI

+ IOC 即 控制反转：**将 Bean 对象的控制权（创建、管理）由程序 转变为 第三方资源管理器（容器）**！

  ```
  以前，我们需要在程序中 编写对象的创建过程，而现在，这项艰巨的任务交给了 Spring 容器！
  Spring 容器会根据 配置文件和注解 自动为我们创建 Bean 对象，并注入该对象所需要的所有外部资源！
  ```

+ DI 即 依赖注入：**在程序运行期间，依赖于容器 动态为 Bean对象 注入所需要的外部资源**！

  DI 和 IOC 只是相同概念的不同角度的描述！

  ```
  IOC：Bean 对象的控制权 交给谁 —— 容器
  DI ：Bean 之间的依赖关系 依赖于谁实现 —— 容器
  ```

+ [大佬们的讨论](https://www.zhihu.com/question/23277575)

##### DI 配置

+ 注解配置（自动配置）
+ Java Config 配置
+ XML 文件配置

###### 注解 配置

+ 通过在 对象定义上 添加注解 @Autowired 或者 @Resource 实现该该对象的注入！

+ 如下注解，用于定义一个**组件类**，各自本质都相同，代表的含义不同而已！
  + **@Controller** ：表示这是 controller 层的 Bean 对象

  + **@Service** ：表示这是 service 层的 Bean 对象

  + **@Repository** ：表示这是 持久层的 Bean 对象

  + **@Component** ：表示这是自定义的组件，通过 @Component 标记，能够让 IOC 容器自动分类

      [源码追踪：@Component 与 @Configuration 区别](www.freesion.com/article/494413591/) 

+ 最后在 Spring 配置文件中配置 \<context:annotation-config/> 元素开启注解

  ```
  在需要该 Bean 对象的地方，添加注解 @ComponentScan(basePackage="xxx.xxx")
  或者在配置文件中配置 <context:component-scan base-package="xxx.xxx">
  ```

  *Spring 会根据 @ComponentScan 注解 或者 \<context:component-scan> 元素扫描指定的类，并为其生成 Bean 对象，加入到 IOC 容器中！*

  ```
  补充：SpringBoot 中的包扫描

   · SpringBoot 启动时，会自动扫描 启动类所在的包，及其子包下的所有组件！
   	因此，在上述包中，标记了 @Component、@Controller、@Service、@Repository 的类，都会被扫描到，并为这些类 创建 Bean对象，加入到 IOC 容器中！
   	
   · 若需要扫描其他包中的组件类，则需要在启动类上，手动添加 @ComponentScan 注解，指定需要扫描的包！
   
   · SpringBoot 启动时，还会自动扫描 配置类，并将配置类中 所有的 Bean对象，添加到 IOC容器中！
   
   · 此外，在启动类上添加 @ServletComponentScan 注解后，Servlet、Filter、Listener 可以直接通过 @WebServlet、@WebFilter、@WebListener 注解自动注册！
   	@ServletComponentScan 注解默认不用加 Servler、Filter、Listener 所在路径，除非他们与启动类不在同一个包下！
  ```

###### Java Config 配置

+ **@Configuration** 用于定义**配置类**，代替 XML 配置文件！

  ```xml
  <!-- @Configuration 配置类相当于  XML 配置文件中的 beans 元素-->
  <beans ...>
      ...
  </beans>
  ```

+ **@Bean** 与 @Configuration 搭配使用，用于**定义 Bean对象**，代替 XML 中的 \<bean> 元素

  ```xml
  <bean id="..." class="xxx.xxx">
      <property name="..." value="..."></property>
      ...
  </bean>
  ```

+ *Spring 会自动扫描被 @Configuration 注释的类，并将配置类中的 Bean对象，添加到 IOC 容器中进行管理！*

  [详见：Java Config 配置](https://www.cnblogs.com/duanxz/p/7493276.html)

##### 注入 的方式

+ 依赖注入 / Bean装配  的方式：

  1. **构造器注入**
  2. **setter 注入**
  3. 工厂方法注入
  4. 接口注入

+ 最常用的就是 构造器注入 和 setter注入！

  **构造器 注入：用于注入 强依赖**

  **setter 注入：用于注入 可选择依赖**

###### 构造器注入

+ 在加载上下文时（初始化容器时），通过构造方法 注入 Bean 的属性值 或者 依赖的对象，它保证了 Bean 对象在实例之后就可以使用！

+ 构造器注入在 **\<constructor-arg>  元素**中声明属性（\<constructor-arg> 中没有 name 属性）

+ 示例：

  ```xml
  <bean id="constructor_bean_injection" class="com.xxx.xxx.Person" scope="prototype">
      <constructor-arg value="lxk" index="0"></constructor-arg>
      <constructor-arg value="22"></constructor-arg>
      
      <!-- 也可以使用 value 元素来注入值 -->
      <constructor-arg type="int">
          <value>22</value>
      </constructor-arg>
  </bean>
  <!--
  scope：作用域
  value：要注入的 值
  index：参数下标
  type：指定类型
  -->
  ```

###### 属性（setter）注入

+ 通过 setter 方法注入 Bean 的属性值 或者 依赖的对象！

  容器会调用 类的 无参构造器 或者 无参static工厂方法 实例化 Bean，再调用该 bean 的setter 方法，实现注入！

+ 在 **\<property> 元素**中，使用 name 属性指定 Bean 的属性名称，使用 value属性 或者 \<value>子元素 指定属性的值！

+ 示例：

  ```xml
  <bean id="setter_bean_injection" class="com.xxx.xxx.Person" scope="singlton">
      <property name="username" value="lxk"></property>
      <property name="age" value="22"></property>
  </bean>
  <!--
   上述 setter 方法应该为如下定义：
  	public void setUsername(String name){...}
  	public void setAge(int age){...}
  -->
  ```

##### 循环依赖

+ [源码追踪_1：Spring 循环依赖](https://www.jianshu.com/p/8bb67ca11831) | [源码追踪_2：Spring 循环依赖](https://www.cnblogs.com/zzq6032010/p/11406405.html) | [源码追踪_3:Spring 循环依赖](https://cloud.tencent.com/developer/article/1497692)

+ 循环依赖：

  A 引用 B，B 引用 A 的情况！

+ Spring 有两种循环依赖：

  构造器注入 循环依赖

  setter注入 循环依赖

###### 构造器注入 循环依赖

+ 类之间的依赖关系，通过 构造器 的形式构建！

  ```java
  @Service
  public class A{
      public A(B b){}
  }
  @Service
  public class B{
      public B(A a){}
  }
  ```

+ 这种 循环依赖的方式 是**无法解决**的，直接抛出 **BeanCurrentlyInCreationException** 异常！

+ **原因：**

  *对 JVM 而言*

  ```
   · 创建对象时，需要调用构造方法实例化对象，而由于 循环引用的原因，构造方法中 对象参数，无法提前被创建，因此，只能抛出异常！
  ```

  *对 Spring 而言*

  ```
   · Spring 是通过 对象创建的中间状态解决 循环依赖的！(中间状态：对象被 空（默认）构造器创建，但是还未真正的对属性进行初始化)
   · 而，构造器注入的方式，是在创建对象的同时，需要对属性进行初始化，所以，构造器注入的方式无法解决循环依赖！
  ```

###### setter注入 循环依赖

+ 类之间的依赖关系，通过 set 方法构建！

+ **单例（singleton）的 setter 注入循环依赖，Spring 采用 Bean创建的 三级缓存 解决！**

  **多例（prototype）的 setter 注入循环依赖，Spring 依旧无法解决，直接抛出 BeanCurrentlyInCreationException 异常！**

  ****

+ **单例循环依赖 解决：**

  + 上文提到：

    ```
    1. setter 注入的特点：首先调用 空构造器 创建对象，再调用 setter 方法注入 外部资源！
    2. Spring 解决循环依赖，依赖于 对象创建的中间状态！
    ```

  为保存 对象创建的中间状态，Spring 为 Bean对象的创建，提供了 **三级缓存**：

  1. 一级缓存：保存使用 空构造器 创建的对象：被空构造器创建的对象，就会存入该缓存！
  2. 二级缓存：保存正在 通过set 方法初始化的对象：正在通过 set 注入属性的对象，就会存入该缓存！
  3. 三级缓存：保存初始化完成的 Bean：对象正式创建完成后，就会放入这个缓存！

  *解决过程：*

  1. 首先，从第三级中尝试获取 Bean A，没有的话 再从 第二级中获取，还是没有，则从 第一级中获取。

     若三级都没有，说明 对象A 还未被 空构造器创建，执行下一步！

  2. 使用 空构造器创建 对象A，将A 存入缓存中！

  3. set 方法注入 依赖的对象B，而此时 依赖的对象B 还未创建，则触发 对象B 的创建，同时将A 存入缓存中！

  4. 重复第1、2步，构造对象B！

  5. 通过 set 方法注入依赖的对象A，而此时，A还未被正式创建，尝试创建对象A ！

  6. 执行上述 第1步，**由于，A 已经被空构造器创建，因此，已经存在于 缓存中，可以直接取出并返回！**

  7. **从缓存中获取 A对象，并完成 B 的 set 注入！**

  8. B对象已经正式创建完成，此时，可以正式创建A对象！

  ****

+ **多例循环依赖 异常：**

  + 单例对象的循环依赖 之所以能够解决，主要在于，对象中间态的保存，而也正是单例模式下，能够保证 对象保存是有效的！

    即：已经通过 空构造器创建的对象，下次尝试再次创建时，该对象能够当作已经创建的结果，并返回！

    而，**多例模式下，每一次都要求创建新的 对象，自然，缓存对象不再起作用，那也就无法解决循环依赖问题！**

    那，*多例循环依赖，如何检测呢？*

  + *检测：*

    对于多例对象，创建过程中 Spring 会维护一个 *ThreadLocal，其 value 为一个 set，用于保存当前线程中，正在创建的 Bean 对象的 名称* ！

    **当通过该 set 检测到，即将要创建的 Bean已经处于创建状态中，则说明，发生了循环依赖，抛出异常！**

###### 循环依赖 解决方案

1. **重新设计**

   采用更好的 层次结构 代替循环依赖！

   如果一定需要循环依赖，则还有如下解决方法！

2. **setter 单例注入 循环依赖**（Spring 自带解决方案）

3. **@Lazy**

   ```
    · 在构造方法的参数上，使用 @Lazy 表示延迟加载所要注入的Bean！
    · 当 Spring 检测到，字段上添加了 @Lazy 注解时，并不会直接创建该 Bean对象，而是为其生成一个 代理类！
    · 当真正需要 该Bean对象时，才会真正的创建 Bean！
   ```

   ```
    · 解决 构造器注入循环依赖：
    	1. 检测到 参数对象为懒加载模式，为参数对象 生成一个 代理类！
    	2. 使用代理类，注入正在构造的 Bean对象中！
    
    · 此时，就不是 Bean 对象之间的循环依赖了，而是 Bean 与 代理类之间的依赖关系！
    	而，真正所要 注入的对象，会在使用时，才会创建！
    	能够巧妙的避开 构造器必须同时相互构造的问题！
   ```

4. **@PostConstruct**

   + 标记在方法上，表示该方法会在 Servler 构造器之后执行！

     **具体执行时间：在  @Autowired注入 之后执行！**

   + *补充 @Autowired 知识*

     ```
      · 补充 @Autowired 注入：
      	1. @Autowired 加在 属性变量 上，表示 setter 注入！
      	2. @Autowired 加在 构造器上，表示 构造器 注入！
      
      · 建议加在 构造器上，使用 构造器注入的方式，这能避免出现 NPE 问题！
     ```

     @Autowired setter 注入的 NPE 问题：

     ```java
     public class ControllerTest{
         @Autowired
         private Service service;
         private Object obj;
         
         public ControllerTest(){
             this.obj=service.obj;
         }
     }
     /*
      · 执行上述代码，出现 NPE 问题！
      
      · 原因：
      	setter注入 在 构造器之后执行，因此，通过构造器初始化 obj属性时，service 还未注入！
      	此时，报错 NPE！
      	
      · 解决办法：使用 构造器注入！
     */
     public class ControllerTest{
         private Service service;
         private Object obj;
         
         @Autowired
         public ControllerTest(Service service){
             this.service=service;
             this.obj=service.obj;
         }
     }
     ```

   + 使用 @PostConstruct 解决循环依赖问题：

     ```java
     /*
      · 为 CircularDependencyA 注入 CircularDependencyB ！
      · 注入 CircularDependencyB 之后，通过 @PostConstruct 将 CircularDependencyA 设置到 circB中！
     */
     @Component
     public class CircularDependencyA {
      
         @Autowired
         private CircularDependencyB circB;
      
         @PostConstruct
         public void init() {
             circB.setCircA(this);
         }
      
         public CircularDependencyB getCircB() {
             return circB;
         }
     }
     /*
      ·只将 CircularDependencyB 声明为组件，并不为其注入 CircularDependencyA，circA 通过被注入的 circB 的 set 方法设置！
     */
     @Component
     public class CircularDependencyB {
      
         private CircularDependencyA circA;
          
         private String message = "Hi!";
      
         public void setCircA(CircularDependencyA circA) {
             this.circA = circA;
         }
          
         public String getMessage() {
             return message;
         }
     }
     ```

   + **补充：@PostConstruct 实现 静态Bean对象！**

     ```
      · 为什么需要使用 @PostConstruct ？
      	因为，Spring的依赖注入，是基于对象实现的，即：是将 外部资源 注入到 Bean对象中！
      	如果 将资源声明为 static 那么，该资源将属于 类，而不再是属于具体的对象，因此，无法实现依赖注入（不会报错，但是为 null，一但使用，就会NPE ！）
      	
      	因此，可以注入 非静态的对象，再通过 @PostConstruct 注解的方法，将 非静态的Bean 赋值给 静态的 对象！ 
     ```

     ```java
     @Component
     public class Util{
         // 注入 Bean
         @Autowired
         private Service service;
         
         // 定义 静态对象
         private static Service serviceStatic;
         
         // 偷梁换柱：在 @PostConstruct 注解的方法中，将已注入的 Bean 赋值给静态的 对象！
         @PostConstruct
         private void init(){
             serviceStatic=service;
         }
         // 静态方法：只能使用静态的 对象！
         public static void test(){
             System.out.println(serviceStatic);
         }
     }
     ```

5. **实现 ApplicationContextAware 和 InitializingBean**

+ [详见：循环依赖 解决方案](https://blog.csdn.net/bingguang1993/article/details/88915576)

#### Spring AOP

##### AOP 概念

+ AOP（Aspect oriented programming）：面向切面编程

  *重构 服务层 代码，将原本存在于 服务层 中的 公共业务 抽离，单独作为一个模块，并将其以 切面增强的 形式 添加到 核心业务 周围。使得服务层 专注于 核心业务的开发，而 公共业务 的代码 得以复用！*

+ **AOP 常见概念**

  1. *连接点（Joinpoint）*：类中需要被 增强 的方法！

  2. *切入点（PointCut）*：根据所要附加的公共业务的不同，对 连接点 进行分类，每一类称为 一个切入点！

  3. *通知/增强（Advice）*：所要增强的行为、所要增加的公共业务（日志、安全、事务）

     + 前置通知（before）：在 连接点之前（方法执行之前） 添加的 通知！

     + 后置通知（after）：在连接点之后（方法执行之后）添加的 通知！

     + 异常通知：在连接点发生异常（方法抛出异常时）添加的 通知！

     + 最终通知：在 连接点 退出之后（包括正常退出、异常退出）添加的 通知！

       ----

     + 环绕通知（Around）：在通知中 调用 连接点！（上述 4 种都是在 通知 外执行连接点）

       ```
        · 环绕通知中，可以 在调用连接点 之前或者之后，添加公共业务、增强行为，也可以决定是否执行连接点！
        
        · 注意：前置通知只有在抛出异常后，连接点才不会被执行！
       ```

  4. *切面 （Aspect）*：切入点 和 通知 构成一个切面！

  5. *引介（Introduction）*：引介 是一种特殊的通知！

     ```
     在不修改类代码的前提下，Introduction 可以在运行期，动态地 为类 添加一些方法或者属性！
     ```

     ----

  6. *目标（Target）*：被代理的 目标对象！

  7. *织入（Weaving）*：为`目标(Target)`添加`通知(公共业务)`的过程，即：增强 目标对象 的过程！

  8. *代理（Proxy）*：是 Spring 运行时动态创建的对象，用于代理 目标对象！

     ```
      · 为 目标对象 织入 通知后，Spring 会为目标对象创建代理对象，它代表了 被增强后的目标对象！
      	此后，当访问 被增强的方法 时，其实，都是在调用 代理对象中 增强后的版本，而不是直接访问原始的目标对象！
     ```

##### AOP 实现

+ AOP 是基于 **动态代理** 实现的，有 JDK动态代理 和 CGLIB动态代理 两种实现方式！

  + JDK 动态代理：基于 接口（实现），创建代理类！
  + CGLIB 动态代理：基于 类（继承），创建代理类！

  [详见：JDK动态代理 vs CGLIB 动态代理](C:\Users\10652\Desktop\Fighting\temp_over\Java 基础\java_4_反射与代理.md)

+ **Spring AOP 对两种代理的 选择：**Spring会自动在 两种代理实现之间进行转换！

  + *如果目标对象 实现了接口，默认情况下，Spring 会采用 JDK 动态代理实现 AOP*，也可以强制使用 cglib 实现 AOP
  + *如果目标对象 没有实现接口，必须采用 cglib 实现 AOP* ！

#### IOC 初始化过程

+ [源码_清晰明了：Spring IOC](https://yikun.github.io/2015/05/28/Spring-IOC%E6%A0%B8%E5%BF%83%E6%BA%90%E7%A0%81%E5%AD%A6%E4%B9%A0/) | [源码分析：Spring IOC](https://www.javadoop.com/post/spring-ioc) 

+ IOC 容器

  ```
   · Spring 通过 注解/Java Config/XML 等配置，描述了 Bean 对象之间的依赖关系！
   
   · IOC 容器通过 反射，解析配置，创建 Bean 对象，并建立 Bean对象 之间的依赖关系！
   	除此之外，IOC容器 还提供了 Bean实例缓存、生命周期管理、Bean实例代理、时间发布、资源装载等高级服务！
  ```

+ Bean 的创建过程：`IOC 容器初始化、Bean 的实例化`

  <img src="image\IOC 图解启动过程.jpg" style="zoom: 120%;" />

+ **IOC 容器初始化**

  *IOC 容器的初始化过程，就是对 Bean外部资源的 读取、解析 与 注册！*

  <img src="image\IOC 过程.png" style="zoom:73%;" />

  1. Resource 读取配置：从 配置文件 中读取资源

     ```
      · Spring 使用外部资源描述一个 Bean 对象，因此，IOC 容器第一步就需要读取 外部资源！
      	由 ResourceLoader 的 Resource 接口完成资源的读取！
     ```

  2. BeanDefinition 解析资源

     ```
      · 解析外部资源，并根据 Bean 的配置信息，在容器内部创建对应的 BeanDefinition 对象！
     ```

  3. BeanDefinition 注册

     ```
      · 将 BeanDefinition 注册到 BeanDefinition注册表（BeanDefinitionMap）中！
      	Spring 会在容器中维护一个 BeanDefinitionMap，存储 Beanfinition 对象，方便 Bean 的管理！ 
     ```

+ **Bean 的实例化 与 注入**

  + 当 完成 IOC 初始化之后，如果没有设置 `延迟加载(lazy-init)`，则，立即实例化 Bean 对象！

    ```
     · 根据 BeanDefinition 注册表，加载、实例化 Bean，建立 Bean 与 Bean 之间的依赖关系！
     	最终，将 Bean 存放到容器的 Bean 缓存池！
    ```

  + **默认情况下，`lazy-init` 属性为 `false`**，表示在**容器启动时立即加载 Bean**

    当配置 `lazy-init=true` 时，该 Bean 的创建会在 应用程序第一次 `向容器获取Bean`[`getBean()`] 时发生！

#### Bean 生命周期

#### Bean 作用域 scope

+ Spring 支持 *5 种 Bean 的作用域*

  1. **singleton**（单例）

     *默认作用域*

     ```
      · 在 Spring IOC 容器中仅仅存在一个 Bean 实例，Bean 以单例的方式存在！
      
      · singleton 采用了 单例模式，可以设定 Bean 加载的时间！
      	lazy-init = true ，启动懒加载：在第一次从容器中获取 Bean 时加载 Bean
      	lazy-init = false，立即加载：在容器启动时就加载 Bean
     ```

  2. **prototype**（原型）

     ```
      · 每次获取 Bean 时，都会新创建一个 Bean实例
      	即：每次调用 getBean() 从容器中获取 Bean ，或者将该 Bean 注入到 另一个 Bean 中时，都会新创建一个 Bean！
      
      · prototype 作用域的 Bean，并不会在 容器启动时自动创建，而是在该 Bean 被第一次访问时被创建！
     ```

  3. **request**

     ```
      · 每个 HTTP 请求到来时，都会创建一个新的 Bean，该 Bean 仅仅在当前 HTTP request 中有效！
      	request作用域 仅适用于 WebApplicationContext 环境！（Web 程序）
     ```

  4. **session**

     ```
      · 每次建立一个 session 都会创建一个 Bean 实例，该 Bean 仅仅在当前 session 中有效！
      	session作用域 仅适用于 WebApplicationContext 环境！（Web 程序）
     ```

  5. **global-session**

     ```
      · 在全局的 HTTP Session 中，共享同一个 Bean
     	该作用域 仅适用于 WebApplicationContext 环境！
     	
      · 实际上：global-session 作用域只在 partlet web 应用程序中有意义！
      	partlet 中定义了 全局session 的概念，它被所有构成某个 portlet web 应用程序的各种不同的 portlet 共享！
     ```

+ *单例（singleton）Bean 线程安全问题*

  + 线程安全：共享变量 在各个线程的 工作内存中不能保证状态一致，便出现线程安全问题！

  + **在 Spring 中，大多数 singleton Bean 都是无状态的（例如：DAO、Service、Controller 等一般不包含可变状态），因此对于这些  Bean 而言，不会有线程安全问题！**

    **只有无状态的 Bean 才能够在 多线程环境下共享！**

  + *对于可变属性，有如下处理方式：*

    1. `将 Bean 作用域修改为 session、request、prototype！`
    2. `使用 ThreadLocal 缓存可变属性！`

#### BeanFactory 组件

##### BeanFactory

+ BeanFactory 

  *BeanFactory 是Spring中最顶层的接口，包含了各种 Bean 的定义，用于读取Bean配置文件、管理 Bean 的加载 以及 实例化、控制 Bean 的生命周期、维护 Bean 之间的依赖关系等！*

+ **BeanFactory 采用延迟加载的形式注入 Bean**

  *即：*

  ```
   · 只有在使用到某个 Bean（调用 getBean()）时，才会加载并实例化 Bean 对象！
  ```

  *弊端：*

  ```
   · 不能发现一些存在于 Spring 配置中的问题。例如：如果 Bean 的某个属性没有注入，则 BeanFactory 加载后，直到第一次使用 Bean 时才会抛出异常！
  ```

+ **优点：**

  启动时占用资源少，应用能够快速启动！

+ **缺点：**

  1. 运行速度相对较慢，因为所有的 Bean 加载都放到了运行时期！
  2. 有可能出现 空指针 异常
  3. 由 Bean 工厂 创建的 Bean 生命周期相对简单！

##### ApplicationContext

+ ApplicationContext

  *ApplicationContext 是 BeanFactory 的派生，拥有 BeanFactory 所具有的所有功能，除此之外还提供了更完整的框架功能：*

  1. 继承了 MessageSource ，支持国际化

  2. 提供 统一的资源文件访问方式

     ```
      1. FileSystemXml
      2. ClassPathXml
      3. WebXml
      
      还是说 Resource ？
     ```

  3. 提供 在 Listener 中注册 Bean 的事件（向注册为 监听器的 bean 发布事件）

  4. 提供 同时加载多个配置文件 的功能！

  5. 载入多个（有继承关系）上下文，使得每一个上下文都专注于 一个特定的层次，例如：应用的 Web 层

+ ApplicationContext 的**三种实现方式**：

  1. *FileSystemXmlApplicationContext*

     ```
     提供 XML配置文件 的系统路径（全路径名），从 XML配置文件中读取 上下文！
     ```

  2. *ClassPathXmlApplicationContext*

     ```
     提供 XML配置文件 的类路径，在 classpath 下查找 XML配置文件 读取上下文！
     ```

  3. *WebXmlApplicationContext*

     ```
     读取 Web 应用的 XML配置文件，获取 Web 应用的上下文！
     ```

+ **ApplicationContext 在容器启动时，一次性创建 所有的 Bean**

+ **优点：**

  1. 程序运行时一次性加载所有的 Bean，能够在容器启动时就检查出 Bean 的配置是否错误，有利于检查Bean所依赖的属性是否注入！
  2. 程序运行速度相对较快，在运行期间，不再创建 Bean，需要的时候直接使用！

+ **缺点：**

  1. 程序启动缓慢，因为一次性创建了所有的 Bean
  2. 运行期内存占用较大，难免会浪费内存！

#### Spring 事务

+ Spring  的事务 拥有 事务的 特性！

  ```
  1. 拥有 ACID 四种事务特性：原子性、一致性、隔离性、持久性！
  2. 会出现 并发一致性读问题：修改丢失、脏读、不可重读、幻读！
  3. 拥有事务的隔离级别：四种隔离级别：读未提交、读已提交、可重复读、串行化！
  ```

  [详见：事务](C:\Users\10652\Desktop\Fighting\temp_over\MySQL\数据库_2_系统原理.md) | [Spring 事务管理](https://juejin.im/post/5b00c52ef265da0b95276091#heading-12) | [Spring 支持的两种事务](https://juejin.im/post/5b010f27518825426539ba38)

+ Spring 支持两种 事务 方式：

  1. *编程式事务管理* ：**使用 Transaction Template 编写事务代码，手动提交、回滚！**

  2. *声明式事务管理* ：通过 **注解（@Transactional）、配置文件** 等方式自动为程序 添加 事务功能！

     ```
      · 声明式事务管理：基于 Spring AOP，对方法进行 拦截，为方法 附加事务处理的功能，如果方法抛出异常，将会自动回滚！
     ```

+ 事务选择：

  + **编程式事务管理**

    ```
     · 优点：事务粒度能够达到 代码块 级别！
     
     · 缺点：
    	手动编写代码 处理事务，使得事务的处理 与 核心的业务紧耦合，不能复用 事务处理 的同时，还具有代码侵入性，不利于代码的维护！
    ```

  + **声明式事务管理**

    ```
     · 优点：
     	利用 AOP 织入的方式，将事务处理功能 与 核心业务分离，非侵入性开发：业务分工明确、易于代码的维护和扩展！
     	
     · 缺点：
     	事务粒度 只能够 达到 方法级别：无法做到 编程式事务的细粒度！
    ```

+ *事务传播行为*

  + **当事务方法 被 另一个事务方法 调用时，被调用的方法，应该在自己的事务中执行呢，还是在 外围的事务中执行？这个由事务的传播行为决定！**

    Spring 定义了 7 中事务传播行为：[详解：事务传播行为](https://blog.csdn.net/weixin_39625809/article/details/80707695)

    ```
    解说：
    	当前事务：调用者中的事务
    	传播行为：调用者中的事务 对 被调用者中的事务的影响，传播行为标注在 被调用者事务上！
    ```

  + `支持当前事务`的传播行为：*@Transaction(propagation=“xxx”)*

    1. TransactionDefinition.PROPAGATION_**REQUIRED** ：*最常用！*

       ```
       · 如果当前 存在 事务，则加入该事务；否则，创建一个新事务！
       ```

       ```java
       /*
        · 解释：
       	加入该事务：被调用者 共用 调用者的事务，一起成功/回滚
       	创建一个新事务：调用者中无事务，被调用者自己创建创建一个事务自己执行
       */
       ```

    2. TransactionDefinition.PROPAGATION_**SUPPORTS**

       ```
       如果当前 存在 事务，则加入该事务；否则 以非事务的方法继续执行！
       ```

    3. TransactionDefinition.PROPAGATION_**MANDATORY**

       ```
       如果当前 存在 事务，则加入该事务；否则 抛出异常！（mandatory：强制性）
       ```

    4. TransactionDefinition.PROPAGATION_**NESTED**

       ```
       如果当前 存在 事务，则 在当前事务中 内嵌一个新事务执行程序；
       如果当前 没有 事务，则 事务传播行为 与  REQUIRED 相同！
       ```

       ```java
       /*
        · 解释：
       	嵌套的事务 依赖于 外层事务：嵌套的事务回滚不会影响外部事务的执行，而外层事务发生回滚，则 嵌套的事务 一并回滚！
        · 这是因为：
        	执行嵌套事务的方法前，会设置安全点！当 嵌套事务 执行失败，则外部事务恢复到安全点前的状态，因此，嵌套事务的执行不影响外层事务！而，由于嵌套事务执行完毕之后，还未提交，需要等待 外层事务执行完毕，因此，外层事务回滚时，内层事务也会回滚！
       */
       ```

  + `不支持当前事务`的传播行为：

    1. TransactionDefinition.PROPAGATION_**REQUIRES_NEW**

       ```
       创建一个新事务，如果当前存在事务，则挂起！
       ```

       ```java
       /*
        · 解释：
       	调用者的事务 与 被调用者自己创建的事务 相互独立！
       	当调用者事务 执行到 被调用的方法时，调用者事务将被挂起，被调用的方法会创建一个事务执行，当该事务执行完毕（提交/回滚）之后，调用者事务才会继续执行！各自的回滚/提交，互不影响！
       */
       ```

    2. TransactionDefinition.PROPAGATION_**NOT_SUPPORTED**

       ```
       以非事务的方式执行，如果当前存在事务，则挂起！
       ```

    3. TransactionDefinition.PROPAGATIOM_**NEVER**

       ```
       以非事务的方式执行，如果当前存在事务，则抛出异常！
       ```

###  SpringMVC

#### 简析 消息处理过程

+ SpringMVC 是一种 **基于请求驱动的** *轻量级 Web 层框架* ！

  Spring MVC 的核心是 **DispatcherServlet（请求分发器/前端控制器）**，所有的请求 都会通过 DispatcherServlet 统一分发！

+ **controller 消息处理流程/原理：**

  <img src="image\Spring MVC原理.png" style="zoom: 50%;" />

  1. DispatcherServlet *接收客户端请求*，并准备将 请求 分派给对应的 Controller

  2. 分派请求之前，前端控制器 会 *首先查询 HandlerMapping，定位 Controller*

     ```
     · HandlerMapping 负责完成 客户端请求到 Controller 的映射！
     ```

  3. DispatcherServlet *将请求 分派给 Controller* ！

  4. Controller 调用 业务逻辑*处理请求，返回 ModelAndView* ！

     ```
     · ModelAndView 中包含了 模型（Model）和视图（View）
     ```

  5. DispatcherServlet *根据 ModelAndView 查询 ViewResolver*，*找到指定返回的 视图* ！

  6. 最后，*将 Model 中的数据传递给找到的 视图*，并将结果*显示到客户端* ！

+ 注意：

  ```
   · 从外部来看，DispatcherServlet 是整个 Web 应用的控制器！
   · 从内部来看，Controller 是单个 Http 请求处理的控制器，而 ModelAndView 是请求处理过后返回的 模型和视图！
  ```

#### HandlerMapping

+ 在 SpringMVC 中，任何请求 都对应了一个 处理器（Handler），用来处理请求；

  ```
  · 这个 处理器 是 Object 类型 ，它可以代表 类、方法、对象 等！
  · 在 Controller 层中被 @RequestMapping 标注的方法，就对应于 处理器！
  ```

+ *作用*

  **根据 request 查找 处理器执行链（HandlerExecutionChain），它包含了请求相关的 Handler 和 Interceptors ！**

+ *实现*

  + Spring MVC 采用模板方法模式，设计了 HandlerMapping 的整体结构！在 SpringMVC 中，所有的HandlerMapping 都继承于 AbstractHandlerMapping！

  + SpringMVC 中很多组件采用了 模板方法模式 —— 首先使用一个 抽象类进行模板的整体设计，然后在子类中通过实现 模板方法 完成各自独特的任务！（AQS 也是这种设计！）

+ HandlerMapping 分为 *两个系列*

  1. *AbstractUrlHandlerMapping* 系列

     ```
     通过 url 匹配 HandlerExecutionChain
     ```

  2. *AbstractHandlerMethodMapping* 系列

     ```
      · 这个系列不仅仅是通过 url 来匹配 HandlerExecutionChain，匹配条件还包括 request的类型（GET、POST等）、请求的参数、Header等！

      · 同时，该系列将 Method 作为 Handler 使用，它有一个专门的类型：HandlerMethod！
      	这是我们用的最多的一种 Handler，例如：Controller 中被 @RequestMapping 注释的方法，就是使用这种 Handler！
     ```

#### HandlerAdapter

+ 实际上，请求到达 Controller 层之前，会首先经过 HandlerAdapter！

  <img src="image\Spring MVC原理_2.png" style="zoom: 50%;" />

+ *请求到 controller 的具体过程：*

  1. **当 请求到来时，DispatchServlet 会遍历 HandlerMapping 列表，根据 请求（request）查找对应的 处理器执行链（HandlerExecutionChain）**

     ```
      · HandlerExecutionChain ：处理器执行链
      	包含了与当前 request 相匹配的 Handler ，以及相关的拦截器 interceptor
      	
      · Handler 和 interceptor 的执行：
     	1. 执行时 interceptor 的 preHandle方法，再执行 Handler，将请求交给 Controller 处理！
     	2. 返回时，执行完 Handler 后，再执行 interceptor 的 postHandle 方法
     ```

     <img src="image\Spring MVC 详细原理.png" style="zoom: 67%;" />

  2. **遍历 HandlerAdapter 列表，根据 HandlerExecutionChain 中的 Handler 找到对应的 处理器适配器（HandlerAdapter）！**

  3. **HandlerAdapter 调用 handle 方法处理 Handler，获取最终的 ModelAndView！**

     ```
     HandlerAdapter 中有多种 处理器适配器：
      · RequestMappingHandlerAdapter（继承于 AbstractHandlerMethodAdapter ）
      	能够处理任意方法（最复杂）
      	
      · HttpRequestHandlerAdapter
      	处理 HttpRequestHandler 类型的 handler！
      	主要适配 静态资源处理器，用于处理 静态资源的请求！
      
      · SimpleControllerHandlerAdapter
      	处理 Controller 类型的 handler！
      	即：处理 Controller 层的请求！
      
      · SimpleServletHandlerAdapter 
      	处理 Servlet 类型的 handler
     ```

### SpringBoot

#### SpringBoot 介绍

+ `SpringBoot 基于 Spring4.0 设计，不仅继承了 Spring 框架原有的特性，还通过简化配置进一步简化 Spring 应用的搭建和开发！`

  `但它并不是代替 Spring 的解决方案，而是和 Spring 框架紧密结合，用于提升 Spring 开发者体验的工具`

  SpringBoot 中有两个重要策略：*开箱即用*、*约定优于配置*！

  + **开箱即用：**

    ```
     · SpringBoot 提供了很多的依赖模块，开发者只需要在 pom 文件中添加相关依赖，在程序中以注解的方式就能使用这些模块功能！SpringBoot 会自动将 所需要的 Bean 注入到上下文中！

     · 开箱即用，帮助我们免去大量繁琐的 Bean以及Bean依赖 的 XML 配置！
    ```

  + **约定优于配置：**

    ```
     · 百度百科：约定优于配置 也称作 按照按照约定编程，这是一种软件设计规范！通过约定好的默认配置，减少开发者在开发过程中的抉择次数，而开发人员只需要规定 约定中不适合实际应用 的部分！

     · 例如：
     	实体类 blog ，那对象关系映射时，默认查找名为 blog 的数据库，这就是一种约定！
     	数据库中的 下划线命名方式 和 Java 程序程序中的 驼峰命名 方式，也是一种约定！
     
     · SpringBoot 为我们提供了相关的默认配置，使用默认配置，就能简化开发过程，快速搭建项目！
    ```

    SpringBoot 中的约定：

    1. 约定：MAVEN 的目录结构：

       + 默认 src-main-resources：存放静态资源 及 配置文件
       + 默认 src-main-java：存放 java 源码的文件
       + 默认 target：存放 java 源码编译后生成的字节码文件！

    2. 约定：默认的配置文件名为 application.properties 或者 application.yml，并且配置文件唯一！

    3. 约定：以 starter 的形式减少依赖，因此集成了许多 starter！（第三方模块）

    4. 约定：

       如果添加了 spring-boot-start-web 依赖，就认为这是一个 web 环境，于是自动帮助我们导入 SpringMVC 的相关依赖 和 一个内置的 Tomcat 容器！

+ **特性：**在 Spring 基础上拥有独有的特性：

  1. *创建 独立运行的Spring应用。*（将应用压缩成 jar 包，可以直接运行 jar 包启动应用程序，而不需要另外配置一个 web 服务器）
  2. 能够使用*内嵌的Tomcat*、Jetty或Undertow，*不需要部署war。*
  3. 提供*定制化的启动器starters，简化第三方依赖配置。*（简化 MAVEN 依赖配置）
  4. 追求极致的*自动配置Spring。*
  5. 提供一些*生产环境的特性*，比如特征指标、健康检查和外部配置、应用监控。
  6. *零代码生成和零XML配置*

+ SpringBoot 核心模块

  <img src="image\SpringBoot 核心模块.png" style="zoom: 50%;" />

  1. spring-boot 是 SpringBoot 的核心工程
  2. starters：是 SpringBoot 的启动服务工程！
  3. autoconfigure：是 SpringBoot 实现自动配置的 核心工程！
  4. actuator：提供 SpringBoot 应用的外围支撑性功能。例如：应用状态监控管理、应用健康指标表等
  5. tools：提供了 SpringBoot 开发者的常用工具集

#### 启动过程

+ 启动类上的注解是 **@SpringBootApplication**，是 SpringBoot 的核心注解，包含了如下注解：

  1. *@SpringBootConfiguration* ：组合了 @Configuration 注解，实现配置文件的功能

  2. *@EnableAutoConfiguration* ：打开自动配置的功能！

     ```
      · 可以选择关闭某个模块的自动配置：
     	例如：关闭数据源的自动配置：
     		@SpringBootApplication( exclude={DataSourceAutoConfiguration.class} )
     ```

  3. *@ComponentScan* ：Spring组件扫描

+ SpringBoot 通过 **SpringApplication 的 run()** 方法*启动应用程序* ！它是个静态方法，主要实现分为两个步骤：

  1. **初始化：创建并初始化一个 SpringApplication 对象！**

     初始化流程中最重要的是 通过 *SpringFactoriesLoader* 加载 *spring.factory（META-INF 目录下）* 文件中的 *ApplicationContextInitializer* 和 *ApplicationListener* 两个接口实现类的所有实例！

     ```
      · SpringFactoriesLoader 用于在指定的配置文件 META-INF/spring.factory 中加载配置！
      	spring.factory 是 类的全路径名称！
      	SpringFactoriesLoader 也是 自动配置的根基！
      
      · ApplicationContextInitializer 	
      · ApplicationListener
     ```

  2. **启动：执行 SpringApplication 对象 的 run() 方法！**

+ 详见：[源码：SpringBoot 启动原理](https://www.jianshu.com/p/ef6f0c0de38f) | [注解角度：SpringBoot 启动原理](https://cloud.tencent.com/developer/article/1449134) | [注解+源码：SpringBoot 启动流程分析](https://blog.csdn.net/hfmbook/article/details/100507083) 

#### 自动配置

+ @EnableAutoConfiguration、@Configuration、@ConditionalOnClass 是自动配置的核心，实现了 Spring的自动配置！
