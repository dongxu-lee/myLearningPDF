# Mybatis学习笔记
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

### 代码及文档地址 
代码地址：
1. [https://github.com/dongxu-lee/IPersistence_Mybatis](https://github.com/dongxu-lee/IPersistence_Mybatis)

文档地址：
1. github文档地址：[https://github.com/dongxu-lee/myLearningPDF/blob/main/Mybatis.md](https://github.com/dongxu-lee/myLearningPDF/blob/main/Mybatis.md)
2. 语雀文档地址：[https://www.yuque.com/bailihang-3fszp/bkgbrq/phfogm](https://www.yuque.com/bailihang-3fszp/bkgbrq/phfogm)


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





















