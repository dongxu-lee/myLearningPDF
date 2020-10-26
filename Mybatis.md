# Mybatis学习笔记

### 代码及文档地址 
代码地址：
1. [https://github.com/dongxu-lee/IPersistence_Mybatis](https://github.com/dongxu-lee/IPersistence_Mybatis)

文档地址：
1. github文档地址：[https://github.com/dongxu-lee/myLearningPDF/blob/main/Mybatis.md](https://github.com/dongxu-lee/myLearningPDF/blob/main/Mybatis.md)
2. 语雀文档地址：[https://www.yuque.com/bailihang-3fszp/bkgbrq/phfogm](https://www.yuque.com/bailihang-3fszp/bkgbrq/phfogm)

## 第一部分 自定义持久层框架
### 1.1 JDBC问题分析
1. 数据库配置信息存在硬编码问题（解决：配置文件）
2. 频繁创建释放数据库连接（解决：连接池）
3. sql语句、设置参数、获取结果集参数存在硬编码问题（解决：配置文件）
4. 手动封装返回结果集（解决：反射、内省）
### 1.2 自定义持久层框架设计分析
#### 使用端：（项目）：引入自定义持久层框架的jar包
提供两部分配置信息：数据库配置信息、sql配置信息：sql语句、参数类型、返回值类型<br>
使用配置文件来提供这两部分配置信息：
- (1) sqlMapConfig.xml：存放数据库配置信息，存放mapper.xml的全路径
- (2) mapper.xml：存放sql配置信息

#### 自定义持久层框架本身：【工程】：本质就是对JDBC代码进行了封装
1. 加载配置文件：根据配置文件路径，加载配置文件成字节输入流，存储在内存中
    > 创建Resources类  方法：InputStream getResourcesAsStream(String path)
    
2. 创建两个javaBean：【容器对象】：存放的就是对配置文件解析出来的内容
    > Configuration：核心配置类：存放sqlMapConfig.xml解析出来的内容<br>
    > MapperedStatement：映射配置类：存放mapper.xml解析出来的内容

3. 解析
    > 创建类：SqlSessionFactoryBuilder  方法：build(inputStream in)<br>
    > 第一：使用dom4j解析配置文件，把解析出的内容封装到容器对象中<br>
    > 第二：创建SqlSessionFactory对象；生产sqlSession：会话对象（<font color=red>工厂模式</font>）

4. 创建SqlSessionFactory接口及实现类DefaultSqlSessionFactory
    > openSession()：生产sqlSession

5. 创建SqlSession接口及实现类DefaultSession
    > 定义对数据库的CRUD操作

6. 创建Executor接口及实现类SimpleExecutor实现类
    > 执行的就是JDBC代码



## 第二部分 Mybatis基础回顾及高级应用
### 1.1 Mybatis相关概念

ORM：对象关系映射

简介：Mybatis是一款基于ORM的半自动轻量级的持久层框架。


### 1.2 Mybatis常用配置解析
##### 1) environments标签

```xml
<environments default="development"> # --> default指定默认的环境名称
    <environment id="development"> # --> id指定当前环境的名称
        <transactionManager type="JDBC" /> # --> type=“JDBC” 指定事务管理类型是JDBC
        <dataSource type="POOLED"> # --> 指定当前的数据源类型是连接池
            <property name="driver" value="${jdbc.driver}" /> # --> 数据源配置的基本参数
            <property name="url" value="${jdbc.url}" />
            <property name="username" value="${jdbc.username}" />
            <property name="password" value="${jdbc.password}" />
        </dataSource>
    </environment>
</environments>
```

事务管理器transactionManager有两种：
 - JDBC：依赖从数据源得到的连接来管理事务，即依赖数据库
 - MANAGED：几乎什么也不做，由代码来实现事务整个过程

数据源dataSource有三种：
 - UNPOOLED：不使用连接池，每次请求都要建新的连接
 - POOLED：使用JDBC连接池
 - JNDI：为了能在EJB或应用服务器这类容器中使用


##### 2) mapper标签
 - 使用相对于类路径的资源引用
```xml
<mapper resource="com/ldx/AbcMapper.xml" />
```
 - 使用完全限定资源定位符（URL）
```xml
<mapper url="file:///var/Abc.xml" />
```
 - 使用映射器接口实现类的完全限定类名
```xml
<mapper class="com.ldx.AbcMapper" />
```
 - 将包内的映射器接口实现全部注册为映射器
```xml
<package name="com.ldx" />
```

### 1.3 相关API

```java
//1.Resource工具类，配置文件的加载，把配置文件家在城字节输入流
InputStream resourceAsStream = Resources.getResourceAsStream("sqlMapConfig.xml");
//2.解析了配置文件，并创建了sqlSessionFactory工厂
SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(resourceAsStream);
//3.生产sqlSession。默认开是一个事务，但不会自动提交 
//在进行增删改时，要手动提交事务，sqlSession.commit()
//如果openSession使用openSession(true)参数，事务自动提交
SqlSession sqlSession = sqlSessionFactory.openSession();
sqlSession.close();
//sqlSession.commit();
```


### 1.4 概念深入

```
在sqlMapConfig.xml的<configuration>标签中
```

##### 1) Properties 导入配置文件

```xml
<!-- 位置必须是标签里的第一位 -->
<properties resource="jdbc.properties" />
```

##### 2) typeAliases 别名

```xml
<typeAliases>
	<!-- 给单独的实体起别名 -->
    <!-- <typeAlias type="com.ldx.pojo.User" alias="user"></typeAlias> -->
    <!-- 批量起别名：该包下的所有类起别名，不区分大小写 -->
    <package name="com.ldx.pojo"></package>
</typeAliases>
```


### 1.5 动态SQL
```xml
<!-- 动态之if -->
<if test="id!=null">
  id = #{id}
</if>

<!-- 动态之foreach -->
<select id="findByIds" parameterType="list" resultType="user">
    select * from user
    <where>
        <foreach collection="array" open="id in (" close=")" item="id" separator=",">
            #{id}
        </foreach>
    </where>
</select>
```

### 1.6 Mybatis缓存
##### 一级缓存
同一个sqlSession里，相同的sql查询，除第一次外的查询直接从缓存里获取数据。

> 一、一级缓存到底是什么？
> 答：一个HashMap成员变量
> 二、一级缓存何时被创建？
> 答：在执行器Executor中创建（方法createCacheKey()）
> 三、createCacheKey方法何时调用？
> 答：Executor的query方法中判断缓存是否有cacheKey，有就直接返回，没有则创建缓存


##### 二级缓存
二级缓存是基于mapper文件的namespace的，也就是多个sqlSession可以共享一个mapper中的二级缓存区域。如果两个mapper的namespace相同，两个mapper的sql查询也共享同一个二级缓存区域。



### 1.7 Mybatis插件
原理：拦截器


## 第三部分 Mybatis源码剖析

### 1.1 层次结构
##### 1) SqlSession
作为Mybatis工作的顶层API接口，作为会话访问，完成增删改查功能

##### 2) Executor
Mybatis执行器，是Mybatis的核心，负责SQL动态语句的生成和查询缓存的维护

##### 3) StatementHandler
负责处理JDBC的Statement的交互，包括对Statement设置参数，以及将JDBC返回的resultSet结果集转换成List

###### BoundSql-->MappedStatement(SqlSource、ResultMap)  in  Configuration

##### 4.1) ParameterHandler （Mybatis-->JDBC）
负责根据传递的参数值，对Statement对象设置参数

##### 4.2) ResultSetHandler（JDBC-->Mbatis）
负责将resultSet集合转换为List

##### 5) TypeHandler<T>
负责jdbcType与javaType之间的数据转换：
1. 负责对Statement对象设置特定的参数；
2. 对Statement返回的结果集resultSet，取出特定的列

##### 6) ResultSet（JDBC）
返回结果集

###### Statement (PrepareStatement、SimpleStatement、CallableStatement)

### 1.2 Mybatis实现方式
1. 传统方式
2. mapper代理方式：JDK动态代理

> 代理方式：获取到的mapper对象都是代理对象，调用任何mapper方法，底层都是调用代理对象的invoke()方法


### 1.3 设计模式--Builder构建者模式
原理：使用多个简单对象一步一步构建成一个复杂的对象


在Mybatis中的应用：核心对象Configuration使用了XmlConfigBuilder来进行构造

```java
private void parseConfiguration(XNode root) {
    try {
      // issue #117 read properties first
      propertiesElement(root.evalNode("properties"));
      Properties settings = settingsAsProperties(root.evalNode("settings"));
      loadCustomVfs(settings);
      loadCustomLogImpl(settings);
      typeAliasesElement(root.evalNode("typeAliases"));
      pluginElement(root.evalNode("plugins"));
      objectFactoryElement(root.evalNode("objectFactory"));
      objectWrapperFactoryElement(root.evalNode("objectWrapperFactory"));
      reflectorFactoryElement(root.evalNode("reflectorFactory"));
      settingsElement(settings);
      // read it after objectFactory and objectWrapperFactory issue #631
      environmentsElement(root.evalNode("environments"));
      databaseIdProviderElement(root.evalNode("databaseIdProvider"));
      typeHandlerElement(root.evalNode("typeHandlers"));
      mapperElement(root.evalNode("mappers"));
    } catch (Exception e) {
      throw new BuilderException("Error parsing SQL Mapper Configuration. Cause: " + e, e);
    }
  }
```



### 1.4 设计模式--工厂模式
##### 简单工厂模式
原理：对不同实例的创建做了一层封装，这些实例都有相同父类


在Mybatis中的应用：Mybatis在核心接口SqlSession的创建过程用到了工厂模式

##### 工厂方法模式
原理：和简单工厂模式中工厂负责生产所有产品相比，工厂方法模式将生成具体产品的任务分发给具体的产品工厂


##### 抽象工厂模式
原理：和工厂方法模式相比，抽象工厂模式可以生产更多产品




### 1.5 设计模式--代理模式
原理：给某个对象提供一个代理，并由代理对象控制对原对象的引用。
##### 动态代理


在Mybatis中的应用：代理模式可以认为是Mybatis的核心模式，正是由于这个模式，我们只需要写mapper接口，就可以由Myabtis后台帮我们实现具体SQL的执行。
> 当我们使用Configuration的getMapper方法是，会调用mapperRegistry.getMapper方法，而该方法又会调用mapperProxyFactory.newInstance(sqlSession)来生成一个具体的代理。

##### 静态代理


### 设计模式代码地址
[https://github.com/dongxu-lee/pattern](https://github.com/dongxu-lee/pattern)
























