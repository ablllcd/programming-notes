## Spring 框架

框架就是一个半成品应用，通过添加自己的代码来实现完整程序。而 Spring Framework就是一个轻量级开源框架。

## Quick Start

1. 创建新项目作为父项目，在父项目下创建module作为子项目
2. 在子项目中添加spring依赖

```
<dependencies>
    // spring 依赖
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>6.1.2</version>
    </dependency>

    // junit 依赖
    <dependency>
        <groupId>org.junit.jupiter</groupId>
        <artifactId>junit-jupiter-api</artifactId>
        <version>5.10.1</version>
        <scope>test</scope>
    </dependency>
</dependencies>
```

3. 在子项目中创建User类

```
package com.example.pojo;

public class User {
    public void hello(){
        System.out.println("hello, here is User.hello() func!");
    }
}
```
4. 在resource下创建bean.xml，将User类添加其中

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="user" class="com.example.pojo.User"></bean>
</beans>
```

5. 在单元测试中创建User实例并且测试

```
@Test
public void userTest(){
    // 加载xml配置文件
    ApplicationContext applicationContext = new ClassPathXmlApplicationContext("bean.xml");

    // 获取bean对象
    User user = (User)applicationContext.getBean("user");

    // 使用对象
    user.hello();
}
```



## IOC 控制反转

IOC (Inverse of control)是spring 框架中在一个重要思想：对象的创建和管理不再由程序员负责，而是由Spring来负责。这有助于程序的松耦合。

Spring中使用IOC容器来负责对象的实例化和初始化，并控制对象和对象之间的依赖关系。由IOC管理的对象被称为bean。

IOC本质上也是一个JAVA类来实现的，只要实现了BeanFactory接口的JAVA类都可以当作IOC。JAVA中IOC接口和实现类为：

![Alt text](pic/SpringIOC-Structure.png)



## 基于 XML 进行 Bean 管理

在xml文件中通过id和class定义bean元素，id可以自定义，class是类的全类名：

```
<bean id="user" class="com.example.pojo.User"></bean>
```

之后将xml交给beanFactory的实现类，由其进行对象管理
```
ApplicationContext applicationContext = new ClassPathXmlApplicationContext("bean.xml");
```

### Bean 获取

1. 根据id获取

```
User user = (User)applicationContext.getBean("user");
```

2. 根据类名获取

```
User user = (User) applicationContext.getBean(PersonInterface.class);
```

该方法常用于通过接口来获取对应的实现类，但如果xml文件中根据该接口找到了两个或更多实现类，则会报错。

### 依赖注入

BeanFactory创建类默认用无参构造，依赖注入允许beanfactory创建对象的时候为其赋值。*（不过写死到配置文件中有什么用？）*

**1. 通过setter**

首先在类中写set方法
```
public void setName(String myname){
    this.name = myname;
}
```

然后在xml中配置property
```
<bean id="user" class="com.example.pojo.User">
    <property name="name" value="cain"></property>
</bean>
```

这里name="name"是指要调用setName方法，"name"对应setName中的Name，value是传入set方法的参数。

这样创建bean时，对象就有值了
```
User user = (User)applicationContext.getBean("user");
System.out.println(user)
```

**2. 通过有参构造器**

首先在类中写有参构造器
```
public User(String myname, int age) {
    this.name = myname;
    this.age = age;
}
```

然后在xml中配置要传递的值
```
<bean id="user" class="com.example.pojo.User">
    <constructor-arg name="myname" value="cain"></constructor-arg>
    <constructor-arg name="age" value="18"></constructor-arg>
</bean>
```

**3. 特殊值处理**

在上述两种方法赋值的过程中，会有一些特殊情况：

* 值为null
```
<constructor-arg name="myname">
    <null></null>
</constructor-arg>
```

* 使用了特殊字符，例如<>，需要用CDATE标签来包含特殊值

```
<constructor-arg name="myname">
    <value><![CDATA[a<b]]></value>
</constructor-arg>
```

**4. 复杂类型赋值**

类中的成员变量也可能为其它类，列表，数组等

* 类对象：使用ref标签引用另一个xml元素

```
<bean id="user" class="com.example.pojo.User">
    <constructor-arg name="name" value="cain"></constructor-arg>
    <constructor-arg name="age" value="18"></constructor-arg>
    <constructor-arg name="card" ref="card"></constructor-arg>
</bean>

<bean id="card" class="com.example.pojo.Card">
    <property name="cname" value="cain's card"></property>
</bean>
```

* 数组：用array标签包围value

```
<bean id="user" class="com.example.pojo.User">
    <constructor-arg name="name" value="cain"></constructor-arg>
    <constructor-arg name="age" value="18"></constructor-arg>
    <constructor-arg name="hobbies">
        <array>
            <value>swim</value>
            <value>dance</value>
        </array>
    </constructor-arg>
</bean>
```

* 列表：类似于数组，将array标签替换为list就行

* Map: 用Map标签和entry标签

```
<bean id="user" class="com.example.pojo.User">
    <constructor-arg name="name" value="cain"></constructor-arg>
    <constructor-arg name="age" value="18"></constructor-arg>
    <constructor-arg name="pair">
        <map>
            <entry key="my son" value="18"></entry>
        </map>
    </constructor-arg>
</bean>
```

* 外部文件：用外部文件中的值来赋值

首先写外部文件，例如jdbc.properties:
```
username=root
password=123456
url=jdbc:mysql://localhost:3306/
driver=com.mysql.cj.jdbc.Driver
```

然后在bean.xml文件中添加一个context scheme
```
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context = "http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context.xsd">
```

指明要用的外部文件和值
```
<context:property-placeholder location="classpath:jdbc.properties"></context:property-placeholder>

<bean id="druidDataSource" class="com.alibaba.druid.pool.DruidDataSource">
    <property name="url" value="${url}"></property>
</bean>
```

**5. bean的构造时机**

xml文件中的bean有scope属性，其有两个值：singleton和prototype

singleton是默认值，表示该类为单实例，在IOC容器初始化时就创建实例

prototype表示多实例，在获取bean时才创建实例（每次调用都会创建一个新的实例）

之所以用单例，是因为没必要每个请求都新建一个对象，这样子既浪费CPU又浪费内存；

之所以用多例，是为了防止并发问题；即一个请求改变了对象的状态，此时对象又处理另一个请求，而之前请求对对象状态的改变导致了对象对另一个请求做了错误的处理；


```
<bean id="user" class="com.example.pojo.User" scope="prototype">
    <constructor-arg name="name" value="cain"></constructor-arg>
    <constructor-arg name="age" value="18"></constructor-arg>
</bean>
```

**6. bean的生命周期**

bean的生命周期为：构造函数->setter->初始化->使用bean->摧毁bean

其中初始化方法和摧毁方法需要在类中声明，以及在xml中指出
```
<bean id="user" class="com.example.pojo.User" scope="prototype" init-method="initMethod" destroy-method="destroyMethod">
        <constructor-arg name="name" value="cain"></constructor-arg>
        <constructor-arg name="age" value="18"></constructor-arg>
</bean>
```

```
public void initMethod(){
    System.out.println("initMethod is called");
}

public void destroyMethod(){
    System.out.println("destroyMethod is called");
}
```

其实在初始化前后还有处理器函数可以设置，这俩函数会作用于xml文件中的所有bean对象，暂且不知道有啥用，所以略过。

**7. 自动注入**

在类中声明成员变量和setter方法后，如果该成员变量是个类，而这个成员变量的类也放在IOC容器中了，那么可以使用autowire属性来让spring自己去找setter方法应该传递的值。

```
public class User{
    private String name;
    private int age;
    private Card card;

    public User(String name, int age) {
        this.name = name;
        this.age = age;
        System.out.println("parameter construct method is called");
    }

    public void setCard(Card card){
        System.out.println("setCard func is called");
        this.card = card;
    }
}
```

```
<bean id="user" class="com.example.pojo.User" autowire="byType">
    <constructor-arg name="name" value="cain"></constructor-arg>
    <constructor-arg name="age" value="18"></constructor-arg>
<!--        <property name="card" ref="card"></property>-->
</bean>

<bean id="card" class="com.example.pojo.Card">
    <property name="cname" value="gold card"></property>
</bean>
```

autowire查找setter需要的参数的方式有两种：byType和byName。

byType就是通过setter参数类型在pom文件中找对应类型的bean，所以该方法要求bean为单例

byName很奇怪，没搞懂

## 基于注解管理bean

注解是java中的特殊标识，它允许在不修改原有代码的情况下添加新的信息。

### 创建bean

首先需要在xml中开启component-scan，并指出要扫描的包（包括其子包）

```
<context:component-scan base-package="com.cain"></context:component-scan>
```

之后对需要添加为bean的类加上注解即可
```
@Component(value="user")
public class User {
    public String name;
}
```
注解中的value对应xml文件中bean元素的id，不设置的默认值为类名小写

注解类型有：@Componet,@Controller,@Service,@Repository，其本质没有区别，不过可以给bean分类，然后根据类别剔除注解，例如：

```
<context:component-scan base-package="com.cain">
    <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
</context:component-scan>
```

@Controller注解的类将不会被注册为bean

### Bean对象的依赖注入

可以使用@Autowired注解实现依赖注入，它等价于xml中的autowire="byType"

@Autowried注解可以用于：成员变量，setter，构造器，形参变量。但本质上都是对被注解的元素进行自动依赖注入

```
@Component
public class User {
    @Autowired
    public Card card;
}
```

如果希望使用autowier="byName"，可以搭配@Qualifier来使用

```
@Component
public class User {
    @Autowired
    @Qualifier(value = "cardID")
    public Card card;
}
```

```
@Component(value = "cardID")
public class Card {
    public void show(){
        System.out.println("I'm a card");
    }
}
```

@Autowired是Spring提供的注解，其实jdk拓展中有个@Resource注解也可以实现相同的功能。根据jdk不同版本，要使用该注解可能需要引用拓展。

@Resource注解默认是byName搜索bean，如果没有搜索到，则使用byType搜索。所以功能比@Autowired稍微强点，但是不多，所以略过。

### 全注解开发

bean的创建和依赖注入都由注解完成，现在xml文件中只是开启了component scan并指明扫描bean的范围。为此可以创建配置类来完全取代xml文件：

```
@Configuration  //声明为配置类
@ComponentScan("com.cain")  //开启组件扫描
public class SpringConfig {
}
```

修改读取xml文件为读取配置类
```
public void userTest(){
    AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(SpringConfig.class);
    User user = applicationContext.getBean("user",User.class);
    user.card.show();
}
```

## IOC原理解析

IOC容器主要依赖反射技术和注解来实现

1. 首先定义注解来标记要注册为bean的类，和要依赖注入的属性

2. 创建beanFactory的实现类，该类的成员变量有Map容器来存储<Class,Object>
   
3. 该类的构造函数需要一个bean检查范围（比如com.cain），根据该检查范围可以获取这个包以及它的子包下的所有类的全类名，根据全类名可以判断各个类是否包含@Bean注解，如果包含@Bean注解，利用反射机制来创建实例，将这个类的<Class,Object>存储到Map容器中。

4. 构造函数检查完所有类后，Map容器已经存储了所有bean了。遍历Map中的object，利用反射机制检查各个类的属性是否包含@Autowired注解，如果有被注解的成员变量，根据该成员变量的类型，在Map中找对应的object进行自动注入

5. 这样bean创建，存储和依赖注入都完成了


## AOP

AOP （Aspect Oriented Programming）是指面向切片编程，它是一种改善面向对象编程的思想。

**其产生思路如下：**

JAVA使用OOB编程，虽然OOB相较于函数式编程有很大进步，但是它本身还是有一些问题的。

例如很多类都需要日志功能，如果每个类都自己写日志功能的话其实是做了很多重复工作：

![Alt text](pic/AOP-OOB1.png)

既然代码重复，自然就想到把这个功能抽取出来作为一个单独的类：

![Alt text](pic/AOP-OOB2.png)

不过每个类要使用日志功能的话，需要引用Logger类，并调用它的函数。这意味这如果我修改了Logger的接口，我需要修改所有引用Logger的所有类。

那能不能进一步降低耦合呢？这就提出了AOP的思想：

![Alt text](pic/AOP-aop.png)

我不在创建类了，而是创建一个Aspect（本质上也是一个特殊类），在Aspect里写日志功能，并且通过Aspect Configuration来指明我要在哪个类里的什么时机调用（例如在Object A的function 1调用后调用Logging Aspect）。 这样我修改Logging Aspect后，对于其它类的代码完全不需要修改。

**AOP思想的技术实现**

有一个设计模式的思想和AOP高度重合，那就是代理技术（具体见java/javaFunc.md）。而java中实现的动态代理类就可以用来实现AOP思想，但是动态代理用起来非常麻烦，所以spring就自己实现了srping AOP框架来封装了动态代理类，从而方便地实现了AOP思想。

### Quick Start

1. 添加依赖

```
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-aop</artifactId>
    <version>6.1.2</version>
</dependency>

<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-aspects</artifactId>
    <version>6.1.2</version>
</dependency>
```

2. 修改xml文件

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context.xsd
       http://www.springframework.org/schema/aop
       http://www.springframework.org/schema/aop/spring-aop.xsd
       http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd">

    // 设置bean扫描范围
    <context:component-scan base-package="com.cain.AOP"></context:component-scan>
    // 开启自动依赖
    <aop:aspectj-autoproxy></aop:aspectj-autoproxy>

</beans>
```

3. 创建基本类

```
@Component
public class Triangle {
    public void show(){
        System.out.println("I'm a triangle");
    }
}
```

4. 创建aspect类

```
@Aspect
@Component
public class LoggerAspect {
    @Before("execution(void show())")
    public void test(){
        System.out.println("Logger aspect is executed");
    }
}
```

5. 测试

```
@Test
public void aopTest(){
    ClassPathXmlApplicationContext applicationContext = new ClassPathXmlApplicationContext("bean.xml");
    Triangle triangle = applicationContext.getBean("triangle", Triangle.class);
    triangle.show();
}
```


### 通知
Aspect 类中的方法被成为通知，通知作用于的方法被称为目标方法。

#### 通知类型
![Alt text](pic/notice.png)

不同通知类型，除了位置不同，能够获取的参数和能做到事情也是不同的：
```
@Aspect
@Component
public class LoggerAspect {
    // JoinPoint 是所有类型都可以获取的参数，它包含了目标函数的相关信息
    @Before("execution(int sum(..))")
    public void beforeTest(JoinPoint joinPoint){
        System.out.println("before is called");
        System.out.println(Arrays.toString(joinPoint.getArgs()));
        System.out.println(joinPoint.getSignature().getName());
    }

    // After 和 Before 几乎相同
    @After("execution(int sum(..))")
    public void afterTest(){
        System.out.println("after is called");
    }

    // AfterReturning 因为是返回后，可以查看返回结果
    // 标签中returning的值和函数中参数的名称必须一致
    @AfterReturning(value = "execution(int sum(..))", returning = "result")
    public void afterReturnTest(Object result){
        System.out.println("AfterReturn is called, result is "+result);
    }

    // AfterThrowing 可以获取异常值
    // 标签中throwing的值和函数中Throwable 变量名称必须一致
    @AfterThrowing(value = "execution(int sum(..))", throwing = "ex")
    public void afterThrowingTest(Throwable ex){
        System.out.println("AfterThrowing is called, exception is "+ex);
    }
    // round 需要自己负责目标函数调用,也需要将目标函数结果返回
    // 目标函数的执行需要ProceedingJoinPoint
    @Around("execution(int sum(..))")
    public Object aroundTest(ProceedingJoinPoint joinPoint) throws Throwable {
        System.out.println("around is called");
        Object res = joinPoint.proceed();
        return res;
    }
}
```

#### 通知执行顺序
如果多个通知作用于相同的目标方法，可以通过@Order()注解来指示通知的执行顺序。


### 切入点表达式
切入点表达式就是“execution(* com.example.itheima.service.*.*(..))”，它声明了去哪找目标方法，被称为切入点表达式。

切入点表达式有多种类型，这里介绍exectuion和@annotation两种。

#### Execution
![Alt text](pic/execution.png)
![Alt text](pic/execution2.png)
````
@Around("execution(* com.example.itheima.service.*.*(..))")
````

其中路径必须到达方法。即`com.example.itheima.service`到达service包；`.*`选择service包下的所有类；`.*.*`选择了service包下所有类的所有方法；(..)表示任意参数类型的方法都被选择。此外，开头的 `*`表示任意返回类型的方法都被选择。

#### 抽取切入点表达式
由于AOP中每个方法都要写切入点表达式，可以用@Pointcut来将它提取出来。

````
@Component
@Aspect
public class TestAspect {
    @Pointcut("execution(* com.example.itheima.controller.*.*(..))")
    public void pt(){}

    @Before("pt()")
    public void Before(){
        System.out.println("Before ...");
    }

    @After("pt()")
    public void After(){
        System.out.println("After ...");
    }
}
````

#### @Annotation
@Annotation是基于注解定位切入点的。

1. 自定义注解
2. 对目标方法添加自定义注解
3. 在AOP类的方法上，添加@Annotation(自定义注解的全类名)作为切入点表达式。

### AOP原理

Spring的AOP实现原理其实很简单，就是通过动态代理实现的。如果我们为Spring的某个bean配置了切面，那么Spring在创建这个bean的时候，实际上创建的是这个bean的一个`代理对象`，我们后续对bean中方法的调用，实际上调用的是代理类重写的代理方法。**（注意：只有用AOP的时候IOC容器里存储的才是代理对象，如果没有Aspect类，则存储的是原本的类）**

不过代理对象的生成方式有两种：JDK代理对象和CGLIB代理对象。Spring会根据不同的情况使用不同的代理方法。例如JDK动态代理是通过目标对象继承的接口来创建代理类的，如果没有bean接口就无法用该方法，只能转CGLIB。

以下是一个测试代理的方式：
```
public void testProxy() {
    ApplicationContext context =
        new AnnotationConfigApplicationContext(AOPConfig.class);
	// 注意，这里只能通过Human.class获取，而无法通过Student.class
    // 因为在Spirng容器中使用JDK动态代理，Ioc容器中，存储的是一个类型为Human的代理对象
    Human human =  context.getBean(Human.class);
    human.display();
    // 输出代理类的父类，以此判断是JDK还是CGLib
    // 如果输出是Proxy说明是JDK（因为JDK方式的代理类是Proxy的子类）
    System.out.println(human.getClass().getSuperclass());
}
```

上述代码判断代理方法的方式很简单，查看代理对象的父类就行，因为JDK中代理类继承的是Proxy方法，而CGLIB中继承的目标类。


### 基于XML的AOP

在上述过程中，都是利用@Aspect和@Before等注解来声明aspect和通知，除了注解声明，也可以在xml文件中声明。知道有就行，需要用在查。

## Spring 整合 Junit

在上述代码中，每次获取bean都需要调用ClassPathXmlApplicationContext类，十分麻烦。Spring提供依赖来整合Junit，从而可以方便地获取bean。

1. 引入依赖

```
// Spring 整合 Junit 依赖
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-test</artifactId>
    <version>6.1.2</version>
</dependency>

// Junit 本身依赖
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter-api</artifactId>
    <version>5.10.1</version>
    <scope>test</scope>
</dependency>
```

2. 创建单元测试，并且使用bean

```
@SpringJUnitConfig(locations = "classpath:bean.xml")
public class AOPTest {
    @Autowired
    private Triangle triangle;

    @Test
    public void aopTest(){
        int res = triangle.sum(1,2,3);
        System.out.println("res is "+res);
    }
}
```

其中用@SpringJUnitConfig指明xml文件位置，并且用@Autowired来创建bean。

