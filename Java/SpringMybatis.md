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

    // 指明Mapper实现文件
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

接口中方法的参数被mybatis存储在map中，**默认键值对**为arg0,arg1,param1,param2。

```
public Employee getByNameAndAge(String name,int age);
```

```
<select id="getByNameAndAge" resultType="org.example.pojo.Employee">
    select * from employee where name=#{arg0} and age=#{arg1}
</select>
```

此外也可以构建**map**，传递map：
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

## 查询结果映射

### table字段名和成员变量不一致
1. 数据库查询时使用别名


    可以在sql中的select语句中利用as来进行名称转换，例如：
    ```
    <select id="getByID" resultType="org.example.pojo.Employee">
        select uid as id,name,age,gender,position from employee where uid = #{id}
    </select>
    ```

2. 下划线命名法转驼峰命名法

    在mybatis配置文件中添加setting
    ```
    <settings>
        <setting name="mapUnderscoreToCamelCase" value="true"/>
    </settings>
    ```

3. 使用resultMap

    在映射文件中使用resultMap标签，先从select映射到resultMap，在映射到实体类。（resultType流程好像是一致的，只是使用了默认的resultMap）
    ```
    <resMapultMap id="empResultMap" type="org.example.pojo.Employee">
        <id property="id" column="uid"/>
        <result property="name" column="name"/>
    </resultMap>

    <select id="getAllUsers" resultMap="empResultMap">
        select * from employee
    </select>
    ```

### 多对一映射

在数据库中经成存在多对一，一对多的情况，比如employee里记录department的id。但是在java中，employee类的department不是一个int类型的ID，而是一个department的引用。为此需要特殊的映射关系：

1. 级联属性赋值
   
    ```
    <resultMap id="empResultMapper" type="org.example.pojo.Employee">
        <id property="id" column="uid"/>
        <result property="department.name" column="departName"/>
        <result property="department.id" column="dept_id"/>
    </resultMap>

    <select id="getAllEmps" resultMap="empResultMapper">
        select employee.*,dept.name as departName,dept.dept_id
        from employee,dept,dept_emp
        where employee.uid=dept_emp.uid and dept_emp.dept_id=dept.dept_id;
    </select>
    ```

    其中使用JOIN来合并表信息，在resultMap中使用department.name的方式进行赋值。不过需要注意：如果两个员工处于同一个部门，每个员工都会创建一个新的部门，造成内存浪费。

2. association标签来配置嵌套关系：

    ```
    <resultMap id="empResultMapper" type="org.example.pojo.Employee">
        <id property="id" column="uid"/>
        <association property="department" javaType="org.example.pojo.Department">
            <id property="id" column="dept_id"/>
            <result property="name" column="departName"/>
        </association>
    </resultMap>

    <select id="getAllEmps" resultMap="empResultMapper">
        select employee.*,dept.name as departName,dept.dept_id
        from employee,dept,dept_emp
        where employee.uid=dept_emp.uid and dept_emp.dept_id=dept.dept_id;
    </select>
    ```
3. 分步查询

    ```
    <resultMap id="empResultMapper" type="org.example.pojo.Employee">
        <id property="id" column="uid"/>
        <association property="department"
                     select="org.example.mapper.DeptMapper.getDeptByID"
                     column="dept_id">
        </association>
    </resultMap>

    <select id="getAllEmps" resultMap="empResultMapper">
        select * from employee,dept_emp where employee.uid=dept_emp.uid
    </select>
    ```
    resultMap对于查询到的每一行，除了根据名称赋值外，还将dept_id作为参数调用org.example.mapper.DeptMapper.getDeptByID方法，其返回的Dept类型直接赋值给employee的dept。

    分布查询的主要用途为`延迟加载`，例如开启延迟加载后，调用getAllEmps获取的Emp列表并没有Dept对象，只有真的用到的时候才会执行association里的sql语句（也就是分布查询才允许延迟加载这种优化）。

    ```
    <association property="department"
                     select="org.example.mapper.DeptMapper.getDeptByID"
                     column="dept_id"
                     fetchType="lazy">
    </association>
    ```

### 一对多映射

一对多映射就是department类中有一个列表来存储Employee。

1. 使用collection标签

    ```
    <resultMap id="deptResultMap" type="org.example.pojo.Department">
        <id property="id" column="dept_id"/>
        <result property="name" column="dname"/>
        <collection property="employees" ofType="org.example.pojo.Employee">
            <id property="id" column="uid"/>
            <result property="name" column="name"/>
            <result property="age" column="age"/>
            <result property="gender" column="gender"/>
            <result property="position" column="position"/>
        </collection>
    </resultMap>
    <select id="getDeptAndEmpByID" resultMap="deptResultMap">
        select employee.*, dept.name as dname, dept.dept_id
        from dept,dept_emp,employee
        where dept.dept_id=dept_emp.dept_id
        and employee.uid=dept_emp.uid
        and dept.dept_id = #{id}
    </select>
    ```
    这是一次将结果都查出来，然后在resultMap中一一赋值。其中列表对应collection标签。

2. 分步查询

    ```
    <resultMap id="deptResultMap" type="org.example.pojo.Department">
        <id property="id" column="dept_id"/>
        <result property="name" column="name"/>
        <collection property="employees"
                    select="org.example.mapper.EmployeeMapper.getEmpsByDept"
                    column="dept_id">
        </collection>
    </resultMap>
    <select id="getDeptAndEmpByID" resultMap="deptResultMap">
        select * from dept where dept_id=#{id}
    </select>
    ```

## 额外辅助功能

### 1. 返回自动递增的主键

```
public int insertUser(Employee employee);
```

注意useGeneratedKeys表明要返回自动生成的key，keyProperty是指返回值要传给哪个参数。
```
<insert id="insertUser" useGeneratedKeys="true" keyProperty="uid">
    insert into employee values (null,#{name},#{age},#{gender},#{position})
</insert>
```

```
@Test
public void insertUser(){
    EmployeeMapper employeeMapper = sqlSession.getMapper(EmployeeMapper.class);
    Employee employee = new Employee(-1, "cain", 23, 1, "worker");
    employeeMapper.insertUser(employee);
    sqlSession.commit();
    System.out.println(employee);
}
```

### 2. 事务功能

配置文件中的事务管理类型为JDBC，默认需要手动提交事务：

```
employeeMapper.insertUser(employee);
sqlSession.commit();
```

如果希望每个操作都自动提交，创建sqlSession时可以设置：

```
sqlSession = sqlSessionFactory.openSession(true);
```

# Spring Boot 配置Mybatis

## Quick Start
1. 创建SpringBoot项目并添加依赖

    ```
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <dependency>
        <groupId>org.mybatis.spring.boot</groupId>
        <artifactId>mybatis-spring-boot-starter</artifactId>
        <version>2.3.2</version>
    </dependency>
    <dependency>
        <groupId>com.mysql</groupId>
        <artifactId>mysql-connector-j</artifactId>
        <version>8.2.0</version>
    </dependency>
    ```

    这里有两个需要注意的：（1）要选择对应jdk版本的springboot版本 （2）mybatis没有被springboot管理版本，使用时要注意springboot和mybatis版本是否一致

2. 在application.properties中填写数据库配置信息：
    ```
    spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
    spring.datasource.url=jdbc:mysql://localhost:3306/db01
    spring.datasource.username=root
    spring.datasource.password=123456
    ```

3. 创建实体类来接收SQL查询结果

    ```
    @Data
    @Builder
    @AllArgsConstructor
    @NoArgsConstructor
    public class User implements UserDetails {
        private Integer id;
        private String firstname;
        private String lastname;
        private String password;
    }
    ```

4. 创建Mapper接口
    ```
    @Mapper
    public interface UserMapper {
        @Select("select * from user")
        public List<User> list();
    }
    ```
5. 在applicaiton类上添加注解开启组件扫描
    ```
    @SpringBootApplication
    @MapperScan("com.cain.mapper")
    public class SecurityApplication {
        public static void main(String[] args) {
            SpringApplication.run(SecurityApplication.class,args);
        }
    }
    ```

## 基本操作

### Mapper注解

被@Mapper注解标记的接口会被自动实现

todo:为什么

### 增加
````
@Insert("insert into user( name, gender) values " +
        "( #{name}, #{gender})")
public int insert(User user);
````
如果sql语句错误，例如主键重复，程序也会报错。

#{name}和#{gender}是User的成员变量，可以直接写，不需要get方法。

如果有返回值，就是数据库被影响的条数。

### 增加-主键返回
有时在插入后，我们需要返回主键，则需要添加注释@potions
````
@Options(keyProperty = "id", useGeneratedKeys = true)
@Insert("insert into user( name, gender) values " +
        "( #{name}, #{gender})")
public int insert(User user);
````

````
@Test
public void insertTest(){
    User u = new User(1, "user", "g");
    int num = userMapper.insert(u);
    System.out.println("id is "+u.getId());
}
````
这样新增条目的主键会被赋值到User的id变量中。

### 删除
````
@Delete("delete from user where id=#{id}")
public void delete(int id);
````

### 修改
````
@Update("update user set name = #{newname} where name = 'newname'")
public void update(String newName);
````

### 查找
````
@Select("select * from user")
public List<User> list();
````
User的成员变量需要和查询到的数据列数和名称一一对应，User也需要设置get/set方法。这样查询到的数据会自动封装为User，并且封装到List中。


## 数据库预编译和拼接
### 预编译
预编译的语句是 #{变量}

    @Mapper
    public interface EmpMapper {
       @Delete("delete from employee where uid=#{id}")
       public int delete(Integer id);
    }

输出的日志为
````
==>  Preparing: delete from employee where uid=?
==> Parameters: 1(Integer)
<==    Updates: 0
````

其优势为：`性能高`，没有`sql注入问题`

### 拼接
直接进行变量和sql语句的字符串拼接：${变量}
````
@Mapper
public interface EmpMapper {
    @Delete("delete from employee where uid=${id}")
    public int delete(Integer id);
}
````
输出日志为
````
==> Preparing: delete from employee where uid=1
==> Parameters: 
<== Updates: 0
````
有`sql注入问题`，使用场景不多。

上述两个操作的返回值是：该sql操作影响的数据数目。


### XML 文件
除了直接用标注外，还可以使用XML配置sql语句。具体请看：https://www.bilibili.com/video/BV1m84y1w7Tb?p=130

### 动态sql
对于sql语句，除了用参数替换值外，还可以通过if，for，等标签来动态构造，具体怎么做等用到在查。

## 数据库连接池
执行sql语句前，程序需要先和数据库建立连接。在框架中这步骤被隐藏了，但是在jdbc中可以看到详细的过程。为了优化数据库连接的创建和释放，提出了数据库连接池的概念。

1. 数据库连接池是一个容器，负责分配和管理`连接(conncection)`。
2. 它允许程序重复使用现有的连接，而不是每次使用都要创建和释放。
3. 如果用户占有连接但没有使用，超过一定时间，连接池会自动将其收回。

连接池接口：DataSource

连接池产品：C2P0，DBCP等，springboot默认的是Hikari。

指定连接池：在pom中添加连接池的dependency，即自动切换为该连接池。例如：
````
<!-- 数据库连接池 -->
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid-spring-boot-starter</artifactId>
    <version>1.2.20</version>
</dependency>
````
会将连接池切换为阿里的Druid。