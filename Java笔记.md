## Java 笔记

### 一、泛型
- 限制泛型可用类型：` class MyClass<T extends anyClass>`,这里限制T必须满足条件：继承或者实现anyClass.
- **<? extends AnyClass>**： 在使用泛型类声明对象引用时再限定泛型类型，比如这么写：
```
class MyClass<T>{
    ....
}
MyClass<? extends List> testClass = null;
testClass = new MyClass<ArrayList>();
```
- **<? super AnyClass>**:上一个类型限定符表明传入类型必须继承了AnyClass，而这个限定符表明，传入类型要么是AnyClass的类型，要么是AnyClass类型的上层类型。向上限制。


### 二、注解（Annotation）

- 使用@interface来定义注解
- 注解的成员变量声明格式：`String describe() default "defaultValue"`;
- 对注解使用`@Target()`和`@Retention()`用来指明注解的作用目标和有效范围。
- **如果在定义注解类型时，将@Retention设置为RetentionPolicy.RUNTIME,那么在运行程序时通过反射就可以获取到相关的注解信息。**

### 三、多线程
- 在windows、Linux中，一个Java中的线程就对应操作系统的一个轻量级进程（轻量级进程与内核线程一一对应）。
- Java线程的调度方式：抢占式调度策略。
- Java线程的6个状态：
    1. New:创建之后尚未启动的线程。
    2. Runable:包括了操作系统线程的Running和Ready状态，也就是说处于这个状态的线程可能在运行中，也可能在等待CPU为它分配时间。
    3. Wating:处于这个状态的线程，不会被CPU分配时间，只有当被其他线程显示地唤醒时，才能被调度执行。
    4. Time Waiting:处于这个状态的线程也不会被CPU分配执行时间，不过无须等待被其他线程显示地唤醒，在一定时间之后它们会由系统自动唤醒。
    5. Blocked:当线程等待其他线程释放排他锁的时候处于Blocked状态。
    6. Terminated:线程结束执行。
- **让线程处于Waiting状态的方法：** 1）没有设置Timeout参数的Object.wait()方法。2）没有设置Timeout参数的Thread.join()方法。3）LockSupport.park()方法。
- **让线程处于TimeWating的方法：** 1）设置了Timeout参数的Object.wait()方法。2）设置了Timeout参数的Thread.join()方法。3）Thread.sleep()方法。4）LockSupport.parkNanos()方法。5）LockSupport.parkUtil()方法.

### 四、锁机制
- **synchronized**:可以作为修饰符修饰普通方法、静态方法，也可以在方法内部使用。重点：synchronized作用于对象，进入临界区之后，要想阻塞线程或者唤醒阻塞的线程需要调用**被锁对象**的wait()或notifyAll()、notify()方法，比如对this加锁，就应该调用this.wait(),this.notify()
```
class Demo{
    //修饰普通方法，对当前对象加锁
	public synchronized void testMethod(){
	}
	//修饰静态方法：对当前这个类的class对象加锁，即对Demo.class加锁，Demo.class在内存中唯一，由于锁的对象不同，因此与上面的锁不是同一个锁。
	public static synchronized void testMethod2(){
	}
	private final Object lock = new Object();
	public void testMethod3(){
		//此处的this，可以换成任意一个不变的对象，如果是this则对当前对象加锁
		synchronized(this){
		}
		//对别的对象加锁
		synchronized(lock){
		}
	}
}
```

- **Lock**：java.util.concurrent.locks.Lock 提供的互斥锁，是一个接口，常用实现类**ReentrantLock**结合**Condition**使用,下面是例子。

```
public class Demo {
    private Lock lock = new ReentrantLock();
    private Condition condition = lock.newCondition();
    private int count = 0;
    public void testMethod(){
        //Condition对象提供类似于Object对象的wait(),notifyAll()函数,用于当某个条件满足时阻塞或者唤醒线程
        //condition.await();
        //condition.signalAll();
        lock.lock();
        count++;
        lock.unlock();
    }
}
```
- **Lock和synchronized的区别**：
1.使用Lock，如果线程加锁失败，线程进入阻塞状态，此时阻塞状态的线程仍然能相应中断，而synchronized阻塞的线程不能相应中断。
2.Lock支持超时：当设置超时参数之后，如果尝试一段时间仍不能获取锁，则返回一个错误，而不是直接进入阻塞状态。
3.Lock支持非阻塞获取锁，如果获取失败则直接返回而不是进入阻塞状态。

- **信号量 Semophore**:相比于上面的两种锁机制，信号量的特点是可以允许多个线程同时进入临界区操作。

```
class Demo{
    //Semophore的permits参数可以理解为同时能进入临界区的线程数量
    private Semaphore sm = new Semaphore(1);
    public void test() throws InterruptedException {
        sm.acquire();
        //do something
        sm.release();
    }
}
```
        