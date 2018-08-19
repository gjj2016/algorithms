# 类加载原理 #

## Java类加载的五个步骤 ##

加载：根据类的全称找到字节码文件，并根据字节码文件创建Class对象

验证：安全验证

准备：为类变量分配内存并且初始化为初始值（int的初始值为0，Boolean的初始值为false等），注意为类变量赋值在后面的初始化阶段，成员变量随着对象一起在堆上分配内存，而final变量在编译期间就已经分配内存。

解析：符号引用替换为直接引用

初始化：如果该类有父类，那么先初始化父类，然后为类变量赋值。

**类加载器**的任务就是在类的加载阶段根据类名找到字节码文件并创建Class对象。

Java中类加载器分为三种：启动（Bootstrap）类加载器、扩展（Extension）类加载器、系统（System）类加载器（也叫应用类加载器）

- 启动类加载器：是最顶层的类加载器，加载JVM运行所必需的类(<JAVA_HOME/lib>)，主要包括包名以java,javax,sun开头的类（rt.jar）

- 扩展类加载器：指的是Launcher类的静态内部类ExtClassLoader加载扩展目录（<JAVA_HOME/lib/ext>）下的类

- 系统类加载器：指的是Launcher的静态内部类AppClassLoader，加载classpath下的类

## 类加载的双亲委派模式 ##

- 双亲委派模式的概念：当需要加载一个类时，最底层的自定义类加载器向上委托给系统类加载器、系统类加载器再向上委托给扩展类加载器、扩展类加载器再向上委托给启动类加载器，如果最顶层的类加载器能够加载这个类，那么由上层的类加载器直接加载，如果上层类加载器无法加载这个类，那么才由这个类加载器去加载。

- 双亲委派模式的优点：安全性、防止类重复加载。

- 双亲委派模式的实现方式：


java.lang包下有如下类继承关系：

抽象类ClassLoader---->SecureClassLoader---->URLClassLoader---->ExtClassLoader,AppClassLoader

同时ExtClassLoader,AppClassLoader也是Launcher类的静态内部类。

	protected Class<?> loadClass(String name, boolean resolve)
		  throws ClassNotFoundException
	  {
		  synchronized (getClassLoadingLock(name)) {
			  // 先从缓存查找该class对象，找到就不用重新加载
			  Class<?> c = findLoadedClass(name);
			  if (c == null) {
				  long t0 = System.nanoTime();
				  try {
					  if (parent != null) {
						  //如果找不到，则委托给父类加载器去加载
						  c = parent.loadClass(name, false);
					  } else {
					  //如果没有父类，则委托给启动加载器去加载
						  c = findBootstrapClassOrNull(name);
					  }
				  } catch (ClassNotFoundException e) {
					  // ClassNotFoundException thrown if class not found
					  // from the non-null parent class loader
				  }

				  if (c == null) {
					  // If still not found, then invoke findClass in order
					  // 如果都没有找到，则通过自定义实现的findClass去查找并加载
					  c = findClass(name);

					  // this is the defining class loader; record the stats
					  sun.misc.PerfCounter.getParentDelegationTime().addTime(t1 - t0);
					  sun.misc.PerfCounter.getFindClassTime().addElapsedTimeFrom(t1);
					  sun.misc.PerfCounter.getFindClasses().increment();
				  }
			  }
			  if (resolve) {//是否需要在加载时进行解析
				  resolveClass(c);
			  }
			  return c;
		  }
	  }
ClassLoader类中的loadClass()方法是双亲委派模式的实现者，loadClass()方法先从缓存中查找是否已经加载过这个类，然后看是否有上层类加载器，如果有则委托给上层类加载器，如果没有则交给启动类加载器，如果仍然加载失败，会调用findClass()方法完成类的加载。

ClassLoader中并没有实现findClass()方法。

如果我们在定义类加载器时选择继承ClassLoader类而非URLClassLoader，必须手动编写findclass()方法的加载逻辑以及获取字节码流的逻辑。

如果我们在定义类加载器时选择继承URLClassLoader，URLClassLoader中存在一个URLClassPath类，通过这个类就可以找到要加载的字节码流，也就是说URLClassPath类负责找到要加载的字节码，再读取成字节流，最后通过defineClass()方法创建类的Class对象。

类加载器间的关系

- 启动类加载器，由C++实现，没有父类。
- 拓展类加载器(ExtClassLoader)，由Java语言实现，父类加载器为null
- 系统类加载器(AppClassLoader)，由Java语言实现，父类加载器为ExtClassLoader
- 自定义类加载器，父类加载器肯定为AppClassLoader。

**在JVM中表示两个class对象是否为同一个类对象存在两个必要条件**

- 类的完整类名必须一致，包括包名。
- 加载这个类的ClassLoader(指ClassLoader实例对象)必须相同。


想要由同一个字节码文件加载的Class对象不同，有有两种方式：

- 重写loadClass()方法，并且去掉从缓存中查找的那行代码（因为类名相同的会直接从缓存中找到）
- 直接调用findClass()方法。


		//创建两个不同的自定义类加载器实例
		FileClassLoader loader1 = new FileClassLoader(rootDir);
		FileClassLoader loader2 = new FileClassLoader(rootDir);
		//通过findClass创建类的Class对象
		Class<?> object1=loader1.findClass("com.zejian.classloader.DemoObj");
		Class<?> object2=loader2.findClass("com.zejian.classloader.DemoObj");
	
		System.out.println("findClass->obj1:"+object1.hashCode()); //findClass->obj1:723074861
		System.out.println("findClass->obj2:"+object2.hashCode()); //findClass->obj2:895328852

## 自定义类加载器 ##
实现自定义类加载器需要继承ClassLoader或者URLClassLoader，继承ClassLoader则需要自己重写findClass()方法并编写加载逻辑，
继承URLClassLoader则可以省去编写findClass()方法以及class文件加载转换成字节码流的代码。

**编写自定义类加载器的意义：**

- 当class文件不在ClassPath路径下，默认系统类加载器无法找到该class文件，在这种情况下我们需要实现一个自定义的ClassLoader来加载特定路径下的class文件生成class对象。
- 当一个class文件是通过网络传输并且可能会进行相应的加密操作时，需要先对class文件进行相应的解密后再加载到JVM内存中，这种情况下也需要编写自定义的ClassLoader并实现相应的逻辑。
- 当需要实现热部署功能时(一个class文件通过不同的类加载器产生不同class对象从而实现热部署功能)，需要实现自定义ClassLoader的逻辑。


		public class FileClassLoader extends ClassLoader {
			private String rootDir;
	
			public FileClassLoader(String rootDir) {
				this.rootDir = rootDir;
			}
	
			/**
			 * 编写findClass方法的逻辑
			 * @param name
			 * @return
			 * @throws ClassNotFoundException
			 */
			@Override
			protected Class<?> findClass(String name) throws ClassNotFoundException {
				// 获取类的class文件字节数组
				byte[] classData = getClassData(name);
				if (classData == null) {
					throw new ClassNotFoundException();
				} else {
					//直接生成class对象
					return defineClass(name, classData, 0, classData.length);
				}
			}
	
			/**
			 * 编写获取class文件并转换为字节码流的逻辑
			 * @param className
			 * @return
			 */
			private byte[] getClassData(String className) {
				// 读取类文件的字节
				String path = classNameToPath(className);
				try {
					InputStream ins = new FileInputStream(path);
					ByteArrayOutputStream baos = new ByteArrayOutputStream();
					int bufferSize = 4096;
					byte[] buffer = new byte[bufferSize];
					int bytesNumRead = 0;
					// 读取类文件的字节码
					while ((bytesNumRead = ins.read(buffer)) != -1) {
						baos.write(buffer, 0, bytesNumRead);
					}
					return baos.toByteArray();
				} catch (IOException e) {
					e.printStackTrace();
				}
				return null;
			}
	
			/**
			 * 类文件的完全路径
			 * @param className
			 * @return
			 */
			private String classNameToPath(String className) {
				return rootDir + File.separatorChar
						+ className.replace('.', File.separatorChar) + ".class";
			}
	
			public static void main(String[] args) throws ClassNotFoundException {
				String rootDir="/Users/zejian/Downloads/Java8_Action/src/main/java/";
				//创建自定义文件类加载器
				FileClassLoader loader = new FileClassLoader(rootDir);
	
				try {
					//加载指定的class文件
					Class<?> object1=loader.loadClass("com.zejian.classloader.DemoObj");
					System.out.println(object1.newInstance().toString());
	
					//输出结果:I am DemoObj
				} catch (Exception e) {
					e.printStackTrace();
				}
			}
		}
自定义类加载器继承ClassLoader需要重写findClass()方法，在findClass()方法中实现将字节码文件转换为字节数组，然后通过defineClass()方法创建Class对象。将字节码文件不放在classpath下，启动类加载器、扩展类加载器及系统类加载器都无法完成加载，因此只能由自定义类加载器加载这个类。自定义类加载器调用loadClass()方法，这是父类ClassLoader的方法，执行到findClass()方法时会调用子类中重写的findClass()方法。

		public class FileUrlClassLoader extends URLClassLoader {
	
			public FileUrlClassLoader(URL[] urls, ClassLoader parent) {
				super(urls, parent);
			}
	
			public FileUrlClassLoader(URL[] urls) {
				super(urls);
			}
	
			public FileUrlClassLoader(URL[] urls, ClassLoader parent, URLStreamHandlerFactory factory) {
				super(urls, parent, factory);
			}
	
	
			public static void main(String[] args) throws ClassNotFoundException, MalformedURLException {
				String rootDir="/Users/zejian/Downloads/Java8_Action/src/main/java/";
				//创建自定义文件类加载器
				File file = new File(rootDir);
				//File to URI
				URI uri=file.toURI();
				URL[] urls={uri.toURL()};
	
				FileUrlClassLoader loader = new FileUrlClassLoader(urls);
	
				try {
					//加载指定的class文件
					Class<?> object1=loader.loadClass("com.zejian.classloader.DemoObj");
					System.out.println(object1.newInstance().toString());
	
					//输出结果:I am DemoObj
				} catch (Exception e) {
					e.printStackTrace();
				}
			}
		}
继承URLClassLoader则可以省去编写findClass，但是需要将字节码文件的路径转换为URL数组，因为URLClassLoader的所有构造器都必须传入一个URL数组，然后调用loadClass()方法，这个方法是ClassLoader类的，URLClassLoader没有重写，然后loadClass()方法会调用URLClassLoader类中实现的findClass()方法和defineClass()方法。

**线程上下文类加载器**

实际上核心包的SPI类对外部实现类的加载都是基于线程上下文类加载器执行的，如JDBC的接口包在rt.jar中，核心SPI类由启动类加载器加载，
当需要加载具体的实现类的时候，由于实现类（如MySQL）的包放在classpath下，启动类加载器无法向下委托系统类加载器去加载具体的实现类，
因此启动类加载器会委托给线程上下文类加载器去加载具体的实现类。

默认的线程上下文类加载器是系统类加载器

	public Launcher() {
			// 首先创建拓展类加载器
			ClassLoader extcl;
			try {
				extcl = ExtClassLoader.getExtClassLoader();
			} catch (IOException e) {
				throw new InternalError(
					"Could not create extension class loader");
			}

			// Now create the class loader to use to launch the application
			try {
				//再创建AppClassLoader并把extcl作为父加载器传递给AppClassLoader
				loader = AppClassLoader.getAppClassLoader(extcl);
			} catch (IOException e) {
				throw new InternalError(
					"Could not create application class loader");
			}

			//设置线程上下文类加载器，稍后分析
			Thread.currentThread().setContextClassLoader(loader);
	//省略其他没必要的代码......
			}
	}
在Launcher类中先创建ExtClassLoader，然后ExtClassLoader作为父加载器传给AppClassLoader，
最后会降线程上下文类加载器设置为AppClassLoader。不将线程上下文类加载器直接设置为AppClassLoader是因为不同的服务使用的线程上下文类加载器是不同的，只是JDBC服务使用的恰好是AppClassLoader.
