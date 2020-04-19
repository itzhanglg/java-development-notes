### 一.ThreadLocal详解

#### 1.ThreadLocal的简单使用及场景

前文介绍了线程同步的三种方式: **轻量级的Atomic, volatile 和重量级的 synchronized**. 其实都是采用了同步的方式解决了线程的安全问题. 而ThreadLocal解决线程安全问题的思路是**线程封闭**.

ThreadLocal 是 JDK包提供的,它提供了线程本地变量,若创建了一个ThreadLocal 变量,那么访问这个变量的每个线程都会有这个变量的一个本地副本.当多个线程操作这个变量时,实际操作的是自己本地内存里面的变量,因此避免了线程的安全问题.

ThreadLocal的使用场景: ThreadLocal中存储的变量是线程隔离的

- **存储需要在线程隔离的数据**: 如线程执行的上下文信息，每个线程是不同的，但是对于同一个线程来说会共享同一份数据; Spring MVC的 RequestContextHolder 的实现就是使用了ThreadLocal；
- **跨层传递参数**: 放入ThreadLocal跨层传递的变量一般也是具有上下文属性的。比如用户的信息等

一般来说，在实践中，我们会把ThreadLocal对象声名为static final，作为私有变量封装到自定义的类中。另外提供static的set和get方法: 

```java
public final class OperationInfoRecorder {

    private static final ThreadLocal<OperationInfoDTO> THREAD_LOCAL = new ThreadLocal<>();

    private OperationInfoRecorder() {
    }

    public static OperationInfoDTO get() {
        return THREAD_LOCAL.get();
    }

    public static void set(OperationInfoDTO operationInfoDTO) {
        THREAD_LOCAL.set(operationInfoDTO);
    }

    public static void remove() {
        THREAD_LOCAL.remove();
    }
}
```

用static和final修饰的目的是:

- **static 确保全局只有一个保存OperationInfoDTO对象的ThreadLocal实例**
- **final 确保ThreadLocal的实例不可更改**。防止被意外改变，导致放入的值和取出来的不一致。另外还能防止ThreadLocal的内存泄漏.

每个线程为同一个ThreadLocal对象set不同的值，但各个线程打印出来的依旧是自己保存进去的值，并没有被其它线程所覆盖。

#### 2.ThreadLocal源代码分析

首先看一下 ThreadLocal 相关类的类图结构:

![run12](../../../media/pictures/run12.jpg)

Thread类中有两个变量,他们都是ThreadLocalMap类型的变量,而ThreadLocalMap是一个定制化的Hashmap.默认情况下,每个线程中的这两个变量都为null,只有当前线程第一次调用ThreadLocal的set或get方法时才会创建他们.其实每个线程的本地变量不是存放在ThreadLocal对象里而是存放在调用线程的threadLocals变量里面.

换句话说 ThreadLocal 类型的本地变量存放在具体的线程内存空间中. ThreadLocal就是一个工具壳. 

如果调用线程一直不终止,那么这个本地变量会一直存放在调用线程的threadLocals变量里,所以当不需要使用本地变量时可以调用remove方法删除该本地变量.

下面简单分析 `ThreadLocal` 的 set, get 及 remove 方法的源代码:

**set方法:** 会把当前threadLocal对象作为key，你想要保存的对象作为value，存入map

```java
public void set(T value) {
    // 获取当前线程
    Thread t = Thread.currentThread();
    // 将当前线程作为key,去查找对应的线程变量,找到则设置.这个map其实是和Thread绑定的
    ThreadLocalMap map = getMap(t);
    if (map != null)
        // 把value值设置到threadLocals中,即把当前变量值放入当前线程的内存变量threadLocals中
        map.set(this, value);
    else
        // 第一次调用创建当前线程对应的HashMap
        createMap(t, value);
}

// getMap的作用是获取线程自己的变量threadLocals,threadLocal变量被绑定到了线程的成员变量上
ThreadLocalMap getMap(Thread t) {
    return t.threadLocals;
}

//第一次调用创建当前线程的threadLocals变量
void createMap(Thread t, T firstValue) {
    t.threadLocals = new ThreadLocalMap(this, firstValue);
}
```

`Thread`类中:

```java
 /* ThreadLocal values pertaining to this thread. This map is maintained
  * by the ThreadLocal class. */
ThreadLocal.ThreadLocalMap threadLocals = null;
```

threadLocals这个属性由ThreadLocal来维护。threadLocals的访问控制决定在包外是无法直接访问的。所以我们在使用的时候只能通过ThreadLocal对象来访问。

**再来看get方法**: 

```java
public T get() {
    // 获取当前线程
    Thread t = Thread.currentThread();
    // 获取当前线程的threadLocals变量
    ThreadLocalMap map = getMap(t);
    // 如果threadLocals不为null,则返回对应本地变量的值
    if (map != null) {
        ThreadLocalMap.Entry e = map.getEntry(this);
        if (e != null) {
            @SuppressWarnings("unchecked")
            T result = (T)e.value;
            return result;
        }
    }
    // threadLocals为null则初始化当前线程的threadLocals成员变量
    return setInitialValue();
}

// 进行初始化
private T setInitialValue() {
    // 初始化为null
    T value = initialValue();
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    // 如果当前线程的threadLocals变量不为null,则设置当前线程的本地变量值为null
    if (map != null)
        map.set(this, value);
    else
        // 如果当前线程的threadLocals变量为null,创建当前线程的threadLocals变量
        createMap(t, value);
    return value;
}

// 初始化为null
protected T initialValue() {
    return null;
}
// 创建当前线程的threadLocals变量
void createMap(Thread t, T firstValue) {
    t.threadLocals = new ThreadLocalMap(this, firstValue);
}
```

**再来看下remove方法**:

```java
public void remove() {
    ThreadLocalMap m = getMap(Thread.currentThread());
    // 若当前线程的threadLocals变量不为null,则删除当前线程中指定ThreadLocal实例的本地变量
    if (m != null)
        m.remove(this);
}
```

总结: **在每个线程内部都有一个名为 threadLocals 的成员变量,该变量的类型为HashMap,其中key为我们定义的 ThreadLocal 变量的 this 引用, value则为我们使用set方法设置的值. 每个线程的本地变量存放在线程自己的内存变量 threadLocals 中,如果当前线程一直不消亡,那么这些本地变量会一直存在,所以可能会造成内存溢出,因此使用完毕后,会调用remove方法删除对应线程的 threadLocals 中的本地变量.**

#### 3.ThreadLocalMap分析

Thread对象中用来保存变量副本的ThreadLocalMap的定义就在ThreadLocal中。**ThreadLocalMap是ThreadLocal的静态内部类**, ThreadLocalMap的功能其实是和HashMap类似的. 

在ThreadLocalMap中使用WeakReference包装后的ThreadLocal对象作为key，也就是说这里对ThreadLocal对象为弱引用。当ThreadLocal对象在ThreadLocalMap引用之外，再无其他引用的时候能够被垃圾回收。

```java
static class ThreadLocalMap {

    static class Entry extends WeakReference<ThreadLocal<?>> {
        /** The value associated with this ThreadLocal. */
        Object value;

        Entry(ThreadLocal<?> k, Object v) {
            super(k);
            value = v;
        }
    }
    // ...
}    
```

如果ThreadLocal对象被回收，那么ThreadLocalMap中保存的key值就变成了null，而value会一直被Entry引用，而Entry又被threadLocalMap对象引用，threadLocalMap对象又被Thread对象所引用，那么当Thread一直不终结的话，value对象就会一直驻留在内存中，直至Thread被销毁后，才会被回收。这就是ThreadLocal引起内存泄漏问题。

可以通过以下两种方式来避免这个问题：

- **把ThreadLocal对象声明为static**，这样ThreadLocal成为了类变量，生命周期不是和对象绑定，而是和类绑定，延长了声明周期，避免了被回收
- 在使用完ThreadLocal变量后，**手动remove掉**，防止ThreadLocalMap中Entry一直保持对value的强引用。导致value不能被回收

### 二.AQS分析

