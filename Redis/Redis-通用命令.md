# Redis通用命令

## key通用命令

### key的特征

* key是一个字符串，通过key获取redis中保存的数据

* 对于key自身状态相关操作，例如：删除、判定存在，获取类型等
* 对于key有效性控制相关操作，例如：有效期设定，判断是否有效，有效状态切换等
* 对于key快速查询操作，例如：按指定策略查询key

### key基本操作

* 删除指定key

  ```
  del key
  ```

* 获取key是否存在

  ```
  exists key
  ```

* 获取key的类型

  ```
  type key
  ```

### key时效性控制

* 为指定key设置有效期

  ```
  expire key seconds
  pexpire key milliseconds
  expireat key timestamp
  pexpireat key milliseconds-timestamp
  ```

* 获取key的有效时间

  ```
  ttl key		// 返回-2代表已经消失了，返回-1表示存在但是没有设置有效期
  pttl key  	
  ```

* 切换key从时效性转换为永久性

  ```
  persist key
  ```

### key查询模式

* 查询key

  ```
  keys pattern // pattern表示匹配模式
  ```

  **模式规则：**

  ![image-20200511203649035](https://raw.githubusercontent.com/Bogdanxin/cloudImage/master/20200511210515.png)

### 其他操作

* 为key改名

  ```
  rename key newkey		// 如果修改key的名字，是已经有的key的名字，则会覆盖相同名字的key
  renamenx key newkey		// 如果这个名字的key存在，就不能修改名字，否则就修改
  ```

* 对所有key排序

  ```
  sort  // 仅仅排序，不懂原来的数据，而且必须是set、sorted_set或list
  ```

* 其他key通用操作

  ```
  help @generic
  ```

  

## 数据库通用指令

### 基本操作

* 切换数据库

  ```
  select index
  ```

* 其他操作

  ```
  quit
  ping 
  echo message
  ```

* 数据移动

  ```
  move key db		// 移动操作必须保证移动道的数据库没有这个key
  ```

* 数据清除

  ```
  dbsize		// 查看数据总量
  flushdb
  flushall
  ```

  