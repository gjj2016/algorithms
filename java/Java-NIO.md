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
- 关闭通道除了需要关闭通道之外，还需要把SelectionKey[]数组上的SelectionKey取消

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


