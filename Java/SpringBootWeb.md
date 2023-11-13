## Spring Boot Quick Start

1. 创建springboot工程并添加web依赖
2. 创建请求处理类

    ````
    package com.cain.italias.controller;

    import org.springframework.web.bind.annotation.RequestMapping;
    import org.springframework.web.bind.annotation.RestController;

    @RestController
    public class DeptController {
        @RequestMapping("/hello")
        public String hello(){
            System.out.println("Hello world");
            return "return of hello funciton";
        }
    }
    ````
3. 启动app,可根据端口号进行访问

*Note:创建SpringBoot工程是联网下载的* 

## Web服务器
想要别人能够通过网络访问到自己的资源，需要所谓的web服务器来负责处理协议以及实现部署。

而在框架中，我们不是将资源交给服务器，单纯依赖服务器来实现所有功能，而是需要和服务器进行交互来实现一些业务逻辑。为此我们希望框架中的web服务器可以：

1. 封装HTTP协议，便于web开发
2. 部署web项目，使其能够通过浏览器访问

在上述springBoot的web依赖中已经包含了`Tomcat` Web服务器了。

### Tomcat服务器
Tomcat支持Servlet规范（java servlet感觉是java用于web的类，之后需深入了解），可以运行Servlet程序，因此也被成为Servlet容器。

## Tomcat 在Spring boot中的应用
![Alt text](pic/Springboot-Tomcat.png)

首先，Springboot底层提供了DispatcherServlet类，这个类实现了Servlet接口，也就符合Servlet规范。所以它可以被Tomcat识别，也可以用它来使用Tomcat。

浏览器的请求由Tomcat接收，然后它将其封装为Servlet，Servlet再被处理为Controller。响应也是同理，先在Controller中指明要返回的值，然后（我猜是springboot）将Controller转为Servlet，Tomcat将Servlet发送给客户端。<br>
（浏览器的请求和响应分别被Tomcat封装为HttpServletRequest和HttpServletResponse类，这个要等学了Servlet才能搞懂，暂且不管）

我们通过写Controller类便可以实现浏览器的请求和响应。

## 请求响应操作
就如同quickstart中那样，创建请求处理类即可。所谓的请求处理类就是加了`@RestController`的类，类中用`@RequestMapping(/path)`注解来实现对应URL的请求处理方法。

### 注解：
`@RequestMapping(/path)`用来修饰方法，`path`指的是URL。它将HTTP请求映射到被注解的方法上<br>
`@RestController`其实包括了`@ResponseBody`和`@Controller`。<br>
`@ResponseBody`可以作用于类或者方法，它会将方法的返回值封装为浏览器的响应。<br>
`@Controller`用于类，指明该类为Controller组件。

#### RequestMapping指定请求方法
````
@RequestMapping(value = "/depts",method = RequestMethod.GET)
````

为了简写可以用如下方法：
````
@GetMapping("depts")

//类似的
@PostMapping("depts")
@PutMapping("depts")
````

### 处理不同请求类型

#### 传统模式：
````
@RequestMapping("/trad")
public String traditional(HttpServletRequest request){
    String name = request.getParameter("name");
    System.out.println(name);
    return "paramater name: "+name;
}
````

这里传递参数HttpServletRequest，然后从中提起想要的参数并且进行类型转换，处理完数据后，利用return将数据返回给浏览器。

#### SpringBoot模式：

方法的参数名需要和浏览器请求用的get/post方法中的参数名相同。也是利用return将数据返还给浏览器。

#### 1. 简单参数
通常用于get方法，参数在url中
````
@RequestMapping("/sim")
public String simplePara(String name){
    System.out.println(name);
    return "paramater name: "+name;
}
````


#### 2. 路径参数
有时参数会通过路径传递
````
@DeleteMapping("depts/{id}")
public Result deleteMethodDepts(@PathVariable Integer id){
    return deptService.deptDelete(id);
}
````

#### 3. Json参数
json数据在http请求体中，需要加@RequestBody，json内容会自动和标注的变量匹配。

注意： @RequestBody通常用于json或者xml数据，它并非适用所有类型的参数体，例如http body 为text或者form-data类型时无法匹配。它要求Content-Type为application/json或者application/xml类型。
````
@PostMapping("depts")
public Result postMethodDepts(@RequestBody Department dept){
    return deptService.deptAdd(dept);
}
````

#### 4. 文件
上传文件要求前端三要素：input type = file; method=post; enctype="multipart/form-data"。
````
<form action="/upload" method="post" enctype="multipart/form-data">
    name: <input type="text" name="username"> <br>
    file: <input type="file" name="file"><br>
    <input type="submit" value="submit">
</form>
````

文件类型储存到MultipartFile类型中，注意变量名和input name要相同
````
@PostMapping("/upload")
public void upload(MultipartFile file) {
    System.out.println(file.getOriginalFilename());
}
````
MultipartFile封装了一些方法，有获取文件名，获取文件内容，存储文件等功能

## 分层解耦
以请求响应为例，我们可以将全部的代码都写在@RequsetMapping修饰的方法中，但这不利于维护，也不符合`单一职责原则`。

例子：
````
@RestController
public class EmployeeController {
    @RequestMapping("/EmpList")
    public List<Employee>getEmpList(){
        // 1. 获取数据
        // 数据可能来自文件或数据库，方便起见我直接在程序中创建数据来模拟读取过程
        // Employee 有俩属性：String name; String gender
        // 数据中用0，1代表性别
        Employee e1 = new Employee("cain", "1");
        Employee e2 = new Employee("gala", "0");
        List<Employee> ls = new ArrayList<>();
        ls.add(e1); ls.add(e2);

        // 2. 对数据进行处理
        ls.forEach(e ->{
            if(e.getGender() == "0")
                e.setGender("Female");
            else
                e.setGender("Male");
        });

        // 3. 响应请求
        return ls;
    }
}
````

### 三层架构
`Controller`: 控制层，负责接受请求和响应数据。<br>
`Service`:业务逻辑层，处理具体的业务逻辑<br>
`Dao(Data Access Object)`:数据访问层，负责数据库的操作。

对于Service和Dao层，通常创建一个接口和一个实现包，结构如下：

![Alt text](pic/3-layer-sturcture.png)

项目代码见：D:\Programming\JavaStudy\springBoot\quickstart

### 解耦
耦合：层与层之间或者模块与模块之间的依赖程度。

在三层框架中，Controller层需要创建Service层类，Service层需要创建Dao层类，所有它们之间的耦合度很高，当底层的类发生改变时，上层的代码也需要改变，这不好维护。为此，springboot提出了一些新概念。

### 控制反转 Inversion of Contorl
对象的创建控制权由程序自身转移到外部（容器）。<br>容器负责创建和管理对象。

对于要交给IOC容器的类，只需要为其加上注解@Component

````
@Component
public class EmpServiceImp implements EmpService {
    // 使用DI来注入empDap接口
    @Autowired
    private EmployeeDAO empDao;

    @Override
    public List<Employee> getEmpList() {
        List<Employee> ls = empDao.getEmpList();
        // 2. 对数据进行处理
        ls.forEach(e -> {
            if (e.getGender() == "0")
                e.setGender("Female");
            else
                e.setGender("Male");
        });

        return ls;
    }
}
````

### 依赖注入 Dependency Injection
容器为程序提供其所依赖的资源。

用法： 声明成员变量（通常是一个接口），然后使用@Autowire注解来为其赋值。与IOC搭配使用。
````
@RestController
public class EmployeeController {
    // 使用DI为empService接口注入
    @Autowired
    private EmpService empService;
    
    @RequestMapping("/EmpList")
    public List<Employee>getEmpList(){
        List<Employee> ls = empService.getEmpList();
        // 3. 响应请求
        return ls;
    }
}
````

如果IOC容器中有多个实现该接口的类，则会报错。为此可以使用@Primary, @Qualifier, @Resource。不过它们的功能只是允许显式指明要使用哪个Bean，感觉用处不大。

### Bean对象
IOC容器中创建和管理的对象，称为Bean。

#### Bean 类型
被@Component注解声明的类被当作Bean对象。除了宽泛的@Component外，还有衍生的@Controller,@Service,@Repository来分别创建控制层，逻辑层和数据层的Component。

![Alt text](pic/beanType.png)

#### Bean 扫描
@ComponentScan注解用来声明去哪里寻找Bean。默认范围是注解所在的当前包和子包。

@ComponentScan包含在启动类的@SpringBootApplication注解中。

## 配置文件
### 配置文件类型
Spring Boot 支持property和yml两种类型的配置文件，它们俩语法不同，但是作用是完全相同的。

### 参数配置化
项目中有很多代码需要配置参数，例如连接数据库，连接云端服务器等，与其将代码分散配置，不如放在配置文件中统一管理。

#### property
在property文件中，可以定义变量并赋值
````
myTest.v1 = hello
myTest.v2 = cain
````

#### @Value
@Value只能修饰成员变量，框架在启动时会查找application.property中同名变量并为被@Value修饰的类创建实例并且作为bean。
````
public someclass{
    @Value("${myTest.v1}")
    private String v1;

    @Value("${myTest.v2}")
    private String v2;
}
````

#### @ConfigurationProperties
@Value在修饰很多值的时候会显得臃肿，@ConfigurationProperties可以修饰整个类，而不是单个成员变量。

创建一个配置类，该类必须有get/set方法以及需要成为bean，所有有@Data和@Component。（一般来说，该配置类会在utils包下）

@ConfigurationProperties(prefix = "")会在property文件中查找prefix为特定值的变量，并根据变量名来匹配类中的成员变量。

````
@Data
@Component
@ConfigurationProperties(prefix = "ali.yun")
public class AliYunConfigure {
    private String endpoint;
    private String accessKeyId;
    private String bucketName;
}
````
application.property
````
ali.yun.endpoint = "endpoint1"
ali.yun.access-key-id= "123456"
ali.yun.bucket-name="bucket1"
````

在使用时需要创建bean对象：
````
@Autowired
private AliYunConfigure aliYunConfigure;

@RequestMapping("/test")
public void test() {
    System.out.println(aliYunConfigure.getEndpoint());
    System.out.println(aliYunConfigure.getAccessKeyId());
    System.out.println(aliYunConfigure.getBucketName());
}
````

### 工具包
有个工具包可以智能识别被标注的变量，从而在写property时，能够自动补全变量名。

````
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-configuration-processor</artifactId>
</dependency>
````