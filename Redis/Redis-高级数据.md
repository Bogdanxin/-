# 高级数据

## Bitmaps

通过一个bit位来表示某个元素对应的值或者状态,其中的key就是对应元素本身。Bitmaps 本身不是一种数据结构，实际上它就是字符串（key 对应的 value 就是上图中最后的一串二进制），但是它可以对字符串的位进行操作。 Bitmaps 单独提供了一套命令，所以在 Redis 中使用 Bitmaps 和使用字符串的方法不太相同。可以把 Bitmaps 想象成一个以 位 为单位的数组，数组的每个单元只能存储 0 和 1，数组的下标在Bitmaps中叫做偏移量。

### 基础操作

* 获取指定key对应偏移量上的bit值

  ```
  getbit key offset
  ```

* 设置指定key对应偏移量上的bit值，value只能是1或者0

  ```
  setbit key offset value
  ```

### 扩展操作

* 对指定key按位进行交、并、非、异或操作，并将结果保存到destKey中，就是将操作的到的数复制一份到新的key中

  ```
  bitop op destKey key1 key2 ...
  ```

  op:and、 or、 not、 xor

* 统计指定key中1的数量

  ```
  bitcount key [start end]
  ```

## HyperLogLog

* 基数是数据集去重后元素的个数
* HyperLogLog是用来做基数统计用的运用的LogLog算法

 ### 基本统计

* 添加数据

  ```
  pfadd key element [element ...]
  ```

* 统计数据

  ```
  pfcount key [key...]
  ```

* 合并数据

  ```
  pfmerge destkey sourcekey [sourcekey...]
  ```

注意：

* HyperLogLog用于进行基数统计，不是集合，不保存数据，只记录数量而不是具体数据
* 核心是基数估计算法，最终数值存在一定误差



## GEO

坐标方法

### 基本操作

* 添加坐标点

  ```
  geoadd key longitude latitude member [longitude latitude member ...]
  ```

  key作为一个容器，只有在这个容器中才能够使用，不同容器key中的坐标是不能互相使用的。

  longitude、latitude是坐标

  member就是坐标点的名称

* 获取坐标点

  ```
  geopos key member [member ...]
  ```

* 计算坐标点

  ```
  geodist key member1 member [unit]
  ```

  unit表示单位

* 根据坐标求范围内的数据

  ```
  georadius key longitude latitude radius m|km|ft|mi [withcoord] [withdist] [withhash] [count count]
  ```

* 根据点求范围内数据

  ```
  georadiusbymember key member radius m|km|ft|mi [withcoord] [withdist] [withhash] [count count]
  ```

* 获取指定点的坐标hash值

  ```
  geohash key member [member ...]
  ```

  