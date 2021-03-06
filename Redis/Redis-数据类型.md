# Redis-数据类型

## Redis 数据存储格式

* reids自身是一个Map，其中所有的数据都是采用key: value的形式存储
* 
* 数据类型指的是存储的数据类型，也就是value部分的类型，key永远是字符串



## string类型

* 存储的数据：单个数据，最简单的数据存储类型，也是最常用的
* 存储格式：一个存储空间保存一个数据
* 存储内容：通常使用字符串，如果字符串以整数形式展示，可以作为数字操作

```redis
key1 -> string1
key2 -> string2
```

### 基本操作

单一的**增删改查**

```redis
set key value // 添加/修改数据
get key // 获取数据
del key // 删除数据
```

多个**增删改查**

```redis
mset key1 value1 key2 value2 ...// 添加/修改多个数据
mget key1 key2 ... // 获取多个数据
strlen key // 获取数据字符个数（字符串长度）
append key value // 追加信息道原始信息候补（如果原始信息存在就追加，否则新建）
```

### 拓展操作

* 设置数据数据增加指定范围的值

  ```powershell
  incr key
  incrby key increment
  incrbyfloat key increment
  ```

* 设置数值数据减少指定范围的值

  ```shell
  decr key
  decrby key increment
  ```

  redis 用于控制数据库表中主键id，为数据表主键提供生成策略，保障数据库表的主键唯一性

* 设置数据具有指定生命周期

  ```shell
  setex key seconds value
  psetex key millisecond value
  ```

  redis控制数据的生命周期，通过数据是否失效空指业务行为，适用于所有具有时效性限定控制的操作

### 应用场景

* 在redis中为大V设定用户信息，以用户主键和属性为key，后台设定定时刷新策略即可

  ```shell
  set user:id:123:fans 12345667
  set user:id:123:blog 123455
  ```

* 在redis中以json格式存储信息，定时刷新

  ```shell
  set user:id:123 {id:123, name:xxx, fans:123456677, blogs:4342}
  ```

redis应用于各种结构性和非结构性高热度数据访问加速

* **key的设置约定**（数据库中热点数据key命名惯例）

  ![image-20200507092557880](https://raw.githubusercontent.com/Bogdanxin/cloudImage/master/20200507092619.png)

  

## hash类型

### 基本操作

* 单个操作

  ```shell
  hset key field value // 添加/修改数据
  hget key field  // 获取数据
  hgetall key  
  hdel key fieldl [field2] // 删除数据
  ```

* 多个数据操作

  ```shell
  hmset key field1 value1 field2 value2 ... // 添加/修改多个数据
  hmget key field1 field2 ...  // 获取多个数据
  hlen key  // 获取哈希表中字段数量
  hexits key field   // 获取哈希表中是否存在指定字段
  ```

### 扩展操作

* 获取哈希表中所有字段名或者字符值

  ```shell
  hkeys key // 获取指定key下的所有字段（field）
  hvals key // 获取指定key下的所欲value
  ```

* 设置指定字段的数值数增加指定范围的值

  ```shell
  hincrby key field increment
  hincrbyfloat key field increment
  ```

### 应用场景

电商的购物车中，每个用户的购物车就是一个hash，商品id就是一个field，对应的数据就是value，但是由于商品信息是不经常改变的，所以可以这样设计

```shell
hmset user1 商品1:num 100 商品1信息 {...}
hmset user2 商品2:num 200 商品2信息 {...}
```

抢购话费券，分别有100元、50元、10元各1000张

## list类型

* 数据存储需求：存储多个数据，比对数据进行存储空间的顺序进行区分
* 需要存储结构：一个存储空间保存多个数据，而且通过数据可以体现进入顺序
* list类型：保存多个数据，底层使用双向链表存储结构实现

![image-20200507154958085](https://raw.githubusercontent.com/Bogdanxin/cloudImage/master/20200507154959.png)

### 基本操作

```
// 添加/修改数据
lpush key value1 [value2] ...
rpush key value1 [value2] ...
// 获取数据
lrange key start stop // 使用 0 -1 就是反着查，从倒数第一个开始查
lindex key index
llen key
// 获取并移除数据
lpop key
rpop key
```

### 拓展操作

* 规定时间获取并移除数据

  ```shell
  blpop key1 [key2] timeout
  brpop key1 [key2] timeout
  ```

  作用就是等待timeout时间内，将list中的数据pop出来，如果list中有数据，就将数据直接pop出来，如果没有，就等待timeout时间，这个时间内，如果有一个进入，则直接pop出

* 移除指定数据

  ```
  lrem key count value // count指的是移除个数，value指的是要移除的值
  ```

应用于具有操作先后顺序的数据控制



## set 类型

* 新的存储需求：存储大量的数据，在查询方面提供更高的效率
* 需要的存储结构：能够保存大量的数据，高效的存储机制，便于查询
* set类型：与hash存储机构完全相同，仅用于存储键，不存值（nil），而且键是不允许重复的

![image-20200508091506068](https://raw.githubusercontent.com/Bogdanxin/cloudImage/master/20200508102537.png)

### 基本操作

```
sadd key member1 [member2] // 添加数据
smembers key				   // 获取全部数据
srem key member1 [member2] // 删除数据
```

```
scard key // 获取集合数据总量
sismember key member // 判断集合中是否包含指定数据
```

### 拓展操作

* 随机获取集合中的指定数量的数据

  ```
  srandmember key [count]
  ```

* 随机获取集合中的某个数据，并将该数据移出集合

  ```
  spop key
  ```

* 求两个集合的交、并、差集

  ``` 
  sinter key1 [key2]
  sunion key1 [key2]
  sdiff key1 [key2]
  ```

* 求两个交集的交、并、差集并存储到指定集合中

  ```
  sinterstore destination key1 [key2]
  sunionstore destination key1 [key2]
  sdiffstore destination key1 [key2]
  ```

* 将指定数据从原始集合中移动到目标集合中

  ```
  smove source destination member
  ```

**Redis可以应用于同类型不重复数据的合并操作**

## sorted_set 类型

* 新的存储需求：数据排序有利于数据的有效展示，需要提供一种可以根据滋生特征进行排序的方式

* 需要的存储结构：新的存储模型，可以保存可排序的数据
* sorted_set 类型：在set的存储结构基础上添加可排序的字段

![image-20200508102527673](https://raw.githubusercontent.com/Bogdanxin/cloudImage/master/20200508102528.png)

### 基本操作

* 增删改查

  ```
  // 添加数据
  zadd key score1 member1 [score2 member2] 
  // 获取全部数据
  zrange key start stop [withscores]
  zrevrange key start stop [withscores]
  // 删除数据
  zrem key member [member...]
  ```

* 按条件获取数据

  ```
  zrangebyscore key min max [withscores] [limit] // 在min和max之间的数据，limit表示根据索引查询
  zrevrangebyscore key max min [withscores]
  ```

* 条件删除数据

  ```
  zremrangebyrank key start stop  // 根据索引删除
  zremrangebyscore key min max    // 根据大小间距删除
  ```

* 获取集合数据总量

  ```
  zcard key
  zcount key min max
  ```

* 集合交、并操作

  ```
  zinterstore destination numkeys key [key...] // numkey表示一共有几个集合要操作，destination表示把操作得出的数据放入的集合
  zunionstore destination numkeys key [key...]
  ```

  这两个操作有很多功能，可以查看一下

* 获取数据对应的索引（排名）

  ```
  zrank key member
  zrevrank key member
  ```

* score值获取与修改

  ```
  zscore key member // 获取指定member里面的score
  zincrby key increment member  // 将指定member的数据进行加减
  ```

**注**：sorted_set底层存储基于set结构，因此数据不能重复，如果重复添加相同数据，score值将被反复覆盖，保留最后一次修改结果

