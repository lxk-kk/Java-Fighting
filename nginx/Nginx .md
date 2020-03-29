#### 【Nginx】

##### Nginx 介绍

+ Nginx 是一个`轻量级且高性能`的 Http 和 反向代理服务器，同时也是一个 IMAP/POP3/SMTP 服务器（电子邮件代理）

##### Nginx 功能

+ 作为 Web 服务器：相比于 Apache ，它占用资源更少，支持更多的并发连接，性能较高！
+ 可以作为 邮件代理服务器：支持 IMAP/POP3/SMTP 等协议
+ 可以作为 反向代理服务器
+ 可以作为简单的 负载均衡器，Nginx 是 七层应用层上的负载均衡服务器，它支持 6 种 负责均衡策略！ 
+ 可以实现动静分离：静态资源可以放到另外 文件服务器 或者直接放到 Ningx 缓冲上，根据配置将静态请求 与 动态请求分离开！

##### 正向代理、反向代理

+ 正向和反向其实就是目标与主体的倒置的区别！
+  [反向代理(nginx) vs 正向代理](https://mp.weixin.qq.com/s/Adr8GkfoMjjbrTsPX86uLw)

###### 正向代理

+ 正向代理（forward proxy）：代理服务器 位于`客户端 和 目标服务器`之间，为了从目标服务器获取内容，客户端向代理发送一个请求，并指定目标服务器，然后代理 向 目标服务器 转交请求并将目标服务器响应的内容，返回给客户端！

+ 客户端 能明确知道 目标服务器的存在，但是，目标服务器，对客户端一无所知，它只对代理服务器响应！

+ 优点：

  + 隐藏客户端真实 IP，免受攻击

  + 提高访问速度

    通常 代理服务器 都设置了一个较大的硬盘缓冲区，会将部分请求的相应保存到缓存中，当代理服务器接收到相同的 访问请求时，就能直接由缓存中取出信息，返回给用户，以提高访问速度！

  + 突破访问限制

    通过代理服务器，用户能够突破自身 ip 的访问限制，例如：科学上网（访问外国网站）。

###### 反向代理

+ 反向代理（reverse proxy）：代理服务器同样是 位于 客户端 与 目标服务器之间，但对 客户端而言，代理服务器就是它的目标服务器，而代理服务器接收到 客户端请求后，会向真正的目标服务器转发请求，并将响应内容返回给 客户端！

+ 客户端 不知道真正的目标服务器的存在，它只和代理服务器通信！

+ 优点：

  + 对外隐藏真实目标服务器，提高内部服务器的安全

    外部网络用户 通过反向代理访问内部服务器，只能看到反向代理服务器的 ip地址 和 端口，内部服务器对于外部网络来说是完全不可见的。

    反向代理服务器 上 没有保存任何的信息资源，所有网页程序都保存在 内部服务器 上，对反向代理服务器的攻击并不能使真正的网页信息系统收到破坏，这样就提高了系统的安全性！

  + 提高 对 内部服务器的访问速度

    反向代理具备缓存功能：能加快用户的访问速度！

  + 节约了有限的 IP 资源

    例如学校网，有多个内部服务器需要向外部提供服务时，由于公网的 ip 数量限，并不能为每一个内部服务器都提供一个公网 ip，此时就能通过反向代理技术，对外提供一个公网 ip，由代理服务器将不同的请求转发到相应的内部服务器上！

  + 负载均衡

    反向代理服务器可以做负载均衡，根据 所有真实服务器（提供同一个服务的服务器） 的负载情况，将客户端请求分发到不同的真实服务器上！

##### 负载均衡

###### 介绍

+ 负载均衡：是一种计算技术，用在`多个计算机（计算机集群）、网络连接、CPU、磁盘驱动器 或其他资源`中 均衡分配负载，以达到`资源的最优化使用，最大化吞吐率、最小的响应时间、同时避免过载`的目的！

+ 应用

  是高性能、单点故障（高可用）、扩展性（水平伸缩）的解决方案！

###### 负载均衡的模型

+ 四层负载均衡

  ​		四层负载均衡工作在 OSI 模型的传输层（由低向上第四层），由于在传输层 只有 TCP/UDP 协议，这两种协议包含了 源IP、目标IP 及对应的 端口号！四层负载均衡服务器 在 接收到客户端请求后，通过修改数据包的地址信息（IP+端口）将流量转发到应用服务器上！

+ 七层负载均衡

  ​		七层负载均衡工作在 OSI 模型的应用层（最顶层），常用 http、radius、dns等。七层负载就可以基于这些协议来负载。这些应用层协议中会包含很多有意义的内容。例如：同一个 WEB 服务器的负载均衡，处理根据 IP+port 进行负载外，还可根据第七层的 URL 、浏览器类别、语言 来决定是否要进行负载均衡！

###### 负载均衡的工具

+ LVS 

  在 四层传输层 实现负载均衡

+ Nginx 

  在 七层应用层 实现负载均衡

+ Keepalived 

  在 七层应用层 实现负载均衡

+ HAProxy

  使用 c 语言编写，能提供高可用、负载均衡、以及基于 TCP和 HTTP 的应用层序处理！

  在 七层应用层 实现负载均衡

###### 负载均衡的算法

+ 轮询（Round Robin）

  顺序 循环地将请求 转发到各个服务器结点上！若其中某个服务器结点故障，则将其取出，不参与轮询，直到其恢复正常为止！

+ 比率（Ratio）

  给每个服务器分配一个加权值，根据这个权值，把用户请求分配到各个服务器结点上，当其中某个服务器发生故障，负载均衡器就会将其取出，不参与下一次轮询，直到恢复为止！

+ 优先权（Priority）

  给所有服务器分组，每个组定义优先权，负载均衡器 将用户请求分配给优先级最高的服务器组，同一个组内采取轮询 或 比率算法，分配用户请求；当最高优先级中所有服务器出现故障时，负载均衡器才会将请求分配到 次优先级的服务器组。这种方式，实际上为用户提供了一种热备份的方式？

+ 最少连接方式（Least Connection）

  将新的请求转发给 那些处理最少连接的服务器，当某个服务器出现故障... ...

+ 最快模式（Fastest）

  将请求转发给响应最快的服务器，当某个服务器故障时... ...

+ 观察模式（Observed）

  连接数目 与 响应时间这两项的最佳平衡，最为新请求的处理服务器

+ ... ...

##### Nginx 负责均衡

###### 配置

+ Nginx 在 upstream 模块中实现 负载均衡的配置！模块内的 server 表示 服务器列表，其后是 应用服务器ip

```nginx
# 负载均衡如下三个应用服务器
upstream dynamic_service {
    server localhost:8080;
    server lcoalhost:8081;
    server localhost:8082;
}

server {
    listen 80;
    server_name localhost;
    # 模块化
    location / {
        # 反向代理 将 / 下的请求，转发到 dynamic_service 中
        proxy_pass dynamic_service;
    }
}
```

###### Nginx 负载均衡 6 中策略

​		`轮询（默认方式）、权重、ip hash、最少连接(least_conn)、响应时间(fair)、依据 URL 分配方式(url_hash)`，其中 fair 和 url_hash 是第三方提供的负载均衡策略，需要安装第三方插件！

1. **轮询**（配置缺省时的默认策略）

   + 最基本的配置就是如上示例，负载均衡器 将会按照 upstream 中配置的顺序 循环地 将请求转发到各个应用服务器上！具有如下参数：

     ```java
     /*
      · fail_timeout 与 max_fails ：两者联合使用
      	设置在 fail_timeout 允许是时间内 的最大失败次数，如果在这个时间内，所有针对该服务器的请求都失败了，则认为该服务器停机了！
      
      · fail_time ：设置服务器 会被 认为停机的时间长度，默认为 10s
      · backup ：标记该服务器为备用服务器，当主服务器停止时，请求会被发送到该服务器结点上！
      · down ：标记服务器永久停机了
     */
     ```

     ```nginx
     upstream dynamic_service {
         server localhost:8080;
         server lcoalhost:8081 backup;
         server localhost:8082 max_timeout=20s max_fails=5;
     }
     ```

   + 在轮询中，如果 服务器 down 了，会自动剔除该服务器！
   + 优点：实现简单，请求均匀分配
   + 缺点：由于后端服务器的性能通常有差异，性能好的能够多承担请求压力，所以请求平均分配不能很好的利用服务器资源！

2. **加权**（wight）

   + 是 轮询的改进方式，轮询是各个服务器结点权值相同的加权轮询！可以根据后端服务器的性能差异，为各个服务器设置权值，性能较好的权值较大，转发请求时，权值便决定了分配请求的比例！

   + 通过 *wight 参数* 配置，其`默认值为 1`

     ```nginx
     # 负载均衡如下三个应用服务器
     upstream dynamic_service {
         server localhost:8080 wight=1;
         server lcoalhost:8081 wight=2;
         server localhost:8082 max_timeout=20s max_fails=5;
     }
     ```

   + 缺点：权值需要静态配置，无法自动调节！

   + 该策略 适用于 服务器性能差异较大的情况！

3. **ip_hash**

   + 指定 负载均衡器 按照 基于`客户端IP`分配请求！这个策略 确保了`相同的客户端请求一直发送到相同的服务器上`，使得每个访客都会 固定访问 某个后端服务器，避免了`跨服务器会产生 session 覆盖的问题`！

     会根据` Source IP、Destination IP、URL  或者 其他`，计算 `hash值 或者 md5 值`，再取模：**hash值 % N**

   + 通过*ip_hash* 配置

     ```nginx
     # 负载均衡如下三个应用服务器
     upstream dynamic_service {
         ip_hash;
         server localhost:8080;
         server lcoalhost:8081 wight=2;
         server localhost:8082 max_timeout=20s max_fails=5;
     }
     ```

   + 注意：

     1、nginx 1.3.1 之前，不能在 ip_hash 中使用 权重！

     2、ip_hash 不能与 backup 参数同时使用

     3、此策略适用于 有状态服务，例如：session

     4、当有服务器需要剔除时，需要手动 down 掉！

   + 缺点：

     *1、*若某个服务器结点 宕机，则需要剔除对应的结点，此时几乎所有的请求路由都发生了变化，导致命中率急剧下降！**hash值 % （N-1）**

     ​	`解决办法`：添加备份结点，不过`主备切换是需要时间`的，并且两者之间的`数据同步也是有延迟`的

     *2、*扩容时，也会遇到上述类似的问题！**hash值 % （N+1）**

     ​	`解决办法`：一般采用` 2 倍扩容`的方式，能保证一半以上的请求路由与原来保持一致！`不过也会带来服务器资源浪费的问题！`

     *3、*对于热点请求，`可能造成雪崩效应`，因为`无论是 基于 URL 还是 IP `其最终都是将请求转发到单机上，突然来的高峰请求（例如：秒杀、抢票）可能让单机引用无法承受，被打崩，最终各个结点，一个一个的宕机，造成雪崩效应！

4. **最少连接数 LC**（least_conn）

   + 把请求转发到 活动连接数 最少的 后端服务器上。轮询算法是把请求平均转发给各个后端，使他们的负载大致相同，但是有些请求占用的时间很长，会导致其所在的后端服务器负载较高，这种情况下，最少链接数策略就能达到更好的负载均衡效果！

   + 通过 *least_conn* 配置！

     ```nginx
     # 负载均衡如下三个应用服务器
     upstream dynamic_service {
         least_conn;
         server localhost:8080;
         server lcoalhost:8081 wight=2;
         server localhost:8082 max_timeout=20s max_fails=5;
     }
     ```

   + 适用于`请求处理时间长短不一`造成`服务器过载`的情况！

5. 第三方策略

   ​		  第三方的负载均衡策略的实现需要安装第三方插件！

   1. **响应时间**（fair）

      ​		按照服务端是响应时间来分配请求，响应时间短的优先分配！

      ```nginx
      # 通过 fair 参数配置
      # 负载均衡如下三个应用服务器
      upstream dynamic_service {
          fair;
          server localhost:8080;
          server lcoalhost:8081 wight=2;
          server localhost:8082 max_timeout=20s max_fails=5;
      }
      ```

   2. **依据 URL 分配**（url_hash）

      ​		按照访问的 url 结果进行分配，是`相同的 url 定向到同一个后端服务器，要配合缓存命中使用`！

      ​		同一个资源多次请求，可能会被转发到不同的服务器上，导致不必要的多次传输相同的数据，缓存命中率不高，还导致时间资源的浪费。而使用 url_hash，可以使得同一个 url 会被转发到同一台服务器上，一旦缓存住了资源，在收到此请求，就会从缓存中读取！（这不是和 ip_hash 差不多嘛？）

      ```nginx
      # 通过 hash $request_uri; 配置
      # 负载均衡如下三个应用服务器
      upstream dynamic_service {
          hash $request_uri;
          server localhost:8080;
          server lcoalhost:8081 wight=2;
          server localhost:8082 max_timeout=20s max_fails=5;
      }
      ```

##### 其他 负载均衡算法

1. **加权最小链接数（WLC）**

   加权最小链接数（Weighted Least Connection），在最小连接数的基础上，根据后端服务器的性能差异，优化 LC 的性能，高权值的服务器可以承受更多的连接负载！

2. **一致性 Hash**

   [详见：一致性hash算法详解](https://blog.csdn.net/qq_40551994/article/details/100991581)

##### Nginx 配置示例

```nginx
# user  nobody;

# 开启进程数 <=CPU数 
worker_processes  1;

# 错误日志保存位置
# error_log  logs/error.log;
# error_log  logs/error.log  notice;
# error_log  logs/error.log  info;

# 进程号保存文件
# pid        logs/nginx.pid;

# 每个进程最大连接数（最大连接=连接数x进程数）每个worker允许同时产生多少个链接，默认1024
events {
    worker_connections  1024;
}
http {

    # 【1】设置代理缓存的功能，配置缓存保存路径（相对路径、绝对路径）
    proxy_cache_path cache_catalogue levels=1:2 keys_zone=my_cache_name:10m;

    # 文件扩展名与文件类型映射表
    include       mime.types;
    # 默认文件类型
    default_type  application/octet-stream;

    # 日志文件输出格式 这个位置相于全局设置
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    # 请求日志保存位置
    # access_log  logs/access.log  main;
	
    # 打开发送文件
    sendfile        on;
    # tcp_nopush     on;

    # 连接超时时间
    keepalive_timeout  65;

    # 打开gzip压缩
    # gzip  on;
	
    # 设定请求缓冲
    # client_header_buffer_size 1k;
    # large_client_header_buffers 4 4k;
	
    # webapp
    # upstream myapp {   
    # server 192.168.1.171:8080 weight=1 max_fails=2 fail_timeout=30s;   
    # server 192.168.1.172:8080 weight=1 max_fails=2 fail_timeout=30s;   
    # } 

    # 指定在服务器中启动一个服务
    # 配置虚拟主机，基于域名、ip和端口
    server {
			# 监听端口
        listen       80;
			# 监听域名（在浏览器中访问的hostname）
        server_name  localhost;

        # charset koi8-r;
		
			# nginx访问日志放在logs/host.access.log下，并且使用main格式（还可以自定义格式）
        # access_log  logs/host.access.log  main;
		
			# 【2】 location
        location / {
            proxy_cache my_cache_name;
            proxy_set_header Host $host;
            root   html;
            index  index.html index.htm;
        }

			# 配置反向代理tomcat服务器：拦截.jsp结尾的请求转向到tomcat
        # location ~ \.jsp$ {
        #    proxy_pass http://192.168.1.171:8080;
        # }		
        
			# 错误页面及其返回地址
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }

}
```

###### Nginx 代理缓存功能

```nginx
# 【1】 代理缓存功能
proxy_cache_path cache_catalogue levels=1:2 keys_zone=my_cache_name:10m;
# 如下解析：
cache_catalogue		# 设置相对路径目录
levels				# 是否设置二级文件夹
<!--
	由于cache_catalogue 可以用在很多不同的代理服务器中，如果没有声明则默认将所有代理放在一个文件夹一个目录中，没有分子目录。这样随着缓存量得增多，查找的时间会越来越长，所以分一个二级文件夹，效率会高一点。
-->
keys_zone=my_cache_name:10m
	y_cache_name 	# 缓存文件：每个 server 中都可以使用缓存，可通过缓存名称决定使用哪个缓存
	10m				# 缓存文件大小（兆）

<!--
 · 由于代理缓存是在代理层设置，所以每一个新的请求都会经过代理，如果请求已经缓存过一次了，则后面相同的请求就会直接使用代理的缓存。它更高效，对于通用的内容能够更好的提供缓存的功能。

 · Cache-Control HTTP 首部部字段，http1.1 中的重要规则，用于控制页面缓存。有以下属性：
	1：public	：指定使用 public 时，明确表明，其他用户也可以使用缓存
	2：private	：只允许客户端缓存，代理服务器不可缓存！
	3：no-cache	：不缓存过期的资源，缓存会向源服务器进行有效性验证！
	4：no-store	：任意结点都不允许缓存！
	5、max-age = n	：表示缓存在 n 秒后失效
	6、s-maxage = n	：作于与 max-age 一致，s-maxage 只对代理服务器有效，在代理服务器中，s-maxage 将会覆盖 max-age 与 Expires 头部！

	示例：
	Cache-Control : private,max-age=60,no-cache

 · Vary HTTP 首部字段
	当代理服务器接收到 带有 Vary 首部字段指定获取资源的请求后时，则表明，只有头部字段中 带有 Vary字段，并且 Vary 字段值相等时，才能使用缓存，这意味着，不仅要 url 相同，还需要指定头部信息相同才行！

	示例：
	Vary : Accept-Language	：国际化时可根据语言的不同，获取到对应的缓存内容！
	Vary : User-agent		：多端访问时，手机端 和 PC端 各自使用各自的缓存！
-->
```

###### Nginx location配置

```nginx
# 【2】 location
# 斜杠/：表示该域下的所有http请求都代理到proxy_pass设置的url上
location / {
    # 解析：设置缓存到声明的缓存名指定的位置
    proxy_cache my_cache_name;

    # 负载均衡反向代理
    proxy_pass http://myapp;
    # 【设置 客户端 host】
    proxy_set_header Host $host;
    # 设置客户端真实ip地址
    proxy_set_header X-real-ip $remote_addr;
    
    # 返回根路径地址（相对路径:相对于/usr/local/nginx/）
    root   html;
    # 默认访问文件
    index  index.html index.htm;
}

# 【设置 客户端 host】
proxy_set_header Host $host;
<!--
 · Host $host：修改头部host为$host
		$host 表示在 nginx 中 原http请求 携带的 host 变量
 · 原因：
	nginx 是代理服务器，会将请求转发到服务端，而此请求是 Nginx 发出的，所以 请求中的 host 为代理服务器的 host，并不是 客户端的！所以如果想要保持客户端的host，需要在这里设置 proxy_set_header
	任何中间代理服务器都可以修改 http 协议的请求，因为它是明文传输，但是由于https协议整个过程是加密的，中间代理没有办法解析，所以不能修改。
-->
```

