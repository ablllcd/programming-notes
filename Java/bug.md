## Maven
### 1. lifecircle中的compile失败
   
    [ERROR] Failed to execute goal org.apache.maven plugins:maven-compiler-plugin:3.11.0:compile (default-compile) on project demo: Fatal error compiling: 无效的标记: --release -> [Help 1]

**原因1：maven与jdk版本不匹配**

解决方法：

1. 在pom文件检查jdk版本

        <properties>
        <java.version>1.8</java.version>
        </properties>

2. 在pom中检查maven版本（版本不能太高）

        <build>
            <plugins>
                <plugin>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-maven-plugin</artifactId>
                    <version>2.3.3.RELEASE</version>
                </plugin>
            </plugins>
        </build>
    或者

        <dependencies>
        ...
        </dependencies>
        <build>
            <plugins>
                <plugin>
                    <groupId>org.apache.maven.plugins</groupId>
                    <artifactId>maven-compiler-plugin</artifactId>
                    <configuration>
                        <source>1.8</source>
                        <target>1.8</target>
                    </configuration>
                </plugin>
            </plugins>
        </build>

3. 右击pom文件，查看effective pom，确保更改生效

**原因2：srpingboot与jdk版本不匹配**

解决方法：

在initialize时，选自较低版本的springboot框架。

例如我选择了spring boot 3.x版本后，哪怕选java 8版本，pom中依然显示java 17，这可能暗示了不支持java 8。更换为spring boot 2.x版本后问题解决。

## Junit
### 1. 无法找到匹配的方法/类
    java.lang.Exception: No tests found matching Method

原因：只下载了junit.jar包，没有下载hamcrest-core.jar包

解决方法：把hamcrest-core.jar包也下载到classpath中

## SpringBoot
### 1. Tomcat端口号被占用
解决方法：在项目`main/resource/application.properties`中添加代码：
````
server.port=8000
````
来设置端口号。

### 2. JDBC 报错
错误信息：java.lang.IllegalStateException: Failed to load ApplicationContext

情况：在springboot框架下的测试用例中使用jdbc导致报错。

原因：不明

解决方法： 不再spring框架中使用，而在单独程序中的main方法中使用可正常运行。
