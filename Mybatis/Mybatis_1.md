#### Mybatis

##### 概述

+ 介绍

  ```
   · MyBatis 是一个 半ORM（对象关系映射）的持久层框架，支持自定义 SQL、存储过程 以及 高级映射。
   	MyBatis 内部封装了 JDBC，对外免除了 JDBC 代码的编写、参数设置 以及 获取结果集等工作。
   	通过 XML配置文件 或者 注解的方式，将 POJO 对象映射为数据库中的记录，避免了 JDBC 代码手动设置参数、获取结果集等繁琐的步骤；并将 SQL语句 与接口方法一一绑定，最终调用接口方法就会触发 SQL 语句的执行，并返回结果集！
  ```

##### 框架对比

+ *Hibernate*

  **优点：**

  ```
   1. 全自动的 对象关系映射 框架，会自动生成 SQL语句，并根据对象关系模型 自动将结果集 转换为 对应的类型！
   	使得开发人员 不再关注 数据库层面的 SQL编写，将重心放在 业务逻辑上，提高开发效率！
   	
   2. 能够做到 数据库无关，可移植性好！
  ```

  **缺点：**

  ```
  1. 不支持编写 动态SQL，它是对 JDBC 的深度封装，灵活性更低，难以优化！
  2. 查询默认会查询出全部字段，可以查询指定字段，但程序较为繁琐！
  ```

  **其他：**

  ```
   · JPA 基于 Hibernate 实现，它底层封装的是 Hibernate
   · 学习门槛高，精通门槛更高！
  ```

+ *MyBatis*

  **优点：**

  ```
   1. 半 ORM 框架，手写 SQL语句，并且支持动态 SQL语句！
   	灵活性高，能够按照开发者的意愿优化查询，提升查询性能！
  ```

  **缺点：**

  ```
   1. 无法做到数据库无关，可以移植性差！（sql 语句是与 数据库相关的）
   	如果需要支持多种数据库，则需要自定义多套 SQL映射文件，工作量大！
  ```

##### 核心组件

+ **SqlSessionFactoryBuilder**

  ```
   · SqlSessionFactoryBuilder 是一个 SqlSessionFactory 的构建器。
   	SqlSessionFactoryBuilder 通过 加载解析 XML 配置文件(mybatis-config.xml)或者通过一个 预先配置的 Configuration 实例 来构建 SqlSessionFactory 实例！
  ```

  ```
   · 配置文件：
   	mybatis-config.xml 配置文件（或者 Configuration 配置类）定义了 MyBatis 的核心配置。包括：数据源配置、事务管理器 等内容！
   	1. 数据源：数据库连接
   	2. 事务管理器：决定事务作用域 以及 事务的控制方式
  ```

  ```
   · 生命周期：
   	当 SqlSessionFactory 被构建完之后，构建器就可以被丢弃了，因此 构建器作用域 一般仅限于方法中，甚至是某个代码块中！
  ```

+ **SqlSessionFactory**

  ```
   · 基于 MyBatis 的应用都以一个 SqlSessionFactory 实例为核心，它用于创建 会话(SqlSession)
   	程序每次访问数据库都需要通过 SqlSession 实例完成！
  ```

  ```
   · 生命周期：
   	SqlSessionFactory 一旦被创建就应该存在于 程序的整个运行期，用于为每一个线程创建一个 SqlSession。
   	因此，采用单例模式创建 SqlSessionFactory 实例，为整个 Web 应用程序服务！
  ```

+ **SqlSession**

  ```
   · SqlSession 提供了在数据库执行 SQL 命令所需要的所有方法！可以通过 SqlSession 实例直接执行已经映射的 SQL 语句！
  ```

  示例：

  ```java
  // 通过 mapper.xml 配置文件的 命令空间+id 的方式调用 SQL 映射语句！
  try (SqlSession session = sqlSessionFactory.openSession()) {
    Blog blog = (Blog)session.selectOne("org.mybatis.example.BlogMapper.selectBlog", 101);
  }
  // 或者：
  // 通过 Mapper 接口调用 SQL 映射语句！
  try (SqlSession session = sqlSessionFactory.openSession()) {
    BlogMapper mapper = session.getMapper(BlogMapper.class);
    Blog blog = mapper.selectBlog(101);
  }
  ```

  ```
   · 生命周期：
   	每个线程都应该有自己的 SqlSession 实例，它是线程不安全的，不能被共享。
   	因此 SqlSession 的最佳作用域是 request作用域 甚至是 方法作用域！
   
   · 每次创建的 SqlSession 对象都必须即使关闭，否则会占用数据库连接池的活动资源，影响系统性能！可以在 try 代码块中打开 SqlSession，在 finally 代码块中关闭 SqlSession
  ```

+ **Mapper**

  ```
   · Mapper 也叫映射器，由 Java接口 和 xml配置文件（注解）共同组成，给出了对应的 SQL 和 映射规则，主要负责提供 SQL语句，并返回结果集。
   
   · Mapper 实例由 SqlSession 实例创建
  ```

  获取 mapper 实例：

  ```java
  XXMapper xxMapper = sqlSession.getMapper(XXMapper.class);
  ```

  ```
   · 生命周期：
   	映射器实例应该在调用 它们的方法中 被获取，并且使用完毕之后即可丢弃！
   	因此，应该将其作用域限制在方法中，甚至限制在 代码块中！
  ```

##### 工作机制

1. MyBatis 通过声明一个 Mapper 接口，与 mapper.xml 配置文件一一对应！

   ```
    · xml 文件将 接口的全限定名作为命名空间，通过接口方法作为 映射SQL语句的 唯一ID，实现与指定的 Mapper 接口绑定！
   ```

2. 程序中 通过注入 Mapper 接口的实现类实例（JDK 动态代理），实现接口方法的调用！再通过接口的**全限定名 + 方法名** 到 mapper.xml 配置文件中查询指定的 SQL映射语句！

   *注意：*

   ```
    1. Mapper 接口方法不能重载！
    	因为是按照 接口全限定名+接口方法名 的方式查询对应的 SQL语句！
    
    2. XML配置文件中的 mapper_id 能否重复？
    	如果 xml 文件中定义了命令空间，则不同命令空间中的 id 可以重复，相同命令空间内的 id 不能重复！
    	如果 xml 文件中没有定义 命令空间，则 id 不能重复！
   ```

3. 最后执行 SQL，并将结果集 按照 返回值类型 进行映射！

##### \# vs $

+ [#{} 与 ${}](https://blog.csdn.net/siwuxie095/article/details/79190856)

+ **相同点：**

  都用来为`SQL映射文件`中的`sql语句`传递参数！

+ **不同点：**

  + **#{}** 表示**占位符 "?"**

    ```
     · 会有预编译过程，预编译过后 #{} 参数会使用 "?" 代替，编译之后将参数当作 字符串(引号括起来) 代替占位符 ?，最后执行！
     
     · #{} 能够有效的防止 sql注入，参数不会被当作 sql语句 编译的部分，sql语义固定！
    ```

  + **${}** 表示**字符串拼接**

    ```
     · 将参数与 SQL 语句直接拼接，并编译，编译之后便会执行 SQL！
     
     · ${} 可能会发生 sql注入，参数会当作 sql语句 编译的一部分，可能会改变 sql的语义！
    ```

##### 缓存机制

+ MyBatis 缓存机制分为 **一级缓存** 和 **二级缓存**

+ *一级缓存：*

  + 一级缓存是 同一个 SqlSession 中的缓存！
  + 基于 HashMap 的本地缓存，其存储作用域为 Session，当 Session flush 或者 close 之后，这个 Session 中所有的缓存将被清除！
  + `默认打开`的是 一级缓存！

+ *二级缓存：*

  + 二级缓存是 同一个 SqlSessionFactory 中的缓存！

  + 与一级缓存相似，默认采用的是 HashMap 的本地存储！

    不同之处在于，二级缓存的存储作用域为 Mapper(Namespace)，并且可以自定义存储源！
