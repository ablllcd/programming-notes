## 可变参数
````
public static void main(String[] args) {
    test(1,2,3);
}

public static void test(int ...args) {
    System.out.println(Arrays.toString(args));
}
````
可变参数本质上是一个`数组`,其声明方式是`类型...参数名`。

注意：1. 一个形参列表中只能有一个可变参数。 2. 可变参数只能放在形参列表最后

## Lambda表达式
用于简化`匿名内部类`的写法

格式为：(匿名类中`被重写的方法`的参数)->{方法体}

前提：只能用于`函数式接口`的匿名内部类，函数式接口指的是只包含一个抽象方法的接口。可选注解：@FunctionalInterface

简化规则：1. 参数类型可以忽略 2. 参数只有一个时，"()"也可以省略 3. 如果方法体只有一行代码，"{}"可以省略,同时也必须要省略"return"和";"。

````
@FunctionalInterface
interface Swimming{
    void swim();
}
public static void main(String[] args){
    Swimming s = ()-> System.out.println("the swimming func");
    s.swim();
}
````

## 方法引用
用于进一步简化lambda表达式。

使用场景：lambda表达式中，函数体只调用一次其它方法。而且lambda表达式的参数与其它方法的参数匹配。则可使用方法引用。

语法：`(p1,p2)->{其它方法(p1,p2)} 可以简化为"其它方法的类::其它方法"`。其中"其它方法"可以为静态方法或者实例方法，甚至可以为"p1.其它方法(p2)"，不过这默认第一个参数为主调参数。也可以为构造器方法，不过语法改为"类名::new"。

例子：
````
@FunctionalInterface
interface Swimming{
    void swim();
}
public static void main(String[] args){
    Swimming s = System.out::println;
    s.swim("call swim func");
}
````

## Classpath
classpath只是声明去哪里找包，还需要把要用到jar包下载到classpath下。要注意的是jar包的包名**不等于**import的包名，import的包名是jar包的文件结构。

如果是引用当前包下的类不需要`import`，如果是引用其它包下的类，则需要使用`import`关键字。


## Reflection 反射
反射是用来获取类和操作类的，Java也提供了`Class`关键字来引用类。

其应用是可以从更高的层次上操作类，增强泛用性
### 获取类
    public static void main(String[] args) throws ClassNotFoundException{
        // 1. 类名.class
        Class s1 = Student.class;
        System.out.println(s1.getName());
        System.out.println(s1.getSimpleName());

        // 2. Class.forName()
        Class s2 = Class.forName("com.cain.Student");
        System.out.println(s2.getName());

        // 3. 对象.getClass()
        Student cain = new Student();
        Class s3 = cain.getClass();
        System.out.println(s3.getName());
    }


## 代理

代理模式是常用的设计模式之一，其特征是代理类与被代理类有相同的接口，代理类可以为被代理类方法执行进行前置后置处理，增强被代理类方法

### 动态代理

没完全明白，待修改

动态代理主要依赖java.lang.reflect.Proxy类，它有一个方法可以动态创建代理类。动态代理的意思是指该类是在运行时创建的，没有预生成的class文件。

1. 首先创建接口和目标类

```
public interface ICalculator {
    public int add(int a,int b);
    public int delete(int a,int b);
}
```

```
public class Calculator implements ICalculator{
    @Override
    public int add(int a, int b) {
        return a+b;
    }

    @Override
    public int delete(int a, int b) {
        return a-b;
    }
}
```

2. 根据Proxy.newProxyInstance创建要求的参数：

    ClassLoader 和 interfaces 跟反射有关，不是很明白具体作用，大概是拿到接口类，知道接口有哪些，从而创建对应的实现类

    InvocationHandler是重要的类，对目标类的增强就是在这里实现的

    大概流程为：

    ![Alt text](pic/dynamicProxy.png)

```
public class LoggerProxyFactory {
    private Object target;
    public LoggerProxyFactory(Object target){
        this.target = target;
    }

    public Object getProxyInstance(){
        ClassLoader loader = target.getClass().getClassLoader();
        Class[] interfaces = target.getClass().getInterfaces();
        InvocationHandler invocationHandler = new InvocationHandler(){
            @Override
            public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                System.out.println(method.getName()+" is called");
                Object result = method.invoke(target,args);
                return result;
            }
        };
        return Proxy.newProxyInstance(loader,interfaces,invocationHandler);
    }
}
```

3. 创建代理类并使用

```
@Test
public void dynamicProxy(){
    Calculator c = new Calculator();
    LoggerProxyFactory proxyFactory = new LoggerProxyFactory(c);
    ICalculator cProxy = (ICalculator) proxyFactory.getProxyInstance();
    System.out.println(cProxy.add(3,4));
}
```

注意：动态代理类的本质是使用Proxy.newProxyInstance来创建，而代理类增加的动作在InvocationHandler里实现。而上述中的LoggerProxyFactory是一种工厂思想，可有可无。