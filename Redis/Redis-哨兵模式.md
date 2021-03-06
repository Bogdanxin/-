# 哨兵模式

## 介绍

哨兵是一个分布式系统，用于对主从结构中每台服务器进行监控，当出现故障时，通过投票选择新的master并将所有的slave连接到新的master

**作用：**

1. 监控：不断检查master和slave是否正常运行。master存活检测、master与slave运行情况检查
2. 通知：当被监控服务器出现问题，向其他（哨兵间、客户端）发送通知
3. 自动故障转移：断开master和slave连接，选取一个slave作为master，将其他slave连接到新的master，并告知客户端新的服务器地址

## 启用哨兵模式

就是将配置文件进行修改

## 工作原理

### 监控阶段

同步各个节点状态信息

1. 获取各个哨兵状态（是否在线）
2. 获取master状态：
   * master属性：runid、role：master
   * 各个slave详细信息
3. 获取所有slave的状态
   * runid
   * role：slave
   * master_host、master_port
   * ...

### 通知阶段

sentinel之间创建一个频道网络，发布、分享信息。当某一个sentinel对master和slave网络进行询问后，得到的回答会发布到这个频道网络中。

### 故障转移阶段

**sentinel向master进行判断相应**

* 如果有一个sentinel1对master发送指令时，master没有回应，则再向他发送信息等待响应，到达上限后，则sentinel1则将该master标记为SRI_S_DOWN（**主观下线**）
* 然后sentinel1将master判断的状态发送到sentinel的频道中，这时其他的sentinel在对这个master进行发送消息，并进行相同的判断，如果超过半数sentinel都认为master已经没有相应了，此时master标记为SRI_O_DOWN（**客观下线**），并进行下一步

**选择领头的哨兵，对master进行下线**

投票机制

**服务器列表中挑选备选master**

* 在线的
* 响应快的
* 与原master断开时间短的（与原master最晚断开的）

* 优先原则：
  1. 优先级
  2. offset
  3. runid

**发送指令（sentinel）**

* 向新的master发送slaveof no one执行，代表不和原master连接
* 向其他slave发送slaveof 新的ip端口

