# Java NIO #
Java NIO属于非阻塞IO，这是与传统IO最本质的区别。传统IO包括socket和文件IO都是阻塞式的，即一个动作的执行，必须等待先前的动作完成；非阻塞的IO在线程执行一个动作，不用等待动作执行完，可以去做别的事情，这是因为NIO是基于Channel的，而不是基于流的。每个线程可以同时监听多个注册到Selector上的Channel，   NIO 是一种同步非阻塞的 IO 模型。同步是指线程不断轮询 IO 事件是否就绪，非阻塞是指线程在等待 IO 的时候，可以同时做其他任务。同步的核心就是 Selector，Selector 代替了线程本身轮询 IO 事件，避免了阻塞同时减少了不必要的线程消耗；非阻塞的核心就是通道和缓冲区，当 IO 事件就绪时，可以通过写到缓冲区，保证 IO 的成功，而无需线程阻塞式地等待。

## Buffer 缓冲区 ##
Java NIO中所有的缓冲区都继承于 Buffer这个抽象类。Buffer类似于一块可以被读写的区域，因此有读模式和写模式两种模式，在Buffer类中通过三个变量控制缓冲区的读和写position（下一个读或写的数组下标），limit（在读模式和写模式下有不同的含义），capacity（数组的容量）：

- 写模式：往数组中写入数据时候，将limit设置为capacity，表示最多可以写入capacity个元素，position就表示下一个写入的下标。通过Buffer的flip()方法可以从写模式切换到读模式。

	    public final Buffer flip() {
	        limit = position;
	        position = 0;
	        mark = -1;
	        return this;
	    }
- 读模式：从数组中读取数据的时候，将limit设置为position，表示最多只有这么多个元素可以读，position设置为0，表示从头开始读数据。通过调用Buffer的clear()方法或者compact()方法可以从读模式切换到写模式。

	    public final Buffer clear() {
	        position = 0;
	        limit = capacity;
	        mark = -1;
	        return this;
	    }
clear()方法是不保留数据的（即使有些数据还没有读完），因为直接将position设置为0，表示从头开始写入数据，limit重新设置capacity。

	    public ByteBuffer compact() {
	
	        System.arraycopy(hb, ix(position()), hb, ix(0), remaining());
	        position(remaining());
	        limit(capacity());
	        discardMark();
	        return this;
	    }
compact()方法不是Buffer类中的方式，是在Buffer的各个子类（ByteBuffer、CharBuffer等）中定义的抽象方法，并且在（ByteBuffer、CharBuffer等）的子类中提供了实现，compact()方法会保留还没读的数据，先将没读的数据拷贝到数组的最前面，然后设置position为下一个写入的下标，limit重写设置为capacity。


## ByteBuffer 字节缓冲区 ##
ByteBuffer继承于Buffer，ByteBuffer仍然是个抽象类，我们只能通过它的allocate(int capacity)方法获取一个非直接字节缓冲区（缓冲区不是直接在内存中的），也可以通过warp(byte[] b)方法创建一个非直接字节缓冲区。如果想要创建一个直接字节缓冲区，可以使用allocateDirect(int capacity)方法，直接缓冲区不是在堆上分配的，因此不受GC的管理，其创建和释放过程比较耗时，但是直接缓冲区上数据的读写比较快。

	//在堆上创建一个字节缓冲区
    public static ByteBuffer allocate(int capacity) {
        if (capacity < 0)
            throw new IllegalArgumentException();
        return new HeapByteBuffer(capacity, capacity);
    }
	//创建一个直接字节缓冲区
    public static ByteBuffer allocateDirect(int capacity) {
        return new DirectByteBuffer(capacity);
    }

**HeapByteBuffer：**HeapByteBuffer是 ByteBuffer的默认实现类，在堆上创建一个字节缓冲区.HeapByteBuffer主要有以下几个方面的功能：

- 读数据:

		//获取position处的元素，并将position++
	    public byte get() {
	        return hb[ix(nextGetIndex())];
	    }
		//获取指定下标处的元素，此时position是没有变的。
	    public byte get(int i) {
	        return hb[ix(checkIndex(i))];
	    }
		//获取从position开始的length个元素，并拷贝到指定数组中，position会更新为position + length
		public ByteBuffer get(byte[] dst, int offset, int length) {
	        checkBounds(offset, length, dst.length);
	        if (length > remaining())
	            throw new BufferUnderflowException();
	        System.arraycopy(hb, ix(position()), dst, offset, length);
	        position(position() + length);
        return this;
    }

- 写数据：

		//将指定元素插入到position处，并将position++（如果满了将会抛出异常）
		public ByteBuffer put(byte x) {
		
		        hb[ix(nextPutIndex())] = x;
		        return this;
		}

		//将指定元素插入到指定位置，position位置不会改变
	 	public ByteBuffer put(int i, byte x) {
	
	        hb[ix(checkIndex(i))] = x;
	        return this;
	    }

		//从position处开始写入指定数组中的元素，position会被更新为（position + length）
	    public ByteBuffer put(byte[] src, int offset, int length) {
	
	        checkBounds(offset, length, src.length);
	        if (length > remaining())
	            throw new BufferOverflowException();
	        System.arraycopy(src, offset, hb, ix(position()), length);
	        position(position() + length);
	        return this;
		}

- 读写char, int ,short, double等其他基本数据类型

		//获取一个字符，Bits类根据大端还是小端用不同的方式组合两个字节
		public char getChar() {
	        return Bits.getChar(this, ix(nextGetIndex(2)), bigEndian);
	    }
		//大端存储模式，高位字节保存在低字节部分，因此组合的时候低字节在前面，高字节在后面
		static char getCharB(ByteBuffer bb, int bi) {
        	return makeChar(bb._get(bi    ),
                        bb._get(bi + 1));
    	}
		//小端存储模式，高位字节保存在高字节部分，因此组合的时候高字节在前面，低字节在后面
		static char getCharL(ByteBuffer bb, int bi) {
        	return makeChar(bb._get(bi + 1),
                        bb._get(bi    ));
    	}
	
		//写入一个char字符的时候也是一样
	    public ByteBuffer putChar(char x) {
	
	        Bits.putChar(this, ix(nextPutIndex(2)), x, bigEndian);
	        return this;
		}

- 用ByteBuffer包装成其他基本数据类型的缓冲区（CharBuffer、IntBuffer等）

		//ByteBuffer的asCharBuffer()方法
	    public CharBuffer asCharBuffer() {
	        int size = this.remaining() >> 1;
	        int off = offset + position();
	        return (bigEndian
	                ? (CharBuffer)(new ByteBufferAsCharBufferB(this,
	                                                               -1,
	                                                               0,
	                                                               size,
	                                                               size,
	                                                               off))
	                : (CharBuffer)(new ByteBufferAsCharBufferL(this,
	                                                               -1,
	                                                               0,
	                                                               size,
	                                                               size,
	                                                               off)));
	    }
asCharBuffer()方法也会根据机器是大端模式还是小端模式，创建不同的对象。如果是大端模式，将会创建ByteBufferAsCharBufferB类对象，由于ByteBufferAsCharBufferB对象是由ByteBuffer对象包装而来的，虽然ByteBufferAsCharBufferB的父类是CharBuffer，但是ByteBufferAsCharBufferB类中操作的并不是字符数组而是字节数组，所以ByteBufferAsCharBufferB类对象在读写的时候仍然借助Bits类完成，即先通过字节组成字符再读写。根据ByteBuffer得到其他类型的缓冲区也是一样的实现原理。

- 创建只读型缓冲区

	    public ByteBuffer asReadOnlyBuffer() {
	
	        return new HeapByteBufferR(hb,
	                                     this.markValue(),
	                                     this.position(),
	                                     this.limit(),
	                                     this.capacity(),
	                                     offset);
	    }
asReadOnlyBuffer()会创建HeapByteBufferR类对象，每种缓冲区都可以创建对应的只读型缓冲区，只读型缓冲区就直接继承创建它的这个类，如HeapByteBufferR继承于HeapByteBuffer，在HeapByteBufferR类中重写所有的put方法，所有的put方法都直接抛出异常，而get方法仍然使用父类的get方法。

**MappedByteBuffer**继承于ByteBuffer，是直接字节数组的抽象父类，在MappedByteBuffer类中只定义了三个和物理磁盘相关的方法：

- isLoaded()：用于判断这个字节数组是否存在物理磁盘上
- load()：将直接字节数组中的数据写到物理磁盘上
- force()：强制将直接字节数组中的数据写到物理磁盘上

**DirectByteBuffer**继承于MappedByteBuffer，用于创建直接字节数组的类。由于直接字节数组不是在堆上分配内存，因此不受GC控制，创建和释放过程比较繁琐，通过通过Unsafe类的allocateMemory()方法分配内存空间，而且DirectByteBuffer的所有get()和put()方法都是通过Unsafe类完成

	//读取position处的元素
    public byte get() {
        return ((unsafe.getByte(ix(nextGetIndex()))));
    }

	//在position处写入元素
    public ByteBuffer put(byte x) {

        unsafe.putByte(ix(nextPutIndex()), ((x)));
        return this;
    }

## Channel ##
关于同步、异步、阻塞和非阻塞的区别：同步和异步说的是消息的通知机制，阻塞非阻塞说的是线程的状态 。

- 同步阻塞IO：client在调用read()方法时，stream里没有数据可读，线程停止向下执行，直至stream有数据。
- 同步非阻塞IO：client在调用read()方法时，stream里没有数据可读，read()方法就返回了，线程可以去干别的事，但是需要有一个线程监听这stream中是否有数据准备好。
- 异步非阻塞IO：服务端调用read()方法，若stream中无数据则返回，程序继续向下执行。当stream中有数据时，操作系统会负责把数据拷贝到用户空间，然后通知这个线程，这里的消息通知机制就是异步！

NIO 是一种同步非阻塞的 IO 模型。同步是指线程不断轮询 IO 事件是否就绪，非阻塞是指线程在等待 IO 的时候，可以同时做其他任务。同步的核心就是 Selector，Selector 代替了线程本身轮询 IO 事件，避免了阻塞同时减少了不必要的线程消耗；非阻塞的核心就是通道和缓冲区，当 IO 事件就绪时，可以通过写道缓冲区，保证 IO 的成功，而无需线程阻塞式地等待。

**Channel**：是NIO中通道的接口类，只提供了两个抽象方法，isOpen()判断通道是否打开，close()关闭一个通道。

**AbstractInterruptibleChannel**：是一个抽象类，实现了Channel接口。任何一个通道如果想要实现在中断时实现异步关闭通道，那么必须继承这个类，这主要体现在两个方面：

- 当某个线程阻塞在channel上，而另一个线程调用了channel的close()方法，那么阻塞的线程会收到AsynchronousCloseException
- 如果某个线程阻塞在channel上，另一个线程调用了阻塞线程的interrupt()方法，那么阻塞线程会收到ClosedByInterruptException，并且通道会被关闭。

AbstractInterruptibleChannel抽象类中定义了一组协同方法begin()和end()方法来完成这两个功能，因此当线程执行一个可能阻塞的IO操作时，必须把这个IO操作放在begin()方法和end()方法之间，才能实现channel的异步关闭。

    protected final void begin() {
        if (interruptor == null) {
            interruptor = new Interruptible() {
                    public void interrupt(Thread target) {
                        synchronized (closeLock) {
                            if (!open)
                                return;
                            open = false;
                            interrupted = target;
                            try {
								//收到中断请求后会回调AbstractInterruptibleChannel类的close()方法关闭通道
                                AbstractInterruptibleChannel.this.implCloseChannel();
                            } catch (IOException x) { }
                        }
                    }};
        }
        blockedOn(interruptor);
        Thread me = Thread.currentThread();
        if (me.isInterrupted())
            interruptor.interrupt(me);
    }

    protected final void end(boolean completed)
        throws AsynchronousCloseException
    {
        blockedOn(null);
        Thread interrupted = this.interrupted;
		//中断触发器不为空，会抛出ClosedByInterruptException
        if (interrupted != null && interrupted == Thread.currentThread()) {
            interrupted = null;
            throw new ClosedByInterruptException();
        }
		//触发器为空，没有中断，但是在阻塞的过程中channel被关闭了，抛出AsynchronousCloseException
        if (!completed && !open)
            throw new AsynchronousCloseException();
    }

**SelectableChannel**：是一个抽象类，继承于AbstractInterruptibleChannel。需要注册到Selector上的通道必须继承这个类

- public abstract int validOps();获取这个通道支持的事件
- public abstract boolean isRegistered();通道是否注册到了某个Selector上
- public abstract SelectionKey keyFor(Selector sel);这个通道在指定Selector上注册的事件
- public abstract SelectionKey register(Selector sel, int ops, Object att) throws ClosedChannelException;注册这个通道到指定的Selector上
- public abstract SelectableChannel configureBlocking(boolean block) throws IOException;修改这个通道为非阻塞或阻塞
- public abstract boolean isBlocking();如果这个通道是阻塞模式，返回true

**AbstractSelectableChannel**：是一个抽象类，是SelectableChannel的基础实现类。在AbstractSelectableChannel类中定义了一个SelectionKey数组，记录这个channel注册到了哪些Selector上，定义了一个keyCount记录这个channel注册的次数。并且channel的最初模式设置为阻塞模式。

- 判断这个channel是否注册了：keyCount不为0就表示注册了

	    public final boolean isRegistered() {
	        synchronized (keyLock) {
	            return keyCount != 0;
	        }
	    }

- 获取这个通道在指定Selector上的注册事件，给定Selector，在SelectionKey[]数组中查找Selector与指定Selector相等的SelectionKey
	
	    public final SelectionKey keyFor(Selector sel) {
	        return findKey(sel);
	    }

- 将这个通道注册到指定Selector上，如果给定的事件ops不是这个通道支持的事件validOps()将会抛出异常，而且阻塞的channel无法注册到Selector上，具体在注册的时候如果channel已经注册到了这个Selector上，那么更新ops和附加信息，如果这个channel还没有注册到这个Selector上，将会调用Selector 类的register(SelectableChannel ch, int ops, Object o)方法完成注册

		public final SelectionKey register(Selector sel, int ops, Object att) throws ClosedChannelException{
	        synchronized (regLock) {
	            if (!isOpen())
	                throw new ClosedChannelException();
	            if ((ops & ~validOps()) != 0)
	                throw new IllegalArgumentException();
	            if (blocking)
	                throw new IllegalBlockingModeException();
	            SelectionKey k = findKey(sel);
	            if (k != null) {
	                k.interestOps(ops);
	                k.attach(att);
	            }
	            if (k == null) {
	                // New registration
	                synchronized (keyLock) {
	                    if (!isOpen())
	                        throw new ClosedChannelException();
	                    k = ((AbstractSelector)sel).register(this, ops, att);
	                    addKey(k);
	                }
	            }
	            return k;
	        }
	    }
- 关闭通道除了需要关闭通道之外，还需要把SelectionKey[]数组上的SelectionKey取消，而SelectionKey类中的cancel()方法又会调用AbstractSelector中的cancel()方法，AbstractSelector类中的cancel()方法将这个SelectionKey加入到cancelledKeys集合中。

	    protected final void implCloseChannel() throws IOException {
	        implCloseSelectableChannel();
	        synchronized (keyLock) {
	            int count = (keys == null) ? 0 : keys.length;
	            for (int i = 0; i < count; i++) {
	                SelectionKey k = keys[i];
	                if (k != null)
	                    k.cancel();
	            }
	        }
	    }

- 判断channel是否是阻塞模式，只有非阻塞channel才能注册到Selector上
	
	    public final boolean isBlocking() {
	        synchronized (regLock) {
	            return blocking;
	        }
	    }

- 修改channel的阻塞模式

	    public final SelectableChannel configureBlocking(boolean block) throws IOException {
	        synchronized (regLock) {
	            if (!isOpen())
	                throw new ClosedChannelException();
	            if (blocking == block)
	                return this;
	            if (block && haveValidKeys())
	                throw new IllegalBlockingModeException();
	            implConfigureBlocking(block);
	            blocking = block;
	        }
	        return this;
	    }

**SelectionKey**：每次channel注册到一个Selector都会返回一个SelectionKey对象，因此SelectionKey描述的是一次注册事件中channel和Selector之间的映射关系。

- public abstract SelectableChannel channel();获取SelectionKey对应的channel
- public abstract Selector selector();获取对应SelectionKey对应的channel
- public abstract boolean isValid();这个SelectionKey是否有效
- public abstract void cancel();把这个SelectionKey置为无效
- public abstract int interestOps();获取这个SelectionKey关注的事件
- public abstract SelectionKey interestOps(int ops);设置这个SelectionKey关注的事件
- public abstract int readyOps();这个SelectionKey关注的事件中就绪了的事件
- public static final int OP_READ = 1 << 0;读事件
- public static final int OP_WRITE = 1 << 2;写事件
- public static final int OP_CONNECT = 1 << 3;channel连接到服务器的连接事件
- public static final int OP_ACCEPT = 1 << 4;服务器准备好了接受连接事件
- channel上有读事件就绪、写事件就绪（isWritable）、连接事件就绪(isConnectable)、接受连接事件就绪(isAcceptable)

	public final boolean isReadable() {
	        return (readyOps() & OP_READ) != 0;
	    }

**AbstractSelectionKey**：是一个抽象类，是SelectionKey的基础实现类。一个channel注册到一个Selector上返回一个SelectionKey，这个SelectionKey初始就是有效的。

- 取消一个SelectionKey，先将SelectionKey置为无效，然后调用Selector的cancel(SelectionKey key)方法完成具体的取消操作

		public final void cancel() {
	        synchronized (this) {
	            if (valid) {
	                valid = false;
	                ((AbstractSelector)selector()).cancel(this);
	            }
	        }
	    }

**SelectionKeyImpl**：SelectionKey的最终实现类，继承于AbstractSelectionKey。在SelectionKeyImpl类中定义了int类型interestOps、int类型的readyOps，实现了SelectionKey抽象类中的interestOps()、readyOps()、interestOps(int ops)这三个方法。

## Selector ##
Selector是NIO得以实现的核心模块之一，NIO属于同步非阻塞IO，同步的核心就是 Selector，Selector 代替了线程本身轮询 IO 事件，避免了阻塞同时减少了不必要的线程消耗；

**Selector**：是一个抽象类。提供了静态方法open()创建一个Selector对象，open()方法是使用SelectorProvider类完成的。

- public abstract boolean isOpen();这个Selector是否打开了
- public abstract SelectorProvider provider();获取创建这个Selector的SelectorProvider
- public abstract Set<SelectionKey> keys();获取这个Selector上所有的注册的channel对应的SelectionKey
- public abstract Set<SelectionKey> selectedKeys();获取这个Selector上所有就绪了的channel对应的SelectionKey
- public abstract int selectNow() throws IOException;非阻塞的执行一次选择操作，没有就绪的channel就立即返回0，否则返回有多少个channel就绪了。
- public abstract int select(long timeout) throws IOException;执行一次选择操作，并且阻塞一段时间，在这段时间里，如果有channel就绪了，将会返回有多少个channel就绪了，如果到达了指定时间还没有channel就绪，就返回0.
- public abstract int select() throws IOException;阻塞执行选择操作，一直阻塞到有channel就绪，然后返回有多少个channel就绪了
- public abstract Selector wakeup();使阻塞在select()方法上的线程立即返回。

		Selector selector = Selector.open();
		channel.configureBlocking(false);
		SelectionKey key = channel.register(selector, SelectionKey.OP_READ);
		while(true) {
		  int readyChannels = selector.select();
		  if(readyChannels == 0) continue;
		 
		  Set selectedKeys = selector.selectedKeys();
		  Iterator keyIterator = selectedKeys.iterator();
		 
		  while(keyIterator.hasNext()) {
		    SelectionKey key = keyIterator.next();
		 
		    if(key.isAcceptable()) {
		        // a connection was accepted by a ServerSocketChannel.
		    } else if (key.isConnectable()) {
		        // a connection was established with a remote server.
		    } else if (key.isReadable()) {
		        // a channel is ready for reading
		    } else if (key.isWritable()) {
		        // a channel is ready for writing
		    }
		 
		    keyIterator.remove();
		  }
		}

**AbstractSelector**：是一个抽象类，是Selector的基础实现类.

- 定义了一个boolean类型的open变量，初始化为true，表示一个Selector创建时就为打开状态。
- 定义了一个cancelledKeys集合，表示已经取消的SelectionKey的集合。SelectionKey中的cancel()方法会调用AbstractSelector类的cancel()方法，cancel()方法将SelectionKey加入到cancelledKeys集合中。

	    void cancel(SelectionKey k) {                       // package-private
	        synchronized (cancelledKeys) {
	            cancelledKeys.add(k);
	        }
	    }

- 定义了一个SelectorProvider对象，用于实现Selector类中的provider()方法。
- 定义了一个反注册方法deregister(SelectionKey key)，调用的是channel的方法，将key从SelectionKey[] keys数组中移除

	    protected final void deregister(AbstractSelectionKey key) {
	        ((AbstractSelectableChannel)key.channel()).removeKey(key);
	    }

- 定义了一组协同方法begin()和end()，与AbstractInterruptibleChannel类中协同方法类似，在一个可能阻塞的IO操作前使用begin()方法，在IO操作之后使用end()方法，但是这里的协同方法是为了在产生中断之后使select()方法立即返回。

	    protected final void begin() {
	        if (interruptor == null) {
	            interruptor = new Interruptible() {
	                    public void interrupt(Thread ignore) {
							//产生中断之后，调用wakeup()方法唤醒select()方法
	                        AbstractSelector.this.wakeup();
	                    }};
	        }
	        AbstractInterruptibleChannel.blockedOn(interruptor);
	        Thread me = Thread.currentThread();
	        if (me.isInterrupted())
	            interruptor.interrupt(me);
	    }

**SelectorImpl**：仍然是一个抽象类，继承于AbstractSelector类，进一步实现了Selector类。

- 定义了一个keys集合，用于实现Selector类的keys()方法，直接返回这个集合。
- 定义了一个selectedKeys集合，用于实现Selector类的selectedKeys()方法，直接返回这个集合。
- 将三个select()方法均委托给一个抽象方法，待子类进一步实现。

		//委托给lockAndDoSelect()方法
		public int select(long timeout) throws IOException{
	              if (timeout < 0)
	                  throw new IllegalArgumentException("Negative timeout");
	              return lockAndDoSelect((timeout == 0) ? -1 : timeout);
	    }
		//调用上一个方法
		public int select() throws IOException {
             return select(0);
        }
		//委托给lockAndDoSelect()方法
		public int selectNow() throws IOException {
             return lockAndDoSelect(0);
        }

		//同步锁，进一步委托给doSelect(long time)这个抽象方法。
		private int lockAndDoSelect(long timeout) throws IOException {
              synchronized (this) {
                  if (!isOpen())
                      throw new ClosedSelectorException();
                  synchronized (publicKeys) {
                      synchronized (publicSelectedKeys) {
                          return doSelect(timeout);
                      }
                  }
              }
          }

- 进一步实现了close()方法，但是没有完全实现，还是委托给一个抽象方法

 		//先调用wakeup()方法使select()方法立即返回，然后同步锁，调用抽象方法implClose()
		public void implCloseSelector() throws IOException {
             wakeup();
             synchronized (this) {
                 synchronized (publicKeys) {
                     synchronized (publicSelectedKeys) {
                         implClose();
                     }
                 }
             }
         }

- 初步实现了register(channel, int ops, Object att)方法，委托给抽象方法implRegister(SelectionKey k)方法

		protected final SelectionKey register(AbstractSelectableChannel ch, int ops, Object attachment){
             if (!(ch instanceof SelChImpl))
                 throw new IllegalSelectorException();
			//新建一个SelectionKey对象
             SelectionKeyImpl k = new SelectionKeyImpl((SelChImpl)ch, this);
			//设置附加信息
             k.attach(attachment);
             synchronized (publicKeys) {
                 implRegister(k);
             }
			//设置感兴趣事件
             k.interestOps(ops);
             return k;
         }
- 定义了一个方法处理cancelledKeys集合，委托给抽象方法implDereg(SelectionKey k)方法

	       void processDeregisterQueue() throws IOException {
	             // Precondition: Synchronized on this, keys, and selectedKeys
				//获取cancelledKeys集合
	             Set cks = cancelledKeys();
	             synchronized (cks) {
	                 if (!cks.isEmpty()) {
	                     Iterator i = cks.iterator();
	                     while (i.hasNext()) {
	                         SelectionKeyImpl ski = (SelectionKeyImpl)i.next();
	                         try {
	                             implDereg(ski);
	                         } catch (SocketException se) {
	                             IOException ioe = new IOException("Error deregistering key");
	                             ioe.initCause(se);
	                             throw ioe;
	                         } finally {
	                             i.remove();
	                         }
	                     }
	                 }
	             }
	         }

## SelectorProvider ##
SelectorProvider为Selector、DatagramChannel、SocketChannel、ServerSocketChannel、Pipe这些Selector和channel提供打开方法。

**SelectorProvider**：是一个抽象类

-     public abstract DatagramChannel openDatagramChannel() throws IOException;打开UDP通信channel
-     public abstract Pipe openPipe() throws IOException;打开一个管道
-     public abstract AbstractSelector openSelector() throws IOException;打开一个Selector
- public abstract ServerSocketChannel openServerSocketChannel() throws IOException;打开一个服务器socket channel
-     public abstract SocketChannel openSocketChannel() throws IOException;打开一个TCP通信channel

**SelectorProviderImpl**：是一个抽象类，是SelectorProvider的基础实现类。

- 打开UDP通信channel，返回的是UDP channel的实现类对象。

	 	public DatagramChannel openDatagramChannel() throws IOException   {  
	        return new DatagramChannelImpl(this);  
	    }  
- 打开TCP通信channel，返回的是Socket channel的实现类对象。

	    public SocketChannel openSocketChannel() throws IOException  {  
	        return new SocketChannelImpl(this);  
	    }  

- 打开TCP通信服务器端的channel，返回的是ServerSocket channel的实现类对象。

	    public ServerSocketChannel openServerSocketChannel() throws IOException  {  
	        return new ServerSocketChannelImpl(this);  
	    } 

- 打开一个管道，返回的是pipe实现类对象。

		public Pipe openPipe() throws IOException  {  
	        return new PipeImpl(this);  
	    }  
- 打开Selector的方法没有实现，待子类实现。

**WindowsSelectorProvider **：SelectorProvider的最终实现类，继承于SelectorProviderImpl。实现了Selector的打开方法

- 打开Selector。调用的是Selector的最终实现类WindowsSelectorImpl，通过WindowsSelectorImpl的构造函数返回一个Selector

		public AbstractSelector openSelector() throws IOException {
              return new WindowsSelectorImpl(this);
        }


## 再看 channel ##

**FileChannel**：是一个抽象类，继承于AbstractInterruptibleChannel类，没有继承SelectableChannel，即FileChannel无法注册到Selector上。FileChannel也无法以非阻塞模式读写。通过阻塞方式对文件读写。

- 打开一个FileChannel，一般通过传统IO流获取FileChannel。但是FileChannel类中也定义了open()函数打开一个FileChannel，getChannel()方法底层就是调用的FileChannel的open()方法

		FileInputStream fis = new FileInputStream("C:\\mycode\\hello.txt");
		
		FileChannel inChannel = fis.getChannel();

- 从FileChannel读取数据:public abstract int read(ByteBuffer dst) throws IOException;将通道中的数据读出来,并写道指定ByteBuffer中.FileChannel也支持分散写,即写到多个ByteBuffer中.

		public abstract long read(ByteBuffer[] dsts, int offset, int length) throws IOException;

- 向FileChannel写数据: public abstract int write(ByteBuffer src) throws IOException;将ByteBuffer缓冲区中的数据写入到channel中.FileChannel也支持聚集写,即多个ByteBuffer中的数据写到channel中.

		public abstract long write(ByteBuffer[] srcs, int offset, int length) throws IOException;

- 在FileChannel的某个特定位置进行数据的读/写操作,改变文件position的位置.
		
		//获取文件position的位置
		public abstract long position() throws IOException;

		//设置文件position
		public abstract FileChannel position(long newPosition) throws IOException;

- 将channel中的数据写入到另一个channel，或将另一个channel中的数据写入到这个channel中

		//将本channel中的数据写入到另一个“可写入”的channel，从另一个channel的position处开始写入
    	public abstract long transferTo(long position, long count, WritableByteChannel target) throws IOException;

		//将另一个channel中的数据写入到本channel中，从另一个channel的position处开始
		public abstract long transferFrom(ReadableByteChannel src, long position, long count) throws IOException;

- 获取channel关联的文件的大小

		public abstract long size() throws IOException;

- 截取一个指定大小的文件，截断channel关联的文件为size大小，size之后的数据会被丢弃

		public abstract FileChannel truncate(long size) throws IOException;

**DatagramChannel**：是一个抽象类，真正的实现类是其子类DatagramChannelImpl，继承于AbstractSelectableChannel类。因此有阻塞和非阻塞两种模式，在非阻塞模式下可以注册到Selector上。UDP通信的channel。

- 创建一个DatagramChannel。调用DatagramChannel的静态方法open()，通过SelectorProvider去创建DatagramChannelImpl的实例

	    public static DatagramChannel open() throws IOException {
        	return SelectorProvider.provider().openDatagramChannel();
    	}
- DatagramChannel支持的事件：读和写

	    public final int validOps() {
        	return (SelectionKey.OP_READ | SelectionKey.OP_WRITE);
    	}

- 报文流本来是无连接的，在没有连接到一个指定地址时，channel可以同时发送数据报到多个远程地址、也可以同时从多个远程地址接收数据报。通过DatagramChannel的send(ByteBuffer src, SocketAddress add)方法发送数据报到指定远程地址，通过DatagramChannel的receive(ByteBuffer dst)方法从任意远程地址接收数据报，receive()方法会返回一个SocketAddress对象用以标识数据报来自哪个远程地址。在没有建立连接的时候，每一次调用send()方法发送数据报或者调用receive()方法接收数据报时都会接收安全检查。

		//发送数据报到指定远程地址
	    public abstract int send(ByteBuffer src, SocketAddress target) throws IOException;
	
		//从任意远程地址接收数据报，并返回数据来自哪个地址
		public abstract SocketAddress receive(ByteBuffer dst) throws IOException;

- 报文流也可以建立连接。建立连接后，channel将只能从指定的远程地址接收数据报、同时也只能发送数据报到指定的远程地址。由于已经连接到了指定的远程地址，因此在发送或者接收数据报的时候可以调用write()方法已经read()方法。write(ByteBuffer src)方法将数据报发送到指定远程地址、read(ByteBuffer dst)方法从指定远程地址接收数据报。指定连接到指定远程地址的channel才能调用write()方法和read()方法，每次调用write()方法和read()方法时不需要接收安全检查。**将DatagramChannel置于已连接的状态可以使除了它所“连接”到的地址之外的任何其他源地址的数据报被忽略。这是很有帮助的，因为不想要的包都已经被网络层丢弃了，从而避免了使用代码来接收、检查然后丢弃包的麻烦。**
		
		//将channel连接到指定远程地址
    	public abstract DatagramChannel connect(SocketAddress remote) throws IOException;

		//断开channel与远程地址间的连接
		public abstract DatagramChannel disconnect() throws IOException;

		//channel是否连接到了某个远程地址
		public abstract boolean isConnected();

		//发送数据报到指定远程地址，支持聚集写
		public abstract int write(ByteBuffer src) throws IOException;
		public final long write(ByteBuffer[] srcs) throws IOException {
        	return write(srcs, 0, srcs.length);
    	}
		public abstract long write(ByteBuffer[] srcs, int offset, int length) throws IOException;

		//从指定远程地址接收数据报，支持分散读
		public abstract int read(ByteBuffer dst) throws IOException;
		public final long read(ByteBuffer[] dsts) throws IOException {
        	return read(dsts, 0, dsts.length);
    	}
		public abstract long read(ByteBuffer[] dsts, int offset, int length) throws IOException;

- 如果channel处于阻塞模式：调用send()方法或者write()方法，调用线程可能会休眠直到数据报被加入传输队列。如果channel是非阻塞的：send()方法或者write()方法、read()返回值要么是字节缓冲区的字节数，要么是“0”，receive()方法的返回值要么是远程地址对象要么是null。

- 报文流可以绑定也可以不绑定。如果channel负责监听，那么必须绑定到一个指定端口，channel将会一直监听这个端口。当channel没有绑定的时候，仍然能够接收数据报，使用的是动态分配的端口号。已经绑定的channel将从指定端口接收或者发送数据报。

- 报文流是不可靠传输。1）假如receive()方法提供的ByteBuffer没有足够的剩余空间来存放您正在接收的数据包，没有被填充的字节都会被悄悄地丢弃。2）如果send()方法给定的ByteBuffer传输队列没有足够空间来承载整个数据报，那么什么内容都不会被发送。3）传输过程中的协议可能将数据报分解成碎片。例如，以太网不能传输超过1,500个字节左右的包。如果您的数据报比较大，那么就会存在被分解成碎片的风险，成倍地增加了传输过程中包丢失的几率。被分解的数据报在目的地会被重新组合起来，接收者将看不到碎片。但是，如果有一个碎片不能按时到达，那么整个数据报将被丢弃。

**SocketChannel**：是一个抽象类，真正的实现类是其子类SocketChannelImpl，继承于AbstractSelectableChannel类。因此有阻塞和非阻塞两种模式，在非阻塞模式下可以注册到Selector上。客户端的TCP通信的channel。

- 创建一个SocketChannel对象，通过SocketChannel的静态方法open委托给SelectorProvider类创建SocketChannelImpl类对象。

	    public static SocketChannel open() throws IOException {
	        return SelectorProvider.provider().openSocketChannel();
	    }

- SocketChannel支持的事件：读、写、发起连接

    public final int validOps() {
        return (SelectionKey.OP_READ | SelectionKey.OP_WRITE | SelectionKey.OP_CONNECT);
    }

- channel必须在使用之前连接到远程地址，在阻塞模式下，连接操作会一直阻塞直到连接成功或者失败；在非阻塞模式下，连接操作即使没有连接成功也会立刻返回，因此需要通过finishConnect()方法判断是否连接成功并一直尝试连接。

		//channel是否成连接到一个远程地址
		public abstract boolean isConnected();

		//channel是否正处于连接中
		public abstract boolean isConnectionPending();

		//channel连接一个远程地址
		public abstract boolean connect(SocketAddress remote) throws IOException;

		//channel是否完成了连接
		public abstract boolean finishConnect() throws IOException;

		while(!channel.finishConnect()){
			//由于必须在连接成功之后才能进行IO操作，必须等待连接成功
			doSomethingElse();
		}


- 往channel中写数据，支持聚集写。写完之后，Selector的select()方法会检测到这个channel的WRITE事件就绪了
		
		public abstract int write(ByteBuffer src) throws IOException;
	
		public final long write(ByteBuffer[] srcs) throws IOException {
	        return write(srcs, 0, srcs.length);
	    }
	
	    public abstract long write(ByteBuffer[] srcs, int offset, int length) throws IOException;
	
- 从channel中读取数据，支持分散读。Selector的select()方法会检测到这个channel的READ事件就绪了

	    public abstract int read(ByteBuffer dst) throws IOException;
	
	    public final long read(ByteBuffer[] dsts) throws IOException {
	        return read(dsts, 0, dsts.length);
	    }    
		public abstract long read(ByteBuffer[] dsts, int offset, int length) throws IOException;

SocketChannel的一个实例：

		public class MyClient {
			private static Selector selector = null;
		    private volatile static boolean stop = false;
		    private static SocketChannel channel = null;
			
			public static void main(String[] args) {
		        selector = Selector.open();
				try {
		            channel = SocketChannel.open();
		            channel.configureBlocking(false);
		            channel.connect(new InetSocketAddress("127.0.0.1", 7777));
					//注册一个连接请求事件
		            channel.register(selector, SelectionKey.OP_CONNECT);
		
					try {
			            while (!stop) {
			                selector.select();
			                Set<SelectionKey> selectedKeys = selector.selectedKeys();
			                Iterator<SelectionKey> iterator = selectedKeys.iterator();
			                while (iterator.hasNext()) {
			                    SelectionKey key = iterator.next();
								//连接就绪
			                    if (key.isConnectable()) {
									//服务器接收了连接请求
		                			SocketChannel sc = (SocketChannel) key.channel();
							        if (sc.finishConnect()) {
							            // 将关注的事件变成read
							            sc.register(selector, SelectionKey.OP_READ);
							            doWrite(sc, "dddddd");
							        }
					            }
					            // 读就绪
					            if (key.isReadable()) {
					                //服务器有数据过来了，处理数据，再发送数据
					            }
			                    iterator.remove();
			                }
			
			            }
			        } catch (IOException e) {
			            e.printStackTrace();
			        }
		        } catch (ClosedChannelException e) {
		            System.out.println("client: 失去主机连接");
		            e.printStackTrace();
		        } catch (IOException e) {
		            e.printStackTrace();
		        }
		    }
		}

**ServerSocketChannel**：是一个抽象类，真正的实现类是其子类ServerSocketChannelImpl，继承于AbstractSelectableChannel类。因此有阻塞和非阻塞两种模式，在非阻塞模式下可以注册到Selector上。服务器端的TCP通信的channel。

- 创建一个ServerSocketChannel，通过ServerSocketChannel的静态方法open委托给SelectorProvider类创建ServerSocketChannelImpl类对象。

	    public static ServerSocketChannel open() throws IOException {
	        return SelectorProvider.provider().openServerSocketChannel();
	    }

- ServerSocketChannel支持的事件：接收连接

	    public final int validOps() {
	        return SelectionKey.OP_ACCEPT;
	    }

- ServerSocketChannel必须先绑定到一个端口上，一直监听这个端口。

    	public abstract ServerSocketChannel bind(SocketAddress local, int backlog) throws IOException;

- 获取与这个ServerSocketChannel关联的SocketChannel，即发起连接请求的SocketChannel

		public abstract SocketChannel accept() throws IOException;

ServerSocketChannel的一个实例：

	public class MyService {
	    public static Selector selector = null;
	
	    public static void main(String[] args) {
	        selector = Selector.open();// 打开selector
			ServerSocketChannel server = ServerSocketChannel.open();
	        server.socket().bind(new InetSocketAddress(7777), 1024);
	        server.configureBlocking(false);
			//服务器开始监听等待连接，注册ACCEPT事件
	        server.register(selector, SelectionKey.OP_ACCEPT);
	
			while (true) {
	            try {
	                selector.select(1000); // 阻塞selector
	                // ================如果有新连接
	                Set<SelectionKey> selectedKeys = selector.selectedKeys();// 获得事件集合;
	                // ================遍历selectedKeys
	                Iterator<SelectionKey> iterator = selectedKeys.iterator();
	                SelectionKey key = null;
	                while (iterator.hasNext()) {
	                    key = iterator.next();// 获得到当前的事件
	                    // ===============处理事件
	                     // 连接就绪，有客户端请求连接，注册的事件发生
			            if (key.isAcceptable()) {
			                // 获得对应的ServerSocketChannel
					        ServerSocketChannel ssc = (ServerSocketChannel) key.channel();
					        // 得到对应的SocketChannel 
					        SocketChannel channel = ssc.accept();
					        // 处理socketChannel
					        channel.configureBlocking(false); 
					        channel.register(selector, SelectionKey.OP_READ); 
			            }
			            // 读就绪
			            if (key.isReadable()) {
			                //客户端有数据过来了，之前注册的READ事件来了
							
			            }
	                    // ===============
	                    iterator.remove(); // 移除事件
	                }
	            } catch (IOException e) {
	                e.printStackTrace();
	            }
	        }
	    }
	｝