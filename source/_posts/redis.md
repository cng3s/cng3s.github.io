---
title: redis
date: 2020-03-26 19:57:33
tags: 软件
---
# Redis常用数据类型
* 数据类型 type
  * string / hash / list / set（无序集合） / sorted set（有序集合）
* 编码方式 encoding
  * raw / int / ht / zipmap / linkedlist
  * ziplist / intset
* 虚拟内存 vm

# 数据结构的Redis命令
* string
* hash
* list
* set
* sorted set

# 基本命令
* 指定启动端口 -- 不指定端口则默认启动（6379）
  * redis-server --port port_number
  * redis-cli -p port_number -h ip_address -a password（密码可以在配置文件中配置）
  * redis-cli -p port_number -h ip_address(默认本机127.0.0.1) shutdown
* 指定配置文件启动
  * redis-server configution_file_path
* 清屏
  * clear
* 清空所有名字空间里的所有数据结构并把所有的数据都持久化到本地硬盘中
  * flushall
* 清空当前名字空间的所有数据结构
  * flushdb
* 了解 redis 的当前信息
  * info

# Redis键命令
* 更换名字空间
  * select namespace_number (默认从0开始)
  * 切换不同的名字空间里面的元素变量也是当前名字空间创建的
* 建立键值对
  * set key value 
* 删除键
  * del key
* 判断键是否存在
  * exists key
* 查看key的剩余生存时间
  * ttl key
* 查看所有的key
  * keys *
* 设置key的剩余生存时间
  * expire key ttl (单位：s)
  * 设置的key不存在返回 -2
  * 设置的key是永久的返回 -1
  * ttl秒后key就被删掉了
* 查看 key 类型
  * type key
* 随机生成一个key
  * randomkey
* 获得key的value
  * get key
* 重命名key
  * rename old_keyname new_keyname
  * 如果new_keyname已经存在，则new_keyname的value被赋值为old_keyname
  * 而old_keyname被删除，new_keyname原来的value也被覆盖了
  * 即使new_keyname不存在，仍是键为new_keyname，值为old_keyname
* 以nx结尾的命令
  * 这些命令都是带有判断操作的
  * 比如renamenx a b，且a, b键都存在，这个时候就不会生效
  * 再比如setnx a aa, 且a存在，则不会生效。如果不存在则生效。
  * 且这个操作要么都成功，要么都失败，这个特性叫原子性
  * 比如 msetnx q q uuu uuu，如果q存在且uuu不存在，则uuu也不会被赋值，因为q建立失败了
* 以ex结尾的命令
  * ex即expire的缩写，即这些键带有生存时间
  * 如 setex c 100 c , key: c, value:c, ttl:100s
  * 且以p开头的
    * 设置为毫秒级别
    * psetex d 10000 d, key: d, value: d, ttl: 10000ms
* 获取元素范围
  * getrange key left_limit right_limit
  * 如果数据结构没有range范围就返回空，像线性集合的数据结构都有range
* 获取并设置
  * getset key newvalue
  * 设置key为newvalue并且返回oldvalue
  * 可以理解为先 get 操作再 set 操作
* 同时设置一个或多个
  * mset key1 val1 key2 val2 ....
  * 其中key1 key2 ... 都是string类型
* 同时获取一个或多个
  * mget key1 key2 key3 ....
* 递增或递减
  * incr key
    * key的value每次增加1
  * val必须是integer类型才能用，否则失败
  * incrby key range
    * 每次增加range的值
  * decr key
  * decrby key range


# Redis数据结构
## string
* 追加
  * append key append_value
  * 如果对integer的value追加string，则类型转为string
* 获取长度
  * strlen key

## hash
* h开头都是hash相关的数据结构的操作
* hexists / hget / hgetall / hkeys /
* hset map name jim - 创建一个名为map的哈希，并放入 name-jim
* hkeys map - 返回map所有的key
* hvals map - 返回map所有的value
* hgetall map - 返回map所有的键值对
* hlen map - 返回map的元素
* hmget map key1 key2 - 返回key1 key2的值
* hmset key1 value1 key2 value2
* hdel map key1 key2 ...
* hsetnx map keyname keyvalue 

## list
* l开头的都是链表相关的操作且是从左开始
* r开头的也是链表相关的操作且从右开始
* 从名为list的列表最左边开始逐个放入元素
  * lpush list 1 2 3 4 5 6 7 8 9 10
  * list: 10 9 8 7 6 5 4 3 2 1
  * rpush rlist 1 2 3 4 5 6 7 8 9 10
  * rlist: 1 2 3 4 5 6 7 8 9 10
* 链表长度
  * llen list
* 获取链表范围元素
  * lrange list left_limit right_limit 都是左闭右闭区间，不要和go搞混
* 设置链表pos位置的元素值为val
  * lset list pos val
* 获取pos元素值
  * lindex list pos
* 移除
  * lpop list - 移除list最左边的元素
  * rpop list

## set
* s开头的是set，且set是不可重复的。添加重复的元素会失败。
* 创建并添加
  * sadd set a b c d
* 获取set中元素数量
  * scard set
* rename set set1
* 查看set1中的元素
  * smembers set1
* sdiff set1 set2
  * 求差集 set1 - set2
* 求交集
  * sinter set1 set2
* 求并集
  * sunion set1 set2
* 返回set1的number个随机元素
  * srandmember set1 number
  * srandmember set1 2 - 随机两个元素返回
* 判断key的value是不是成员
  * sismember set1 a - 判断a是不是set1的成员
* 移出一个或多个set1成员
  * srem set1 member1 member2 ....
* 移除set2中一个**随机的**元素并返回
  * spop set2
  * 可以用作订单号池，生成一些不重复的订单号放在redis set中，需要生成订单号的时候直接从这里拿

## 有序集合 sortedset
* sortedset 是哈希表保证了o(1)，但是还可以保证顺序
* z开头的都是有序集合
* 添加和创建3个键 - 值对，其中键必须是数字来保证sortedset的顺序性
  * 也可以看作键 - 键对，第一个键是sortedset用来保证顺序的，第二个键是我们需要的内容
  * zadd sortedset 100 a 200 b 300 c
* zcard sortedset 返回sortedset元素个数
* 返回键为指定区间的元素
  * zcount sortedset left_limit right_limit
  * zcount sortedset 0 220 --> 返回2，即 100 200两个
* zrank key  返回key的索引值
  * zrank sortedset a --> 0，即排在第一个
* 增加键的大小
  * zincrby sortedset incr_value value
  * zincrby sortedset 1000 a --> a的顺序变成了最后一个(200 b 300 c 1100 a)
* 根据范围拿元素
  * zrange sortedset left right -- 只返回排序后的键
  * zrange sortedset left right withscores -- 同时返回值

# 注意
* key 都是string类型的，不论是数字还是字符串当key
* redis区间都是左闭右闭，不要和go搞混咯（go是左闭右开）

# Redis小技巧
1. redis-server不能通过Ctrl-C关闭，而应该通过redis-cli shutdown关闭  
使用redis-cli shutdown关闭redis-server中的数据会持久化到硬盘，否则会丢失。  
当然你也可以通过save命令持久化，但前者是个更好的习惯。

2. redis可以通过:来区别名字空间，例如：
```
set chneg:666:chnegkey chnegvalue  
```
就代表了chneg名字空间下的 666名字空间下的 chnegkey 对应的值是 chnegvalue。同理：
```  
set chneg:666:heikey keyvalue
```
此时heikey就和chnegkey是一个名字空间下的两个不同的key  
这个功能一定是一个非常好用的功能，不同的业务在同一个大的db（通过select指定的名字空间）下我们可以设计不同的名字空间来区别业务，查找也非常方便。但如果真的是要使用:则需要转义（/:）一下就可以了。

3. monitor 命令
* 该命令可以监视所有和redis通信的信息，排查问题的时候非常有用