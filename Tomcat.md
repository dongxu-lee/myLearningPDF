# Tomcat学习笔记

### 代码及文档地址 
代码地址：
1. [123](123)

文档地址：
1. github文档地址：[123](123)
2. 语雀文档地址：[123](123)


## 第一部分 Tomcat高级使用及其原理剖析
### 1.1 浏览器访问服务器的流程
![在这里插入图片描述](https://cdn.nlark.com/yuque/0/2020/png/2739510/1604999301824-0d97617a-534a-414c-919d-b35920576e23.png?x-oss-process=image%2Fresize%2Cw_746)
### 1.2 Tomcat处理请求大致流程

Tomcat兼具<B>HTTP服务器</B>和<B>Servlet容器</B>的功能

> HTTP服务器在接收到请求之后把请求交给Servlet容器来处理，Servlet容器通过Servlet接口调用业务类。Servlet接口和Servlet容器这一整套内容叫作Servlet规范。

### 1.3 Servlet容器处理请求流程
当用户请求某个URL资源时：
1) HTTP服务器会把请求信息使用ServletRequest对象封装起来
2) 进一步去调用Servlet容器中某个具体的Servlet
3) 在2)中，Servlet容器拿到请求后，根据URL和Servlet的映射关系，找到相应的Servlet
4) 如果Servlet还没有被加载，就用反射机制创建这个Servlet，并调用Servlet的init方法来完成初始化
5) 接着调用这个具体Servlet的service方法来处理请求，请求处理结果使用ServletResponse对象封装
6) 把ServletResponse对象返回给HTTP服务器，HTTP服务器会把响应发送给客户端

### 1.4 Tomcat总体架构
Tomcat设计了两个核心组件<B>连接器（Connector）</B>和<B>容器（Container）</B>来完成Tomcat两大核心功能；

<B>连接器，负责对外交流</B>：处理Socket连接，负责网络字节流与Request和Response对象的转化；

<B>容器，负责内部处理</B>：加载和管理Servlet，以及具体处理Request请求


### 1.5 连接器组件Coyote简介及支持的协议和IO模型
#### Coyote简介
Coyote是Tomcat中连接器的组件名称，是对外的接口。客户端通过Coyote与服务器建立连接、发送请求并接受响应。

 - Coyote封装了底层的网络通信（Socket请求及响应处理）
 - Coyote使Catalina容器与具体的请求协议及IO操作方式完全解耦
 - Coyote将Socket输入转换为Request对象，进一步封装后交由Catalina容器进行处理，处理请求完成后，Catalina通过Coyote提供的Response对象将结果写入输出流
 - Coyote负责的是具体协议（应用层）和IO（传输层）相关内容

#### Tomcat Coyote支持的IO模型与协议
应用层：
应用层协议     | 描述
-------- | -----
HTTP/1.1  | 大部分Web应用采用的访问协议
AJP  | 用于和WX集成（如Apache），一实现对静态资源的优化及集群部署
HTTP/2  | HHTP2.0大幅提升了Web性能

传输层：
IO模型     | 描述
-------- | -----
NIO  | 非阻塞IO，采用Java NIO类库实现
NIO2  | 异步IO，采用JDK 7最新的NIO2类库实现
APR  | 采用Apache可移植运行库实现，需要单独安装APR库

### 1.6 Tomcat Servlet容器Catalina
![在这里插入图片描述](https://cdn.nlark.com/yuque/0/2020/png/2739510/1604999337735-b6a348dc-62df-49f6-ae78-d63923a59d1d.png)


### 1.7 Catalina的结构

Tomcat就是一个Catalina的实例，Tomcat启动时会初始化这个实例，通过加载server.xml完成其他实例的创建和管理，Server创建并管理多个服务，每个服务又有多个Connector和一个Container

### 1.8 Tomcat服务器核心配置详解及Server标签
地址：tomcat目录下conf/server.xml文件

~~~xml
<!-- 创建一个Server实例 -->
<server>
  <!-- 定义监听器 -->
  <listener />
  <!-- 定义服务器的全局JNDI资源 -->
  <GlobalNamingResources />
  <!-- 定义一个Service服务，一个Server标签可以有多个Service服务实例 -->
  <service />
</server>
~~~


### 1.9 Executor标签和Connector标签
~~~xml
<!--
默认情况下，没有共享线程池配置，如果想配置，配置如下：
name：线程池名称
namePrefix：创建的每个线程的名称前缀，单个线程名称：namePrefix+threadNumber
maxThreads：最大线程数
minSpareThreads：活跃线程数
maxIdleTime：线程空闲时间
maxQueueSize：排队最大线程数目
prestartminSpareThreads：启动线程池时是否启动minSpareThreads部分线程
threadPriority：线程池中线程优先级，默认5，值从1到10
className：线程池实现类
-->
<Executor name="commonThreadPool"
	namePrefix="thread-exec-"
	maxThreads="200"
	minSpareThreads="100"
	maxIdleTime="60000"
	maxQueueSize="Integer.MAX_VALUE"
	prestartminSpareThreads="false"
	threadPriority="5"
	className="org.apache.catalina.core.StandardThreadExecutor" />
~~~

~~~xml
<!-- redirectPort:当前service不支持SSL，如果当前请求符合规范，但需要SSL传输，则会把请求转发到redirectPort设置的端口 -->
<connector port="8080" protocol="HTTP/1.1" execuotr="commonThreadPool"
		connectionTimeout="20000"
		redirectPort="8443" URIEncoding="UTF-8" />
~~~

### 1.10 Engine标签和Host标签
Engine用于配置一个虚拟机引擎：
~~~xml
<!--
name：指定Engine名称
defaultHost：默认虚拟机名称，对应下面Host标签中的name。
             当客户端请求指向的主机无效时，请求发给默认的localhost
-->
<Engine name="Catalina" defaultHost="localhost" />
~~~
Host用于配置一个虚拟主机：
~~~xml
<!--
name：请求域名
appBase：项目所在目录
-->
<Host name="www.abc.com" appBase="webapps"
	  unpackWARs="true" autoDeploy="true" />
~~~

### 1.11 Context标签
Context用于配置一个web应用
~~~xml
<!--
docBase：Web应用目录或者War包的部署路径。
         可以是绝对路径，也可以是相对于Host appBase的相对路径
path：Web应用的Context路径.
      比如Host是www.abc.com，访问地址为http://www.abc.com:8080/myweb
-->
<Context docBase="/Users/tdky/web_demo" path="myweb" />
~~~

























## 第二部分 Tomcat源码剖析及调优
### 2.1 
















