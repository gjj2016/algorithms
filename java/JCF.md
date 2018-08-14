# Java Collection Framework #
## 1、泛型 ##
Java泛型不需要Java虚拟机的支持，只是编译阶段的“语法糖”，变量的类型之所以可以泛化，是因为所有类型都是继承于Object，即Java的单继承保证了泛型。泛型的存在简化了编程，但是在使用时需要显示的强制类型转换。当我们规定了容器的类型之后，只允许放入该类型的变量，在使用时不需要显示的强制类型转换，由编译器完成强制类型转换。
## 2、Java的内存管理 ##
Java GC自动完成了Java程序的内存管理，所有的Java对象均存放在堆上，而且对象只能通过引用访问。容器里存放的其实是对象的引用而不是对象本身。

![](https://cdn.get233.com/hran/2017/01/24/148526650653957_JCF_Collection_Interfaces.png?imageView2/2/w/1920/q/75/format/webp)

## ArrayList ##
ArrayList实现了List接口，ArrayList中可以放入null元素。底层通过数组实现，如果在声明一个ArrayList时没有指定数组大小，那么默认的初始容量为10.

需要注意ArrayList中的capacity、size、elementData。其中capacity是底层数组的容量大小，size是数组中元素的个数，elementData是底层数组。





## LinkedList ##
LinkedList底层通过双向链表实现。拥有一个头指针first指向第一个元素，用于一个尾指针指last向最后一个元素。当链表为空时，first==last==null。在LinkedList有一个内部类Node，Node类的定义如下：


	class Node{
		Node prev;
		Node next;
		E element;
	}

LinkedList在实际使用上可以除了可以作为双向链表使用外，还可以作为栈和队列使用，因为LinkedList实现了Deque接口，拥有这个接口的addFirst(), addLast(), peek()等操作栈和队列的方法。

LinkedList虽然底层通过双向链表实现，但是也记录了size大小，index元素下标。如它的add方法：

	public void add(int index, E data)｛
		//先通过下标找到要插入的位置
		if(index<(size>>1)){   //如果下标在前半段，从前往后找
			
		}else{                //如果下标在后半段，从后往前找
			
		}
		
	｝
LinkedList中也可以放入null元素，如它的remove(Object o)方法：

	public E remove(Object o){
		if(o == null){
			//从前往后判断 x.element == o
			//找到对应的x节点之后，将这个节点删除（如果是第一个节点，需要修改first指针，同样如果是最后一个节点，需要修改last指针）
		}else{
			//从前往后判断x.element.equals(o)
			//找到对应的x节点之后，将这个节点删除（如果是第一个节点，需要修改first指针，同样如果是最后一个节点，需要修改last指针）
		}
	}
LinkedList也有get方法和set方法，均需要根据下标转换为对应节点。

## ArrayDeque ##
ArrayDeque类是最常用的栈和队列的操作类，该类实现了Deque接口，是一个双端口队列。底层通过一个循环数组实现，即除了有一个数组之外，还有一个head指针始终指向第一个元素，还有一个tail指针始终指向下一个可以插入的位置。

ArrayDeque中不允许放入null元素，而且底层数组的长度一直为2的指数倍，最小长度为8，每次扩容的时候会增加一倍，如addFirst（）方法：

	public void addFirst(E e){
		if (e == null){                       //放入null的时候直接抛出空指针异常
			throw new NullPointerException();
		}
		//先插入这个元素，再去考虑扩容的事，因为tail始终指向下一个可以插入的元素，表示一定可以插入一个元素。
		element[head=(head-1) & (element.length-1)] = e;
		if(head == tail){                    //表示数组满了，扩容为原长度的两倍
			int p = head;
			int n = element.length;
			int r = n-p;                    //head右边的元素各数
			int newCapacity = n<<1;
			if(newCapacity < 0){
				//放不下了，抛异常
			}
			Object[] a = new Obejct[newCapacity];
			//先把head右边的所有元素移到新数组的最前端
			System.arraycopy(element, head, a, 0, r);
			//再把head左边的所有元素移动新数组中
			System.arraycopy(element, 0, a, r, p);
			head = 0;
			tail = n;
			element = (E[])a;
		}
	}
ArrayDeque处理下标越界的方法很优秀，处理head为-1的方法：
>head = (head -1) & (element.length-1)

处理tail为element.length的方法为：
>tail = （tail+1）& (element.elngth - 1)

## TreeMap ##
TreeMap实现了SortedMap，可以理解为有顺序的Map，其顺序体现在会按照key对Map中的元素排序，key的大小可以通过其自身的比较大小的方法或者在构造时传入比较器来比较key的大小。

TreeMap底层通过红黑树实现，这意味着TreeMap的containsKey()、get()、put()、remove()方法都有着log(n)的时间复杂度。具体地，红黑树是有着如下约素的二叉查找树：

1. 每个节点要么是红色，要么是黑色
2. 根节点必须是黑色
3. 红色节点不能连续（即红色节点的父节点和子节点必须是黑色）
4. 从根节点到任一叶子节点路径上黑色节点数量必须一样

当插入或者删除元素时，红黑树的结构会发生变化，这时可以通过改变节点颜色或者调整结构使其满足红黑树的约素条件，调整结构的方法有左旋和右旋：

**左旋：**以x为中心的左旋意味着将x的右子树围绕x逆时针旋转，即将x的右子节点变为x的父节点，右子节点的右子节点不变，右子节点的左子节点变为x的右子节点。

	private void rotate_left(Entry<K,V> x){
		if(x != null){
			Entry<K,V> r = x.right;
			x.right = r.left;
			if(r.left != null){
				r.left.parent = x;
			}
			r.parent = x.parent;
			if(x.parent == null){
				root = r;
			}else if(x.parent.left == x){
				x.parent.left = r;
			} else{
				x.parent.right = r;
			}
			r.left = x;
			x.parent = r;
		}
	}
**右旋：**以x为中心的右旋意味着x的左子树围绕x顺时针旋转，即x的左子节点变为x的父节点，x的左子节点的左子节点不变，x的左子节点的右子节点变为x的左子节点。

	private void rotate_right(Entry<K,V> x){
		if(x != null){
			Entry<K,V> r = x.left;
			x.left = r.right;
			if(r.right != null){
				r.right.parent = x;
			}
			r.parent = x.parent;
			if(x.parent == null){
				root = r;
			}else if(x.parent.left == x){
				x.parent.left = r;
			} else{
				x.parent.right = r;
			}
			r.right = x;
			x.parent = r;
		}
	}
寻找**后继结点**（比当前节点大的节点中最小的那个）

	private Entry<K,V> successor(Entry<K,V> e){
		if(e == null){
			return null;
		}
		//如果右子树不为空，那么后继结点就是右子树中最小的那个节点
		if(e.right != null){
			Entry<K,V> p = e.right;
			while(p.left != null){
				p = p.left;
			}
			return p;
		} else {
			//如果右子树为空，那么后继结点是第一个往右走的父节点，因为往左走的父节点都小于它。
			Entry<K,V> r = e.parent;
			Entry<K,V> p = e;
			while(r!=null && r.right == p){
				p = r;
				r = r.parent;
			}
			return r;
		}
	}
TreeMap中不允许key为null，如get(Object key)方法，根据key返回对应的value

	public V get(Object key){
		if(key == null){
			//不允许key为null，否则抛出空指针异常
		}
		//先通过key找到对应的Entry
		Entry<K,V> p = root;
		while(p != null){
			if(key.compareTo(p.key) < 0){//往左子树找
				//根据key的本身顺序或者比较器定义的顺序比较key值。
				p = p.left;
			} else {                    //往右子树找
				p = p.right;
			} else {
				return p;
			}
		}
		//没有这个key
		return null;
	}
往TreeMap中插入元素的时候，调用put(K key, V value)方法，如果该key已经存在，那么设置新值为value并返回旧值，如果该key不存在，那么找到合适的位置插入key，然后调整红黑树使其满足约素条件。

	public V put（K key, V value）{
		Entry<K,V> t = root;
		if(t == null){
			//红黑树为空，插入该节点，并设置为root
		}else{
			if(comparator == null){
				//构造时没有传入比较器，那么将使用key的自身比较大小
				//key.compareTo(p)
			} else{
				//传入了比较器，根据比较器比较key的大小
				//comparator.compare(key, p)
			}
			//找到合适的插入位置后，根据最后一次比较结果选择将新节点插入到左子树还是右子树
			//调整红黑树使其满足约素条件
	 	}
		
	}
从TreeMap中移除节点的时候，调用remove(K key)方法，该方法同样先在红黑树中找到该key对应的Entry，然后删除该Entry。删除的时候分两种情况：1）该Entry左右字数都为空或者左子树和右子树有一个为空，那么可以直接删除这个Entry或者直接使用左子树或右子树替代这个Entry；2)该Entry的左右字数都不为空，这时可以先用该Entry的后继结点代替这个Entry，因为该节点的后继结点的右子树肯定为空，那么满足了情况1.
注意：无论是哪种情况，删除了Entry之后，都需要重新调整红黑树使其满足约素条件。

	public V remove(K key){
		//首先找到该Entry
		Entry<K,V> x = getEntry(key);
		if(x == null){
			return null;
		}
		if(x.left!=null && x.right!=null){
			//左右字数都不为空，与后继结点交换
			Entry<K,V> s = successor(x);
			x.key = s.key;
			x.value = s.value;
			x = s;
		}
		Entry<K,V> replacement = (x.left==null?x.left:x.right);
		if(replavement != null){
			//有一个子树不为空
			replacement.parent = x.parent;
			if(x.parent.left = x){
				x.parent.left = replacement;
			} else if(x.parent.right == x){
				x.parent.right = replacement;
			}
			x.parent = x.left = x.right = null;
		} else if(x.parent == null){
			root = null;
		} else{
			//两个子树都为空
			//释放空指针
		}
	}
## TreeSet ##
TreeSet基于TreeMap实现，在TreeSet中直接定义了一个TreeMap，相当于一个只使用key的TreeMap.

这种设计模式成为**适配器模式。**

## HashMap ##
与TreeMap有序存放Entry不同，HashMap中的Entry是无序的。HashMap底层依靠一个数组和冲突链表实现，HashMap中两个重要的方法是hashCode()方法和equals()方法，其中hashCode方法决定了key对应的hash值，而hash值通过*(hash)&(table.length-1)*可以得到在数组中的下标，由于HashMap中数组的大小必须是2的指数倍，因此相当于对数组长度取模。而equals方法可以判断同一个冲突链表中的Entry是否相等。**注意：底层数组中是没有存放Entry的，只是指向存放在这个下标对应的冲突链表。**HashMap中可以放入key为null元素，以及value为null的元素。key为null的hash值为0，对应的数组下标也是0，可见最多只能放入一个key为null的Entry。

根据key获得对应的value

	public V get(Object key){
		//获得下标	
		int hash = (key==null)?0:hash(key);
		int index = hash&(table.length-1);
		//获得冲突链表
		Entry<K,V> e = table[index];
		for(; e!=null; e=e.next){
			Object k;
			if((k=e.key)==key && k.equals(key)){
				return e;
			}
		}
		return null;
	}

往HashMap中放入元素

	public V put(Object key, V value){
		//如果key已经存在，那么设置新value值，返回旧value
		if(size>threshold && table[index]!=null){
			//扩容并且重新hash
			resize(2*table.length);
			hash = (key==null)? 0 : hash(key);
			index = hash&(table.length-1);
		}
		Entry<K,V> e = table[index];
		//头插入法
		table[index] = new Entry<K,V>(hash, key, value, e);
		size++;
	}

## HashSet ##
HashSet依靠HashMap实现，在HashSet中直接定义了一个HashMap。这也是**适配器模型。**

## LinkedHashMap ##
LinkedHashMap在底层实现上与HashMap相似，唯一的不同点在于：LinkedHashMap使用了一个header哑元，将所有的Entry连接起来，连接方式是双向链表。这也决定了LinkedHashMap在迭代的时候不需要遍历整个数组，而只需要遍历header连接的双向链表。

根据key从LinkedHashMap中取出对应的value，与HashMap中的一样，先获取冲突链表，再遍历冲突链表。

往LinkedHashMap中put元素的时候，除了需要把Entry采用头插入法插入到冲突链表中，还需要把Entry插入到header指向的双向链表中。插入到冲突链表中的方式与HashMap中一样，插入到双向链表中只需要将Entry插入到header节点的前一个节点即可。

从LinkedHashMap中remove元素的时候，跟插入时候也是一样，需要从冲突链表中删除Entry，还需要从双向链表中删除Entry。

## LinkedHashSet ##
LinkedHashSet的实现基于LinkedHashMap，在LinkedHashSet中直接定义了一个LinkedHashMap。这也是**适配器模式。**

## PriorityQueue ##
PriorityQueue是Java中的优先队列，每次从队列中取出的元素一定是最小的。PriorityQueue依靠小顶堆实现，而小顶堆又是通过数组实现的，因此，PriorityQueue底层是依靠数组实现的小顶堆（每个节点的左右子节点都比自身大的完全二叉树）。因此堆顶元素是最小的，而且
>leftchildNO = parentNO*2 + 1

>rightchildNO = parentNO*2 + 2

>parentNO = (childNO-1)/2

往PriorityQueue新加元素的时候，**注意：PriorityQueue中不能放入null元素。**先将元素插入到数组最后面，即加入到完全二叉树的某个叶子节点。插入之后可能破坏了小顶堆的特性，因此需要调整，调整方法为自底向上的与父节点比较，直到比父节点大。

	private void siftUp(int k, E x){
		while(k>0){
			int parentNO = (k-1)>>>2;
			//既可以使用元素的自然大小顺序，也可以使用构造时传入的比较器
			if(comparator.compare(x, queue[parentNO]) >= 0){
				break;
			}
			queue[k] = queue[parentNO];
			k = parentNO;
			
		}
		queue[k] = x;
	}
PriorityQueue中element和peek等方法能在常数时间内完成，只需要获取首元素即可。

PriorityQueue中remove或者poll等方法需要获取并删除首元素，删除了首元素之后，就破坏了小顶堆的特性，因此先获取堆顶元素用于返回，然后用数组的最后一个元素覆盖首元素，将最后一个元素置为null。此时小顶堆被破坏，我们采用自顶向下的方式调整小顶堆。

	private void siftDown(int k, E x){
		//	
		int half = size >>> 1;
		while(k < half){
			int left = 2*k + 1;
			Object o = queue[left];
			int right = left +1;
			if(right<size && queue[right]<o){
				left = right;
				o = queue[right];
			}
			if(comparator.compare(x, o) >= 0){
				break;
			}
			queue[k] = o;
			k = left;
		}
		queue[k] = x;
	}


## WeakHashMap ##
WeakHashMap结构上与HashMap几乎一致，唯一的区别在于WeakHashMap中所有的Entry都是弱引用，弱引用就是该引用可能会被GC自动回收。WeakHashMap的这种特性特别适合于***缓存场景***，WeakHashMap中的Entry要想不被GC自动回收，那么在WeakHashMap外必须还有强引用指向这个Entry。

Java中是没有WeakHashSet这个类的，但是我们可以通过Collections这个工具类的newSetFromMap(new WeakHashMap<>())来构造一个WeakHashSet。