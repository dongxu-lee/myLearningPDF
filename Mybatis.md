# Mybatis学习笔记
## 第一部分 自定义持久层框架
### 1.1 JDBC问题分析
1. 数据库配置信息存在硬编码问题（解决：配置文件）
2. 频繁创建释放数据库连接（解决：连接池）
3. sql语句、设置参数、获取结果集参数存在硬编码问题（解决：配置文件）
4. 手动封装返回结果集（解决：反射、内省）
### 1.2 自定义持久层框架设计分析
1. 加载配置文件：根据配置文件路径，加载配置文件成字节输入流，存储在内存中
    - 创建Resources类  方法：InputStream getResourcesAsStream(String path)
2. 创建两个javaBean：存放解析出的配置文件的内容
    - Configuration
    - MapperedStatement
3. 解析配置文件: dom4j
    - 使用dom4j解析配置文件，把解析出的内容封装到容器对象中
    - 创建SqlSessionFactory对象；生产sqlSession：会话对象（工厂模式）
4. 创建SqlSessionFactory接口及实现类DefaultSqlSessionFactory
    - openSession()：生产sqlSession
5. 创建SqlSession接口及实现类DefaultSession
    - 定义对数据库的CRUD操作
6. 创建Executor接口及实现类SimpleExecutor实现类
    - 执行的就是JDBC
  



