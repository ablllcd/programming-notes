## 介绍

Mybatis Plus是在Mybatis的基础上进一步封装，允许在不修改现有代码的情况下提高开发效率，非常丝滑。

## Quick Start

1. 导入依赖：只需要将Mybatis原有项目中Mybatis的依赖替换为Mybatis Plus即可，Mybatis Plus包含了Mybatis

```
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-boot-starter</artifactId>
    <version>3.5.5</version>
</dependency>
```

2. Mapper继承BaseMapper类

```
@Mapper
public interface UserMapper extends BaseMapper<User> {
//    @Select("select * from user")
//    public List<User> list();

}
```

这里只需要指明泛型即可，单表查询的基本操作已经全部在BaseMapper里实现了。
