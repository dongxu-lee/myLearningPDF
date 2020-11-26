# 高性能RPC框架Apache 2Dubbo学习笔记

## *重点：Dubbo和SpringCloud的区别？
答：1. 注册的服务的区别

Dubbo是基于RPC协议来实现数据传输的，Provider对外暴露接口，Consumer根据接口的规则调用。也就是Provider向zookeeper注册的是接口信息，Consumer从zookeeper获取接口信息，然后调用。

SpringCloud的服务发现是基于http协议来实现的，Provider对外暴露应用信息，比如应用名称、ip地址等，Consumer获取应用信息，根据规则选择其中一个应用，然后依靠http协议发送请求。

<B>Dubbo暴露接口信息，SpringCloud暴露应用信息</B>

2.服务注册与治理

<B>Dubbo依赖zookeeper实现服务注册和治理，SpringCloud依赖Eureka实现服务发现和治理。</B>

3.服务更新机制

Dubbo依赖于zookeeper，zookeeper每次有变动，会通知所有订阅此节点的client，将当前数据全量同步给client。每次更新都同步全量数据。

Eureka在启动时从server做一次全量拉取，之后server会把变化的信息放到一个queue里，client从中获取增量信息。每次更新仅同步增量信息。

<B>Dubbo每次都是全量更新，SpringCloud第一次全量，此后增量</B>

4.服务更新反馈机制

<B>Dubbo依赖zookeeper，每次变动都是立即反馈，SpringCloud只有在client拉取时才知道变动，延时30秒</B>

5.节点性质的区别

<B>Dubbo有明显的Provider和Consumer角色，SpringCloud两种都是client</B>


### 代码及文档地址 
代码地址：
1. [https://github.com/dongxu-lee/dubbo_apache](https://github.com/dongxu-lee/dubbo_apache)

文档地址：
1. github文档地址：[https://github.com/dongxu-lee/myLearningPDF/blob/main/Dubbo.md](https://github.com/dongxu-lee/myLearningPDF/blob/main/Dubbo.md)
2. 语雀文档地址：[https://www.yuque.com/bailihang-3fszp/bkgbrq/iwnql8](https://www.yuque.com/bailihang-3fszp/bkgbrq/iwnql8)

## 第一部分：架构演变及Dubbo使用回顾
### 1.1 相关配置
#### dubbo:registry
代表该模块所使用的注册中心

1. id：当前有多个注册中心时，需要配置该值以区分
2. address：注册中心的访问地址
3. protocol：注册中心使用的挟制，比如zookeeper，直接写zookeeper://xxx.xxx.xxx.xxx:2181
4. timeout：注册超时

#### dubbo:protocol
数据传输使用的协议

1. id：在多个协议使用时需要指定
2. name：指定协议名称。默认使用dubbo

#### dubbo:service
指定当前需要对外暴露的服务信息。和dubbo:reference大致相同

1. interface：对外暴露的接口
2. ref：具体实现对象的引用，Ibanez指spring中的beanId
3. version：对外暴露的版本号

#### dubbo:reference
消费者的配置

1. id：指定该bean在注册到spring中的id
2. interface：服务接口名
3. version：当前服务版本
4. registry：指定所具体使用的注册中心地址
#### dubbo:method
指定具体方法级别在进行RPC操作时候的配置

1. name：指定方法名称
2. async：是否异步 默认false

#### dubbo:service和dubbo:reference
1. mock：方法调用出现错误时，当作服务降级来统一对外返回结果
2. timeout：当前方法或接口中所有方法的超时时间
3. check：启动时，检查生产者是否有该服务，一般设置为false
4. retries：指定当前服务执行时出现错误或超时时的重试机制
> 1.注意提供者是否有幂等，否则会有数据一致性问题
> 2.注意提供者是否有类似缓存机制，如出现错误时不断重试导致雪崩
5. executes：用于在提供者做配置，来确保最大的并行度
> 1.可能导致集群功能无法充分利用或者堵塞
> 2.但是也可以启动部分对应用的保护功能
> 3.可以不做配置，结合后面的熔断限流使用


## 第二部分：Dubbo深入配置和高级实战
### 2.1 JDK中的SPI
1. 当服务提供者提供了接口的一种具体实现后，在META-INF/services目录下创建一个以“接口全限定名”为命名的文件，文件内容是实现类的全限定名；
2. 接口实现类所在的jar包放在主程序的classpath中（就是主程序引用实现类所在项目的jar包）；
3. 主程序通过java.util.ServiceLoader动态装载实现模块，它通过扫描META-INF/services目录下的配置文件找到实现类的全限定类名，把类加载到JVM；
4. SPI的实现类必须有一个无参构造。

### 2.2 Dubbo中的SPI
1. 创建接口，接口要加dubbo注解@SPI；
2. 编写实现类并被主程序引入，实现类resources目录下创建META-INF/dubbo目录，目录下创建一个以“接口全限定名”为命名的文件，文件内容是“形参=实现类的全限定名”，如human=com.ldx.HumanServiceImpl；
3. 主程序使用ExtensionLoader（扩展加载器）加载实现类；

### 2.3 Dubbo中的Adaptive
利用URL动态选择实现类 
### 2.4 Dubbo过滤器
~~~java
@Activate(group={CommonConstants.CONSUMER,CommonConstants.PROVIDER})
public class DubboInvokeFilter implements Filter {
	@Override
	public Result invoke(Invoker<?> invoker, Invocation invocation) throws RpcException {
		try{
			return invoker.invoke(invocation);
		}catch(Exception e) {
			...
		}
		return null;
	}
}
~~~
### 2.5 负载均衡
可以在服务提供方配置，也可以在消费者方配置
~~~java
// 消费者方
@Reference(check=false,loadbalance="random")

// 服务提供方
@Service(loadbalance="random")
~~~

可以自定义负载均衡

### 2.6 异步调用
配置method的async属性
### 2.7 线程池
Dubbo有两种线程池，fix固定大小线程池、cache缓存线程池。默认fix，大小是200。
### 2.8 路由



