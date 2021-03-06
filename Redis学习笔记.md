# 《Redis设计与实现》笔记

### 字符串对象

---

字符串的编码有三种类型：

1. int:底层使用整数保存值
2. embstr：使用embstr编码的SDS保存值
3. row：使用SDS保存值

在一定条件下，int和embstr这两种编码方式会转成row

| 命令                           | 作用                                            |
| ------------------------------ | ----------------------------------------------- |
| SET [key] [value]              | 创建字符串对象                                  |
| GET [key]                      | 获取value                                       |
| APPEND [key] [value]           | 向字符串追加值                                  |
| INCRBYFLOAT [key] [value]      | 将字符串对象加上浮点值，如果字符转可以转为float |
| INCRBY [key] [value]           | 将字符串对象加上整数值，针对于int编码的对象     |
| DECRBY [key] [value]           | 将字符串对象减去整数值，针对于int编码的对象     |
| STRLEN [key]                   | 计算字符串长度                                  |
| SETRANGE [key] [index] [value] | 将指定索引位置设置为指定字符                    |
| GETRANGE [key] [start] [end]   | 获取指定索引位置的字符                          |

### 列表对象

---

列表对象的编码有两种类型：ziplist和linkedlist，当**同时满足**以下的条件时，使用ziplist进行编码

1. 列表对象保存的所有字符串元素的长度都小于64字节
2. 列表对象保存的元素数量小于512个

以上两个条件的上限可以通过配置文件修改：list-max-ziplist-value和list-max-ziplist-entries

| 命令                                               | 作用                             |
| -------------------------------------------------- | -------------------------------- |
| LPUSH [key] [value]                                | 在表头加入元素                   |
| RPUSH [key] [value]                                | 在表尾加入元素                   |
| LPOP [key]                                         | 从表头移除元素                   |
| RPOP [key]                                         | 从表尾移除元素                   |
| LINDEX [key] [index]                               | 获取指定索引位置元素             |
| LLEN [key]                                         | 获取列表长度                     |
| LINSERT [key] [BEFORE\|AFTER] [existValue] [value] | 在指定元素的前/后插入元素        |
| LREM [key] [count] [value]                         | 移除count数量个与value相同的元素 |
| LTRIM [key] [start] [end]                          | 移除索引范围之外的元素           |
| LSET [key] [index] [value]                         | 将指定索引位置的元素修改为新元素 |

### 哈希对象

---

哈希对象的编码方式有两种：ziplist和hashtable。当**同时满足**以下的条件时，使用ziplist进行编码

1. 哈希对象保存的键值对的键和值的大小不超过64字节。
2. 哈希对象保存的键值对的数量小于512个。

以上两个条件上限可以通过配置文件修改：hash-ziplist-max-value和hash-ziplist-max-entries

| 命令                         | 作用                     |
| ---------------------------- | ------------------------ |
| HSET [key] [h_key] [h_value] | 向哈希对象添加键值对     |
| HGET [key] [h_key]           | 获取哈希对象中指定键的值 |
| HEXISTS [key] [h_key]        | 判断指定的键是否存在     |
| HDEL [key] [h_key]           | 删除指定的键值对         |
| HLEN [key]                   | 获取键值对数量           |
| HGETALL [key]                | 获取多有键值对           |

### 集合对象

---

集合对象的编码方式有两种：inset和hashtable。当**同时满足**以下的条件时，使用intset进行编码

1. 集合对象保存的多有元素都为整数值
2. 集合对象所保存的元素数量不超过512个

上述第二个条件的上限可以在配置文件中配置：set-max-intset-entries。

| 命令                    | 作用                                 |
| ----------------------- | ------------------------------------ |
| SADD [key] [value]      | 向集合对象添加元素                   |
| SCARD [key]             | 获取集合对象的元素个数               |
| SISMEMBER [key] [value] | 判断给定元素值是否存在于集合对象     |
| SMEMBERS [key]          | 获取集合对象的所有元素               |
| SRANDMEMBER [key]       | 从集合对象中随机返回一个元素         |
| SPOP [key]              | 从集合对象中随机返回一个元素，并删除 |
| SREM [key] [value]      | 从集合对象中移除指定元素，值为value  |

### 有序集合对象

---

有序集合对象的编码方式有两种：ziplist和skiplist。当**同时满足**以下的条件时，使用ziplist进行编码

1. 有序集合保存的元素数量小于128个。
2. 有序集合保存的元素长度小于64字节。

以上两个条件的上限是可以配置的：zset-max-ziplist-entries和zset-max-ziplist-value

| 命令                                | 作用                                              |
| ----------------------------------- | ------------------------------------------------- |
| ZADD [key] [score] [value]          | 向有序集合添加元素                                |
| ZCARD [key]                         | 获取有序集合的元素数量                            |
| ZCOUNT [key] [score_1] [score_2]    | 获取分值在指定区间的元素数量                      |
| ZRANGE [key] [index_1] [index_2]    | 获取指定索引范围的元素[index_1,index_2]，从左开始 |
| ZREVRANGE [key] [index_1] [index_2] | 获取指定索引范围的元素[index_1,index_2]，从右开始 |
| ZRANK [key] [value]                 | 获取元素的排名，从左遍历                          |
| ZREVRANK [key] [value]              | 获取元素的排名，从右遍历                          |
| ZREM [key] [value]                  | 删除指定元素值                                    |
| ZSCORE [key] [value]                | 获取元素的分值                                    |

### 第一部分重点回顾

---

1. Redis数据库中的键和值都是对象
2. Redis共有5种对象，分别是：字符串对象、列表对象、哈希对象、集合、有序集合。每个对象至少有两种编码方式，也即至少两种底层实现。
3. 服务器在实行命令之前，会检查给定的键的类型是否能够执行指定的命令，而检查一个键的类型就是检查键对应的值对象的类型。
4. Redis的对象系统带有对象引用计数实现的垃圾回收机制，当一个对象不再被使用时，对象占用的内存会被释放。
5. Redis共享值为0-9999的字符串对象
6. 对象会记录自己最后一次被访问的时间，这个时间可以用来计算对象的空转时间。

### 通用命令

| 命令                   | 作用                     |
| ---------------------- | ------------------------ |
| DEL [key]              | 删除key及对应的value对象 |
| EXPIRE [key] [seconds] | 设置键的过期时间         |
| REANME                 |                          |
| TTL                    |                          |
| TYPE                   |                          |
| OBJECT                 |                          |

### 对象回收

---

- 对象引用计数：引用数为0的对象会被回收
- 过期自动回收：到了过期时间的对象会被回收
- 空转时长回收：如果设定了maxmemory参数，当redis占用内存达到这个值时，会回收空转时长较高的对象

### Redis对象删除策略

- 惰性删除
- 定期删除

### RDB持久化方式

---

（一）redis可以使用``SAVE``命令或者``BGSAVE``命令将当前的数据库状态持久化到磁盘中，运行这两个命令之后得到一个.rdb后缀的文件，其中保存了执行命令时数据库中的所有数据。

（二）两个命令的区别在于，``SAVE``命令会阻塞redis直到持久化工作完成，期间不能响应其他命令。``BGSAVE``单独创建一个进程来做持久化操作，不会阻塞redis服务。

（三）可以通过redis.conf配置RDB自动间隔性保存策略如下图：

![](./resources/redis_save_config.png)

第一个配置项的意思是：在上次持久化之后至少900秒内发生了至少1次修改redis中的数据，满足要求则进行持久化操作。默认使用``BGSAVE``。

（四）redis服务启动之后会自动读取.rdb文件进行数据载入。

（五）RDB文件是二进制文件，执行持久化时，将数据库中的数据（键值对）进行压缩生成二进制文件。

### AOF持久化方式

---

（一）通过redis.conf配置“**appendonly  yes**”，可以开启AOF持久化功能。AOF持久化的方式是将对数据的写命令保存到文件中，所有命令满足请求协议格式。

（二）通过配置“appendfsync”选项的值可以决定AOF持久化的效率和安全性，有以下三个值可以选择：

1. always:每次事件循环都将aof_buf缓冲区中的所有内容写入AOF文件，并且同步AOF文件。安全性最高，最多丢失一个事件循环中所产生的命令数据。
2. everysec:每次事件循环都将aof_buf缓冲区中的所有内容写入AOF文件,并且每隔一秒就由**子线程**来进行一次同步。安全性较高，最多丢失一秒钟产生的命令数据。该值是默认项。
3. no:每次事件循环都将aof_buf缓冲区中的所有内容写入AOF文件，由操作系统决定同步时间。

（三）AOF重写：通过``BGREWRITEAOF``命令，可以对AOF文件进行重写。解决了自动追加写AOF文件导致，文件体积增大，载入时间长的问题。AOF重写会根据当前的数据库状态，对于每个键值对，使用更少的写命令记录数据库内容。重写过程由子进程处理。

（四）AOF重写过程中，自动追加写的方式仍然在继续，并且，会单独使用一个叫做重写缓冲区的内存空间来存储在执行重写过程中服务端执行的写操作。等待重写完成，会将这部分缓存的写操作使用追加到新的AOF文件中。

### 事件

---

- Redis服务器是一个事件驱动程序，服务器处理的事件分为文件事件和时间事件
- 文件事件处理器是基于**Reactor模式**实现的网络通信程序
- 文件事件是对套接字操作的抽象
- 文件事件分为AE_READABLE事件和AE_WRITABLE事件
- 时间事件分为定事件和周期性事件，定时事件只在指定的时间到达一次，而周期性事件每隔一段时间会到达一次
- Redis服务器一般情况下只执行serverCron函数一个时间事件，并且这个事件是周期性事件
- 文件事件和时间事件之间是合作关系，服务器会轮流处理这两种事件，并且处理事件的过程中也不会进行抢占
- 时间事件的实际处理时间通常会比设定的到达时间晚一些

### 客户端

---

- 服务器状态结构使用clients链表将客户端状态连接起来，新添加的客户端状态会放到链表末尾。
- 客户端状态的flags属性使用不同的标志来表示客户端的角色，以及客户端当前所处的状态。
- 输入缓冲区记录了客户端发送的命令请求，这个缓冲区的大小不能超过1GB。
- 输出缓冲区的限制值有两种，如果输出缓冲区的大小超过了服务器设置的硬性限制，那么客户端会被立即关闭，如果客户端在一定时间内，一直超过服务器设置的软性限制，那么客户端一会被关闭。
- 命令的参数和参数的个数会记录在客户端状态的argv和argc属性里面，而cmd属性则记录了客户端要执行的命令对应的实现函数。
- 当一个客户端通过网络连接连上服务器时，服务器会为这个客户端创建相应的客户端状态。网络连接关闭、发送了不合协议格式的命令请求、成为CLIENT KILL命令的目标、空转时间超时、输出缓冲区的大小超出限制，这些原因都会导致客户端被关闭。
- 处理lua脚本的伪客户端在服务器初始化时创建，这个客户端会一直存在，直到服务器关闭。
- 载入AOF文件时使用的伪客户端在载入工作开始时动态创建，载入工作完成之后关闭。

### 服务器

---

- 一个命令请求从发送到完成需要以下步骤：
  1. 客户端将命令发送给服务器
  2. 服务器读取命令请求，分析出命令参数
  3. 命令执行器根据参数查找命令的实现函数，然后执行实现函数
  4. 服务器将命令回复发送给客户端
- serverCron函数默认每个100ms执行一次，它的工作主要包含更新服务器状态信息，处理服务器接收的SIGTERM信号，管理客户端资源和数据库状态，检查并执行持久化操作等。
- 服务器从启动能够为客户端处理命令请求需要执行以下步骤：
  1. 初始化服务器状态
  2. 载入服务器配置
  3. 初始化服务器数据结构
  4. 还原数据库状态
  5. 执行事件循环

### 复制

---

- redis在2.8版本开始支持部分重同步功能
- 部分重同步通过复制偏移量、复制积压缓冲区、服务器运行ID三个部分来实现
- 主服务器通过向从服务器转播命令来更新从服务器的状态，从而保持主从一致，而从服务器则通过向主服务器发送命令来进行心跳检测，以及命令丢失检测。

