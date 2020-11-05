# Spring MVC学习笔记

### 代码及文档地址 
代码地址：
1. [https://github.com/dongxu-lee/MVC_Spring](https://github.com/dongxu-lee/MVC_Spring)

文档地址：
1. github文档地址：[https://github.com/dongxu-lee/myLearningPDF/blob/main/SpringMVC.md](https://github.com/dongxu-lee/myLearningPDF/blob/main/SpringMVC.md)
2. 语雀文档地址：[https://www.yuque.com/bailihang-3fszp/bkgbrq/pgrrkt](https://www.yuque.com/bailihang-3fszp/bkgbrq/pgrrkt)


## 第一部分 SpringMVC基础回顾及其高级深入
### 1.1 SpringMVC流程

### 1.2 SpringMVC九大组件
- HandlerMapping（处理器映射器）
请求到达后，HandlerMapping找到请求对应的处理器Handler和Interceptor
- HandlerAdapter（处理器适配器）
让固定的Servlet处理方法调用Handler来进行处理
- HandlerExceptionResolver
处理Handler产生的异常情况
- ViewResolver（视图解析器）
解析视图
- RequestToViewNameTranslator
从请求中获取ViewName
- LocaleResolver
从请求获取Locale
- ThemeResolver
解析主题
- MultipartResolver
上传请求
- FlashMapManager
FlashMap可以用于重定向时携带参数，FlashMapManager用于管理FlashMap

### 1.3 url-pattern配置
一、后缀，如：*.do

二、 / 不会拦截.jsp，但会拦截静态资源

解决方案一：
~~~xml
<!-- 把对静态资源的处理交给tomcat -->
<mvc:default-servlet-handler />
~~~

解决方案二：
~~~xml
<!-- mvc框架自己处理静态资源 -->
<mvc:reosurces location="classpath:/" mapping="/resource/**" />
~~~

三、 /* 会拦截所有，包括.jsp


### 1.4 数据输出机制Model、Map、ModelMap
##### ModelMap
~~~java
@RequestMapping("/handle11")
public String handle11(ModelMap modelMap) {
    Date date = new Date();
    mpdelMap.addAttribute("date", date);
    return "success";
}
~~~

##### Model
~~~java
@RequestMapping("/handle12")
public String handle12(Model model) {
    Date date = new Date();
    model.addAttribute("date", date);
    return "success";
~~~

#####  Map
~~~java
@RequestMapping("/handle13")
public String handle13(Map<String, Object> map) {
    Date date = new Date();
    map.put("date", date);
    return "success";
~~~

### 1.5 包装类型参数
~~~java
// 包装类型
public class QueryVo {
    private String fee;
    private int mount;
    private User user;
    // getter and setter
}

// 包含在包装类型李的pojo类User
public class User {
    private int id;
    private String name;
    // getter and setter
}

// handler
@RequestMapping("/handle")
public Stirng handle(Query query) {
    // ...
}

//请求url，使用点号进一步赋值
// http://localhost:8080/handle?user.id=1&user.name='jack'
~~~

### 1.6 日期类型参数
~~~java
// handler
@Requestmapping("/handle")
public String handle(Date birthday) {
    // ...
}
// 需要定义日期类型转换器来自动转换，否则参数会被认为是String类型导致报错

// 日期类型转换器
public class DateConverter implements Converter<String, Date> {
    @Override
    public Date convert(String source) {
        SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd);
        try {
			Date parse = sdf.parse(source);
			return parse;
		} catch(Exception e) {
			e.printStackTrace();
		}
		return null;
    }
}

// 在配置文件里注册，配置如下：
~~~

~~~xml
<!-- 自动注册最合适的处理器映射器，处理器适配器 -->
<mvc:annotation-driven conversion-service="conversionServiceBean">

<!-- 注册自定义类型转换器 -->
<bean id="conversionServiceBean" class="org.springframework.format.support.FormattingConversionServiceFactoryBean">
    <property>
		<set>
			<bean class="com.ldx.edu.converter.DateConverter"></bean>
		</set>
	</property>
</bean>
~~~

### 1.7 监听器、过滤器、拦截器对比

 - Servlet：处理Request请求和Response响应
 - 过滤器（Filter）：对Request请求起到过滤作用，作用在Servlet之前
 - 监听器（Listener）：随Web应用的启动而启动
 - 拦截器（Interceptor）：是SpringMVC等表现层框架自己的，只会拦截访问的控制器方法

从配置的角度也能看到：servlet、filter、listener都是在web.xml中配置的，interceptor是在表现层框架的配置文件里配置的


### 1.8 多个拦截器执行流程

如有两个拦截器，按顺序定义并注册为interceptor01、interceptor02，实际执行顺序为：

- preHandler01
- preHandler02
- postHandler02
- postHandler01
- afterHandler02
- afterHandler01

### 1.9 SpringMVC异常处理
@ControllerAdvice

@ExceptionHandler


### 1.10 重定向传递flash属性
~~~java
@RequestMapping("/handleRedirect")
public String handleRedircet(String name, RedirectAttributes redirectAttributes) {
	//return "redirect:handle01?name=" + name; // 拼接参数安全性、参数长度都有局限
	redirectAttributes.addFlashAttribute("name", name);
	return "redirect:handle01";
}
~~~




## 第二部分 自定义MVC框架
见代码地址
## 第三部分 SpringMVC源码剖析及其SSM整合
见代码地址


## 第四部分 SpringData高级应用及其源码剖析

### 4.1 Spring Data JPA介绍
dao层框架，和Mybatis在使用方式和底层机制不同


### 4.2 重要代码
具体应用：见代码地址
~~~java
package com.ldx.dao;

import com.ldx.pojo.Resume;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.JpaSpecificationExecutor;
import org.springframework.data.jpa.repository.Query;

import java.util.List;

/**
 * JpaRepository<操作的实体类型，主键类型>，封装了CRUD
 * JpaSpecificationExecutor<操作的实体类型>，封装了分页、排序等复杂查询
 */
public interface ResumeDao extends JpaRepository<Resume, Long>, JpaSpecificationExecutor<Resume> {

    //jpql查询
    @Query("from Resume where id=?1") //注意此处操作的是实体类和属性，而不是表和字段
    public Resume findByJpql(Long id);

    @Query("from Resume where id=?1 and name=?2")
    public Resume findByJpql2(Long id, String name);

    // 原生SQL查询
    @Query(value = "select * from tb_resume where name like ?1 and address like ?2", nativeQuery = true)
    public List<Resume> findBySql(String name, String address);

    /**
     * 方法命名规则查询
     * 方法名： findBy + 属性名（首字母大写） + 查询方式（模糊查询、等价查询，如果不写，默认等价）
     */
    public List<Resume> findByNameLike(String name);

}



~~~

~~~java
import com.ldx.dao.ResumeDao;
import com.ldx.pojo.Resume;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.PageRequest;
import org.springframework.data.domain.Pageable;
import org.springframework.data.domain.Sort;
import org.springframework.data.jpa.domain.Specification;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

import javax.persistence.criteria.*;
import java.util.List;
import java.util.Optional;

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = {"classpath:applicationContext.xml"})
public class ResumeDaoTest {

    @Autowired
    private ResumeDao resumeDao;

    /**
     * --------针对查询的使用进行分析----
     * 方式一：调用集成的接口中的方法   findById(),findOne()
     * 方式二：引入jpql（jpa查询语言）
     * 方式三：引入原生sql
     * 方式四：在接口自定义方法，使用方法命名规则查询
     * 方式五：动态查询--service传入dao的条件不确定，把service条件封装为specification，传给dao
     *
     */


    @Test
    public void testFindById() {
        Optional<Resume> optional = resumeDao.findById(1L);
        Resume resume = optional.get();
        System.out.println(resume);
    }

    @Test
    public void testFindByJpql() {
        Resume jpql = resumeDao.findByJpql(1L);
        System.out.println(jpql);
    }

    @Test
    public void testFindBySql() {
        List<Resume> list = resumeDao.findBySql("李%", "上海%");
        for (int i = 0; i < list.size(); i++) {
            Resume resume = list.get(i);
            System.out.println(resume);
        }
    }

    // 测试排序
    @Test
    public void testSort() {
        Sort sort = new Sort(Sort.Direction.DESC, "id");
        List<Resume> list = resumeDao.findAll(sort);
        for (int i = 0; i < list.size(); i++) {
            Resume resume = list.get(i);
            System.out.println(resume);
        }
    }

    // 测试分页
    @Test
    public void testPage() {
        Pageable pageable = PageRequest.of(0, 2);
        Page<Resume> all = resumeDao.findAll(pageable);
        System.out.println(all);
    }

    @Test
    public void testMethdoName() {
        List<Resume> list = resumeDao.findByNameLike("李%");
        for (int i = 0; i < list.size(); i++) {
            Resume resume = list.get(i);
            System.out.println(resume);
        }
    }

    // 动态查询，指定单个条件
    @Test
    public void testSpecfication() {

        Specification<Resume> specification = new Specification<Resume>() {
            @Override
            public Predicate toPredicate(Root<Resume> root, CriteriaQuery<?> criteriaQuery, CriteriaBuilder criteriaBuilder) {
                // 获取name属性
                Path<Object> name = root.get("name");

                // 使用CriteriaBuilder针对name属性构建条件
                Predicate predicate = criteriaBuilder.equal(name, "张三");

                return predicate;
            }
        };

        Optional<Resume> optional = resumeDao.findOne(specification);
        Resume resume = optional.get();
        System.out.println(resume);
    }


    // 动态查询，指定多个条件
    @Test
    public void testSpecfications() {

        Specification<Resume> specification = new Specification<Resume>() {
            @Override
            public Predicate toPredicate(Root<Resume> root, CriteriaQuery<?> criteriaQuery, CriteriaBuilder criteriaBuilder) {

                Path<Object> name = root.get("name");
                Path<Object> address = root.get("address");

                // 使用CriteriaBuilder针对name属性构建条件
                Predicate predicate1 = criteriaBuilder.equal(name, "张三");
                Predicate predicate2 = criteriaBuilder.like(address.as(String.class), "北%");

                //组合两个条件
                Predicate and = criteriaBuilder.and(predicate1, predicate2);

                return and;
            }
        };

        Optional<Resume> optional = resumeDao.findOne(specification);
        Resume resume = optional.get();
        System.out.println(resume);
    }


}

~~~




