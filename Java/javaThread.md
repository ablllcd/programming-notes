## 线程创建

### 继承Thread类
为了创建线程，我们可以创建一个类来基础java.lang.Thread类，并重写run()方法来指定线程要执行的任务。

````
public class PrintThread extends Thread{
    @Override
    public void run(){
        for (int i = 0; i < 5; i++) {
            System.out.println("Print thread run:"+i);
        }
    }
}
````

之后在main方法中，只需要创建该类并执行start()方法即可。（注意不是run方法）
````
public static void main(String[] args) throws Exception {
    Thread t1 = new PrintThread();
    t1.start();

    for (int i = 0; i < 5; i++) {
        System.out.println("main run:"+i);
    }
}
````

缺点：继承了Thread类就无法继承其它类了。

### 实现Runnable接口
另一种方法是创建一个类实现java.lang.Runnable接口，从而成为一个任务类。在该类中重载Runnable接口的run方法，并在其中指定线程要做的事情。
````
public class Thread2 implements Runnable{
    @Override
    public void run(){
        for (int i = 0; i < 5; i++) {
            System.out.println("Thread2: "+i);
        }
    }
}
````

在main函数中，则可以创建任务类并且用其来构造Thread对象。
````
public static void main(String[] args) throws Exception {
    Thread2 task = new Thread2();
    Thread t2 = new Thread(task);
    t2.start();

    for (int i = 0; i < 5; i++) {
        System.out.println("main run:"+i);
    }
}
````

要注意的是：任务类并不是线程类，只是构造线程类的参数。

### 实现Callable接口
前两种方法都是无返回值的，如果需要线程的返回值，则可以使用这种方法。

首先，创造一个类实现java.util.concurrent.Callable接口，重载它的call方法并在其中指定线程要执行的任务和返回值。</br>注意，Callable的泛型等同于call函数返回值的类型。

````
public class Thread3 implements Callable<String>{
    private int n;
    public Thread3(int n){
        this.n = n;
    }
    @Override
    public String call(){
        int sum = 0;
        for (int i = 0; i < n; i++) {
            System.out.println("Thread3: "+i);
            sum += i;
        }
        return "Sum of 1 to "+n+" is "+sum;
    }
}
````

然后在main函数中，将实现callable的类封装成FutureWrok类。这个FutureWork类实现了Runnable接口，所有将它交给Thread类来构建线程。

返回值则是用FutureWork的get方法来获得。注意，该方法为了获取线程的返回值，它可能会阻塞main线程来等待Futurework的线程执行完。
````
public static void main(String[] args) throws Exception {
    Callable<String> call = new Thread3(5);
    FutureTask<String> f1 = new FutureTask<>(call);
    Thread t = new Thread(f1);
    t.start();

    String rs = f1.get();
    System.out.println(rs);
}
````

## Thread类常见方法
````
public static void main(String[] args) throws Exception {
    // 1. getName 获取线程的名字
    // 2. setName(String) 为线程设置名字
    Thread t1 = new Thread(()->{
        System.out.println("Thread is running");
    });
    t1.start();
    System.out.println(t1.getName());
    t1.setName("First thread");
    System.out.println(t1.getName());

    // 3. currentThread() 获取当前程序的线程
    Thread t2 = new Thread(()->{
        Thread cur = Thread.currentThread();
        cur.setName("Second Thread");
        String name = cur.getName();
        System.out.println(name+" is running");
    });
    t2.start();

    // 4. Thread(Stirng name) 在构造函数中传递线程名字
    Thread t3 = new Thread(()->{
        System.out.println("Thread is running");
    }, "Third Thread");
    System.out.println(t3.getName());

    // 5. sleep() 休眠
    Thread.sleep(5000);
    System.out.println("Wake up!");

    // 6. join() 等待该线程结束
    Thread t4 = new Thread(()->{
        for (int i = 0; i < 5; i++) {
            System.out.println("Thread is running: "+i);        
        }
    });
    t4.start();
    t4.join();
    for (int i = 0; i < 5; i++) {
        System.out.println("main:"+i);
    }
}
````

## 线程安全问题
多个线程同时访问和更改资源，可能出现问题。

解决方法：通过加锁来保证线程同步，也就是确保线程会有序地来访问资源。

### 同步代码块
语法： 
````
synchronized(同步锁){
    操作资源代码块
}
````
例子：

资源类
````
public class Account {
    private String name;
    private int money;
    public Account(String name, int money) {
        this.name = name;
        this.money = money;
    }

    public void fetch(int money) throws InterruptedException{
        synchronized(this){
            if(money > this.money){
            System.out.println(name+" has no enough balance");
            }else{
                System.out.println(name+" fetched successfully");
                Thread.sleep(1000);
                this.money -= money;
                System.out.println(name+" remains "+this.money);
            }
        }
    } 
}
````
线程类
````
public class myThread extends Thread{
    private Account account;
    public myThread(Account acc){
        account = acc;
    }
    @Override
    public void run(){
        try {
            account.fetch(10000);
        } catch (InterruptedException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
    }
}
````
主函数
````
public static void main(String[] args) throws Exception {
    Account acc1 = new Account("HKBC-100", 10000);
    myThread t1 = new myThread(acc1);
    myThread t2 = new myThread(acc1);
    t1.start();t2.start();
    Account acc2 = new Account("HKBC-101", 10000);
    myThread t3 = new myThread(acc2);
    myThread t4 = new myThread(acc2);
    t3.start();t4.start();
}
````

注意： 同步锁可以为任意对象，例如字符串变量，常量等。为了实现锁功能，该对象应为唯一变量。但多个资源理论上应该有多个锁，所以建议资源类的实例方法中使用`this`作为锁，静态方法中使用`类名.class`作为锁。

### 同步方法
语法： 在需要同步的方法前加synchronized关键字。这种方法其实也是有隐含锁的，实例方法默认this为锁，静态方法默认类名.class为锁。

````
public synchronized void fetch(int money) throws InterruptedException{
    if(money > this.money){
    System.out.println(name+" has no enough balance");
    }else{
        System.out.println(name+" fetched successfully");
        Thread.sleep(1000);
        this.money -= money;
        System.out.println(name+" remains "+this.money);
    }
}
````

### Lock锁
程序员自己创建锁，并且自己实现加锁和解锁功能。

我们可以使用ReentrantLock类来创建锁对象，该类实现了Lock接口。通常每个实现类都有一个ReentrantLock对象作为成员变量，来确保每个类都有自己的锁。

````
public class Account {
    private String name;
    private int money;
    private final Lock lk = new ReentrantLock();
    
    public Account(String name, int money) {
        this.name = name;
        this.money = money;
    }

    public void fetch(int money) throws InterruptedException{
        lk.lock();
        if(money > this.money){
            System.out.println(name+" has no enough balance");
        }else{
            System.out.println(name+" fetched successfully");
            Thread.sleep(1000);
            this.money -= money;
            System.out.println(name+" remains "+this.money);
        }
        lk.unlock();
    } 
}
````

注意：程序有可能出现异常，而为了保证锁能够被释放，常常把被加锁的代码放到try-catch中，然后将释放锁的代码放入finally中。

## 线程通信
在某些场景下，我们可能不希望简单的竞争条件，而是线程相互协调，这就需要进行线程通信。

在Object类中有wait()和notify()/notifyAll()方法来帮助实现线程通信。不过只有线程锁调用wait()和notify()方法才有意义，因为只有它知道有哪些线程。

wait()方法会让当前线程睡眠并释放锁，等待被唤醒。
notify()会唤醒睡眠的线程

例子：厨师生产包子，消费者吃包子，但是两者是通过通信来相互协调。

包子类
````
public class Hamburg {
    private List<String> hams = new LinkedList<>();

    public synchronized void make() {
        try {
            String name = Thread.currentThread().getName();
            while (true) {
                if (hams.size() == 0) {
                    System.out.println(name + " make one hamburg!");
                    Thread.sleep(1000);
                    hams.add("one");
                    this.notifyAll();
                    this.wait();
                } else {
                    this.notifyAll();
                    this.wait();
                }
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    public synchronized void eat() {
        try {
            String name = Thread.currentThread().getName();
            while (true) {
                if (hams.size() > 0) {
                    System.out.println(name + " eat one ham!");
                    hams.clear();
                    this.notifyAll();
                    this.wait();
                } else {
                    this.notifyAll();
                    this.wait();
                }
            }
        } catch (Exception e) {
            // TODO: handle exception
        }
    }
}
````

主函数
````
public static void main(String[] args) throws Exception {
    Hamburg hams = new Hamburg();

    new Thread(() -> {
        hams.make();
    }, "cooker1").start();

    new Thread(() -> {
        hams.make();
    }, "cooker2").start();

    new Thread(() -> {
        hams.eat();
    }, "consumer1").start();

    new Thread(() -> {
        hams.eat();
    }, "consumer2").start();
}
````

## 线程池
旨在复用线程，节省计算机资源

线程池接口：ExecutorService；
</br>常用实现类：ThreadPoolExecutor。

### ThreadPoolExecutro

参考连接：https://docs.oracle.com/javase/8/docs/api/

````
ExecutorService pool = new ThreadPoolExecutor(
            3, 5, 1, TimeUnit.MINUTES,
            new ArrayBlockingQueue<>(4),
            Executors.defaultThreadFactory(), 
            new ThreadPoolExecutor.DiscardPolicy());

// 处理Runnable任务
Runnable task1 = new MyRunnable();
pool.execute(task1);    // core thread
pool.execute(task1);    // core thread
pool.execute(task1);    // queue
pool.execute(task1);    // queue
pool.execute(task1);    // queue
pool.execute(task1);    // tempory thread
pool.execute(task1);    // tempory thread
pool.execute(task1);    // discard

// 处理Callable任务
MyCallable task2 = new MyCallable(10);
Future<Integer> res = pool.submit(task2);
System.out.println(res.get());

// 结束线程池
pool.shutdown();    

pool.shutdown();    
````

### Executors
底层通过ThreadPoolExecutor实现
````
ExecutorService pool2 = Executors.newFixedThreadPool(3);
````
