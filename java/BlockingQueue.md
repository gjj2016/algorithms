# 阻塞队列 #
[http://http://blog.csdn.net/javazejian/article/details/77410889 ](http://http://blog.csdn.net/javazejian/article/details/77410889 )
## 阻塞队列的接口支持BlockingQueue ##
BlockingQueue继承于Queue接口，与平常使用的队列（如ArrayDeque）的最大区别在于，ArrayDeque在head==tail！=null即队列满了之后，会自动申请扩容，阻塞队列当队列为空时，执行删除操作，那么执行删除操作的线程会被一直阻塞至队列不为空；如果阻塞队列满了，执行添加操作，那么执行添加操作的线程会被一直阻塞至队列有空余空间；即阻塞队列支持阻塞添加和阻塞删除。
	
	public interface BlockingQueue<E> extends Queue<E> {
	    //返回 true 或者抛出异常
	    boolean add(E e);

		//返回true 或者 false
	    boolean offer(E e);

		//返回 true 或者一直阻塞至有可用空间
	    void put(E e) throws InterruptedException;
	
		//返回 true 或者阻塞一段时间
	    boolean offer(E e, long timeout, TimeUnit unit) throws InterruptedException;

		//返回 队首元素 或者一直阻塞至有元素
	    E take() throws InterruptedException;

		//返回 队首元素 或者阻塞一段时间	
	    E poll(long timeout, TimeUnit unit) throws InterruptedException;
	
		//返回 true 或者 false
	    boolean remove(Object o);

		//返回 队首元素 或者 null
		E poll();

		//返回 队首元素 或者 抛出异常
		E element();

		//返回 队首元素 或者 null
		E peek();
	}
在BlockingQueue中take()方法、put()方法以及poll(time)、offer(time)方法提供了阻塞添加和阻塞删除的功能。

## 阻塞队列的实现类----ArrayBlockingQueue ##
	public class ArrayBlockingQueue<E> extends AbstractQueue<E>
	        implements BlockingQueue<E>, java.io.Serializable {
	
	    /** 存储数据的数组 */
	    final Object[] items;
	
	    /**获取数据的索引，主要用于take，poll，peek，remove方法 */
	    int takeIndex;
	
	    /**添加数据的索引，主要用于 put, offer, or add 方法*/
	    int putIndex;
	
	    /** 队列元素的个数 */
	    int count;
	
	
	    /** 控制并非访问的锁 */
	    final ReentrantLock lock;
	
	    /**notEmpty条件对象，用于通知take方法队列已有元素，可执行获取操作 */
	    private final Condition notEmpty;
	
	    /**notFull条件对象，用于通知put方法队列未满，可执行添加操作 */
	    private final Condition notFull;
	
	    /**
	       迭代器
	     */
	    transient Itrs itrs = null;
	
	}
ArrayBlockingQueue底层通过ReentrantLock以及Condition实现，想要操作队列（不管是入队或者出队），都需要先获取lock锁，根据是否可被中断可以调用ReentrantLock类的lock()方法或者lockInterruptibly()方法；往阻塞队列中添加元素时，如果队列已满，那么线程会被阻塞在notFull这个阻塞队列上；往阻塞队列上删除元素时，如果队列为空，那么线程会被阻塞在notEmpty这个阻塞队列上。ArrayBlockingQueue底层通过循环数组实现，putIndex用于指示添加元素的下标，takeIndex用于指示删除元素的下标。

## ArrayBlockingQueue(阻塞)添加元素过程 ##
	//add方法实现，间接调用了offer(e)
	public boolean add(E e) {
	        if (offer(e))
	            return true;
	        else
	            throw new IllegalStateException("Queue full");
	    }
	
	//offer方法
	public boolean offer(E e) {
	     checkNotNull(e);//检查元素是否为null
	     final ReentrantLock lock = this.lock;
	     lock.lock();//加锁
	     try {
	         if (count == items.length)//判断队列是否满
	             return false;
	         else {
	             enqueue(e);//添加元素到队列
	             return true;
	         }
	     } finally {
	         lock.unlock();
	     }
	 }
	
	//入队操作
	private void enqueue(E x) {
	    //获取当前数组
	    final Object[] items = this.items;
	    //通过putIndex索引对数组进行赋值
	    items[putIndex] = x;
	    //索引自增，如果已是最后一个位置，重新设置 putIndex = 0;
	    if (++putIndex == items.length)
	        putIndex = 0;
	    count++;//队列中元素数量加1
	    //唤醒调用take()方法的线程，执行元素获取操作。
	    notEmpty.signal();
	}
add()方法和offer()方法不是阻塞添加方法，方法会立即返回；add()方法委托给offer()方法实现，offer()方法往队列中添加元素先判断元素是否为null，阻塞队列中不允许放入null元素，调用lock()方法获取不可中断的锁，在enqueue()方法中完成插入元素操作，如果putIndex==items.length，那么重新将putIndex置为0，并且唤醒take线程，即唤醒阻塞在notEmpty阻塞队列上的线程。

	//put方法，阻塞时可中断
	 public void put(E e) throws InterruptedException {
	     checkNotNull(e);
	      final ReentrantLock lock = this.lock;
	      lock.lockInterruptibly();//该方法可中断
	      try {
	          //当队列元素个数与数组长度相等时，无法添加元素
	          while (count == items.length)
	              //将当前调用线程挂起，添加到notFull条件队列中等待唤醒
	              notFull.await();
	          enqueue(e);//如果队列没有满直接添加。。
	      } finally {
	          lock.unlock();
	      }
	  }
put()方法执行的是阻塞添加过程，当队列满了之后，当前线程会被阻塞在notFull等待队列上。直到被唤醒，也就是队列不为空了，调用enqueue()方法完成元素插入。

## ArrayBlockingQueue(阻塞)删除的过程 ##
	public E poll() {
	      final ReentrantLock lock = this.lock;
	       lock.lock();
	       try {
	           //判断队列是否为null，不为null执行dequeue()方法，否则返回null
	           return (count == 0) ? null : dequeue();
	       } finally {
	           lock.unlock();
	       }
	    }
	 //删除队列头元素并返回
	 private E dequeue() {
	     //拿到当前数组的数据
	     final Object[] items = this.items;
	      @SuppressWarnings("unchecked")
	      //获取要删除的对象
	      E x = (E) items[takeIndex];
	      //将数组中takeIndex索引位置设置为null
	      items[takeIndex] = null;
	      //takeIndex索引加1并判断是否与数组长度相等，
	      //如果相等说明已到尽头，恢复为0
	      if (++takeIndex == items.length)
	          takeIndex = 0;
	      count--;//队列个数减1
	      if (itrs != null)
	          itrs.elementDequeued();//同时更新迭代器中的元素数据
	      //删除了元素说明队列有空位，唤醒notFull条件对象添加线程，执行添加操作
	      notFull.signal();
	      return x;
	    }
poll()方法非阻塞的移除队首元素、移除成功或会唤醒阻塞在notFull等待队列上的线程，此时队列已经不是满的了，put线程会唤醒可以回到put()方法中继续执行

	public boolean remove(Object o){
		//	ArrayBlockingQueue元素不为空
		if(o == null){
			return false;
		}
		final Object[] items = this.items;
		final ReentrantLock lock = this.lock;
		lock.lock();
		try{
			if(count > 0){
				final int putIndex = this.putIndex;
				int i = this.takeIndex;
				//从takeIndex一直找到putIndex，找到待删除元素的下标
				do{
					if(o.equals(items[i])){
						removeAt(i);
						return true;
					}else{
						if(++i == items.length){
							i = 0;
						}
					}
				}while(i != putIndex);
			} 
			return false;
			
		} finally{
			lock.unlock();
		}
	}
	//执行删除指定下标的元素
	private void removeAt(final int removeIndex){
		final Object[] items = this.items;
		if(removeIndex == takeIndex){
			//如果删除的队首元素，直接删除即可
			items[taleIndex] = null;
			if(++takeIndex == items.length){
				takeIndex = 0;
			}
			count--;	
		}else {
			//如果删除的不是队首元素，那么需要移动元素
			final int putIndex = this.putIndex;
			int i = removeIndex;
			while(true){
				int next = i+1;
				if(next == items.length){
					next = 0;
				}
				if(next != putIndex){
					//还没移到最后一个元素
					items[i] = items[next];
					i = next;
				}else {
					//移到了最后一个元素了，修改putIndex
					items[i] = null;
					this.putIndex = i;
					break;
				}
			}
			count--;
		}
		//删除了一个元素，队列不满了，唤醒put线程
		notFull.signal();
	}
remove()方法和poll()方法都是非阻塞方法，方法会立即返回。

	//从队列头部删除，队列没有元素就阻塞，可中断
	 public E take() throws InterruptedException {
	    final ReentrantLock lock = this.lock;
	      lock.lockInterruptibly();//中断
	      try {
	          //如果队列没有元素
	          while (count == 0)
	              //执行阻塞操作
	              notEmpty.await();
	          return dequeue();//如果队列有元素执行删除操作
	      } finally {
	          lock.unlock();
	      }
	    }
take()方法是阻塞删除方法，当队列为空时，会把当前线程阻塞在notEmpty等待队列上。

## 阻塞队列的实现类----LinkedBlockingQueue ##
	public class LinkedBlockingQueue<E> extends AbstractQueue<E>
	        implements BlockingQueue<E>, java.io.Serializable {
	
	    /**
	     * 节点类，用于存储数据
	     */
	    static class Node<E> {
	        E item;
	
	        /**
	         * One of:
	         * - the real successor Node
	         * - this Node, meaning the successor is head.next
	         * - null, meaning there is no successor (this is the last node)
	         */
	        Node<E> next;
	
	        Node(E x) { item = x; }
	    }
	
	    /** 阻塞队列的大小，默认为Integer.MAX_VALUE */
	    private final int capacity;
	
	    /** 当前阻塞队列中的元素个数 */
	    private final AtomicInteger count = new AtomicInteger();
	
	    /**
	     * 阻塞队列的头结点
	     */
	    transient Node<E> head;
	
	    /**
	     * 阻塞队列的尾节点
	     */
	    private transient Node<E> last;
	
	    /** 获取并移除元素时使用的锁，如take, poll, etc */
	    private final ReentrantLock takeLock = new ReentrantLock();
	
	    /** notEmpty条件对象，当队列没有数据时用于挂起执行删除的线程 */
	    private final Condition notEmpty = takeLock.newCondition();
	
	    /** 添加元素时使用的锁如 put, offer, etc */
	    private final ReentrantLock putLock = new ReentrantLock();
	
	    /** notFull条件对象，当队列数据已满时用于挂起执行添加的线程 */
	    private final Condition notFull = putLock.newCondition();
	
	}
LinkedBlockingQueue使用单链表维护了一个阻塞队列，这个阻塞队列的默认大小为Integer.MAX_VALUE，在构造阻塞队列的时候强烈建议传入一个int值设置阻塞队列的大小。除此之外，阻塞队列内部有两个锁，分别为takeLock控制移除元素的锁，putLock控制插入元素的锁，也就是LinkedBlockingQueue中移除元素和插入元素时可以同时进行的（ArrayBlockingQueue中移除元素和插入元素使用的是同一个锁，因此不能同时进行移除和插入），takeLock锁创建了notEmpty这个等待队列，当队列为空时take线程会被阻塞在notEmpty等待队列上；putLock锁创建了notFull这个等待队列，当队列为满时put线程会被阻塞在notFull等待队列上。此外，head节点是阻塞队列的头结点，是一个哑元；last节点是阻塞队列的尾节点，指向最后一个节点，所有的元素会被封装为一个Node节点。这里记录元素个数的count声明为AtomicInteger类型，这是因为插入元素和移除元素是并发的，插入元素和移除元素都会改变count的值，因此需要使用CAS操作修改count的值。

## LinkedBlockingQueue(阻塞)插入元素的过程 ##
	public boolean add(E e){
		if(offer(e)){
			return true;
		} else {
			// throw exception
		}
	}
add()方法委托给offer()方法

	public boolean offer(E e){
		if(e == null){
			//不能放入null
			return false;
		}
		final AtomicInteger count = this.count;
		if(count.get() == capacity){
			return false;
		}
		int c = -1;
		Node<E> node = new Node<>(e);
		final ReentrantLock lock = this.putLock;
		lock.lock();
		try{
			if(count.get() < capacity){
				enqueue(node);
				c = count.getAndIncreame();
				//还有剩余容量，唤醒notFull等待队列上的put线程
				if(c+1 < capacity){
					notFull.signal();
				}
			}
		} finally{
			lock.unlock();
		}
		//插入元素之前队列为空，需要唤醒notEmpty等待队列上的take线程
		if(c == 0){
			signalNotEmpty();
		}
		return c >= 0;
	}
	//元素插入到队尾
	private void enqueue(Node<E> e){
		last.next = node;
		last = node;
	}
	//获取takeLock，唤醒notEmpty上的take线程
	private void signalNotEmpty(){
		final ReentrantLock lock = this.takeLock;
		lock.lock();
		try{
			notEmpty.signal();
		} finally{
			lock.unlock();
		}
	}
首先LinkedBlockingQueue阻塞队列中还是不能放入null， 如果阻塞队列满了，offer()方法立即返回false，如果没有满，会先获取putLock，将元素封装成Node节点插入到队列末尾，并将count的值自增，如果自增后的count值仍然小于队列容量，那么唤醒其它的put线程，即唤醒notFull等待队列上的线程执行插入操作；如果count自增前的值为0，说明之前可能有take线程阻塞在notEmpty等待队列上，因此需要唤醒notEmpty等待队列上的take线程，在执行signal方法前应该先获取takeLock锁，只有获得了锁才能执行signal方法。

如果添加了元素之后仍然有剩余空间，LinkedBlockingQueue会唤醒其它的put线程，而在ArrayBlockingQueue中不会唤醒put线程，而是仅仅唤醒take线程，这是因为LinkedBlockingQueue中put线程和take线程是锁分离的，而ArrayBlockingQueue中如果唤醒了put线程，那么take()线程由于得不到锁可能一直无法执行。

只有当插入元素前队列为空的(c==0)时候才会唤醒take线程，而不是当(c>0)的时候也去唤醒take线程，因为take线程在删除元素之后也会唤醒take线程，c>0的时候也唤醒take线程意义不大。

    public void put(E e) throws InterruptedException {
        if (e == null) throw new NullPointerException();
        
        int c = -1;
        Node<E> node = new Node(e);
        final ReentrantLock putLock = this.putLock;
        final AtomicInteger count = this.count;
        putLock.lockInterruptibly();
        try {
            /*
             * Note that count is used in wait guard even though it is
             * not protected by lock. This works because count can
             * only decrease at this point (all other puts are shut
             * out by lock), and we (or some other waiting put) are
             * signalled if it ever changes from capacity. Similarly
             * for all other uses of count in other wait guards.
             */
            while (count.get() == capacity) {
                notFull.await();
            }
            enqueue(node);
            c = count.getAndIncrement();
			//还有空余空间，唤醒其它put线程
            if (c + 1 < capacity)
                notFull.signal();
        } finally {
            putLock.unlock();
        }
		//插入元素之前队列为空，才去唤醒take线程
        if (c == 0)
            signalNotEmpty();
    }
put()方法实现了阻塞插入，当队列满的时候会一直堵塞。

## LinkedBlockingQueue（阻塞）删除元素的过程 ##
	public boolean remove(Object o) {
	   if (o == null) return false;
	     fullyLock();//同时对putLock和takeLock加锁
	     try {
	         //循环查找要删除的元素
	         for (Node<E> trail = head, p = trail.next;
	              p != null;
	              trail = p, p = p.next) {
	             if (o.equals(p.item)) {//找到要删除的节点
	                 unlink(p, trail);//直接删除
	                 return true;
	             }
	         }
	         return false;
	     } finally {
	         fullyUnlock();//解锁
	     }
	    }
	
	//两个同时加锁
	void fullyLock() {
	       putLock.lock();
	       takeLock.lock();
	   }
	
	void fullyUnlock() {
	      takeLock.unlock();
	      putLock.unlock();
	  }

    void unlink(Node<E> p, Node<E> trail) {
        // assert isFullyLocked();
        // p.next is not changed, to allow iterators that are
        // traversing p to maintain their weak-consistency guarantee.
        p.item = null;
        trail.next = p.next;
        if (last == p)
            last = trail;
        if (count.getAndDecrement() == capacity)
            notFull.signal();
    }
remove方法删除的是给定的元素，由于不知道给定元素在什么位置，需要同时获取putLock和takeLock，然后开始从head往后找，一直找到last节点，删除这个节点并修改相关引用之后，需要将count-1，此时，如果count-1之前的值等于capacity，说明删除元素前队列是满的，因此需要唤醒put线程，即notFull等待队列上的线程。

	public E poll() {
	         //获取当前队列的大小
	        final AtomicInteger count = this.count;
	        if (count.get() == 0)//如果没有元素直接返回null
	            return null;
	        E x = null;
	        int c = -1;
	        final ReentrantLock takeLock = this.takeLock;
	        takeLock.lock();
	        try {
	            //判断队列是否有数据
	            if (count.get() > 0) {
	                //如果有，直接删除并获取该元素值
	                x = dequeue();
	                //当前队列大小减一
	                c = count.getAndDecrement();
	                //如果队列未空，继续唤醒等待在条件对象notEmpty上的消费线程
	                if (c > 1)
	                    notEmpty.signal();
	            }
	        } finally {
	            takeLock.unlock();
	        }
	        //判断c是否等于capacity，这是因为如果满说明NotFull条件对象上
	        //可能存在等待的添加线程
	        if (c == capacity)
	            signalNotFull();
	        return x;
	    }
	
	  private E dequeue() {
	        Node<E> h = head;//获取头结点
	        Node<E> first = h.next; 获取头结的下一个节点（要删除的节点）
	        h.next = h; // help GC//自己next指向自己，即被删除
	        head = first;//更新头结点
	        E x = first.item;//获取删除节点的值
	        first.item = null;//清空数据，因为first变成头结点是不能带数据的，这样也就删除队列的带数据的第一个节点
	        return x;
	    }
poll()方法移除队首元素，如果队列为空，那么直接返回null，队列不为空，则需要获取takeLock，修改相应引用删除队首元素，并将count值减1，如果count值减1之后大于0，说明阻塞队列上还有元素，则唤醒阻塞在notEmpty上的take线程。如果count的值减1之前等于capacity，说明删除队首元素之前队列满的，那么会唤醒阻塞在notFull等待队列上的put线程。

	public E take() throws InterruptedException {
	        E x;
	        int c = -1;
	        //获取当前队列大小
	        final AtomicInteger count = this.count;
	        final ReentrantLock takeLock = this.takeLock;
	        takeLock.lockInterruptibly();//可中断
	        try {
	            //如果队列没有数据，挂机当前线程到条件对象的等待队列中
	            while (count.get() == 0) {
	                notEmpty.await();
	            }
	            //如果存在数据直接删除并返回该数据
	            x = dequeue();
	            c = count.getAndDecrement();//队列大小减1
	            if (c > 1)
	                notEmpty.signal();//还有数据就唤醒后续的消费线程
	        } finally {
	            takeLock.unlock();
	        }
	        //满足条件，唤醒条件对象上等待队列中的添加线程
	        if (c == capacity)
	            signalNotFull();
	        return x;
	    }
take()方法的逻辑比较简单，和poll()方法的逻辑几乎一致，除了队列为空的时候take()方法会一直阻塞。

## LinkedBlockingQueue阻塞指定时间 ##
	//在指定时间内阻塞添加的方法，超时就结束
	 public boolean offer(E e, long timeout, TimeUnit unit)
	        throws InterruptedException {
	
	        if (e == null) throw new NullPointerException();
	        //将时间转换成纳秒
	        long nanos = unit.toNanos(timeout);
	        int c = -1;
	        //获取锁
	        final ReentrantLock putLock = this.putLock;
	        //获取当前队列大小
	        final AtomicInteger count = this.count;
	        //锁中断(如果需要)
	        putLock.lockInterruptibly();
	        try {
	            //判断队列是否满
	            while (count.get() == capacity) {
	                if (nanos <= 0)
	                    return false;
	                //如果队列满根据阻塞的等待
	                nanos = notFull.awaitNanos(nanos);
	            }
	            //队列没满直接入队
	            enqueue(new Node<E>(e));
	            c = count.getAndIncrement();
	            //唤醒条件对象上等待的线程
	            if (c + 1 < capacity)
	                notFull.signal();
	        } finally { 
	            putLock.unlock();
	        }
	        //唤醒消费线程
	        if (c == 0)
	            signalNotEmpty();
	        return true;
	    }
阻塞指定的时间，需要借助Condition对象的awaitNanos()方法，达到等待时间之后，offer()方法会返回false。
