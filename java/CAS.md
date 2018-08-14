# Compare And Swap #
## CAS的思想 ##
CAS是一种无锁策略，其核心思想为CAS(V,E,N)对变量V有一个预期值，如果读出来的值与预期值一样，那么更新变量V的值为新值N，如果读出来的值与预期值不一致，说明这个值已经被其他线程修改，这是可以重新读取变量的值或者放弃操作。

CAS是一个操作系统原语，其执行过程不会被其他线程打断，因此不存在这种情况：读出来的值和预期值一样，正要写入新值时切换到其他线程执行，并且把变量V的值更改为其他值，从早造成数据的不一致性。当多个线程同时对同一变量做CAS操作时，只会有一个线程胜出，其他的线程均会失败，失败的线程允许重新尝试或者放弃操作。

## Unsafe类 ##
Unsafe类位于sun.misc包下，需要查看openjdk源码才能看到这个类的源码。Unsafe类为CAS操作提供了支持，Unsafe类可以直接操作内存，分配内存、释放内存等操作，Unsafe类只能被特定的一些类实例化，用户自定义的类无法获得Unsafe类对象，只能通过反射获取theUnsafe字段。

	        // 通过反射得到theUnsafe对应的Field对象
	        Field field = Unsafe.class.getDeclaredField("theUnsafe");
	        // 设置该Field为可访问
	        field.setAccessible(true);
	        // 通过Field得到该Field对应的具体对象，传入null是因为该Field为static的
	        Unsafe unsafe = (Unsafe) field.get(null);
Unsafe类中的所有方法都是native的，主要功能有一下几点：

**- 操作内存。**

 //分配内存指定大小的内存

>public native long allocateMemory(long bytes);

//根据给定的内存地址address设置重新分配指定大小的内存

>public native long reallocateMemory(long address, long bytes);

//用于释放allocateMemory和reallocateMemory申请的内存

>public native void freeMemory(long address);

//将指定对象的给定offset偏移量内存块中的所有字节设置为固定值

>public native void setMemory(Object o, long offset, long bytes, byte value);

//设置给定内存地址的值

>public native void putAddress(long address, long x);

//获取指定内存地址的值

>public native long getAddress(long address);


//设置给定内存地址的long值

>public native void putLong(long address, long x);

//获取指定内存地址的long值

>public native long getLong(long address);

//设置或获取指定内存的byte值

>public native byte  getByte(long address);
>
public native void  putByte(long address, byte x);

//其他基本数据类型(long,char,float,double,short等)的操作与putByte及getByte相同

//操作系统的内存页大小

>public native int pageSize();

**-创建对象**

//通过一个类的class，在不调用构造函数的前提下创建该类的对象。

>public native Object allocateInstance(Class cls) throws InstantiationException;

**-操作类和对象以及类字段**

//获取字段f的内存地址

>public native long objectFieldOffset(Field f);

//获取静态字段的内存地址

>public native long staticFieldOffset(Field f);

//获取这个静态字段对应的是那个类

>public native Object staticFieldBase(Field f);


//获取/设置这个对象在这个内存地址的int值

>public native int getInt(Object o, long offset);

>public native void putInt(Object o, long offset, int x);


//获取/设置这个对象在这个内存地址的引用类型的值

>public native Object getObject(Object o, long offset);

>public native void putObject(Object o, long offset, Object x);

//其他基本数据类型(long,char,byte,float,double)的操作与getInthe及putInt相同

//设置给定对象的int值，使用volatile语义，即设置后立马更新到内存对其他线程可见

>public native void  putIntVolatile(Object o, long offset, int x);

//获得给定对象的指定偏移量offset的int值，使用volatile语义，总能获取到最新的int值。

>public native int getIntVolatile(Object o, long offset);

//其他基本数据类型(long,char,byte,float,double)的操作与putIntVolatile及getIntVolatile相同，引用类型putObjectVolatile也一样。

//与putIntVolatile一样，但要求被操作字段必须有volatile修饰

>public native void putOrderedInt(Object o,long offset,int x);

**-操作数组**
//获取数组第一个元素的偏移地址

>public native int arrayBaseOffset(Class arrayClass);

//数组中一个元素占据的内存空间,arrayBaseOffset与arrayIndexScale配合使用，可定位数组中每个元素在内存中的位置

>public native int arrayIndexScale(Class arrayClass);

**-CAS操作**
//第一个参数o为给定对象，offset为对象内存的偏移量，通过这个偏移量迅速定位字段并设置或获取该字段的值，

//expected表示期望值，x表示要设置的值，下面3个方法都通过CAS原子指令执行操作。

>public final native boolean compareAndSwapObject(Object o, long offset,Object expected, Object x);                                                                                                  

>public final native boolean compareAndSwapInt(Object o, long offset,int expected,int x);

>public final native boolean compareAndSwapLong(Object o, long offset,long expected,long x);

JDK1.8中新增了几个CAS操作，新增的操作基于上面三个方法：

	 //1.8新增，给定对象o，根据获取内存偏移量指向的字段，将其增加delta，
	 //这是一个CAS操作过程，直到设置成功方能退出循环，返回旧值
	public final int getAndAddInt(Object o, long offset, int delta){
		long v;
		do{
			v = getIntVolatile(o, offset);
		} while (!compareAndSwap(o, offset, v, v+delta));
		return v;
	}
	//1.8新增，方法作用同上，只不过这里操作的long类型数据
	 public final long getAndAddLong(Object o, long offset, long delta) {
	     long v;
	     do {
	         v = getLongVolatile(o, offset);
	     } while (!compareAndSwapLong(o, offset, v, v + delta));
	     return v;
	 }
	
	 //1.8新增，给定对象o，根据获取内存偏移量对于字段，将其 设置为新值newValue，
	 //这是一个CAS操作过程，直到设置成功方能退出循环，返回旧值
	 public final int getAndSetInt(Object o, long offset, int newValue) {
	     int v;
	     do {
	         v = getIntVolatile(o, offset);
	     } while (!compareAndSwapInt(o, offset, v, newValue));
	     return v;
	 }
	// 1.8新增，同上，操作的是long类型
	 public final long getAndSetLong(Object o, long offset, long newValue) {
	     long v;
	     do {
	         v = getLongVolatile(o, offset);
	     } while (!compareAndSwapLong(o, offset, v, newValue));
	     return v;
	 }
	
	 //1.8新增，同上，操作的是引用类型数据
	 public final Object getAndSetObject(Object o, long offset, Object newValue) {
	     Object v;
	     do {
	         v = getObjectVolatile(o, offset);
	     } while (!compareAndSwapObject(o, offset, v, newValue));
	     return v;
	 }
**-线程的挂起和恢复**

//线程调用该方法，线程将一直阻塞直到超时，或者是中断条件出现。  
>public native void park(boolean isAbsolute, long time);  

//终止挂起的线程，恢复正常.java.util.concurrent包中挂起操作都是在LockSupport类实现的，其底层正是使用这两个方法，  
>public native void unpark(Object thread); 


## java.util.concurrent.atomic包 ##
**AtomicInteger：**为int类型的变量实现CAS操作。类似的实现类还有**AtomicLong、AtomicBoolean**。

	public class AtomicInteger{
		private static final Unsafe unsafe = Unsafe.getInstance();
		private volatile int value;
		private static final  long valueoffset;
	
		static{
			valueoffset = unsafe.objectFieldOffset(AtomicInteger.class.getDeclaredField("value"));
		}
		//获取并设置新值
		public final int getAndSet(int newValue) {
			return unsafe.getAndSetInt(this, valueoffset, newvalue);
		}
		//获取并加1
		public final int getAndIncrement() {
			return unsafe.getAndAddInt(this, valueoffset, 1);
		}
	}

**AtomicReference：**为引用类型实现CAS更新操作，更新的是这个引用的所有字段。

	public class ReferenceInteger<V>{
		private static final Unsafe unsafe = Unsafe.getInstance();
		private volatile V value;
		private static final  long valueoffset;
	
		static{
			valueoffset = unsafe.objectFieldOffset(ReferenceInteger.class.getDeclaredField("value"));
		}
		//获取并设置新值
		public final int getAndSet(V newValue) {
			return (V)unsafe.getAndSetObject(this, valueoffset, newvalue);
		}
		
	}

**AtomicIntegerArray:**为int类型的数组的的每个元素提供CAS更新。类似的实现类还有**AtomicLongArray、AtomicReferenceArray**。

	public class AtomicIntegerArray{
		private static final Unsafe unsafe = Unsafe.getInstance();
		//获取数组的首地址
		private static final int baseoffset = unsafe.arrayBaseOffset(int[].class);
		private final int[] array;
		private final static int shift;
	
		static{
			//获取每个元素占用多少个字节 int---scale=4
			int scale = unsafe.arrayIndexScale(int[].class);
			//Integer.numberOfLeadingZeros(scale)将scale转换为32位的二进制，1左边的0的个数 shift=2
			shift = 31 - Integer.numberOfLeadingZeros(scale);
		}
	
		public AtomicIntegerArray(int length){
			array = new int[length];
		}
		
		public final int getAndIncrement(int i) {
	      return getAndAdd(i, 1);
		}
		public final int getAndAdd(int i, int delta) {
	    	long offset = checkedByteOffset(i);
			while(true){
				int current = getRaw(offset);
				if(compareAndSetRaw(offset, current, current+delta)){
					return current;
				}
			}
		}
		private boolean compareAndSetRaw(long offset, int expect, int update) {
	        return unsafe.compareAndSwapInt(array, offset, expect, update);
	    }
		//这里使用的也是volatile，因为数组中的每个元素并不是volatile声明的，这里使用带volatile方法很有必要
		private int getRaw(long offset){
			return unsafe.getIntVolatile(array, offset);
		}
	
		private long checkedByteOffset(int k){
			if(k<0 || k>=array.length){
				//下标越界
			}
			//返回下标对应的地址
			return baseoffset+(k<<shift);
		}
		
	}

**AtomicIntegerFieldUpdater：**为一个类的int类型的字段提供原子更新。类似的实现类还有**AtomicLongFieldUpdater、AtomicReferenceFieldUpdater**

	public abstract class  AtomicIntegerFieldUpdater<T> {
		//AtomicIntegerFieldUpdater本身是一个抽象类，提供了静态方法newUpdater()返回实现类的实例
		public static <U> AtomicIntegerFieldUpdater<U> newUpdater(Class<U> tclass, String fieldName) {
	        return new AtomicIntegerFieldUpdaterImpl<U>(tclass, fieldName, Reflection.getCallerClass());
	    }
		
		//
		Field field = tclass.getDeclaredField(fieldName);
		//
		long fieldOffset = unsafe.objectFieldOffset(field);
	
	
		public int getAndAdd(T obj, int delta) {
	        for (;;) {
	            int current = get(obj);
	            int next = current + delta;
	            if (compareAndSet(obj, current, next))
	                return current;
	        }
	    }
	
		publci int get(T obj){
			return unsafe.getIntVolatile(obj, offset);
		}
	
		public boolean compareAndSet(Object o, int expact, int newValue){
			return unsafe.compareAndSwapInt(o, fieldOffset, expact, newValue);
		}
	}
AtomicIntegerFieldUpdater的CAS更新字段对字段有一定要求：1）字段不能是static修饰，2）字段必须是volatile修饰，3）字段必须对更新类可见，4）字段不能是final

**CAS的ABA问题：**ABA问题指的是一个线程A先获取到变量V的值，在CAS更新变量值之前，另外的线程将变量V的值连续改变了两次，连续改变两次之后变量V的值没变，此时线程A获取到变量V的值与期望的值一致，因此认为变量V的值没有发生改变，这可能会引起一些错误。比如一个栈，线程A获取到栈顶为top，线程A想要入栈一个元素，在CAS更新前，另一线程执行操作出栈一个元素，再将top入栈，此时线程A无法判断出top已经发生了改变，由此引起问题。

**AtomicStampedReference：**利用时间戳解决ABA问题。

	public class AtomicStampedReference<T>{
		//将要修改的变量和对应的时间戳构造为一个Pair
		private class Pair{
			private final T reference;
			private final int stamp;
			static Pair of(T reference, int stamp){
				return new Pair(reference, stamp);
			}
		}
	
		private static final unsafe = Unsafe.getUnsafe();
		private volatile Pair<V> pair;
		private final static pairOffset = unsafe.objectFieldOffset(AtomicStampedReference.class.getDeclaredField("pair"));
	
		//会同时比较reference和stamp
		public boolean compareAndSet(V   expectedReference,
	                                 V   newReference,
	                                 int expectedStamp,
	                                 int newStamp) {
	        Pair<V> current = pair;
	        return
	            expectedReference == current.reference &&
	            expectedStamp == current.stamp &&
	            ((newReference == current.reference &&
	              newStamp == current.stamp) ||
	             casPair(current, Pair.of(newReference, newStamp)));
	    }
	
		private boolean casPair(Pair current, Pair newValue){
			return unsafe.compareAndSwapObject(this, pairoffset, current, newValue);
		}
		
	}

