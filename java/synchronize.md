# Java中的并发、同步、锁 #
## synchronized 关键字 ##
synchronized关键字可以用于修饰实例方法、静态方法、代码块；修饰实例方法时以当前对象作为锁对象，修饰静态方法时以当前类的class对象作为锁对象，修饰代码块时以任意对象作为锁对象。

**修饰实例方法：**此时新创建的线程必须以同一对象为锁对象，不然无法避免线程不安全问题。如下代码以不同对象新建线程，将会出现线程安全问题。

	public class AccountingSyncBad implements Runnable{
	    static int i=0;
	    public synchronized void increase(){
	        i++;
	    }
	    @Override
	    public void run() {
	        for(int j=0;j<1000000;j++){
	            increase();
	        }
	    }
	    public static void main(String[] args) throws InterruptedException {
	        //new新实例
	        Thread t1=new Thread(new AccountingSyncBad());
	        //new新实例
	        Thread t2=new Thread(new AccountingSyncBad());
	        t1.start();
	        t2.start();
	        //join含义:当前线程A等待thread线程终止之后才能从thread.join()返回
	        t1.join();
	        t2.join();
	        System.out.println(i);
	    }
	}

**修饰静态方法：**即使新建线程时传入不同的对象，线程安全仍能保障，因为此时以class为锁对象。

	public class AccountingSyncClass implements Runnable{
	    static int i=0;
	
	    /**
	     * 作用于静态方法,锁是当前class对象,也就是
	     * AccountingSyncClass类对应的class对象
	     */
	    public static synchronized void increase(){
	        i++;
	    }
	
	    /**
	     * 非静态,访问时锁不一样不会发生互斥
	     */
	    public synchronized void increase4Obj(){
	        i++;
	    }
	
	    @Override
	    public void run() {
	        for(int j=0;j<1000000;j++){
	            increase();
	        }
	    }
	    public static void main(String[] args) throws InterruptedException {
	        //new新实例
	        Thread t1=new Thread(new AccountingSyncClass());
	        //new心事了
	        Thread t2=new Thread(new AccountingSyncClass());
	        //启动线程
	        t1.start();t2.start();
	
	        t1.join();t2.join();
	        System.out.println(i);
	    }
	}
**修饰代码块：**可见减少同步区域的范围，将不需要同步执行的代码提到同步块外面。此时需要注意锁对象对于所有线程应该是唯一的。

**synchronized的底层实现原理**

synchronized在同步块中是通过进出管程实现，enterMoniter开始进入同步块以及exitMoniter退出同步块。通过反编译字节码文件可以看到显示的使用了enterMoniter和exitMoniter。

synchronized修饰方法的时候，是隐式的使用了enterMoniter和exitMoniter。通过一个标志位ACC_SYNCHRONIZED设置为1来标志这个是同步方法。

**JVM对synchronized的优化措施**

- 重量级锁：早期使用synchronized关键字，加的锁就是重量级锁。处于效率的考虑，引入了偏向所、轻量级锁。
- 偏向锁：如果一个线程获得了该锁，那么这个锁就偏向这个线程，当这个线程再次获取这个锁的时候，可以不进行同步操作，直接获取这个锁，从而省去了获取锁带来的开销。当其他线程尝试获取这个锁时，偏向锁失效，升级为轻量级锁。偏向锁适用于多线程竞争不激烈的场景，某些资源总是被某个线程使用。
- 轻量级锁：当偏向锁失效后，锁会升级为轻量级锁。
- 自旋锁：当轻量级锁失效后，为了避免线程之间的切换，线程获取轻量级锁失败后，将不会直接挂起，而是做几个空循环，空循环结束后，如果还不能获取到锁，那么将会挂起这个线程，锁升级为重量级锁。自选的理由是线程占用临界区的时间不会太久，通过自选可以避免线程的切换。

**应用层面对 锁 的优化措施**

- 锁消除：通过上下文扫描，消除不存在共享资源竞争的锁。如StringBuffer是线程安全的，如果StringBuffer定义为局部变量，由于局部变量不存在资源竞争，那么JVM将会消除Stringbuffer中的同步操作。
- 锁分离：如LinkedBlockingQueue中将读取元素的锁和加入元素的锁分离，读元素的同时可以加入元素，读写分离互不影响。
- 减少锁的粒度：如ConcurrentHashMap将整个hashMap分层几个部分，每个部分分别持有读写所，读不同段的读写可以同步进行。
- 锁粗化：避免反复的获取锁释放所的过程，使用一个锁将几个同步块合并在一起。

**synchronized 锁时可重入的**

一个线程持有了synchronized锁后，再次请求这个锁，是允许的。如下同步块和同步方法均是以this为锁对象。由于synchronized底层是通过Moniter实现的，因此每一次获取这个锁后，计数器都会加1.释放锁的时候，申请了多少次锁就需要释放多少次，直到计数器为0；

	public class AccountingSync implements Runnable{
	    static AccountingSync instance=new AccountingSync();
	    static int i=0;
	    static int j=0;
	    @Override
	    public void run() {
	        for(int j=0;j<1000000;j++){
	
	            //this,当前实例对象锁
	            synchronized(this){
	                i++;
	                increase();//synchronized的可重入性
	            }
	        }
	    }
	
	    public synchronized void increase(){
	        j++;
	    }
	
	
	    public static void main(String[] args) throws InterruptedException {
	        Thread t1=new Thread(instance);
	        Thread t2=new Thread(instance);
	        t1.start();t2.start();
	        t1.join();t2.join();
	        System.out.println(i);
	    }
	}

**线程中断**

关于线程中断有三个方法需要注意：Thread类的成员方法interrupt()方法和isInterrupted()方法，其中interrupt()方法用来中断一个线程，只有当这个线程被阻塞了或者尝试做阻塞操作时，才可以被中断，否则中断不会响应。调用interrupt()方法后会抛出一个InterruptedException，并且中断位会被重置为非中断状态。isInterrupted()方法用于判断一个线程是否被中断。还有一个方法是Thread类的静态方法interrupted()方法，这个方法用于判断这个线程是否被中断，并且将中断位重置为非中断状态。

	public class InterruputThread {
	    public static void main(String[] args) throws InterruptedException {
	        Thread t1=new Thread(){
	            @Override
	            public void run(){
	                while(true){
	                    System.out.println("未被中断");
	                }
	            }
	        };
	        t1.start();
	        TimeUnit.SECONDS.sleep(2);
	        t1.interrupt();
	
	        /**
	         * 输出结果(无限执行):
	             未被中断
	             未被中断
	             未被中断
	             ......
	         */
	    }
	}
如上述代码尝试中断一个没有阻塞的线程，将不会得到响应。但是我们可以通过isInterrupted()方法判断中断位来处理这个中断请求，interrupt()方法会将中断位置为true。

	public class InterruputThread {
	    public static void main(String[] args) throws InterruptedException {
	        Thread t1=new Thread(){
	            @Override
	            public void run(){
	                while(true){
	                    //判断当前线程是否被中断
	                    if (this.isInterrupted()){
	                        System.out.println("线程中断");
	                        break;
	                    }
	                }
	
	                System.out.println("已跳出循环,线程中断!");
	            }
	        };
	        t1.start();
	        TimeUnit.SECONDS.sleep(2);
	        t1.interrupt();
	
	        /**
	         * 输出结果:
	            线程中断
	            已跳出循环,线程中断!
	         */
	    }
	}

**线程中断与synchronized**

当一个线程在等待synchronized锁的时候，调用interrupt()方法将不会得到响应。

**等待唤醒机制与synchronized**

等待唤醒机制主要指的是wait()、notify()、notifyAll()方法。这三个方法必须在synchronized同步块中或者同步方法中使用，因为这三个方法都依赖于Moniter，而synchronized关键字是获取Moniter的。

wait()方法与sleep()方法的区别在于wait方法会释放锁，而sleep方法不会释放锁。

**Java中锁的类型**
- 乐观锁：假定冲突不会发生，只在提交结果时才会检测数据完整性是否被破坏（使用版本号或时间戳）----无锁
- 悲观锁：认为冲突一定会发生，在操作前屏蔽一切可能违反数据完整性的操作。                  ----加锁
- 共享锁
- 排它锁