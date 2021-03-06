

# 主从复制

## 多台服务器连接方案

* 数据提供方：master

  主服务器、主节点、主库、主客户端

* 接收数据方：slave

  从服务器、从节点、从库、从客户端

![image-20200518181939320](https://raw.githubusercontent.com/Bogdanxin/cloudImage/master/20200518181940.png)

* 解决问题：数据同步	

## 主从复制

将master中的数据即时、有效的复制到slave中

**特征：**一个master可以拥有多个slave，一个slave只对应一个master

**职责：**

* master：
  1. 写数据
  2. 执行写操作时，将出现变化的数据自动同步到slave
* slave：
  1. 读数据

## 工作流程

分为三个阶段：

1. 建立连接
2. 数据同步
3. 命令传播

![image-20200518182948833](https://raw.githubusercontent.com/Bogdanxin/cloudImage/master/20200518182950.png)

### 建立连接

1. 在slave端，设置master地址和端口，保存master信息
2. 建立socket连接
3. slave定时发送ping指令（定时）
4. 身份验证
5. slave向master发送slave端口信息

![image-20200518183534283](https://raw.githubusercontent.com/Bogdanxin/cloudImage/master/20200518183535.png)

### 数据同步

1. slave请求同步数据
2. master创建RDB同步数据
3. slave恢复RDB同步数据
4. 请求部分同步数据（部分数据指的是将RDB之后的数据进行同步，RDB是不包含缓冲区的数据的）
5. 恢复部分同步工作

![image-20200518195215616](https://raw.githubusercontent.com/Bogdanxin/cloudImage/master/20200518195217.png)

状态：

slave：具有master端所有数据，包含RDB过程中接收的数据

master：保存slave当前数据同步的位置

### 命令传播阶段

* master数据库状态被修改后，导致主从服务器数据库状态不一致，此时需要让主从数据同步到一致的状态，同步的动作称为命令传播
* master将接收到的数据变更命令发送给slave，slave接收命令后执行命令

### 服务器运行ID（runid）

* 概念：服务器运行ID是每一台服务器**每次运行**的身份识别码，一台服务器多次运行可以生成多个运行id
* 组成：运行id由40位字符组成，是一个随机的16位进制字符
* 作用：运行id被用于在服务器间进行传输，识别身份，如果想要两次操作均对同一台服务器进行，必须每次操作携带对应的运行id，用于对方识别。
* 实现方式：运行id在每台服务器启动时自动生成，master在首次连接slave时，就会将自己的运行ID发送给slave，slave保存此ID，通过info Server命令，可以查看节点的runid

### 复制缓存区

* 概念：复制缓冲区又名复制积压缓冲区，是一个先进先出（FIFO）队列，用于存储服务器指定过

  的命令，每次传播命令，master都会将传播的命令记录下来，并存储在复制缓冲区中。

### 总结

#### 工作原理：

* 组成：偏移量和字节值
* 工作原理：
  1. 通过offset区分不同的slave当前数据传播差异
  2. master记录已经发送消息对应的offset
  3. slave记录已经接收信息对应的offset

![image-20200519095224239](https://raw.githubusercontent.com/Bogdanxin/cloudImage/master/20200519095225.png)

#### 心跳机制：

* 进入命令传播阶段后，master和slave间需要进行信息交换，使用心跳机制进行维护，实现双方连接保持在线
* master心跳：
  1. 指令：PING
  2. 周期：repl-ping-slave-period决定，默认10秒
  3. 作用：判断slave是否在线
  4. 查询：INFO replication，获取slave最后一次连接时间间隔，lag项维持在0或1是为正常
* slave心跳任务：
  1. 指令：REPLCONF ACK {offset}
  2. 周期：1秒
  3. 作用：汇报slave自己的复制偏移量，获取最新的数据变更指令
  4. 作用2：判断master是否在线

<font color="red">注意事项：</font>

* slave多数掉线，或者延迟过高时，master位保障数据稳定性，将拒绝所有信息同步操作

  ```
  min-slave-to-write 2
  min-slave-max=lag 8
  ```

  slave数量少于2个，或者所有slave延迟都大于10秒时，强制关闭master写功能，停止数据同步

* slave数量由slave发送REPLCONF ACK命令做确认

* slave延迟由slave发送REPLCONF ACK命令做确认

总的工作流程

![image-20200519100946534](D:/Program Files/Typora/upload/image-20200519100946534.png)