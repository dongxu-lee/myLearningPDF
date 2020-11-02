# Spring学习笔记

### 代码及文档地址 
代码地址：
1. [https://github.com/dongxu-lee/IoCAOP_Spring](https://github.com/dongxu-lee/IoCAOP_Spring)

文档地址：
1. github文档地址：[https://github.com/dongxu-lee/myLearningPDF/blob/main/Spring.md](https://github.com/dongxu-lee/myLearningPDF/blob/main/Spring.md)
2. 语雀文档地址：[https://www.yuque.com/bailihang-3fszp/bkgbrq/zcg13k](https://www.yuque.com/bailihang-3fszp/bkgbrq/zcg13k)


## 第一部分 自定义Ioc&AOP框架
### 1.1 IoC

##### 什么是IoC？
由IoC容器负责实例化对象并管理，需要哪个对象，去找IoC容器要

控制：对象创建的权利

翻转：控制权交给外部环境（spring框架、IoC容器）


### 1.2 AOP

##### AOP解决的问题？

在不改变原有业务逻辑的情况下，悄无声息的把横切逻辑代码应用到原有的业务逻辑中





## 第二部分 Spring IoC高级应用与源码剖析
### 2.1 Spring IoC基础知识说明
##### 1. 纯xml（bean信息定义全部配置在xml中）
##### 2. xml+注解（部分在xml，部分用注解）
上述两种的启动方式：

JavaSE应用：

~~~java
ApplicationContext applicationContext = new ClassPathXmlApplicationContext("beans.xml");
或者
new FileSystemXmlApplicationContext("c:/beans.xml");
~~~

JavaWeb应用：

~~~java
ContextLoaderListener(监听器去加载xml)
~~~


##### 3. 纯注解
启动方式：

JavaSE应用：

~~~java
ApplicationContext applicationContext = new AnnotationConfigApplicationContext(SpringConfig.class);
~~~

JavaWeb应用：

~~~java
ContextLoaderListener(监听器去加载注解配置类)
~~~

### 2.2 BeanFactory和ApplicationContext的区别
BeanFactory是顶层接口；

ApplicationContext是BeanFactory的一个子接口。

### 2.3 纯xml模式
1. applicationContext.xml

~~~xml
<!-- Spring ioc 实例化Bean的三种方式 -->
<!-- 方法一：使用无参构造器（推荐） -->
<bean id="connectionUtils" class="com.ldx.utils.ConnectionUtils"></bean>
~~~


2. web.xml

~~~xml
<!-- 配置文件 -->
<context-param>
  <param-name>contextConfigLocation</param-name>
  <param-value>classpath:applicationContext.xml</param-value>
</context-param>
<!-- 使用监听器启动Spring的IoC容器 -->
<listen>
  <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
</listen>
~~~

3. MyServlet.java 自定义servlet

~~~java
public class MyServlet extends HttpServlet {
    private UserService userService = null;
	@Override
	public void init() throws ServletException {
		WebApplicationContext webApplicationContext = WebApplicationContextUtils.getWebApplicationContext(this.getServletContext());
		ProxyFactory proxyFactory proxyFactory = (ProxyFactory) webApplicationContext.getBean("proxyFactory");
		UserService userService = (UserService) proxyFactory.getJdkProxy(wenApplicationContext.getBean("userService"));
	}
}
~~~


### 2.4 Bean创建的方式以及Bean标签属性
##### Spring实例化Bean的三种方式
~~~xml
<!-- Spring ioc 实例化Bean的三种方式 -->
<!-- 方法一：使用无参构造器（推荐） -->
<bean id="connectionUtils" class="com.ldx.utils.ConnectionUtils"></bean>

<!-- 另外两种方式是为了我们自己new的对象加入到IOC管理 -->
<!-- 方法二：静态方法 -->
<bean id="connectionUtils" class="com.ldx.factory.CreateBeanFactory" factory-method="getInstanceStatic"></bean>

<!-- 方法三：实例化方法 -->
<bean id="createBeanFactory" class="com.ldx.factory.CreateBeanFactory"></bean>
<bean id="connectionUtils" factory-bean="createBeanFactory" factory-method="getInstance"></bean>
~~~


~~~java
public class CreateBeanFactory {

	//1.无参构造
	public CreateBeanFactory() {}
	//2.静态方法
	public static CreateBeanFactory  getInstanceStatic() {
		return new CreateBeanFactory();
	}
}
~~~

##### Bean的作用范围
~~~xml
<bean scope="singleton" />
~~~
singleton：到哪里，IOC容器中只有一个该对象，默认为singleton

prototype：原型（多例），每次都新建对象，Spring只创建对象，不管理对象

##### bean的其他属性
~~~xml
<bean init-method="init" destory-method="destory" />
~~~


### 2.5 高级特性之lazy-init延迟加载
ApplicationContext默认启动服务器时对所有bean进行实例化

延迟加载是在第一次向容器getBean时实例化的
~~~xml
<bean id="" class="" lazy-init="true" />
~~~


### 2.6 高级特性之FactoryBean
##### FactoryBean和BeanFactory

BeanFactory接口是容器的顶级接口，定义了容器的一些基础行为。

Spring中有两种bean，一种是普通bean，一种是工厂bean，FactoryBean可以生成某一个类型的bean实例，也就是说我们可以借助它实现自定义bean的创建过程。

### 2.7 高级特性之后置处理器

## 第三部分 Spring AOP高级应用与源码剖析
### 3.1 AOP术语

Joinpoint 连接点：可以用于把增强代码加入到业务主线中的点，这些点就是方法。

Pointcut 切入点：哪些已经把增强代码加入到业务主线进来之后的连接点

Advice 通知/增强：切面类中用于增强功能的方法

Target 目标对象：被代理对象

Proxy 代理：代理对象

Weaving 织入：把增强应用到目标对象来创建新代理对象的过程

Aspect 切面：增强代码定义在一个类里，这个类就是切面类


### 3.2 纯xml
~~~xml
<!-- 横切逻辑bean -->
<bean id="logUtils" class="com.ldx.utils.LogUtils">
<!-- aspect = 锁定方法 + 锁定方法的特殊时机 + 横切逻辑 -->
<aop:config>
  <aop:aspect id="logAspect" ref="logUtils">

	<!-- 切入点锁定方法，使用aspectj语法表达式 -->
    <aop:pointcut id="pt1" expression="execution(public void com.ldx.service.impl.UserImpl(java.lang.String))">

	<aop:before ... />
	<aop:after ... />
	<aop:after-returning ... />
	<aop:after-throwing ... />
    <aop:around method="arroundMethod" pointcut-ref="pt1" />
  </aop:aspect>
</aop:config>
~~~

### 3.3 半注解，半xml

~~~xml
<!-- 开启注解驱动 -->
<apo:aspectj-autoproxy />
~~~

### 3.4 事务传播策略
<B>PROPAGATION_xxx</B>

REQUIRED  当前没有事务就创建一个事务，有就加入当前事务

SUPPORTS 支持当前事务，如果当前没有事务，就以非事务方式执行

MANDATORY 使用当前事务，如果没有抛异常

REQUIRES_NEW 新建事务，如果存在当前事务，当前事务挂起

NEVER 以非事务运行，有事务抛异常

NESTED 如果当前存在事务，则嵌套事务内执行，如果当前没有事务，执行类似REQUIRED的操作


### 3.5 声明式事务纯xml模式
~~~xml
<!-- 开启注解扫描，base-package指定扫描的包路径 -->
<context:component-scan base-package="com.ldx.edu" />

<!-- 引入外部资源文件 -->
<context:property-placeholder location="classpath:jdbc.properties" />

<!-- 第三方jar中的bean定义在xml中 -->
<bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
    <property name="driverClassName" value="${jdbc.driver}" />
    <property name="url" value="${jdbc.url}" />
    <property name="username" value="${jdbc.username}" />
    <property name="password" value="${jdbc.password}" />
</bean>

<bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
    <constructor-arg name="dataSource" ref="dataSource" />
</bean>

<!-- spring声明式事务配置，声明式事务就是一个aop -->
<bean id="transactionManager" class="org.springframework.jdbc.dataSource.DataSourceTransactionManager">
    <constructor-arg name="dataSource" ref="dataSource"></constructor-arg>
</bean>

<tx:advice id="txAdvice" transaction-manager="transactionManager">
    <tx:attributes>
    	<!-- 一般性配置 -->
    	<tx:method name="*" read-only="false" propagation="REQUIRED" isolation="DEFAULT" timeout="-1" />
    	<!-- 针对查询的覆盖性配置 -->
    	<tx:method name="query*" read-only="true" propagation="SUPPORTS" />
    </tx:attributes>
</tx:advice>

<aop:config>
    <!-- advice-ref指向增强=横切逻辑+方位 -->
    <aop:advisor advice-ref="txAdvice" pointcut="execution(* com.ldx.edu.service.impl.TransServiceImpl.*(..))">
</aop:config>
~~~

### 3.6 声明式事务半注解及全注解
##### 半注解
~~~xml
<!-- 开启注解扫描，base-package指定扫描的包路径 -->
<context:component-scan base-package="com.ldx.edu" />

<!-- 引入外部资源文件 -->
<context:property-placeholder location="classpath:jdbc.properties" />

<!-- 第三方jar中的bean定义在xml中 -->
<bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
    <property name="driverClassName" value="${jdbc.driver}" />
    <property name="url" value="${jdbc.url}" />
    <property name="username" value="${jdbc.username}" />
    <property name="password" value="${jdbc.password}" />
</bean>

<bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
    <constructor-arg name="dataSource" ref="dataSource" />
</bean>

<!-- spring声明式事务配置，声明式事务就是一个aop -->
<bean id="transactionManager" class="org.springframework.jdbc.dataSource.DataSourceTransactionManager">
    <constructor-arg name="dataSource" ref="dataSource"></constructor-arg>
</bean>

<!-- <tx:advice id="txAdvice" transaction-manager="transactionManager">
    <tx:attributes>
    	<!-- 一般性配置 -->
    	<tx:method name="*" read-only="false" propagation="REQUIRED" isolation="DEFAULT" timeout="-1" />
    	<!-- 针对查询的覆盖性配置 -->
    	<tx:method name="query*" read-only="true" propagation="SUPPORTS" />
    </tx:attributes>
</tx:advice>

<aop:config>
    <!-- advice-ref指向增强=横切逻辑+方位 -->
    <aop:advisor advice-ref="txAdvice" pointcut="execution(* com.ldx.edu.service.impl.TransServiceImpl.*(..))">
</aop:config>-->

<!-- 开启注解驱动 -->
<tx:annotation-driven transaction-manager="transactionManager" />
~~~

配置上述xml后，在serviceImpl上使用@Transactional注解即可


##### 纯注解

在spring配置类上加@EnableTransactionManagement





