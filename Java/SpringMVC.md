## MVC 概述

首先MVC是程序架构的一种思想，它将项目分为Module，View，和Controller。这方便项目的开发和管理，是独立于技术的。

而在java中，Module层通常使用实体类来存储数据，用Service类来处理数据操作；View层通常是用html或者jsp文件来实现；而Controller层则是用Servlet技术来实现。

整体流程为：用户和html页面交互->发送请求到Servlet->Servlet调用Service类来获取数据或者处理逻辑->Servlet根据Service的结果返回页面或者重定向

## Spring MVC 概述

而Spring MVC是Spring中的一个框架，可以方便MVC类型的程序开发。既然是框架，自然提供了半成品代码方便程序的构建。而且作为Spring的一个子项目，它可以和Spring的其它框架衔接，例如使用Spring Framwork的IOC容器。

Spring MVC的核心是封装了Servlet，并且提供了一个DispatcherServlet（前端控制器）。所有的请求都先经过DispatcherServlet，在它预处理后再分发给其它的Servlet。这样的好处就是自己创建和使用servlet会变得更加简单。从中也可以看出，SpringMVC其实主要是对Controller层的封装，对于Module和View没有太多的改进。

## Quick Start

1. 创建WEB项目，配置tomcat（查看/Editor/idea笔记）

2. 添加配置文件

```
    <dependencies>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-webmvc</artifactId>
            <version>6.1.2</version>
        </dependency>
<!--        <dependency>-->
<!--            <groupId>javax.servlet</groupId>-->
<!--            <artifactId>javax.servlet-api</artifactId>-->
<!--            <version>4.0.1</version>-->
<!--            <scope>provided</scope>-->
<!--        </dependency>-->
        <dependency>
            <groupId>jakarta.servlet</groupId>
            <artifactId>jakarta.servlet-api</artifactId>
            <version>6.0.0</version>
            <scope>provided</scope>
        </dependency>
    </dependencies>
```

注意：Spring6用的是jakarta.servlet，而不再是javax.servlet了。与此同时，tomcat也需要改成10版本。

3. 写web.xml配置文件，注册dispatcherServlet

```
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">
    
    <servlet>
        <servlet-name>DispatcherServlet</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <!-- 注册Spring的配置文件 -->
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:SpringMVC.xml</param-value>
        </init-param>
    </servlet>
    <servlet-mapping>
        <servlet-name>DispatcherServlet</servlet-name>
<!--        表示所有请求都映射到DispatcherServlet，但是.jsp文件除外。-->
<!--        这个跟jsp文件的性质有关，没用到jsp文件，所以不懂-->
        <url-pattern>/</url-pattern>
    </servlet-mapping>
</web-app>
```

4. 写Spring配置文件（之前的bean.xml），开启组件扫描

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context.xsd
       http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd">

    <context:component-scan base-package="com.cain.controller"></context:component-scan>

</beans>
```

5. 写自己的Servlet注册函数

```
@RestController
public class HelloController {
    @RequestMapping("/")
    public void index(){
        System.out.println("index is called");
    }
}
```

## @RestMapping 匹配用户请求

@RestMapping用来匹配url到Servlet方法，其中Value参数必须设置，其它参数可选。

### Value 参数

value 通过资源路径进行匹配，可以设置多个url

```
@RequestMapping(value = {"/index","/"})
public void index(){
    System.out.println("index is called");
}
```

### Method 参数

Method 根据请求方法进行筛选

```
@RequestMapping(
        value = {"/index"},
        method = {RequestMethod.POST}
)
public String index(){
    System.out.println("index is called");
    return "index page";
}
```

也可以通过衍生注解名进行配置

```
@PostMapping (
        value = {"/index"}
)
public String index(){
    System.out.println("index is called");

    return "index page";
}
```

### Parameter 参数

params要求必须携带的参数，也可通过```a=b a!=b !a```来进行更具体的限制。

```
@RequestMapping (
        value = {"/index"},
        params = {"username"}
)
public String index(){
    System.out.println("index is called");
    return "index page";
}
```

### Header 参数

根据请求头进行限定，要求必须有或者没有某个字段，或者限制字段的值

```
@RequestMapping (
        value = {"/index"},
        headers = ("Host=localhost:8080")
)
public String index(){
    System.out.println("index is called");
    return "index page";
}
```

还有其它参数，需要了再查看API文档

### ANT风格路径通配符

RequestMapping支持ANT风格的路径通配符

* ? 匹配任何单字符
* *匹配0或者任意数量的 字符
* ** 匹配0或者更多的目录这里注意了单个* 是在一个目录内进行匹配。而** 是可以匹配多个目录，一定不要迷糊

### 从路径获取变量

通过在RequestMapping中设置占位符，然后在方法参数中声明并使用。这种方式常用于Restful的url。

```
@RequestMapping("/ParaTest/{username}")
public String paraTest(@PathVariable("username") String username){
    System.out.println("paratest is called");
    return (String) username;
}
```

## 处理Request请求

### 调用原生Servlet

@Controller是由DispatcherServlet调用的，而DispatcherServlet是包含request和response参数的，所以它可以直接穿给@Controller的方法：

```
@RequestMapping("/servletRequest")
public String servletRequest(HttpServletRequest httpServletRequest){
    String user = httpServletRequest.getParameter("user");
    return user;
}
```

其中HttpServletRequest是由DispatcherServlet来赋值的


### SpringMVC 请求参数

获取Request大多是为了获取其中的参数，对此SpringMVC进行了封装，将Request中的参数取出，直接传递给@Controller中的方法：

```
@GetMapping("/directPara")
public String directPara(@RequestParam("username") String username){
    System.out.println("paratest is called: "+username);
    return username;
}
```


这里@RequestParam("username")匹配get请求中的username参数。值得注意的是：在之前的spring版本中，可以不设置@RequestParam注解，只要参数名和请求中的参数名一致就行，但是spring6好像不允许了。

**实体类封装参数**

除了指定String或者其它基本类型接受参数，还可以直接传递一个实例对象，根据成员变量的名称自动封装请求参数到实例对象中去：

```
@RequestMapping("/pojoTest")
public String pojoTest(User user){
    System.out.println(user);
    return "pojoTest";
}
```
```
@Data
public class User {
    private String username;
    private String password;
}
```

### SpringMVC 请求头

http请求头中也是键值对，也可以用类似@RequestParam的方法获取相关信息

```
@GetMapping("/headerPara")
public String headerPara(@RequestHeader("host") String localhost){
    return localhost;
}
```

### SpringMVC Cookie

此外对于cookie也有注解来获取相关信息：@CookieValue。暂且没用，跳过.


## Model和View

在controller中，我们已经映射到了url，也可以获得请求中的参数。为此我们可以做很多事，例如从mysql获取数据，存储数据，以及做逻辑处理，而这又涉及到了另一个分层架构：controller-service-dao。

正如我们上边所述，model层对应的就是service和dao两部分，而这一块会用Mybatis框架来实现，所以这里略过。

那现在的问题是，如何利用数据来构建View？其实如果做前后端分离就不用考虑了，直接返回数据给前端工程师就好，这也是流行的做法。不过下边还是讲一下一体式开发中如何构建view。

## 向View传递数据

### 传统Request域

*注意：前文为了方便没有整合thymeleaf，而是用来@RestController注解。下边为了构建view，是添加了thymeleaf注解的，如何添加看笔记：Spring-Other.md*

数据传递的思路是从controller跳转到html模板，通过共享的Request域来实现。

```
@RequestMapping("/t1")
public String t1(HttpServletRequest httpServletRequest,User user){
    httpServletRequest.setAttribute("username",user.getUsername());
    httpServletRequest.setAttribute("password",user.getPassword());

    return "success";
}
```

###  SpringMVC 共享Request数据

**ModelAndView**

这个类可以由DispatcherServlet提供，也可以自己new，但是返回类型必须是ModelAndView。

```
@RequestMapping("/t2")
public ModelAndView t2(ModelAndView modelAndView, User user){
//        ModelAndView modelAndView = new ModelAndView();
    modelAndView.addObject("username",user.getUsername());
    modelAndView.addObject("password",user.getPassword());

    modelAndView.setViewName("success");
    return modelAndView;
}
```

**Model**

```
@RequestMapping("/t3")
public String t2(Model model){
    model.addAttribute("username","model test");

    return "success";
}
```

此外还有通过Map参数，ModelMap参数等方法共享数据。（不过Map,Model,ModelMap的实例对象其实是同一个，它继承了ModelMap，并且实现了Map和Model接口）

### 类似的还有Session和Application域对象，略过

## 内部跳转和重定向

在SpringMVC中内部跳转和重定向被认为是View，这样的话@Controller中方法的返回值都是View了，应该是为了贯彻MVC的思想吧。

一般的方法会返回一个String类型的对象，这被认为是ViewName，被bean中的ThymeleafViewResolver处理为Thymeleaf View。而要进行跳转就需要返回InertnalResource View的View Name，而重定向就需要返回Redirect View 的Name。其特征如下：

* 普通视图View Name：无前缀
* Internal Resource：forward前缀
* Redirect：redirect前缀

### 内部跳转：InternalResource View

```
@RequestMapping("/thymeleafView")
public ModelAndView thymeleaf(ModelAndView modelAndView){
    System.out.println("thymeleaf is called");
    modelAndView.setViewName("success");
    return modelAndView;
}

@RequestMapping("/internal")
public ModelAndView internal(ModelAndView modelAndView){
    modelAndView.addObject("username","internal user");
    modelAndView.addObject("password","2211");
    modelAndView.setViewName("forward:/thymeleafView");
    return modelAndView;
}
```

内部跳转直接的参数共享，View的参数传递都是由ModelAndView实现的

### 重定向： Redirect View

```
@RequestMapping("/redirect")
public String redirectView(HttpServletRequest request){
    ServletContext servletContext = request.getServletContext();
    servletContext.setAttribute("username","redirect user");
    servletContext.setAttribute("password","7788");

    return "redirect:/thymeleafView";
}
```

可以用servletContext传参，但是不好用，会很麻烦

## 视图控制器 View Controller

如果Controller中的方法比较简单，都是直接返回视图：
```
@RequestMapping("/index")
public String index(){
    return "index";
}
```
其实可以在spring的xml配置文件中配置视图控制器：
```
<mvc:view-controller path="/index" view-name="index"></mvc:view-controller>
```
但是有个弊端是，它会默认关闭注解功能，也就是Controller中其它的映射都失效了，所以需要在配置文件中把spring的注解功能打开
```
<mvc:annotation-driven></mvc:annotation-driven>
```

## HttpMessageConverter

HttpMessageConver是Spring框架提供的一个类，用来进行http报文和java对象之间的转换。它的功能通过httpRequest和response都能实现，只是它更加方便一些：

### @RequestBody

将请求体封装，但也要求请求必须带请求体
```
@RequestMapping("/testConverter")
public String converter(@RequestBody String request){
    System.out.println("request body: "+request);
    return "success";
}
```

### RequestEntity

```
@RequestMapping("/testConverter")
public String converter(RequestEntity<String> request){
    System.out.println("request header: "+request.getHeaders());
    System.out.println("request body: "+request.getBody());
    return "success";
}
```

### @ResponseBody

默认情况下返回字符串是交给thymeleaf来进行页面查找，普通的servlet想要返回页面是进行重定向，想要返回字符串是通过response.write()。而@Response可以直接将返回的字符串作为响应体

```
@RequestMapping("/res")
@ResponseBody
public String res(){
    return "res result";
}
```

虽然http响应体只能是字符串，如果添加了jackson依赖，在添加@ResponseBody注解的话，会自动将java对象转为json格式的字符串

```
@RequestMapping("/res")
@ResponseBody
public User res(){
    return new User("cain","222");
}
```

```
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.16.1</version>
</dependency>
```

此外还有@RestController，它是@ResponseBody+@Controller

## 用配置类代替xml文件

servlet6中有jakarta.servlet.ServletContainerInitializer接口，服务器会在类路径查找实现了该接口的类，并将其当作配置类。其中Spring写了SpringServletContainerInitializer实现了该类，并且Spring的这个实现类会去查找WebApplicationInitializer的实现类作为配置类，而Spring用AbstractAnnotationConfigDispatcherServletInitializer实现类部分WebApplicationInitializer，我们创建配置类继承它就行：

其中getRootConfigClasses和getServletConfigClasses都是用来指明Spring的配置类，只不过Root的应用范围更广，详情见：https://blog.csdn.net/sujz12345/article/details/52975576

```
// web.xml配置文件对应的配置类
public class WebInit extends AbstractAnnotationConfigDispatcherServletInitializer {
    @Override
    protected Class<?>[] getRootConfigClasses() {
        return new Class[0];
    }

    // 指明Spring相关配置对应的配置类
    @Override
    protected Class<?>[] getServletConfigClasses() {
        return new Class[]{DispatcherServletConfig.class};
    }

    // 指明dispatcherServlet对应的url
    @Override
    protected String[] getServletMappings() {
        return new String[]{"/"};
    }
}
```

DispatcherServletConfig
```
@Configuration
@ComponentScan("com.cain.controller")
@EnableWebMvc
public class DispatcherServletConfig implements WebMvcConfigurer {

    @Override
    public void addViewControllers(ViewControllerRegistry registry) {
        registry.addViewController("/index").setViewName("index");
    }

    @Bean
    ThymeleafViewResolver viewResolver(SpringTemplateEngine engine){
        ThymeleafViewResolver resolver = new ThymeleafViewResolver();
        resolver.setTemplateEngine(engine);
        resolver.setCharacterEncoding("UTF-8");
        return resolver;

    }

    @Bean
    SpringTemplateEngine templateEngine(SpringResourceTemplateResolver templateResolver){
        SpringTemplateEngine engine = new SpringTemplateEngine();
        engine.setTemplateResolver(templateResolver);
        return engine;
    }

    @Bean
    SpringResourceTemplateResolver templateResolver(){
        SpringResourceTemplateResolver resolver = new SpringResourceTemplateResolver();
        resolver.setPrefix("/WEB-INF/templates/");
        resolver.setSuffix(".html");
        resolver.setTemplateMode("HTML");
        resolver.setCharacterEncoding("UTF-8");
        return resolver;
    }
}
```