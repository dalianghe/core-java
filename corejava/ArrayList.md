# <center>ArrayList</center>

### - 继承关系
![](http://i.imgur.com/7ZVqBJT.png)

### - 原理分析

ArrayList实现List接口，底层以数组结构实现，默认构造不初始化数组大小，而是在第一次add操作时进行底层数组容量的初始化设置，即通过拷贝数组的方式（Arrays.copyOf()方法）设置Object[] elementData的值（默认大小为10）。填充超过容量大小时，会再次进行以拷贝数据方式自动扩容。因在add或remove（通过System.arraycopy()方法）都伴随着数据拷贝动作，故添加、删除会影响系统性能。
在使用ArrayList时最好调用ArrayList(int initialCapacity)构造设置底层数组初始值。

### - 数据结构
![](http://i.imgur.com/EEldO03.png)

> List<String> list = new ArrayList<String>();  
> list.add("aaa");  
> list.add("bbb");  
> list.add("ccc");  
> System.out.println(list.get(0)); 输出：aaa  
> list.remove(0);  
> System.out.println(list.get(0)); 输出：bbb  

### - 特点

![](http://i.imgur.com/qkiriqw.png)

### - 优缺点
优点  

	1、ArrayList底层以数组实现，是一种随机访问模式，再加上它实现了RandomAccess接口，因此查找（get）非常快；
	2、ArrayList在顺序添加元素时非常方便，只是在数组末端添加一个元素而已。
缺点  

	1、插入元素时，涉及一次元素复制，如果要复制的元素很多，性能消耗较大；
	2、删除元素时，涉及一次元素复制，如果要复制的元素很多，性能消耗较大

### - 源码分析

类

	public class ArrayList<E> extends AbstractList<E>
		implements List<E>, RandomAccess, Cloneable, java.io.Serializable
	ArrayList实现List接口

属性

	// 缺省初始容量
	private static final int DEFAULT_CAPACITY = 10;
	// 使用空数组初始化数组
	private static final Object[] EMPTY_ELEMENTDATA = {};
	// 使用默认初始容量（DEFAULT_CAPACITY）实例化数组
	private static final OBject[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};
	// 数组元素存储区，ArrayList的容量是此属性的length。任何的空数组在第一次调用add方法时被初始化为DEFAULT_CAPACITY大小。
	//被修饰为transient用于序列化ArrayList时，只序列化已有的elementData数组中的元素，不对elementData本身序列化（ArrayList重写writeObject方法实现）
	transient Object[] elementData;
	// ArrayList大小，外部通过调用size()方法使用此值
	private int size;

构造

	// 构造默认初始容量DEFAULT_CAPACITY空列表
	public ArrayList(){}
	// 构造指定初始容量的空列表
	public ArrayList(int initialCapacity){}
	// 构造包含指定collection的元素的列表
	public ArrayList(Collection<? extends E> c){}

方法

	添加
	/**
	* 将指定的元素添加到列表的尾部
	*/
	public boolean add(E e){
		// 检查是否需要扩容
		ensureCapacityInternal(size + 1);  // Increments modCount!!
		// 赋值
        elementData[size++] = e;
        return true;
	}
	private void ensureCapacityInternal(int minCapacity) {
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
			// 比较默认的容量10和传入的容量，返回大点的数
            minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
        }
        ensureExplicitCapacity(minCapacity);
    }
	private void ensureExplicitCapacity(int minCapacity) {
		// 记录数组修改次数
        modCount++;
        // overflow-conscious code
		// 添加的元素超过数组容量时进行扩容
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);// 扩容
    }
	/**
	* 扩容
	*/
	private void grow(int minCapacity) {
        // overflow-conscious code
		// 记录当前list的容量
        int oldCapacity = elementData.length;
		// 扩容为原来的1.5倍
        int newCapacity = oldCapacity + (oldCapacity >> 1);
		// 如果扩展1.5倍还不能满足，直接扩展为需求值
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
	/**
	* 将指定的元素添加到制定的列表位置上。
	*/
	public boolean add(int index,E element){
		rangeCheckForAdd(index);
        ensureCapacityInternal(size + 1);  // Increments modCount!!
		//移动index后的所有元素，index越小需移动的元素越多，新能越差
        System.arraycopy(elementData, index, elementData, index + 1, size - index);
        elementData[index] = element;
        size++;
	}
	/**
	* 删除制定索引元素
	*/
	public E remove(int index) {
        rangeCheck(index);

        modCount++;
        E oldValue = elementData(index);

        int numMoved = size - index - 1;
        if (numMoved > 0)
 			// 将删除索引后的元素前移
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--size] = null; // clear to let GC do its work

        return oldValue;
    }
	/**
	* 删除制定元素
	*/
	public boolean remove(Object o) {
        if (o == null) {
            for (int index = 0; index < size; index++)
                if (elementData[index] == null) {
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
	/**
	* 清空列表中的所有元素，调用此方法后列表大小为0
	*/
	public void clear() {
        modCount++;

        // clear to let GC do its work
        for (int i = 0; i < size; i++)
            elementData[i] = null;

        size = 0;
    }

	/**
	* 序列化数组，对elementData数组中的元素，只对元素进行序列化操作
	*/
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





