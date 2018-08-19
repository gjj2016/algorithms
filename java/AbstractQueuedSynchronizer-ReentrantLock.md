# Java中的显示加锁、释放锁 #
## 接口支持 java.util.concurrent.locks.Lock ##
Lock接口中提供了如下方法供灵活的显示加锁和释放锁

	public interface Lock {
	    //加锁
	    void lock();
	
	    //解锁
	    void unlock();
	
	    //可中断获取锁，与lock()不同之处在于可响应中断操作，即在获
	    //取锁的过程中可中断，注意synchronized在获取锁时是不可中断的
	    void lockInterruptibly() throws InterruptedException;
	
	    //尝试非阻塞获取锁，调用该方法后立即返回结果，如果能够获取则返回true，否则返回false
	    boolean tryLock();
	
	    //根据传入的时间段获取锁，在指定时间内没有获取锁则返回false，如果在指定时间内当前线程未被中并断获取到锁则返回true
	    boolean tryLock(long time, TimeUnit unit) throws InterruptedException;
	
	    //获取等待通知组件，该组件与当前锁绑定，当前线程只有获得了锁
	    //才能调用该组件的wait()方法，而调用后，当前线程将释放锁。
	    Condition newCondition();
	｝

## 构建锁和同步组件的基础框架----抽象类 AbstractQueuedSynchronizer ##
AbstractQueuedSynchronizer是构建锁和同步组件的基础框架，AbstractQueuedSynchronizer本身是一个抽象类，但是没有一个抽象方法。AbstractQueuedSynchronizer定义的是实现锁的基本方法，具体的实现方式交给子类具体的视线，因为共享锁和排他锁在具体实现上是不同的，但是基本思想是通用，把通用的基本部分在AbstractQueuedSynchronizer类中实现，把不同的实现部分定义为**模板方法**供子类重写，这种设计模式称为**模板模式。**

**AQS的内部机理：**AQS内部维护着两类队列，第一类队列是同步队列，同步队列是一个双向链表，同步队列维护的是请求锁的线程队列，当线程请求锁而锁已经被占用时，该线程会被封装称一个Node节点（Node是AQS的内部类），Node节点的信息有：指向前驱节点的指针、该线程的状态、线程、指向后继节点的指针。封装成Node节点后会插入到同步队列的队尾。第二类队列是在某个Condition上阻塞的线程，即线程试图获取某个资源（Condition）时，该资源已经没有了，线程调用wait方法阻塞，阻塞的线程会被封装称一个Node节点插入到这个Condition的等待队列的队尾。

	public abstract class AbstractQueuedSynchronizer extends AbstractOwnableSynchronizer{
		//指向同步队列队头
		private transient volatile Node head;
		
		//指向同步的队尾
		private transient volatile Node tail;
		
		//同步状态，0代表锁未被占用，1代表锁已被占用
		private volatile int state;
		
		//AQS的模板方法
		
		//独占模式下获取锁的方法
	    protected boolean tryAcquire(int arg) {
	        throw new UnsupportedOperationException();
	    }
	
	    //独占模式下解锁的方法
	    protected boolean tryRelease(int arg) {
	        throw new UnsupportedOperationException();
	    }
	
	    //共享模式下获取锁的方法
	    protected int tryAcquireShared(int arg) {
	        throw new UnsupportedOperationException();
	    }
	
	    //共享模式下解锁的方法
	    protected boolean tryReleaseShared(int arg) {
	        throw new UnsupportedOperationException();
	    }
	    //判断是否为持有独占锁
	    protected boolean isHeldExclusively() {
	        throw new UnsupportedOperationException();
	    }
	}

同步队列上head是一个哑元，而tail指向队列的最后一个元素。

	static final class Node {
	    //共享模式
	    static final Node SHARED = new Node();
	    //独占模式
	    static final Node EXCLUSIVE = null;
	
	    //标识线程已处于结束状态
	    static final int CANCELLED =  1;
	    //等待被唤醒状态
	    static final int SIGNAL    = -1;
	    //条件状态，
	    static final int CONDITION = -2;
	    //在共享模式中使用表示获得的同步状态会被传播
	    static final int PROPAGATE = -3;
	
	    //等待状态,存在CANCELLED、SIGNAL、CONDITION、PROPAGATE 4种
	    volatile int waitStatus;
	
	    //同步队列中前驱结点
	    volatile Node prev;
	
	    //同步队列中后继结点
	    volatile Node next;
	
	    //请求锁的线程
	    volatile Thread thread;
	
	    //等待队列中的后继结点，这个与Condition有关，稍后会分析
	    Node nextWaiter;
	
	    //判断是否为共享模式
	    final boolean isShared() {
	        return nextWaiter == SHARED;
	    }
	
	    //获取前驱结点
	    final Node predecessor() throws NullPointerException {
	        Node p = prev;
	        if (p == null)
	            throw new NullPointerException();
	        else
	            return p;
	    }
	
	    //.....
	}

Node节点主要由四个部分构成：prev、waitStatus、thread、next。waitStatus是线程状态的主要标志位，当waitStatus=CANCELLED，表示这个线程在等待锁的过程中被中断或者超时了；waitStatus=SIGNAL表示这个线程在等待锁；waitStatus=CONDITION表示这个线程刚刚从等待队列中移到同步队列上开始等待获取锁，在同步队列上遇上waitStatus=CONDITION需要把waitStatus设置为SIGNAL；waitStatus=PROPAGATE是共享锁中的。

## Lock的实现类----ReentrantLock ##
ReentrantLock实现了Lock接口，定义了一套排他锁的加锁和释放锁的规则。

	//查询当前线程保持此锁的次数。
	int getHoldCount() 
	
	//返回目前拥有此锁的线程，如果此锁不被任何线程拥有，则返回 null。      
	protected  Thread   getOwner(); 
	
	//返回一个 collection，它包含可能正等待获取此锁的线程，其内部维持一个队列，这点稍后会分析。      
	protected  Collection<Thread>   getQueuedThreads(); 
	
	//返回正等待获取此锁的线程估计数。   
	int getQueueLength();
	
	// 返回一个 collection，它包含可能正在等待与此锁相关给定条件的那些线程。
	protected  Collection<Thread>   getWaitingThreads(Condition condition); 
	
	//返回等待与此锁相关的给定条件的线程估计数。       
	int getWaitQueueLength(Condition condition);
	
	// 查询给定线程是否正在等待获取此锁。     
	boolean hasQueuedThread(Thread thread); 
	
	//查询是否有些线程正在等待获取此锁。     
	boolean hasQueuedThreads();
	
	//查询是否有些线程正在等待与此锁有关的给定条件。     
	boolean hasWaiters(Condition condition); 
	
	//如果此锁的公平设置为 true，则返回 true。     
	boolean isFair() 
	
	//查询当前线程是否保持此锁。      
	boolean isHeldByCurrentThread() 
	
	//查询此锁是否由任意线程保持。        
	boolean isLocked() 

ReentrantLock中的所有方法都依靠其内部类Sync实现， Sync是一个抽象类，继承自AbstractQueuedSynchronizer。 
    
	//默认构造，创建非公平锁NonfairSync
	public ReentrantLock() {
	    sync = new NonfairSync();
	}
	//根据传入参数创建锁类型
	public ReentrantLock(boolean fair) {
	    sync = fair ? new FairSync() : new NonfairSync();
	}
	
	//加锁操作
	public void lock() {
	     sync.lock();
	}
ReentrantLock默认创建的是非公平锁，也可以通过构造时传入true来创建公平所。ReentrantLock的所有方法都由Sync这个类实现，而Sync这个抽象类又有两个子类，分别是NonFairSync和FairSync，这两个类都是ReentrantLock的内部类，分别对应非公平锁和公平锁的实现。

## 非公平锁的lock()加锁过程 ##
	public void lock() {
	        sync.lock();
	}
调用Sync这个抽象类中的lock()方法

	abstract void lock();
Sync类中lock()方法为抽象方法，因为非公平锁和公平锁获取锁的方式不同，Sync类中没有给出lock()方法的具体实现，以非公平锁为例，会调用NonFairSync类的lock()方法

	final void lock() {
	            if (compareAndSetState(0, 1))
	                setExclusiveOwnerThread(Thread.currentThread());
	            else
	                acquire(1);
	}
先通过CAS操作获取锁，如果获取成功，那么把自己设置为锁的持有者；如果获取失败，那么进入acquire(int args)方法，这个方法在AQS中实现

	public final void acquire(int arg) {
	        if (!tryAcquire(arg) && acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
	            selfInterrupt();
	}
先尝试获取锁，调用tryAcquire()方法，而tryAcquire()方法是模板方法，因为排它锁和共享锁的获取锁的方式不同，所以这个方法只能由子类重写，因此会调用NonFairSync这个类中的tryAcquire()方法

	protected final boolean tryAcquire(int acquires) {
	            return nonfairTryAcquire(acquires);
	        }
调用的是Sync类中的nonfairTryAcquire()方法
	
	final boolean nonfairTryAcquire(int acquires) {
	            final Thread current = Thread.currentThread();
	            int c = getState();
	            if (c == 0) {
	                if (compareAndSetState(0, acquires)) {
	                    setExclusiveOwnerThread(current);
	                    return true;
	                }
	            }
	            else if (current == getExclusiveOwnerThread()) {
	                int nextc = c + acquires;
	                if (nextc < 0) // overflow
	                    throw new Error("Maximum lock count exceeded");
	                setState(nextc);
	                return true;
	            }
	            return false;
	        }
nonfairTryAcquire()方法很简单，先尝试CAS获取锁，如果获取成功，那么把自己设置为锁的持有者；如果获取失败，那么看锁的持有者是否是自己，如果是，那么属于重入锁，将state加1.如果tryAcquire()方法成功，那么acquire()方法结束，已经获得了锁；如果tryAcquire()方法失败，那么将会执行acquireQueued(addWaiter(Node.EXCLUSIVE), arg)

	private Node addWaiter(Node mode) {
	    //将请求同步状态失败的线程封装成结点
	    Node node = new Node(Thread.currentThread(), mode);
	
	    Node pred = tail;
	    //如果是第一个结点加入肯定为空，跳过。
	    //如果非第一个结点则直接执行CAS入队操作，尝试在尾部快速添加
	    if (pred != null) {
	        node.prev = pred;
	        //使用CAS执行尾部结点替换，尝试在尾部快速添加
	        if (compareAndSetTail(pred, node)) {
	            pred.next = node;
	            return node;
	        }
	    }
	    //如果第一次加入或者CAS操作没有成功执行enq入队操作
	    enq(node);
	    return node;
	}
其中addWaiter()方法将当前线程封装称模式为EXCLUSIVE的Node节点插入到同步队列的末尾。如果队列不为空，尝试插入到tail后面，然后通过CAS操作修改tail指针（因为这里存在锁个线程同时修改tail指针的情况），如果队列不为空或者CAS修改tail指针失败，那么执行enq()方法通过死循环将Node节点插入到队尾并CAS修改head指针和tail指针。使用死循环是为了一直执行CAS知道成功修改head指针和tail指针。

	final boolean acquireQueued(final Node node, int arg) {
	    boolean failed = true;
	    try {
	        boolean interrupted = false;
	        //自旋，死循环
	        for (;;) {
	            //获取前驱结点
	            final Node p = node.predecessor();
	            //当且仅当p为头结点才尝试获取同步状态
	            if (p == head && tryAcquire(arg)) {
	                //将node设置为头结点
	                setHead(node);
	                //清空原来头结点的引用便于GC
	                p.next = null; // help GC
	                failed = false;
	                return interrupted;
	            }
	            //如果前驱结点不是head，判断是否挂起线程
	            if (shouldParkAfterFailedAcquire(p, node) &&
	                parkAndCheckInterrupt())
	                interrupted = true;
	        }
	    } finally {
	        if (failed)
	            //最终都没能获取同步状态，结束该线程的请求
	            cancelAcquire(node);
	    }
	}
而acquireQueued()方法让线程在死循环中尝试获取锁。如果当前线程的前驱节点为head节点（head节点是一个哑元），这个线程才会去执行tryAcquire()方法尝试获取锁，这符合先进先出的调度法。这个线程会在死循环中一直调用tryAcquire()方法；对于前驱节点不是head节点的线程以及tryAcquire()方法获取锁失败的前驱节点为head的线程，都会判断是否需要挂起线程

	private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
	        //获取当前结点的等待状态
	        int ws = pred.waitStatus;
	        //如果为等待唤醒（SIGNAL）状态则返回true
	        if (ws == Node.SIGNAL)
	            return true;
	        //如果ws>0 则说明是结束状态，
	        //遍历前驱结点直到找到没有结束状态的结点
	        if (ws > 0) {
	            do {
	                node.prev = pred = pred.prev;
	            } while (pred.waitStatus > 0);
	            pred.next = node;
	        } else {
	            //如果ws小于0又不是SIGNAL状态，
	            //则将其设置为SIGNAL状态，代表该结点的线程正在等待唤醒。
	            compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
	        }
	        return false;
	    }
	
	private final boolean parkAndCheckInterrupt() {
	        //将当前线程挂起
	        LockSupport.park(this);
	        //获取线程中断状态,interrupted()是判断当前中断状态，
	        //并非中断线程，因此可能true也可能false,并返回
	        return Thread.interrupted();
	}
shouldParkAfterFailedAcquire()根据waitStatus判断是否需要挂起线程，如果当前节点的前驱节点的waitStatus为SIGNAL，表示前一个线程正在等待锁，那么当前线程会被挂起LockSupport.park(this);如果当前节点的前驱节点的waitStatus为CANCELLED，表示前一个节点已经失效，则一直往前找到一个没有失效的节点，并将这个节点设置为自己的前驱节点；如果当前节点的前驱节点的waitStatus为CONDITION，表示这个节点是从等待队列过来的，需要将其waitStatus设置为SIGNAL。

## 非公平锁的可中断加锁----lockInterruptibly（） ##
基于AQS的锁时可中断的，但是synchronized的等待锁的过程是不可中断的。

	 public void lockInterruptibly() throws InterruptedException {
	        sync.acquireInterruptibly(1);
	    }
ReentrantLock中的lockInterruptibly()方法会调用AQS的acquireInterruptibly()方法

	 public final void acquireInterruptibly(int arg)
	            throws InterruptedException {
	        if (Thread.interrupted())
	            throw new InterruptedException();
	        if (!tryAcquire(arg))
	            doAcquireInterruptibly(arg);
	    }
acquireInterruptibly()方法先尝试获取锁，获取失败调用AQS的doAcquireInterruptibly()方法

    private void doAcquireInterruptibly(int arg) throws InterruptedException {
        final Node node = addWaiter(Node.EXCLUSIVE);
        boolean failed = true;
        try {
            for (;;) {
                final Node p = node.predecessor();
                if (p == head && tryAcquire(arg)) {
                    setHead(node);
                    p.next = null; // help GC
                    failed = false;
                    return;
                }
                if (shouldParkAfterFailedAcquire(p, node) &&
                    parkAndCheckInterrupt())
					//被中断之后直接抛出异常
                    throw new InterruptedException();
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
    }
doAcquireInterruptibly()方法与不可中断的获取锁方法acquireQueued()十分相似，只是被中断之后直接抛出异常，将当前节点从同步队列中移除。

## 非公平锁的释放锁的过程----unlock() ##
	//ReentrantLock类的unlock
	public void unlock() {
	    sync.release(1);
	}
调用的是AQS中的release()方法

	//AQS类的release()方法
	public final boolean release(int arg) {
	    //尝试释放锁
	    if (tryRelease(arg)) {
	
	        Node h = head;
	        if (h != null && h.waitStatus != 0)
	            //唤醒后继结点的线程
	            unparkSuccessor(h);
	        return true;
	    }
	    return false;
	}
release()方法会先调用tryRelease()方法尝试释放锁，而tryRelease()方法是模板方法，由子类Sync重写了，由于公平锁和非公平锁的释放锁的过程是一样的，因此在Sync中给出了tryRelease()方法的实现

	//ReentrantLock类中的内部类Sync实现的tryRelease(int releases) 
	protected final boolean tryRelease(int releases) {

      int c = getState() - releases;
      if (Thread.currentThread() != getExclusiveOwnerThread())
          throw new IllegalMonitorStateException();
      boolean free = false;
      //判断状态是否为0，如果是则说明已释放同步状态
      if (c == 0) {
          free = true;
          //设置Owner为null
          setExclusiveOwnerThread(null);
      }
      //设置更新同步状态
      setState(c);
      return free;
  	}
由于ReentrantLock是重入锁，因此可能会多次调用unlock()方法直到state设置为0，锁被完全释放后，会去唤醒后继节点

	private void unparkSuccessor(Node node) {
	    //这里，node一般为当前线程所在的结点。
	    int ws = node.waitStatus;
	    if (ws < 0)//置零当前线程所在的结点状态，允许失败。
	        compareAndSetWaitStatus(node, ws, 0);
	
	    Node s = node.next;//找到下一个需要唤醒的结点s
	    if (s == null || s.waitStatus > 0) {//如果为空或已取消
	        s = null;
	        for (Node t = tail; t != null && t != node; t = t.prev)
	            if (t.waitStatus <= 0)//从这里可以看出，<=0的结点，都是还有效的结点。
	                s = t;
	    }
	    if (s != null)
	        LockSupport.unpark(s.thread);//唤醒
	}
unparkSuccessor()方法唤醒的一定是同步队列中最前面还没放弃的节点，其waitStatus是<=0的，这个节点可能不是head的next节点，但一定是第一个还有效的节点。这个线程被唤醒之后，将回到acquireQueued()方法里（在哪里被挂起，唤醒之后就在哪里），被唤醒的线程继续自旋，执行死循环，此时这个线程已经是前驱节点为head的节点了，将开始尝试获取锁。

## 公平锁获取锁----lock() ##
	public void lock() {
	        sync.lock();
	}
调用Sync这个抽象类中的lock()方法

	abstract void lock();
Sync类中lock()方法为抽象方法，因为非公平锁和公平锁获取锁的方式不同，Sync类中没有给出lock()方法的具体实现，公平锁中会调用FairSync类的lock()方法

        final void lock() {
            acquire(1);
        }
还是调用AQS的acquire()方法

    public final void acquire(int arg) {
        if (!tryAcquire(arg) &&
            acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
            selfInterrupt();
    }
同样的，还是先调用tryAcquire()方法尝试获取锁，而tryAcquire()方法是模板方式，此时会调用FairSync类重写的tryAcquire()

        protected final boolean tryAcquire(int acquires) {
            final Thread current = Thread.currentThread();
            int c = getState();
            if (c == 0) {
                if (!hasQueuedPredecessors() &&
                    compareAndSetState(0, acquires)) {
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
            else if (current == getExclusiveOwnerThread()) {
                int nextc = c + acquires;
                if (nextc < 0)
                    throw new Error("Maximum lock count exceeded");
                setState(nextc);
                return true;
            }
            return false;
        }
    }
与非公平锁的唯一区别在于如果此时锁没有被占用，会先判断同步队列是否为空，如果同步队列不为空，则需要放弃获取锁加入到同步队列末尾，如果同步队列为空，才会尝试获取锁。除此之外，其他的操作全部使用AQS中的方法，因为其他操作与非公平锁时一样的。

## 等待唤醒机制----Condition接口 ##
	public interface Condition {
	
	 /**
	  * 使当前线程进入等待状态直到被通知(signal)或中断
	  * 当其他线程调用singal()或singalAll()方法时，该线程将被唤醒
	  * 当其他线程调用interrupt()方法中断当前线程
	  * await()相当于synchronized等待唤醒机制中的wait()方法
	  */
	 void await() throws InterruptedException;
	
	 //当前线程进入等待状态，直到被唤醒，该方法不响应中断要求
	 void awaitUninterruptibly();
	
	 //调用该方法，当前线程进入等待状态，直到被唤醒或被中断或超时
	 //其中nanosTimeout指的等待超时时间，单位纳秒
	 long awaitNanos(long nanosTimeout) throws InterruptedException;
	
	  //同awaitNanos，但可以指明时间单位
	  boolean await(long time, TimeUnit unit) throws InterruptedException;
	
	 //调用该方法当前线程进入等待状态，直到被唤醒、中断或到达某个时
	 //间期限(deadline),如果没到指定时间就被唤醒，返回true，其他情况返回false
	  boolean awaitUntil(Date deadline) throws InterruptedException;
	
	 //唤醒一个等待在Condition上的线程，该线程从等待方法返回前必须
	 //获取与Condition相关联的锁，功能与notify()相同
	  void signal();
	
	 //唤醒所有等待在Condition上的线程，该线程从等待方法返回前必须
	 //获取与Condition相关联的锁，功能与notifyAll()相同
	  void signalAll();
	}
Condition接口中定义了await()方法和signal()方法，其作用和wait()方法及notify()相同。但是在使用上更加灵活，Condition接口的一个实例对应一个等待队列，可以定义多个等待队列，如生产者和消费者场景中，可以为阻塞的生产者线程定义一个等待队列，同时为阻塞的消费者线程定义一个等待队列，那么在唤醒线程的时候，可以唤醒特性的线程。在使用synchronized同步的时候，唤醒线程的时候唤醒的是全部的阻塞线程，本来生产者想唤醒的是消费者，而由于唤醒了全部的线程，生产者线程也被唤醒了，在使用上没有等待队列灵活。

## await()和signal()----ConditionObject类 ##
ConditionObject实现了Condition接口，是AQS的内部类，与AQS维护的等待队列密切相关。

	 public class ConditionObject implements Condition, java.io.Serializable {
	    //等待队列第一个等待结点
	    private transient Node firstWaiter;
	    //等待队列最后一个等待结点
	    private transient Node lastWaiter;
	    //省略其他代码.......
	}
等待队列上的节点也是被封装成Node节点的，但是节点的内容不太一样。Node：waitStatus、Thread、nextWaiter。其中waitStatus是当前线程的状态，在等待队列上只可能去两个值，一是CONDITION，二是CANCELED；Thread表示当前线程；nextWaiter表示下一个节点；等待队列与同步队列不同，等待队列是单链表，firstWaiter指向第一个节点，第一个节点就是可用的节点，lastWaiter指向最后一个节点。

	public final void await() throws InterruptedException {
	      //判断线程是否被中断
	      if (Thread.interrupted())
	          throw new InterruptedException();
	      //创建新结点加入等待队列并返回
	      Node node = addConditionWaiter();
	      //释放当前线程锁即释放同步状态
	      int savedState = fullyRelease(node);
	      int interruptMode = 0;
	      //判断结点是否同步队列(SyncQueue)中,即是否被唤醒
	      while (!isOnSyncQueue(node)) {
	          //挂起线程
	          LockSupport.park(this);
	          //判断是否被中断唤醒，如果是退出循环。
	          if ((interruptMode = checkInterruptWhileWaiting(node)) != 0)
	              break;
	      }
	      //被唤醒后执行自旋操作争取获得锁，同时判断线程是否被中断
	      if (acquireQueued(node, savedState) && interruptMode != THROW_IE)
	          interruptMode = REINTERRUPT;
	       // clean up if cancelled
	      if (node.nextWaiter != null) 
	          //清理等待队列中不为CONDITION状态的结点
	          unlinkCancelledWaiters();
	      if (interruptMode != 0)
	          reportInterruptAfterWait(interruptMode);
	  }
当Condition对象调用await()方法后，会先调用addConditionWaiter()方法，将这个线程封装成Node节点，waitStatus设置为CONDITION，然后插入到等待队列的队尾；然后调用fullyRelease()释放锁，这里是完全释放锁，即调用tryRealease()方法将state设置为0；之后调用isOnSyncQueue()判断这个节点是否在同步队列上，注意这里循环判断这个节点是否在同步队列上，如果不在同步队列里，那么这个线程会被挂起LockSupport.park(this);

	public final void signal() {
	     //判断是否持有独占锁，如果不是抛出异常
	   if (!isHeldExclusively())
	          throw new IllegalMonitorStateException();
	      Node first = firstWaiter;
	      //唤醒等待队列第一个结点的线程
	      if (first != null)
	          doSignal(first);
	 }
signal()方法会先判断是否是独占锁，如果不是独占锁，那么将会直接抛出异常，这也说明了共享锁是不能使用Condition的。然后调用dosignal()方法

	 private void doSignal(Node first) {
	     do {
	             //移除条件等待队列中的第一个结点，
	             //如果后继结点为null，那么说没有其他结点将尾结点也设置为null
	            if ( (firstWaiter = first.nextWaiter) == null)
	                 lastWaiter = null;
	             first.nextWaiter = null;
	          //如果被通知节点没有进入到同步队列并且条件等待队列还有不为空的节点，则继续循环通知后续结点
	         } while (!transferForSignal(first) &&
	                  (first = firstWaiter) != null);
	        }
	
	//transferForSignal方法
	final boolean transferForSignal(Node node) {
	    //尝试设置唤醒结点的waitStatus为0，即初始化状态
	    //如果设置失败，说明当期结点node的waitStatus已不为
	    //CONDITION状态，那么只能是结束状态了，因此返回false
	    //返回doSignal()方法中继续唤醒其他结点的线程，注意这里并
	    //不涉及并发问题，所以CAS操作失败只可能是预期值不为CONDITION，
	    //而不是多线程设置导致预期值变化，毕竟操作该方法的线程是持有锁的。
	    if (!compareAndSetWaitStatus(node, Node.CONDITION, 0))
	         return false;
	
	        //加入同步队列并返回前驱结点p
	        Node p = enq(node);
	        int ws = p.waitStatus;
	        //判断前驱结点是否为结束结点(CANCELLED=1)或者在设置
	        //前驱节点状态为Node.SIGNAL状态失败时，唤醒被通知节点代表的线程
	        if (ws > 0 || !compareAndSetWaitStatus(p, ws, Node.SIGNAL))
	            //唤醒node结点的线程
	            LockSupport.unpark(node.thread);
	        return true;
	    }
doSignal()方法中会获取等待队列上的第一个节点，并将这个节点加入到同步队列中，并唤醒这个线程。被唤醒的线程会重新回到await()方法的while循环中（在哪里被挂起，被唤醒之后就在哪里），此时节点已经在同步队列上，那么跳出while循环，执行acquireQueued()方法，开始在同步队列上和其他线程一样的竞争锁。