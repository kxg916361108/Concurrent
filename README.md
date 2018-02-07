# Concurrent
A)、活跃性问题：活跃性问题的形式之一就是无意中造成的无限循环，从而使循环之后的代码无法得到执行，如：线程A等待B释放其持有的资源，而线程B永远都不释放
该资源，那么A就会永远地等待下去。
2、线程安全性 
  a)、无状态对象一定是线程安全的(既不包含任何域，也不包含任何对其他类中域的引用)。
  b)、竞态条件，当某个计算的正确性取决于多个线程的交替执行时序时，那么就会发生竞态条件。最常见的竞态条件就是“先检查，后执行”操作------基于一种可能
  失效的观察结果来做出判断或者执行某个操作。(线程安全的类需要清除竞态条件)。
  c)、要保持状态的一致性，就需要在单个原子操作中更新所有相关的状态变量。
  d)、内置锁(监视器锁):Java对象被用做一个实现同步的锁，被成为内置锁。线程在进入同步代码块之前会自动获得锁，并在退出同步代码块时自动释放锁，而无论是
  通过正常的控制路径退出，还是通过从代码中抛出异常退出。获得锁的唯一途径就是进入这个锁保护的同步代码块或方法。
  synchronized(lock) {
  }
  两部分组成，一个是作为锁的对象引用。一个是作为由锁保护的代码块。内置锁是一种互斥锁。内置锁是可重入锁。
  e)、对于每个包含多个变量的不变性条件，其中涉及的所有变量都需要由同一个锁来保护。
  f)、虽然synchronized方法可以确保单个操作的原子性，但如果把多个操作合并为一个复合操作，还是需要额外的加锁机制。此外将每个方法都作为同步方法还可能导致
  活跃性问题或者性能问题。
  g)当执行时间较长的计算或者可能无法快速完成的操作时(例如，网络I/O或者控制台I/O)，一定不要持有锁。
  h)、synchronized可以保证原子性、确定临界区和保证可见性。

3、对象的共享
  a)、可见性：通常我们无法确保执行读操作的线程能够适时地看到其他线程写入的值，优势甚至是根本不可能的事。为了确保多个线程之间对内存写入操作的可见性，必须
  使用同步机制。
  b)、volatile 变量不会被缓存在寄存器或者对其他处理器不可见的地方，因此在读取volatile类型的变量时总会返回最新写入的值。volatile变量正确的使用方式包
  括:确保它们自身状态的可见性，确保它们所引用对象的状态的可见性，以及标识一些重要的程序生命周期时间的发生。(volatile变量只能保证可见性)
  3.2、发布和逸出
    发布一个对象指使对象能够在当前作用域之外的代码中使用。如：将一个指向该对象的引用保存到其他代码可以访问的地方，或者在某个非私有的方法中返回该引用，或
    者将引用传递到其他类的方法中。
    public static Set<Secret> knowSecrets;
    //可以被其他代码访问的公有方法。
    public void init() {
      knowSecrets = new HashSet<>();
    }
    当某个不该被发布的对象被发布时，就称为逸出。
构造过程中this引用的逸出。
public class ThisEscape {
　　public ThisEscape(EventSource source) {
　　　　source.registerListener(new EventListener() {
　　　　　　public void onEvent(Event e) {
　　　　　　　　doSomething(e);
　　　　　　}
　　　　});
　　}
　　void doSomething(Event e) {
　　}
　　interface EventSource {
　　　　void registerListener(EventListener e);
　　}
　　interface EventListener {
　　　　void onEvent(Event e);
　　}
　　interface Event {
　　}
}
这将导致this逸出，所谓逸出，就是在不该发布的时候发布了一个引用。在这个例子里面，当我们实例化ThisEscape对象时，会调用source的registerListener
方法，这时便启动了一个线程，而且这个线程持有了ThisEscape对象（调用了对象的doSomething方法），但此时ThisEscape对象却没有实例化完成（还没有返回
一个引用），所以我们说，此时造成了一个this引用逸出，即还没有完成的实例化ThisEscape对象的动作，却已经暴露了对象的引用。其他线程访问还没有构造好的
对象，可能会造成意料不到的问题。
最后，书里面给出了正确构造过程：
public class SafeListener {
　　private final EventListener listener;

　　private SafeListener() {
　　　　listener = new EventListener() {
　　　　　　public void onEvent(Event e) {
　　　　　　　　doSomething(e);
　　　　　　}
　　　　};
　　}
　　public static SafeListener newInstance(EventSource source) {
　　　　SafeListener safe = new SafeListener();
　　　　source.registerListener(safe.listener);
　　　　return safe;
　　}
　　void doSomething(Event e) {
　　}
　　interface EventSource {
　　　　void registerListener(EventListener e);
　　}
　　interface EventListener {
　　　　void onEvent(Event e);
　　}
　　interface Event {
　　}
　}
在这个构造中，我们看到的最大的一个区别就是：当构造好了SafeListener对象（通过构造器构造）之后，我们才启动了监听线程，也就确保了SafeListener对象是
构造完成之后再使用的SafeListener对象。
对于这样的技术，书里面也有这样的注释：
具体来说，只有当构造函数返回时，this引用才应该从线程中逸出。构造函数可以将this引用保存到某个地方，只要其他线程不会在构造函数完成之前使用它。


  
  
