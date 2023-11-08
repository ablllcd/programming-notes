## 三层架构
三层架构不一定属于Spring boot，算是一个思想吧，因为后边要用到，在这里写一下。

### Controller
控制层：接受前端的请求，做出处理，并响应数据。(其中需要调用`Service`层)

### Service
逻辑处理层：处理具体的业务逻辑。(其中需要调用`DAO`层)

### Data Access Object
数据层：对持久的数据进行增删改查。

## 核心容器
### Inversion of Control (IOC) 
思想：对象的创建不再由`程序`来负责，而是由`外部`来负责创建，程序直接使用外部创建的对象。

目的：降低代码的耦合度。

实现： Spring提供了一个容器，称为IOC容器，来充当思想中的`外部`，负责创建和提供对象。在IOC容器中存放的对象被称为`Bean`。

### Dependency Injection (DI)
思想：在容器的bean和bean直接建立依赖。

目的：在使用某个bean时，它所依赖的bean也会被创建，确保被使用的bean能够运行