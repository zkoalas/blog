##  Gateway网关

### 1.简介

SpringCloud Gateway是Spring官方基于Spring5.0,SpringBoot2.0和Project Reactor等技术开发的网关,旨在为微服务框架提供一种简单而有效的统一的API路由管理方式,统一访问接口.SpringCloud Gateway作为SpringCloud生态体系中的网关,目标是替代NetFlixZull,其不仅提供统一的路由方式,并且基于Filter链的方式提供了网关基本的功能.比如:安全、监控/埋点、和限流等.它是基于Nttey的响应式开发模式.

核心概念

* **路由(route)** 路由是网关最基础的部分,路由信息由一个ID、一个目的URL、一组断言工厂和一组Filter组成.如果断言为真,则说明请求URL和配置的路由匹配.
* **断言(predicates)** java8中的断言函数,SpringCloud Gateway中的断言函数输入类型是Spring5.0框架中的ServerWebExchange. SpringCloud Gateway中的断言函数允许开发者去定义匹配来自Http Request中的任何信息,比如请求头和参数等.
* **过滤器(filter)** 一个标准的Spring webFilter, Spring Cloud Gateway中的Filter分为两种类型,分别是Gateway Filter和Global Filter.过滤器Filter可以对请求和响应进行处理.

### 2.路由配置

* 添加坐标

  ```java
  <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-gateway</artifactId>
  </dependency>
  ```

* 添加配置文件

  ```yaml
  server:
    port: 8080 #端口
  spring:
    application:
      name: api-gateway-server #服务名称
  #配置SpringCloudGateway的路由
    cloud:
      gateway:
        routes:
          #配置路由:路由id，路由到微服务的uri，断言(判断条件)
          - id: product-service #保持唯一
            uri: http://127.0.0.1:9001 #目标微服务请求地址
            predicates:
              - Path=/product/** #路由条件 path: 路由匹配条件
  ```

* 路由规则

  RoutePredicateFactory

  > **datetime**(请求时间检验条件谓语)
  >
  > > AfterRoutePredicateFactory:请求时间满足在配置时间之后
  > >
  > > BeforeRoutePredicateFactory:请求时间满足在配置时间之前
  > >
  > > BetweenRoutePredicateFactory:请求时间满足在配置时间之间
  >
  > **Cookie**(请求cookie校验条件谓语)
  >
  > > CookieRoutePredicateFactory:请求指定Cookie正则匹配指定值
  >
  > **Header**(请求header检验条件谓语)
  >
  > > HeaderRoutePredicateFactory:请求header正则匹配指定值
  > >
  > > CloudFoundryRouteServiceRoutePredicateFactory:请求headers是否包含指定的名称
  >
  > **Host**(请求host检验条件谓语)
  >
  > > HostRoutePredicateFactory:请求host匹配指定值
  >
  > **Method**(请求method检验条件谓语)
  >
  > > MethodRoutePredicateFactory:请求Method匹配配置的method
  >
  > **Path**(请求path检验条件谓语)
  >
  > > PathRoutePredicateFactory:请求路径正则匹配指定值
  >
  > **Queryparam**(请求查询参数检验条件谓语)
  >
  > > QueryRoutePredicateFactory:请求查询参数正则匹配指定值
  >
  > **RemoteAddr**(请求远程地址检验条件谓语)
  >
  > > RemoteAddrRoutePredicateFactory:请求远程地址匹配配置指定值

* 动态路由(面向服务的路由)

  1.添加坐标

  ```pro
  <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
  </dependency>
  ```

  2.配置注册中心

  ```properties
  #Eureka注册中心
  eureka:
    client:
      service-url:
        defaultZone: http://localhost:9000/eureka/
    instance:
      prefer-ip-address: true #使用IP地址注册
  ```

  3.配置gateway网关lb://

  ```properties
  #配置SpringCloudGateway的路由
    cloud:
      gateway:
        routes:
          #配置路由:路由id，路由到微服务的uri，断言(判断条件)
          - id: product-service #保持唯一
            uri: lb://service-product #lb:// 根据微服务名称从注册中心中拉取服务请求路径
            predicates:
              - Path=/product/** #路由条件 path: 路由匹配条件
  ```

  4.网关路由加前缀**/product-service**

  ```properties
  server:
    port: 8080 #端口
  spring:
    application:
      name: api-gateway-server #服务名称
  #配置SpringCloudGateway的路由
    cloud:
      gateway:
        routes:
          #配置路由:路由id，路由到微服务的uri，断言(判断条件)
          - id: product-service #保持唯一
  #          uri: http://127.0.0.1:9001 #目标微服务请求地址
            uri: lb://service-product #lb:// 根据微服务名称从注册中心中拉取服务请求路径
            predicates:
  #            - Path=/product/** #路由条件 path: 路由匹配条件
              - Path=/product-service/**  #将当前请求转发到 http:127.0.0.1:9001/product/1
            filters: #配置路由过滤器
              - RewritePath=/product-service/(?<segment>.*),/$\{segment} #路径重写的过滤器,在yml中$写为$\
  #Eureka注册中心
  eureka:
    client:
      service-url:
        defaultZone: http://localhost:9000/eureka/
    instance:
      prefer-ip-address: true #使用IP地址注册
  
  ```

  http://localhost:8080/product-service/product/1

  5.配置自动的根据微服务名称进行路由转发

  ```properties
  spring:
    application:
      name: api-gateway-server #服务名称
  #配置SpringCloudGateway的路由
    cloud:
      gateway:
        routes:
          #配置路由:路由id，路由到微服务的uri，断言(判断条件)
          - id: product-service #保持唯一
  #          uri: http://127.0.0.1:9001 #目标微服务请求地址
            uri: lb://service-product #lb:// 根据微服务名称从注册中心中拉取服务请求路径
            predicates:
  #            - Path=/product/** #路由条件 path: 路由匹配条件
              - Path=/product-service/**   #将当前请求转发到 http:127.0.0.1:9001/product/1
            filters: #配置路由过滤器
              - RewritePath=/product-service/(?<segment>.*),/$\{segment} #路径重写的过滤器,在yml中$写为$\
        #配置自动的根据微服务名称进行路由转发  http://localhost:8080/service-product/product/1
        discovery:
          locator:
            enabled: true #开启根据微服务名称自动转发
            lower-case-service-id: true #微服务名称以小写形式呈现
  #Eureka注册中心
  eureka:
    client:
      service-url:
        defaultZone: http://localhost:9000/eureka/
    instance:
      prefer-ip-address: true #使用IP地址注册
  
  ```

  开启微服务名称转发

  ```properties
  spring:
    cloud:
      gateway:
        #配置自动的根据微服务名称进行路由转发  http://localhost:8080/service-product/product/1
        discovery:
          locator:
            enabled: true #开启根据微服务名称自动转发
            lower-case-service-id: true #微服务名称以小写形式呈现
  ```

  

### 3.过滤器

* 过滤器的生命周期

  SpringCloudGateway的Filter的生命周期不像zuul的那么丰富,她只有两个:"pre"和"post".

  1.PRE: 这种过滤器在请求被路由之前调用.我们可以利用这种过滤器实现身份认证、在集群中选择请求的微服务、记录调试信息等.

  2.POST: 这种过滤器在路由到微服务以后执行.这种过滤器可用来为响应添加标准的HTTP header、收集统计信息和指标、将响应从微服务发送到客户端等.

* 过滤器类型

  SpringCloudGateway的Filter从作用范围可分为另外两种GatewayFilter和GlobalFilter.

  **GatewayFilter**: 用在单个路由或者一个组的路由上.

  **GlobalFilter**:应用到所有的路由上.

* 局部过滤器

  局部过滤器(GatewayFilter),是针对单个路由的过滤器.可以对访问的URL过滤,进行切面处理.在SpringCloudGateway中通过GatewayFilter的形式内置了很多不同类型的局部过滤器.

  | 过滤器工厂                  | 作用                                                         | 参数                                                         |
  | --------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
  | AddRequestHeader            | 为原始请求添加Header                                         | Header的名称及值                                             |
  | AddRequestParameter         | 为原始请求添加请求参数                                       | 参数名称及值                                                 |
  | AddResponseHeader           | 为原始响应添加Header                                         | Header的名称及值                                             |
  | DedupeResponseHeader        | 剔除响应头中重复的值                                         | 需要去重的Header名称及去重策略                               |
  | Hystrix                     | 为路由器引入Hystrix的断路器保护                              | HystrixCommand的名称                                         |
  | FallbackHeaders             | 为fallbackUri的请求头中添加具体的异常信息                    | Header的名称                                                 |
  | PrefixPath                  | 为原始请求路径添加前缀                                       | 前缀路径                                                     |
  | PreserveHostHeader          | 为请求添加一个preserveHostHeader=true的属性,路由过滤器会检查改属性以决定是否需要发送原始的Host | 无                                                           |
  | RequestRateLimiter          | 用于对请求限流,限流算法为令牌桶                              | keyResolver、rateLimiter、statusCode、denyEmptyKey、emptyKeyStatus |
  | RedirectTo                  | 将原始请求重定向到指定的URL                                  | http状态码及重定向的url                                      |
  | RemoveHopByHopHeadersFilter | 为原始请求删除IETF组织规定的一系列Header                     | 默认就会启用,可以通过配置指定仅删除那些Header                |
  | RemoveRequestHeader         | 为原始请求删除某个Header                                     | Header名称                                                   |
  | RemoveResponseHeader        | 为原始响应删除某个Header                                     | Header名称                                                   |
  | RewritePath                 | 重写原始的请求路径                                           | 原始路径正则表达式以及重写后路径的正则表达式                 |
  | RewriteResponseHeader       | 重写原始响应中的某个Header                                   | Header名称,值的正则表达式,重写后的值                         |
  | SaveSession                 | 在转发请求之前,强制执行websession::save操作                  | 无                                                           |
  | SecureHeaders               | 为原始响应添加一系列起安全作用的响应头                       | 无,支持修改这些安全响应头的值                                |
  | SetPath                     | 修改原始的请求路径                                           | 修改后的路径                                                 |
  | SetResponseHeader           | 修改原始响应中某个Header的值                                 | Header名称,修改后的值                                        |
  | SetStatus                   | 修改原始响应的状态码                                         | HTTP状态码,可以是数字,也可以是字符串                         |
  | StripPrefix                 | 用于截断原始请求的路径                                       | 使用数字表示要截断的路径的数量                               |
  | Retry                       | 针对不同的响应进行重试                                       | retries、statuses、methods、series                           |
  | RequestSize                 | 这是允许接口最大请求包的大小.如果请求包大小超过设置的值,则返回413 Payload Too Large | 请求包大小,单位为字节,默认值为5M                             |
  | ModifyRequestBody           | 在转发请求之前修改原始请求体内容                             | 修改后的请求体内容                                           |
  | ModifyResponseBody          | 修改原始响应体的内容                                         | 修改后的响应体内容                                           |

  每个过滤器工厂都对应一个实现类,并且这些类的名称必须以GatewayFilterFactory结尾,这是SpringCloudGateway的一个约定,例如AddRequestHeader对应的实现类为AddRequestHeaderGatewayFilterFactory.对于这些过滤器的使用方式可以参考官方文档.

* 全局过滤器

  全局过滤器(GlobalFilter)作用于所有路由,SpringCloudGateway定义了GlobalFilter接口,用户可以自定义实现自己的GlobalFilter.通过全局过滤器可以实现对权限的统一校验,安全性验证等功能,并且全局过滤器也是程序员使用比较多的过滤器.

  SpringCloudGateway内部也是通过一系列的内置全局过滤器对整个路由转发进行处理如下:

  > GloubalFilter
  >
  > > LoadBalancer: 负载均衡客户端相关过滤器
  > >
  > > > LoadBalancerClientFilter:通过负载均衡客户端根据路径的URL解析转换成真实的请求URL
  > >
  > > HttpClient:http客户端相关过滤器
  > >
  > > > NettyRoutingFilter
  > > >
  > > > NettyWriteResponseFilter
  > > >
  > > > 通过HttpClient客户端转发请求真实的URL并将响应写入到当前的请求响应中
  > >
  > > Websocket:Websocket相关过滤器
  > >
  > > > WebsocketRoutingFilter: 负责处理Websocket类型的请求响应信息
  > >
  > > ForwardPath:路径转发相关过滤器
  > >
  > > > ForwardPathFilter: 解析路径并将路径转发
  > >
  > > RouteToRequestUrl:路由url相关过滤器
  > >
  > > > RouteToRequestUrlFilter:转换 路由中 的URI
  > >
  > > WebClient:WebClient相关过滤器
  > >
  > > > WebClientHttpRoutingFilter
  > > >
  > > > WebClientWriteResponseFilter
  > > >
  > > > 通过WebClient客户端转发请求真实的URL并将响应写入到当前的请求响应中

* 自定义全局过滤器

  ```java
  /**
   * 自定义一个全局过滤器
   *      实现globalFilter，ordered接口
   */
  @Component
  public class LoginFilter implements GlobalFilter, Ordered {
  
      //执行过滤器中的业务逻辑
      @Override
      public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
          System.out.println("执行了自定义的全局过滤器");
          return chain.filter(exchange);//继续向下执行
      }
  
      //指定过滤器的执行顺序，返回值越小，执行优先级越高
      @Override
      public int getOrder() {
          return 0;
      }
  }
  ```

  

### 4.统一鉴权

内置的过滤器已经可以完成大部分的功能,但是对于企业开发的一些业务功能处理,还是需要自己编写过滤器来实现的,那么我们一起通过代码的形式自定义一个过滤器,去完成统一的鉴权校验.

* 鉴权逻辑

  开发中的鉴权逻辑:

  + 当客户端第一次请求服务时,服务端对用户进行信息认证(登陆)
  + 认证通过,将用户信息进行加密形成token,返回给客户端,作为登陆凭证
  + 服务端对token进行解密,判断是否有效.

  ![image-20200921092441463](/Users/andy/Library/Application Support/typora-user-images/image-20200921092441463.png)

* 代码实现

  下面的我们自定义一个GlobalFilter,去校验所有请求参数中是否包含"token",如果不包含请求参数"token"则不转发路由,否则执行正常的逻辑.

  

### 5.网关限流



### 6.网关的高可用

