# ArrayList源码解析

## 初始化及构造函数

```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{
    private static final long serialVersionUID = 8683452581122892189L;

    /**
     * 默认初始化的容量
     */
    private static final int DEFAULT_CAPACITY = 10;

    /**
     * 初始化一个空实例的共享空数组实例。
     */
    private static final Object[] EMPTY_ELEMENTDATA = {};

    /**
     *用于默认大小的空实例的共享空数组实例。我们将此与 empty _ elemtdata 区分开来, 以了解	 	  *添加分离第一元素时充气的程度
     */
    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

    /**
	 * 元素对象数据
     */
    transient Object[] elementData; // non-private to simplify nested class access

    /**
     * 当前 集合 的大小
     *
     * @serial
     */
    private int size;

    /**
     * 创建 ArrayList 的时候直接给元素对象开辟一个指定的容量大小
     * 否则 直接初始化一个空的元素对象
     *
     * @param 
     * @throws 
     *        
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

    /**
     * 如果直接 new 一个空的构造函数的 ArrayList 内部直接创建一个默认空的对象数组
     */
    public ArrayList() {
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
    }

    /**
     *
     * @param c 外部直接传入一个集合
     * @throws NullPointerException if the specified collection is null
     */
    public ArrayList(Collection<? extends E> c) {
    	//将外部传入进来的集合数据转化为数组赋值给 元素对象数组
        elementData = c.toArray();
        //更新集合大小
        if ((size = elementData.length) != 0) {
            // 判断引用的数据类型，并将引用转换成 Object 数组引用
            if (elementData.getClass() != Object[].class)
                elementData = Arrays.copyOf(elementData, size, Object[].class);
        } else {
            // 如果传入进来的是一个空的集合，那么直接创建一个空数组
            this.elementData = EMPTY_ELEMENTDATA;
        }
    }


```



## 添加

```java

	public boolean add(E e) {
    	// 检查是否需要扩容
        ensureCapacityInternal(size + 1);  // Increments modCount!!
    	// 扩容完毕后，将元素存入末尾，然后size++    
    	elementData[size++] = e;
        return true;
    }

   
    public void add(int index, E element) {
        // 查看插入位置是否超过集合最大范围
        if (index > size || index < 0)
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
		// 检查是否扩容
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        // 扩容完毕后，将数组进行copy，并插入index位置
        System.arraycopy(elementData, index, elementData, index + 1,
                         size - index);
        elementData[index] = element;
        size++;
    }
```

添加的原理很简单，先判断添加时候数组是否已经满了，或者判断要添加的index是否超过范围。满了则进行扩容，超过范围抛出异常。

将对应的元素进行添加，添加到末尾，就直接size++，如果是指定位置index，就将index位置的元素替换为该元素



## 删除

```java
	public E remove(int index) {
        // 查看要删除位置是否超过范围
        if (index > size || index < 0)
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
        
		//在使用迭代器遍历的时候，用来检查列表中的元素是否发生结构性变化（列表元素数量发生改变）了，主要在多线程环境下需要使用，防止一个线程正在迭代遍历，另一个线程修改了这个列表的结构。
        modCount++;
        E oldValue = elementData(index);
		// 计算出复制数组时候要移动的位置
        int numMoved = size - index - 1;
        if (numMoved > 0)
            // 数组复制，覆盖要删除的元素
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        
        elementData[--size] = null; // clear to let GC do its work

        return oldValue;
    }


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

modcount 作用：在使用迭代器遍历的时候，用来检查列表中的元素是否发生结构性变化（列表元素数量发生改变）了，主要在多线程环境下需要使用，防止一个线程正在迭代遍历，另一个线程修改了这个列表的结构。



## 扩容

```java
//下面是ArrayList的扩容机制
//ArrayList的扩容机制提高了性能，如果每次只扩充一个，
//那么频繁的插入会导致频繁的拷贝，降低性能，而ArrayList的扩容机制避免了这种情况。
    /**
     * 如有必要，增加此ArrayList实例的容量，以确保它至少能容纳元素的数量
     * @param   minCapacity   所需的最小容量
     */
    private void ensureCapacityInternal(int minCapacity) {
        // 如果当前数组为空，最小容量就是默认容量和最小容量的最大值
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
            minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
        }
    	ensureExplicitCapacity(minCapacity);
	}

// 传入最小容量，操作次数++，
private void ensureExplicitCapacity(int minCapacity) {
    modCount++;
    // overflow-conscious code
    // 如果最小容量大于数组的长度，就进行扩容。
    if (minCapacity - elementData.length > 0)
        grow(minCapacity);
}

private void grow(int minCapacity) {
    // overflow-conscious code
    int oldCapacity = elementData.length;
    // 将新的容量变为原来容量的1.5倍
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    // 如果扩容过后，还是小于最小的容量，那么新容量就是最小的容量
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    // 如果新的容量大于定义的数组最大容量，如果不是异常，那么就将新的容量变为Integer.MAX_VALUE
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    // minCapacity is usually close to size, so this is a win:
    elementData = Arrays.copyOf(elementData, newCapacity);
}
```

通过源码可以看出来，ArrayList通过内部数组（动态数组）的数据结构实现。每次添加元素，都是先判断是否达到容量上限，如果达到，则进行扩容。扩容一般扩容到原来大小的1.5倍。

删除方法实现思路就是将删除位置进行覆盖。将index位置之后的所有数据全体前移一位。



## ArrayList中的序列化

https://my.oschina.net/u/3492343/blog/2999587

* 为什么用 transient 关键字？

  由于ArrayList底层是动态数组，会存在有些元素为null的情况，数组并不是都是满的。所以为了避免因为空元素而导致的空间浪费。使用了transient 关键字

* 如何实现数组的序列化？

  ArrayList中定义了两个方法：`readObject()` 和 `writeObject()`, **`writeObject`**方法把**`elementData`**数组中的元素遍历的保存到输出流（**ObjectOutputStream**）中。**`readObject`**方法从输入流（**ObjectInputStream**）中读出对象并保存赋值到**`elementData`**数组中。

* 如果一个类中包含writeObject 和 readObject 方法，那么这两个方法是怎么被调用的?

  在使用ObjectOutputStream的writeObject方法和ObjectInputStream的readObject方法时，会通过反射的方式调用。

```java
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

    /**
     * Reconstitute the <tt>ArrayList</tt> instance from a stream (that is,
     * deserialize it).
     */
    private void readObject(java.io.ObjectInputStream s)
        throws java.io.IOException, ClassNotFoundException {
        elementData = EMPTY_ELEMENTDATA;

        // Read in size, and any hidden stuff
        s.defaultReadObject();

        // Read in capacity
        s.readInt(); // ignored

        if (size > 0) {
            // be like clone(), allocate array based upon size not capacity
            int capacity = calculateCapacity(elementData, size);
            SharedSecrets.getJavaOISAccess().checkArray(s, Object[].class, capacity);
            ensureCapacityInternal(size);

            Object[] a = elementData;
            // Read in all elements in the proper order.
            for (int i=0; i<size; i++) {
                a[i] = s.readObject();
            }
        }
    }

```



## 

## fail-fast

modCount 用来记录 ArrayList 结构发生变化的次数。结构发生变化是指添加或者删除至少一个元素的所有操作，或者是调整内部数组的大小，仅仅只是设置元素的值不算结构发生变化。

在进行序列化或者迭代等操作时，需要比较操作前后 modCount 是否改变，如果改变了需要抛出 ConcurrentModificationException。



一：快速失败（fail—fast）

在用迭代器遍历一个集合对象时，如果遍历过程中对集合对象的内容进行了修改（增加、删除、修改），则会抛出Concurrent Modification Exception。

原理：迭代器在遍历时直接访问集合中的内容，并且在遍历过程中使用一个 modCount 变量。集合在被遍历期间如果内容发生变化，就会改变modCount的值。每当迭代器使用hashNext()/next()遍历下一个元素之前，都会检测modCount变量是否为expectedmodCount值，是的话就返回遍历；否则抛出异常，终止遍历。

注意：这里异常的抛出条件是检测到 modCount！=expectedmodCount 这个条件。如果集合发生变化时修改modCount值刚好又设置为了expectedmodCount值，则异常不会抛出。因此，不能依赖于这个异常是否抛出而进行并发操作的编程，这个异常只建议用于检测并发修改的bug。

场景：java.util包下的集合类都是快速失败的，不能在多线程下发生并发修改（迭代过程中被修改）。

二：安全失败（fail—safe）

采用安全失败机制的集合容器，在遍历时不是直接在集合内容上访问的，而是先复制原有集合内容，在拷贝的集合上进行遍历。

原理：由于迭代时是对原集合的拷贝进行遍历，所以在遍历过程中对原集合所作的修改并不能被迭代器检测到，所以不会触发Concurrent Modification Exception。

缺点：基于拷贝内容的优点是避免了Concurrent Modification Exception，但同样地，迭代器并不能访问到修改后的内容，即：迭代器遍历的是开始遍历那一刻拿到的集合拷贝，在遍历期间原集合发生的修改迭代器是不知道的。

场景：java.util.concurrent包下的容器都是安全失败，可以在多线程下并发使用，并发修改。

**总结：快速失败可以看做是一种在多线程环境下防止出现并发修改的预防策略，直接通过抛异常来告诉开发者不要这样做。而安全失败虽然不抛异常，但是在多个线程中修改集合，开发者同样要注意多线程带来的问题。**







## 源码中的拷贝

### System.arraycopy()和Arrays.copyOf()方法

``add()``方法中的扩容方法使用的是``Arrays.copyOf()``方法。`addAll()`方法中使用的是`System.arraycopy()`方法。

两者都是浅拷贝，都是创建一个新的数组，然后将原数组中的元素引用也放到了新数组中，并没有创建新的元素。

不同之处为

1. `arrayCopy()`需要目标数组，将原数组拷贝到定义的数组中，而且可以选择拷贝的起点和放入数组中的位置
2. `copyOf()`就是自动创建一个数组，并返回

## 特点

顺序存储，能够进行快速取值，但是添加或者删除性能较差

