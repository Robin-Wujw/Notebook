# Java高级

## 多线程

### 多线程的创建：

#### 方式一：继承于Thread类

* 创建一个继承于Thread类的子类
* 重写Thread类中的run() ---->将此线程执行的操作声明在run()中
* 创建Thread类的子类对象
* 通过此对象调用start()

```java
//例子：遍历一下100以内的所有的偶数
//1.创建一个继承于Thread类的子类
class MyThread extends Thread{
    //2.重写run()
    @Override
    public void run(){
        for(int i=0;i<100;i++){
            if(i%2==0){
                System.out.println(Thread.currentThread().getName() + ":" + i);
            }
        }
    }
}
public class ThreadTest {
    public static void main(String[] args) {
        //3.创建Thread类的子类对象
        MyThread t1 = new MyThread();
        //通过此对象调用Start():①启动当前线程②调用当前线程的run()
        t1.start();
        //问题一：我们不能通过直接调用run()的方式启动线程。
        //t1.run();
        //问题二：再启动一个线程，遍历100以内的偶数。不可以还让已经start()的线程去执行。会报IllegalThread
        //t1.start();
        //我们需要重新创建一个线程的对象
        MyThread t2 = new MyThread();
        t2.start();
        //如下操作仍然是在main线程中执行
        for(int i=0;i<100;i++){
            if(i%2==0){
                System.out.println(Thread.currentThread().getName() + ":" + i+"************main*****************");
            }
        }
    }
}
```

#### 方式二：实现Runnable接口

* 创建一个实现了Runnable接口的类
* 实现类去实现Runnable中的抽象方法:run()
* 创建实现类的对象
* 将此对象作为参数传递到Thread类的构造器中，创建Thread类的对象
* 通过Thread类的对象调用start()

```java
//1.创建一个实现Runnable接口的类
class MThread implements Runnable{
    // 2.实现类去实现Runnable中的抽象方法:run()
    @Override
    public void run() {
       for(int i =0; i<100;i++){
           if(i % 2 ==0){
               System.out.println(Thread.currentThread().getName()+":"+i);
           }
       }
    }
}
public class ThreadTest1 {
    public static void main(String[] args) {
        //3.创建实现类的对象
        MThread mThread = new MThread();
        //4.将此对象作为参数传递到Thread类的构造器中，创建Thread类的对象
        Thread t1 = new Thread(mThread);
        //5.通过Thread类的对象调用start():①启动线程②调用当前线程的run()-->调用了Runnable类型
        //的target的run()
        t1.setName("线程一：");
        t1.start();
        //再启动一个线程，遍历100以内的偶数
        Thread t2 = new Thread(mThread);
        t2.setName("线程二：");
        t2.start();
    }
}
```

#### 比较创建线程的两种方式：

* 开发中：优先选择：实现Runnable接口的方式
* 原因：
  * 实现的方式没有类的单继承性的局限性
  * 实现的方式更适合来处理多个线程有共享数据的情况。
* 联系： public class Thread implements Runnable
* 相同点： 两种方式都要重写run(),将线程要执行的逻辑声明在run()中。

#### 测试Thread中的常用方法：

* start():启动当前线程；调用当前线程的run()

* run():通常需要重写Thread类中的此方法，将创建的线程要执行的操作声明在此方法中

* currentThread():静态方法，返回执行当前代码的线程

* getName():获取当前线程的名字

* setName():设置当前线程的名字

* yield():释放当前cpu的执行权

* join():在线程a中调用线程b的join(),此时线程a就进入阻塞状态，直到线程b完全执行完以后，线程a才结束阻塞状态。

* stop():已过时。当执行此方法时，强制结束当前线程。

* sleep(long milli time): 让当前线程"睡眠"指定的millitime毫秒。在指定的millitime毫秒时间内，当前线程是阻塞状态。

* isAlive():判断当前线程是否存活。

* 线程的优先级：

  * MAX_PRIORITY: 10  MIN_PRIORITY: 1

  ​       NORM_PRIORITY: 5 ---->默认的优先级

  * 如何获取和设置当前线程的优先级：
    * getPriority():获取线程的优先级
    * setPriority(int p):设置线程的优先级
  * 说明：高优先级的线程要抢占低优先级线程cpu的执行权。但是只是从概率上讲，高优先级的线程高概率的情况下被执行。并不意味着只有高优先级的线程执行完以后低优先级的线程才执行。

* 线程通信: wait() / notify() / notifyAll():此三个方法定义在Object类中

* 线程的分类： 守护线程 用户线程

#### 方式三： 实现Callable接口。 ---JDK 5.0 新增
* 如何理解实现Callable接口的方式创建多线程比实现Runnable接口创建多线程方式强大？
  * call()可以有返回值的。
  * call()可以抛出异常，被外面的操作捕获，获取异常的信息
  * Callable()是支持泛型的

```java
//1.创建一个实现Callable接口的实现类
class NumThread implements Callable{
    //2.实现call方法，将此线程需要执行的操作声明在call()中
    @Override
    public Object call() throws Exception {
        int sum = 0;
        for (int i = 1; i <= 100; i++) {
            if(i%2 ==0){
                System.out.println(i);
                sum += i;
            }
        }
        return sum; 
    }
}
public class ThreadNew {
    public static void main(String[] args) {
        //3.创建Callable接口实现类的对象
        NumThread numThread = new NumThread();
        //4.将此Callable接口实现类的对象作为参数传递到FutureTask构造器中，创建FutureTask的对象
        FutureTask futureTask = new FutureTask(numThread);
        //5.将FutureTask的对象作为参数传递到Thread类的构造器中，创建Thread对象，并调用start()
        new Thread(futureTask).start();
        try {
            //6.获取Callable中call方法的返回值
            //get()返回值即为FutureTask构造器参数Callable实现类重写的call()的返回值。
            Object sum = futureTask.get();
            System.out.println("总和为：" + sum);
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }
    }
}
```

#### 创建线程的方式四：使用线程池
* 好处：
  * 提高响应速度(减少了创建新线程的时间)
  * 降低资源消耗(重复利用线程池中线程，不需要每次都创建)
  * 便于线程管理
    * corePoolSize: 核心池的大小
    * maximumPoolSize: 最大线程数
    * keepAliveTime: 线程没有任务时最多保持多长时间后会终止
* 面试题：创建多线程有几种方式： 四种

### 线程安全问题

* 例子：创建三个窗口卖票，总票数100张，使用实现Runnable接口的方式：
  * 问题：卖票过程中，出现了重票、错票 --->出现了线程的安全问题
  * 问题出现的原因：当某个线程操作车票的过程中，尚未操作完成时，其他线程参与进来也操作车票。
  * 如何解决：当一个线程在操作ticket的时候，其他线程不能参与进来。直到线程a操作完ticket时，其他线程才可以操作ticket。这种情况即使线程a出现了阻塞，也不能被改变。

在Java中 我们通过同步机制来解决线程安全问题

#### 同步机制：

##### 方式一： 同步代码块

```java
synchronized(同步监视器){
 //需要被同步的代码
}
```

* 说明：

  *  操作共享数据的代码，即为需要被同步的代码 -->不能包含代码多了，也不能包含代码少了。

  * 共享数据：多个线程共同操作的变量。比如：ticket就是共享数据。

  * 同步监视器，俗称：锁。任何一个类的对象，都可以充当锁。

    要求： 多个线程必须要共用同一把锁。

    补充：在实现Runnable接口创建多线程的方式中，我们可以考虑使用this充当同步监视器。

  * 在继承Thread类创建多线程的方式中，慎用this充当同步监视器，考虑使用当前类充当同步监视器

```java
/**
 *  使用同步代码块解决继承Thread类的线程安全问题
 * 例子：创建三个窗口卖票，总票数为100张，使用继承Thread类的方式
 *
 * 说明：在继承Thread类创建多线程的方式中，慎用this充当同步监视器，
 * 考虑使用当前类充当同步监视器
 * @Author: Robin_Wujw
 * @Date: 2022-03-28 22:05
 */
class Window2 extends Thread{

    private static int ticket = 100;
    private static Object obj = new Object();
    @Override
    public void run() {
        while(true){
            //正确的
            //synchronized (obj){
            synchronized (Window2.class){//Class clazz = java.Window2.class
            //java.Window2.class只会加载一次
            //错误的: this代表着t1,t2,t3三个对象
            //synchronized (this){
            if(ticket > 0){
                try {
                    Thread.sleep(100);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(getName() + ":卖票，票号为：" + ticket);
                ticket --;
            }else{
                break;
            }
        }
        }
    }
}
public class WindowTest2 {
    public static void main(String[] args) {
        Window2 t1 = new Window2();
        Window2 t2 = new Window2();
        Window2 t3 = new Window2();

        t1.setName("窗口一");
        t2.setName("窗口二");
        t3.setName("窗口三");

        t1.start();
        t2.start();
        t3.start();

    }
}
```

##### 方式二： 同步方法

* 如果操作共享数据的代码完整的声明在一个方法中，我们不妨将此方法声明同步的。

```java
* 使用同步方法解决实现Runnable接口的线程安全问题
 *  关于同步方法的总结：
 *  1.同步方法仍然涉及到同步监视器，只是不需要我们显式的声明。
 *  2.非静态的同步方法，同步监视器是：this
 *    静态的同步方法，同步监视器是：当前类本身
 */
class Window3 implements Runnable{
    private int ticket = 100;
    Object obj = new Object();
    @Override
    public void run() {
        while(true){
            show();
        }
    }
    private synchronized void show(){
       // synchronized (this) {
            if (ticket > 0) {
                try {
                    Thread.sleep(100); //阻塞
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread().getName() + ":卖票，票号为：" + ticket);
                ticket--;
            }
        }
        }
  //  }

public class WindowTest3 {
    public static void main(String[] args) {
        Window3 w = new Window3();

        Thread t1 = new Thread(w);
        Thread t2 = new Thread(w);
        Thread t3 = new Thread(w);

        t1.setName("窗口1");
        t2.setName("窗口2");
        t3.setName("窗口3");

        t1.start();
        t2.start();
        t3.start();
    }
}
```

```java
* 使用同步方法来处理继承Thread类的方式中的线程安全问题
 */
class Window4<ticket> extends Thread{

    private static int ticket = 100;
    private static Object obj = new Object();
    @Override
    public void run() {
        while(true){
            show();
        }
    }
    private static synchronized void show(){//同步监视器：java.Window4
    //private synchronized void show(){// 同步监视器： t1,t2,t3。此种解决方式是错误的
        if(ticket > 0){
            try {
                Thread.sleep(100);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName() + ":卖票，票号为：" + ticket);
            ticket --;
        }
    }
}
public class WindowTest4 {
    public static void main(String[] args) {
        Window4 t1 = new Window4();
        Window4 t2 = new Window4();
        Window4 t3 = new Window4();

        t1.setName("窗口一");
        t2.setName("窗口二");
        t3.setName("窗口三");

        t1.start();
        t2.start();
        t3.start();

    }
}
```

同步的方式，解决了线程的安全问题。 ---好处

操作同步代码时，只能有一个线程参与，其他线程等待，相当于是一个单线程的过程，效率低。 ---局限性

##### 方式三： Lock锁 -----JDK5.0新增

* 面试题：synchronized 与 lock的异同？

  * 相同：二者都可以解决线程安全问题
  * 不同：synchronized机制在执行完相应的同步代码以后，自动的释放同步监视器

    Lock需要手动的启动同步(Lock()),同时结束同步也需要手动的实现(unlock())

* 优先使用顺序：

  Lock -> 同步代码块(已经进入了方法体，分配了相应资源) ->同步方法

* 面试题：如何解决线程安全问题？有几种方式

```java
class Window implements Runnable{
    private int ticket = 100;
    //1.实例化ReetrantLock
    private ReentrantLock lock = new ReentrantLock();
    @Override
    public void run() {
        while(true){
            try {
                //2.调用锁定方法：lock()
                lock.lock();
                if(ticket > 0){
                    try {
                        Thread.sleep(100);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    System.out.println(Thread.currentThread().getName() + ":售票，票号为：" + ticket);
                    ticket --;
                }else{
                    break;
                }
            }finally {
                //3.调用解锁方法: unlock()
                lock.unlock();
            }

        }
    }
}
public class LockTest {
    public static void main(String[] args) {
        Window w = new Window();
        Thread t1 = new Thread(w);
        Thread t2 = new Thread(w);
        Thread t3 = new Thread(w);

        t1.setName("窗口1");
        t2.setName("窗口2");
        t3.setName("窗口3");
        t1.start();
        t2.start();
        t3.start();
    }
}
```

##### 使用同步机制将单例模式中的懒汉式改写为线程安全的

```java
/**
 * 使用同步机制将单例模式中的懒汉式改写为线程安全的
 * @Author: Robin_Wujw
 * @Date: 2022-03-30 13:43
   */

public class BankTest {
    public static void main(String[]args){
        Bank bank = Bank.getInstance();
    }
}

class Bank{

    private Bank(){}
    
    private static Bank instance = null;
    
    public static  Bank getInstance(){
        //方式一：效率稍差

//        synchronized (Bank.class) {
//            if(instance == null){
//                instance = new Bank();
//            }
//            return instance;
//        }
        //方式二：效率更高
        if(instance == null) {
            synchronized (Bank.class) {
            if(instance == null){
                instance = new Bank();
                }
            }
        }
        return instance;
        }
}
```

#### 死锁

演示线程的死锁问题

 * 死锁的理解：不同的线程分别占用对方需要的同步资源不放弃，都在等待对方放弃自己需要的同步资源，就形成了线程的死锁
 * 说明：
    * 出现死锁后，不会出现异常，不会出现提示，只是所有的线程都处于阻塞状态，无法继续。
    * 我们使用同步时，要避免出现死锁。

```java
/**
 * @Author: Robin_Wujw
 * @Date: 2022-04-01 14:06
 */
public class ThreadTest {
    public static void main(String[] args) {
        StringBuffer s1 = new StringBuffer();
        StringBuffer s2 = new StringBuffer();
        new Thread(){
            @Override
            public void run() {
                synchronized (s1){
                    s1.append("a");
                    s2.append("1");

                    try {
                        Thread.sleep(100);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    synchronized (s2){
                        s1.append("b");
                        s2.append("2");
                        System.out.println(s1);
                        System.out.println(s2);
                    }
                }

            }
        }.start();
        new Thread(new Runnable() {
            @Override
            public void run() {
                synchronized (s2){
                    s1.append("c");
                    s2.append("3");
                    try {
                        Thread.sleep(100);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    synchronized (s1){
                        s1.append("d");
                        s2.append("4");
                        System.out.println(s1);
                        System.out.println(s2);
                    }
                }
            }
        }).start();
    }
}
```

### 线程的通信

* 涉及到的三个方法：
  * wait():一旦执行此方法，当前线程就进入阻塞状态，并释放同步监视器。
  * notify():一旦执行此方法，就会唤醒被wait的一个线程。如果有多个线程被wait，就唤醒优先级高的那个
  * notifyall():一旦执行此方法，就会唤醒所有被wait的线程。
* 说明：
  * wait(),notify(),notifyall()三个方法必须使用在同步代码块或同步方法中。
  * wait(),notify(),notifyall()三个方法的调用者必须是同步代码块或同步方法中的同步监视器。否则会出现IllegalMonitorStateException异常
  * wait(),notify(),notifyall()三个方法定义在java.lang.Object类中
* 面试题：sleep() 和 wait()的异同？
  * 相同点：一旦执行方法，都可以使得当前的线程进入阻塞状态。
  * 不同点：
    * 两个方法声明的位置不同：Thread类中声明sleep(),Object类中声明wait()
    * 调用的要求不同：sleep()可以在任何需要的场景下调用。wait()必须使用在同步代码块/同步方法中调用
    * 关于是否释放同步监视器：如果两个方法都使用在同步代码块或同步方法中，sleep()不会释放锁，而wait()会