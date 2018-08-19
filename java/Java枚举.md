# Java枚举的实现原理 #
[http://blog.csdn.net/javazejian/article/details/71333103 ](http://blog.csdn.net/javazejian/article/details/71333103 )
## Java枚举是什么 ##

	//枚举类型，使用关键字enum
	enum Day {
	    MONDAY, TUESDAY, WEDNESDAY,
	    THURSDAY, FRIDAY, SATURDAY, SUNDAY
	}
通过enum关键字定义的一个有限集合就是一个枚举。与class关键字定义的类一样，枚举经过编译之后，也会产生一个字节码文件：
	
	//反编译Day.class
	final class Day extends Enum
	{
	    //编译器为我们添加的静态的values()方法
	    public static Day[] values()
	    {
	        return (Day[])$VALUES.clone();
	    }
	    //编译器为我们添加的静态的valueOf()方法，注意间接调用了Enum也类的valueOf方法
	    public static Day valueOf(String s)
	    {
	        return (Day)Enum.valueOf(com/zejian/enumdemo/Day, s);
	    }
	    //私有构造函数
	    private Day(String s, int i)
	    {
	        super(s, i);
	    }
	     //前面定义的7种枚举实例
	    public static final Day MONDAY;
	    public static final Day TUESDAY;
	    public static final Day WEDNESDAY;
	    public static final Day THURSDAY;
	    public static final Day FRIDAY;
	    public static final Day SATURDAY;
	    public static final Day SUNDAY;
	    private static final Day $VALUES[];
	
	    static 
	    {    
	        //实例化枚举实例
	        MONDAY = new Day("MONDAY", 0);
	        TUESDAY = new Day("TUESDAY", 1);
	        WEDNESDAY = new Day("WEDNESDAY", 2);
	        THURSDAY = new Day("THURSDAY", 3);
	        FRIDAY = new Day("FRIDAY", 4);
	        SATURDAY = new Day("SATURDAY", 5);
	        SUNDAY = new Day("SUNDAY", 6);
	        $VALUES = (new Day[] {
	            MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY
	        });
	    }
	}
经过编译之后的枚举可以看出转化为了一个类，并且继承于Enum这个抽象类。定义的每一个枚举中的元素，变成了对象，一个枚举元素就是一个枚举对象。除了定义的枚举对象之外，编译器还为这个类添加了private类型的构造方法，静态方法values()方法用于以数组形式返回所有的枚举对象，静态方法valueOf(String s)方法通过给定的枚举对象的名字返回这个枚举对象，这个方法实际调用的是父类的valueOf(Class<?> clz, String s)方法。并且在静态块中实例化了定义的枚举对象，调用的就是编译器自己加的private构造方法，并且将所有的枚举对象构成一个数组赋值给编译器自己加的$VALUES[]数组。
## Java枚举的共同父类----Enum ##
**Enum抽象类的常见方法**

- compareTo(E o)	比较此枚举与指定对象的顺序，比较的是序号
- equals(Object other)	比较指定对象 == 当前对象，返回 true。
- getDeclaringClass()	返回与此枚举常量的枚举类型相对应的 Class 对象
- name()	返回此枚举常量的名称
- ordinal()	返回枚举常量的序数，从0开始编号
- valueOf(Class<T> enumType, String name)	静态方法，返回枚举对象。枚举类编译之后编译器加的valueOf()方法就是调用这个方法

注意：枚举类编译之后，编译器加的values()方法在父类Enum中是没有，所以如果把枚举对象向上转型为父类对象，那么这个父类对象是没有values()方法的。

**Class对象对枚举的支持**

- getEnumConstants()	返回该枚举类型的所有元素，如果Class对象不是枚举类型，则返回null。
- isEnum()	当且仅当该类声明为源代码中的枚举时返回 true

		//枚举对象向上转型
		Enum<?> e = Day.MONDAY;
		//e没有了values()方法
		//e.values();
		//这里调用的getDeclaringClass()是子类的方法，因此返回的是Day的Class对象
	
		Class<?> clz = e.getDeclaringClass();
		if(clz.isEnum()){
			Day[] days = clz.getEnumConstants();
		}

**枚举的进阶用法**

- 枚举类中定义构造函数：只能定义private的构造函数，而且我们无法调用构造函数创建枚举对象，只有JVM才能创建枚举对象；

- 覆盖抽象类Enum中的方法：但是抽象类Enum类只有toString()方法不是final修饰的

- 枚举类定义抽象方法：可以在枚举中定义抽象方法，但是每个枚举实例都必须要重写这个抽象方法
	
	
		public enum Day{
			MONDAY{
				@Override
				publi void test(){
				}
			};
			public abstract void test();
		}
	
- 枚举类实现接口：枚举类可以实现接口，但是不能继承一个类；毕竟所有枚举都默认继承Enum这个抽象类

- 枚举嵌套：在一个枚举中可以定义另外的枚举

- 枚举支持switch：枚举对象用在case中时不需要使用 枚举名.枚举对象 的形式.

		enum Color {GREEN,RED,BLUE}
	
		public static void printName(Color color){
				switch (color){
					case BLUE: //无需使用Color进行引用
						System.out.println("蓝色");
						break;
					case RED:
						System.out.println("红色");
						break;
					case GREEN:
						System.out.println("绿色");
						break;
				}
			}
		
**枚举与单例模式**

- 单例模式一：饿汉式；是线程安全的，但是不具备延迟加载特性。

		public class Singleton{
			private static Singleton ins = new Singleton();
			private Singleton(){}
			public static Singleton getIns(){
				return ins;
			}
		}
- 单例模式二：懒汉式；使用同步方法确保线程安全，具备延迟加载，但是同步方法开销较大


		public class Singleton{
			private static volatile Singleton ins;
			private Singleton(){}
			public synchronized static Singleton getIns(){
				if(ins == null){
					ins = new Singleton();
				}
				return ins;
			}
		}
- 单例模式三：双重锁检查；使用同步块保证线程安全，具备延迟加载


		public class Singleton{
			private static volatile Singleton ins;
			private Singleton(){}
			public static Singleton getIns(){
				if(ins == null){
					synchronized(Singleton.class){
						if(ins == null){
							ins = new Singleton();
						}
					}
				}
				return ins;
			}
		}
- 单例模式四：内部类；保证线程安全，具备延迟加载。静态内部类只会被加载一次


		public class Singleton{
			private static class Inner{
				private static Singleton ins = new Singleton();
			}
			private Singleton(){}
			public static Singleton getIns(){
				return Inner.ins;
			}
		}
**单例模式必须考虑的四个问题：线程安全、延迟加载、序列化安全、反射安全。**


以上四种模式在反序列化时，每次反序列化出来的对象都是一个新对象，即不是序列化安全的：


		public class Singleton{
			private static Singleton ins = new Singleton();
			private Singleton(){}
			public static Singleton getIns(){
				return ins;
			}
			//反序列化的时候直接返回对象
			private Object readObject(OutputStream os){
				return ins;
			}
		}
通过反射获取单例类的私有构造函数，仍然可以创建类的实例：

		public class Singleton{
			private static Singleton ins = new Singleton();
			private static volatile boolean flag = true;
			private Singleton(){
				if(flag){
					flag = false;
				} else {
					throw new Exception();
				}
			}
			public static Singleton getIns(){
				return ins;
			}
		}
- 单例模式五：枚举单例；通过枚举实现单例模式，枚举在序列化时真正序列化的不是枚举对象，而是枚举对象的name，从而反序列化的时候通过valueOf(name)方法获取枚举对象，这保证了序列化安全;获取单例类的class对象，Class类的newInstance()方法会调用枚举类的构造函数，但是枚举类的构造函数只能被JVM调用；或者获取Constractor对象，调用Constractor类的newInstance()方法，但是修饰为enum的会直接抛出异常，因此，也是反射安全的。

		public enum Singleton{
			INS;
			
		}

## 枚举的专属Map----EnumMap ##

EnumMap是Map的一个实现类，是专门为枚举所用的Map，因为EnumMap的key值必须为Enum类型，即key必须是枚举类型。

>Map<Color,Integer> enumMap=new EnumMap<>(Color.class);


构造一个EnumMap只需要传入枚举类型的Class对象，EnumMap内部维护这两个数组：

key值数组：枚举的所有枚举对象按声明顺序存储在这个数组中

value数组：数组下标对应于枚举对象的ordinal()下标值，存储的是每个key值对应的value值。

在EnumMap中key不能为null，因为key必须为某个枚举对象。

在EnumMap中value在初始时都为null，为null表示这个EnumMap集合还没包含对应的key值；

如果往EnumMap中put(Color.RED, null)是允许的，但是实际存储在EnumMap中的是(Color.RED, NULL)，即加入value为null的时候会被包装成NULL（NULL定义为一个Object对象）。

其它的使用和HashMap是一样的。

## Java中的位向量----EnumSet的底层原理 ##

位向量的意思就是用一个bit位来表示一个数是否存在，比如想把40存在一个int数组中去，我们可以使用一个bit位来表示：

40/32 = 1：表示40这个数在int数组中的下标为1

40%32 = 8：表示40这个数在a[1]这个int值的第9位（第0位表示的是32这个数）

那么如果a[1]的第9位为1表示存在40这个数。

二进制表示这个过程为：

//40

00000000 00000000 00000000 00101000

//40/32等价于40>>5，得到的数就对应在数组中的下标

00000000 00000000 00000000 00000001

//40%32等价于40&（32-1），

00000000 00000000 00000000 00101000

00000000 00000000 00000000 00011111

//得到的数为8，那么在a[1]中的第(1<<8)位置为1

00000000 00000000 00000000 00001000 

将位向量添加一个数、删除一个数、查询一个数是否存在写成一个类：

	public class BitVector{
		private int count;
		private int[] a;
		private int BIT_LENGTH = 32;
		private int SHIFT = 5;
		private int MASK = 0x1f;  //2^5-1
		
		public BitVector(int count){
			this.count = count;
			a = new int[(count-1)/BITLENGTH + 1];
		}
		//添加一个数
		public void set(int i){
			int p = i>>SHIFT;
			int s = i & MASK;
			a[p] |= 1<<s;
		}
		//删除一个数
		public void clear(int i){
			int p = i>>SHIFT;
			int s = i & MASK;
			a[p] &= ~(1<<s);
		}
		//查询一个数是否存在
		public boolean get(int i){
			int p = i>>SHIFT;
			int s = i & MASK;
			return Integer.bitCount(a[p] & (1<<s));
		}
	}

## 枚举的专属Set集合-----EnumSet ##

EnumSet是一个抽象类，只能存放枚举对象。底层基于位向量实现，即往集合中加入、删除、查询等操作都是基于位运算。

EnumSet有两个具体的实现类，对于枚举个数小于等于64个的，会创建RegularEnumSet，而对于枚举个数大于64个的，会创建JumboEnumSet。

EnumSet类是个抽象类，我们只能使用EnumSet的静态工厂方法创建EnumSet的实例对象，静态方法有：

	创建一个具有指定元素类型的空EnumSet。
	EnumSet<E>  noneOf(Class<E> elementType)       
	//创建一个指定元素类型并包含所有枚举值的EnumSet
	<E extends Enum<E>> EnumSet<E> allOf(Class<E> elementType)
	// 创建一个包括枚举值中指定范围元素的EnumSet
	<E extends Enum<E>> EnumSet<E> range(E from, E to)
	// 初始集合包括指定集合的补集
	<E extends Enum<E>> EnumSet<E> complementOf(EnumSet<E> s)
	// 创建一个包括参数中所有元素的EnumSet
	<E extends Enum<E>> EnumSet<E> of(E e)
	<E extends Enum<E>> EnumSet<E> of(E e1, E e2)
	<E extends Enum<E>> EnumSet<E> of(E e1, E e2, E e3)
	<E extends Enum<E>> EnumSet<E> of(E e1, E e2, E e3, E e4)
	<E extends Enum<E>> EnumSet<E> of(E e1, E e2, E e3, E e4, E e5)
	<E extends Enum<E>> EnumSet<E> of(E first, E... rest)
	//创建一个包含参数容器中的所有元素的EnumSet
	<E extends Enum<E>> EnumSet<E> copyOf(EnumSet<E> s)
	<E extends Enum<E>> EnumSet<E> copyOf(Collection<E> c)
根据新建的EnumSet的是否包含元素，可以调用不同的of方法

**RegularEnumSet类**

对于枚举个数小于等于64个，在of方法中会创建RegularEnumSet实例，RegularEnumSet类使用一个long类型的变量的每一位记录是否包含对应ordinal()的枚举对象。

	public boolean add(E e){
		//不允许加入null
		if(e == null){
			//抛异常
		}
		long oldElement = elements;
		elements |= (1L<<(Enum)e.ordinal());
		return oldElement!=elements;
	}
加入一个元素就将elements变量的对应位设置为1

	public boolean remove(E e){
		//不允许加入null
		if(e == null){
			//抛异常
		}
		long oldElement = elements;
		elements &= ~(1L<<(Enum)e.ordinal());
		return oldElement!=elements;
	}
移除一个元素就是将elements变量的对应位设置为0

	public boolean contains(E e){
		//不允许加入null
		if(e == null){
			//抛异常
		}
		return (elements & (1L<<(Enum)e.ordinal()) != 0);
	}
如果包含这个元素那么对应位值&不应该为0

	public boolean containsAll(Collention<?> c){
		if(!(c instanceof RegularEnumSet)){
			//如果不是RegularEnumSet类型，交给父类去判断，父类会将c作为一个普通的集合处理
			return super.containsAll(c);
		}
		RegularEnumSet<?> es = (RegularEnumSet<?>)c;
		if(c.getClass() != elementsType){
			//类型不一样，如一个是Day，另一个是Color
			return es.isEmpty();
		}
		//c与elements的补集没有交集，那么c是elements的子集
		return (~elements & es.elements) == 0);
	}	
RegularEnumSet类迭代枚举对象的过程：

	public boolean hasNext() {
            return elements != 0;
        }

    @SuppressWarnings("unchecked")
    public E next() {
            if (elements == 0)
                throw new NoSuchElementException();
     
            // -elementswei elements按位取反再加一，对应的就是elemens的最低位上的第一个1
			lastReturned = elements & -elements;
            //取值后减去已取得lastReturned
            elements -= lastReturned;
            //返回在指定 long 值的二进制补码表示形式中最低位（最右边）的 1 位之后的零位的数量
            return (E) universe[Long.numberOfTrailingZeros(lastReturned)];
    }
迭代RegularEnumSet需要通过itreator方法获取迭代器，迭代获取下一个元素的原理就是通过位运算找到elements最低位的第一个1，然后通过存储着所有枚举对象的universe数组获取对应的枚举对象。

**JumboEnumSet**

JumboEnumSet底层使用long数组来存储是否包含对应的枚举对象，当枚举个数大于64个时会创建JumboEnumSet对象。

每个方法在实现上与RegularEnumSet的唯一区别在于多了一个获取数组下标的过程，通过e.ordinal()>>>6获取在数组中的下标。