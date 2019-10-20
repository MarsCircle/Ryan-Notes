# Java并发编程笔记(一）

------------------------------------------

## 什么是线程?

**线程**是进程中的一个实体，线程本身是不会独立存在的。

那么问题又来了,啥是进程?

《Java并发编程之美》里说的是，**进程**是代码在数据集合上的一次运行活动，**是系统进行资源分配的基本单位**，线程是进程的一个执行路径，一个**进程中至少有一个线程**，进程中的**多个线程共享进程的资源**。

操作系统在分配资源时是把资源分配给进程的，但是CPU资源比较特殊，它是被分配到线程的，所以说**线程也是CPU分配的基本单位**。main函数所在线程也称为**主线程**。

----------------------------------------------------------------------------------------------

## 实现多线程的方法

* ###   两种实现多线程的方法

  	* Runnable方法

  ```java
  //用Runnable的方式创建线程
  public class RunnableStyle implements Runnable {
  
      public static void main(String[] args) {
          Thread thread = new Thread ( new RunnableStyle () );
          thread.start ();
      }
      @Override
      public void run() {
          System.out.println("实现多线程");
      }
  }
  ```
  
  * Thread方法

   ``` java
//用Thread方式实现线程
	public class ThreadStyle extends Thread{

	    @Override
	    public void run() {
	        System.out.println ( "实现多线程" );
	    }
	
	    public static void main(String[] args){
	        new ThreadStyle ().start ();
	    }
	}
	 ```

* ### 那么该用哪个呢？

  * Runnable方法中创建线程机制与具体任务实现（run方法）是分开的。

  * Thread方法每完成一个具体任务都要建立一个新的线程，而Runnable后续可以使用线程池等工具，使用Runnable比较节约资源。

  * 由于Java不支持双继承，Thread方法继承了Thread类，则无法继承其他的类。

　**因此综上所述，用Runnable方法更好一些**

* ### 两者本质上的对比

  我们找到Thread类里的run方法

```Java
@Override
public void run(){
    if(target ! = null){
        target.run();
    }
}
```

### 顺便看一下target是什么

`` private Runnable target``

当我们使用第一种方法时，实现Runnable接口的时候，实际上传进来一个target对象，执行了Runnable的run方法。

而方法二呢，子类重写了父类的方法，因此本质上就是执行重写后的那句话,不再调用Thread里的run方法。

* ### 把两种方法发在一起验证一下

  ```java
  //同时使用Runnable和Thread两种实现线程的方式
  public class BothRunnableThread {
  
      public static void main(String[] args) {
          new Thread ( new Runnable( )  {
              @Override
              public void run() {
                  System.out.println ( "Runnable" );
              }
          }) {
              @Override
              public void run() {
                  System.out.println ("Thread");
              }
          }.start();
      }
  }
  ```

  发现最后的输出只有

  ```java
  Thread
  
  Process finished with exit code 0
  
  ```

  说明即使target传入也依旧不会执行Runnable类的run方法，最后还是会执行Thread里重写的run方法。

  所以总的来说，创建线程只有一种方法那就是**创建Thread类**，但是实现线程的的**run方法**的方式却有两种：

  * 实现Runnable接口的run方法，然后把target传给Thread类
* 重写Thread的run方法，不在运行原run方法
  
  ----------------------------------------------------------------------
## 启动线程的方式: start()

### **start()方法**含义

* 首先这个方法是用来**请求JVM**，若在其空闲时就启动新线程。并不是说只要用start方法就立即启动新线程。

  所以会发生启动的顺序与请求的顺序不符合的情况，比如第一个请求的线程可能是第三个被创建好的。

* 其次呢，start()方法会让**两个线程**同时运行，其中一个就是主线程或者其他的一个线程去执行start方法去创建新线程。

  * 在第上一步说的创建的新线程在启动前还会有一些准备工作，在启动前会先获取除CPU之外的其他资源（TODO:去查一下具体都有啥）。在做好准备工作后，线程才会被JVM或操作系统调度到执行状态去等待获取CPU资源，在获取CPU之后才会到运行状态。

* 不能重复的执行start()方法，一个线程执行两次会报错。

  看一下start的源码

  ```java
  public synchronized void start() {
          /**
           * This method is not invoked for the main method thread or "system"
           * group threads created/set up by the VM. Any new functionality added
           * to this method in the future may have to also be added to the VM.
           *
           * A zero status value corresponds to state "NEW".
           */
          if (threadStatus != 0)
              throw new IllegalThreadStateException();
  
          /* Notify the group that this thread is about to be started
           * so that it can be added to the group's list of threads
           * and the group's unstarted count can be decremented. */
          group.add(this);
  
          boolean started = false;
          try {
              start0();
              started = true;
          } finally {
              try {
                  if (!started) {
                      group.threadStartFailed(this);
                  }
              } catch (Throwable ignore) {
                  /* do nothing. If start0 threw a Throwable then
                    it will be passed up the call stack */
              }
          }
      }
  
      private native void start0();
  ```

  这里在最开始就会先检查线程的状态，由于一开始线程初始化时会让threadStatus为０，所以在后面如果线程的状态码不为０，则会抛出异常。所以一个线程在执行之后再次start，状态码不为０会抛出异常．

### run()方法，如果执行不会去创建子线程，会直接在主线程运行

```java
public class StartAndRunMethod {
    public static void main(String[] args) {
        Runnable runnable = () ->{
            System.out.println (Thread.currentThread ().getName ());
        };
        runnable.run();//最后输出main，使用的为主线程

        new Thread ( runnable ).start ();//输出Thread-0，创建新线程
    }
}
```

* 既然start()方法会调用run()方法，为什么我们选择调用start方法，而不是直接调用run方法呢？

  因为调用start方法才是真正意义上的启动了一个线程，它会去经历线程的各个生命周期。而我们直接调用run方法，它就仅仅是一个方法而已，不会去创建子线程 ，就像上面那个代码还是只会调用主线程，所以输出的是main.

---------------------------------------

## 停止线程的方法

### 使用interrupt来通知线程停止

由于被停止的线程不一定是我们自己写的，我们不一定了解线程内部的运行逻辑，如果贸然停止可能会出问题。所以如果希望能安全正确的停止线程,Java就把停止线程的权利赋予了该线程本身，所以我们如果想停止线程正确的方法应该去使另外的一个线程去**通知**该被停止的线程，应该运行线程本身的停止机制了。

所以想要使线程顺利停止，需要在run方法里加入响应停止信号的机制例如判断：

``` 
Thread.currentThread ().isInterrupted () 
```

```Java
public class StopThread implements Runnable {

    @Override
    public void run(){
        int num = 0;
        while(!Thread.currentThread ().isInterrupted () && num <= Integer.MAX_VALUE / 2){//响应中断
            if(num % 10000 == 0) {
                System.out.println (num+"是10000的倍数");
            }
            num++;
        }
        System.out.println ("任务结束");
    }

    public static void main(String[] args) throws InterruptedException {
        Thread thread = new Thread ( new StopThread () );
        thread.start ();
        Thread.sleep ( 1000 );
        thread.interrupt ();
    }

}

```

如果希望线程在阻塞状态(sleep,wait)下也会被正常被中断，要在run方法加入捕获异常的语句

```java
java.lang.InterruptedException: sleep interrupted
	at java.base/java.lang.Thread.sleep(Native Method)
	at threadcoreknowledge.creatthread.stopthread.RightWayStopThreadWithSleep.lambda$main$0(RightWayStopThreadWithSleep.java:15)
	at java.base/java.lang.Thread.run(Thread.java:835)
```

**但是这种情况下try catch不能放在循环内部**，如果放在循环内部,就算抛出异常也不会终止循环本身，因为sleep方法会清除中断标记位，如果只用try,catch,异常就会在throwInMethod()里被吞掉了（看下面代码）,不会反应异常到run()方法，该用下面这种写法

```Java
public class StopThread implements Runnable{
    @Override
    public void run(){
        while (true) {//即使用
            System.out.println ("go");
            try {
                throwInMethod();
            } catch (InterruptedException e) {
                //由于异常已经上传到这里，这里就可以加上保存日志或停止程序等相关操作
                e.printStackTrace ();
            }
        }
              //因为@Ｏverride所以run方法本身没有写异常处理的话，这里只能用try/catch
    }
/*
     private void throwInMethod(){
        try {
            Thread.sleep ( 2000 );
        } catch (InterruptedException e) {
            e.printStackTrace ();
        }
    }
这里有关异常抛出后的处理应该在run()方法里编写
但是这里不该用try/catch去只打印异常，应该把捕获异常写在方法签名里，传给上一层run()方法
 */
   private void throwInMethod() throws InterruptedException {
        Thread.sleep ( 2000 );
    }
    public static void main(String[] args) throws InterruptedException {
        Thread thread = new Thread ( new StopThread () );
        thread.start ();
        Thread.sleep ( 1000 );
        thread.interrupt ();
    }
}
```

又或者可以在catch语句中调用Thread.currentThread().interrupt()

```java
    @Override
    public void run(){
        while (true) {
            if(Thread.currentThread ().isInterrupted ()){
                System.out.println ("捕获到异常");//这里写捕获到之后的应对
                break;
            }
            System.out.println ("go");
            throwInMethod();

        }
    }
    private void throwInMethod(){
        try {
            Thread.sleep ( 2000 );
        } catch (InterruptedException e) {
            Thread.currentThread ().interrupt ();//继续抛出中断信息
            e.printStackTrace ();
        }
    }

```

----------------------------------------------

## wait   ,  notify  ,  notifyAll

**若线程调用wait则会进入阻塞状态,满足以下任意条件就会被唤醒**

* 当另一个线程调用这个对象的notify()方法且刚好唤醒的是本线程（貌似是随机唤醒）
* 另一个线程调用的这个对象的notifyAll()方法
* 过了wait(long timeout)规定的超时时间，如果传入０就是永久等待
* 线程自身调用了interrupt()

**这三个方法的特点**

* 若想执行以上三个方法，则必须获得锁，否则会抛异常的
* 如果使用notify(),则只会唤醒一个线程，且是任意一个
* 都属于Object类
* 释放锁只会释放wait对应对象的那把锁，如果一个线程有多把锁，则释放的顺序很重要，不然容易死锁

**这里顺便说一下线程的六个状态**

* 最普通的状态流程一般是　**NEW(新创建状态)**> **RUNNABLE(可运行状态)**->**TERMINATED(终止状态)**

* 其中在 **RUNNABLE(可运行状态)**下可通过

  * 进入**synchronized**修饰的相关方法或代码块中进入**BLOCKED(被阻塞状态)**,在获取锁之后返回

  * 通过Object.wait(),Thread.join(),LockSupport.park()进入**WAITING(等待状态)**

    通过Object.notify(),Object.notifyAll(),LockSupport.unpark()返回

  * 通过Thread.sleep(time),Object.wait(time),Thread.join(time),LockSupport.parkNanos(time),LockSupprot.parkUnil(time)方法进入**TIMED_WAITING(计时等待状态）**

   由 Object.notify(),Object.notifyAll(),LockSupport.unpark()返回或计时结束自动返回

这里程序从Object.wait()状态刚被唤醒时，通常不能立刻抢到锁，那就会从Waiting先进入Blocked状态，抢到锁之后再转换Runnable状态

如果发生异常，可以直接跳到终止Terminated(终止状态)，不必再遵循路径，比如可以从**WAITING(等待状态)**直接到**TERMINATED(终止状态)**

### 为什么线程中 wait() , notify() 和 notifyAll() 定义在Object类里？而sleep定义在Thread类里？

主要是因为在Java中 wait() , notify() 和 notifyAll() 是锁级别的操作，是属于某一个对象的，因为锁是绑定在某一个对象上的而不是某一个线程中，如果定义在线程类中则有很大的局限，一个线程是可以有多个锁的，如果定义在线程类中则太不灵活了。

wait和notify的使用例子

```java
//wait和notify的基本用法
public class wait {
    public static Object object = new Object ();

    static class Thread1 extends Thread{
        @Override
        public void run(){
            synchronized (object){
                System.out.println ("线程"+ Thread.currentThread ().getName ()+"开始执行了");
                try {
                    object.wait ();//释放掉锁
                } catch (InterruptedException e) {
                    e.printStackTrace ();
                }
                System.out.println ( "线程"+ Thread.currentThread ().getName ()+"获取到了锁" );
            }
        }
    }
    static class Thread2 extends Thread {
        @Override
        public void run(){
            synchronized (object){
                object.notify ();
                System.out.println ("线程"+Thread.currentThread ().getName ()+"调用了notify()");
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        Thread1 thread1 = new Thread1 ();
        Thread2 thread2 = new Thread2 ();
        thread1.start ();
        Thread.sleep ( 200 );//保证两个线程的运行顺序
        thread2.start ();
    }
}
```

```java
/usr/lib/jvm/jdk-12.0.2/bin/java -javaagent:/home/ryan/.local/share/JetBrains/Toolbox/apps/IDEA-U/ch-0/192.6262.41/lib/idea_rt.jar=35421:/home/ryan/.local/share/JetBrains/Toolbox/apps/IDEA-U/ch-0/192.6262.41/bin -Dfile.encoding=UTF-8 -classpath /home/ryan/Java_高并发/out/production/Java_高并发 threadcoreknowledge.creatthread.threadobjectclasscommonmethods.wait
线程Thread-0开始执行了
线程Thread-1调用了notify()
线程Thread-0获取到了锁

Process finished with exit code 0
```

从结果可以看出线程object.wait ();后释放掉锁，由另一个线程执行notify()获取到锁之后，再运行notify之后的代码，之后wait的线程才被唤醒重新获得锁

我们再试一下notifyAll()

```java
//三个线程，线程１和线程２首先被阻塞，用线程３唤醒他们
public class WaitNotifyAll implements Runnable {
    private static final Object reasourceA = new Object ();

    public static void main(String[] args) throws InterruptedException {
        Runnable r = new WaitNotifyAll ();
        Thread threadA = new Thread ( r );
        Thread threadB = new Thread ( r );
        Thread threadC = new Thread ( new Runnable () {
            @Override
            public void run() {
                synchronized (reasourceA){
                    reasourceA.notifyAll ();
                    System.out.println ("线程C notify");
                }
            }
        } );
        threadA.start ();
        threadB.start ();
        Thread.sleep(200);//要保证ＡＢＣ三个线程的运行顺序，保证唤醒前AB在等待状态
        threadC.start ();
    }

    @Override
    public void run(){
        synchronized (reasourceA){
            System.out.println (Thread.currentThread ().getName ()+"获取Ａ的锁");
            try{
                System.out.println (Thread.currentThread ().getName ()+"等待下一步");
                reasourceA.wait ();//陷入等待
                System.out.println (Thread.currentThread ().getName ()+"唤醒");//已经被唤醒
            } catch (InterruptedException e) {
                e.printStackTrace ();
            }
        }
    }
}
```

```java
/usr/lib/jvm/jdk-12.0.2/bin/java -javaagent:/home/ryan/.local/share/JetBrains/Toolbox/apps/IDEA-U/ch-0/192.6262.41/lib/idea_rt.jar=38353:/home/ryan/.local/share/JetBrains/Toolbox/apps/IDEA-U/ch-0/192.6262.41/bin -Dfile.encoding=UTF-8 -classpath /home/ryan/Java_高并发/out/production/Java_高并发 threadcoreknowledge.creatthread.threadobjectclasscommonmethods.WaitNotifyAll
Thread-0获取Ａ的锁
Thread-0等待下一步
Thread-1获取Ａ的锁
Thread-1等待下一步
线程C notify
Thread-0唤醒
Thread-1唤醒
```













































