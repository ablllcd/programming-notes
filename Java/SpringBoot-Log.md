## 说明

这里介绍spring boot项目中对logback日志框架的使用，具体关于logback的介绍看javaLog.md文档。

## 添加依赖

spring-boot-starter-web 包含了spring-boot-starter 包含了 spring-boot-starter-logging 包含了 logback依赖。

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

所以一般spring boot项目已经引入依赖了

## 配置

在普通项目中，我们创建logback.xml文件用来声明配置信息。而在spring项目中，在applicaiton.properties文件中声明配置信息来快速配置。也可以用logback.xml进行详细配置。



