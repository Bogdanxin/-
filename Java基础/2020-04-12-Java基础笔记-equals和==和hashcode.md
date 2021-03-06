[TOC]

https://www.cnblogs.com/skywang12345/p/3324958.html

# equals和==

## 一、==

* 对于基本数据类型，使用 "=="，比较的是他们的值
* 对于引用数据类型，使用 "=="，比较的是他们在**内存中存放的地址**（确切的说，是**堆内存**中存放的地址）

对于引用数据类型，如果是new出来的对象，让他们用 "=="比较，是绝对错误的，因为new一次就会开辟一个新的内存空间，也就会导致两者内存存放地址不同。

## 二、equals()方法

Object类中定义了equals方法，默认方法是比较两个对象的内存地址，这样是不可行的。

所以每个继承类都重写了这个方法。重写之后，基本都是比较数据内容是否相同了



## 三、区别

### == 作用：

* 基本类型：比较的就是值是否相同
* 引用类型：比较的就是两个对象的内存地址值是否相同

### equals()作用：

引用类型：默认情况下（Object类）比较的是地址值，但是重写方法和其继承类都是有重写方法的，这就是比较两个对象内容是不是相同了。



所以进行字符串判断是不是相同的方法，一定要用equals()

## 四、具体实例

对String类的`equals()`和`==`进行查看

```java
String str1 = "ABC";
String str2 = "ABC";
System.out.println(str1 == str2);// true
System.out.println(str1.equals(str2));// true
```

这里的第二个==没有问题，就是比较的两个对象的内容。

第一个==是比较的两者的地址值，而因为直接`String str = "xxx"`出来的字符串，都是要先查看字符池的，str1和str2都指向的字符串池中的同一个字符串。没有问题



第二种情况就有所不同了

```java
String str1 = "ABC";
String str2 = new String("ABC");
String str3 = str2; // 引用的传递

System.out.println(str1 == str2); // false
System.out.println(str2 == str3); // true
System.out.println(str1 == str3); // false

System.out.println(str1.equals(str2)); // true
System.out.println(str1.equals(str3)); // true
System.out.println(str2.equals(str3)); // true
```

因为str2是new出来的新的对象，这样就就是在堆内存中新创建一个空间，这样str1和str2指向的地址值就不同了，这样的==就不再是true了，而equals依旧比较的是内容，所以还是true。

## 五、equals()方法重写

### 1. 遵循原则：

* 对称性：x.equals(y) == y.equals(x)。
* 自反性：x.equals(x)必须返回是"true"。
* 传递性：x.equal(y)和y.equal(z)成立时，x.equal(z)要成立。
* 一致性：x.equals(y)，只要x和y内容一直不变，结果不变。
* 非空性，x.equals(null)，永远返回是"false"；x.equals(和x不同类型的对象)永远返回是"false"。

### 2.一定要重写hashcode和equals方法

* 因为在添加到set集合中，会利用创建对象的hashcode和equals方法对对象进行判重。
* 重写equals方法是用来判定对象的内容的，而hashcode的重写是用来辅助equals的，因为大量数据时，不可能用equals一直比较。
  + 先用重写后的hashcode判断，做到尽量hashcode不重复，这样就给equals省去很多时间
  + 如果hashcode还是相同，只好使用equals进行进一步判断，只有equals也返回true，才表明两者相同。

1、如果两个对象equals，Java运行时环境会认为他们的hashcode一定相等。 
2、如果两个对象不equals，他们的hashcode有可能相等。 
3、如果两个对象hashcode相等，他们不一定equals。 
4、如果两个对象hashcode不相等，他们一定不equals。 

# Hashcode

https://www.cnblogs.com/dolphin0520/archive/2012/09/28/2700000.html

关于哈希表

## hashCode方法返回值

* Object基类的hashCode方法默认返回对象的内存地址
* 但是在某些场景下需要覆写hashCode函数，比如需要使用Map存放对象时候，覆写后的hashCode就不是对象的内存地址了

## HashCode作用

* 获取哈希码，配合散列表使用，用于确定对象的存储地址，如HashMap、Hashtable、HashSet
* hashCode相同，equals不一定true，散列表中，先比较的是hashcode，不同直接存放，相同再比较equals是否true。如果true对于set就不存放了，false则重新计算散列值/产生单链表。
* hashCode()的默认行为是对堆上的对象产生独特值。如果没有重写hashCode()，则该class的两个对象无论如何都不会相等（即使这两个对象指向相同的数据）。
* java中的hashCode方法就是根据一定的规则将与对象相关的信息（比如对象的存储地址，对象的字段等）映射成一个数值，这个数值称作为**散列值**
* hashCode是jdk根据对象的地址或者字符串或者数字算出来的int类型的数值，是native方法。

