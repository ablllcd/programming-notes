## JPA 简介
JPA全称JAVA Persistence API，是SUN公司提出的ORM（object relation mapping）规范，旨在：

1. 简化持久层开发，避免繁琐和不同的JDBC和SQL代码
2. 统一持久层开发框架，在各种持久层框架上提供了这么一层规范

所以JPA并不是持久层框架，而是持久层框架的规范（接口），来要求各种框架都满足JPA的要求，从而使得程序员在不修改代码的情况下更换持久层框架。

该规范提供了：

1. ORM映射：JPA提供了XML和注解两种元数据，用于描述表和对象的关系
2. JPA的API：用于操作实体对象，执行CRUD操作
3. JPQL查询语言：提供面向对象（非数据库）的查询语言，避免和SQL的耦合


## Spring DATA JPA 简介
Spring Data JPA不同于JPA，JPA本身只是一个概念，而SPRING DATA JPA则是基于这个概念的实现框架。该框架一个显著的功能就是可以根据规定自动创建方法。

注意：spring-boot-starter-data-jpa依赖默认使用hibernate作为ORM框架，该框架再利用JDBC来访问MYSQL；而Mybatis好像并没有被Spring DATA JPA所支持。


## Quick Start (利用Spring Boot )


