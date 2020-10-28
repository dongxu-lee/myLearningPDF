# Spring学习笔记

### 代码及文档地址 
代码地址：
1. [https://github.com/dongxu-lee/IoCAOP_Spring](https://github.com/dongxu-lee/IoCAOP_Spring)

文档地址：
1. github文档地址：[https://github.com/dongxu-lee/myLearningPDF/blob/main/Spring.md](https://github.com/dongxu-lee/myLearningPDF/blob/main/Spring.md)
2. 语雀文档地址：[123](123)


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
### 3.1







