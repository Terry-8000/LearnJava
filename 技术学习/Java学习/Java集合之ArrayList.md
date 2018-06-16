# Java集合之ArrayList - 吃透增删查改

**从源码看初始化以及增删查改，学习ArrayList。**

先来看下ArrayList定义的几个属性：

```java 
private static final int DEFAULT_CAPACITY = 10;
private static final Object[] EMPTY_ELEMENTDATA = {};
private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};
transient Object[] elementData;  // 保存对象
private int size; // ArrayList的长度
```

从这里可以看到ArrayList内部使用数组实现的。

## 一. 初始化

**1. ArrayList()**

无参的构造器：

```java
	/**
     * Constructs an empty list with an initial capacity of ten.
     */
    public ArrayList() {
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
    }
```

可以看到这个构造器初始化了一个空数组。这里有个疑问，就是注释明明说是构造了一个长度为10的数组，其实这是在添加第一个元素的时候才进行的扩充。

**2. ArrayList(int initialCapacity)**

指定长度的构造器：

```java
	/**
     * Constructs an empty list with the specified initial capacity.
     *
     * @param  initialCapacity  the initial capacity of the list
     * @throws IllegalArgumentException if the specified initial capacity
     *         is negative
     */
    public ArrayList(int initialCapacity) {
        if (initialCapacity > 0) {
            this.elementData = new Object[initialCapacity];
        } else if (initialCapacity == 0) {
            this.elementData = EMPTY_ELEMENTDATA;
        } else {
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        }
    }
```

这个构造器显式的指明了数组的长度，其实如果小于10的话，在添加第一个元素的时候还是会扩充到长度为10的数组。

## 二. 增加元素

**1. add(E e)**

```java
	/**
     * Appends the specified element to the end of this list.
     *
     * @param e element to be appended to this list
     * @return <tt>true</tt> (as specified by {@link Collection#add})
     * 添加指定的元素到List的末尾
     */
    public boolean add(E e) {
        ensureCapacityInternal(size + 1);  // 扩充数组长度
        elementData[size++] = e;           // 最后一个赋值为e
        return true;
    }
```

那么它是如何扩充数组长度的呢？追踪代码来到**grow(int minCapacity)**方法:

```java
/*
* 可以看到它是通过Arrays.copyOf方法将原数组拷贝到了一个数组长度为newCapacity的新数组里面
*/
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
```

**2. add(int index, E element) **

```java
	/**
     * 在指定的位置添加一个元素
     */
    public void add(int index, E element) {
        rangeCheckForAdd(index); // 检查index是否合法

        ensureCapacityInternal(size + 1);  // 扩充数组长度
        System.arraycopy(elementData, index, elementData, index + 1,
                         size - index);  // 通过拷贝返回一个新数组
        elementData[index] = element;
        size++;
    }
```

这里用到了System类的arraycopy方法，它的用法如下：

System. arraycopy(Object src,  int  srcPos, Object dest, int destPos,int length)

src：原数组；

srcPos：源数组要复制的起始位置；

dest：目标数组；

destPos：目标数组放置的起始位置；

length：复制的长度

这个方法也可以实现自身的复制。上面的就是利用了自身赋值的特性进行数组的拷贝。相当于将index后面的所有元素后移了一位，将新元素插入到index位置。

## 三. 删除元素

**1. remove(int index)**

```java
 	/*
 	* 和add(int index, E element)类似，使用System.arraycopy进行自身拷贝，相当于将index后面的元素
 	* 像前移动了一位，覆盖掉需要删除的元素，将最后一个元素置位null，等待JVM自动回收
 	*/
	public E remove(int index) {
        rangeCheck(index);

        modCount++;
        E oldValue = elementData(index);

        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--size] = null; // clear to let GC do its work

        return oldValue;
    }
```

**2. remove(Object o)**

```java
	/*
	* 先通过for循环找到o所在的位置，再通过fastRemove(index)移除，实现方法和remove(int index)一样
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
```

## 四. 查找元素

```java
	/*
	* 查找元素就比较简单了，直接通过数组的下角标进行返回
	*/
	public E get(int index) {
        rangeCheck(index);

        return elementData(index);
    }
```

## 五. 修改元素

```java
	/*
	* 修改元素也比较简单，直接通过数组的下角标进行赋值
	*/
	public E set(int index, E element) {
        rangeCheck(index);

        E oldValue = elementData(index);
        elementData[index] = element;
        return oldValue;
    }
```

## 总结

从源码可以看到，ArrayList以数组方式实现，查找和修改的性能比较好，增加和删除的性能就比较差了。

