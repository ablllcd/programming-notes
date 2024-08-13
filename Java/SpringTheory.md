## 大概思路

1. 例如Spring通过new AnnotationConfigApplicationContext(Config.class)作为IOC容器。那么所有的起点就是ApplicationContext的构造方法。

### 构造方法

总体来说，构造方法里干两件事：扫描bean class并创建beanDefinition；创建单例bean.

1. 在构造方法中，首先ApplicationContext(实际上应该是BeanFactory)通过反射读取配置类中的@ComponentScan的值，从而判断哪些类文件需要进行检查。然后将被@Component注解标识的类筛选出来
2. 找到bean class后通过类加载器将这些类加载，为其构造beanDefinition并存储到beanDefinition池中（Map）。beanDefinition至少存储该bean的scope和type。此外，当判断出该bean class为单例时，创建该bean的实例并且存储到单例池中（Map）。

### getBean方法

IOC容器的核心方法就是获取bean。当构造方法结束，所有的bean都有对应的beanDefinition，并且单例bean已经生成了实例。

那当getBean方法被调用时，有两种情况：

1. 单例则从单例池中返回
2. 多例则创建bean并且返回

