# LinkedList源码解析

## 简介

内部为双向链表，能够进行快速的插入和删除。但是内部的查找效率较低。LinkedList实现了List接口和Deque接口，就拥有了队列的性值。线程不安全的，可以通过调用静态类Collection中的synchronizedList方法

## 内部类Node

```java
  private static class Node<E> {
        E item; // 节点值
        Node<E> next; // 后续节点
        Node<E> prev; // 前驱节点

        Node(Node<E> prev, E element, Node<E> next) {
            this.item = element;
            this.next = next;
            this.prev = prev;
        }
    }
```



![Node](https://camo.githubusercontent.com/df94e406b94b97fe2f094082e3977f30121448f6/68747470733a2f2f757365722d676f6c642d63646e2e786974752e696f2f323031382f332f31392f313632336533363366653034353062303f773d36303026683d34383126663d6a70656726733d3138353032)

## 构造方法

```java
	public LinkedList() {
    }
	// 使用已知的集合进行创建
    public LinkedList(Collection<? extends E> c) {
        this();
        addAll(c);
    }
```

## 添加方法

### add()方法

**`add(E e)`**：将元素添加到链表尾部

```java
	public boolean add(E e) {
        linkLast(e);
        return true;
    }
```

```java
	// 将元素添加到链表尾部
	void linkLast(E e) {
        // 首先获取当前链表的尾部
        final Node<E> l = last;
        // 创建节点值为e的Node，其前驱节点为l，后续节点为null
        final Node<E> newNode = new Node<>(l, e, null);
        // 然后让新的节点作为最后的链表的最后的节点
        last = newNode;
        // 如果前驱节点为空，说明链表只有当前节点一个，那么新节点也作为头节点
        if (l == null)
            first = newNode;
        else
            // 不然就是将前驱节点的后继节点作为新节点
            l.next = newNode;
        // 记录链表的大小和操作次数
        size++;
        modCount++;
    }
```

**`add(int index, E element)`**：将元素添加到指定位置

```java
	public void add(int index, E element) {
        // 查看index是否符合条件，是否在链表中
        checkPositionIndex(index); 

        // 如果index == size，则添加到链表的尾部
        if (index == size)
            linkLast(element);
        // 否则则正常添加到指定位置
        else
            linkBefore(element, node(index));
    }
```

这里只看l``inkBefore(E element, Node<E> succ)``方法 和其参数 ``node(index)``

```java
	// node(int index)方法，传入index，返回指定位置的节点 
	Node<E> node(int index) {
        // assert isElementIndex(index);
		// 通过查看index是在链表前一半还是后一半，决定是通过头节点查找还是通过尾节点查找
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

	// linkBefore(E e, Node<E> succ) 方法，将元素插入到指定节点前驱节点上
	void linkBefore(E e, Node<E> succ) {
        // assert succ != null;
        // 找到指定节点的前驱节点
        final Node<E> pred = succ.prev;
        // 创建含有元素值的节点
        final Node<E> newNode = new Node<>(pred, e, succ);
        // 连接节点，并将size++，操作次数++
        succ.prev = newNode;
        if (pred == null)
            first = newNode;
        else
            pred.next = newNode;
        size++;
        modCount++;
    }
```

**`addAll(Collection<? extend E> e)`**：将集合插入到链表尾部

```java
	public boolean addAll(Collection<? extends E> c) {
        return addAll(size, c);
    }
```

**`addAll(int index, Collection<? extend E> e)`**：将集合插入到指定位置

```java
	public boolean addAll(int index, Collection<? extends E> c) {
        // 首先检查index是否合法
        checkPositionIndex(index);
   
        Object[] a = c.toArray();
        int numNew = a.length;
        if (numNew == 0)
            return false;

        // 找到要插入的前后节点，pred就是前驱节点、succ是后继节点
        Node<E> pred, succ;
        if (index == size) {
            succ = null;
            pred = last;
        } else {
            succ = node(index);
            pred = succ.prev;
        }

        // 进行插入
        for (Object o : a) {
            @SuppressWarnings("unchecked") E e = (E) o;
            Node<E> newNode = new Node<>(pred, e, null);
           	// 判断先序节点是否为空
            if (pred == null)
                first = newNode;
            else
                pred.next = newNode;
            pred = newNode;
        }

        // 进行最后的连接
        if (succ == null) {
            last = pred;
        } else {
            pred.next = succ;
            succ.prev = pred;
        }

        size += numNew;
        modCount++;
        return true;
    }

```

## 获取数据

`get(int index)`获取指定位置的元素

```java
	public E get(int index) {
        checkElementIndex(index);
        return node(index).item;
    }
```

`getFirst() ` 获取头节点

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

`getLast()` 获取尾节点

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

### 根据对象获取索引index

`int indexOf(Object o)`从头开始遍历查找

```java
	// 判断对象是否为空，然后进行遍历	
	public int indexOf(Object o) {
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
```

`int lastIndexOf(Object o)`从尾部开始遍历查找

`contains(Object o)`：检查对象o是否存在于链表中，使用的就是查找方法



## 删除方法

**remove()** ,**removeFirst(),pop():** 删除头节点

```
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
```

**removeLast(),pollLast():** 删除尾节点

```
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
```

**区别：** removeLast()在链表为空时将抛出NoSuchElementException，而pollLast()方法返回null。

**remove(Object o):** 删除指定元素

```
public boolean remove(Object o) {
        //如果删除对象为null
        if (o == null) {
            //从头开始遍历
            for (Node<E> x = first; x != null; x = x.next) {
                //找到元素
                if (x.item == null) {
                   //从链表中移除找到的元素
                    unlink(x);
                    return true;
                }
            }
        } else {
            //从头开始遍历
            for (Node<E> x = first; x != null; x = x.next) {
                //找到元素
                if (o.equals(x.item)) {
                    //从链表中移除找到的元素
                    unlink(x);
                    return true;
                }
            }
        }
        return false;
    }
```

当删除指定对象时，只需调用remove(Object o)即可，不过该方法一次只会删除一个匹配的对象，如果删除了匹配对象，返回true，否则false。

unlink(Node x) 方法：

```
E unlink(Node<E> x) {
        // assert x != null;
        final E element = x.item;
        final Node<E> next = x.next;//得到后继节点
        final Node<E> prev = x.prev;//得到前驱节点

        //删除前驱指针
        if (prev == null) {
            first = next;//如果删除的节点是头节点,令头节点指向该节点的后继节点
        } else {
            prev.next = next;//将前驱节点的后继节点指向后继节点
            x.prev = null;
        }

        //删除后继指针
        if (next == null) {
            last = prev;//如果删除的节点是尾节点,令尾节点指向该节点的前驱节点
        } else {
            next.prev = prev;
            x.next = null;
        }

        x.item = null;
        size--;
        modCount++;
        return element;
    }
```

**remove(int index)**：删除指定位置的元素

```
public E remove(int index) {
        //检查index范围
        checkElementIndex(index);
        //将节点删除
        return unlink(node(index));
    }
```