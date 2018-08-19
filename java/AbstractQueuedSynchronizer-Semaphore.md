# Java信号量的实现类Semaphore #
## Semaphore的基本方法 ##
Semaphore是Java中信号量的实现类，用来定义一个资源的数量，当可用资源数大于1时，允许多个线程同时访问，当没有可用资源时，新来的线程将进入等待。
	
	//创建具有给定的许可数和非公平的公平设置的Semaphore。
	Semaphore(int permits) 
	
	//创建具有给定的许可数和给定的公平设置的Semaphore，true即为公平锁     
	Semaphore(int permits, boolean fair) 
	
	//从此信号量中获取许可，不可中断
	void acquireUninterruptibly() 
	
	//返回此信号量中当前可用的许可数。      
	int availablePermits() 
	
	//获取并返回立即可用的所有许可。    
	int drainPermits() 
	
	//返回一个 collection，包含可能等待获取的线程。       
	protected  Collection<Thread> getQueuedThreads();
	
	//返回正在等待获取的线程的估计数目。        
	int getQueueLength() 
	
	//查询是否有线程正在等待获取。       
	boolean hasQueuedThreads() 
	
	//如果此信号量的公平设置为 true，则返回 true。          
	boolean isFair() 
	
	//仅在调用时此信号量存在一个可用许可，才从信号量获取许可。          
	boolean tryAcquire() 
	
	//如果在给定的等待时间内，此信号量有可用的许可并且当前线程未被中断，则从此信号量获取一个许可。        
	boolean tryAcquire(long timeout, TimeUnit unit) 

Semaphore在结构上和ReentrantLock类几乎一致，Semaphore内部同样定义了三个内部类，一个Sync是抽象类，并且继承了AbstractQueuedSynchronizer抽象类，一个NonFairSync继承了Sync，是非公平锁的实现类，一个FairSync继承了Sync，是公平锁的实现类。
Semaphore同样是基于AQS基础框架实现的，Semaphore的所有acquire方法和release方法最终都会委托给tryAcquireShare()方法和tryReleaseShare()方法，这两个方法都是AQS中的模板方法，由子类提供具体的实现。（这一点和ReentrantLock的各种lock方法和unlock方法最终都会委托给tryAcquire()方法和tryRelease()方法一样）。

## 非公平锁acquire()获取资源的过程--可中断 ##
	//默认创建公平锁，permits指定同一时间访问共享资源的线程数
	public Semaphore(int permits) {
	        sync = new NonfairSync(permits);
	    }
Semaphore会默认创建非公平锁，传入的int参数指定可以同时访问的线程数，这个int值也会设置给state

	//Semaphore中的获取资源方法
    public void acquire() throws InterruptedException {
        sync.acquireSharedInterruptibly(1);
    }
acquire()方法获取一个资源，acquire()方法是可被中断的，会调用AQS的acquireSharedInterruptibly()方法

	//AQS中的方法
    public final void acquireSharedInterruptibly(int arg)
            throws InterruptedException {
        if (Thread.interrupted())
            throw new InterruptedException();
        if (tryAcquireShared(arg) < 0)
            doAcquireSharedInterruptibly(arg);
    }
由于acquire方法是可中断的，acquireSharedInterruptibly()遇到中断就会抛出异常，然后委托给tryAcquireShared()方法，尝试获取一个资源，tryAcquireShared()方法是AQS中的模板方法，会调用子类NonFairSync的tryAcquireShared()方法

		//NonFairSync中重写的tryAcquireShared方法
        protected int tryAcquireShared(int acquires) {
            return nonfairTryAcquireShared(acquires);
        }

tryAcquireShared()方法会调用父类Sync的nonfairTryAcquireShared方法

		//Sync类的非公平获取锁方式
        final int nonfairTryAcquireShared(int acquires) {
            for (;;) {
                int available = getState();
                int remaining = available - acquires;
                if (remaining < 0 ||
                    compareAndSetState(available, remaining))
                    return remaining;
            }
        }
nonfairTryAcquireShared()方法在死循环中保证CAS执行完成或者可用资源数小于0.如果CAS执行成功，那么将返回一个大于等于0的数，此时AQS的acquireSharedInterruptibly()方法中的条件判断tryAcquireShared(arg) < 0将不满足，线程成功获取到资源；如果返回一个小于0的数，那么条件判断满足，将执行AQS中的doAcquireSharedInterruptibly()方法

	//AQS的方法
    private void doAcquireSharedInterruptibly(int arg) throws InterruptedException {
        final Node node = addWaiter(Node.SHARED);
        boolean failed = true;
        try {
            for (;;) {
                final Node p = node.predecessor();
                if (p == head) {
                    int r = tryAcquireShared(arg);
                    if (r >= 0) {
                        setHeadAndPropagate(node, r);
                        p.next = null; // help GC
                        failed = false;
                        return;
                    }
                }
                if (shouldParkAfterFailedAcquire(p, node) && parkAndCheckInterrupt())
                    throw new InterruptedException();
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
    }
doAcquireSharedInterruptibly()方法首先调用addWaiter()方法将线程以共享模式SHARED封装为Node节点加入到同步队列中，如果当前节点的前驱节点是head，那么线程自旋调用tryAcquireShared方法尝试获取资源，如果成功获取到资源，那么tryAcquireShared()方法会返回一个大于等于0的int值，成功获取资源后会将当前节点设置为head节点，并且如果可用资源数大于0，将会唤醒后继节点setHeadAndPropagate(node, r);如果前驱节点不是head，那么判断是否需要挂起当前线程，如果遇到中断，将直接抛出中断异常。

## 非公平锁获取资源acquireUninterruptibly()的过程---不可中断 ##
    //Semaphore的不可中断获取资源方法
	public void acquireUninterruptibly() {
        sync.acquireShared(1);
    }

	//AQS中的acquireShared()方法，没有中断检查，同样委托给tryAcquireShared()方法尝试获取资源
	//如果成功获取资源，那么方法结束，如果没有获取到资源
    public final void acquireShared(int arg) {
        if (tryAcquireShared(arg) < 0)
            doAcquireShared(arg);
    }

	//AQS中的不可中断的获取资源方法，遇到中断不会抛出异常，这是与doAcquireSharedInterruptibly()方法的唯一区别
    private void doAcquireShared(int arg) {
        final Node node = addWaiter(Node.SHARED);
        boolean failed = true;
        try {
            boolean interrupted = false;
            for (;;) {
                final Node p = node.predecessor();
                if (p == head) {
                    int r = tryAcquireShared(arg);
                    if (r >= 0) {
                        setHeadAndPropagate(node, r);
                        p.next = null; // help GC
                        if (interrupted)
                            selfInterrupt();
                        failed = false;
                        return;
                    }
                }
                if (shouldParkAfterFailedAcquire(p, node) &&
                    parkAndCheckInterrupt())
                    interrupted = true;
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
    }

## 非公平锁释放资源release()的过程 ##
	//Semaphore中的release()方法
    public void release() {
        sync.releaseShared(1);
    }

	//AQS中的方法，委托给tryReleaseShared()方法，这是个模板方法，会调用子类Sync中的重写的方法
    public final boolean releaseShared(int arg) {
        if (tryReleaseShared(arg)) {
            doReleaseShared();
            return true;
        }
        return false;
    }

	//不管是公平锁还是非公平锁，释放资源的逻辑是一样的，因此这个方法定义在Sync类中，
	//在死循环中保证CAS执行成功，增加state的值，释放资源后调用doReleaseShared()方法
        protected final boolean tryReleaseShared(int releases) {
            for (;;) {
                int current = getState();
                int next = current + releases;
                if (next < current) // overflow
                    throw new Error("Maximum permit count exceeded");
                if (compareAndSetState(current, next))
                    return true;
            }
        }
	
	private void doReleaseShared() {
	    /* 
	     * 保证释放动作(向同步等待队列尾部)传递，即使没有其他正在进行的  
	     * 请求或释放动作。如果头节点的后继节点需要唤醒，那么执行唤醒  
	     * 动作；如果不需要，将头结点的等待状态设置为PROPAGATE保证   
	     * 唤醒传递。另外，为了防止过程中有新节点进入(队列)，这里必  
	     * 需做循环，所以，和其他unparkSuccessor方法使用方式不一样  
	     * 的是，如果(头结点)等待状态设置失败，重新检测。 
	     */  
	    for (;;) {
	        Node h = head;
	        if (h != null && h != tail) {
	            // 获取头节点对应的线程的状态
	            int ws = h.waitStatus;
	            // 如果头节点对应的线程是SIGNAL状态，则意味着头
	            //结点的后继结点所对应的线程需要被unpark唤醒。
	            if (ws == Node.SIGNAL) {
	                // 修改头结点对应的线程状态设置为0。失败的话，则继续循环。
	                if (!compareAndSetWaitStatus(h, Node.SIGNAL, 0))
	                    continue;
	                // 唤醒头结点h的后继结点所对应的线程
	                unparkSuccessor(h);
	            }
	            else if (ws == 0 &&
	                     !compareAndSetWaitStatus(h, 0, Node.PROPAGATE))
	                continue;                // loop on failed CAS
	        }
	        // 如果头结点发生变化，则继续循环。否则，退出循环。
	        if (h == head)                   // loop if head changed
	            break;
	    }
	}
doReleaseShared()方法的主要作用是唤醒后继节点.唤醒的线程重新回到被挂起的地方开始执行,即doAcquireShared()方法中开始自旋获取资源。

## 公平锁获取资源acquire()的过程 ##
        protected int tryAcquireShared(int acquires) {
            for (;;) {
                if (hasQueuedPredecessors())
                    return -1;
                int available = getState();
                int remaining = available - acquires;
                if (remaining < 0 ||
                    compareAndSetState(available, remaining))
                    return remaining;
            }
        }
唯一的区别在于FairSync重写的tryAcquireShared()方法，会先判断是否有线程hasQueuedPredecessors()在等待获取资源，如果有，那么直接将这个线程封装成Node节点插入到同步队列的尾部。