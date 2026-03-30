
本文档记录spring框架中和测试相关的内容。

# 依赖导入

## MAVEN

### SpringBoot项目中导入依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
```

只需要导入`spring-boot-starter-test`依赖就可以了，因为它已经包含了JUnit、Mockito等常用的测试框架。

## 文件所处位置

测试类应该放在`src/test/java`目录下，与主代码（`src/main/java`）保持相同的包结构。例如，如果你的主代码在`com.example.demo`包下，那么测试类也应该在`com.example.demo`包下。

# 理论知识

## Spring-Test和JUnit的关系

* spring-test：负责把 Spring 容器能力带进测试里（例如：让你在测试类里 @Autowired Bean；支持事务回滚；提供 MockMvc 测 MVC）
* JUnit：负责定义/执行测试（@Test、断言、生命周期等）

# 基本用法