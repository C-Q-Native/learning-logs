---
title: "Java 面试题汇编 (一)"
date: 2021-06-18T16:23:19+08:00
categories:
- java
tags:
- java
- Interview
---
**1. map里，hashmap无序的，想要有序的该怎么办？**  
HashMap
- Map存储过程：计算key的hashCode值，确定位置；如果位置上有元素，用equals判断是否一致，如果一致，不存，如果不一致，遍历下一个节点继续判断equals。注：添加是添加到链表的尾部。
加载因子（扩容条件）：默认0.75，空间存储75%后扩容。
- 扩容：将原来的元素重新计算位置再存入；计算需要时间，因此不可以频繁扩容，扩容是成倍的。
- HashMap根据键可以很快找到他的值，因此具有很快的访问速度，遍历时，取得数据的顺序是完全随机的。
- HashMap不支持线程的同步，即任一时刻可以有多个线程同时写HashMap;可能会导致数据的不一致。
- HashMap的键只允许一条为null，值可以多个为null  
<!--more-->
LinkedHashMap——有序的Map

- 使用Map接口的哈希表和链表实现，具有可预知的迭代顺序。此实现与HashMap的不同之处在于：LinkedHashMap维护着一个双向循环链表。此链表定义了迭代顺序，该迭代顺序通常就是存放元素的顺序。
- 如果在Map中重新存入已有的key，那么key的位置不会发生改变，只是将value值替换。
- LinkedHashMap的内部维持了一个双向链表，保存了数据的插入顺序，遍历时，先得到的数据便是先插入的
遍历速度比HashMap慢

**2.Java线程池的分析和使用**   
合理利用线程池能够带来三个好处。第一：降低资源消耗。通过重复利用已创建的线程降低线程创建和销毁造成的消耗。第二：提高响应速度。当任务到达时，任务可以不需要的等到线程创建就能立即执行。第三：提高线程的可管理性。线程是稀缺资源，如果无限制的创建，不仅会消耗系统资源，还会降低系统的稳定性，使用线程池可以进行统一的分配，调优和监控。但是要做到合理的利用线程池，必须对其原理了如指掌。  
2.1 线程池的创建  
我们可以通过ThreadPoolExecutor来创建一个线程池。  
创建一个线程池需要输入几个参数：  
- corePoolSize（线程池的基本大小）：当提交一个任务到线程池时，线程池会创建一个线程来执行任务，即使其他空闲的基本线程能够执行新任务也会创建线程，等到需要执行的任务数大于线程池基本大小时就不再创建。如果调用了线程池的prestartAllCoreThreads方法，线程池会提前创建并启动所有基本线程。
- runnableTaskQueue（任务队列）：用于保存等待执行的任务的阻塞队列。可以选择以下几个阻塞队列。  
*ArrayBlockingQueue*：是一个基于数组结构的有界阻塞队列，此队列按 FIFO（先进先出）原则对元素进行排序。  
*LinkedBlockingQueue*：一个基于链表结构的阻塞队列，此队列按FIFO （先进先出） 排序元素，吞吐量通常要高于ArrayBlockingQueue。静态工厂方法Executors.newFixedThreadPool()使用了这个队列。  
*SynchronousQueue*：一个不存储元素的阻塞队列。每个插入操作必须等到另一个线程调用移除操作，否则插入操作一直处于阻塞状态，吞吐量通常要高于LinkedBlockingQueue，静态工厂方法Executors.newCachedThreadPool使用了这个队列。  
*PriorityBlockingQueue*：一个具有优先级得无限阻塞队列。
- maximumPoolSize（线程池最大大小）：线程池允许创建的最大线程数。如果队列满了，并且已创建的线程数小于最大线程数，则线程池会再创建新的线程执行任务。值得注意的是如果使用了无界的任务队列这个参数就没什么效果。
- ThreadFactory：用于设置创建线程的工厂，可以通过线程工厂给每个创建出来的线程设置更有意义的名字，Debug和定位问题时非常又帮助。
- RejectedExecutionHandler（饱和策略）：当队列和线程池都满了，说明线程池处于饱和状态，那么必须采取一种策略处理提交的新任务。这个策略默认情况下是AbortPolicy，表示无法处理新任务时抛出异常。  
*AbortPolicy*：直接抛出异常。  
*CallerRunsPolicy*：只用调用者所在线程来运行任务。  
*DiscardOldestPolicy*：丢弃队列里最近的一个任务，并执行当前任务。  
*DiscardPolicy*：不处理，丢弃掉。  
可以根据应用场景需要来实现RejectedExecutionHandler接口自定义策略。如记录日志或持久化不能处理的任务。  

2.2 向线程池提交任务  
2.2.1. 使用execute提交的任务
execute方法没有返回值，所以无法判断任务知否被线程池执行成功。

```
threadsPool.execute(new Runnable() {
    @Override
    public void run() {
        // TODO Auto-generated method stub
    }
});
```

2.2.2. 使用submit 方法来提交任务
它会返回一个future,那么我们可以通过这个future来判断任务是否执行成功，通过future的get方法来获取返回值，get方法会阻塞住直到任务完成，而使用get(long timeout, TimeUnit unit)方法则会阻塞一段时间后立即返回，这时有可能任务没有执行完。

```
try {
    Object s = future.get();
} catch (InterruptedException e) {
    // 处理中断异常
} catch (ExecutionException e) {
    // 处理无法执行任务异常
} finally {
    // 关闭线程池
    executor.shutdown();
}
```

2.3. 线程池的关闭  
可以通过调用线程池的shutdown或shutdownNow方法来关闭线程池，但是它们的实现原理不同，shutdown的原理是只是将线程池的状态设置成SHUTDOWN状态，然后中断所有没有正在执行任务的线程。shutdownNow的原理是遍历线程池中的工作线程，然后逐个调用线程的interrupt方法来中断线程，所以无法响应中断的任务可能永远无法终止。shutdownNow会首先将线程池的状态设置成STOP，然后尝试停止所有的正在执行或暂停任务的线程，并返回等待执行任务的列表。  
只要调用了这两个关闭方法的其中一个，isShutdown方法就会返回true。当所有的任务都已关闭后,才表示线程池关闭成功，这时调用isTerminaed方法会返回true。至于我们应该调用哪一种方法来关闭线程池，应该由提交到线程池的任务特性决定，通常调用shutdown来关闭线程池，如果任务不一定要执行完，则可以调用shutdownNow。
2.4 线程池的分析  
首先线程池判断基本线程池是否已满？没满，创建一个工作线程来执行任务。满了，则进入下个流程。  
其次线程池判断工作队列是否已满？没满，则将新提交的任务存储在工作队列里。满了，则进入下个流程。  
最后线程池判断整个线程池是否已满？没满，则创建一个新的工作线程来执行任务，满了，则交给饱和策略来处理这个任务。  
2.4. 合理的配置线程池  
要想合理的配置线程池，就必须首先分析任务特性，可以从以下几个角度来进行分析：  
任务的性质：CPU密集型任务，IO密集型任务和混合型任务。  
任务的优先级：高，中和低。  
任务的执行时间：长，中和短。  
任务的依赖性：是否依赖其他系统资源，如数据库连接。  
任务性质不同的任务可以用不同规模的线程池分开处理。CPU密集型任务配置尽可能少的线程数量，如配置Ncpu+1个线程的线程池。IO密集型任务则由于需要等待IO操作，线程并不是一直在执行任务，则配置尽可能多的线程，如2*Ncpu。混合型的任务，如果可以拆分，则将其拆分成一个CPU密集型任务和一个IO密集型任务，只要这两个任务执行的时间相差不是太大，那么分解后执行的吞吐率要高于串行执行的吞吐率，如果这两个任务执行时间相差太大，则没必要进行分解。我们可以通过Runtime.getRuntime().availableProcessors()方法获得当前设备的CPU个数。  
优先级不同的任务可以使用优先级队列PriorityBlockingQueue来处理。它可以让优先级高的任务先得到执行，需要注意的是如果一直有优先级高的任务提交到队列里，那么优先级低的任务可能永远不能执行。  
执行时间不同的任务可以交给不同规模的线程池来处理，或者也可以使用优先级队列，让执行时间短的任务先执行。  
依赖数据库连接池的任务，因为线程提交SQL后需要等待数据库返回结果，如果等待的时间越长CPU空闲时间就越长，那么线程数应该设置越大，这样才能更好的利用CPU。  

**4. 锁详解**  
4.1  synchronized vs ReentrantLock  
synchronized 属于独占式悲观锁，是通过 JVM 隐式实现的，synchronized 只允许同一时刻只有一个线程操作资源。  
ReentrantLock 是 Lock 的默认实现方式之一，它是基于 AQS（Abstract Queued Synchronizer，队列同步器）实现的，它默认是通过非公平锁实现的，在它的内部有一个 state 的状态字段用于表示锁是否被占用，如果是 0 则表示锁未被占用，此时线程就可以把 state 改为 1，并成功获得锁，而其他未获得锁的线程只能去排队等待获取锁资源。  
4.2 公平锁 VS 非公平锁  
公平锁的含义是线程需要按照请求的顺序来获得锁；而非公平锁则允许“插队”的情况存在，所谓的“插队”指的是，线程在发送请求的同时该锁的状态恰好变成了可用，那么此线程就可以跳过队列中所有排队的线程直接拥有该锁。  
而公平锁由于有挂起和恢复所以存在一定的开销，因此性能不如非公平锁，所以 ReentrantLock 和 synchronized 默认都是非公平锁的实现方式。  
4.3 悲观锁和乐观锁  
悲观锁： 指的是数据对外界的修改采取保守策略，它认为线程很容易会把数据修改掉，因此在整个数据被修改的过程中都会采取锁定状态，直到一个线程使用完，其他线程才可以继续使用。  
乐观锁： 乐观锁和悲观锁的概念恰好相反，乐观锁认为一般情况下数据在修改时不会出现冲突，所以在数据访问之前不会加锁，只是在数据提交更改时，才会对数据进行检测。  
Java 中的乐观锁大部分都是通过 CAS（Compare And Swap，比较并交换）操作实现的，CAS 是一个多线程同步的原子指令，CAS 操作包含三个重要的信息，即内存位置、预期原值和新值。如果内存位置的值和预期的原值相等的话，那么就可以把该位置的值更新为新值，否则不做任何修改。  
CAS 可能会造成 ABA 的问题，ABA 问题指的是，线程拿到了最初的预期原值 A，然而在将要进行 CAS 的时候，被其他线程抢占了执行权，把此值从 A 变成了 B，然后其他线程又把此值从 B 变成了 A，然而此时的 A 值已经并非原来的 A 值了，但最初的线程并不知道这个情况，在它进行 CAS 的时候，只对比了预期原值为 A 就进行了修改，这就造成了 ABA 的问题。  
ABA 的常见处理方式是添加版本号或者时间戳，每次修改之后更新版本号，每次修改之前校验版本号是否被修改过，修改过则需要重新获取值再修改，这样就解决了 ABA 的问题。  
4.4 可重入锁  
可重入锁也叫递归锁，指的是同一个线程，如果外面的函数拥有此锁之后，内层的函数也可以继续获取该锁。在 Java 语言中 ReentrantLock 和 synchronized 都是可重入锁。   
可重入锁的实现原理，是在锁内部存储了一个线程标识，用于判断当前的锁属于哪个线程，并且锁的内部维护了一个计数器，当锁空闲时此计数器的值为 0，当被线程占用和重入时分别加 1，当锁被释放时计数器减 1，直到减到 0 时表示此锁为空闲状态。  
4.5 共享锁和独占锁  
只能被单线程持有的锁叫独占锁，可以被多线程持有的锁叫共享锁。  
独占锁指的是在任何时候最多只能有一个线程持有该锁，比如 synchronized 就是独占锁，而 ReadWriteLock 读写锁允许同一时间内有多个线程进行读操作，它就属于共享锁。  
独占锁可以理解为悲观锁，当每次访问资源时都要加上互斥锁，而共享锁可以理解为乐观锁，它放宽了加锁的条件，允许多线程同时访问该资源。  
4.6 自适应自旋锁  
自适应式自旋锁意味着自旋的时间不再是固定的时间了，比如在同一个锁对象上，如果通过自旋等待成功获取了锁，那么虚拟机就会认为，它下一次很有可能也会成功 (通过自旋获取到锁)，因此允许自旋等待的时间会相对的比较长，而当某个锁通过自旋很少成功获得过锁，那么以后在获取该锁时，可能会直接忽略掉自旋的过程，以避免浪费 CPU 的资源，这就是自适应自旋锁的功能。  
4.7 锁升级  
锁升级其实就是从偏向锁到轻量级锁再到重量级锁升级的过程  
偏向锁是指在无竞争的情况下设置的一种锁状态。偏向锁的意思是它会偏向于第一个获取它的线程，当锁对象第一次被获取到之后，会在此对象头中设置标示为“01”，表示偏向锁的模式，并且在对象头中记录此线程的 ID，这种情况下，如果是持有偏向锁的线程每次在进入的话，不再进行任何同步操作，如 Locking、Unlocking 等，直到另一个线程尝试获取此锁的时候，偏向锁模式才会结束，偏向锁可以提高带有同步但无竞争的程序性能。但如果在多数锁总会被不同的线程访问时，偏向锁模式就比较多余了，此时可以通过 -XX:-UseBiasedLocking 来禁用偏向锁以提高性能。  
轻量锁是通过比较并交换（CAS，Compare and Swap）来实现的，它对比的是线程和对象的 Mark Word（对象头中的一个区域），如果更新成功则表示当前线程成功拥有此锁；如果失败，虚拟机会先检查对象的 Mark Word 是否指向当前线程的栈帧，如果是，则说明当前线程已经拥有此锁，否则，则说明此锁已经被其他线程占用了。  
重量锁是相对于轻量锁而言的，synchronized 是通过操作系统的互斥量（mutex lock）来实现的，这种实现方式需要在用户态和核心态之间做转换，有很大的性能消耗，这种传统实现锁的方式被称之为重量锁。
4.8 volatile和synchronized  
`synchronized` 关键字和 `volatile` 关键字是两个互补的存在，而不是对立的存在！
- `volatile` 关键字是线程同步的轻量级实现，所以 `volatile`性能肯定比`synchronized`关键字要好 。但是 `volatile` 关键字只能用于变量而 `synchronized` 关键字可以修饰方法以及代码块 。
- `volatile` 关键字能保证数据的可见性，但不能保证数据的原子性。`synchronized` 关键字两者都能保证。
- `volatile`关键字主要用于解决变量在多个线程之间的可见性，而 `synchronized` 关键字解决的是多个线程之间访问资源的同步性。
  
4.9 可中断锁和不可中断锁  
- **可中断锁** ：获取锁的过程中可以被中断，不需要一直等到获取锁之后 才能进行其他逻辑处理。ReentrantLock 就属于是可中断锁。
- **不可中断锁** ：一旦线程申请了锁，就只能等到拿到锁以后才能进行其他的逻辑处理。 synchronized 就属于是不可中断锁。

4.10 线程持有读锁还能获取写锁吗？  
- 在线程持有读锁的情况下，该线程不能取得写锁(因为获取写锁的时候，如果发现当前的读锁被占用，就马上获取失败，不管读锁是不是被当前线程持有)。
- 在线程持有写锁的情况下，该线程可以继续获取读锁（获取读锁时如果发现写锁被占用，只有写锁没有被当前线程占用的情况才会获取失败）。

4.11 读锁为什么不能升级为写锁  
写锁可以降级为读锁，但是读锁却不能升级为写锁。这是因为读锁升级为写锁会引起线程的争夺，毕竟写锁属于是独占锁，这样的话，会影响性能。

另外，还可能会有死锁问题发生。举个例子：假设两个线程的读锁都想升级写锁，则需要对方都释放自己锁，而双方都不释放，就会产生死锁。

**5. Bean的生命周期，单例？如何多例？**  
实例化Instantiation 、属性赋值Populate 、初始化Initalization 、销毁Destruction  
Bean默认是单例的，也就是说当我们向IOC容器中添加一个Bean，多次获取Bean，默认是同一个Bean，我们也可以显示地声明Bean为单例，使用@Scope设置属性为singleton即可

```
@Configuration
public class MyConfiguration {

    @Bean
    @Scope("singleton")
    public Person person(){
        return new Person(18,"wang");
    }
}
```
多例模式Bean  
将@Scope的值改为prototype，则该Bean为多例属性，每次从容器中取出，都是取出一个该Bean的原型复制出来的对象

```
@Configuration
public class MyConfiguration {

    @Bean
    @Scope("prototype")
    public Person person(){
        return new Person(18,"wang");
    }
}
```

**6. map的四种遍历方式**  
遍历方式一:需要说明的是，该方法不是最优选择，一般不推荐使用  

```
for (String key:map.keySet()){

    System.out.println("key= "+key+" and value= "+map.get(key));

}
```
遍历方式二:通过对map entrySet的遍历，也可以同时拿到key和value，一般情况下，性能上要优于keyset,这一种也是最常用的遍历方法,但Iterator也有其优势：在用foreach遍历map时，如果改变其大小，会报错，但如果只是删除元素，可以使用Iterator的remove方法删除元素  

```
Set<Map.Entry<String, String>> entries = map.entrySet();

for (Map.Entry<String, String> entry:entries){

    String value = entry.getValue();
    
    String key = entry.getKey();
    
    System.out.println("key="+key+"value="+value);

}
```
遍历方式三：Iterator

```
Iterator<Map.Entry<String, String>> iterator = map.entrySet().iterator();

while (iterator.hasNext()){

    Map.Entry<String, String> next = iterator.next();
    
    System.out.println("key="+next.getKey()+"value="+next.getValue());

}
```
遍历方式四:

```
for (String v:map.values()){

    System.out.println("value="+v);

}
```

**7. 深拷贝和浅拷贝的区别**    
1.浅拷贝： 如果一个类中有指针对象，那么在拷贝这个类的对象的时候，默认的拷贝方式是只拷贝指针本身，而不重新构建并拷贝指针所指内容。这就叫做浅拷贝Shallow Copy。Java的 Object.clone() 的机制是浅拷贝    
2.深拷贝： 如果拷贝的方式是不仅仅拷贝指针，而且把指针所指的内容也新建一份，那就叫深拷贝Deep Copy。实现深拷贝常用的实现方式有2种：Serializable,Cloneable。

**8.ThreadLocal原理及使用场景**   
ThreadLocal意为线程本地变量，用于解决多线程并发时访问共享变量的问题。  
场景一：在重入方法中替代参数的显式传递  
场景二：全局存储用户信息  
场景三：解决线程安全问题  

**9. GC Roots 包含哪些**  
虚拟机栈(栈帧中的本地变量表)中引用的对象  
本地方法栈(Native 方法)中引用的对象  
方法区中类静态属性引用的对象  
方法区中常量引用的对象    
所有被同步锁持有的对象    

**10. Spring IoC处理循环依赖**  
bean的生命周期：
- 实例化：执行了bean的构造方法，bean中依赖的对象还未赋值
- 设置属性：给bean中依赖的对象赋值，若被依赖的对象尚未初始化，则先进行该对象的生命周期（递归）。
- 初始化：执行bean的初始化方法，回调方法等。

解决循环依赖的思路就藏在这三个步骤中，在实例化与设置属性两个步骤之间引入缓存机制，将已经创建好实例但是并没有设置属性的bean放到缓存里，缓存中是没有属性设置的实例对象。假设A对象和B对象互相依赖，A对象的创建需要引用到B对象，而B对象的创建也需要A对象。在创建A对象的时候可以将其放入到缓存中，当B对象创建的时候直接从缓存里引用A对象（此时的A对象只完成了实例化，没有进行设置属性的操作，因此不是完成的A对象，我们称之为半成品A对象），当B对象利用这个半成品的A对象完成实例创建以后（三个步骤都完成），再被A对象引用进去，则A对象也完成了创建。  
上文提到的缓存在这里做一个解释，我们将其分为三级，每级缓存都起到不同的作用，如下表格所示：

- 一级缓存：用于存放完全初始化好的 bean，也就是完成三个步骤的bean，拿出来的bean是可以直接使用的。
- 二级缓存：存放原始的 bean 对象，此时的对象只进行了实例化但是没有填充属性，也就是我们所说的“半成品对象”，它的建立是用来解决循环依赖的。
- 三级缓存：用来存放 bean 工厂对象，这个工厂对象是用来产生bean对象的实例的。

源码 | 级别 | 描述
---|--- | ---
singletonObjects | 一级缓存 | 用于存放完全初始化好的 bean，从该缓存中取出的 bean 可以直接使用
earlySingletonObjects | 二级缓存 | 存放原始的 bean 对象（尚未填充属性），用于解决循环依赖
singletonFactories | 三级缓存 | 存放 bean 工厂对象，用于解决循环依赖

解决循环依赖的整个过程是：
先从一级缓存里取bean实例，如果没有对应的bean实例，二级缓存里取，如果二级缓存中也没有bean实例，singletonFactories三级缓存里获取。由于三级缓存存放着产生bean实例的工厂类，因此可以通过该工厂类产生bean实例。  
这里可以调用工厂类暴露的getObject方法返回早期暴露对象引用，也是我们所说的半成品bean，也可以成为earlySingletonObject。并且将这个半成品bean放到二级缓存里，在三级缓存里删除该bean。什么时候这个半成品填充了属性以后，就被移动到一级缓存中，也就是被作为可以使用的已经完成初始化的实例bean了，处理循环依赖的过程宣告完毕。

**11. java 什么情况下捕获不到Exception？**  
- 线程的异常：如果在一个线程中发生了未被捕获的异常，那么该线程将终止。在主线程中无法捕获子线程抛出的异常。可以使用Thread.UncaughtExceptionHandler接口来捕获子线程的异常。
- finally块中的异常：在finally块中抛出的异常将覆盖try块中抛出的异常。如果在finally块中抛出异常，那么该异常将被传递到调用finally块的方法中，并且try块中抛出的异常将被忽略。这是由于finally块中的异常总是被视为程序中最终的异常。
- 异常类之间有严格的集成关系，Throwable的两个子类Error和Exception，Exception的两个子类CheckedException和RuntimeException。通常捕获异常catch的时候最大catch到Exception这个类就为止了，当然这能够处理大部分的异常情况。但是值得注意的是，Exception不能捕捉到所有的异常。比如InvocationTargetException。像这类 java.lang.NoClassDefFoundError: org/apache/commons/collections4/IterableUtils，由于NoClassDefFoundError是Throwable的Error子类，所以Exception是捕捉不到的。
  
**12. java面向对象中的继承 有什么缺点？**  
Java面向对象中的继承是一种重要的特性，它允许一个类继承另一个类的属性和方法。虽然继承有很多优点，比如代码重用、代码组织、提高代码的可维护性和可扩展性等，但是继承也存在一些缺点，包括：

- 复杂性增加：继承层次结构越复杂，代码就越难以理解和维护。如果多个子类继承自同一个父类，那么对父类的修改可能会影响所有子类的行为。
- 耦合性增加：子类和父类之间的耦合性会增加，因为子类的实现依赖于父类的实现。这会增加代码的依赖性和难以维护性。
- 可重用性降低：如果子类只是为了继承而继承，那么它们可能会继承许多不需要的属性和方法，这会降低代码的可重用性。
- 父类改变时的影响：如果父类的实现发生改变，那么所有的子类都可能会受到影响，这会增加代码的脆弱性。
- 程序设计复杂性增加：当一个类有多个子类时，程序员必须考虑如何为每个子类实现正确的行为，这会增加程序设计的复杂性。
因此，在使用继承时，应该考虑这些缺点，以确保代码的可维护性和可扩展性。应该尽量避免使用过于复杂的继承层次结构，并且要确保子类只继承其需要的属性和方法。另外，可以考虑使用接口和组合等替代方案。  

**13. java的策略模式**  
Java的策略模式（Strategy Pattern）是一种行为设计模式，它允许你定义一系列算法，将每种算法都封装起来，并且使它们之间可以互换。  
在策略模式中，你可以定义一组算法类，这些算法类实现了一个公共的接口或者抽象类。在客户端代码中，你可以使用这些算法类来进行组合，从而达到不同的行为表现。  
以下是一个Java实现策略模式的例子：  
首先，我们定义一个策略接口：  
```
public interface PaymentStrategy {
    void pay(double amount);
}

```
然后，我们定义一些实现这个接口的策略类：  
```
public class CreditCardStrategy implements PaymentStrategy {
    private String name;
    private String cardNumber;
    private String cvv;
    private String expirationDate;
 
    public CreditCardStrategy(String name, String cardNumber, String cvv, String expirationDate) {
        this.name = name;
        this.cardNumber = cardNumber;
        this.cvv = cvv;
        this.expirationDate = expirationDate;
    }
 
    @Override
    public void pay(double amount) {
        System.out.println(amount + " paid with credit/debit card");
    }
}
 
public class PayPalStrategy implements PaymentStrategy {
    private String email;
    private String password;
 
    public PayPalStrategy(String email, String password) {
        this.email = email;
        this.password = password;
    }
 
    @Override
    public void pay(double amount) {
        System.out.println(amount + " paid with PayPal");
    }
}

```
接下来，我们定义一个Context类，用于操作不同的策略：  
```
public class ShoppingCart {
    private List<Item> items;
 
    public ShoppingCart() {
        this.items = new ArrayList<>();
    }
 
    public void addItem(Item item) {
        this.items.add(item);
    }
 
    public void removeItem(Item item) {
        this.items.remove(item);
    }
 
    public double calculateTotal() {
        double sum = 0;
        for (Item item : items) {
            sum += item.getPrice();
        }
        return sum;
    }
 
    public void pay(PaymentStrategy paymentStrategy) {
        double amount = calculateTotal();
        paymentStrategy.pay(amount);
    }
}

```
在这个例子中，我们的Context类是ShoppingCart。它使用了PaymentStrategy接口作为参数，这个参数可以是CreditCardStrategy或者PayPalStrategy。这种设计可以使得我们在运行时动态地改变算法。

最后，我们可以使用这些类来完成一个简单的购物车程序：
```
public static void main(String[] args) {
    ShoppingCart cart = new ShoppingCart();
    Item item1 = new Item("1234",10);
    Item item2 = new Item("5678",40);
    cart.addItem(item1);
    cart.addItem(item2);
    cart.pay(new PayPalStrategy("myemail@example.com", "mypwd"));
    cart.pay(new CreditCardStrategy("John Doe", "1234567890123456", "786", "12/15"));
}

```

**14. java的装饰模式**  
Java的装饰模式（Decorator Pattern）是一种结构性设计模式，它允许你在运行时动态地添加新的行为或功能到对象中，而不需要使用继承。

装饰模式的核心思想是将对象包装在一个装饰器对象中，这个装饰器对象可以对原始对象进行扩展，添加新的功能或者修改原始功能，同时保持接口的一致性。

以下是一个Java实现装饰模式的例子：  

首先，我们定义一个基础组件类：
```
public interface Car {
    void assemble();
}

```
然后，我们定义一些实现这个接口的具体组件类：
```
public class BasicCar implements Car {
    @Override
    public void assemble() {
        System.out.print("Basic Car.");
    }
}
 
public class SportsCar implements Car {
    @Override
    public void assemble() {
        System.out.print("Sports Car.");
    }
}
```
接下来，我们定义一个装饰器类，用于扩展基础组件类：
```
public class CarDecorator implements Car {
    protected Car car;
 
    public CarDecorator(Car car) {
        this.car = car;
    }
 
    @Override
    public void assemble() {
        this.car.assemble();
    }
}
```
在这个装饰器类中，我们使用了基础组件类作为成员变量，这个成员变量可以是任何实现了Car接口的具体组件类。然后，我们重写了assemble()方法，通过调用成员变量的assemble()方法来实现扩展。

最后，我们定义一些具体的装饰器类，用于添加新的功能：
```
public class AirConditionedCar extends CarDecorator {
    public AirConditionedCar(Car car) {
        super(car);
    }
 
    public void assemble() {
        super.assemble();
        System.out.print(" Adding air conditioning feature.");
    }
}
 
public class GPSNavigationCar extends CarDecorator {
    public GPSNavigationCar(Car car) {
        super(car);
    }
 
    public void assemble() {
        super.assemble();
        System.out.print(" Adding GPS navigation feature.");
    }
}
```
在这个例子中，我们定义了两个具体的装饰器类：AirConditionedCar和GPSNavigationCar，它们都继承了CarDecorator类，并重写了assemble()方法。在这个方法中，我们首先调用基础组件的assemble()方法，然后添加新的功能。

**15. 状态模式**  
状态模式（State Pattern）是一种行为型设计模式，它允许对象在其内部状态发生改变时改变其行为。状态模式将状态封装在一个对象内部，并将该对象的行为委托给表示当前状态的对象。这使得对象的行为能够根据内部状态的改变而改变，而无需在代码中使用大量的条件语句来判断对象当前的状态。

在Java中实现状态模式，一般需要定义一个状态接口或抽象类，来声明状态所具有的行为，然后针对不同的状态实现具体的状态类。同时，需要一个上下文类来维护对象的状态，并将状态的改变委托给具体的状态类。以下是一个简单的状态模式示例代码：
```
//定义状态接口
public interface State {
    void handle(Context context);
}

//具体状态类A
public class ConcreteStateA implements State {
    public void handle(Context context) {
        //具体实现A状态的行为
        context.setState(new ConcreteStateB()); //状态转换
    }
}

//具体状态类B
public class ConcreteStateB implements State {
    public void handle(Context context) {
        //具体实现B状态的行为
        context.setState(new ConcreteStateA()); //状态转换
    }
}

//上下文类
public class Context {
    private State state;

    public Context(State state) {
        this.state = state;
    }

    public void setState(State state) {
        this.state = state;
    }

    public void request() {
        state.handle(this); //委托状态对象处理请求
    }
}

//客户端代码
public class Client {
    public static void main(String[] args) {
        Context context = new Context(new ConcreteStateA());
        context.request(); //调用A状态的行为
        context.request(); //调用B状态的行为
    }
}
```
在这个例子中，State接口声明了状态所具有的行为，ConcreteStateA和ConcreteStateB实现了具体的状态类，并在处理请求时根据状态的不同实现了不同的行为。Context类维护了当前的状态，并将请求委托给具体的状态类来处理。客户端代码创建了Context对象，并通过多次调用request方法来模拟状态的改变和行为的变化。

**16.  代理模式**  
代理模式（Proxy Pattern）是一种结构型设计模式，它允许通过代理对象控制对其它对象的访问。代理对象可以作为其它对象的接口，以便对外提供相同的服务或功能，同时还可以在访问其它对象时提供额外的控制或增强功能。

在Java中实现代理模式，一般需要定义一个抽象的Subject接口或类，用于声明代理对象和实际对象之间的公共行为。同时需要一个具体的RealSubject类，它实现了真正的业务逻辑。最后，需要一个代理类来实现Subject接口或继承Subject类，并在其中持有一个实际对象的引用，以便在需要时将请求转发给实际对象，或者在请求前或请求后进行一些额外的操作。以下是一个简单的代理模式示例代码：
```
//抽象主题接口
public interface Subject {
    void request();
}

//具体主题类
public class RealSubject implements Subject {
    public void request() {
        System.out.println("RealSubject is handling the request.");
    }
}

//代理类
public class Proxy implements Subject {
    private RealSubject realSubject;

    public void request() {
        if (realSubject == null) {
            realSubject = new RealSubject();
        }
        //在调用实际对象的方法前或后可以添加额外的逻辑
        System.out.println("Proxy is handling the request.");
        realSubject.request();
    }
}

//客户端代码
public class Client {
    public static void main(String[] args) {
        Subject proxy = new Proxy();
        proxy.request(); //调用代理对象的方法
    }
}
```
在这个例子中，Subject接口声明了代理对象和实际对象之间的公共行为，RealSubject实现了真正的业务逻辑。Proxy类实现了Subject接口，并持有一个RealSubject对象的引用，在处理请求时调用实际对象的方法前或后可以添加额外的逻辑。客户端代码创建了一个代理对象，并调用其request方法来处理请求。

需要注意的是，代理模式可能会引入一定的复杂度，因为代理对象需要维护一个实际对象的引用，并在代理对象的方法中将请求转发给实际对象。但是，代理模式可以有效地解耦客户端和实际对象之间的直接交互，同时还可以为实际对象提供额外的控制或增强功能。


**17. 适配器模式**
适配器模式（Adapter Pattern）是一种结构型设计模式，用于将一个类的接口转换成客户端所期望的另一个接口，以便两个不兼容的接口能够协同工作。适配器模式包含三个角色：目标接口、适配器和适配者。其中目标接口是客户端所期望的接口，适配器是将适配者的接口转换成目标接口的中间件，适配者是需要被转换的原有接口。

在Java中实现适配器模式，一般需要先定义一个目标接口，然后创建一个适配器类，实现目标接口，并持有一个适配者对象的引用，以便在需要时将目标接口的方法调用转发给适配者对象。适配器类还可以在转发方法前或后添加额外的逻辑，以便适应不同的需求。以下是一个简单的适配器模式示例代码：
```
//目标接口
public interface Target {
    void request();
}

//适配者类
public class Adaptee {
    public void specificRequest() {
        System.out.println("Adaptee's specific request.");
    }
}

//适配器类
public class Adapter implements Target {
    private Adaptee adaptee;

    public Adapter(Adaptee adaptee) {
        this.adaptee = adaptee;
    }

    public void request() {
        adaptee.specificRequest(); //调用适配者对象的方法
    }
}

//客户端代码
public class Client {
    public static void main(String[] args) {
        Adaptee adaptee = new Adaptee();
        Target target = new Adapter(adaptee);
        target.request(); //调用目标对象的方法
    }
}
```
在这个例子中，Target接口定义了客户端所期望的接口，Adaptee类是需要被转换的原有接口，Adapter类实现了目标接口，并持有一个Adaptee对象的引用，在处理请求时将目标接口的方法调用转发给适配者对象。客户端代码创建了一个Adaptee对象和一个Adapter对象，并将Adaptee对象传递给Adapter对象，以便将适配者的接口转换成目标接口。

需要注意的是，适配器模式可以有效地解决不兼容接口的问题，但是在实现适配器时需要考虑一些细节问题，例如如何处理适配器和适配者的关系、如何处理接口的兼容性等。另外，适配器模式也可能会引入一定的复杂度，因为需要增加一个适配器类来实现接口的转换。

**18. 组合模式**  
组合模式（Composite Pattern）是一种结构型设计模式，用于将对象组合成树形结构，以表示“部分-整体”的层次结构。组合模式通过让单个对象和组合对象具有相同的接口，来使得客户端无需区分单个对象和组合对象的不同，从而可以统一处理它们。

在Java中实现组合模式，需要先定义一个抽象的组件类，它是组合中所有对象的基类，声明了一些通用的操作方法，例如添加子节点、删除子节点、获取子节点等。然后定义叶子节点类和组合节点类，它们分别实现了组件类的操作方法。组合节点类还需要持有一个子节点列表，以便对子节点进行管理。以下是一个简单的组合模式示例代码：
```
//抽象组件类
public abstract class Component {
    protected String name;

    public Component(String name) {
        this.name = name;
    }

    public abstract void add(Component c);
    public abstract void remove(Component c);
    public abstract Component getChild(int index);
    public abstract void operation();
}

//叶子节点类
public class Leaf extends Component {
    public Leaf(String name) {
        super(name);
    }

    public void add(Component c) {
        throw new UnsupportedOperationException();
    }

    public void remove(Component c) {
        throw new UnsupportedOperationException();
    }

    public Component getChild(int index) {
        throw new UnsupportedOperationException();
    }

    public void operation() {
        System.out.println("Leaf " + name + " is operated.");
    }
}

//组合节点类
public class Composite extends Component {
    private List<Component> children = new ArrayList<>();

    public Composite(String name) {
        super(name);
    }

    public void add(Component c) {
        children.add(c);
    }

    public void remove(Component c) {
        children.remove(c);
    }

    public Component getChild(int index) {
        return children.get(index);
    }

    public void operation() {
        System.out.println("Composite " + name + " is operated.");
        for (Component c : children) {
            c.operation();
        }
    }
}

//客户端代码
public class Client {
    public static void main(String[] args) {
        Component root = new Composite("root");
        Component node1 = new Composite("node1");
        Component node2 = new Composite("node2");
        Component leaf1 = new Leaf("leaf1");
        Component leaf2 = new Leaf("leaf2");
        Component leaf3 = new Leaf("leaf3");

        root.add(node1);
        root.add(node2);
        node1.add(leaf1);
        node1.add(leaf2);
        node2.add(leaf3);

        root.operation();
    }
}
```
在这个例子中，Component类是抽象组件类，Leaf类是叶子节点类，Composite类是组合节点类，客户端代码创建了一棵树形结构，并调用根节点的operation()方法，从而操作整个树。需要注意的是，叶子节点类不能添加子节点、删除子节点和获取子节点，而组合节点类可以对子节点进行管理，并且在执行operation()方法时需要递归地调用子节点的operation()方法。

**19.Spring IOC容器Bean初始化流程**  
创建Bean的时序图：  
由于一个Bean的创建经历了若干个类，调用了很多方法，为了将这个过程理解清楚，这里根据图1的时序图给大家展开描述。这里涉及到几个主要调用的类，我们会根据重点，按照图上的红色数字标号进行讲解。

![image](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/09e32eccd40f47af824f13023fe5cbab~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.image)  
如图1 所示，按照红色标号的顺序如下：

- 首先通过ClassPathXmlApplicationContext获取要创建的Bean，这里的方法是getBean。
- 在AbstractBeanFactory中会通过调用doGetBean方法完成获取Bean的操作。
- 此时DefaultSingletonBeanResigstry会执行getSingleton通过单例的方式返回Bean。
- 在获取Bean以后会通过createBean方法调用AbstractAutowireCapableBeanFactory中的方法对Bean进行初始化。这里重要的三个方式分别是：doCreateBean、createBeanInstance、instantiateBean。从图上可以看出Bean的初始化主要过程都是在这个类中完成的。
- 紧接着通过instantiate调用SimpleInstantiationStrategy中的instantiateWithMethodInjection做初始化的操作。
- CglibSubclassingInstantiationStrategy作为CGLIB的代理实现策略类，通过CglibSubclassCreator.instantiate方法进行初始化，并且返回AbstractAutowireCapableBeanFactory。此时AbstractAutowireCapableBeanFactory会接着执行populateBean和applyPropertyValues方法对Bean进行填充，以及设置属性值。
- 期间会通过调用BeanDefinitionValueResolver中的resolveValueIfNecessary方法完成对BeanDefinition中的值填充。
- 以及调用AbstractPropertyAccessor中的setPropertyValues方法进行属性值设置。完成上述操作以后，在AbstractAutowireCapableBeanFactory中执行initializeBean方法返回给AbstractBeanFactory，然后由AbstractBeanFactory的getObjectForBeanInstance方法返回Bean给调用的客户端，完成Bean的初始化过程。

**20. springboot如何获取一个bean的加载时间和销毁时间** 
在Spring Boot中，可以通过实现BeanPostProcessor接口来获取Bean的加载时间和销毁时间。

获取Bean加载时间：  
在实现`BeanPostProcessor`接口时，可以在`postProcessBeforeInitialization`方法中记录Bean加载的时间：  
```
@Component
public class BeanLoadTimeLogger implements BeanPostProcessor {

    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        long startTime = System.nanoTime();
        System.out.println("Bean '" + beanName + "' is about to be initialized.");
        return bean;
    }

}
```
在上面的代码中，`postProcessBeforeInitialization`方法会在Bean初始化之前被调用，并且会记录Bean加载的时间。  
获取Bean销毁时间：  
同样，在实现`BeanPostProcessor`接口时，可以在`postProcessBeforeDestruction`方法中记录Bean销毁的时间：  
```
@Component
public class BeanLoadTimeLogger implements BeanPostProcessor {

    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        long startTime = System.nanoTime();
        System.out.println("Bean '" + beanName + "' is about to be initialized.");
        return bean;
    }

    @Override
    public void postProcessBeforeDestruction(Object bean, String beanName) throws BeansException {
        long endTime = System.nanoTime();
        System.out.println("Bean '" + beanName + "' is about to be destroyed.");
        System.out.println("Bean '" + beanName + "' was alive for " + (endTime - startTime) + " nanoseconds.");
    }

}
```
在上面的代码中，postProcessBeforeDestruction方法会在Bean销毁之前被调用，并且会记录Bean销毁的时间，然后计算Bean的存活时间。

需要注意的是，在使用BeanPostProcessor时，需要将实现类注册为Spring的Bean，例如使用@Component注解。同时，实现类中需要使用@Override注解覆盖BeanPostProcessor接口中的两个方法。

**21.sleep() 方法和 wait() 方法对比**  
*共同点* ：两者都可以暂停线程的执行。
*区别* ：  
- sleep() 方法没有释放锁，而 wait() 方法释放了锁 。
- wait() 通常被用于线程间交互/通信，sleep()通常被用于暂停执行。
- wait() 方法被调用后，线程不会自动苏醒，需要别的线程调用同一个对象上的 notify()或者 notifyAll() 方法。sleep()方法执行完成后，线程会自动苏醒，或者也可以使用 wait(long timeout) 超时后线程会自动苏醒。
- sleep() 是 Thread 类的静态本地方法，wait() 则是 Object 类的本地方法。

**22. 为什么 wait() 方法不定义在 Thread 中**  
wait() 是让获得对象锁的线程实现等待，会自动释放当前线程占有的对象锁。每个对象（Object）都拥有对象锁，既然要释放当前线程占有的对象锁并让其进入 WAITING 状态，自然是要操作对应的对象（Object）而非当前的线程（Thread）。

类似的问题：为什么 sleep() 方法定义在 Thread 中？

因为 sleep() 是让当前线程暂停执行，不涉及到对象类，也不需要获得对象锁。

**23. 可以直接调用 Thread 类的 run 方法吗？**  
new 一个 Thread，线程进入了新建状态。调用 start()方法，会启动一个线程并使线程进入了就绪状态，当分配到时间片后就可以开始运行了。 start() 会执行线程的相应准备工作，然后自动执行 run() 方法的内容，这是真正的多线程工作。 但是，直接执行 run() 方法，会把 run() 方法当成一个 main 线程下的普通方法去执行，并不会在某个线程中执行它，所以这并不是多线程工作。

总结： 调用 start() 方法方可启动线程并使线程进入就绪状态，直接执行 run() 方法的话不会以多线程的方式执行。

**24. 基本类型和包装类型**  
- 成员变量包装类型不赋值就是 null ，而基本类型有默认值且不是 null。
- 包装类型可用于泛型，而基本类型不可以。
- 基本数据类型的局部变量存放在 Java 虚拟机栈中的局部变量表中，基本数据类型的成员变量（未被 static 修饰 ）存放在 Java 虚拟机的堆中。包装类型属于对象类型，我们知道几乎所有对象实例都存在于堆中。
- 相比于对象类型， 基本数据类型占用的空间非常小。

*基本数据类型存放在栈中是一个常见的误区！ 基本数据类型的成员变量如果没有被 static 修饰的话（不建议这么使用，应该要使用基本数据类型对应的包装类型），就存放在堆中。*

**25. 包装类型的缓存机制**  
Java 基本数据类型的包装类型的大部分都用到了缓存机制来提升性能。  
Byte,Short,Integer,Long 这 4 种包装类默认创建了数值 [-128，127] 的相应类型的缓存数据，Character 创建了数值在 [0,127] 范围的缓存数据，Boolean 直接返回 True or False。  

*Integer 缓存源码*：
```
public static Integer valueOf(int i) {
    if (i >= IntegerCache.low && i <= IntegerCache.high)
        return IntegerCache.cache[i + (-IntegerCache.low)];
    return new Integer(i);
}
private static class IntegerCache {
    static final int low = -128;
    static final int high;
    static {
        // high value may be configured by property
        int h = 127;
    }
}
```
如果超出对应范围仍然会去创建新的对象，缓存的范围区间的大小只是在性能和资源之间的权衡。  
两种浮点数类型的包装类 Float,Double 并没有实现缓存机制。
下面的代码的输出结果是 true 还是 false 呢？
```
Integer i1 = 40;
Integer i2 = new Integer(40);
System.out.println(i1==i2);
```
`Integer i1=40` 这一行代码会发生装箱，也就是说这行代码等价于 `Integer i1=Integer.valueOf(40)` 。因此，i1 直接使用的是缓存中的对象。而`Integer i2 = new Integer(40)` 会直接创建新的对象。  
因此，答案是 false 。

**记住：所有整型包装类对象之间值的比较，全部使用 equals 方法比较。**

*自动装箱与拆箱*  
- 装箱：将基本类型用它们对应的引用类型包装起来；装箱其实就是调用了 包装类的valueOf()方法
- 拆箱：将包装类型转换为基本数据类型；拆箱其实就是调用了 xxxValue()方法。

**26. 事务传播行为**  
事务传播行为是为了解决业务层方法之间互相调用的事务问题。

- Propagation.REQUIRED：如果当前存在事务，则加入该事务，如果当前不存在事务，则创建一个新的事务。
- Propagation.SUPPORTS：如果当前存在事务，则加入该事务；如果当前不存在事务，则以非事务的方式继续运行。
- Propagation.MANDATORY：如果当前存在事务，则加入该事务；如果当前不存在事务，则抛出异常IllegalTransactionStateException。
- Propagation.REQUIRES_NEW：重新创建一个新的事务，如果当前存在事务，延缓当前的事务。
- Propagation.NESTED：如果没有，就新建一个事务；如果有，就在当前事务中嵌套其他事务。
- Propagation.NOT_SUPPORTED：以非事务的方式运行，如果当前存在事务，暂停当前的事务。
- Propagation.NEVER：以非事务的方式运行，如果当前存在事务，则抛出异常。

**27. spring transaction的隔离级别**  
- TransactionDefinition.ISOLATION_DEFAULT :使用后端数据库默认的隔离级别，MySQL 默认采用的 REPEATABLE_READ 隔离级别 Oracle 默认采用的 READ_COMMITTED 隔离级别.
- TransactionDefinition.ISOLATION_READ_UNCOMMITTED :最低的隔离级别，使用这个隔离级别很少，因为它允许读取尚未提交的数据变更，可能会导致脏读、幻读或不可重复读
- TransactionDefinition.ISOLATION_READ_COMMITTED : 允许读取并发事务已经提交的数据，可以阻止脏读，但是幻读或不可重复读仍有可能发生
- TransactionDefinition.ISOLATION_REPEATABLE_READ : 对同一字段的多次读取结果都是一致的，除非数据是被本身事务自己所修改，可以阻止脏读和不可重复读，但幻读仍有可能发生。
- TransactionDefinition.ISOLATION_SERIALIZABLE : 最高的隔离级别，完全服从 ACID 的隔离级别。所有的事务依次逐个执行，这样事务之间就完全不可能产生干扰，也就是说，该级别可以防止脏读、不可重复读以及幻读。但是这将严重影响程序的性能。通常情况下也不会用到该级别。

**28. 事务失效场景**  
- 注解导致的事务失效：
  - @Transactional 注解属性 propagation 设置错误，设置了以非事务的状态运行
  - @Transactional 注解属性 rollbackFor 设置错误，实际抛出的错误跟设置不一致
- 方法修饰符导致的事务失效
  - @Transactional 应用在非 public 修饰的方法上，在事务方法拦截执行中，非public方法不执行。
  - 在动态代理层面，若为也需要代理的方法为public才能正常代理。如JDK动态代理，通过接口代理，接口方法默认都是public。而Cglib基于类的代理中会默认判断是否为public方法。
- 同一个类中方法调用，导致@Transactional失效，在当前的bean中非事务方法调用事务方法为什么不生效？当进行方法拦截的时候，方法拦截器首先获取当前动态代理的对象所代理的原始对象。如果判断当前的方法比如save方法没有Advice(增强)，则直接调用原对象的方法，即这个时候调用的是FirstApp.save方法。
- 异常被 catch 导致@Transactional失效
- 数据库引擎不支持事务

**29. 对象的创建**  

Step1:类加载检查

虚拟机遇到一条 new 指令时，首先将去检查这个指令的参数是否能在常量池中定位到这个类的符号引用，并且检查这个符号引用代表的类是否已被加载过、解析和初始化过。如果没有，那必须先执行相应的类加载过程。

Step2:分配内存

在类加载检查通过后，接下来虚拟机将为新生对象分配内存。对象所需的内存大小在类加载完成后便可确定，为对象分配空间的任务等同于把一块确定大小的内存从 Java 堆中划分出来。分配方式有 “指针碰撞” 和 “空闲列表” 两种，选择哪种分配方式由 Java 堆是否规整决定，而 Java 堆是否规整又由所采用的垃圾收集器是否带有压缩整理功能决定。

内存分配的两种方式 ：

指针碰撞 ：
适用场合 ：堆内存规整（即没有内存碎片）的情况下。
原理 ：用过的内存全部整合到一边，没有用过的内存放在另一边，中间有一个分界指针，只需要向着没用过的内存方向将该指针移动对象内存大小位置即可。
使用该分配方式的 GC 收集器：Serial, ParNew
空闲列表 ：
适用场合 ： 堆内存不规整的情况下。
原理 ：虚拟机会维护一个列表，该列表中会记录哪些内存块是可用的，在分配的时候，找一块儿足够大的内存块儿来划分给对象实例，最后更新列表记录。
使用该分配方式的 GC 收集器：CMS
选择以上两种方式中的哪一种，取决于 Java 堆内存是否规整。而 Java 堆内存是否规整，取决于 GC 收集器的算法是"标记-清除"，还是"标记-整理"（也称作"标记-压缩"），值得注意的是，复制算法内存也是规整的。

内存分配并发问题

在创建对象的时候有一个很重要的问题，就是线程安全，因为在实际开发过程中，创建对象是很频繁的事情，作为虚拟机来说，必须要保证线程是安全的，通常来讲，虚拟机采用两种方式来保证线程安全：

CAS+失败重试： CAS 是乐观锁的一种实现方式。所谓乐观锁就是，每次不加锁而是假设没有冲突而去完成某项操作，如果因为冲突失败就重试，直到成功为止。虚拟机采用 CAS 配上失败重试的方式保证更新操作的原子性。
TLAB： 为每一个线程预先在 Eden 区分配一块儿内存，JVM 在给线程中的对象分配内存时，首先在 TLAB 分配，当对象大于 TLAB 中的剩余内存或 TLAB 的内存已用尽时，再采用上述的 CAS 进行内存分配

Step3:初始化零值

内存分配完成后，虚拟机需要将分配到的内存空间都初始化为零值（不包括对象头），这一步操作保证了对象的实例字段在 Java 代码中可以不赋初始值就直接使用，程序能访问到这些字段的数据类型所对应的零值。

Step4:设置对象头

初始化零值完成之后，虚拟机要对对象进行必要的设置，例如这个对象是哪个类的实例、如何才能找到类的元数据信息、对象的哈希码、对象的 GC 分代年龄等信息。 这些信息存放在对象头中。 另外，根据虚拟机当前运行状态的不同，如是否启用偏向锁等，对象头会有不同的设置方式。

Step5:执行 `init` 方法

在上面工作都完成之后，从虚拟机的视角来看，一个新的对象已经产生了，但从 Java 程序的视角来看，对象创建才刚开始，`init` 方法还没有执行，所有的字段都还为零。所以一般来说，执行 new 指令之后会接着执行 `init` 方法，把对象按照程序员的意愿进行初始化，这样一个真正可用的对象才算完全产生出来。


**30. 对象内存分配流程**  
java对象分配流程  
1. 首先运行栈上分配。编译器通过逃逸分析及标量替换，决定对象应该分配在栈上还是堆中。如果逃逸分析及标量替换其中一个未启用，则直接分配在堆中。如果决定分配在堆中，那么执行步骤2
2. 进行TLAB分配。如果TLAB的空间`TALB_TOP+SIZE <= TLAB_END`，对象可以直接分配在TLAB中，那么`TLAB_TOP`加上对象`SIZ`E进行位置移动。若不能执行步骤3
3. 重新申请一块TALB，并尝试存储对象。若对象过大仍无法存储在TLAB中，执行步骤4
4. 判断是否满足进入老年代的条件(`PretenureSizeThreshold`参数)，若满足直接进入老年代，不满足进行新生代分配
5. 将对象存储在新生代`Eden`中，`EDEN_TOP`指针移位。若新声代无法存储对象，执行`Young GC`，并尝试重新分配对象。
6. `Young GC` 后重新分配对象，若仍然无法分配。对象直接进入老年代。

**31. Java虚拟机内存调优**  
![参数1](https://raw.githubusercontent.com/rbmonster/file-storage/main/learning-note/learning/basic/jvmParameter.jpg)
堆参数
![参数2](https://raw.githubusercontent.com/rbmonster/file-storage/main/learning-note/learning/basic/jvmParameter.jpg)

*基本概念重述*  
跨代引用解决方案  

Card Table：中文翻译为卡表，主要是用来标记卡页的状态，每个卡表项对应一个卡页。当卡页中一个对象引用有写操作时，写屏障将会标记对象所在的卡表状态改为 dirty，卡表的本质是用来解决跨代引用的问题。

内存分配  
1. TLAB：Thread Local Allocation Buffer 的简写，基于 CAS 的独享线程(Mutator Threads)可以优先将对象分配在 Eden 中的一块内存，因为是 Java 线程独享的内存区没有锁竞争，所以分配速度更快，每个 TLAB 都是一个线程独享的。
2. CAS+失败重试

*工具整理*  

命令行终端  
- 标准终端类：jps、jinfo、jstat、jstack、jmap  
-  功能整合类：jcmd、vjtools、arthas、greys  

可视化界面  
- 简易：JConsole、JVisualvm、HA、GCHisto、GCViewer   
- 进阶：MAT、JProfiler  

命令行推荐 Arthas ，可视化界面推荐 JProfiler，此外还有一些在线的平台 gceasy、heaphero、fastthread ，美团内部的 Scalpel(一款自研的 JVM 问题诊断工具，暂时未开源)也比较好用。

*GC 调优目的*  
将转移到老年代的对象数量降低到最小，减少 GC 的执行时间。

*GC 调优策略*  
- 策略 1：将新对象预留在新生代，由于 Full GC 的成本远高于 Minor GC，因此尽可能将对象分配在新生代是明智的做法，实际项目中根据 GC 日志分析新生代空间大小分配是否合理，适当通过“-Xmn”命令调节新生代大小，最大限度降低新对象直接进入老年代的情况。
- 策略 2：大对象进入老年代，虽然大部分情况下，将对象分配在新生代是合理的。但是对于大对象这种做法却值得商榷，大对象如果首次在新生代分配可能会出现空间不足导致很多年龄不够的小对象被分配的老年代，破坏新生代的对象结构，可能会出现频繁的 full gc。因此，对于大对象，可以设置直接进入老年代(当然短命的大对象对于垃圾回收来说简直就是噩梦)。-XX:PretenureSizeThreshold 可以设置直接进入老年代的对象大小。
- 策略 3：合理设置进入老年代对象的年龄，-XX:MaxTenuringThreshold 设置对象进入老年代的年龄大小，减少老年代的内存占用，降低 full gc 发生的频率。
  > 为什么从Young GC的对象最多经历15次Young GC还存活就会进入Old区(年龄是可以调的，默认是15)hotspots的markword的图中，用了4个bit去表示分代年龄，那么能表示的最大范围就是0-15。
- 策略 4：设置稳定的堆大小，堆大小设置有两个参数：-Xms 初始化堆大小，-Xmx 最大堆大小。
- 策略5：注意： 如果满足下面的指标，则一般不需要进行 GC 优化：
  > MinorGC 执行时间不到50ms； Minor GC 执行不频繁，约10秒一次； Full GC 执行时间不到1s； Full GC 执行频率不算频繁，不低于10分钟1次。

*调优指标*  
- 延迟(Latency)：也可以理解为最大停顿时间，即垃圾收集过程中一次 STW 的最长时间，越短越好，一定程度上可以接受频次的增大，GC 技术的主要发展方向。
- 吞吐量(Throughput)：应用系统的生命周期内，由于 GC 线程会占用 Mutator 当前可用的 CPU 时钟周期，吞吐量即为 Mutator 有效花费的时间占系统总运行时间的百分比，例如系统运行了 100 min，GC 耗时 1 min，则系统吞吐量为 99%，吞吐量优先的收集器可以接受较长的停顿。
- 内存占用：java堆区所占的内存大小；

主要抓住两点:  
- 吞吐量: 吞吐量优先，意味着在单位时间内，STW的时间最短
- 延迟(暂停时间): 暂停时间优先，意味这尽可能让单次STW的时间最短

*调优的时机*  
虚拟机书籍提供的建议，如果满足下面的指标，则一般不需要进行 GC 优化：  
MinorGC 执行时间不到50ms； Minor GC 执行不频繁，约10秒一次；  
Full GC 执行时间不到1s； Full GC 执行频率不算频繁，不低于10分钟1次。  

不得不考虑进行JVM调优的是那些情况呢？  
1. Heap内存（老年代）持续上涨达到设置的最大内存值，正常超过75%以上还未进行回收的情况；
2. Full GC 次数频繁；
3. GC 停顿时间过长（超过1秒）；
4. 应用出现OutOfMemory 等内存异常；
5. 应用中有使用本地缓存且占用大量内存空间；
6. 系统吞吐量与响应性能不高或下降。

*问题排查思路*   
- 时序分析：先发生的事件是根因的概率更大，通过监控手段分析各个指标的异常时间点，还原事件时间线，如先观察到 CPU 负载高(要有足够的时间 Gap)，那么整个问题影响链就可能是：
  > CPU 负载高 -> 慢查询增多 -> GC 耗时增大 -> 线程Block增多 -> RT 上涨。
- 概率分析：使用统计概率学，结合历史问题的经验进行推断，由近到远按类型分析，如过往慢查的问题比较多，那么整个问题影响链就可能是：
  > 慢查询增多 -> GC 耗时增大 -> CPU 负载高 -> 线程 Block 增多 -> RT上涨。
- 实验分析：通过故障演练等方式对问题现场进行模拟，触发其中部分条件(一个或多个)，观察是否会发生问题，如只触发线程 Block 就会发生问题，那么整个问题影响链就可能是：
  > 线程Block增多 -> CPU 负载高 -> 慢查询增多 -> GC 耗时增大 -> RT 上涨。
- 反证分析：对其中某一表象进行反证分析，即判断表象的发不发生跟结果是否有相关性，例如我们从整个集群的角度观察到某些节点慢查和 CPU 都正常，但也出了问题，那么整个问题影响链就可能是：
  > GC 耗时增大 -> 线程 Block 增多 -> RT 上涨。

**32. Java内存模型JMM**  
![内存模型](https://raw.githubusercontent.com/rbmonster/file-storage/main/learning-note/learning/jvm/java-memory.jpg)

java内存模型是关注在虚拟机中把变量值存储到内存和从内存取出变量这样的底层细节。
> 此处的变量包括了实例字段、静态字段和构成数组对象的元素，但是不包括局部变量和方法参数。 
如果局部变量是一个reference类型，引用的对象在java堆中，但是reference本身在java栈的局部变量表中是线程私有。

java内存模型规定了所有变量都存储在主内存，每条线程还有自己的工作内存，线程的工作内存保存了被该线程使用的变量的主内存副本。线程对变量的所有操作都必须在工作内存中进行，而不能直接读写主内存的数据。不同线程的内存数据无法直接访问，均得通过主内存。
> 关于线程内存的复制，如果主内存有一个10MB的对象，线程会把对这个对象的引用、对象中在线程中的字段进行复制，但不会整个复制。

如果要把java的内存强行与Java内存区域做对应的话：
- 主内存 -> java 堆
- 工作内存 -> 虚拟机栈
- 从更基础层次上，主内存直接对应物理硬件的内存。

*volatile*  
性能：volatile变量的读操作性能与普通变量几乎没有差别，但是写操作可能会慢些，因为需要插入内存屏障指令来保证处理器不乱序执行。
> 当读一个 volatile 变量时, JMM 会把该线程对应的本地内存置为无效。线程接下来将从主内存中读取共享变量。
线程在【读取】共享变量时，会先清空本地内存变量值，再从主内存获取最新值
线程在【写入】共享变量时，不会把值缓存在寄存器或其他地方(就是刚刚说的所谓的「工作内存」)，而是会把值刷新回主内存

*保证变量可见性*  
定义：当一条线程修改了这个变量的值，新值对于其他线程是立即可见的。 正常的变量修改的过程中，工作内存与主内存同步是存在延迟的，也就出现了可见性的问题。
> volatile并发下是安全的？
不，只能保证取值的时候是线程安全，当获取值之后做操作如果不是原子操作，很可能变量已经被其他线程改变。如多线程对volatile变量符号运算。

*禁止指令重排序优化*  
定义：普通的变量只能保证代码执行完成获取的结果一致，而不能保证执行顺序与代码顺序一致。

例子：如下双重加锁的例子，volatile保证了变量初始化完立即刷新到主内存中，能让其他线程可以获取到。

防止指令重排uniqueInstance = new Singleton(); 这段代码其实是分为三步执行：

- 为uniqueInstance 分配内存空间
- 初始化 uniqueInstance
- 将 uniqueInstance 指向分配的内存地址 但是由于 JVM 具有指令重排的特性，执行顺序有可能变成 1>3>2。指令重排在单线程环境下不会出现问题，但是在多线程环境下会导致一个线程获得还没有初始化的实例。例如，线程 T1 执行了 1 和 3，此时 T2 调用 getUniqueInstance() 后发现 uniqueInstance 不为空，因此返回 uniqueInstance，但此时 uniqueInstance 还未被初始化。

*synchronized*  
定义：synchronized 关键字解决的是多个线程之间访问资源的同步性，synchronized关键字可以保证被它修饰的方法或者代码块在任意时刻只能有一个线程执行。

- 【进入】synchronized 块的内存语义是把在 synchronized 块内使用的变量从线程的工作内存中清除，从主内存中读取
- 【退出】synchronized 块的内存语义事把在 synchronized 块内对共享变量的修改刷新到主内存中

synchronized关键字用法及锁范围:
1. 对于普通实例方法，锁的是当前实例对象，通常指 this
2. 对于静态同步方法，锁的是当前类的 Class 对象，如 ThreeSync.class
3. 对于同步代码块，锁的是 synchronized 括号内的对象

**33. long 和 double的型变量的特殊规则**  
long和double的非原子协定：对于64位的数据类型long和double，虚拟机允许将没有被volatile修饰的64位数据的读写操作分为两次32位的操作来进行。

