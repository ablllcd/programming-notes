## 大概思路

例如Spring通过new AnnotationConfigApplicationContext(Config.class)作为IOC容器。那么所有的起点就是ApplicationContext的构造方法。

### 构造方法

总体来说，构造方法里干两件事：扫描bean class并创建beanDefinition；创建单例bean.

1. 在构造方法中，首先ApplicationContext(实际上应该是BeanFactory)通过反射读取配置类中的@ComponentScan的值，从而判断哪些类文件需要进行检查。然后将被@Component注解标识的类筛选出来
2. 找到bean class后通过类加载器将这些类加载，为其构造beanDefinition并存储到beanDefinition池中（Map）。beanDefinition至少存储该bean的scope和type。此外，当判断出该bean class为单例时，创建该bean的实例并且存储到单例池中（Map）。

### getBean方法

IOC容器的核心方法就是获取bean。当构造方法结束，所有的bean都有对应的beanDefinition，并且单例bean已经生成了实例。

那当getBean方法被调用时，有两种情况：

1. 单例则从单例池中返回
2. 多例则创建bean并且返回



## beanFactory讲解

以下代码是beanFactory的使用示例：

```
package com.cain.springtheory;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.config.BeanFactoryPostProcessor;
import org.springframework.beans.factory.config.BeanPostProcessor;
import org.springframework.beans.factory.support.AbstractBeanDefinition;
import org.springframework.beans.factory.support.BeanDefinitionBuilder;
import org.springframework.beans.factory.support.DefaultListableBeanFactory;
import org.springframework.context.annotation.AnnotationConfigUtils;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

public class BeanFactoryTest {
    public static void main(String[] args) {
        // 创建bean 容器，此时容器为空
        DefaultListableBeanFactory beanFactory = new DefaultListableBeanFactory();

        // 创建bean定义（class, type）
        AbstractBeanDefinition beanDefinition =
                BeanDefinitionBuilder.genericBeanDefinition(Config.class).setScope("singleton").getBeanDefinition();

        // 向bean容器中添加 bean 定义
        beanFactory.registerBeanDefinition("config", beanDefinition);

        // 发现此时bean容器中只有config类，而没有config中被@bean修饰的bean1,bean2
        System.out.println("---------------------------------");
        for (String definitionName : beanFactory.getBeanDefinitionNames()) {
            System.out.println(definitionName);
        }

        // 通过辅助类给bean容器添加后处理器类
        AnnotationConfigUtils.registerAnnotationConfigProcessors(beanFactory);

        // 调用辅助类添加的 beanFactory 后处理器，其重要功能就是添加额外的bean，例如处理@Bean注解
        for (BeanFactoryPostProcessor postProcessor : beanFactory.getBeansOfType(BeanFactoryPostProcessor.class).values()) {
            postProcessor.postProcessBeanFactory(beanFactory);
        }

        // 此时bean1和bean2被添加
        System.out.println("---------------------------------");
        for (String definitionName : beanFactory.getBeanDefinitionNames()) {
            System.out.println(definitionName);
        }
        // 然而bean1中没有bean2
//        System.out.println("---------------------------------");
//        System.out.println(beanFactory.getBean(Bean1.class).getBean2());

        // 注册bean后处理器类为后处理器(bean后处理器也是配置类添加的，只是它们只被放到了bean容器，并没有实际注册为后处理器)
        // bean后处理器负责bean生命周期的相关工作，例如处理@Autowired，@Resource等注解
        beanFactory.getBeansOfType(BeanPostProcessor.class).values().forEach((beanPostProcessor) -> {
            beanFactory.addBeanPostProcessor(beanPostProcessor);
        });

        // 为了避免getBean时才创建单例bean，可以提前将单例bean创建好
        beanFactory.preInstantiateSingletons();
        // 此时bean1中有bean2
        System.out.println("---------------------------------");
        System.out.println(beanFactory.getBean(Bean1.class).getBean2());

        beanFactory.getDependencyComparator();
    }


    @Configuration
    static class Config {
        @Bean
        public Bean1 bean1(){
            return new Bean1();
        }
        @Bean
        public Bean2 bean2(){
            return new Bean2();
        }
    }

    static class Bean1 {

        public Bean1(){
            System.out.println("Bean1 constructor");
        }

        @Autowired
        private Bean2 bean2;

        public Bean2 getBean2() {
            return bean2;
        }
    }
    static class Bean2 {
        public Bean2(){
            System.out.println("Bean2 constructor");
        }
    }
}

```

从中我们可以观察到beanFactory的一些特点：

1. 不会主动调用beanFactory后处理器
2. 不会主动绑定bean后处理器
3. 不会主动创建单例bean
4. 不会主动解析 ${} 和 #{}

beanFactory只是充当基础的bean容器。

### 后处理器排序

前边提到AnnotationConfigUtils为beanFactory添加了许多后处理器，那如何决定后处理器的执行顺序呢？例如一个bean同时被@Autowired和@Resource注解，应该先使用哪个对应的后处理器？

其实在AnnotationConfigUtils添加后处理器时，也设置了beanFactory的比较器。该比较器会获得后处理器的order属性从而决定哪个后处理器先执行。

## ApplicationContext讲解

ApplicationContext内部包含beanFactory，并且做了一些额外的工作来使用beanFactory。

ApplicationContext有很多实现：
```
package com.cain.springtheory;

import org.springframework.beans.factory.config.ConfigurableListableBeanFactory;
import org.springframework.beans.factory.support.DefaultListableBeanFactory;
import org.springframework.beans.factory.xml.XmlBeanDefinitionReader;
import org.springframework.boot.autoconfigure.web.servlet.DispatcherServletRegistrationBean;
import org.springframework.boot.web.embedded.tomcat.TomcatServletWebServerFactory;
import org.springframework.boot.web.server.WebServerFactory;
import org.springframework.boot.web.servlet.context.AnnotationConfigServletWebServerApplicationContext;
import org.springframework.boot.web.servlet.server.ServletWebServerFactory;
import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.annotation.Bean;
import org.springframework.context.support.ClassPathXmlApplicationContext;
import org.springframework.context.support.FileSystemXmlApplicationContext;
import org.springframework.core.io.ClassPathResource;
import org.springframework.web.context.WebApplicationContext;
import org.springframework.web.context.support.AnnotationConfigWebApplicationContext;
import org.springframework.web.servlet.DispatcherServlet;
import org.springframework.web.servlet.ModelAndView;
import org.springframework.web.servlet.mvc.Controller;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

public class ApplicationContextTest {

    public static void main(String[] args) {
//        testClassPath();
//        testFileSystem();
//        testXmlReader();
        testConfig();
//        testWebConfig();
    }

    public static void testClassPath(){
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("beans.xml");
        for (String beanDefinitionName : context.getBeanDefinitionNames()) {
            System.out.println(beanDefinitionName);
        }
    }

    public static void testFileSystem(){
        FileSystemXmlApplicationContext context = new FileSystemXmlApplicationContext("src/main/resources/beans.xml");
        for (String beanDefinitionName : context.getBeanDefinitionNames()) {
            System.out.println(beanDefinitionName);
        }
    }

    public static void testConfig(){
        AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(Config.class);
        ConfigurableListableBeanFactory beanFactory = applicationContext.getBeanFactory();
        for (String beanName : beanFactory.getBeanDefinitionNames()) {
            System.out.println(beanName);
        }

        Bean1 bean1 = beanFactory.getBean(Bean1.class);
        System.out.println(bean1.getBean2());
    }

    public static void testWebConfig(){
        AnnotationConfigServletWebServerApplicationContext applicationContext =
                new AnnotationConfigServletWebServerApplicationContext(WebConfig.class);
        ConfigurableListableBeanFactory beanFactory = applicationContext.getBeanFactory();
        for (String beanName : beanFactory.getBeanDefinitionNames()) {
            System.out.println(beanName);
        }
    }

    static class WebConfig{
        // 创建内嵌tomcat服务器
        @Bean
        public ServletWebServerFactory webServerFactory(){
            return new TomcatServletWebServerFactory();
        }

        // 创建DispatcherServlet
        @Bean
        public DispatcherServlet dispatcherServlet(){
            return new DispatcherServlet();
        }

        // 将DispatcherServlet注册到tomcat服务器上
        @Bean
        public DispatcherServletRegistrationBean registrationBean(){
            return new DispatcherServletRegistrationBean(dispatcherServlet(), "/");
        }

        // 创建Controller
        @Bean("/hello")
        public Controller controller1(){
            return new Controller() {
                @Override
                public ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response) throws Exception {
                    response.getWriter().println("hello world");
                    return null;
                }
            };
        }
    }

    static class Config{
        @Bean
        public Bean1 bean1(Bean2 bean2){
            Bean1 bean1 = new Bean1();
            bean1.setBean2(bean2);
            return bean1;
        }
        @Bean
        public Bean2 bean2(){
            return new Bean2();
        }
    }

    static class Bean1{
        private Bean2 bean2;

        public void setBean2(Bean2 bean2) {
            this.bean2 = bean2;
        }
        public Bean2 getBean2(){
            return bean2;
        }
    }
    static class Bean2{}
}

```

beans.xml
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="Bean1" class="com.cain.springtheory.ApplicationContextTest.Bean1">
        <property name="bean2" ref="Bean2"></property>
    </bean>
    <bean id="Bean2" class="com.cain.springtheory.ApplicationContextTest.Bean2"></bean>
</beans>
```

### ClassPathXmlApplicationContext实现过程

```
//创建bean容器
DefaultListableBeanFactory beanFactory = new DefaultListableBeanFactory();

//创建reader并设置读取到哪个容器
XmlBeanDefinitionReader reader = new XmlBeanDefinitionReader(beanFactory);

//读取xml文件
reader.loadBeanDefinitions(new ClassPathResource("beans.xml"));

for (String beanName : beanFactory.getBeanDefinitionNames()) {
    System.out.println(beanName);
}
```


## bean的生命周期

```
package com.cain.springtheory.lifecycle;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

import javax.annotation.PostConstruct;
import javax.annotation.PreDestroy;

@Component
public class BeanLifeCycle {
    Bean1 bean;

    public BeanLifeCycle() {
        System.out.println("构造方法");
    }

    @Autowired
    public void autowiredMethod(Bean1 bean1) {
        this.bean = bean1;
        System.out.println("依赖注入方法");
    }

    @PostConstruct
    public void init() {
        System.out.println("初始化方法");
    }

    @PreDestroy
    public void destroy() {
        bean = null;
        System.out.println("销毁方法");
    }
}
```

bean的生命周期为：构造实例->依赖注入->初始化->销毁

bean的创建除了要经过自己的生命周期外，还会经过后处理器类。后处理器会作用于所有bean的创建，其方法包括：

```
package com.cain.springtheory.lifecycle;

import org.springframework.beans.BeansException;
import org.springframework.beans.PropertyValues;
import org.springframework.beans.factory.config.DestructionAwareBeanPostProcessor;
import org.springframework.beans.factory.config.InstantiationAwareBeanPostProcessor;
import org.springframework.stereotype.Component;

import java.beans.PropertyDescriptor;

@Component
public class MyPostProcessor implements InstantiationAwareBeanPostProcessor, DestructionAwareBeanPostProcessor {
    @Override
    public void postProcessBeforeDestruction(Object o, String s) throws BeansException {
        if(s.equals("beanLifeCycle")){
            System.out.println("后处理器：销毁前执行");
        }
    }

    @Override
    public Object postProcessBeforeInstantiation(Class<?> beanClass, String beanName) throws BeansException {
        if(beanName.equals("beanLifeCycle")){
            System.out.println("后处理器：构造前执行，返回的对象会替换掉bean");
        }
        return null;
    }

    @Override
    public boolean postProcessAfterInstantiation(Object bean, String beanName) throws BeansException {
        if(beanName.equals("beanLifeCycle")){
            System.out.println("后处理器：构造后执行，返回false会跳过依赖注入");
        }
        return true;
    }

    @Override
    public PropertyValues postProcessProperties(PropertyValues pvs, Object bean, String beanName) throws BeansException {
        if(beanName.equals("beanLifeCycle")){
            System.out.println("后处理器：依赖注入时执行");
        }
        return pvs;
    }

    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        if(beanName.equals("beanLifeCycle")){
            System.out.println("后处理器：初始化前执行， 返回的对象会替换掉bean");
        }
        return bean;
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        if(beanName.equals("beanLifeCycle")){
            System.out.println("后处理器：初始化后执行， 返回的对象会替换掉bean");
        }
        return bean;
    }
}

```

### 模板方法

spring将后处理器和bean生命周期的组合依赖于一个思想：模板方法。下边是一个简易的模拟流程：

```
package com.cain.springtheory.lifecycle;


import java.util.ArrayList;
import java.util.List;

public class TemplateMethodTest {
    public static void main(String[] args) {
        MyBeanFactory myBeanFactory = new MyBeanFactory();
        //添加后处理器
        myBeanFactory.addPostProcessor(new BeanPostProcessor() {
            @Override
            public void inject() {
                System.out.println("解析@Autowired");
            }
        });

        myBeanFactory.getBean();
    }

    static class MyBeanFactory{
        List<BeanPostProcessor> beanPostProcessorList = new ArrayList<BeanPostProcessor>();

        public void addPostProcessor(BeanPostProcessor beanPostProcessor){
            beanPostProcessorList.add(beanPostProcessor);
        }

        public Object getBean(){
            // 固定方法1
            System.out.println("构造bean");
            // 固定方法2
            System.out.println("初始化bean");
            // 拓展方法1
            for(BeanPostProcessor beanPostProcessor : beanPostProcessorList){
                beanPostProcessor.inject();
            }
            // 固定方法3
            System.out.println("销毁bean");

            return null;
        }
    }

    interface BeanPostProcessor{
        void inject();
    }
}
```

其中bean的生命周期是固定方法，而后处理器是动态的，可拓展的方法。对于拓展方法，通过接口+集合的方式实现。这样后处理器的添加并不会影响原有代码。


## 常见Bean后处理器
```
package com.cain.springtheory.postprocessor;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.beans.factory.support.AutowireCandidateResolver;
import org.springframework.beans.factory.support.DefaultListableBeanFactory;
import org.springframework.context.annotation.CommonAnnotationBeanPostProcessor;
import org.springframework.context.annotation.ContextAnnotationAutowireCandidateResolver;
import org.springframework.context.support.GenericApplicationContext;
import org.springframework.stereotype.Component;

import javax.annotation.PostConstruct;
import javax.annotation.PreDestroy;
import javax.annotation.Resource;

public class A04Application {
    public static void main(String[] args) {
        // 这个applicationContext是“干净”的，没有自带后处理器
        GenericApplicationContext genericApplicationContext = new GenericApplicationContext();
        // 添加bean
        genericApplicationContext.registerBean("bean1", Bean1.class);
        genericApplicationContext.registerBean("bean2", Bean2.class);
        genericApplicationContext.registerBean("bean3", Bean3.class);

        // 添加后处理器bean
        genericApplicationContext.registerBean(AutowiredAnnotationBeanPostProcessor.class);     //负责处理 @Autowired @Value 注解
        genericApplicationContext.getDefaultListableBeanFactory()
                .setAutowireCandidateResolver(new ContextAnnotationAutowireCandidateResolver());    //由于上述bean处理@Value有问题，所以要设置这么个东西

        genericApplicationContext.registerBean(CommonAnnotationBeanPostProcessor.class);        //负责处理 @Resource @PostConstruct @PreDestroy
        
        // 初始化容器，包括：将后处理器bean注册为后处理器；运行后处理器；创建bean
        genericApplicationContext.refresh();
        genericApplicationContext.close();
    }
}

@Component
class Bean1{
     Bean2 bean2;
    private Bean3 bean3;

    @Autowired
    public void setBean2(Bean2 bean2) {
        this.bean2 = bean2;
        System.out.println("@Autowired works");
    }

    @Resource
    public void setBean3(Bean3 bean3) {
        this.bean3 = bean3;
        System.out.println("@Resource works");
    }

    @Autowired
    public void setValue(@Value("${JAVA_HOME}") String value) {
        // autowired 不会为字符串类型查找bean进行注入，所以这里配合@Value使用
        System.out.println(value);
    }

    @PostConstruct
    public void init(){
        System.out.println("Bean1 @PostConstruct works");
    }

    @PreDestroy
    public void destroy(){
        System.out.println("Bean1 @PreDestroy works");
    }
}

@Component
class Bean2{}

@Component
class Bean3{}
```

上述代码中给出了两个常见后处理器的作用：

1. AutowiredAnnotationBeanPostProcessor：负责处理@Autowired和@Value注解
2. CommonAnnotationBeanPostProcessor：负责处理@Resource @PostConstruct @PreDestroy注解

前边讲bean的生命周期时，将后处理器方法和生命周期方法分开讲解，好像是两者独立。而现在看来，bean的生命周期方法本身也是依赖于后处理器的。


### AutowiredAnnotationBeanPostProcessor分析

```
public class AutowiredProcessorStudy {
    public static void main(String[] args) {
        // 创建beanFactory，添加bean2, bean3并且配置@Value相关的解析器
        DefaultListableBeanFactory beanFactory = new DefaultListableBeanFactory();
        beanFactory.registerSingleton("bean2", new Bean2());
        beanFactory.registerSingleton("bean3", new Bean3());
        beanFactory.setAutowireCandidateResolver(new ContextAnnotationAutowireCandidateResolver());

        // 创建@Autowired的后处理器
        AutowiredAnnotationBeanPostProcessor processor = new AutowiredAnnotationBeanPostProcessor();
        processor.setBeanFactory(beanFactory);

        // 对比bean1来查看后处理器的效果
        Bean1 bean1 = new Bean1();
        System.out.println(bean1);
        processor.postProcessProperties(null, bean1,"bean1");
        System.out.println(bean1);
    }

}
```

其中AutowiredAnnotationBeanPostProcessor的核心方法是调用postProcessProperties方法，再调用该方法后，bean1即完成了@Autowired注解的处理并且实现了依赖注入。

此外，这里的后处理器需要绑定beanFactory是因为解析@Autowired时需要进行依赖注入，而依赖注入需要IOC容器。

postProcessProperties源码：
```
public PropertyValues postProcessProperties(PropertyValues pvs, Object bean, String beanName) {
    InjectionMetadata metadata = this.findAutowiringMetadata(beanName, bean.getClass(), pvs);

    try {
        metadata.inject(bean, beanName, pvs);
        return pvs;
    } catch (BeanCreationException var6) {
        BeanCreationException ex = var6;
        throw ex;
    } catch (Throwable var7) {
        Throwable ex = var7;
        throw new BeanCreationException(beanName, "Injection of autowired dependencies failed", ex);
    }
}
```

该方法很简单，核心步骤只有两个：
1. findAutowiringMetadata来读取该bean类中有哪些方法和字段被@Autowired标记了。
2. metadata.inject 对被注解的方法和字段进行依赖注入即可。在上一步中已经得到Method或者Field了，那么很容易得到需要注入的bean的类型，根据类型在IOC容器中进行查找即可。

    ```
    // 根据findAutowiringMetadata()方法的返回值可以获得Field
    Field bean3Filed = Bean1.class.getField("bean3");
    // 封装为DependencyDescriptor
    DependencyDescriptor dependencyDescriptor = new DependencyDescriptor(bean3Filed, false);
    // 从IOC容器中查找
    Bean3 o = (Bean3) beanFactory.doResolveDependency(dependencyDescriptor, null,null,null);
    ```


## 常见beanFactory后处理器

```
package com.cain.springtheory.beanFactoryPostProcessor;

import org.mybatis.spring.mapper.MapperScannerConfigurer;
import org.springframework.context.annotation.ConfigurationClassPostProcessor;
import org.springframework.context.support.GenericApplicationContext;

public class ApplicationContext {
    public static void main(String[] args) {
        GenericApplicationContext applicationContext = new GenericApplicationContext();
        applicationContext.registerBean("config", Config.class);
        applicationContext.registerBean(ConfigurationClassPostProcessor.class);     // 负责处理 @ComponentScan @Bean等注解
        applicationContext.registerBean(MapperScannerConfigurer.class, db->{
            db.getPropertyValues().add("basePackage", "com.cain.springtheory.beanFactoryPostProcessor.mappers");
        });         //mybatis提供的，负责扫描mapper并创建bean
        applicationContext.refresh();

        for (String beanDefinitionName : applicationContext.getDefaultListableBeanFactory().getBeanDefinitionNames()) {
            System.out.println(beanDefinitionName);
        }

        applicationContext.close();
    }
}
```

这里有两个常见的beanFactory后处理器：
1. ConfigurationClassPostProcessor：处理@ComponentScan @Bean等注解
2. MapperScannerConfigurer: 根据basepackage处理mapper接口

### @ComponentScan分析

对于ConfigurationClassPostProcessor中对@ComponentScan注解的处理，我们用以下代码进行了模拟：

```
public static void componentScanSimulation() throws IOException {
    GenericApplicationContext context = new GenericApplicationContext();

    // 查看配置类上是否有@ComponentScan
    ComponentScan componentScan = AnnotationUtils.findAnnotation(Config.class, ComponentScan.class);
    // 读取@ComponentScan中的basepackages
    for (String basePackage : componentScan.basePackages()) {
        // 包名转路径名
        String path = "classpath*:"+basePackage.replace(".", "/")+"/*.class";
        // 利用applicationContext来读取resource
        Resource[] resources = context.getResources(path);
        // 利用CachingMetadataReaderFactory来读取resource的内容
        CachingMetadataReaderFactory readerFactory = new CachingMetadataReaderFactory();
        // 判断每个文件（resource）是否包含@Component及其衍生注解
        for (Resource resource : resources) {
            MetadataReader metadataReader = readerFactory.getMetadataReader(resource);
            if( metadataReader.getAnnotationMetadata().hasAnnotation(Component.class.getName()) ||
                    metadataReader.getAnnotationMetadata().hasMetaAnnotation(Component.class.getName())){
                // 添加该类为beanDefinition
                AbstractBeanDefinition beanDefinition = BeanDefinitionBuilder
                        .genericBeanDefinition(metadataReader.getClassMetadata().getClassName())
                        .getBeanDefinition();
                // 为该beanDefinition取名
                AnnotationBeanNameGenerator beanNameGenerator = new AnnotationBeanNameGenerator();
                String beanName = beanNameGenerator.generateBeanName(beanDefinition, context);
                // 将该beanDefinition添加到IOC容器
                context.registerBeanDefinition(beanName, beanDefinition);
            }
        }
    }

    // 查看被添加的beanDefinition
    for (String beanDefinitionName : context.getBeanDefinitionNames()) {
        System.out.println(beanDefinitionName);
    }
}
```

其思路还是很简单的：

1. 读取配置类上的@ComponentScan注解，获取其中的basePackages
2. 根据basePackages中的包名，将其转换为路径名，读取该路径下的class文件
3. 判断class文件是否包含@Component及衍生注解
4. 如何包含，则创建为beanDefinition并添加到IOC容器中

### @Bean分析

```
GenericApplicationContext context = new GenericApplicationContext();

CachingMetadataReaderFactory readerFactory = new CachingMetadataReaderFactory();
// 读取配置类的信息
MetadataReader metadataReader = readerFactory.getMetadataReader("com.cain.springtheory.beanFactoryPostProcessor.Config");
// 获取被Bean注释的方法
for (MethodMetadata annotatedMethod : metadataReader.getAnnotationMetadata().getAnnotatedMethods(Bean.class.getName())) {
    // 将该方法设置为工厂方法并获取beanDefinition
    BeanDefinitionBuilder builder = BeanDefinitionBuilder.genericBeanDefinition();
    builder.setFactoryMethod(annotatedMethod.getMethodName());
    builder.setAutowireMode(AbstractBeanDefinition.AUTOWIRE_CONSTRUCTOR);   //设置bean方法的依赖注入模式
    AbstractBeanDefinition beanDefinition = builder.getBeanDefinition();

    // 将beanDefinition注册到IOC容器
    context.registerBeanDefinition(annotatedMethod.getMethodName(),beanDefinition);
}
```

其思路为：
1. 读取配置类的信息，获取被@Bean注解的方法
2. 将被注解的方法设置为工厂方法并获取beanDefinition（代码是这么做的，具体怎么处理工厂方法来获取bd由BeanDefinitionBuilder来实现）
3. 将方法名作为bean名，将beanDefinition添加到IOC容器即可

### Mapper分析

```
public static void mapperSimulation() throws IOException {
    GenericApplicationContext context = new GenericApplicationContext();

    // 根据basePackage路径读取目录下的class文件，将其读为resource
    PathMatchingResourcePatternResolver patternResolver = new PathMatchingResourcePatternResolver();
    Resource[] resources = patternResolver.getResources("classpath*:com/cain/springtheory/beanFactoryPostProcessor/mappers/*.class");
    CachingMetadataReaderFactory readerFactory = new CachingMetadataReaderFactory();
    for (Resource resource : resources) {
        MetadataReader metadataReader = readerFactory.getMetadataReader(resource);
        ClassMetadata classMetadata = metadataReader.getClassMetadata();
        // 判断每个resource是否为接口
        if (classMetadata.isInterface()) {
            // 为该接口生成MapperFactoryBean
            AbstractBeanDefinition beanDefinition = BeanDefinitionBuilder.genericBeanDefinition(MapperFactoryBean.class)
                    .addConstructorArgValue(classMetadata.getClassName())
                    .setAutowireMode(AbstractBeanDefinition.AUTOWIRE_BY_TYPE)
                    .getBeanDefinition();
            // 生成辅助beanDefinition，只是用来生成bean name
            AbstractBeanDefinition bd2 = BeanDefinitionBuilder.genericBeanDefinition(classMetadata.getClassName()).getBeanDefinition();
            AnnotationBeanNameGenerator beanNameGenerator = new AnnotationBeanNameGenerator();
            String beanName = beanNameGenerator.generateBeanName(bd2, context);

            // 添加bd到IOC容器
            context.registerBeanDefinition(beanName, beanDefinition);
        }
    }
}    
```

其流程为：
1. 根据basepackage查找目录下的class文件并读取
2. 判断该class是不是接口
3. 为接口生成MapperFactoryBean,并根据MapperFactoryBean构造函数和依赖注入的需要配置BeadDefinitionBuilder。

（关于MapperFactoryBean如何工作的，看视频https://www.bilibili.com/video/BV1P44y1N7QG?p=24&vd_source=9cddb128bdf59e171399ffe93da6d348）

## Aware和InitializingBean接口

自己写的bean还可以实现各种Aware接口，这些接口用来提供回调函数。回调函数会在bean被创建时由Spring调用，调用时相关参数也会被传进回调函数。

```
public class Bean1 implements BeanNameAware, BeanFactoryAware, InitializingBean{
    @Override
    public void afterPropertiesSet() throws Exception {
        System.out.println("Initializing Bean");
    }

    @Override
    public void setBeanFactory(BeanFactory beanFactory) throws BeansException {
        System.out.println("Bean1 BeanFactoryAware setBeanFactory "+beanFactory);
    }

    @Override
    public void setBeanName(String s) {
        System.out.println("Bean1 setBeanName: " + s);
    }
}
```

上述是一些常见的回调函数：
1. BeanNameAware:可以获取bean的名字
2. BeanFactoryAware：可以获取beanFactory
3. InitializingBean:可以设置初始化方法

然而，这些功能也可以通过其它注解来实现，例如如果我们想得到BeanFactory，我们可以通过@Autowired注解来获得；对于InitializingBean，也可以使用@PostConstruct注解

```
@Autowired
public void fetchBeanFactory(BeanFactory beanFactory){
    System.out.println("Use @Autowired to get "+beanFactory);
}

@PostConstruct
public void postConstruct(){
    System.out.println("PostConstruct");
}
```

这里Aware和@Autowired等注解的核心区别是：Aware属于内置方法，Spring创建bean时一定会调用；而@Autowired等注解属于拓展方法，需要对应的bean后置处理器来实现，可能失效。

## 后处理器，回调函数等执行流程

首先看一个@Autowired失效的例子：
```
public class A06ApplicationContext {
    public static void main(String[] args) {
        GenericApplicationContext context = new GenericApplicationContext();
//        context.registerBean("myBean1", Bean1.class);
        context.registerBean("config1", Config1.class);
        context.registerBean(AutowiredAnnotationBeanPostProcessor.class);
        context.registerBean(CommonAnnotationBeanPostProcessor.class);
        context.registerBean(ConfigurationClassPostProcessor.class);
        context.refresh();
        context.close();
    }
}

@Configuration
public class Config1 {
    @Autowired
    public void fetchBeanFactory(BeanFactory beanFactory){
        System.out.println("Use @Autowired to get "+beanFactory);
    }

    @PostConstruct
    public void postConstruct(){
        System.out.println("PostConstruct");
    }

    @Bean
    public Bean1 bean1(){
        return new Bean1();
    }

    @Bean
    public BeanFactoryPostProcessor aaa(){
        return (ConfigurableListableBeanFactory beanFactory)->{
            System.out.println("BeanFactoryPostProcessor");
        };
    }
}
```

其中@Autowired和@PostConstruct并未生效。这跟后处理器的执行流程有关，一般情况下执行流程为：

![alt text](pic/后处理器流程1.png)

这里BeanFactoryPostProcessor执行时，如果发现@Bean注解中没有BeanFactory类型的bean，则会等到配置类创建和初始化后再创建@Bean注解的bean。（因为@Bean注解是通过注册为工厂方法来生成bean的，所以需要配置类的实例才能运作。而配置类实例的创建需要bean后处理器，所以如何@Bean注解中没有重要的bean，通常会推迟@bean注解生效）

然而，如果BeanFactoryPostProcessor发现配置类中@Bean注解了返回值为BeanFactory类型的方法，则必须立刻创建配置类以及BeanFactory类型的bean，从而确保BeanFactory在最开始都执行了。此时流程如下：

![alt text](pic/后处理器流程2.png)

而这种提前创建配置类时，bean后处理器还未注册，所以配置类上的@Autowired等注解不生效。解决方法也很简单，将@Autowired等注解替换为Aware接口即可。

## 初始化和摧毁方法

```
@SpringBootApplication
public class A07ApplicationContext {
    public static void main(String[] args) {
        ConfigurableApplicationContext applicationContext = SpringApplication.run(A07ApplicationContext.class, args);
        applicationContext.close();
    }

    @Bean(initMethod = "myInit")
    public Bean1 bean1(){
        return new Bean1();
    }

    @Bean(destroyMethod = "myDestroy")
    public Bean2 bean2(){
        return new Bean2();
    }
}

class Bean1 implements InitializingBean {
    @PostConstruct
    public void init(){
        System.out.println("init 1");
    }

    @Override
    public void afterPropertiesSet() throws Exception {
        System.out.println("init 2");
    }

    public void myInit(){
        System.out.println("init 3");
    }
}

class Bean2 implements DisposableBean {
    @PreDestroy
    public void preDestroy(){
        System.out.println("destroy 1");
    }

    @Override
    public void destroy() throws Exception {
        System.out.println("destroy 2");
    }

    public void myDestroy(){
        System.out.println("destroy 3");
    }
}
```

上述展示了三种初始化和摧毁方法，顺序打印xxx 1,xxx 2, xxx 3。

## Scope种类以及失效情况

Bean的Scope有5种：
1. singleton： 单例
2. prototype： 多例
3. request： 每次请求创建一个实例
4. session： 每次会话创建一个实例，一般一个浏览器算一个会话，不同标签页共享会话
5. application: 一个应用创建一个实例

示例：
```
@Component
@Scope("request")
public class RequestBean {
}

@Component
@Scope("session")
public class SessionBean {
}

@Component
@Scope("application")
public class ApplicationBean {
}

@RestController
public class HelloController {
    @Lazy
    @Autowired
    ApplicationBean applicationBean;

    @Lazy
    @Autowired
    SessionBean sessionBean;

    @Lazy
    @Autowired
    RequestBean requestBean;

    @GetMapping("/hello")
    public String hello() {
        return "hello"+applicationBean+"|"+sessionBean+"|"+requestBean;
    }
}
```
(这里反射使用到了jdk对象的tostring方法，所以要添加jvm参数：--add-opens=java.base/java.lang=ALL-UNNAMED 才能避免报错illegalAccess)

### Scope失效

**多例对象注入单例，会导致多例失效**

```
@ComponentScan("com.cain.springtheory.ScopeStudy")
public class A09ApplicationContext {
    public static void main(String[] args) {
        AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(A09ApplicationContext.class);
        E bean = applicationContext.getBean(E.class);

        // 获取单例bean上的多例属性
        System.out.println(bean.getF1().getClass());
        System.out.println(bean.getF1());
        System.out.println(bean.getF1());

        applicationContext.close();
    }
}

@Component
class E{
    @Autowired
//    @Lazy
    private F1 f1;

    public F1 getF1() {
        return f1;
    }
}

@Component
@Scope("prototype")
//@Scope(value = "prototype", proxyMode = ScopedProxyMode.TARGET_CLASS)
class F1{
}
```

解决方法也很简单：
1. 在多例属性上添加@Lazy注解
2. 在多例的@Scope上设置proxyMode = ScopedProxyMode.TARGET_CLASS

两者的原理都是相同的，即为多例F1创建代理对象进行依赖注入，而每次调用F1的方法时，代理对象每次都会创建一个新的F1对象，实现多例的效果。

3. 使用ObjectFactory，直接依赖注入ObjectFactory，每次使用F1从工厂创建即可
   ```
    @Autowired
    private ObjectFactory<F1> f1Factory;

    public F1 getF1() {
        return f1Factory.getObject();
    }
   ```

4. 使用ApplicationContext
   ```
    @Autowired
    private ApplicationContext applicationContext;

    public F1 getF1() {
        return applicationContext.getBean(F1.class);
    }
   ``` 

## AOP 

通常AOP的实现是通过动态代理的：
```
@SpringBootApplication
public class A09ApplicationContext {
    public static void main(String[] args) {
        ConfigurableApplicationContext applicationContext = SpringApplication.run(A09ApplicationContext.class, args);
        Bean1 bean1 = applicationContext.getBean(Bean1.class);

        bean1.message();
        System.out.println(bean1.getClass());

        applicationContext.close();
    }
}

@Component
public class Bean1 {
    public void message(){
        System.out.println("some message");
    }
}

@Aspect
@Component
public class MyAspect {
    @Before("execution(void com.cain.springtheory.aopStudy.Bean1.message())")
    public void before(){
        System.out.println("Before Method");
    }
}
```

其中Aspect注解的类需要@Component注解来添加为bean。

然而除了动态代理，还要其他实现AOP的方法。

### ajc编译器

通过添加ajc编译器的插件即可实现AOP代理，该编译器会修改目标类（Bean1）的代码，直接添加代理逻辑，无需生成代理对象。

通过该方法，无需将Aspectj类注册为bean，也可以对static方法直接进行增强，解决了代理类无法处理静态方法的问题。

### agent处理

还可以通过jvm指令javaagent来实现AOP。这里也不需要将Aspet对象添加为bean，实现原理也是通过直接修改目标类的class文件。

### Spring代理选择

首先简单回顾一下AOP的几个概念：

* 切点(pointcut)：@Before("execution(void foo())")，指明哪些方法进行增强
* 通知(advice)：代理增强的逻辑
* 切面(aspect)：一组`切点+通知`
* 更细粒度的切面(advisor)：一个`切点+通知`

其中aspect底层由advisor实现。

代理流程为：
```java
import org.aopalliance.intercept.MethodInterceptor;
import org.springframework.aop.aspectj.AspectJExpressionPointcut;
import org.springframework.aop.framework.ProxyFactory;
import org.springframework.aop.support.DefaultPointcutAdvisor;

public class A15 {
    public static void main(String[] args) {
        // 1. 创建切点
        AspectJExpressionPointcut pointcut = new AspectJExpressionPointcut();
        pointcut.setExpression("execution(* foo())");

        // 2. 创建通知 (注意这里的MethodInterceptor是AOP包下的，不是CGLIB的)
        MethodInterceptor advice = invocation -> {
            System.out.println("前置增强");
            Object result = invocation.proceed();
            System.out.println("后置增强");
            return result;
        };

        // 3. 创建切面
        DefaultPointcutAdvisor advisor = new DefaultPointcutAdvisor(pointcut, advice);

        // 4. 创建代理
        ProxyFactory proxyFactory = new ProxyFactory();
        proxyFactory.addAdvisor(advisor);
        proxyFactory.setTarget(new Target1());
        proxyFactory.setInterfaces(Target1.class.getInterfaces());  //setTarget()方法不会自动判断target是否实现了接口，需要该方法来指明
        proxyFactory.setProxyTargetClass(true);     //用来选择代理方式，默认为false

        I1 proxy = (I1) proxyFactory.getProxy();

        // 5. 使用代理
        proxy.foo();
        proxy.bar();
    }
}

interface I1{
    public void foo();

    public void bar();
}

class Target1 implements I1{

    @Override
    public void foo() {
        System.out.println("Target1 foo");
    }

    @Override
    public void bar() {
        System.out.println("Target1 bar");
    }
}
```

其中我们关注的是ProxyFactory什么时候使用JDK代理，什么时候使用CGLIB代理。

ProxyFactory有setProxyTargetClass方法来设置代理类型，默认值为false，对应JDK代理；当设置为true时则使用CGLIB代理。然而JDK必须有接口，如果ProxyTargetClass为false，而没有通过setInterfaces设置接口，则依旧使用CGLIB代理。

总体来说就是三种类型：
1. ProxyTargetClass为false，设置了接口，使用JDK代理
2. ProxyTargetClass为false，没有设置接口，使用CGLIB代理
3. ProxyTargetClass为true，使用CGLIB代理

### 切点匹配逻辑

```java
import org.springframework.aop.aspectj.AspectJExpressionPointcut;
import org.springframework.context.annotation.Lazy;

public class A16 {
    public static void main(String[] args) throws NoSuchMethodException {
        // 1. 根据表达式进行方法匹配
        AspectJExpressionPointcut pointcut = new AspectJExpressionPointcut();
        pointcut.setExpression("execution(* foo())");
        System.out.println(pointcut.matches(MyTarget1.class.getMethod("foo"), MyTarget1.class));
        System.out.println(pointcut.matches(MyTarget1.class.getMethod("bar"), MyTarget1.class));

        // 2. 根据注解进行方法匹配
        AspectJExpressionPointcut pointcut2 = new AspectJExpressionPointcut();
        pointcut2.setExpression("@annotation(org.springframework.context.annotation.Lazy)");
        System.out.println(pointcut2.matches(MyTarget2.class.getMethod("foo"), MyTarget2.class));
        System.out.println(pointcut2.matches(MyTarget2.class.getMethod("bar"), MyTarget2.class));
    }
}

class MyTarget1 {
    public void foo() {
        System.out.println("MyTarget1.foo()");
    }

    public void bar() {
        System.out.println("MyTarget1.bar()");
    }
}

class MyTarget2 {
    public void foo() {
        System.out.println("MyTarget2.foo()");
    }

    @Lazy
    public void bar() {
        System.out.println("MyTarget2.bar()");
    }
}
```

切点匹配的逻辑还是比较简单的，就是遍历所有类上的方法，然后调用match方法来判断该方法是否匹配。match方法则根据表达式和当前方法来判断是否匹配。

在上述的方法增强中，match应该只需要方法信息即可；然而对于一些加在类上的注解，要对它们进行匹配则需要类信息，所以match方法的接口要求提供类信息和方法信息。

### Aspect 和 Advisor

```java
import org.aopalliance.aop.Advice;
import org.aopalliance.intercept.MethodInterceptor;
import org.aspectj.lang.annotation.After;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;
import org.springframework.aop.Advisor;
import org.springframework.aop.aspectj.AspectJExpressionPointcut;
import org.springframework.aop.aspectj.annotation.AnnotationAwareAspectJAutoProxyCreator;
import org.springframework.aop.framework.autoproxy.AbstractAdvisorAutoProxyCreator;
import org.springframework.aop.framework.autoproxy.AbstractAutoProxyCreator;
import org.springframework.aop.support.DefaultPointcutAdvisor;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.ConfigurationClassPostProcessor;
import org.springframework.context.support.GenericApplicationContext;

import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;
import java.util.List;

public class A017Application {

    public static void main(String[] args) throws NoSuchMethodException, InvocationTargetException, IllegalAccessException {
        // 1.创建IOC容器，添加目标类和负责切面的后处理器
        GenericApplicationContext context = new GenericApplicationContext();
        context.registerBean(Target.class);
        context.registerBean(Aspect1.class);
        context.registerBean(Config.class);
        context.registerBean(ConfigurationClassPostProcessor.class);            //处理配置类
        context.registerBean(AnnotationAwareAspectJAutoProxyCreator.class);     //处理切面的后处理器
        context.refresh();

//        for (String name : context.getBeanDefinitionNames()) {
//            System.out.println(name);
//        }

        // 2.测试负责切面的后处理器的核心方法1：检测目标类有几个切面
        AnnotationAwareAspectJAutoProxyCreator creator = context.getBean(AnnotationAwareAspectJAutoProxyCreator.class);
        // 由于AnnotationAwareAspectJAutoProxyCreator中的方法是protected的，所以这里用反射来调用
        Method findEligibleAdvisors = AbstractAdvisorAutoProxyCreator.class.getDeclaredMethod("findEligibleAdvisors", Class.class, String.class);
        findEligibleAdvisors.setAccessible(true);
        // 调用findEligibleAdvisors获取IOC容器中对Target生效的advisor
        List<Advisor> advisors = (List<Advisor>) findEligibleAdvisors.invoke(creator, Target.class, "target");
        // 总共获取到4个advisor，一个spring自动添加的，一个配置类中的advisor，两个从@Aspect中获得的
        for (Advisor advisor : advisors) {
            System.out.println(advisor);
        }

        // 3.测试负责切面的后处理器的核心方法2：为目标类创建代理对象，实现切面逻辑
        Method wrapIfNecessary = AbstractAutoProxyCreator.class.getDeclaredMethod("wrapIfNecessary", Object.class, String.class, Object.class);
        wrapIfNecessary.setAccessible(true);
        Object proxyTarget = wrapIfNecessary.invoke(creator, new Target(), "target", "target");
        System.out.println(proxyTarget.getClass());
        ((Target) proxyTarget).foo();
    }
}

class Target{
    public void foo(){
        System.out.println("Target foo");
    }

    public void bar(){
        System.out.println("Target bar");
    }
}

@Aspect
class Aspect1{
    @Before("execution(* foo())")
    public void before(){
        System.out.println("Aspect1 before");
    }

    @After("execution(* foo())")
    public void after(){
        System.out.println("Aspect1 after");
    }
}

@Configuration
class Config{
    @Bean
    public Advisor advisor2(Advice advice){
        AspectJExpressionPointcut pointcut = new AspectJExpressionPointcut();
        pointcut.setExpression("execution(* foo())");
        return new DefaultPointcutAdvisor(pointcut,advice);
    }

    @Bean
    public Advice advice(){
        MethodInterceptor methodInterceptor = invocation -> {
            System.out.println("Advisor2 before");
            Object result = invocation.proceed();
            System.out.println("Advisor2 after");
            return result;
        };
        return methodInterceptor;
    }
}
```

上述代码测试了advisor和@Aspect的用法，其中展示了一个核心的后处理器：AnnotationAwareAspectJAutoProxyCreator。该后处理器负责处理切面，@Aspect注解的切面也会被转换为多个advisor切面。

该注解有两个核心方法：

1. findEligibleAdvisors：为target类查找作用在该类上的切面
2. wrapIfNecessary：为target类创建代理并且实现切面逻辑

### 代理创建时机

```java
import javax.annotation.PostConstruct;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;
import org.springframework.aop.aspectj.annotation.AnnotationAwareAspectJAutoProxyCreator;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.CommonAnnotationBeanPostProcessor;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.ConfigurationClassPostProcessor;
import org.springframework.context.support.GenericApplicationContext;

public class ProxyCreateTime {
    public static void main(String[] args) {
        GenericApplicationContext context = new GenericApplicationContext();
        context.registerBean(ConfigurationClassPostProcessor.class);            //处理@Config
        context.registerBean(AnnotationAwareAspectJAutoProxyCreator.class);     //处理@Aspect
        context.registerBean(CommonAnnotationBeanPostProcessor.class);          //处理@PostConstructor
        context.registerBean(AutowiredAnnotationBeanPostProcessor.class);       //处理@Autowired
        context.registerBean(MyConfig.class);
        context.registerBean(MyAspect.class);
        context.refresh();
    }

    @Aspect
    public static class MyAspect {
        @Before("execution(* foo())")
        public void before() {
            System.out.println("MyAspect Before execution");
        }
    }

    @Configuration
    static class MyConfig{
        @Bean
        public Bean1 bean1(){
            return new Bean1();
        }

        @Bean
        public Bean2 bean2(){
            return new Bean2();
        }
    }

    static class Bean1{
        public void foo(){
            System.out.println("Bean1 Foo");
        }

        public Bean1(){
            System.out.println("Bean1 Constructor");
        }

        // 这里就构成了循环依赖，产生不同的代理时机
//        @Autowired
//        public void setBean2(Bean2 bean2){
//            System.out.println("Bean1 setBean2 as" + bean2.getClass());
//        }

        @PostConstruct
        public void postConstruct(){
            System.out.println("Bean1 init");
        }
    }

    static class Bean2{
        public Bean1 bean1;

        public Bean2(){
            System.out.println("Bean2 Constructor");
        }

        @Autowired
        public void setBean1(Bean1 bean1){
            this.bean1 = bean1;
            System.out.println("Bean2 setBean1 as "+bean1.getClass());
        }

        @PostConstruct
        public void postConstruct(){
            System.out.println("Bean2 init");
        }
    }
}
```

在bean的生命周期中，创建代理对象的时机有两个：构造->(代理)依赖注入->初始化(代理)

1. 依赖注入前：如果出现循环依赖则在此时创建代理
2. 初始化后：正常情况在此时创建代理

这一块在循环依赖会有详细说明。

### 切面执行顺序

一个目标类可以添加多个切面，而切面的执行顺序也是可以通过@Order注解控制的，order值越小越先执行。

不过@Order注解对于方法是不生效的，如果切面类实现了Order相关的接口，也可以通过setOrder方法来设置order。

### @Aspect解析为Advisor

```java
public class AspectToAdvisor {
    public static void main(String[] args) {
        // 1.查找Aspect注解的类
        if(MyAspect.class.isAnnotationPresent(Aspect.class)){
            // 2.查找@Befor注解的方法
            for (Method method : MyAspect.class.getDeclaredMethods()) {
                if(method.isAnnotationPresent(Before.class)){
                    // 3. 根据@Before中的值创建切点
                    String expression = method.getAnnotation(Before.class).value();
                    AspectJExpressionPointcut pointcut = new AspectJExpressionPointcut();
                    pointcut.setExpression(expression);

                    // 4. 根据@Befor选择通知类型，根据Method创建通知
                    SingletonAspectInstanceFactory factory =
                            new SingletonAspectInstanceFactory(new MyAspect());    // method的调用需要实例
                    AspectJMethodBeforeAdvice advice = new AspectJMethodBeforeAdvice(method, pointcut, factory);

                    // 5. 创建切面
                    DefaultPointcutAdvisor advisor = new DefaultPointcutAdvisor(pointcut, advice);
                    System.out.println(advisor);
                }
            }
        }
    }
}

@Aspect
class MyAspect{
    @Before("execution(* foo())")
    public void before(){
        System.out.println("Before Aspect");
    }
}
```

上述是@Before注解的切面解析为Advisor的大体思路，注解足够多，不再赘述



