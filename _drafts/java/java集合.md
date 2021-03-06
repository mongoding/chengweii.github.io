java集合
===
## 集合与数组的区别

数组：（可以存储基本数据类型）是用来存现对象的一种容器，但是数组的长度固定，不适合在对象数量未知的情况下使用
集合：  (只能存储对象，对象类型可以不一样）集合可以靠数组实现，也可以靠链表实现，也可以是数组与链表的混合实现，或者是树的实现；长度可变，并提供了丰富的特效，如：去重，排序.

## 集合的层次关系

如图所示：图中，实线边框的是实现类，折线边框的是抽象类，而点线边框的是接口
![](http://topweshare.qiniudn.com/1524530299.png?imageMogr2/thumbnail/!70p)
Collection接口是集合类的根接口，Java中没有提供这个接口的直接的实现类。但是却让其被继承产生了两个接口，就是Set和List。Set中不能包含重复的元素。List是一个有序的集合，可以包含重复的元素，提供了按索引访问的方式。

Map是Java.util包中的另一个接口，它和Collection接口没有关系，是相互独立的，但是都属于集合类的一部分。Map包含了key-value对。Map不能包含重复的key，但是可以包含相同的value。

Iterator，所有的集合类，都实现了Iterator接口，这是一个用于遍历集合中元素的接口，主要包含以下三种方法：
1.hasNext()是否还有下一个元素。
2.next()返回下一个元素。
3.remove()删除当前元素。

## 链表

### ArrayList
ArrayList实现了List接口，是顺序容器，即元素存放的数据与放进去的顺序相同，允许放入null元素，底层通过对象数组实现。  
ArrayList还实现了Serializable、Cloneable接口，支持序列化和克隆。  
ArrayList除了未实现同步外，其余实现与Vector大致相同。  

#### 实现原理
ArrayList内部封装了一个Object类型的数组，从一般的意义来说，它和数组没有本质的差别，甚至于ArrayList的许多方法，如index、indexOf、contains、sort等都是在内部数组的基础上直接调用Array的对应方法。  
每个ArrayList都有一个容量（capacity），表示底层数组的实际大小，容器内存储元素的个数不能多于当前容量。当向容器中添加元素时，如果容量不足，容器会自动增大底层数组的大小。  
size(), isEmpty(), get(), set()方法均能在常数时间内完成，add()方法的时间开销跟插入位置有关，addAll()方法的时间开销跟添加元素的个数成正比。其余方法大都是线性时间。  
为追求效率，ArrayList没有实现同步（synchronized），如果需要多个线程并发访问，用户可以手动同步，也可使用Vector替代。 
> <b>扩容机制:</b> ArrayList自动扩容的方式是通过创建容量为旧数组容量1.5倍的新数组，然后通过Arrays.copyOf()方法将旧数组的值拷贝过去(注：此操作为浅拷贝)。  
说到Arrays.copyOf()方法，此方法内部通过System.arraycopy()方法实现，而System.arraycopy()方法是使用的内存复制，在数组较大时，效率要比手工循环寻址复制要好。

> <b>手动序列化:</b> ArrayList虽然实现了Serializable接口，但它通过定义的 readObject（ObjectInputStream in）和 writeObject(ObjectOutputStream out)方法手动进行序列化和反序列化，如此设计的原因是由于动态扩容的机制，elementData数组中会存在空位置，为了避免序列化时造成空间浪费，采取了手动序列化操作。

> <b>Fail-Fast机制:</b>由于ArrayList不是线程安全的，因此如果在使用迭代器的过程中有其他线程修改了ArrayList，那么将抛出ConcurrentModificationException，这就是所谓fail-fast策略。  
这一策略在源码中的实现是通过modCount域，modCount顾名思义就是修改次数，对ArrayList内容的修改都将增加这个值，那么在迭代器初始化过程中会将这个值赋给迭代器的expectedModCount。在迭代过程中，判断modCount跟expectedModCount是否相等，如果不相等就表示已经有其他线程修改了ArrayList。(注意：在java8之前modCount声明为volatile，保证线程之间修改的可见性，但java8开始,modCount去除了volatile修饰符，毕竟这个类是声明为非线程安全的。volatile变量的读写需要一些内存关卡，开销变大，还没什么卵用。)

```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{
    //...
    transient Object[] elementData; // non-private to simplify nested class access
    public boolean isEmpty() {
        return size == 0;
    }
    public E get(int index) {
        rangeCheck(index);
        return elementData(index);
    }
    public boolean add(E e) {
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        elementData[size++] = e;
        return true;
    }
    private void ensureCapacityInternal(int minCapacity) {
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
            minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
        }

        ensureExplicitCapacity(minCapacity);
    }
    private void ensureExplicitCapacity(int minCapacity) {
        modCount++;
        // overflow-conscious code
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
    }
    private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;
    private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
    private static int hugeCapacity(int minCapacity) {
        if (minCapacity < 0) // overflow
            throw new OutOfMemoryError();
        return (minCapacity > MAX_ARRAY_SIZE) ?
            Integer.MAX_VALUE :
            MAX_ARRAY_SIZE;
    }
    private void writeObject(java.io.ObjectOutputStream s)
        throws java.io.IOException{
        // Write out element count, and any hidden stuff
        int expectedModCount = modCount;
        s.defaultWriteObject();
        // Write out size as capacity for behavioural compatibility with clone()
        s.writeInt(size);
        // Write out all elements in the proper order.
        for (int i=0; i<size; i++) {
            s.writeObject(elementData[i]);
        }
        if (modCount != expectedModCount) {
            throw new ConcurrentModificationException();
        }
    }
    private void readObject(java.io.ObjectInputStream s)
        throws java.io.IOException, ClassNotFoundException {
        elementData = EMPTY_ELEMENTDATA;
        // Read in size, and any hidden stuff
        s.defaultReadObject();
        // Read in capacity
        s.readInt(); // ignored
        if (size > 0) {
            // be like clone(), allocate array based upon size not capacity
            ensureCapacityInternal(size);
            Object[] a = elementData;
            // Read in all elements in the proper order.
            for (int i=0; i<size; i++) {
                a[i] = s.readObject();
            }
        }
    }
    //...
}
```

#### 使用注意
* 基于数组快速随机访问的特性，ArrayList非常适合于高频读取的场景；但由于数组容量不足而自动扩容的机制导致ArrayList的写能力会受数据容量不确定性的影响。
* 由于数组容量不足而自动扩容的机制会造成性能损耗，在实际添加大量元素前，可以提前预估并指定ArrayList实例的容量，或者通过调用ensureCapacity()方法来手动增加ArrayList实例的容量，以减少递增式再扩容导致的性能损耗。
* ArrayList未实现同步，是非线程安全的，所以在多线程场景下建议使用CopyOnWriteArrayList。
* ArrayList.Iterator.remove() 能在循环中删除数据，每次删除完数据操作后，重新把目标集合的modCount 赋给了迭代器的expectedModCount
* 对于基于数组实现的集合，删除数据，插入数据会有数据移位操作，影响性能。

#### ArrayList.SubList
1，该方法返回的是父list的一个视图，从fromIndex（包含），到toIndex（不包含）。fromIndex=toIndex 表示子list为空

2，父子list做的非结构性修改（non-structural changes）都会影响到彼此：所谓的“非结构性修改”，是指不涉及到list的大小改变的修改。相反，结构性修改，指改变了list大小的修改。

3，对于结构性修改，子list的所有操作都会反映到父list上。但父list的修改将会导致返回的子list失效。

4，tips：如何删除list中的某段数据：

list.subList(from, to).clear();

### Vector
Vector内部与ArrayList大致相同，但该类实现了同步，是ArrayList的线程安全版本实现。

#### 实现原理
Vector内部与ArrayList大致相同，但该类实现了同步，在大部分操作方法上使用了synchronized关键字，导致其在多线程场景下效率极其低下，基本已被弃用。

```java
public class Vector<E>
    extends AbstractList<E>
    implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{
    //...
    protected Object[] elementData;
    public synchronized boolean isEmpty() {
        return elementCount == 0;
    }
    public synchronized int lastIndexOf(Object o) {
        return lastIndexOf(o, elementCount-1);
    }
    public synchronized E get(int index) {
        if (index >= elementCount)
            throw new ArrayIndexOutOfBoundsException(index);

        return elementData(index);
    }
    public synchronized boolean add(E e) {
        modCount++;
        ensureCapacityHelper(elementCount + 1);
        elementData[elementCount++] = e;
        return true;
    }
    private void ensureCapacityHelper(int minCapacity) {
        // overflow-conscious code
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
    }
    private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;
    private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        int newCapacity = oldCapacity + ((capacityIncrement > 0) ?
                                         capacityIncrement : oldCapacity);
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
    private static int hugeCapacity(int minCapacity) {
        if (minCapacity < 0) // overflow
            throw new OutOfMemoryError();
        return (minCapacity > MAX_ARRAY_SIZE) ?
            Integer.MAX_VALUE :
            MAX_ARRAY_SIZE;
    }
    //...
}
```

#### 使用注意
* 尽管Vector类实现了线程安全，但对于该类执行多个操作时，为了保证原子性，还是要添加同步代码的，否则还是会存在线程安全问题。

```java
private static Vector<Integer> vector=new Vector();

public static void main(String[] args) {
    while(true){
        for(int i=0;i<10;i++){
            vector.add(i);
        }
        Thread removeThread=new Thread(new Runnable() {         
            @Override
            public void run() {
                for(int i=0;i<vector.size();i++){
                    Thread.yield();
                    vector.remove(i);
                }
            }
        });
        Thread printThread=new Thread(new Runnable() {          
            @Override
            public void run() {
                for(int i=0;i<vector.size();i++){
                    Thread.yield();
                    // 该处会报java.lang.ArrayIndexOutOfBoundsException异常
                    System.out.println(vector.get(i));
                }
            }
        });         
        removeThread.start();
        printThread.start();
        while(Thread.activeCount()>20);
    }
}
```

* ArrayList的线程安全版本除了Vector之外，还可以通过Collections.synchronizedList方法生成同步代理对象获得线程安全的功能（其实现是把原集合包装成了新的同步集合）。
* 尽管Vector是同步的，但其由于在操作上大量使用synchronized关键字,这种方式严重影响效率,在多线程场景下，还是推荐使用CopyOnWriteArrayList。(注意CopyOnWriteArrayList的实现机制，导致它只能保证数据最终一致性，无法保证实时一致；还有比较占用内存，容易引发GC问题，所以请慎重使用。)
*即使 Vector方法有synchronized关键字，对于每个方法是线程安全的，是原子的，但是使用不当人存在线程安全问题，如类似：Vector.set(0,Vector.get(0)+1)，
Vector只保证自身方法操作的原子性。

### LinkedList
LinkedList是基于双向链表实现的。  
基于链表数据结构的特性，LinkedList的插入删除操作性能表现优异，但是随机访问能力很差。  
LinkedList无容量限制（不存在扩容拷贝数据问题）。  
LinkedList允许null元素。  
LinkedList还实现了Deque接口，可用作堆栈、队列或双端队列。  
注意，LinkedList未实现同步，是非线程安全的。

#### 实现原理
①数据存储是基于双向链表实现的。  
②插入数据很快。先是在双向链表中找到要插入节点的位置index，找到之后，再插入一个新节点。 双向链表查找index位置的节点时，有一个加速动作：若index < 双向链表长度的1/2，则从前向后查找; 否则，从后向前查找。  
③删除数据很快。先是在双向链表中找到要插入节点的位置index，找到之后，进行如下操作：node.previous.next = node.next;node.next.previous = node.previous;node = null 。查找节点过程和插入一样。  
④获取数据很慢，需要从Head节点进行查找。  
⑤遍历数据很慢，每次获取数据都需要从头开始。

```java
public class LinkedList<E>
    extends AbstractSequentialList<E>
    implements List<E>, Deque<E>, Cloneable, java.io.Serializable
{
    //...
    transient Node<E> first;
    transient Node<E> last;
    public boolean add(E e) {
        linkLast(e);
        return true;
    }
    void linkLast(E e) {
        final Node<E> l = last;
        final Node<E> newNode = new Node<>(l, e, null);
        last = newNode;
        if (l == null)
            first = newNode;
        else
            l.next = newNode;
        size++;
        modCount++;
    }
    public void push(E e) {
        addFirst(e);
    }
    public E pop() {
        return removeFirst();
    }
    public boolean offer(E e) {
        return add(e);
    }
    public E poll() {
        final Node<E> f = first;
        return (f == null) ? null : unlinkFirst(f);
    }
    public E peek() {
        final Node<E> f = first;
        return (f == null) ? null : f.item;
    }
    //...
}
```

#### 使用注意
* LinkedList的特点是该数据结构的两端读写能力很快，所以其使用场景一般为堆栈、队列或双端队列。

### CopyOnWriteArrayList
CopyOnWrite容器即写时复制的容器，主要的特性为读写分离、最终一致。  

#### 实现原理
当我们往一个容器添加元素的时候，不直接往当前容器添加，而是先加锁，然后将当前容器进行Copy，复制出一个新的容器，然后新的容器里添加元素，添加完元素之后，再将原容器的引用指向新的容器，最后解锁。这样做的好处是我们可以对CopyOnWrite容器进行并发的读，而不需要加锁，因为当前容器不会添加任何元素。所以CopyOnWrite容器也是一种读写分离的思想，读和写不同的容器。CopyOnWrite容器非常有用，可以在非常多的并发场景中使用到。

```java
public class CopyOnWriteArrayList<E>
    implements List<E>, RandomAccess, Cloneable, java.io.Serializable {
    final transient ReentrantLock lock = new ReentrantLock();
    private transient volatile Object[] array;
    public boolean add(E e) {
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            Object[] elements = getArray();
            int len = elements.length;
            Object[] newElements = Arrays.copyOf(elements, len + 1);
            newElements[len] = e;
            setArray(newElements);
            return true;
        } finally {
            lock.unlock();
        }
    }
    final void setArray(Object[] a) {
        array = a;
    }
    public E get(int index) {
        return get(getArray(), index);
    }
    final Object[] getArray() {
        return array;
    }
    public E remove(int index) {
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            Object[] elements = getArray();
            int len = elements.length;
            E oldValue = get(elements, index);
            int numMoved = len - index - 1;
            if (numMoved == 0)
                setArray(Arrays.copyOf(elements, len - 1));
            else {
                Object[] newElements = new Object[len - 1];
                System.arraycopy(elements, 0, newElements, 0, index);
                System.arraycopy(elements, index + 1, newElements, index,
                                 numMoved);
                setArray(newElements);
            }
            return oldValue;
        } finally {
            lock.unlock();
        }
    }
}
```

#### 使用注意
* CopyOnWrite并发容器用于读多写少的并发场景。比如白名单，黑名单，商品类目的访问和更新场景，假如我们有一个搜索网站，用户在这个网站的搜索框中，输入关键字搜索内容，但是某些关键字不允许被搜索。这些不能被搜索的关键字会被放在一个黑名单当中，黑名单每天晚上更新一次。当用户搜索时，会检查当前关键字在不在黑名单当中，如果在，则提示不能搜索。
* 尽量使用批量添加(addAll)。因为每次添加，容器每次都会进行复制，所以减少添加次数，可以减少容器的复制次数。

```java
public class Test {
	private static Logger LOGGER = Logger.getLogger(Test.class);
	public static List<String> list = new CopyOnWriteArrayList<String>();

	public static void main(String[] args) {
        // add操作中有同步操作，会很慢
		for(int j=0;j<100000;j++){ 
            list.add("nihao"+j); 
        }
        list.clear();
		// addAll操作中虽然也有同步操作，但是只做一次，所以很快
		ArrayList<String> arraylist = new ArrayList<String>(100000);
		for (int j = 0; j < 100000; j++) {
			arraylist.add("test" + j);
		}
		list.addAll(arraylist);
	}
}
```

* 因为CopyOnWrite的写时复制机制，所以在进行写操作的时候，内存里会同时驻扎两个对象的内存，旧的对象和新写入的对象，容易引起内存问题。
* CopyOnWriteArrayList只能保证数据的最终一致性，不能保证数据的实时一致性。所以如果你希望写入的的数据，马上能读到，请不要使用CopyOnWriteArrayList。
* CopyOnWriteArrayList虽然是线程安全的，但是只是一种相对的线程安全而不是绝对的线程安全，它只能够保证增、删、改、查的单个操作一定是原子的，不会被打断，但是如果组合起来用，并不能保证线程安全性。
*要保证遍历的安全性，必须通过迭代器遍历的方式，因为通过迭代器遍历依赖的是某一时刻的容器数组对象(snapshot，对于ArrayList 迭代器并没有提供快照)，该对象的数据不会发生变化。  

```java
public class Test {
	private static Logger LOGGER = Logger.getLogger(Test.class);
	public static List<String> list = new CopyOnWriteArrayList<String>();

	public static void main(String[] args) {
		initList();

		new Thread(new Runnable() {
			@Override
			public void run() {
				for (int o = 0; o < list.size(); o++) {
					try {
						// 延长两个原子操作的时间间隔
						if (o == 99999) {
							sleep(3000);
							LOGGER.info("for:" + list.get(o));
						}
					} catch (Exception e) {
						LOGGER.error(e.toString());
					}
				}
			}
		}).start();

		new Thread(new Runnable() {
			@Override
			public void run() {
				int i = 0;
				for (String item : list) {
					try {
						// 延长两个原子操作的时间间隔
						if (i == 99999) {
							sleep(3000);
							LOGGER.info("foreach:" + item);
						}
					} catch (Exception e) {
						LOGGER.error(e.toString());
					}
					i++;
				}
			}
		}).start();

		new Thread(new Runnable() {
			@Override
			public void run() {
				sleep(2000);
				list.remove(99999);
			}
		}).start();
	}

	private static void sleep(long time) {
		try {
			Thread.sleep(time);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
	}
}

public class CopyOnWriteArrayList<E>
    implements List<E>, RandomAccess, Cloneable, java.io.Serializable {
    //...
    public Iterator<E> iterator() {
        return new COWIterator<E>(getArray(), 0);
    }
    static final class COWIterator<E> implements ListIterator<E> {
        /** Snapshot of the array */
        private final Object[] snapshot;
        private int cursor;

        private COWIterator(Object[] elements, int initialCursor) {
            cursor = initialCursor;
            snapshot = elements;
        }
        public boolean hasNext() {
            return cursor < snapshot.length;
        }
        @SuppressWarnings("unchecked")
        public E next() {
            if (! hasNext())
                throw new NoSuchElementException();
            return (E) snapshot[cursor++];
        }
        //...
    }
}
```

### CopyOnWriteArraySet（类似CopyOnWriteArraySet）


## 队列
队列是一种非常常用的数据结构，一进一出，先进先出。 

　　在Java并发包中提供了两种类型的队列，非阻塞队列与阻塞队列，当然它们都是线程安全的，无需担心在多线程并发环境所带来的不可预知的问题。为什么会有非阻塞和阻塞之分呢？这里的非阻塞与阻塞在于有界与否，也就是在初始化时有没有给它一个默认的容量大小，对于阻塞有界队列来讲，如果队列满了的话，则任何线程都会阻塞不能进行入队操作，反之队列为空的话，则任何线程都不能进行出队操作。而对于非阻塞无界队列来讲则不会出现队列满或者队列空的情况。它们俩都保证线程的安全性，即不能有一个以上的线程同时对队列进行入队或者出队操作。 

　　非阻塞队列：ConcurrentLinkedQueue 

　　阻塞队列：ArrayBlockingQueue、LinkedBlockingQueue、……

### Stack
Stack类表示后进先出（LIFO）的对象堆栈。  
它通过五个操作对类Vector进行了扩展 ，允许将向量视为堆栈。它提供了通常的push和pop操作，以及取堆栈顶点的peek方法、测试堆栈是否为空的empty方法、在堆栈中查找项并确定到堆栈顶距离的search方法。  
通过继承Vector类，Stack类可以很容易的实现他本身的功能。因为大部分的功能在Vector里面已经提供支持了。由于Vector是通过数组实现的，这就意味着，Stack也是通过数组实现的，而非链表。

#### 实现原理
Stack类通过继承Vector类实现本身功能。Stack最大的长度取决于vector里面数组能有多长(vector最大能取到Integer.MAX_VALUE)。  

```java
public class Stack<E> extends Vector<E> {
    //...
    public Stack() {
    }
    public E push(E item) {
        addElement(item);
        return item;
    }
    public synchronized E pop() {
        E       obj;
        int     len = size();
        obj = peek();
        removeElementAt(len - 1);
        return obj;
    }
    public synchronized E peek() {
        int     len = size();
        if (len == 0)
            throw new EmptyStackException();
        return elementAt(len - 1);
    }
    public boolean empty() {
        return size() == 0;
    }
    public synchronized int search(Object o) {
        int i = lastIndexOf(o);
        if (i >= 0) {
            return size() - i;
        }
        return -1;
    }
    //...
}
``` 


#### 使用注意
* 由于自身功能基本靠继承Vector实现，所以Vector存在的问题Stack也同样存在。所以，如果你要使用栈做类似的业务.那么单线程的你可以选择LinkedList,多线程情况你可以选择非阻塞队列ConcurrentLinkedQueue或者阻塞队列BlockingQueue。  


### [整理中]ConcurrentLinkedQueue
ConcurrentLinkedQueue是一个基于链接节点的无界线程安全队列，它采用先进先出的规则对节点进行排序，当我们添加一个元素的时候，它会添加到队列的尾部，当我们获取一个元素时，它会返回队列头部的元素。该队列为单向队列，对应双向队列的实现是ConcurrentLinkedDeque。基于CAS的“wait－free”（常规无等待）来实现，CAS并不是一个算法，它是一个CPU直接支持的硬件指令，这也就在一定程度上决定了它的平台相关性。

#### 实现原理

#### 使用注意

### [整理中]ArrayBlockingQueue

#### 实现原理

#### 使用注意

### [整理中]LinkedBlockingQueue

#### 实现原理

#### 使用注意

### [整理中]DelayQueue

#### 实现原理

#### 使用注意

### [整理中]SynchronousQueue

#### 实现原理

#### 使用注意

### HashSet
HashSet实现Set接口，由哈希表（实际上是一个HashMap实例）支持。此类允许使用null元素。HashSet中不允许有重复元素。

#### 实现原理
对于HashSet而言，它是基于HashMap实现的，HashSet跟HashMap一样，都是一个存放链表的数组。HashSet底层使用HashMap来保存所有元素，更确切的说，HashSet中的元素，只是存放在了底层HashMap的key上， 而value使用一个static final的Object对象标识。因此HashSet 的实现比较简单，相关HashSet的操作，基本上都是直接调用底层HashMap的相关方法来完成。HashSet中add方法调用的是底层HashMap中的put()方法，而如果是在HashMap中调用put，首先会判断key是否存在，如果key存在则修改value值，如果key不存在这插入这个key-value。

```java
public class HashSet<E>
    extends AbstractSet<E>
    implements Set<E>, Cloneable, java.io.Serializable
{
    private transient HashMap<E,Object> map;
    private static final Object PRESENT = new Object();
    public boolean add(E e) {
        return map.put(e, PRESENT)==null;
    }
    public boolean remove(Object o) {
        return map.remove(o)==PRESENT;
    }
    //...
}
```

#### 使用注意
* 由于HashSet是基本是完全基于HashMap实现的，所以注意点和HashMap基本一样。

### TreeSet
TreeSet 是一个有序的集合，它的作用是提供有序的Set集合。它继承于AbstractSet抽象类，实现了NavigableSet<E>, Cloneable, java.io.Serializable接口。  
TreeSet 继承于AbstractSet，所以它是一个Set集合，具有Set的属性和方法。  
TreeSet 实现了NavigableSet接口，意味着它支持一系列的导航方法。比如查找与指定目标最匹配项。  
TreeSet 实现了Cloneable接口，意味着它能被克隆。  
TreeSet 实现了java.io.Serializable接口，意味着它支持序列化。  
TreeSet是基于TreeMap实现的。TreeSet中的元素支持2种排序方式：自然排序 或者 根据创建TreeSet 时提供的 Comparator 进行排序。这取决于使用的构造方法。  
TreeSet为基本操作（add、remove 和 contains）提供受保证的 log(n) 时间开销。  
另外，TreeSet未实现同步，是非线程安全的。

#### 实现原理
TreeSet继承于AbstractSet，并且实现了NavigableSet接口。TreeSet的本质是一个"有序的，并且没有重复元素"的集合，它是通过TreeMap实现的。TreeSet中含有一个"NavigableMap类型的成员变量"m，而m实际上是"TreeMap的实例"。

```java
public class TreeSet<E> extends AbstractSet<E>
    implements NavigableSet<E>, Cloneable, java.io.Serializable
{
    private transient NavigableMap<E,Object> m;
    TreeSet(NavigableMap<E,Object> m) {
        this.m = m;
    }
    public TreeSet() {
        this(new TreeMap<E,Object>());
    }
}
```

#### 使用注意
* 由于TreeSet是基本是完全基于TreeMap实现的，所以注意点和TreeMap基本一样。

### LinkedHashSet
LinkedHashSet是具有可预知迭代顺序的Set接口的哈希表和链接列表实现。它继承于 HashSet、又基于 LinkedHashMap 来实现的。

#### 实现原理
LinkedHashSet底层使用LinkedHashMap来保存所有元素，它继承于HashSet，其所有的方法操作上又与HashSet相同，因此 LinkedHashSet 的实现上非常简单，只提供了四个构造方法，并通过传递一个标识参数，调用父类的构造器，底层构造一个LinkedHashMap来实现，在相关操作上与父类HashSet的操作相同，直接调用父类HashSet的方法即可。

```java
public class LinkedHashSet<E>
    extends HashSet<E>
    implements Set<E>, Cloneable, java.io.Serializable {
    public LinkedHashSet(int initialCapacity, float loadFactor) {
        super(initialCapacity, loadFactor, true);
    }
    public LinkedHashSet(int initialCapacity) {
        super(initialCapacity, .75f, true);
    }
    public LinkedHashSet() {
        super(16, .75f, true);
    }
    public LinkedHashSet(Collection<? extends E> c) {
        super(Math.max(2*c.size(), 11), .75f, true);
        addAll(c);
    }
    //...
}
public class HashSet<E>
    extends AbstractSet<E>
    implements Set<E>, Cloneable, java.io.Serializable
{
    //...
    HashSet(int initialCapacity, float loadFactor, boolean dummy) {
        map = new LinkedHashMap<>(initialCapacity, loadFactor);
    }
    //...
}
```

#### 使用注意
* 由于LinkedHashSet是基本是完全基于LinkedHashMap实现的，所以注意点和LinkedHashMap基本一样。

## 哈希表

### HashMap
HashMap 是一个散列表，它存储的内容是键值对(key-value)映射。  
HashMap 继承于AbstractMap，实现了Map、Cloneable、java.io.Serializable接口。  
HashMap 的key、value都可以为null。  
HashMap 的基本操作 containsKey、get、put 和 remove 的时间复杂度是 O(1) 。  
此外，HashMap中的映射不是有序的。
注意，HashMap未实现同步，是非线程安全的。  

#### 实现原理
HashMap底层实现是数组，数组的每一项是一条链表（Java8中是链表或者红黑树），这样设计的目的是为了综合数组的快速访问性和链表的快速存储性。  
HashMap通过hash的方法，使用put和get存储和获取对象。存储对象时，我们将K/V传给put方法时，它调用hashCode计算hash从而得到bucket位置，进一步存储，HashMap会根据当前bucket的占用情况自动调整容量(超过Load Facotr则resize为原来的2倍)。获取对象时，我们将K传给get，它调用hashCode计算hash从而得到bucket位置，并进一步调用equals()方法确定键值对。如果发生碰撞的时候，Hashmap通过链表将产生碰撞冲突的元素组织起来，在Java 8中，如果一个bucket中碰撞冲突的元素超过某个限制(默认是8)，则使用红黑树来替换链表，从而提高速度。

```java
public class HashMap<K,V> extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable {
    transient Node<K,V>[] table;
    static class Node<K,V> implements Map.Entry<K,V> {
        final int hash;
        final K key;
        V value;
        Node<K,V> next;
        Node(int hash, K key, V value, Node<K,V> next) {
            this.hash = hash;
            this.key = key;
            this.value = value;
            this.next = next;
        }
        public final int hashCode() {
            return Objects.hashCode(key) ^ Objects.hashCode(value);
        }
        public final boolean equals(Object o) {
            if (o == this)
                return true;
            if (o instanceof Map.Entry) {
                Map.Entry<?,?> e = (Map.Entry<?,?>)o;
                if (Objects.equals(key, e.getKey()) &&
                    Objects.equals(value, e.getValue()))
                    return true;
            }
            return false;
        }
        //...
    }
    public V get(Object key) {
        Node<K,V> e;
        return (e = getNode(hash(key), key)) == null ? null : e.value;
    }
    final Node<K,V> getNode(int hash, Object key) {
        Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
        if ((tab = table) != null && (n = tab.length) > 0 &&
            (first = tab[(n - 1) & hash]) != null) {
            if (first.hash == hash && // always check first node
                ((k = first.key) == key || (key != null && key.equals(k))))
                return first;
            if ((e = first.next) != null) {
                if (first instanceof TreeNode)
                    return ((TreeNode<K,V>)first).getTreeNode(hash, key);
                do {
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        return e;
                } while ((e = e.next) != null);
            }
        }
        return null;
    }
    static final int hash(Object key) {
        int h;
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    }
    public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
    }
    final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
        else {
            Node<K,V> e; K k;
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
            else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else {
                for (int binCount = 0; ; ++binCount) {
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;
        if (++size > threshold)
            resize();
        afterNodeInsertion(evict);
        return null;
    }
}
```

#### 使用注意
* 由于数组容量不足而自动扩容的机制会造成性能损耗，在实际添加大量元素前，可以提前预估并指定HashMap实例的容量，指定具体的大小（由于HashMap的特殊扩容算法，建议使用guava的工具类Maps.newHashMapWithExpectedSize(7)去创建容器实例）。
* 由于HashMap的存储与查找是基于key对象类型的hashCode()和equals()方法的，所以在自定义的key类型中重写hashCode()和equals()方法时一定要慎重，避免由于重写实现不当而导致在HashMap的查询、存储上引起性能、错误等问题。
* HashMap未实现同步，是非线程安全的，即任一时刻可以有多个线程同时写HashMap，可能会导致数据的不一致。如果需要满足线程安全，可以用 Collections的synchronizedMap方法使HashMap具有线程安全的能力，或者使用ConcurrentHashMap。

### [整理中]ConcurrentHashMap

#### 实现原理

#### 使用注意

### TreeMap
TreeMap 是一个有序的key-value集合，它是通过红黑树实现的。  
TreeMap 继承于AbstractMap，所以它是一个Map，即一个key-value集合。  
TreeMap 实现了NavigableMap接口，意味着它支持一系列的导航方法。比如返回有序的key集合。  
TreeMap 实现了Cloneable接口，意味着它能被克隆。  
TreeMap 实现了java.io.Serializable接口，意味着它支持序列化。   
TreeMap 支持根据其键的自然顺序进行排序，或者根据创建映射时提供的 Comparator 进行排序。  
TreeMap 的基本操作 containsKey、get、put 和 remove 的时间复杂度是 O(log(n))。  
注意，TreeMap未实现同步，是非线程安全的。  

#### 实现原理
TreeMap基于红黑树（Red-Black tree）实现。创建树节点时TreeMap根据其键的自然顺序进行排序，或者根据创建TreeMap时提供的 Comparator 进行排序，具体取决于使用的构造方法。TreeMap的有序遍历是通过 R-B tree LDR(红黑树中序遍历)算法保证的。

```java
public class TreeMap<K,V> extends AbstractMap<K,V>
    implements NavigableMap<K,V>, Cloneable, java.io.Serializable
{
    //...
    static final class Entry<K,V> implements Map.Entry<K,V> {
        K key;
        V value;
        Entry<K,V> left;
        Entry<K,V> right;
        Entry<K,V> parent;
        boolean color = BLACK;
        Entry(K key, V value, Entry<K,V> parent) {
            this.key = key;
            this.value = value;
            this.parent = parent;
        }
        //...
    }
    private final Comparator<? super K> comparator;
    private transient Entry<K,V> root;

    public TreeMap(Comparator<? super K> comparator) {
        this.comparator = comparator;
    }
    final Entry<K,V> getEntry(Object key) {
        // Offload comparator-based version for sake of performance
        if (comparator != null)
            return getEntryUsingComparator(key);
        if (key == null)
            throw new NullPointerException();
        @SuppressWarnings("unchecked")
            Comparable<? super K> k = (Comparable<? super K>) key;
        Entry<K,V> p = root;
        while (p != null) {
            int cmp = k.compareTo(p.key);
            if (cmp < 0)
                p = p.left;
            else if (cmp > 0)
                p = p.right;
            else
                return p;
        }
        return null;
    }
    public V put(K key, V value) {
        Entry<K,V> t = root;
        if (t == null) {
            compare(key, key); // type (and possibly null) check

            root = new Entry<>(key, value, null);
            size = 1;
            modCount++;
            return null;
        }
        int cmp;
        Entry<K,V> parent;
        // split comparator and comparable paths
        Comparator<? super K> cpr = comparator;
        if (cpr != null) {
            do {
                parent = t;
                cmp = cpr.compare(key, t.key);
                if (cmp < 0)
                    t = t.left;
                else if (cmp > 0)
                    t = t.right;
                else
                    return t.setValue(value);
            } while (t != null);
        }
        else {
            if (key == null)
                throw new NullPointerException();
            @SuppressWarnings("unchecked")
                Comparable<? super K> k = (Comparable<? super K>) key;
            do {
                parent = t;
                cmp = k.compareTo(t.key);
                if (cmp < 0)
                    t = t.left;
                else if (cmp > 0)
                    t = t.right;
                else
                    return t.setValue(value);
            } while (t != null);
        }
        Entry<K,V> e = new Entry<>(key, value, parent);
        if (cmp < 0)
            parent.left = e;
        else
            parent.right = e;
        fixAfterInsertion(e);
        size++;
        modCount++;
        return null;
    }
    //...
}
```    

#### 使用注意
* TreeMap是通过红黑树实现的，更适用于对顺序有要求的场景，对于插入顺序有要求的可以使用LinkedHashMap，而对于顺序完全没有要求的，为了更好的读写性能还是建议使用HashMap。
* TreeMap默认支持排序，前提是key类型必须实现java.lang.Comparable接口，否则运行时会报错。

```java
public class TreeMapTest {
	public static void main(String[] args) {
		Map<NoComparable,String> treeMap=new TreeMap<NoComparable,String>();
		treeMap.put(new NoComparable(), "NoComparable");
		System.out.println(treeMap);//TreeMapTest$NoComparable cannot be cast to java.lang.Comparable
	}
	
	public static class NoComparable{
	}
}
```

### LinkedHashMap
LinkedHashMap是Hash表和链表的实现，并且依靠着双向链表保证了迭代顺序是插入顺序或者最后访问顺序。  
LinkedHashMap允许使用null值和null键。  
注意，LinkedHashMap未实现同步，是非线程安全的。

#### 实现原理
对于LinkedHashMap而言，它继承于HashMap、底层使用哈希表与双向链表来保存所有元素。其基本操作与父类HashMap相似，它通过重写父类相关的方法，来实现自己的双向链表特性。

```java
public class LinkedHashMap<K,V> extends HashMap<K,V> implements Map<K,V>
{
    //...
    static class Entry<K,V> extends HashMap.Node<K,V> {
        Entry<K,V> before, after;
        Entry(int hash, K key, V value, Node<K,V> next) {
            super(hash, key, value, next);
        }
    }
    public LinkedHashMap(int initialCapacity, float loadFactor) {  
        super(initialCapacity, loadFactor);  
        accessOrder = false;  
    }
    void addEntry(int hash, K key, V value, int bucketIndex) {  
        createEntry(hash, key, value, bucketIndex);  
        Entry<K,V> eldest = header.after;  
        if (removeEldestEntry(eldest)) {  
            removeEntryForKey(eldest.key);  
        } else {  
            if (size >= threshold)  
                resize(2 * table.length);  
        }  
    }
    void createEntry(int hash, K key, V value, int bucketIndex) {  
        HashMap.Entry<K,V> old = table[bucketIndex];  
        Entry<K,V> e = new Entry<K,V>(hash, key, value, old);  
        table[bucketIndex] = e;  
        e.addBefore(header);  
        size++;  
    }
    private void addBefore(Entry<K,V> existingEntry) {  
        after  = existingEntry;  
        before = existingEntry.before;  
        before.after = this;  
        after.before = this;  
    }
    public V get(Object key) {  
        Entry<K,V> e = (Entry<K,V>)getEntry(key);  
        if (e == null)  
            return null;  
        e.recordAccess(this);  
        return e.value;  
    }
    void recordAccess(HashMap<K,V> m) {  
        LinkedHashMap<K,V> lm = (LinkedHashMap<K,V>)m;  
        if (lm.accessOrder) {  
            lm.modCount++;  
            remove();  
            addBefore(lm.header);  
        }  
    } 
    //...
}
```

#### 使用注意
* LinkedHashMap定义了排序模式accessOrder，该属性为boolean型变量，对于访问顺序，为true；对于插入顺序，则为false。通过排序模式特性很容易构建LRU缓存(将最近一次使用时间离现在时间最远的且出缓存限制大小的数据删除掉)。

```java
public class LRUCache<K, V> {
	private final int MAX_CACHE_SIZE;
	private final float DEFAULT_LOAD_FACTOR = 0.75f;
	LinkedHashMap<K, V> map;

	public LRUCache(int cacheSize) {
		MAX_CACHE_SIZE = cacheSize;
		// 根据cacheSize和加载因子计算hashmap的capactiy，+1确保当达到cacheSize上限时不会触发hashmap的扩容。
		int capacity = (int) Math.ceil(MAX_CACHE_SIZE / DEFAULT_LOAD_FACTOR) + 1;
		map = new LinkedHashMap<K, V>(capacity, DEFAULT_LOAD_FACTOR, true) {
			private static final long serialVersionUID = 1L;

			@Override
			protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
				return size() > MAX_CACHE_SIZE;
			}
		};
	}

	@Override
	public String toString() {
		StringBuilder sb = new StringBuilder();
		for (Map.Entry<K, V> entry : map.entrySet()) {
			sb.append(String.format("%s:%s ", entry.getKey(), entry.getValue()));
		}
		return sb.toString();
	}

	public synchronized void put(K key, V value) {
		map.put(key, value);
	}

	public synchronized V get(K key) {
		return map.get(key);
	}

	public synchronized void remove(K key) {
		map.remove(key);
	}

	public synchronized int size() {
		return map.size();
	}

	public synchronized void clear() {
		map.clear();
	}

	public static void main(String[] args) {
		LRUCache<Integer, String> lru = new LRUCache<Integer, String>(5);
		lru.put(1, "11");
		lru.put(2, "11");
		lru.put(3, "11");
		lru.put(4, "11");
		lru.put(5, "11");
		System.out.println(lru.toString());
		lru.put(6, "66");
		lru.get(2);
		lru.put(7, "77");
		lru.get(4);
		System.out.println(lru.toString());
		System.out.println();
	}
}
```

* LinkedHashMap继承于HashMap，所以HashMap需要注意的地方LinkedHashMap也需要特别注意。

## 功能对比

实现类|线程安全|保持插入顺序|可重复|可排序|扩容算法|性能对比  
-|-|-|-|-|-|-  
ArrayList|否|是|是|否|*1.5|随机访问元素较快，<br/>但插入、删除元素较慢（数组特性）  
Vector|是|是|是|否|*1|随机访问元素较快，<br/>但插入、删除元素较慢（数组特性），<br/>线程同步机制导致其较ArrayList效率降低很多，<br/>故实际应用场景不多  
LinkedList|否|是|是|否|+1|插入、删除元素较快，<br/>但随机访问元素较慢（链表特性）  
HashSet|否|否|否|否|*2|使用散列结构存储，<br/>插入、删除元素较ArrayList快，<br/>随机访问元素较LinkedList快。  
TreeSet|否|否|否|是|+1|使用红黑树结构存储，默认升序，<br/>插入、删除、随机访问元素较HashSet慢。  
LinkedHashSet|否|是|否|否|*2|使用散列+双向链表结构存储，<br/>插入、删除、随机访问元素较HashSet慢。 
HashMap|否|否|否|否|*2|使用散列结构存储，<br/>插入、删除元素较ArrayList快，<br/>随机访问元素较LinkedList快。  
TreeMap|否|否|否|是|+1|使用红黑树结构存储，默认升序，<br/>插入、删除、随机访问元素较HashMap慢。 
LinkedHashMap|否|是|否|否|*2|使用散列+双向链表结构存储，<br/>插入、删除、随机访问元素较HashMap慢。

##  list、set、map 的区别
1、List（有序、可重复）
List里存放的对象是有序的，同时也是可以重复的，List关注的是索引，拥有一系列和索引相关的方法，查询速度快。因为往list集合里插入或删除数据时，会伴随着后面数据的移动，所有插入删除数据速度慢。

2、Set（无序、不能重复）
Set里存放的对象是无序，不能重复的，集合中的对象不按特定的方式排序，只是简单地把对象加入集合中。

3、Map（键值对、键唯一、值不唯一）
Map集合中存储的是键值对，键不能重复，值可以重复。根据键得到值，对map集合遍历时先得到键的set集合，对set集合进行遍历，得到相应的值。

对比如下：

![](http://topweshare.qiniudn.com/1524530524.png?imageMogr2/thumbnail/!70p)

## 集合的遍历方式
在类集中提供了以下四种的常见输出方式：

1）Iterator：迭代输出，是使用最多的输出方式。

2）ListIterator：是Iterator的子接口，专门用于输出List中的内容。

3）foreach输出：JDK1.5之后提供的新功能，可以输出数组或集合。

4）for循环

代码示例如下：

 for的形式：for（int i=0;i<arr.size();i++）{...}

 foreach的形式： for（int　i：arr）{...}

 iterator的形式：
Iterator it = arr.iterator();
while(it.hasNext()){ object o =it.next(); ...}

## ArrayList和LinkedList 区别
ArrayList和LinkedList在用法上没有区别，但是在功能上还是有区别的。LinkedList经常用在增删操作较多而查询操作很少的情况下，ArrayList则相反。

## **Map集合的实现类**

实现类：HashMap、Hashtable、LinkedHashMap和TreeMap

HashMap 

HashMap是最常用的Map，它根据键的HashCode值存储数据，根据键可以直接获取它的值，具有很快的访问速度，遍历时，取得数据的顺序是完全随机的。因为键对象不可以重复，所以HashMap最多只允许一条记录的键为Null，允许多条记录的值为Null，是非同步的

Hashtable

Hashtable与HashMap类似，是HashMap的线程安全版，它支持线程的同步，即任一时刻只有一个线程能写Hashtable，因此也导致了Hashtale在写入时会比较慢，它继承自Dictionary类，不同的是它不允许记录的键或者值为null，同时效率较低。

ConcurrentHashMap

线程安全，并且锁分离。ConcurrentHashMap内部使用段(Segment)来表示这些不同的部分，每个段其实就是一个小的hash table，它们有自己的锁。只要多个修改操作发生在不同的段上，它们就可以并发进行。

LinkedHashMap

LinkedHashMap保存了记录的插入顺序，在用Iteraor遍历LinkedHashMap时，先得到的记录肯定是先插入的，在遍历的时候会比HashMap慢，有HashMap的全部特性。

TreeMap

TreeMap实现SortMap接口，能够把它保存的记录根据键排序，默认是按键值的升序排序（自然顺序），也可以指定排序的比较器，当用Iterator遍历TreeMap时，得到的记录是排过序的。不允许key值为空，非同步的；

## **map的遍历**

第一种：KeySet()
将Map中所有的键存入到set集合中。因为set具备迭代器。所有可以迭代方式取出所有的键，再根据get方法。获取每一个键对应的值。 keySet():迭代后只能通过get()取key 。
取到的结果会乱序，是因为取得数据行主键的时候，使用了HashMap.keySet()方法，而这个方法返回的Set结果，里面的数据是乱序排放的。
典型用法如下：
Map map = new HashMap();
map.put("key1","lisi1");
map.put("key2","lisi2");
map.put("key3","lisi3");
map.put("key4","lisi4");  
//先获取map集合的所有键的set集合，keyset（）
Iterator it = map.keySet().iterator();
 //获取迭代器
while(it.hasNext()){
Object key = it.next();
System.out.println(map.get(key));
}

第二种：entrySet（）
Set<Map.Entry<K,V>> entrySet() //返回此映射中包含的映射关系的 Set 视图。（一个关系就是一个键-值对），就是把(key-value)作为一个整体一对一对地存放到Set集合当中的。Map.Entry表示映射关系。entrySet()：迭代后可以e.getKey()，e.getValue()两种方法来取key和value。返回的是Entry接口。
典型用法如下：
Map map = new HashMap();
map.put("key1","lisi1");
map.put("key2","lisi2");
map.put("key3","lisi3");
map.put("key4","lisi4");
//将map集合中的映射关系取出，存入到set集合
Iterator it = map.entrySet().iterator();
while(it.hasNext()){
Entry e =(Entry) it.next();
System.out.println("键"+e.getKey () + "的值为" + e.getValue());
}
推荐使用第二种方式，即entrySet()方法，效率较高。
对于keySet其实是遍历了2次，一次是转为iterator，一次就是从HashMap中取出key所对于的value。而entryset只是遍历了第一次，它把key和value都放到了entry中，所以快了。两种遍历的遍历时间相差还是很明显的。

## Vector和ArrayList

1，vector是线程同步的，所以它也是线程安全的，而arraylist是线程异步的，是不安全的。如果不考虑到线程的安全因素，一般用arraylist效率比较高。
2，如果集合中的元素的数目大于目前集合数组的长度时，vector增长率为目前数组长度的100%，而arraylist增长率为目前数组长度的50%。如果在集合中使用数据量比较大的数据，用vector有一定的优势。
3，如果查找一个指定位置的数据，vector和arraylist使用的时间是相同的，如果频繁的访问数据，这个时候使用vector和arraylist都可以。而如果移动一个指定位置会导致后面的元素都发生移动，这个时候就应该考虑到使用linklist,因为它移动一个指定位置的数据时其它元素不移动。
ArrayList 和Vector是采用数组方式存储数据，此数组元素数大于实际存储的数据以便增加和插入元素，都允许直接序号索引元素，但是插入数据要涉及到数组元素移动等内存操作，所以索引数据快，插入数据慢，Vector由于使用了synchronized方法（线程安全）所以性能上比ArrayList要差，LinkedList使用双向链表实现存储，按序号索引数据需要进行向前或向后遍历，但是插入数据时只需要记录本项的前后项即可，所以插入数度较快。

## arraylist和linkedlist

1.ArrayList是实现了基于动态数组的数据结构，LinkedList基于链表的数据结构。
2.对于随机访问get和set，ArrayList觉得优于LinkedList，因为LinkedList要移动指针。
3.对于新增和删除操作add和remove，LinedList比较占优势，因为ArrayList要移动数据。 这一点要看实际情况的。若只对单条数据插入或删除，ArrayList的速度反而优于LinkedList。但若是批量随机的插入删除数据，LinkedList的速度大大优于ArrayList. 因为ArrayList每插入一条数据，要移动插入点及之后的所有数据。


## HashMap与TreeMap

1、 HashMap通过hashcode对其内容进行快速查找，而TreeMap中所有的元素都保持着某种固定的顺序，如果你需要得到一个有序的结果你就应该使用TreeMap（HashMap中元素的排列顺序是不固定的）。
2、在Map 中插入、删除和定位元素，HashMap是最好的选择。但如果您要按自然顺序或自定义顺序遍历键，那么TreeMap会更好。使用HashMap要求添加的键类明确定义了hashCode()和 equals()的实现。
两个map中的元素一样，但顺序不一样，导致hashCode()不一样。
同样做测试：
在HashMap中，同样的值的map,顺序不同，equals时，false;
而在treeMap中，同样的值的map,顺序不同,equals时，true，说明，treeMap在equals()时是整理了顺序了的。

## HashTable与HashMap

