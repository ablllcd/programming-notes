## Web服务器
想要别人能够通过网络访问到自己的资源和程序，需要下载`web服务器软件`来负责处理协议以及实现部署。

web服务器可以：

1. 封装HTTP协议，便于web开发
2. 部署web项目，使其能够通过浏览器访问

### Tomcat服务器
服务器和部署的程序交互需要接口，而Tomcat支持的是Servlet规范，可以运行Servlet程序，因此也被成为Servlet容器。

工作流程如下：
1. Client发送http请求给服务器（tomcat）
2. 如果是静态资源，服务器之间返回页面
3. 如果是动态资源，查看Web.xml中注册的servlet
4. 调用对应的servlet实现类（因为tomcat也是知道servlet接口的，并且提供了servlet依赖，所以它知道如何处理servlet类。servlet相当于服务器和java遵循的规则）

**TomCat 下载、配置、启动**

下载地址：https://tomcat.apache.org/ 选择版本下载zip，解压就行，不需要安装过程。

配置端口： 修改conf/server.xml文件中的Connector port

日志乱码： 修改conf/logging.properties中的java.util.logging.ConsoleHandler.encoding，将UTF-8改为GDK

启动：双击bin/startup.mat

**IDEA 配置 Tomcat**

见/Editor/idea.md里的笔记


## Servlet

Servlet是在服务器端运行的java程序，功能有点类似javascript，是用来构建动态页面的。



### quick start

1. 添加依赖 (javaEE就不需要了)

    由于Servlet不是javeSE的一部分，需要先引入依赖
    ```
    <dependency>
        <groupId>javax.servlet</groupId>
        <artifactId>javax.servlet-api</artifactId>
        <version>4.0.1</version>
        <scope>provided</scope>
    </dependency>
    ```

2. 实现Sevlet接口

    ```
    public class HelloServlet implements Servlet {
        @Override
        public void init(ServletConfig servletConfig) throws ServletException {

        }

        @Override
        public ServletConfig getServletConfig() {
            return null;
        }

        @Override
        public void service(ServletRequest servletRequest, ServletResponse servletResponse) throws ServletException, IOException {
            System.out.println("service is started");
        }

        @Override
        public String getServletInfo() {
            return null;
        }

        @Override
        public void destroy() {

        }
    }
    ```

3. 配置Web.xml文件

    ```
    <servlet>
        <servlet-name>demo1</servlet-name>
        <servlet-class>com.cain.servlets.HelloServlet</servlet-class>
    </servlet>
    <servlet-mapping>
        <servlet-name>demo1</servlet-name>
        <url-pattern>/demo1</url-pattern>
    </servlet-mapping>
    ```

### 生命周期

**1. 初始化**

Tomcat通过反射技术，使用全类名创建Servlet，创建之后调用servlet的init方法。该方法只会在servlet被创建后调用一次：

```
@Override
public void init(ServletConfig servletConfig) throws ServletException {
    System.out.println("init...");
}
```

此外，servlet的创建时机也是可以设置的：如果load-on-startup为0或者正数，则启动服务器时创建；如果是负数（也是默认值），则是servlet被访问时创建。
```
<servlet>
    <servlet-name>demo1</servlet-name>
    <servlet-class>com.cain.servlets.HelloServlet</servlet-class>
    <load-on-startup>1</load-on-startup>
</servlet>
```

**2. 运行**

Servelet每次运行都会调用service方法
```
@Override
public void service(ServletRequest servletRequest, ServletResponse servletResponse) throws ServletException, IOException {
    System.out.println("service2 is started");
}
```

**3. 销毁**

通过destroy方法销毁，在类被销毁时调用
```
@Override
public void destroy() {
    System.out.println("destroy...");
}
```

### Sevlet体系结构

Servlet是程序员和服务端之间的约定，程序员根据这个接口构建实现类，服务端根据这个接口使用实现类。但Servlet是很泛型的一个接口，为了方便使用，还有protocal specific的Servlet子接口，例如常见的HttpSevlet。

HttpSevlet继承Sevlet，所以服务器端不需要了解HttpSevlet是什么，它依旧是调用service方法就行。而HttpSevlet是实现了Sevlet接口，添加一些http常用的功能，从而构建成了一个抽象类，从而方便程序员实现Sevlet。而它放到了Sevlet-api依赖中，成为了一种规范，所以我们可以直接使用。

### Request请求

程序员和服务器之间除了Sevlet接口的约定外，还有SevletRequest和SevletResponse接口的约定。服务器会按照接口将它实际收到的请求给封装起来，也会按照接口将response中的消息给返回给客户端。

具体流程为：在Tomcat接受的请求后，它会创建ServletRequest和ServletResponse的实现类，并且用它们作为参数调用servlet的service方法。那service方法可以用ServletRequest查看请求消息，并且用ServletResponse存储返回结果。

#### Request、Resonpse体系结构

类似的，SevletRequest只是泛型的接口，针对不同的网络协议应该有更具体的接口，例如常见的HttpSevletRequest。HttpSevletRequest继承了SevletRequest接口，并且增加了一些方法来获取http特定的消息。

不过不同于Servlet的体系结构，此时不需要程序员来实现特定接口，而是服务器根据它收到实际网络请求类型来构建对应的特定实现类。由于HttpSevletRequest继承于SevletRequest，服务器可以将HttpSevletRequest传给service方法的SevletRequest参数，程序员只需要进行类型转换就行了（如果用了对应的HttpSevlet，这个抽象类已经做了类型转换，这一步都免了）。

Response也是同理，tomcat根据特定的网络请求类型来创建特定的Response类。

#### Request作用

在实际网络请求中的数据都可以在SevletRequest中取出来，由于API繁多，到时候查文档就行。下边是取URL中参数的例子：

```
@Override
public void service(ServletRequest servletRequest, ServletResponse servletResponse) throws ServletException, IOException {
    HttpServletRequest httpServletRequest = (HttpServletRequest) servletRequest;

    System.out.println(httpServletRequest.getQueryString());
}
```

此外，功能复杂时可能需要多个servlet协助，可以实现servlet跳转并且共享信息：

demo1
```
public void service(ServletRequest servletRequest, ServletResponse servletResponse) throws ServletException, IOException {
    HttpServletRequest httpServletRequest = (HttpServletRequest) servletRequest;
    // 设置共享数据
    String msg = "msg from demo1";
    httpServletRequest.setAttribute("msg",msg);

    // servlet跳转
    RequestDispatcher dispatcher = httpServletRequest.getRequestDispatcher("/demo2");
    dispatcher.forward(httpServletRequest,servletResponse);
}
```

demo2
```
@Override
public void service(ServletRequest servletRequest, ServletResponse servletResponse) throws ServletException, IOException {
    // 获取共享数据
    System.out.println(servletRequest.getAttribute("msg"));
}
```

#### Reponse作用

与Request类似，HTTP响应体中的内容都可以在response对象中设置，像状态码，响应体之类的

```
@Override
public void service(ServletRequest servletRequest, ServletResponse servletResponse) throws ServletException, IOException {
    HttpServletResponse httpServletResponse = (HttpServletResponse) servletResponse;
    // 响应字符
    httpServletResponse.getWriter().write("<h1>my response<h1>");
}
```

也可以实现重定向功能，注意这个不同于Servlet跳转。Servlet跳转是内部进行的，对外不可见；而重定向是返回一个重定向消息给客户端，客户端再次发送http请求
```
@Override
public void service(ServletRequest servletRequest, ServletResponse servletResponse) throws ServletException, IOException {
    HttpServletResponse httpServletResponse = (HttpServletResponse) servletResponse;
    HttpServletRequest httpServletRequest = (HttpServletRequest) servletRequest;

    // 获取资源前缀
    String contextPath = httpServletRequest.getContextPath();
    // url跳转
    httpServletResponse.sendRedirect(contextPath+"/demo2");
}
```

### ServletContext 对象

前边的Requset.setAttribute可以保证在一个request请求中的不同servlet共享信息；而ServletContext则是允许在整个项目运行过程中共享信息。（也就是tomcat不重启部署的项目，信息就一直存在）

demo1
```
@Override
public void service(ServletRequest servletRequest, ServletResponse servletResponse) throws ServletException, IOException {
    HttpServletResponse httpServletResponse = (HttpServletResponse) servletResponse;
    HttpServletRequest httpServletRequest = (HttpServletRequest) servletRequest;

    // 设置ServletContext信息
    String globalMsg = "this is a global message";
    servletRequest.getServletContext().setAttribute("msg",globalMsg);

    // url跳转
    String contextPath = httpServletRequest.getContextPath();
    httpServletResponse.sendRedirect(contextPath+"/demo2");
}
```

demo2
```
@Override
public void service(ServletRequest servletRequest, ServletResponse servletResponse) throws ServletException, IOException {
    ServletContext servletContext = servletRequest.getServletContext();
    servletResponse.getWriter().write((String) servletContext.getAttribute("msg"));
}
```

### 注解替代web.xml

web.xml主要功能就是映射url到servlet，在servlet3.0之后，这个工作可以有@WebServlet注解实现：

```
// 映射url到当前servlet
@WebServlet("/demo2")
public class HelloServlet implements Servlet {
}
```

（web.xml内可以不写映射，但是如果我删除web.xml的话会报错）

**一个Servlet匹配多个url**

如果希望一个servlet匹配多个url有两种方法：

一种是通过列表
```
@WebServlet({"/demo2","/ddd"})
```

另一种是通过正则表达式
```
@WebServlet("*.test")
```

