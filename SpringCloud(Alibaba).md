# SpringCloud组件设计原理及实战学习笔记
### 代码及文档地址 
代码地址：

1. sleuth+zipkin+oauth+jwt:[https://github.com/dongxu-lee/sleuth_zipkin](https://github.com/dongxu-lee/sleuth_zipkin)

2. nacos+sentinel:[https://github.com/dongxu-lee/cloud_alibaba](https://github.com/dongxu-lee/cloud_alibaba)

文档地址：

3. github文档地址：[https://github.com/dongxu-lee/myLearningPDF/blob/main/SpringCloud(Alibaba).md](https://github.com/dongxu-lee/myLearningPDF/blob/main/SpringCloud(Alibaba).md)

4. 语雀文档地址：[https://www.yuque.com/bailihang-3fszp/bkgbrq/knaua8](https://www.yuque.com/bailihang-3fszp/bkgbrq/knaua8)

## 第一部分：SpringCloud核心组件原理及实战
### 1.1 Eureka服务注册与发现
#### Eureka客户端
##### 服务注册（服务提供者） 
服务启动时向注册中心发起注册请求，携带服务元数据信息；eureka注册中心吧服务信息保存在Map中
##### 服务续约（服务提供者）
服务每隔30s向注册中心心跳一次，如果没有心跳，90s后注册中心认为服务失效
##### 获取服务列表（服务消费者）
每隔30s从注册中心拉取一份服务列表
#### Eureka服务端
##### 服务下线
服务正常关闭时会发送服务下线的REST给注册中心，注册中心标记服务为下线。
##### 失效剔除
Eureka Server90s没有收到心跳续约，注销服务。
##### 自我保护
15分钟内超过85%的客户端节点都没有正常的心跳，那么eureka就认为客户端和注册中心之间网络故障，Server自动进入自我保护机制。

当处于自我保护机制时：

1. 不会剔除任何服务；
2. Server仍能接收新的服务的注册和查询请求，但不会同步给其他节点，保证当前节点可用，网络稳定时，同步给其他节点；
3. 通过eureka.server.enable-self-preservation配置，默认打开

#### Eureka源码分析
##### Eureka启动过程分析
1. 入口：充分利用SpringBoot自动装配的特性，在eureka Server依赖包里的spring.factories里配置springBoot自动装配eureka的EurekaServerAutoConfiguration类；
2. 分析EurekaServerAutoConfiguration类，有三点：
> 1.  要装配该类需要先装配maker类，这个类从哪儿装配？
> 答：在启动类上要加注解@EnableEurekaServer，这个注解其中一个功能就是装配maker类。
> 2.  对该类解析得到什么？
> 答：该类注入了以下bean：eureka仪表盘页面类、对等节点感知注册器、辅助保存节点信息、eureka上下文；通过线程池更新服务列表。
> 3.  该类注释里导入了initial这样一个配置类，这个类的作用是？
> 答：关注eureka启动后的一些细节

##### Eureka客户端启动过程
1. 入口：充分利用SpringBoot自动装配的特性，在eureka Server依赖包里的spring.factories里配置springBoot自动装配eureka的EurekaClientAutoConfiguration类；
2. 分析上面这个类：
> 1. 读取配置文件
> 2. 注册自己到EurekaServer
> 3. 开启一些定时任务（心跳续约，刷新本地服务缓存列表）

### 1.2 Ribbon负载均衡
#### 负载均衡策略
1. 轮询
2. 随机
3. 重试
4. 最小连接数：选择当前连接数最小的server去访问
5. 可用过滤：先轮询，把选定的server进行各种可用性测试，确定可用再访问
6. 区域权衡：先轮询，过滤掉超时和连接数过多的server，以及不符合要求的zone里的所有节点

#### Ribbon工作原理
Ribbon给restTemplate加了一个拦截器

- IRule：选择实例的时候的负载均衡策略对象
- IPing：向服务发起心跳检测的

#### Ribbon源码解析之RestTemplate对象绑定
给RestTemplate添加@LoadBalanced注解，该注解会给RestTemplate对象添加一个拦截器，该拦截器就是后续拦截请求进行负载处理的。

### 1.3 Hystrix服务熔断与降级
#### Hystrix舱壁模式（线程池隔离策略）
所有加了@HystrixCommand注解的方法都默认共用一个线程池（默认10个线程），如果线程池中的线程当前被全部占用，那么后续的请求只能等待或被拒绝。

解决方法：给每个方法都建一个线程池，实现方法：在@HystrixCommand注解里配置threadPoolKey属性值时，给每一个方法都配置一个唯一的标识，保证每个方法只使用唯一的线程池

#### Hystrix流程定制
~~~java
@HystrixCommand(
	commandProperties = {
		// 1.当调用出现问题时，开启一个时间窗（8s）
		@HystrixProperty(name = "metrics.rollingStats.timeInMilliseconds", value = "8000"),
		// 2.在时间窗内，调用次数是否达到最小请求数（2次）？
		// 如果没有达到，回到第一步
		// 如果达到了，统计失败的请求数占所有请求数的比例，是否达到阈值？
		@HystrixProperty(name = "circuitBreaker.requestVolumeThreshold", value = "2"),
		// 3.阈值设置为50%
		// 达到阈值，跳闸
		// 达不到阈值，回到第一步
		@HystrixProperty(name = "circuitBreaker.errorThresholdPercentage", value = "50"),
		// 4.如果跳闸，开启一个活动窗口（默认5s，这里设置为3s），每隔3s，Hystrix会让一个请求来访问方法
		// 如果成功，重置断路器，回到第一步
		// 如果失败，重复第四步
		@HystrixProperty(name = "circuitBreaker.sleepWindowInMilliseconds", value = "3000")
	}
)
~~~
可以在yml中配置，以实现全局生效。

#### 共用降级方法
在类上使用@DefaultProperties注解统一指定整个类中共用的降级方法

#### Hystrix Dashboard

### 1.4 Feign
#### Feign对日志的支持
1.开启日志配置
~~~java
// NONE:默认的，不打印日志
// BASIC:只打印get/post，url，code，time等基础信息
// HEADERS:在basic基础上多了header信息
// FULL:请求和响应的详细信息
@Configuration
public class FeignConfig() {
	@Bean
	Logger.level feignLevel() {
		return Logger.Level.FULL;
	}
}
~~~
2.配置日志级别为debug
~~~yml
logging:
  level:
    # feign接口
    com.ldx.service.FeignClient: debug
~~~
#### Feign对Hystrix的支持
实质：实现消费者中的Feign接口，当Feign注解接口远程调用方法失败时，改为执行本地的接口实现类，进而实现服务熔断和降级。

注意超时有两个设置，一个是Feign的超时，一个是Hystrix的超时，真正执行的超时时间以这两个设置中的最小值来执行。

#### Feign对请求压缩和响应压缩的支持
#### Feign源码解析
实质：使用了JDK动态代理，在代理方法中，把请求信息封装到RequestTemplate里，使用客户端client执行excute方法（底层是HttpConnection发送请求），执行请求并接受响应。

### 1.5 Gateway网关
#### 基本配置
~~~yml
spring:
  cloud:
    gateway:
      routes: # 路由可以有多个
        - id: service1 #自定义的路由ID，保持唯一
          uri: http://127.0.0.1:8090
          predicates: # 断言
            - Path=/ser1/**
        - id: service2 #自定义的路由ID，保持唯一
          uri: http://127.0.0.1:8080
          predicates: # 断言
            - Path=/ser2/**
~~~
#### 断言工厂

 - DateTime实践类断言：根据请求时间在配置时间之前/之后/之间
 - Cookie类断言：指定Cookie正则匹配指定值
 - Header请求头类断言：指定Header正则匹配指定值/请求头是否包含某个字段
 - Host请求主机类断言：请求Host匹配指定值
 - Method请求方式类断言：指定Method匹配指定请求方式
 - Path请求路径类断言：请求路径正则匹配
 - QueryParam请求参数类断言：查询参数正则匹配
 - RemoteAddr远程地址类断言：请求远程地址匹配指定值

#### 动态路由
在基本配置里，把uri配置为服务名称，从注册中心获取服务，进而实现动态配置服务。
~~~yml
# lb表示从注册中心获取服务
uri: lb://myservice
~~~
#### 过滤器
单个路由过滤：
~~~yml
- id: service1
  ...
  filters:
    # 原路径：http://localhost:8080/resume/time/id
    # 执行过滤器后，实际请求路径： http://localhost:8080/time/id
    # 路径里的第一部分/resume被去掉了
    - stripfix=1 # 请求路径里去掉第一部分
~~~
全局路由过滤：
GlobalFilter

### 1.6 Config配置中心
分为server和client，server配置远程配置文件的地址，client访问server从而获取地址
#### 服务器端基本配置
~~~yml
spring:
  cloud:
    config:
      server:
        git:
          uri: https://github.com/123/ldx-config-repo.git
          username: ldx
          password: 123456
          search-paths:
            - ldx-config-repo
      label: master #分支
~~~
测试访问：http://localhost:9003/master/123.456-dev.yml，查看配置文件内容

#### 客户端基本配置
创建bootstrap.yml，这个文件是系统级别的，优先级比application.yml级别要高
~~~yml
cloud:
  config:
    name: 123.456 #配置文件名称
    profile: dev #后缀名称
    label: master #分支
    uri: http://localhost:9003 #config的server端地址
~~~
使用：和取本地信息一样，直接@Value("mysql.url")就可以获取远程配置文件里的内容。

#### 客户端手动刷新

1. 客户端添加actuator依赖
2. 客户端bootstrap.yml中添加配置（暴露通信断点）
~~~yml
management:
  endpoints:
    web:
      exposure:
        include: /refresh #也可以暴露所有接口 include: "*"
~~~
3. 客户端使用配置信息的类上加@RefreshScope注解
4. 手动向客户端发起POST请求，http://localhost:8080/actuator/refresh，刷新配置信息

<B>注意：手动刷新避免了服务重启</B>

#### 客户端自动刷新
Spring Cloud Bus消息总线：通过一个主题topic连接各个微服务，实现消息的监听和消费

### 1.7 Stream消息驱动
通过上层抽象，把应用程序与具体的消息组件解耦合，屏蔽底层具体MQ消息中间件的差异，就像Hibernate屏蔽具体数据库（mysql或oracle）。

Stream提供了不同的binder，当我们需要切换MQ产品的时候，只需要切换Binder即可。

### 注：SpringCloud两个常见问题及解决方案
#### 1.Eureka服务发现慢的原因
1. Eureka客户端每隔30s向服务端请求一次服务列表
2. Ribbon每隔30s向客户端请求一次服务列表
3. Eureka服务端包括一级缓存（readOnly cacheMap）、二级缓存（readWrite cacheMap）、数据存储层（registry）。客户端先请求一级缓存，没有则请求二级缓存，也没有则请求数据存储层，一级缓存会定期从二级缓存更新服务列表，所以客户端从服务端获取的服务列表也可能不是最新的

解决方案：

1. 服务端：
> #服务发现时间间隔
> eureka.server.response-cache-update-interval-ms=123
> #只读缓存（一级缓存）关闭
> eureka.server.use-read-only-response-cache=false

2. 客户端：
> #定时拉取服务信息的时间间隔
> eureka.client.registryFetchIntervalSeconds
> #Ribbon刷新间隔
> getRefreshIntervalMs

#### 2.SpringCloud超时设置问题
重点关注Ribbon和Hystrix即可


## 第二部分：SpringCloud高级实战
### 2.1 分布式链路追踪Sleuth+Zipkin
#### 基础理论
本质：日志记录

Trace：服务追踪的追踪单元是从客户端发起请求抵达被追踪系统的边界开始，到被追踪的系统响应客户为止的过程

Trace ID：当请求到达分布式系统的入口端点时，只需要服务跟踪框架为该请求创建一个唯一的跟踪标识Trace ID，同时在分布式系统内部流转的时候，框架一直保持这个ID，直到追踪结束

Span：一个Trace由多个Span组成

Span ID：为了统计各处理单元的时间延迟，党请求到达各个服务组件时，由Span ID标记它的开始到结束，从而计算出时间间隔

Span中抽象出了另一个概念：事件。核心事件如下：

- CS：client send/start 客户端/消费者发出一个请求，描述一个span的开始
- SR：server received 服务端/生产者接收请求
- SR-CS 属于请求发送的网络延迟
- SS：server send 服务端/生产者发送应答
- SS-SR 属于服务器消耗时间
- CR：client received 客户端/消费者接收内容
- CR-SS 表示回复需要的时间

## 第三部分：Spring Cloud Alibaba高级实战
### 3.1 Nacos服务注册中心
#### 单例部署server端
在github找到nacos，下载当前稳定版本（1.4.0）

1. windows环境下，下载zip，解压后，编辑bin目录下的startup.cmd，把MODE由cluster（集群模式）改为standalone（单例模式），双击startup.cmd启动；
2. linux环境下，下载tar.gz，解压后执行sh startup.sh -m standalone启动；
3. 访问ui界面：http://127.0.0.1:8848/nacos/index.html，用户名和密码都是nacos
#### 服务提供者
父类配置版本依赖
~~~xml
<!-- 要特别注意spring boot和spring cloud alibaba之间的版本关系 -->
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>com.alibaba.cloud</groupId>
                <artifactId>spring-cloud-alibaba-dependencies</artifactId>
                <version>2.1.2.RELEASE</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
~~~
服务提供者配置发现类
~~~xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>
~~~
启动类加注解
~~~java
@EnableDiscoveryClient
~~~
配置文件配置nacos
~~~yml
spring:
  application:
    name: provider
  cloud:
    nacos:
      discovery:
        server-addr: 127.0.0.1:8848
~~~
服务启动后，在ui页面的服务管理--服务列表里可以看到实例

##### 保护阈值
保护阈值可设置为0-1，当前服务健康实例数/当前服务总实例数<保护阈值，说明当前健康实例比较少，当请求到来时，nacos会把健康和不健康的实例都提供给消费者，所以消费者有可能请求失败，但这样可以避免雪崩（所有请求都访问健康实例）。

#### 服务消费者
服务消费者配置发现类
~~~xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>
~~~
启动类加注解
~~~java
@EnableDiscoveryClient
~~~
注意在注入RestTemplate时一定要加@LoadBalanced注解
~~~java
/**
 * 这里的@LoadBalanced必不可少，因为需要它来根据服务名选择某台服务器，并获取uri
 * @return
*/
@LoadBalanced
@Bean
public RestTemplate restTemplate() {
    return new RestTemplate();
}
~~~
配置文件配置nacos
~~~yml
spring:
  application:
    name: provider
  cloud:
    nacos:
      discovery:
        server-addr: 127.0.0.1:8848
~~~
服务启动后，在ui页面的服务管理--服务列表里可以看到实例

##### 订阅者列表
服务消费者可以直接使用restTemplate通过服务名访问服务，访问之后，在订阅者列表会出现相关信息

#### 领域数据模型
概念 | 描述
--- | ---
Namespace | 命名空间，可以用来隔离生产环境、测试环境等
Group        |   分组，可以用来隔离项目，一般一个项目在一个组里
Service    |     服务，具体指项目中的某个服务
DataId    |      配置文件，具体指某个项目中的某个配置文件

常用方式:

1. Namespace+Group+Service 锁定某个服务
2. Namespace+Group+DataId 锁定某个配置文件

还有一个cluster（集群），也可以实现隔离

#### 数据持久化
application.properties里解除数据库相关注释，并创建相关数据库，执行nacos-mysql.sql

#### Nacos集群
- 复制解压后的文件夹
- 三个application.properties修改server-port
- 三个application.propertie绑定ip：nacos.inetutils.ip-address=127.0.0.1
- 三个nacos复制conf/cluster.conf.example文件，命名为cluster.conf，在配置文件中配置三个集群节点
> 127.0.0.1:8848
> 127.0.0.1:8849
> 127.0.0.1:8850

- 启动三个实例 sh startup.sh -m cluster或者startup.cmd把MODE改为cluster再双击
- 节点分LEADER和FOLLOWER

#### 微服务注册到Nacos集群
和eureka一样，逗号隔开

### 3.2 Nacos服务配置中心
1. 打开配置管理-配置中心，新增一个配置文件，Data ID的格式为
~~~
${spring.application.name}-${spring.profile.active}.${file-extension}
~~~
如果没有spring.profile.active，可以不配置，但要把横线也去掉，比如：provider.yaml

2. Group可以默认DEFAULT_GROUP
3. 填写配置内容，要注意根据选择的配置格式来书写，如：
~~~yml
name: xiaoming
~~~
4. 新增依赖
~~~xml
<dependency>
  <groupId>com.alibaba.cloud</groupId>
  <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
</dependency>
~~~
5. 服务的bootstrap.yml中配置，必须是bootstrap.yml
~~~yml
spring:
  application:
    name: provider
  cloud:
    nacos:
      discovery:
        server-addr: 127.0.0.1:8848
      # 锁定server端的配置文件（读取它的配置项）
      config:
        server-addr: 127.0.0.1:8848
        namespace: 9ae00c78-3a0a-43a1-bd1a-7e5684677227
        group: DEFAULT_GROUP #默认的可以不配置
        file-extension: yaml
~~~
namespace如果是public可以不配置，本地为了测试，新建了一个名为dev的命名空间，并在dev里新建的配置列表中的配置文件；

group如果是默认的DEFAULT_GROUP，可以不配置

6. 直接使用配置
~~~java
    @Value("${name}")
    private String name;
~~~
#### 一个项目可能希望读取多个配置文件
~~~yml
spring:
  application:
    name: provider
  cloud:
    nacos:
      discovery:
        server-addr: 127.0.0.1:8848
      # 锁定server端的配置文件（读取它的配置项）
      config:
        server-addr: 127.0.0.1:8848
        namespace: 9ae00c78-3a0a-43a1-bd1a-7e5684677227
        group: DEFAULT_GROUP #默认的可以不配置
        file-extension: yaml
        # 除了provider.yaml之外，还能读取下面两个配置文件
        extension-configs:
          - dataId: abc.yaml
          - dataId: 123.yaml
~~~
##### 上述情况下，多个配置文件里有相同属性，优先级怎么判断？
首先，provider.yaml优先级最高，因为它是主要配置文件；如果扩展的配置文件里有重复属性，那么按照配置顺序，后面的配置文件会覆盖前面的配置文件的配置，如上，123.yaml里的配置会覆盖abc.yaml里的重复配置。
### 3.3 sentinel
1. github上下载server并启动，访问8080
2. 项目引入依赖
~~~java
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
</dependency>
~~~
3. applicatio.yml配置
~~~yml
spring:
  application:
    name: consumer
  cloud:
    nacos: #nacos配置，如果没有使用nacos，这里就不用配置
      discovery:
        server-addr: 127.0.0.1:8848
    sentinel:
      transport:
        dashboard: 127.0.0.1:8080
        port: 8719
~~~
4. 因为是懒加载，需要执行一次请求后，8080页面才会看到服务
#### 流量控制
##### 流控模式：
- 直接
- 关联：关联资源里的请求超出阈值，当前请求也会被相同的规则流量控制。比如在/regis请求里配置流控，关联资源配置/valid，那么当/valid被流控的时候，/regis也会被流控
- 链路：链路指的是请求链路。比如多个链路的请求都会到当前方法，在链路资源里配置的入口方法一直到当前方法这一条链路会被流控，其他的链路不受影响
##### 流控效果：
- 快速失败
- warm up：预热，比如设置QPS单机阈值为20，预热时长为5s，那么在请求来临时，前5s的QPS会被限制为单机阈值的三分之一（20/3），5s之后，QPS限制恢复为20
- 排队等待：比如QPS设置为5，超时时间设置为3s，这表示每200ms处理一个请求，多出的请求要等待，但这些等待的请求里，如果它的等待时间要超过3s，那么请求会被直接拒绝
#### 服务降级（相当于Hystrix的熔断）
- RT：平均响应时间。当1s内请求数>=5个，平均响应时间超过阈值（页面设置RT），那么在接下来的时间窗口（页面设置时间窗口）内，对该方法的调用都会自动熔断。时间窗口结束后，结束降级。注意sentinel的RT设置上限是4900ms。可以通过配置修改。
- 异常比例：当1s请求>=5个，并且 异常请求个数 / 总请求个数 > 阈值 时，在接下来的时间窗口内，资源降级，对这个方法的调用都会自动返回。
- 异常数：当资源近1分钟的异常数目超过阈值，就会熔断。注意由于统计时间是分钟级别的，如果时间窗口设置的小于60s，则熔断结束后仍可能再次进入熔断状态

#### 兜底方法（相当于Hystrix的降级）
1.普通操作
~~~java
    /** 服务降级 **/
    @RequestMapping("/user")
    @SentinelResource(value = "user", blockHandler = "handle")
    public String user(Long id) {
        return "要查询的用户id：" + id;
    }

    // 降级方法，注意返回值，形参要和正常方法一样；形参多一个BlockException
    public String handle(Long id, BlockException blockException) {
        return "服务有问题，降级了";
    }
~~~
2.抽取兜底方法放在一个类里
~~~java
    /** 服务降级 **/
    @RequestMapping("/user")
    @SentinelResource(value = "user", blockHandlerClass = MyBlockException.class, blockHandler = "handle")
    public String user(Long id) {
        return "要查询的用户id：" + id;
    }

public class MyBlockException {
    // 降级方法，注意static, 注意返回值，形参要和正常方法一样；形参多一个BlockException
    public static String handle(Long id, BlockException blockException) {
        return "服务有问题，降级了";
    }
}
~~~
3.blockException针对的是流量控制和服务降级，不能处理java异常，需要使用fallback来兜底java异常
~~~java
    /** 服务降级 **/
    @RequestMapping("/user")
    @SentinelResource(value = "user",
            blockHandlerClass = MyBlockException.class,
            blockHandler = "handle",
            fallbackClass = MyBlockException.class,
            fallback = "javaerror")
    public String user(Long id) {
        int i = 1/0;//java异常，由fallback里的javaerror来兜底
        return "要查询的用户id：" + id;
    }

public class MyBlockException {

    // 降级方法，注意static, 注意返回值，形参要和正常方法一样；形参多一个BlockException
    public static String handle(Long id, BlockException blockException) {
        return "服务有问题，降级了";
    }

    public static String javaerror(Long id) {
        return "服务有java异常";
    }
}
~~~

### 3.4 基于nacos实现sentinel流控和降级持久化
1.pom.xml
~~~xml
        <dependency>
            <groupId>com.alibaba.csp</groupId>
            <artifactId>sentinel-datasource-nacos</artifactId>
        </dependency>
~~~
2.application.yml
~~~yml
spring:
  application:
    name: consumer
  cloud:
    nacos:
      discovery:
        server-addr: 127.0.0.1:8848
    sentinel:
      transport:
        dashboard: 127.0.0.1:8080
        port: 8719
      datasource:
        flow: #此处flow为自定义数据源名
          nacos:
            server-addr: ${spring.cloud.nacos.discovery.server-addr}
            data-id: ${spring.application.name}-flow-rules
            gruopId: DEFAULT_GROUP
            data-type: json
            rule-type: flow #类型来自RuleType类
        degrade: #此处flow为自定义数据源名
          nacos:
            server-addr: ${spring.cloud.nacos.discovery.server-addr}
            data-id: ${spring.application.name}-degrade-rules
            gruopId: DEFAULT_GROUP
            data-type: json
            rule-type: degrade #类型来自RuleType类
~~~
3.在nacos配置列表里配置
~~~
1.flow
Data ID:consumer-flow-rules
Group:DEFAULT_GROUP
配置内容：
[
    {
        "resource":"getUser",
        "limitApp":"default",
        "grade":1,
        "count":1,
        "strategy":0,
        "controlBehavior":0,
        "clusterMode":false
    }
]

2.degrade
Data ID:consumer-degrade-rules
Group:DEFAULT_GROUP
配置内容：
[
    {
        "resource":"getUser",
        "grade":2,
        "count":1,
        "timeWindow":5
    }
]
~~~
### 3.5 nacos+sentinel+dubbo











