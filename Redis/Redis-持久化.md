# Redis持久化

## 持久化过程保存什么

* 将当前的数据进行保存，**快照**形式，存储数据结果，存储格式简单，关注点在数据（**RDB**）
* 将数据的操作过程进行保存，**日志**形式，存储操作过程，存储格式复杂，关注点再数据的操作过程（**AOF**）

## RDB

### RDB启动方式——save命令

* 命令

  ```java
  save
  ```

* 作用

  手动执行一次保存操作

### RDB启动方式——save指令相关配置

* dbfilename dump.rdb

  说明：设置本地数据库文件名，默认值为dump.rdb

  经验：通常设置为**dump-端口号.rdb**

* dir

  说明：设置存储.rdb文件路径

  经验：通常设置为存储空间较大的目录，目录名称为**data**

* rdbcompression yes

  说明：设置存储值本地数据库时，是否压缩数据，默认为yes，采用lzf压缩

  经验：通常默认为开启状态，可以节省cpu运行时间，但会使存储文件变大

* rdbchecksum yes

  说明：设置是否进行RDB文件可是校验，该校验过程在写文件和读文件过程均进行

  经验：通常默认为开启状态，如果设为no，可以节约读写性能，但是存储一定数据损坏风险
  
* stop-writes-on-bgsave-error yes

  说明：后台存储过程中如果出现错位现象，是否停止保存操作

  经验：通常默认为开启状态

### RDB启动方式——save指令工作原理

save指令执行会阻塞当前redis服务器，直到当前RDB过程完成为止，有可能会造成长时间阻塞，线上环境不建议使用



### RDB启动方式——bgsave指令

* 命令

  ````
  bgsave
  ````

* 作用

  手动启动后台保存操作，但不是立即执行

### RDB启动方式——bgsave指令工作原理

![image-20200512153813908](https://raw.githubusercontent.com/Bogdanxin/cloudImage/master/20200512153826.png)

### RDB启动方式——save配置

* 配置

  ```
  save second changes
  ```

* 作用

  满足限定时间范围内key的变化数量达到指定数量即进行持久化

* second：监控时间范围

  changes：监控key的变化量

* 位置

  在conf文件中进行配置

### RDB启动方式——save配置原理

![image-20200512155307744](https://raw.githubusercontent.com/Bogdanxin/cloudImage/master/20200512155308.png)

注意：

* save配置要根据实际业务情况进行配置，频度过高或者过低都会出现性能问题
* save配置中，对于second和changes设置通常具有互补对应关系，尽量不要设置成包含关系
* save配置启动后执行的时bgsave操作

### 三种方式进行对比

![image-20200512155652269](https://raw.githubusercontent.com/Bogdanxin/cloudImage/master/20200512155653.png)



### RDB优点

* RDB是一个紧凑压缩的二进制文件，存储效率高
* RDB内部存储的是redis在某个时间点的数据快照，非常适合数据备份，全量复制等场景
* RDB恢复数据速度要比AOF快很多
* 应用：服务器每X小时执行bgsave备份，并将RDB文件拷贝到远程机器中，用于灾难恢复

### RDB缺点

* RDB无论是执行指令还是利用配置，无法做到实时持久化，具有较大可能性丢失数据
* bgsave每次运行要执行fork操作创建子进行，要牺牲掉一些性能
* Redis众多版本中未进行RDB文件格式的版本统一，有可能出现各版本之间数据格式无法兼容现象 



## AOF

### 概念

* AOF（append only file）持久化：以独立日志的方式记录每次写命令，重启时再重新执行AOF中的命令达到恢复数据的目的。与RDB相比可以简单描述为<font color="red">改记录数据为记录数据产生过程</font>
* AOF主要作用就是解决了数据持久化的时效性，目前是Redis持久化主要方式

### AOF写数据三种策略（appendfsync）

* always（每次）

  每次写入操作均同步到AOF文件中，数据零误差，性能较低

* everysec（每秒）

  每秒将缓冲区中的指令同步到AOF文件中，数据准确性较高，性能较差，在系统每次突然宕机情况下丢失一秒内的数据

* no（系统控制）

  由操作系统控制每次同步到AOF文件的周期，整体过程**不可控**

### AOF功能开启

* 配置

  ```
  appendonly yes|no
  ```

* 作用

  是否开启AOF持久化功能，默认为不开启状态

* 配置

  ````
  appendfsync always | everysec | no
  ````

* 作用

  AOF写数据策略

### AOF重写

随着命令不断写入AOF，文件会越来越大，为解决这个问题，Redis引入AOF重写机制，压缩文件体积。AOF文件重写是将Redis进程内的数据转化为写命令同步到新的AOF文件过程。即将对同一个数据的若干条命令执行结果转化为最终结果数据对应的指令进行记录

### AOF重写作用

* 降低磁盘占用率，提高磁盘利用率
* 提高持久化效率，降低持久化写时间，提高IO性能
* 降低数据恢复用时，提高数据恢复效率

### AOF 重写规则

* 进程内已超时的数据不再写入文件
* 忽略无效指令，重写时使用进程内数据直接生成，这样新的AOF文件只保留最终数据写入命令
* 对同一个数据多条写命令合并为一条命令

### AOF重写方式

* 手动重写

  ```
  bgrewriteaof
  ```

* 自动重写

  ```
  auto-aof-rewrite-min-size size  // 最小触发的数值
  auto-aof-rewrite-percentage percentage  // 最小触发的百分比
  ```

  

### AOF手动重写原理

![image-20200513142049861](https://raw.githubusercontent.com/Bogdanxin/cloudImage/master/20200513142051.png)

### AOF自动重写原理

* 自动重写触发对比参数

  ````
  aof_current_size // 自动触发的数值
  aof_base_size //基础数值
  ````

  ![image-20200513143354087](https://raw.githubusercontent.com/Bogdanxin/cloudImage/master/20200513143355.png)

  

