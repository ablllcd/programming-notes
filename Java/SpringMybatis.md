## 简介

Mybatis 本身只是一个持久层框架，它不是Spring家族的组件，是独立存在的，只是可以被Spring整合。

Mybatis本身是对JDBC的封装，使得一些操作更加简单。
<br>原文是"MyBatis 是一款优秀的持久层框架，它支持自定义 SQL、存储过程以及高级映射。MyBatis 免除了几乎所有的 JDBC 代码以及设置参数和获取结果集的工作。MyBatis 可以通过简单的 XML 或注解来配置和映射原始类型、接口和 Java POJO（Plain Old Java Objects，普通老式 Java 对象）为数据库中的记录。"

## Quick Start

**0. 添加依赖**

```
<dependencies>
    <dependency>
        <groupId>org.mybatis</groupId>
        <artifactId>mybatis</artifactId>
        <version>3.5.15</version>
    </dependency>
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <version>1.18.30</version>
        <scope>provided</scope>
    </dependency>
    <!-- https://mvnrepository.com/artifact/com.mysql/mysql-connector-j -->
    <dependency>
        <groupId>com.mysql</groupId>
        <artifactId>mysql-connector-j</artifactId>
        <version>8.2.0</version>
    </dependency>
    <!-- https://mvnrepository.com/artifact/junit/junit -->
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.13.2</version>
        <scope>test</scope>
    </dependency>
</dependencies>
```

**1. 创建mybatis 配置文件**

resources/mybatis-config.xml
```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">

<configuration>
    // 使用mysql的配置文件信息
    <properties resource="mySQL.properties"></properties>

    // 配置数据库连接信息
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="${mysql.driver}"/>
                <property name="url" value="${mysql.url}"/>
                <property name="username" value="${mysql.username}"/>
                <property name="password" value="${mysql.password}"/>
            </dataSource>
        </environment>
    </environments>

    // 指明ORM映射文件
    <mappers>
        <mapper resource="mappers/EmployeeMapper.xml"/>
    </mappers>
</configuration>
```

resources/mySQL.properties
```
mysql.driver=com.mysql.cj.jdbc.Driver
mysql.url=jdbc:mysql://localhost:3306/itheima
mysql.username=root
mysql.password=123456
```

**2. 创建Mapper接口**

```
public interface EmployeeMapper {
    public List<Employee> getAllUsers();
}
```

**3. 描述接口行为**

注意：配置文件中的namespace用来指明要映射的Mapper类；id用来自指明Mapper类中对应的方法

resources/mappers/EmployeeMapper.xml
```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "https://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="org.example.mapper.EmployeeMapper">
    <select id="getAllUsers" resultType="org.example.pojo.Employee">
        select * from employee
    </select>
</mapper>
```

**4. 创建pojo对象**

```
@Data
public class Employee {
    int uid;
    String name;
    int age;
    int gender;
    String position;
}
```

**5. 创建测试类**

```
public class EmployeeTest {
    private SqlSession sqlSession;

    @Before
    public void creatSQLSession() throws IOException {
        String resource = "mybatis-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        sqlSession = sqlSessionFactory.openSession();
        System.out.println("sql session is created");
    }
    @Test
    public void startupTest() {
        EmployeeMapper employeeMapper = sqlSession.getMapper(EmployeeMapper.class);
        System.out.println(employeeMapper.getAllUsers());
    }
}
```

## 核心思想

如果使用原始的JDBC，我们需要手动进行：驱动注册，数据库连接，sql语句创建，结果封装，资源释放。其中除了sql语句和结果封装需要用户指明外，其它的东西都是一样的，

所以Mybatis封装了JDBC，通过配置信息来完成其它操作；对于sql和查询结果，它使用了interface加xml配置类来完成。我们指明DAO层的mapper接口（方便和java其它层协作），并且在xml中指明接口中方法的行为（写sql语句并指明封装类型），之后mybatis就可以创建这个接口的实现类。

下边详细讲解数据访问核心就三部分：**配置连接信息，构建SQL语句，和封装查询结果**

## 配置文件 （配置连接信息）

### Mybatis配置文件--Environment
```
<configuration>
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="${mysql.driver}"/>
                <property name="url" value="${mysql.url}"/>
                <property name="username" value="${mysql.username}"/>
                <property name="password" value="${mysql.password}"/>
            </dataSource>
        </environment>
    </environments>
</configuration>
```

Environment是用来指明数据库连接的，每个environment都可以指定一个数据库连接，通过default来选择使用哪个数据库。此外，它还有一些属性：

* transactionManager: 指明事务管理类型
* dataSource type: 是否使用连接池

### Mybatis配置文件--Mappers

```
<configuration>
    <mappers>
        <mapper resource="mappers/EmployeeMapper.xml"/>
        <package name="org.example.mapper"/>
    </mappers>
</configuration>
```

* Mapper标签用于指定单个映射文件，resource值是以项目中的resources目录为根目录，得出xml配置文件的相对目录。
* Package是用于指定多个映射文件，它有两个要求：
    * xml文件和接口处于同一个包下 （xml文件需要创建在resources/org/example/mapper文件夹下）
    * xml文件的名称和接口名称一致 


## 参数传递（构建SQL语句）

上述代码是将SQL语句直接写死了：
```
public List<Employee> getAllUsers();
```
```
<select id="getAllUsers" resultType="org.example.pojo.Employee">
    select * from employee
</select>
```

但实际场景中需要动态构建SQL：

* ${}：用于字符串拼接 
* #{}：用于占位符

**单参数传递**

如果是单参数，{}中的内容写什么都行

**多参数传递**

接口中方法的参数被mybatis存储在mapper中，**默认键值对**为arg0,arg1,param1,param2。

```
public Employee getByNameAndAge(String name,int age);
```

```
<select id="getByNameAndAge" resultType="org.example.pojo.Employee">
    select * from employee where name=#{arg0} and age=#{arg1}
</select>
```

此外也可以构建**mapper**，传递mapper：
```
public Employee getByNameAndAge(Map map);
```

```
<select id="getByNameAndAge" resultType="org.example.pojo.Employee">
    select * from employee where name=#{name} and age=#{age}
</select>
```

```
public void paraTest(){
    EmployeeMapper employeeMapper = sqlSession.getMapper(EmployeeMapper.class);
    HashMap<String, Object> map = new HashMap<>();
    map.put("name","test");
    map.put("age",18);
    System.out.println(employeeMapper.getByNameAndAge(map));
}
```

`（推荐）`不过最推荐的还是通过**注解**来指定键值对的键值：

```
public Employee getByNameAndAge(@Param("name") String name,
                                @Param("age") int age);
```

```
<select id="getByNameAndAge" resultType="org.example.pojo.Employee">
    select * from employee where name=#{name} and age=#{age}
</select>
```

**实体类参数传递**

接口中还可以传递实体类，mybatis会根据{}中的属性名和反射技术去获取值：

```
public Employee getByNameAndAge(Employee employee);
```

```
<select id="getByNameAndAge" resultType="org.example.pojo.Employee">
    select * from employee where name=#{name} and age=#{age}
</select>
```

## 查询结果封装

**查询结果为一行数据**

1. 用POJO类来接受，这种方法要求列名和类中的成员变量名称一致

```
public Employee getByID(int ID);
```

```
<select id="getByID" resultType="org.example.pojo.Employee">
    select * from employee where uid = #{s}
</select>
```

2. 用Map来接收

```
public Map<String,Object> getByID(int ID);
```

```
<select id="getByID" resultType="map">
    select * from employee where uid = #{s}
</select>
```

**查询结果为多行数据**

1. 用POJO列表来接收
```
public List<Employee> getAllUsers();
```

```
<select id="getAllUsers" resultType="org.example.pojo.Employee">
    select * from employee
</select>
```

2. 类似的，也可以用Map列表

3. 用嵌套Map来接收，只是要指明用来构建嵌套Map的属性

```
@MapKey("uid")
public Map<String,Object> getAllUsers();
```

```
<select id="getAllUsers" resultType="map">
    select * from employee
</select>
```

