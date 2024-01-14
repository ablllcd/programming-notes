## Lombok

Lombok是一个代码生成器，使用注解帮助生成getter setter等。

### Maven 依赖
```
<!-- https://mvnrepository.com/artifact/org.projectlombok/lombok -->
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>1.18.30</version>
    <scope>provided</scope>
</dependency>

```

## XML配置文件 转 配置类

见：https://blog.csdn.net/zgq15228328671/article/details/102593103

例子：

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context.xsd
       http://www.springframework.org/schema/tx
       http://www.springframework.org/schema/tx/spring-tx.xsd
       http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd">

    <context:property-placeholder location="classpath:druid.properties"></context:property-placeholder>
    <context:component-scan base-package="com.cain"></context:component-scan>

    <tx:annotation-driven transaction-manager="dataSourceTransactionManager"></tx:annotation-driven>

    <bean id="druidDataSource" class="com.alibaba.druid.pool.DruidDataSource">
        <property name="driverClassName" value="${driver}"></property>
        <property name="url" value="${url}"></property>
        <property name="username" value="${uname}"></property>
        <property name="password" value="${password}"></property>
    </bean>

    <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
        <property name="dataSource" ref="druidDataSource"></property>
    </bean>

    <bean class="org.springframework.jdbc.datasource.DataSourceTransactionManager" id="dataSourceTransactionManager">
        <property name="dataSource" ref="druidDataSource"></property>
    </bean>
</beans>
``` 

配置类
```
@Configuration
@ComponentScan("com.cain")
@EnableTransactionManagement
public class BeanConfig {
    @Bean
    DruidDataSource druidDataSource(){
        DruidDataSource dataSource = new DruidDataSource();
        dataSource.setUrl("jdbc:mysql://localhost:3306/worker");
        dataSource.setUsername("root");
        dataSource.setPassword("123456");
        dataSource.setDriverClassName("com.mysql.cj.jdbc.Driver");

        return dataSource;
    }

    @Bean
    JdbcTemplate jdbcTemplate(DruidDataSource dataSource){
        JdbcTemplate jdbcTemplate = new JdbcTemplate(dataSource);
        return jdbcTemplate;
    }

    @Bean
    DataSourceTransactionManager dataSourceTransactionManager(DruidDataSource dataSource){
        DataSourceTransactionManager dataSourceTransactionManager = new DataSourceTransactionManager(dataSource);
        return  dataSourceTransactionManager;
    }
}
```

注意JUnite也要改
```
@SpringJUnitConfig(BeanConfig.class)
```

**与MVC相关的配置**

```
@Configuration
// 开启mvc注解
@EnableWebMvc
// WebMvcConfigurer类中有很多MVC相关的注解
public class DispatcherServletConfig implements WebMvcConfigurer {
    @Override
    // 配置视图控制器
    public void addViewControllers(ViewControllerRegistry registry) {
        registry.addViewController("/index").setViewName("index");
    }
}
```

## Thymeleaf

Thymeleaf是一个Java template engine，用来处理html，css等模板，常用于SpingMVC中的视图层。本质上就是一个java包，将java model中的数据填充到html模板中，构建成html视图。

java model:
```
@Data
public class User {
    private String username;
    private String password;
}
```

html template 就是含有thymeleaf表达式的html文件：
```
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<p th:text="${username}"></p>
<p th:text="${password}"></p>
</body>
</html>
```

### 整合到Spring MVC

1. 添加依赖

```
<dependency>
    <groupId>org.thymeleaf</groupId>
    <artifactId>thymeleaf-spring6</artifactId>
    <version>3.1.2.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.thymeleaf</groupId>
    <artifactId>thymeleaf</artifactId>
    <version>3.1.2.RELEASE</version>
</dependency>
```

2. 注册为bean

```
<!-- 视图解析器-->
<bean id="viewResolver" class="org.thymeleaf.spring6.view.ThymeleafViewResolver">
    <property name="templateEngine" ref="templateEngine" />
    <property name="characterEncoding" value="UTF-8"/>
</bean>

<bean id="templateEngine" class="org.thymeleaf.spring6.SpringTemplateEngine">
    <property name="templateResolver" ref="templateResolver" />
</bean>

<!-- thymeleaf 模板解析器 -->
<bean id="templateResolver" class="org.thymeleaf.spring6.templateresolver.SpringResourceTemplateResolver">
    <property name="prefix" value="/WEB-INF/templates" />
    <property name="suffix" value=".html" />
    <property name="templateMode" value="HTML" />
    <property name="characterEncoding" value="UTF-8"/>
</bean>
```

3. 根据prefix设置的路径添加html模板

```
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
success!
</body>
</html>
```

4. 在@Controller中返回的字符串会被传给模板解析器解析

```
@Controller
public class HelloController {
    @RequestMapping("/success")
    public String success(){
        return "success";
    }
}
```

## REST/RESTFul

https://www.zhihu.com/question/28557115

1. REST描述的是在网络中client和server的一种交互形式；REST本身不实用，实用的是如何设计 RESTful API（REST风格的网络接口）
2. RESTFul: 看Url就知道要什么；看http method就知道干什么；看http status code就知道结果如何。

### Spring MVC对浏览器的RESTFul支持

Spring MVC 提供了HiddenHttpMethodFilter，它允许将浏览器的POST请求转为DELETE或者PUT，（这是因为浏览器一般只支持post和get方法）,只需要在post请求参数中添加_method参数就行

```
<form th:action="@{/users}" method="post">
<input type="hidden" name="_method" value="delete">
    <input type="submit" value="submit">
</form>
```

web.xml中注册filter
```
<filter>
    <filter-name>MethodTransfer</filter-name>
    <filter-class>org.springframework.web.filter.HiddenHttpMethodFilter</filter-class>
</filter>
<filter-mapping>
    <filter-name>MethodTransfer</filter-name>
    <servlet-name>DispatcherServlet</servlet-name>
</filter-mapping>
```