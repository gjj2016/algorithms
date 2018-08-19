## Class对象以及反射技术 ##
[http://blog.csdn.net/javazejian/article/details/70768369 ](http://blog.csdn.net/javazejian/article/details/70768369 )
## Class对象 ##
**Class类**

- Class类也是一个类，与class关键字不同
- 手动编写的类编译之后会产生一个字节码文件，这个字节码文件对应的就是Class对象，Class对象记录了这个类的所有信息。而且自定义的每个类有且仅有一个Class对象，无论我们创建多少个类对象，对应的都只有一个Class对象
- Class类的构造函数是私有化的，因此只有JVM才能创建Class对象
- Class类提供的是在运行时获取类的信息

**Class对象的创建时机及创建方法**

- Class对象的创建时机：当我们第一次调用这个类的静态成员时（构造方法也可以视为静态成员），Class对象会被创建；由于Class对象的唯一性，Class对象仅被创建一次。
- 创建Class对象方式一：Class clz = 类名.class;使用这种方式创建Class对象，类会被加载，但是不会被初始化。类加载的整个过程是：加载----验证----准备----解析----初始化；使用这种方式创建Class对象，不会触发类的初始化，（在类中定义的静态块不会执行）。
- 创建Class对象方式二：Class clz = 类对象.getClass()；getClass()方法是从Object类中继承来的，使用这种方式由于要创建类对象，所以类会被加载，同时也会被初始化。
- 创建Class对象方式三：Class clz = Class.forName("com.gjj.Test");调用Class类的静态方法创建Class对象，这种方式同样也会加载类，并且完成类的初始化。

**关于类是否会被初始化，Java虚拟机规定有且仅有一下五种情形类必须初始化**

- 调用类的静态成员、构造方法时类必须初始化。但是调用编译时常量不会触发类的初始化，编译时常量指的是在编译期间就知道值的常量，还有一类常量必须等到运行时才能确定值（如Random类给常量赋值）
- 使用反射包(java.lang.reflect)的方法对类进行反射调用时，类必须初始化
- 初始化一个类的时候，如果父类没有初始化，那么父类必须初始化
- 包含main方法的主类会先被初始化
- 动态语言支持什么的

**Class对象的泛型**

- 没有使用泛型的Class对象
> Class clz = int.class;

- 规定泛型的Class对象
> Class<Integer> clz = int.class
> 
规定了泛型之后，在编译时即可发现错误，而不用等到运行时才发现错误，如规定泛型后clz = double.class;将通不过编译

- 指定通配泛型的Class对象
> Class<?> clz = int.class;
> 
>指定通配泛型的Class对象，clz也可以指定为其他任何类型；Class<?>与Class本质上没区别，但是不指定泛型的Class可能意味着忘记了指定泛型，指定为通配泛型的Class<?>可能意味着确实想要创建一个可以指定为任一类型的Class对象 

- 指定部分匹配的Class对象
> Class<? extends Number> clz = int.class;

>此时clz只能指定为Number类或其子类


**Class对象提供的强制转换**

- 强制转换方式一：对象前面加括号，类型不匹配会抛出转换错误；
- 强制转换方式二：instanceof关键字，obj instanceof A
- 强制转换方式三：isInstance(obj)方法，isInstance()方法是Class类的native方法，A.class.isInstance(obj)，当且仅当对象obj是A的对象或者A的子类的对象时返回true。
- 强制转换方式四：cast(obj)方法，cast()方法是Class类的成员方法，A.class.cast(obj)，将对象obj转换为Class对象通配指定的的类型，内部使用仍然是instanceof关键字。

## 反射技术 ##
反射技术指的是在运行时可以获取一个类的所有属性和方法，可以动态的获取类的信息。反射技术主要靠反射包实现，常用的如Class类、Constractor类：描述构造器、Field类：描述字段、Method类：描述方法。

**反射获取类的构造方法**：需要获取类的Class对象

- clz.getConstractor(Class<?> ... parameterType);通过这个类的clz对象的getConstractor()方法，这个方法需要传入构造函数的参数列表的class对象，获取参数列表与传入的参数相同的public类型的构造函数，返回一个Constractor对象

- clz.getConstructors();通过这个类的clz对象的getConstractors()方法，会返回这个类的所有public类型的构造函数。

- clz.getDeclaredConstructor(Class<?> ... parameterType);通过这个类的clz对象的getDeclaredConstructor()方法，这个方法需要传入构造函数的参数列表的class对象，获取参数列表与传入的参数相同的private类型的构造函数，返回一个Constractor对象。


- clz.getDeclaredContractors();通过这个类的clz对象的getDeclaredContractors()方法，会返回这个类的所有构造函数（包括private的）。


- clz.newInstance();通过这个类的clz对象的newInstance()方法，返回这个类的一个对象。（）也会调用构造函数，**注意：只会调用无参构造函数，如果没有无参构造函数，这样创建对象会报错**

**Constractor类的一些方法**：Constractor类的一个对象代表一个构造函数。**注意：如果是private构造函数，那么必须设置constractor.setAccessible(true);设置为可访问。**

- 返回这个构造函数的类：c.getDeclaringClass();  这个方法会返回类的Class对象


- 获取这个构造函数的参数列表的类型：c.getGenericParameterTypes();这个方法会返回这个构造函数的参数列表的类型Type[]，Type是一个接口，Class类实现了这个接口，可以作为一个Class对象理解，但是表示范围要比Class广。


- 获取这个构造函数的参数列表的类型：c.getParameterTypes();这个方法会返回这个构造函数的参数列表的类型Class[]，每一个元素对应一个参数的Class类型


- 返回这个构造函数的名字：c.getName()；返回一个字符串，形如"com.gjj.Test"


- 返回这个构造函数的全称：c.getGenericName();返回一个字符串，形如"public com.gjj.Test(int, java.lang.String)"


- 使用这个构造函数在创建一个对象：c.newInstance(Object .. args);这个方法只能调用自身这个构造函数，不能调用其他的构造函数，这就意味着传入的参数必须和这个构造函数的参数列表对应

**反射获取类的成员变量（包括静态）**

- 获取指定的public字段：clz.getField(String fieldName);获取指定name名称、具有public修饰的字段，包含继承字段，返回Field对象。


- 获取全部的public字段：clz.getFields();获取修饰符为public的字段，包含继承字段，返回Field[]对象数组


- 获取指定字段（包括私有的）：clz.getDeclaredField(String fieldName);获取指定name名称的(包含private修饰的)字段，不包括继承的字段，返回Field对象


- 获取全部字段（包括私有的）：clz.getDeclaredFields();获取Class对象所表示的类或接口的所有(包含private修饰的)字段,不包括继承的字段，返回Field[]对象数组。

**Field类的一些方法**：Field类的一个对象代表一个成员变量。**注意：如果是private成员，那么必须设置field.setAccessible(true);设置为可访问。 另外如果字段被final修饰，在运行时可以对字段进行修改，但是最终实际值不会发生改变**

- f.set(Object obj, Object value)	将指定对象变量上此 Field 对象表示的字段设置为指定的新值。


- f.get(Object obj)	返回指定对象上此 Field 表示的字段的值


- f.getName()	返回此 Field 对象表示的字段的名称


- f.getDeclaringClass()	返回表示类或接口的 Class 对象

**反射获取类的方法（包括静态）**

- 获取指定的public方法：clz.getMethod(String methodName， Class<?> ... parameterTypes);获取指定name名称、指定参数列表类型的具有public修饰的方法，包含继承方法，返回Method对象。


- 获取全部的public方法：clz.getMethods();获取修饰符为public的方法，包含继承方法，返回Method[]对象数组


- 获取指定方法（包括私有的）：clz.getDeclaredMethod(String fieldName， Class<?> ... parameterTypes);获取指定name名称，指定参数列表类型的(包含private修饰的)方法，不包括继承的方法，返回Method对象


- 获取全部方法（包括私有的）：clz.getDeclaredMethods();获取Class对象所表示的类或接口的所有(包含private修饰的)方法,不包括继承的字段，返回Method[]对象数组。


**Method类的一些方法**：Method类的一个对象代表一个方法。**注意：如果是private成员，那么必须设置method.setAccessible(true);设置为可访问。 **

- m.invoke(Object obj, Object... args)	对带有指定参数的指定对象，实现对这个方法的动态调用


- m.getReturnType()	返回一个 Class 对象，该对象描述了此 Method 对象所表示的方法的正式返回类型,即方法的返回类型


- m.getParameterTypes()	按照声明顺序返回参数列表的 Class 对象的数组


- m.getName()	以 String 形式返回此 Method 对象表示的方法名称，即返回方法的名称


- isVarArgs()	判断方法是否带可变参数，如果将此方法声明为带有可变数量的参数，则返回 true；否则，返回 false。


**反射动态操作数组**

- clz.getComponentType()	返回表示数组元素类型的 Class，即数组的类型，如int.class
- clz.isArray()	判定此 Class 对象是否表示一个数组类。

**java.lang.reflect.Array提供的动态创建、操作数组的方法**

- Array.set(Object array, int index)	静态方法，设定指定数组对象在指定索引处的值。


- Array.getLength(Object array)	静态方法，以 int 形式返回指定数组对象的长度


- Array.newInstance(Class<?> componentType, int... dimensions)	静态方法，创建一个具有指定类型和维度的新数组。


- Array.newInstance(Class<?> componentType, int length)	静态方法，创建一个具有指定的类型和长度的一维数组。


- Array.set(Object array, int index, Object value)	静态方法，给指定数组对象在指定索引处设置为指定的新值。
	
	
		public class ArrayReflectTest{
			public static void main(String[] args){
				int[] array = {1,2,3,4,5,6,7,8};
				//获取这个数组的类型的Class对象
				System.out.println(array.getClass().getComponentType());  // int
		
				//使用Array动态创建一个新数组
				Object newArr = java.lang.reflect.Array.newInstance(array.getClass().getComponentType(), 15);
				//Array获取动态数组长度
				int len = Array.getLength(array);
				System.arraycopy(array, 0, newArr, 0, len);
		
				for(int i : (int[])newArr){
					System.out.print(i+"  ");
				} // 1  2  3  4  5  6  7  8  0  0  0  0  0  0  0  
				System.out.println();
			}
		}
利用Array动态创建数组的特性，动态创建一个泛型数组：

		public <T extends Comparable<T>> void min(T[] a) {
			//构建泛型数组
			T[] b = (T[]) Array.newInstance(a.getClass().getComponentType(), a.length);
			for (int i = 0; i < a.length; i++) {
				b[i] = a[i];
			}
			T min = b[0];
			for (int i = 0; i < b.length; i++) {
				if(min.compareTo(b[i]) > 0) {
					min = b[i];
				}
			}
			System.out.println(min);
		}