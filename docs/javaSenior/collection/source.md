> 参考: [通过源码一步一步分析ArrayList扩容机制](https://github.com/Snailclimb/JavaGuide/blob/master/docs/java/collection/ArrayList-Grow.md)

### 一.ArrayList

ArrayList 底层是数组队列,相当于**动态数组**.与 Java 中的数组相比,它的容量能动态增长.线性表的顺序存储,插入删除元素的时间复杂度为**O（n）**,求表长以及增加元素,取第 i 元素的时间复杂度为**O（1）** .

它继承于 **AbstractList**，实现了 **List**, **RandomAccess**, **Cloneable**, **java.io.Serializable** 这些接口。

-   ArrayList 继承了AbstractList，实现了List。它是一个数组队列，提供了相关的添加、删除、修改、遍历等功能。
-   ArrayList 实现了**RandomAccess 接口**， RandomAccess 是一个标志接口，表明实现这个这个接口的 List 集合是支持**快速随机访问**的。在 ArrayList 中，我们即可以通过元素的序号快速获取元素对象，这就是快速随机访问。
-   ArrayList 实现了**Cloneable 接口**，即覆盖了函数 clone()，**能被克隆**。
-   ArrayList 实现**java.io.Serializable 接口**，这意味着ArrayList**支持序列化**，**能通过序列化去传输**。
-   和 Vector 不同，**ArrayList 中的操作不是线程安全的**！所以，建议在单线程中才使用 ArrayList，而在多线程中可以选择 Collections中的静态方法 `List<T> synchronizedList(List<T> list)` 将其转为线程安全的。

#### 1.JDK7

ArrayList是List接口的可变数组的实现。实现了所有可选列表操作，并允许包括 null 在内的所有元素。除了实现 List 接口外，此类还提供一些方法来操作内部用来存储列表的数组的大小。

##### 1.1 核心源码

```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable {
		
	/**
	* 1.属性
	*/
	
	// 底层使用数组实现	
	private transient Object[] elementData;	// 声明对象数组变量

	/**
	* 2.构造方法
	*/

	// 构造一个默认初始容量为 10 的空列表
	public ArrayList() {
        this(10);
    }
	// 构造一个指定初始容量的空列表
	public ArrayList(int initialCapacity) {	// 10
        super();
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        this.elementData = new Object[initialCapacity];	// 分配空间10
    }
    // 构造一个包含指定 collection 的元素的列表
    // 这些元素按照该 collection 的迭代器返回它们的顺序排列的
    public ArrayList(Collection<? extends E> c) {
        elementData = c.toArray();
        size = elementData.length;
        // c.toArray might (incorrectly) not return Object[] (see 6260652)
        if (elementData.getClass() != Object[].class)
            elementData = Arrays.copyOf(elementData, size, Object[].class);
    }
    
    /**
    * 3.存储
    */
    
    // 用指定的元素替代此列表中指定位置上的元素，并返回以前位于该位置上的元素。
    public E set(int index, E element) {
    	RangeCheck(index);

    	E oldValue = (E) elementData[index];
    	elementData[index] = element;
    	return oldValue;
    }
    // 将指定的元素添加到此列表的尾部。
    public boolean add(E e) {
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        elementData[size++] = e;
        return true;
    }
	// 将指定的元素插入此列表中的指定位置。
	// 如果当前位置有元素，则向右移动当前位于该位置的元素以及所有后续元素（将其索引加 1）。
    public void add(int index, E element) {
        rangeCheckForAdd(index);
		// 如果数组长度不足，将进行扩容
        ensureCapacityInternal(size + 1);  // Increments modCount!!
         // 将 elementData 中从 Index 位置开始、长度为 size-index 的元素，
		// 拷贝到从下标为 index+1 位置开始的新的 elementData 数组中。
		// 即将当前位于该位置的元素以及所有后续元素右移一个位置。
        System.arraycopy(elementData, index, elementData, index + 1,
                         size - index);
        elementData[index] = element;
        size++;
    }
    // 从指定的位置开始，将指定 collection 中的所有元素插入到此列表中。
    public boolean addAll(int index, Collection<? extends E> c) {
        rangeCheckForAdd(index);

        Object[] a = c.toArray();
        int numNew = a.length;
        ensureCapacityInternal(size + numNew);  // Increments modCount

        int numMoved = size - index;
        if (numMoved > 0)
            System.arraycopy(elementData, index, elementData, index + numNew,
                             numMoved);

        System.arraycopy(a, 0, elementData, index, numNew);
        size += numNew;
        return numNew != 0;
    }
    
    /**
    * 4.读取
    */
    
    // 返回此列表中指定位置上的元素。
    public E get(int index) {
        rangeCheck(index);

        return elementData(index);
    }
    
    /**
    * 5.删除
    * ArrayList 提供了根据下标或者指定对象两种方式的删除功能
    */
	
	// 移除此列表中指定位置上的元素。
	public E remove(int index) {
        rangeCheck(index);
		// 快速失败的机制，通过记录 modCount 参数来实现
        modCount++;	
        E oldValue = elementData(index);

        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--size] = null; // Let gc do its work

        return oldValue;
    }
    // 移除此列表中首次出现的指定元素（如果存在）。这是应为ArrayList中允许存放重复的元素。
    public boolean remove(Object o) {
    	// 由于 ArrayList 中允许存放 null，因此下面通过两种情况来分别处理。
        if (o == null) {
            for (int index = 0; index < size; index++)
                if (elementData[index] == null) {
                	// 类似 remove(int index)，移除列表中指定位置上的元素。
                    fastRemove(index);	
                    return true;
                }
        } else {
            for (int index = 0; index < size; index++)
                if (o.equals(elementData[index])) {
                    fastRemove(index);
                    return true;
                }
        }
        return false;
    }
	
}
```

##### 1.2 底层扩容

```java
	// 添加元素举例
	public boolean add(E e) {
        ensureCapacityInternal(size + 1);  // 有效长度+1
        elementData[size++] = e;
        return true;
    }
	private void ensureCapacityInternal(int minCapacity) {	// 确认实际容量
        modCount++;
        // overflow-conscious code
        if (minCapacity - elementData.length > 0)	// 当实际容量大于10时，进行扩容
            grow(minCapacity);
    }
	
	private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;
	
	private void grow(int minCapacity) {	// 11
        // overflow-conscious code
        int oldCapacity = elementData.length;	// 10
        int newCapacity = oldCapacity + (oldCapacity >> 1);	// 10 + 10/2 相当于原先默认的1.5倍
        if (newCapacity - minCapacity < 0)	// 若扩容后比实际容量还小
            newCapacity = minCapacity;	// 赋值为实际容量
        if (newCapacity - MAX_ARRAY_SIZE > 0)	// 若扩容后比(Integer.MAX_VALUE-8)还大
            newCapacity = hugeCapacity(minCapacity);	// 赋值为Integer.MAX_VALUE
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf(elementData, newCapacity);	// 赋值为新的数组，数组长度为扩容后的长度
    }
	
	private static int hugeCapacity(int minCapacity) {
        if (minCapacity < 0) // overflow
            throw new OutOfMemoryError();
        return (minCapacity > MAX_ARRAY_SIZE) ?
            Integer.MAX_VALUE :
            MAX_ARRAY_SIZE;
    }
```

数组扩容通过一个公开的方法 `ensureCapacity(int minCapacity)` 来实现。**会将老数组中的元素重新拷贝一份到新的 数组中，每次数组容量的增长大约是其原容量的 1.5 倍**。在实际中我们可以在**构造 ArrayList 实例**时，就指定其容量，以避免数组扩容的发生。或者根据实际需求，通过**调用 ensureCapacity 方法来手动增加 ArrayList 实例的容量**, 以减少递增式再分配的数量。

ArrayList 还给我们提供了**将底层数组的容量调整为当前列表保存的实际元素的大小的功能**。它可以通过trimToSize 方法来实现。

```java
	public void trimToSize() {
        modCount++;
        int oldCapacity = elementData.length;
        if (size < oldCapacity) {
            elementData = Arrays.copyOf(elementData, size);
        }
    }
```

#### 2.JDK8

 ArrayList 是一个动态数组，实现了 List 接口以及 list 相关的所有方法，它允许所有元素的插入，包括 null。另外，ArrayList 和 Vector 除了线程不同步之外，大致相等。

JDK8 : **默认构造空实例，大小是 0；只有在使用到时，才会通过grow方法创建一个大小为 10 的数组**

JDK7 : **默认构造初始容量为 10 的空列表**

在扩容上代码优点区别,但原理都是相同的,都是**原容量的1.5倍.**

##### 2.1 核心源码

```java
// @since   1.2
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable {
    
    /**
    * 1.属性
    * ArrayList所有的方法都是建立在 elementData 属性之上。
    */
    
	// 默认容量的大小
    private static final int DEFAULT_CAPACITY = 10;
    // 空数组常量：用于空实例
    private static final Object[] EMPTY_ELEMENTDATA = {};
    // 默认的空数组常量：用于默认大小空实例的共享空数组实例。
    // 把它从EMPTY_ELEMENTDATA数组中区分出来，以知道在添加第一个元素时容量需要增加多少。
    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};
    // 存放元素的数组，从这可以发现 ArrayList 的底层实现就是一个 Object数组
    transient Object[] elementData;
    // 数组中包含的元素个数
    private int size;
    // 数组的最大上限
    private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;


	/**
	* 2.构造方法
	* 默认情况下，elementData 是一个大小为 0 的空数组；
	* 当我们指定了初始大小的时候 elementData 的初始大小就变成了我们所指定的初始大小了
	*/
   
	// 实例化：按照指定大小分配空间
	public ArrayList(int initialCapacity) {
        if (initialCapacity > 0) {
            this.elementData = new Object[initialCapacity];	// 分配指定长度的数组空间
        } else if (initialCapacity == 0) {
            this.elementData = EMPTY_ELEMENTDATA;	// 空实例
        } else {
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        }
    }
	// 实例化：空实例，大小是 0；只有在使用到时，才会通过grow方法创建一个大小为 10 的数组
	public ArrayList() {
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;	// 空数组实例
    }
    // 形参为Collection实现类对象：可将set对象转为list对象
    public ArrayList(Collection<? extends E> c) {
        elementData = c.toArray();
        if ((size = elementData.length) != 0) {
            // c.toArray might (incorrectly) not return Object[] (see 6260652)
            if (elementData.getClass() != Object[].class)
                elementData = Arrays.copyOf(elementData, size, Object[].class);
        } else {
            // replace with empty array.
            this.elementData = EMPTY_ELEMENTDATA;
        }
    }
   
   
   /**
   * 3.get(int index)方法
   * 根据索引获取ArrayList中的元素
   */
   
   // 根据索引获取集合中元素
   public E get(int index) {
        rangeCheck(index);

        return elementData(index);
    }
    // 判断索引是否越界
    private void rangeCheck(int index) {
        if (index >= size)
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }
    // 直接通过数组下表获取元素
    @SuppressWarnings("unchecked")
    E elementData(int index) {
        return (E) elementData[index];
    }
   

	/**
    * 4.add方法
    * 在插入元素之前，先检查是否需要扩容，后把元素添加到数组中最后一个元素的后面
    */
	
	// 在数组末尾添加元素,复杂度为O(1)
	public boolean add(E e) {
        ensureCapacityInternal(size + 1);
        elementData[size++] = e;
        return true;
    }
    // 在索引为index处插入元素,复杂度为O(n)
    public void add(int index, E element) {
        rangeCheckForAdd(index);	// 判断索引是否越界或是否小于0

        ensureCapacityInternal(size + 1);
        // 调用一个 native 的复制方法，把 index 位置开始的元素都往后挪一位
        System.arraycopy(elementData, index, elementData, index + 1,
                         size - index);
        elementData[index] = element;
        size++;
    }
	// 确认是否是空数组
	private void ensureCapacityInternal(int minCapacity) {
		// 当 elementData 为空数组时，它会使用默认的大小(10)去扩容
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
            minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
        }

        ensureExplicitCapacity(minCapacity);
    }
	// 确认是否需要扩容
	private void ensureExplicitCapacity(int minCapacity) {
		// 快速失败的机制，通过记录 modCount 参数来实现
        modCount++;

        // 实际容量比默认长度大时，进行扩容
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
    }
    // ArrayList 每次扩容都是扩 1.5 倍
    // 然后调用 Arrays 类的 copyOf 方法，把元素重新拷贝到一个新的数组中去
    private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // 初始化，赋值为新的数组，长度为10
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
    // 比较minCapacity和 MAX_ARRAY_SIZE
    private static int hugeCapacity(int minCapacity) {
        if (minCapacity < 0) // overflow
            throw new OutOfMemoryError();
        return (minCapacity > MAX_ARRAY_SIZE) ?
            Integer.MAX_VALUE :
            MAX_ARRAY_SIZE;
    }


	/**
	* 5.set方法
	*/

	// 把下标为index的元素替换为element,复杂度为O(1)
	public E set(int index, E element) {
        rangeCheck(index);	// 判断索引是否越界

        E oldValue = elementData(index);
        elementData[index] = element;
        return oldValue;
    }
    
    
    /**
    * 6.remove方法
    */
    
    // 删除索引为index处的元素,复杂度为O(n)
    public E remove(int index) {
        rangeCheck(index);	// 判断索引是否越界

        modCount++;
        E oldValue = elementData(index);

        int numMoved = size - index - 1;
        if (numMoved > 0)	// 调用一个 native 的复制方法，把 index 位置开始的元素都往前挪一位
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--size] = null; // clear to let GC do its work

        return oldValue;
    }
    
    
    /**
    * 7.size方法
    */
    
    // 返回数组中元素的个数,时间复杂度为 O(1),返回的并不是数组的实际大小
    public int size() {
        return size;
    }
    
    
    /**
    * 8.indexOf方法和lastIndexOf方法
    */
    
    // 返回从前往后遍历查找第一个等于给定元素的值的下标,时间复杂度为O(n)
    public int indexOf(Object o) {
        if (o == null) {
            for (int i = 0; i < size; i++)
                if (elementData[i]==null)
                    return i;
        } else {	// 从前往后遍历
            for (int i = 0; i < size; i++)	// 通过遍历比较数组中每个元素的值来查找的
                if (o.equals(elementData[i]))
                    return i;
        }
        return -1;
    }
    // 返回从后往前遍历查找第一个等于给定元素的值的下标,时间复杂度为O(n)
    public int lastIndexOf(Object o) {
        if (o == null) {
            for (int i = size-1; i >= 0; i--)
                if (elementData[i]==null)
                    return i;
        } else {
            for (int i = size-1; i >= 0; i--)	// 从后往前遍历
                if (o.equals(elementData[i]))
                    return i;
        }
        return -1;
    }
    
    
    /**
     * 从列表中删除所有元素。 
     */
    public void clear() {
        modCount++;

        // 把数组中所有的元素的值设为null
        for (int i = 0; i < size; i++)
            elementData[i] = null;

        size = 0;
    }
    
    /**
     *以正确的顺序（从第一个到最后一个元素）返回一个包含此列表中所有元素的数组。 
     *返回的数组将是“安全的”，因为该列表不保留对它的引用。 （换句话说，这个方法必须分配一个新的数
     组）。
     *因此，调用者可以自由地修改返回的数组。 此方法充当基于阵列和基于集合的API之间的桥梁。
     */
    public Object[] toArray() {
        return Arrays.copyOf(elementData, size);
    }
    
    /**
     * 返回此ArrayList实例的浅拷贝。 （元素本身不被复制。） 
     */
    public Object clone() {
        try {
            ArrayList<?> v = (ArrayList<?>) super.clone();
            //Arrays.copyOf功能是实现数组的复制，返回复制后的数组。参数是被复制的数组和复制的长度
            v.elementData = Arrays.copyOf(elementData, size);
            v.modCount = 0;
            return v;
        } catch (CloneNotSupportedException e) {
            // 这不应该发生，因为我们是可以克隆的
            throw new InternalError(e);
        }
    }

}
```

##### 2.2 System.arraycopy()和Arrays.copyOf()

System.arraycopty():

```java
 	/**
     * 在此列表中的指定位置插入指定的元素。 
     *先调用 rangeCheckForAdd 对index进行界限检查；
     *然后调用 ensureCapacityInternal 方法保证capacity足够大；
     *再将从index开始之后的所有成员后移一个位置；将element插入index位置；最后size加1。
     */
    public void add(int index, E element) {
        rangeCheckForAdd(index);

        ensureCapacityInternal(size + 1);  // Increments modCount!!
        //arraycopy()方法实现数组自己复制自己
        //elementData:源数组;index:源数组中的起始位置;
        //elementData：目标数组；index + 1：目标数组中的起始位置； size - index：要复制的数组元素的数量
        System.arraycopy(elementData, index, elementData, index + 1, size - index);
        elementData[index] = element;
        size++;
    }

```

Arrays.copyOf():

```java
	/**
     *以正确的顺序（从第一个到最后一个元素）返回一个包含此列表中所有元素的数组。 
     *返回的数组将是“安全的”，因为该列表不保留对它的引用。（换句话说，这个方法必须分配一个新的数组）
     *因此，调用者可以自由地修改返回的数组。 此方法充当基于阵列和基于集合的API之间的桥梁。
     */
    public Object[] toArray() {
    //elementData：要复制的数组；size：要复制的长度
        return Arrays.copyOf(elementData, size);
    }
```

两个方法的源码:

```java
// Arrays类
public static <T,U> T[] copyOf(U[] original, int newLength, Class<? extends T[]> newType) {
  @SuppressWarnings("unchecked")
  T[] copy = ((Object)newType == (Object)Object[].class)	// 新建数组
    ? (T[]) new Object[newLength]
    : (T[]) Array.newInstance(newType.getComponentType(), newLength);
  System.arraycopy(original, 0, copy, 0,	// 拷贝
                   Math.min(original.length, newLength));
  return copy;
}

// System类
public static native void arraycopy(Object src,  int  srcPos,
                                        Object dest, int destPos,
                                        int length);
```

**联系：** 看两者源代码可以发现`copyOf()`内部调用了`System.arraycopy()`

**区别：**

-   arraycopy()需要目标数组，将原数组拷贝到你自己定义的数组里，而且可以选择拷贝的起点和长度以及放入新数组中的位置
-   copyOf()是系统自动在内部新建一个数组，并返回该数组。

##### 2.3 内部类

```java
	(1)private class Itr implements Iterator<E>  
    (2)private class ListItr extends Itr implements ListIterator<E>  
    (3)private class SubList extends AbstractList<E> implements RandomAccess  
    (4)static final class ArrayListSpliterator<E> implements Spliterator<E>  
```

ArrayList有四个内部类，

-   其中的**Itr是实现了Iterator接口**，同时重写了里面的**hasNext()**， **next()**， **remove()** 等方法；
-   其中的**ListItr** 继承 **Itr**，实现了**ListIterator接口**，同时重写了**hasPrevious()**， **nextIndex()**， **previousIndex()**， **previous()**， **set(E e)**， **add(E e)** 等方法;
-   **Iterator和ListIterator的区别:** ListIterator在Iterator的基础上增加了添加对象，修改对象，逆向遍历等方法，这些是Iterator不能实现的。

#### 3.Vector

Vector很多方法都跟 ArrayList 一样，只是多加了个 synchronized 来保证线程安全.部分源码:

```java
// @since   JDK1.0
public class Vector<E>
    extends AbstractList<E>
    implements List<E>, RandomAccess, Cloneable, java.io.Serializable{

	// 每次扩容只扩 capacityIncrement 个空间
	protected int capacityIncrement;
	
	//  Vector 的默认大小也是 10，而且它在初始化的时候就已经创建了数组了
	public Vector(int initialCapacity, int capacityIncrement) {
        super();
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        this.elementData = new Object[initialCapacity];
        this.capacityIncrement = capacityIncrement;
    }
    public Vector(int initialCapacity) {
        this(initialCapacity, 0);
    }
    public Vector() {
        this(10);
    }
    
    // newCapacity 默认情况下是两倍的 oldCapacity;
    // 当指定了 capacityIncrement 的值之后，newCapacity 变成了oldCapacity+capacityIncrement
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

}
```

#### 4.总结

JDK8 ArrayList 与 Vector 比较 :

-   ArrayList 创建时的大小为 0；当加入第一个元素时，进行第一次扩容时，默认容量大小为 10。
-   ArrayList 每次扩容都以当前数组大小的 1.5 倍去扩容。
-   Vector 创建时的默认大小为 10。
-   Vector 每次扩容都以当前数组大小的 2 倍去扩容。当指定了 capacityIncrement 之后，每次扩容仅在原先基础上增加 capacityIncrement 个单位空间。
-   ArrayList 和 Vector 的 add、get、size 方法的复杂度都为 O(1)，remove 方法的复杂度为 O(n)。
-   ArrayList 是非线程安全的，Vector 是线程安全的。

参考链接: [ArrayList 源码学习](https://snailclimb.gitee.io/javaguide/#/docs/java/collection/ArrayList)

### 二.LinkedList

LinkedList 是通过一个**双向链表**来实现的，它允许插入所有元素，包括 null，同时，它是**线程不同步**的。LinkedList底层的链表结构使它支持高效的**插入和删除操作**.如果想使LinkedList变成线程安全的，可以调用Collections中的静态方法 `List<T> synchronizedList(List<T> list)` 将其转为线程安全.

#### 1.内部结构

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191225180751305.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poaXhpbmd3dQ==,size_16,color_FFFFFF,t_70)

双向链表每个结点除了数据域之外，还有一个前指针和后指针，分别指向**前驱结点和后继结点**（如果有前驱/后继的话）。另外，双向链表还有一个 first 指针，指向头节点，和 last 指针，指向尾节点。

LinkedList类中的一个**内部私有类Node**:

```java
	// 双向链表节点: 每个节点的结构
	private static class Node<E> {	// 内部类：只有当前类需要使用，外面类不需要使用时
        E item;		// 泛型，obj对象数据
        Node<E> next;	// 后继节点;下一个元素节点地址：指向下一个元素的指针
        Node<E> prev;	// 前驱节点;上一个元素节点地址：指向上一个元素的指针

		// 元素节点分为三部分：前驱节点，本节点的值，后继结点
        Node(Node<E> prev, E element, Node<E> next) {
            this.item = element;
            this.next = next;
            this.prev = prev;
        }
    }
```

#### 2.属性和构造器

```java
// @since 1.2
public class LinkedList<E>
    extends AbstractSequentialList<E>
    implements List<E>, Deque<E>, Cloneable, java.io.Serializable{
    
    //链表的节点个数//指向头节点的指针
    transient int size = 0;
    //指向头节点的指针
    transient Node<E> first;
    //指向尾节点的指针
    transient Node<E> last;
    
    // 空构造方法
    public LinkedList() {
    }
    // 用已有的集合创建链表的构造方法
    public LinkedList(Collection<? extends E> c) {
        this();
        addAll(c);
    }
}
```

#### 3.加节点

在表头或表尾进行插入元素只需要 O(1) 的时间，而在指定位置插入元素则需要先遍历一下链表，所以复杂度为 O(n)。

##### 3.1 表头添加元素

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191225180803951.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poaXhpbmd3dQ==,size_16,color_FFFFFF,t_70)

**addFirst(E e), push(E e), offerFirst(E e)**

```java
	public void addFirst(E e) {	// 在链表头部添加元素节点
        linkFirst(e);
    }
	public void push(E e) {	// 在链表头部添加元素节点
        addFirst(e);
    }
	public boolean offerFirst(E e) {
        addFirst(e);
        return true;
    }

	private void linkFirst(E e) {
        final Node<E> f = first;
        //当前节点的前驱指向 null，后继指针原来的头节点
        final Node<E> newNode = new Node<>(null, e, f);
        first = newNode;	//头指针指向新的头节点
        if (f == null)	//如果链表为空，last节点也指向该节点
            last = newNode;
        else	//否则，将头节点的前驱指针指向新节点，也就是指向前一个元素
            f.prev = newNode;
        size++;
        modCount++;
    }
```

##### 3.2 表尾添加元素

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191225180824621.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poaXhpbmd3dQ==,size_16,color_FFFFFF,t_70)

**add(E e), addLast(E e), offer(E e), offerLast(E e)**

```java
	public boolean add(E e) {	// 在链表结尾添加元素节点
        linkLast(e);
        return true;
    }	
	public void addLast(E e) {	// 在链表结尾添加元素节点
        linkLast(e);
    }
	public boolean offer(E e) {
        return add(e);
    }
	public boolean offerLast(E e) {
        addLast(e);
        return true;
    }

	void linkLast(E e) {	// 加为最后节点
        final Node<E> l = last;	// 声明临时节点用来存储last节点
        final Node<E> newNode = new Node<>(l, e, null);	// 新节点指向原先last节点
        last = newNode;		// 新节点作为最后一个节点
        if (l == null)
            first = newNode;
        else
            l.next = newNode;	// 原先last节点指向新节点(last节点)
        size++;
        modCount++;
    }
```

##### 3.3 指定位置添加元素

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191225180836833.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poaXhpbmd3dQ==,size_16,color_FFFFFF,t_70)

**add(int index, E element)**

```java
	public void add(int index, E element) {	//在指定位置添加元素节点
        checkPositionIndex(index);	//检查索引是否处于[0-size]之间

        if (index == size)	//添加在链表尾部
            linkLast(element);
        else	//添加在链表中间
            linkBefore(element, node(index));	//一个插入节点的值，一个指定的node
    }

	void linkBefore(E e, Node<E> succ) {
        // assert succ != null;
        final Node<E> pred = succ.prev;	//指定节点的前驱
        //当前节点的前驱为指点节点的前驱，后继为指定的节点
        final Node<E> newNode = new Node<>(pred, e, succ);
        //更新指定节点的前驱为当前节点
        succ.prev = newNode;
        //更新前驱节点的后继
        if (pred == null)
            first = newNode;
        else
            pred.next = newNode;
        size++;
        modCount++;
    }
```

#### 4.删除节点

##### 4.1 删除头节点

**remove()** ,**removeFirst(), pop()**

```java
	public E pop() {
        return removeFirst();
    }
	public E remove() {
        return removeFirst();
    }
	public E removeFirst() {
        final Node<E> f = first;
        if (f == null)
            throw new NoSuchElementException();
        return unlinkFirst(f);
    }

	//删除表头节点，返回表头元素的值
	private E unlinkFirst(Node<E> f) {
        // assert f == first && f != null;
        final E element = f.item;
        final Node<E> next = f.next;
        f.item = null;
        f.next = null; // help GC
        first = next;	//头指针指向后一个节点
        if (next == null)
            last = null;
        else
            next.prev = null;	//新头节点的前驱为 null
        size--;
        modCount++;
        return element;
    }
```

##### 4.2 删除尾节点

**removeLast(), pollLast()**

**区别**: removeLast()在链表为空时将抛出NoSuchElementException，而pollLast()方法返回null。

```java
	public E removeLast() {
        final Node<E> l = last;
        if (l == null)
            throw new NoSuchElementException();
        return unlinkLast(l);
    }
	public E pollLast() {
        final Node<E> l = last;
        return (l == null) ? null : unlinkLast(l);
    }

	//删除表尾节点，返回表尾元素的值
	private E unlinkLast(Node<E> l) {
        // assert l == last && l != null;
        final E element = l.item;
        final Node<E> prev = l.prev;
        l.item = null;
        l.prev = null; // help GC
        last = prev;	//尾指针指向前一个节点
        if (prev == null)
            first = null;
        else
            prev.next = null;	//新尾节点的后继为 null
        size--;
        modCount++;
        return element;
    }
```

##### 4.3 删除指定节点

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191225180847708.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poaXhpbmd3dQ==,size_16,color_FFFFFF,t_70)

**remove(Object o), remove(int index)**

```java
	public boolean remove(Object o) { 	// 删除指定元素:只会删除一个匹配的对象
        if (o == null) {	//如果删除对象为null 
            for (Node<E> x = first; x != null; x = x.next) {	//从头开始遍历 
                if (x.item == null) {	//找到元素  
                    unlink(x);	//从链表中移除找到的元素
                    return true;
                }
            }
        } else {
            for (Node<E> x = first; x != null; x = x.next) {//从头开始遍历
                if (o.equals(x.item)) { //找到元素  
                    unlink(x);//从链表中移除找到的元素
                    return true;
                }
            }
        }
        return false;
    }
	public E remove(int index) {	//删除指定位置的元素
        //检查index范围
        checkElementIndex(index);
        //将节点删除
        return unlink(node(index));
    }

	//删除指定节点，返回指定元素的值
	E unlink(Node<E> x) {
        // assert x != null;
        final E element = x.item;
        final Node<E> next = x.next;// 当前节点的后继
        final Node<E> prev = x.prev;// 当前节点的前驱
		//删除前驱指针
        if (prev == null) {
            first = next;	//如果删除的节点是头节点,令头节点指向该节点的后继节点
        } else {
            prev.next = next;	//更新前驱节点的后继为当前节点的后继
            x.prev = null;
        }
		//删除后继指针
        if (next == null) {
            last = prev;	//如果删除的节点是尾节点,令尾节点指向该节点的前驱节点
        } else {
            next.prev = prev;	//更新后继节点的前驱为当前节点的前驱
            x.next = null;
        }

        x.item = null;
        size--;
        modCount++;
        return element;
    }
```

#### 5.获取节点

##### 5.1 获取头节点(index=0)

**getFirst(), element(), peek(), peekFirst()**

**区别：** 在于对链表为空时的处理，是抛出异常还是返回null，其中**getFirst()** 和**element()** 方法将会在链表为空时，抛出NoSuchElementException异常

```java
	public E getFirst() {
        final Node<E> f = first;
        if (f == null)
            throw new NoSuchElementException();
        return f.item;
    }
	public E element() {
        return getFirst();
    }
	public E peek() {
        final Node<E> f = first;
        return (f == null) ? null : f.item;
    }
	public E peekFirst() {
        final Node<E> f = first;
        return (f == null) ? null : f.item;
     }
```

##### 5.2 获取尾节点(index=-1)

**getLast(), peekLast()**

**区别：** **getLast()** 方法在链表为空时，会抛出**NoSuchElementException**，而**peekLast()** 则不会.

```java
	public E getLast() {
        final Node<E> l = last;
        if (l == null)
            throw new NoSuchElementException();
        return l.item;
    }
 	public E peekLast() {
        final Node<E> l = last;
        return (l == null) ? null : l.item;
    }
```

##### 5.3 获取指定节点

**get(int index)**

```java
	public E get(int index) {
        //检查index范围是否在size之内
        checkElementIndex(index);
        //调用Node(index)去找到index对应的node然后返回它的值
        return node(index).item;
    }

	//获取指定下标的元素
	Node<E> node(int index) {	//查找index对应的node
        // assert isElementIndex(index);
      //根据下标是否超过链表长度的一半，来选择从头部开始遍历还是从尾部开始遍历
        if (index < (size >> 1)) {
            Node<E> x = first;
            for (int i = 0; i < index; i++)
                x = x.next;
            return x;
        } else {
            Node<E> x = last;
            for (int i = size - 1; i > index; i--)
                x = x.prev;
            return x;
        }
    }
```

#### 6.其它方法

**set(int index, E element), size(), contains(Object o), indexOf(Object o), lastIndexOf(Object o)**

```java
	public E set(int index, E element) {	//将此列表中指定位置的元素替换为指定的元素
        checkElementIndex(index);
        Node<E> x = node(index);
        E oldVal = x.item;
        x.item = element;
        return oldVal;
    }
	public int size() {	//返回此列表的元素个数
        return size;
    }
	public boolean contains(Object o) {	//检查对象o是否存在于链表中
        return indexOf(o) != -1;
    }
	public int indexOf(Object o) {	//从头遍历找该对象的索引
        int index = 0;
        if (o == null) {
            for (Node<E> x = first; x != null; x = x.next) {
                if (x.item == null)
                    return index;
                index++;
            }
        } else {
            for (Node<E> x = first; x != null; x = x.next) {
                if (o.equals(x.item))
                    return index;
                index++;
            }
        }
        return -1;
    }
	public int lastIndexOf(Object o) {	//从尾遍历找该对象的索引
        int index = size;
        if (o == null) {
            for (Node<E> x = last; x != null; x = x.prev) {
                index--;
                if (x.item == null)
                    return index;
            }
        } else {
            for (Node<E> x = last; x != null; x = x.prev) {
                index--;
                if (o.equals(x.item))
                    return index;
            }
        }
        return -1;
    }
```

参考链接: [LinkedList 源码学习](https://snailclimb.gitee.io/javaguide/#/docs/java/collection/LinkedList)

### 三.HashMap

HashMap源码学习可参考下面文章...



参考链接：[HashMap(JDK1.8)源码学习](https://snailclimb.gitee.io/javaguide/#/docs/java/collection/HashMap)



