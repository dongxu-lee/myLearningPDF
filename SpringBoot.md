# SpringBoot学习笔记

### 代码及文档地址 
代码地址：
1. [https://github.com/dongxu-lee/boot_Spring](https://github.com/dongxu-lee/boot_Spring)

文档地址：
1. github文档地址：[https://github.com/dongxu-lee/myLearningPDF/blob/main/SpringBoot.md](https://github.com/dongxu-lee/myLearningPDF/blob/main/SpringBoot.md)
2. 语雀文档地址：[https://www.yuque.com/bailihang-3fszp/bkgbrq/trv8is](https://www.yuque.com/bailihang-3fszp/bkgbrq/trv8is)

## 第一部分 SpringBoot基础回顾
### 1.1 约定优于配置

## 第二部分 SpringBoot源码剖析
### 2.1 依赖管理
### 2.2 自动配置
### 2.3 SpringApplication实例化过程
1.实例化SpringApplication对象
 - 把项目启动类DemoApplication设置为属性存储起来
 - 设置应用类型是SERVLET应用还是REACTIVE应用
 - 设置初始化器Initializer
 - 设置监听器
 - 初始化mainApplicationClass 属性：用于推断并设置项目main()方法启动的主程序类

2.调用run方法



## 第三部分 SpringBoot高级进阶
### 3.1 整合mybatis、JPA、Redis
见代码地址

### 3.2 Thymeleaf常用标签
th:标签     | 说明
-------- | -----
th:insert  | 布局标签，替换内容到引入的文件
th:replace  | 布局标签，替换整个标签到引入的文件
th:each  | 元素遍历（类似JSP中的c:forEach标签）
th:if  | 条件判断，如果为真
th:unless  | 条件判断，如果为假
th:switch  | 条件判断，进行选择性匹配
th:case  | 条件判断，进行选择性匹配
th:value  | 属性值修改，指定标签属性值
th:href  | 用于设定链接地址
th:src  | 用于设定链接地址
th:text  | 用于指定标签显示的文本内容


### 3.3 Thymeleaf表达式

说明     | 表达式语法
-------- | -----
变量表达式  | ${...}
选择变量表达式  | *{...}
消息表达式  | #{...}
链接URL表达式  | @{...}
片段表达式  | ~{...}

1. 变量表达式 ${...}
获取上下文中的变量值
~~~html
<p th:text="${title}">这是标题</p>
~~~
如果上下文不存在title变量，默认显示“这是标题”，如果有，显示title变量值，当前文本被替换

2. 选择变量表达式 *{...}
从被选定对象中获取属性值
~~~html
<div th:object="${book}">
  <p>title:<span th:text="*{title}">标题</span></p>
</div>
~~~
*{title}获取指定对象book的title属性值

3. 消息表达式#{}
一般用于国际化

4. 链接表达式@{}
~~~html
<a th:href="@{http://localhost:8080/order/details(orderId=${o.id})}">view</a>
<a th:href="@{/order/details(orderId=${o.id})}">view</a>
~~~
上述分别使用了绝对地址和相对地址，在有参表达式中，需要按照@{路径(参数名称=参数值, 参数名称=参数值...)}的形式编写，同时参数值可以使用变量表达式动态赋值

5. 片段表达式~{}

最常见用法是使用th:insert或th:replace属性插入片段
~~~html
<div th:insert="~{thymeleafDemo::title}"></div>
~~~
thymeleafDemo是模板名称，Thymeleaf会自动寻找"/resources/templates/"目录下的thymeleafDemo模板，title是片段名称



