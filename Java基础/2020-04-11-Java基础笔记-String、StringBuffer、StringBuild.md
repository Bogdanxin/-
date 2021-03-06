[TOC]

# String、StringBuffer、StringBuild

## 一、可变性

* 都是final类，不可继承
* String长度不可变，可以为空串，String类中使用char数组保存字符串。这个char数组在String类中是final的
* StringBuffer和StringBuilder长度可变，都是继承自AbstractStringBuilder类，在父类定义的char数组只是一个普通的私有变量，可以通过append方法追加。
* String的char数组是final的，但是StringBuffer和StringBuilder的char数组不是final的

## 二、线程安全性

* String对象是不可变的，也就是一个常量，所以是**线程安全的**。
* StringBuffer对方法加了同步锁（synchronized），所以是线程安全的
* StringBuilder并未将方法添加同步锁，所以是非线程安全的，为JDK5开始为StringBuffer补充的单线程等价类

## 三、性能

* 优先考虑StringBuilder，支持所有StringBuffer的所有操作，因为不执行同步，不会因为线程安全带来的额外系统消耗，速度更快。
* 操作少量数据用String，大量多线程StringBuffer，大量单线程StringBuilder
* 进场要改变内容的字符串不要使用String，因为每次修改都要生成新的对象

## 四、String为什么是不可变的

1. **什么是不可变的对象**：一个对象在创建之后，不能改变他的状态，这个对象就是不可变的。不可变的意思事，不能改变对象的成员变量，包括基本数据类型的值不能改变，引用类型的变量不能指向其他对象，引用数据类型指向的对象也不能改变。

2. 字符串常量池中，存放的字符串是公用的，也就是创建一个新的字符串（注意不是new的时候，而是直接赋值）

   ```java
   String str = "ABC";
   String str1 = new String("ABC");
   String str2 = "ABC";
   // 两者是不相同的str1是在内存中新创建一个空间
   // 而str2是利用了池中也叫做ABC的字符串，没有创建新空间
   ```

   

   先看常量池中有没有这个字符串，有就直接引用这个字符串，没有就创建一个新的字符串对象，之后其他的String对象也能引用。

   如果字符串允许改变，就会影响到其他引用该字符串的其他String对象。

3. 因为虽然value是不可变，也只是value这个引用地址不可变。也就是说Array变量只是stack上的一个引用，数组的本体结构在heap堆。String类里的value用final修饰，只是说stack里的这个叫value的引用地址不可变。没有说堆里array本身数据不可变。

4. **当使用+符号串联String时，底层会转化成StringBuilder，然后进行``append()``方法**

5. 允许String对象缓存HashCode。字符串不变保证了hash码唯一性，可以放心去缓存，不必每次都去计算新的hash码。

6. 由于线程安全性的考虑，不变的String在多线程中可以使用

7. 程序中大量使用字符串，出于安全性的考虑