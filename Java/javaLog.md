## 日志体系结构

![Alt text](pic/logStructure.png)

现有很多日志框架来进行日志技术，而这些框架也遵循某种日志接口，从而方便我们学习。

## Logback简介

LogBack是比较新的一套日志框架，旨在解决Log4j框架的问题。LogBack包含三个模块：core, classic and access。 

Core是核心模块，是其它俩模块的基础。Classic是对SLF4J接口的实现，而access是整合了servlet，从而提供HTTP-ACCESS服务。

## Logback 依赖
要使用LOGBACK，至少要core和classic模块，并且引入SLF4J接口。

````
<dependency>
    <groupId>ch.qos.logback</groupId>
    <artifactId>logback-core</artifactId>
    <version>1.2.3</version>
</dependency>

// classic 依赖于其它两个，所以只引入它就行
<dependency>
    <groupId>ch.qos.logback</groupId>
    <artifactId>logback-classic</artifactId>
    <version>1.2.3</version>
</dependency>

<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-api</artifactId>
    <version>1.7.26</version>
</dependency>
````

## quick start
```
package chapters.introduction;

public class HelloWorld2 {

    public static void main(String[] args) {
        Logger logger = LoggerFactory.getLogger("chapters.introduction.HelloWorld2");
        logger.debug("Hello world");
        
        // 打印Logback自身的日志
        LoggerContext lc = (LoggerContext)LoggerFactory.getILoggerFactory();
        StatusPrinter.print(lc);
    }
}
```
基本操作为：1. 通过类名获取Logger（指明哪个类在产生日志） 2. 利用logger输出日志，输出可以有不同的level。

如果需要，还可以输出Logback框架当前的状态信息，例如在使用哪个配置文件等。

## logback的架构

logback主要由3部分构成：
* logger: 生产日志消息的对象。通常不同类会根据自己的类名来获得不同的logger。这样可以将日志空间进行划分，从而能够禁止某些日志的输出，但是又不会妨碍另一些日志的输出。
* Appender: 日志消息输出的目的地
* Layout: 日志消息的输出格式

### logger 

**logger 创建和获取**

通过LoggerFactory.getLogger(loggerName)获取logger，如果logger不存在，则创建logger。

```
Logger logger = LoggerFactory.getLogger("chapters.introduction.HelloWorld2");
```

Logger创建在logback配置文件中完成。
```
<logger name="com" level="INFO">
    <appender-ref ref="STDOUT"/>
</logger>
```

logger还有点IOC的感觉：
1. 在不同地方通过相同名称获取的logger为同一对象，对该对象的修改也是共享的（单例模式）
2. 父级 logger 会自动寻找并关联子级 logger，即使父级 logger 在子级 logger 之后实例化

**树状结构**

通常logger的名称对应java中的包名+类名，所以logger划分的日志空间也有树状结构。其中根节点为root。其它节点的继承关系由 . 来区分。例如com.example是com的子类。

**输出level**

logger可以指定输出的level，如果日志的level低于logger的输出level，则该日志不被输出。其中TRACE < DEBUG < INFO < WARN < ERROR。

logger的输出level也会收到树状结构的影响，例如：com.aa没有指明level，那它将继承其父类com的level。如果com.aa指明了level，则无需考虑父类的level。其中root默认的level为debug。

```
public void layerTest(){
    // 父类为info level，不会打印debug信息
    ch.qos.logback.classic.Logger logger = (ch.qos.logback.classic.Logger) LoggerFactory.getLogger("com");
    logger.setLevel(Level.INFO);
    logger.debug("debug father");

    // 子类继承父类，也为info level，不会打印debug信息
    ch.qos.logback.classic.Logger logger2 = (ch.qos.logback.classic.Logger) LoggerFactory.getLogger("com.aaa");
    logger2.debug("debug child");

    // 没有父类，没设置level，继承root，level为debug
    ch.qos.logback.classic.Logger logger3 = (ch.qos.logback.classic.Logger) LoggerFactory.getLogger("org");
    logger3.debug("debug other");
}
```

### Appender

logback 允许日志在多个地方进行输出，输出目的地叫做 appender。 而且一个 logger 可以有多个 appender。

**添加appender**

logger 通过 addAppender 方法来新增一个 appender。每一个允许输出的日志都会被转发到该 logger 的所有 appender 中去。

但是树状结构也会影响logger绑定的appender: 子类的消息除了给自身的appender，还会传递给父类的appender。为此我们可以设置additivity属性来选择是否传递消息给父类（默认开启）：

```
<logger name="com" level="INFO">
    <appender-ref ref="STDOUT"/>
</logger>

<logger name="com.aaa" additivity="false">
    <appender-ref ref="STDOUT"/>
</logger>

<root level="debug">
    <appender-ref ref="STDOUT"/>
</root>
```

### Layout

layout 的作用是将日志格式化。PatternLayout 能够根据用户指定的格式来格式化日志，类似于 C 语言的 printf 函数。

例：PatternLayout 通过格式化串 "%-4relative [%thread] %-5level %logger{32} - %msg%n" 会将日志格式化成如下结果：
```
176  [main] DEBUG manual.architecture.HelloWorld2 - Hello world.
```

## 配置文件

在应用程序当中使用日志语句需要耗费大量的精力。推荐使用配置文件来管理这些日志语句。

### 配置文件初始化
以下是 logback 的初始化步骤：

1. logback 会在类路径下寻找名为 logback-test.xml 的文件。
2. 如果没有找到，logback 会继续寻找名为 logback.groovy 的文件。
3. 如果没有找到，logback 会继续寻找名为 logback.xml 的文件。
4. 如果没有找到，将会通过 JDK 提供的 ServiceLoader 工具在类路径下寻找文件 META-INFO/services/ch.qos.logback.classic.spi.Configurator，该文件的内容为实现了 Configurator 接口的实现类的全限定类名。
5. 如果以上都没有成功，logback 会通过 BasicConfigurator 为自己进行配置，并且日志将会全部在控制台打印出来。

对于maven项目，在 src/test/resources 下新建 logback-test.xml。maven 会确保它不会被生成。所以你可以在测试环境中给配置文件命名为 logback-test.xml，在生产环境中命名为 logback.xml。

**配置文件结构：**
```
<configuration>
	<appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
		<encoder>
			<pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
		</encoder>
	</appender>
	
	<logger name="chapters.configuration" level="INFO" />

	<root level="DEBUG">
		<appender-ref ref="STDOUT" />
	</root>
</configuration>
```



## 推荐参考资料
https://github.com/YLongo/logback-chinese-manual