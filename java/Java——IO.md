# Java IO #
Java的IO流分两种

- 字节IO流：read以及write的基本数据对象都是字节byte
- 字符IO流：read以及write的基本数据对象都是字符char，而字符可以理解为字节加上编码集

## 字节流 ##
字节流的一个最大特征就是都是以InputStream和OutputStream结尾，在字节流中InputStream和OutputStream最为所有字节流的父类，Inputstream和OutputStream都是抽象类，read()和write()方法都声明为抽象方法。

- **ByteArrayInputstream & ByteArrayOutputStream**：分别继承于InputStream和OutputStream。都是对其内部的数组进行读写操作。

baos：内部定义了一个字节数组，数组的默认长度为32，当我们调用write(int b), write(byte[] b, int off, int len)等写方法的时候，其实就是将字节写入到baos内部的字节数组中。

bais:内部定义了一个字节数组，在创建ByteArrayInputStream对象是，必须要传入一个字节数组，传入的数组或赋值给bais内部的字节数组，当我们调用read(), read(byte[] b, int off, int len)等方法的时候其实就是在读取内部字节数组然后返回值。

- **PipedInputStream & PipedOutputStream**：分别继承于InputStream和OutputStream。而且这两个类必须搭配在一起使用，完成多线程之间的通信，一个线程通过PipedOutputStream往管道中写输入，另一个线程通过PipedInputStream从管道中读取数据，完成多线程之间的数据交换。

pos:在PipedOutputStream类中定义了一个PipedInputStream对象，并定义了connect()方法与PipedInputStream对象连接。往管道中写入数据时，调用write(), write(byte[] b, int off, int len)等写方法,这些方法最终都会调用PipedInputStream类中receive()方法。

pis:在PipedInputStream类中定义了一个PipedOutputStream对象，并且定义个一个字节数组，用于存放数据、一个int值in，用于控制写数据、一个int值out，用于控制读数据.初始化时in为-1，out为0.在receive()方法中

    protected synchronized void receive(int b) throws IOException {
        checkStateForReceive();
        writeSide = Thread.currentThread();
        //数组满了，写线程阻塞，等待被唤醒
		if (in == out)
            awaitSpace();
        //数组为空的状态，完成初始化
		if (in < 0) {
            in = 0;
            out = 0;
        }
		//往数组中放入元素
        buffer[in++] = (byte)(b & 0xFF);
        //循环数组使用
		if (in >= buffer.length) {
            in = 0;
        }
    }

在read()方法中：
	
	public synchronized int read()  throws IOException {
	        if (!connected) {
	            throw new IOException("Pipe not connected");
	        } else if (closedByReader) {
	            throw new IOException("Pipe closed");
	        } else if (writeSide != null && !writeSide.isAlive()
	                   && !closedByWriter && (in < 0)) {
	            throw new IOException("Write end dead");
	        }
	
	        readSide = Thread.currentThread();
	        int trials = 2;
			//数组中没数据，读线程阻塞，并且唤醒写线程，等待被写线程唤醒
	        while (in < 0) {
	            if (closedByWriter) {
	                /* closed by writer, return EOF */
	                return -1;
	            }
	            if ((writeSide != null) && (!writeSide.isAlive()) && (--trials < 0)) {
	                throw new IOException("Pipe broken");
	            }
	            /* might be a writer waiting */
	            notifyAll();
	            try {
	                wait(1000);
	            } catch (InterruptedException ex) {
	                throw new java.io.InterruptedIOException();
	            }
	        }
			//读出数据
	        int ret = buffer[out++] & 0xFF;
	        if (out >= buffer.length) {
	            out = 0;
	        }
			//此时数组为空了，把in又置为了-1，回到初始状态
	        if (in == out) {
	            /* now empty */
	            in = -1;
	        }
	
	        return ret;
	    }

- **ObjectOutputStream & ObjectInputStream**：对象持久化。对象持久化有两种方式：

1）实现java.io.Serializable接口。只要一个类实现这个接口，那么这个类对象就可以被持久化，但是会有两个方面的问题，一是transient变量无法被持久化（transient变量反序列化出来之后的值时默认值），二是静态变量无法被持久化（静态变量反序列化出来之后的值与序列化的时候的值不一样）

2）实现Externalizable接口。实现这个接口就必须要重写这个接口中的抽象方法，readExternal(InputStream in)方法和writeExterval(OutputStream out)方法，在这两个方法中可以自己决定要序列化哪些变量或者对象，即使是transient变量。但是有一点要注意，实现Externalizable接口的时候必须要给出一个无参数构造器，而且此时readExternal(),writeExternal()方法都是public的，安全性可见一般。

3)另外，还有一种方式是实现Serializable接口，并且自定义序列化方法writeObject(ObjectOutputStream out)和反序列化方法readObject(ObjectInputStream in)，在这两个方法中可以自己决定要序列化哪些变量或对象，即使是transient类型。

关于序列化很重要的一点是序列ID，如果加上了序列ID，对一个对象序列化之后，我们修改了类，在类中加了部分属性，此时仍然可以反序列成功，之后添加的属性为默认值；但是如果不加序列ID，那么将会反序列化失败并抛出异常。

- **FileDescriptor**:文件描述符。关于文件描述符有三个对象：out, in, err分别对象标准输入流（键盘）、标准输出流（控制台）、标准错误流。System.out的out对象其实就是FileDescriptor的out对象。


- **BufferedInputStream & BufferedOutputStream**:继承于FilterInputStream和FilterOutputStream类。同于装饰其他流，具体体现就是在BufferedInputStream类中定义了一个 InputStream对象， 而BufferedOutputStream类中定义了一个OutputStream对象，对底层的流提供缓存功能。

bos:在创建BufferedOutputStream对象的时候，需要传入一个OutputStream对象，BufferedOutputStream将为这个对象提供缓存功能，在BufferedOutputStream类中有一个字节数组，默认大小为8k，往流中写入数据时，会先将数据写入到BufferedOutputStream类中的字节数组中，而不会直接写入到底层流中（效率考虑），只有调用了flush()方法、close()方法或者数组满了才会调用底层的流写入数据。另外，如果一次写入的数据大于了缓存数组的长度，那么将会直接调用底层流将数据写入（避免了先写缓存在从缓存写到底层流的过程）。

bis:在创建BufferedInputStream对象的时候，需要传入一个InputStream对象，BufferedInputStream将为这个对象提供缓存功能，在BufferedInputStream类中有一个字节数组，默认大小为8K。当我们调用read()方法从底层流中读取数据时，会一次性读取数组长度个字节并保存到数组中（效率高出每次读一个字节很多），由于BufferedInputStream是支持mark的，因此当数组中的数据全部使用完了，需要再次从底层流中读取数据的时候：

如果(mark>0)，表示设定了mark值，从mark到position之间的数据需要保留，因此需要先将数组在[mark~poaition]之间的数据拷贝到数组的最前端，然后从继续从底层流中读取(length-position)个字节放到数组的尾部。

如果(mark==0)，表示从[0~position]之间的数据都要保留，那么此时数组将没有剩余的空间继续从底层流中读取数据。此时如果数组长度达到了marklimit（mark()方法的传入的参数），那么将抛弃所有的数据，从数组下标0开始继续从底层流中读取数据；如果数组长度还没达到marklimit，那么数组将扩容至两倍（一直到marklimit），然后保留标记数据，并继续从底层流中读取数据。

	   private void fill() throws IOException {
	        byte[] buffer = getBufIfOpen();
			//没有标记数据，从下标0开始读
	        if (markpos < 0)
	            pos = 0;            /* no mark: throw away the buffer */
	        else if (pos >= buffer.length)  /* no room left in buffer */
	            if (markpos > 0) {  /* can throw away early part of the buffer */
	                //有标记数据
					int sz = pos - markpos;
	                System.arraycopy(buffer, markpos, buffer, 0, sz);
	                pos = sz;
	                markpos = 0;
	            } else if (buffer.length >= marklimit) {
					//数组长度达到了marklimit
	                markpos = -1;   /* buffer got too big, invalidate mark */
	                pos = 0;        /* drop buffer contents */
	            } else if (buffer.length >= MAX_BUFFER_SIZE) {
	                throw new OutOfMemoryError("Required array size too large");
	            } else {            /* grow buffer */
					//没有达到marklimit，扩容至两倍
	                int nsz = (pos <= MAX_BUFFER_SIZE - pos) ?
	                        pos * 2 : MAX_BUFFER_SIZE;
	                if (nsz > marklimit)
	                    nsz = marklimit;
	                byte nbuf[] = new byte[nsz];
	                System.arraycopy(buffer, 0, nbuf, 0, pos);
	                if (!bufUpdater.compareAndSet(this, buffer, nbuf)) {
	                    // Can't replace buf if there was an async close.
	                    // Note: This would need to be changed if fill()
	                    // is ever made accessible to multiple threads.
	                    // But for now, the only way CAS can fail is via close.
	                    // assert buf == null;
	                    throw new IOException("Stream closed");
	                }
	                buffer = nbuf;
	            }
	        count = pos;
	        int n = getInIfOpen().read(buffer, pos, buffer.length - pos);
	        if (n > 0)
	            count = n + pos;
	    }

- **DataInputStream & DataOutputStream**：继承于FilterInputStream和FilterOutputStream类。因此这两个也是装饰类，它们的作用是为底层流提供基本数据类型以及UTF字符串的读写操作，这两个类的搭配使用能够实现与及其无关的基本数据类型的读写，这是因为这两个类在写入基本数据类型的时候都转换为字节来操作。
	
	    public final int readInt() throws IOException {
	        int ch1 = in.read();
	        int ch2 = in.read();
	        int ch3 = in.read();
	        int ch4 = in.read();
	        if ((ch1 | ch2 | ch3 | ch4) < 0)
	            throw new EOFException();
	        return ((ch1 << 24) + (ch2 << 16) + (ch3 << 8) + (ch4 << 0));
	    }
写入/读取一个int数的时候，先读取一个字节为int数的最开始的8位、然后再读取一个字节为int值的接下来8位，读完四个字节后在拼揍成一个int值。

- **PrintStream**：继承于FilterOutputStream类，也是一个装饰类，作用就是为底层流提供各种print函数。由于PrintStream仍然属于字节流，无法操作字符，因此在PrintStream类中通过OutputStreamWriter以及BufferedWriter类实现对字符的打印，几乎所有的print()方法调用的都是BufferedWriter类中的方法，但是两个write方法write(int b), write(byte[] b, int off, int len)由于不涉及到字符操作，因此直接使用的是底层流的write方法

## 字符流 ##
字符流及基础父类分别是Reader和Writer，因此字符流中的所有类都是以Reader和Writer结尾。

- **CharArrayReader & CharArrayWriter**：操作内部字符数组的Reader和Writer，对应于字节流中ByteArrayInputStream和ByteArrayOutputStream。
- **PipedReader & PipedWriter**：管道通信，实现多线程间的数据交换。对应于字节流中PipedInputStream & PipedOutputStream.
- **InputStreamReader & OutputStreamWriter**：这两个类是实现字节流和字符流包装的桥梁，这两个类属于字符流，但是在构造方法中需要传入字节流对象作为参数，将底层的字节用字符集包装之后读出来为字符。
- **FileReader & FileWriter**：分别继承于InputStreamReader 和 OutputStreamWriter，专门用于将文件中的字节流读出来为字符，即构造器中传入的字节流对象规定为FileInputStream或FileOutputStream。
- **BufferedWriter & BufferedReader**：包装类，包装一个Reader或者Writer，为底层流提供缓存功能，对应于字节流中BufferedInputStream & BufferedOutputStream。
- **PrintWriter**：包装类，包装一个Writer，并为这个Writer提供很多print方法。对应于字节流中的PrintStream。



## RandomAccessFile类 ##
RandomAccessFile类既没有继承于字节流，也没有继承于字符流，这个类既能读取文件中的内容，也能往文件中写入数据，并且可以通过seek()方法修改文件游标，实现随机访问文件中的任意位置的数据。