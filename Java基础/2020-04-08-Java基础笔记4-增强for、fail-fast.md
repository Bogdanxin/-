---
layout:		post
title:		Java中的增强for和fail-fast机制
subtitle:	Java基础笔记
date:		2020-04-08
catalog:	true
author:		BogdanXin
tag:
- Java
---

# java中的增强for和fail-fast机制

## 1.Java中的增强for

### 1.1 具体实现

增强for是Java提供的语法糖。

对以下代码进行反编译：

```java
for(Integer i : list) {
    System.out.println(i);
}
```

反编译后：

```java
Integer i;
for(Iterator iterator = list.iterator(); iterator.hasNext(); System.out.println(i)) {
    i = (Integer)iterator.next();
}
```

反编译后，代码执行顺序为：

1. 定义一个Integer变量
2. 获取List迭代器
3. 判断迭代器中是否有未遍历的元素
4. 获取未遍历元素，并赋值给i
5. 输出i的值

由此可得，java中的增强for底层是通过迭代器模式实现的。

### 1.2 注意事项

增强for通过迭代器实现，必然有迭代器的特性。

fail-fast机制中，迭代器遍历元素，对集合进行删除一定注意，使用不当会有可能发生`ConcurrentModificationExceptoin`

```java
for(Student stu : students) {
    if(stu.getId() == 2) {
        students.remove(stu);
    }
}
```

Iterator在工作时，不允许被迭代对象改变。但是可以使用Iterator本身方法``remove()``来删除对象。

```java
Iterator<Student> students = list.interator();
while(students.hasNext()) {
    Student student = students.next();
    if(student.getId() == 2) {
        // 这里使用的Iterator的remove方法移除当前的对象，如果使用List的remove方法，则会出现ConcurrentModificationExceptoin
        students.remove();
    }
}
```

## 2.fail-fast机制

fail-fast,即快速失败机制，它是java集合中的一种错误检测机制，当多个线程（单个线程也可以）,在结构上对集合进行改变时，就有可能会产生fail-fast机制。抛出ConcurrentModificationcationException，当方法检测到对象的并发修改，但不允许这种修改时就抛出该异常。



