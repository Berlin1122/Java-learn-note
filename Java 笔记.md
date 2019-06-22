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
- **<?>**：这个类型通配符，表明待传入的类型不确定，由于传入的类型不确定，所以相当于传入任何确定的类型也没有用。例如：
```
List<?> myList = new ArrayList<>();
myList.add("test");//会导致编译不通过，这里我们显示的指明了传入类型为String,但是在声明时使用了 "?"。
                    //导致不能匹配传入的类型，此列表不能添加元素。这个列表会认为里面的元素类型都是不确定的，所以获取列表的元素也需要类型转换。
```

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
        