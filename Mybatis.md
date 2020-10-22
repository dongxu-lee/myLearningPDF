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



