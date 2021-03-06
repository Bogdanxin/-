# Nginx 限流

[Nginx限流 ](https://cloud.tencent.com/developer/article/1497934?from=information.detail.nginx限流)

## 限制访问频率

### 正常情况下进行访问频率的限制

**Nginx** 中使用`ngx_http_limit_req_module`模块来限制的访问频率，限制的原理实质是基于漏桶算法原理来实现的。在 **nginx.conf** 配置文件中可以使用`limit_req_zone`命令及`limit_req`命令限制单个**IP**的请求处理频率。

我们可以先来看看这两个命令的语法结构：

```javascript
limit_req_zone  key  zone  rate
```

对于上面语法结构的参数简单做下解释：

```javascript
key: 定义需要限流的对象。
zone: 定义共享内存区来存储访问信息。
rate: 用于设置最大访问速率。
```

接下来我们看个简单的例子：

```javascript
http {
  limit_req_zone $binary_remote_addr zone=myLimit:10m rate=5r/s;
}

server {
location / {
    limit_req zone=myLimit;
    rewrite / http://www.niyueling.cn permanent;
  }
}
```

对配置简单做下解释：

1. 上面**binary_remote_addr**就是**key**，表示基于客户端**ip(remote_addr)**进行限流，**binary_**表示压缩内存占用量。
2. 定义了一个大小为**10M**，名称为**myLimit**的内存区，用于存储**IP**地址访问信息。
3. **rate**设置**IP**访问频率，**rate=5r/s**表示每秒只能处理每个**IP**地址的**5**个请求。**Nginx**限流是按照毫秒级为单位的，也就是说**1**秒处理**5**个请求会变成每**200ms**只处理一个请求。如果**200ms**内已经处理完**1**个请求，但是还是有有新的请求到达，这时候**Nginx**就会拒绝处理该请求。

### 流量突发情况下进行访问频率的限制

如果突发流量超出请求被拒绝处理，无法处理活动时候的突发流量，这时候应该如何进一步处理呢？**Nginx**提供**burst**参数结合**nodelay**参数可以解决流量突发的问题，可以设置能处理的超过设置的请求数外能额外处理的请求数。我们可以将之前的例子添加**burst**参数以及**nodelay**参数：

```javascript
http {
  limit_req_zone $binary_remote_addr zone=myLimit:10m rate=5r/s;
}

server {
location / {
    limit_req zone=myLimit burst=5 nodelay;
    rewrite / http://www.niyueling.cn permanent;
  }
}
```

可以看到在原有的**location**中的**limit_req**指令中添加了b**urst=5 nodelay，**如果没有添加**nodelay**参数，则可以理解为预先在内存中占用了**5**个请求的位置，如果有**5**个突发请求就会按照**200ms**去依次处理请求，也就是**1s**内把**5**个请求全部处理完毕。如果**1s**内有新的请求到达也不会立即进行处理，因为紧急程度更低。这样实际上就会将额外的**5**个突发请求以**200ms/**个去依次处理，保证了处理速率的稳定，所以在处理突发流量的时候也一样可以正常处理。**如果添加了nodelay参数**则表示要立即处理这**5**个突发请求。

## **限制并发连接数**

**Nginx**中的**ngx_http_limit_conn_module**模块提供了限制并发连接数的功能，可以使用**limit_conn_zone**指令以及**limit_conn**执行进行配置。接下来我们可以通过一个简单的例子来看下：

```javascript
http {
  limit_conn_zone $binary_remote_addr zone=myip:10m;
  limit_conn_zone $server_name zone=myServerName:10m;
}

server {
location / {
    limit_conn myip 10;
    limit_conn myServerName 100;
    rewrite / http://www.niyueling.cn permanent;
  }
}
```

上面配置了单个**IP**同时并发连接数最多只能**10**个连接，并且设置了整个虚拟服务器同时最大并发数最多只能**100**个链接。当然，只有当请求的**header**被服务器处理后，虚拟服务器的连接数才会计数。

刚才有提到过**Nginx**是基于漏桶算法原理实现的，实际上限流一般都是基于漏桶算法和令牌桶算法实现的。接下来我们来看看两个算法的介绍：

## **漏桶算法**

漏桶算法是网络世界中流量整形或速率限制时经常使用的一种算法，它的主要目的是控制数据注入到网络的速率，平滑网络上的突发流量。漏桶算法提供了一种机制，通过它，突发流量可以被整形以便为网络提供一个稳定的流量。也就是我们刚才所讲的情况。**漏桶算法提供的机制实际上就是刚才的案例：突发流量会进入到一个漏桶，漏桶会按照我们定义的速率依次处理请求，如果水流过大也就是突发流量过大就会直接溢出，则多余的请求会被拒绝。所以漏桶算法能控制数据的传输速率。**

![image-20210305213357592](https://raw.githubusercontent.com/Bogdanxin/cloudImage/master/20210305213357.png)

## **令牌桶算法**

令牌桶算法是网络流量整形和速率限制中最常使用的一种算法。典型情况下，令牌桶算法用来控制发送到网络上的数据的数目，并允许突发数据的发送。**Google**开源项目**Guava**中的**RateLimiter**使用的就是令牌桶控制算法。令牌桶算法的机制如下：存在一个大小固定的令牌桶，会以恒定的速率源源不断产生令牌。如果令牌消耗速率小于生产令牌的速度，令牌就会一直产生直至装满整个令牌桶。

![image-20210305213331068](https://raw.githubusercontent.com/Bogdanxin/cloudImage/master/20210305213331.png)

![image-20210305213303456](https://raw.githubusercontent.com/Bogdanxin/cloudImage/master/20210305213303.png)**漏桶算法与令

**牌桶算法的区别**

**[令牌桶算法(Token Bucket)和漏桶算法(Leaky Bucket)](https://developer.aliyun.com/article/701279)**

**两种算法都能够限制数据传输速率，但令牌桶还允许某种程度的突发传输。因为令牌桶算法只要令牌桶中存在令牌，那么就可以突发的传输对应的数据到目的地，所以更适合流量突发的情形下进行使用。**

