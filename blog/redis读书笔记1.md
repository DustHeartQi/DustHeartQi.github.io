# Redis入门指南前三章读书笔记

此笔记的参照书籍为[Redis入门指南][1]，笔记只整理了前三章的内容。重点为redis的五中数据类型及其命令。
###目录
[TOC]


- **设计者：Salvatore Sanfilippo**

##特性
1.内存存储和持久化
	   Redis数据库中的所有数据都存储在内存中。由于内存的读写速度远快于硬盘，在一台普通的笔记本电脑上，Redis可以在一秒内读写超过十万个键值。
		将数据存储在内存中也有问题，例如，程序退出后内存中的数据会丢失。不过Redis提供了对持久化的支持，即可以将内存中的数据异步写入到硬盘中，同时不影响继续提供服务。


----------


2.功能丰富
1）可以为每个键设置生存时间，使得Redis可以作为缓存系统来使用。
2）Redis和缓存系统Memcached的优劣：Redis是单线程模型，而Memcached支持多线程，所以在多核服务器上后者性能更高一些。如果需要用到高级的数据类型或是持久化等功能，Redis将会是Memcached很好的替代品。
3）作为缓存系统，Redis还可以限定数据占用的最大内存空间，在数据达到空间限制后可以按照一定的规则自动淘汰不需要的键。
4）Redis的列表类型键可以用来实现队列，并且支持阻塞式读取，可以很容易地实现一个高性能的有限级队列。


----------
3.特点
1）Redis提供了一百多个命令，但是常用的却只有十几个，并且每个命令都很容易记忆
2）Redis提供了几十种不同编程语言的客户端库
3）Redis使用C语言开发，代码量只有3万多行，这降低了用户通过修改Redis源代码来使之更适合自己项目需要的门槛。
4）Redis是开源的。
5）包括INCR在内的所有Redis命令都是原子操作。


----------
4.命名要求

Redis对于键的命名并没有强制的要求，但比较好的实践是用“对象类型：对象ID：对象属性”来命名一个键，如使用键user:1:friends来存储ID为1的用户的好友列表。对于多个单词则推荐使用“.”分割。
在redis-cli中容易输入，无需使用双引号包裹。


----------


##数据类型
###字符串类型

1.自增，自减
incrby bar 2   //自动增加，增量为2，如果不是整数则出错
<2
incrby bar 3   //自动增加，增量为2
<5
incr   bar     //自动增加，增量为1，如果不是整数则出错
<6
decrby bar 2
decr bar

incrbyfloat bar 2.7  //增加指定浮点数

2.添加
set key hello   
mset key1 v1 key2 v2 key3 v3
append key " world!"  //追加，有引号是因为出现了空格

3.获取
strlen key   //返回键值的长度
mget key1 key3

4.位操作，一个字节八位（bit）
set foo bar
getbit foo 0   //返回二进制表示的foo索引为0的二进制的值
getbit foo 6   //返回二进制表示的foo索引为6的二进制的值
getbit foo 100000  //超出返回0 
setbit foo 6 0  //设置索引为6的值为0
bitcount foo    //统计二进制表示中1的个数
bitcount foo 0 1 //统计前两个字节中（fo）二进制表示中1的个数

set foo1 bar
set foo2 bar
bitop or res foo1 foo2   //结果保存到res中
get res


e:使用多个字符串类型键存储一个对象
post:42:title      第一篇日志
post:42:author     小白
post:42:time       2012年9月21日
post:42:content    今天是星期五，...


- 实践

1.文章访问量统计：
我们为每篇文章使用一个名为post：文章ID：page.view的键来记录文章的访问量。每次访问文章的时候使用INCR命令使用相应的键值递增
2.生成自增ID：
我们用user：count来存储当前类型对象的数量，没增加一个新对象时都使用INCR命令递增改键的值。
3.存储文章数据：
我们将一篇博客文章的标题、正文、作者与发布时间等多个元素使用序列化函数转换成一个字符串。
字符串类型键还可以存储二进制数据。MessagePack序列化的结果是二进制格式，占用空间更小。


----------


###散列类型
1.命令：
hset key field value
hget key field
hmset key f1 v1  f2 v2  f3 v3

hdel key f1 f2 f3
hdel car price

hmget key f1 f3
hgetall key  //获取key键中所有字段和值
hkeys key	//获取key中所有字段
hvals key	//获取key中所有值
hlen key	//获取key中字段数量

hexists key f1  //判断key键中f1字段是否存在
hsetnx key field value  //nx:if not exists  如果已经存在，不执行任何操作。不存在则写入。
hincrby person score 60



e:
hset car price 500
hset car name BMW
hget car name
hmget car price name
hgetall car
hset car model C200
hexists car model

- 实践

1.存储文章数据
e：使用一个散列类型键存储一个对象


post:32
: title 第一篇日志
: author 小白
: time	2012年9月11日
: content	见天是星期五，...


----------


###列表类型
可以实现文章的分页功能

1.命令：
lpush key v1 v2 v3 
rpush key v1 v2 v3
lpush numbers 2 3
lset numbers 1 7 //在索引值为1的位置添加7
linsert numbers after 7 3  //7后面插入3
linsert numbers before 2 1 //2前面插入1
lpop key
rpop key
rpoplpush lista listb  //从lista到listb


lrem key count value
lrem numbers -1 2  //从右边开始删除1个2
ltrim numbers 1 3  //只留下索引为1 2  3 （1到3）的值

llen key
llen numbers
lindex numbers 0   //获取number中索引值为0的值

lrange key start stop
lrange numbers 0 2
lrange numbers -2 -1


- 实践

存储文章ID列表：
我们使用列表类型键posts:list记录文章ID列表，当发布新文章时使用lpush命令把新文章的ID加入这个列表中，另外删除文章时也要记得把列表中的文章ID删除，就像这样：lrem post:list 1 文章id（删除id为<文章id>的文章）

缺点：
1）散列类型没有类似字符串类型的mget命令那样可以通过一条命令同时获得多个键的键值的版本，所以对于每个文章ID都需要请求一次数据库，也就都会产生一次往返时延，之后我们会介绍使用管道和脚本来优化这个问题。
2）修改文章时，需要修改post:文章ID  中的time字段，还需要按照实际的发布时间重新排列posts:list中的元素顺序，而这一操作相对比较繁琐。
3）文章数量较多时访问中间的页面性能较差。

存储评论列表：
在博客中如果不允许访客修改自己发表的评论，还可以考虑使用列表类型键存储文章的评论。将一条评论的各个元素序列化成字符串后作为列表类型键中的元素来存储。
我们使用列表类型键  post:文章ID:comments 来存储某个文章的所有评论。 


----------
###集合类型
	集合类型的常用操作时向集合中加入或删除元素、判断某个元素是否存在等，由于集合类型在Redis内部是使用值为空的散列表（hash table）实现的，所以这些操作的时间复杂度都是O（1）。最方便的是多个集合类型键之间还可以进行并集、交集和差集运算，稍后就会看到灵活运用这一特性带来的便利。

1.命令
sadd letters a 
sadd letters a b c
srem letters c d

smembers letters
sdiff seta setb setc  //差集
sinsert seta setb     //交集
sunion seta setb      //并集

sdiffstore destination key
sinterstore destination key
sunionstore destination key

srandmember letters  //随机从集合中获取一个元素
srandmember letters 3

- 实践

对每篇文章使用键名为 post:文章ID:tags 的键存储该篇文章的标签。
sadd post:42:tags,闲言碎语，技术文章，Java

找到同时属于“java”、“mysql”、和“redis”这三个标签的文章：三个集合取交集


----------
###有序集合类型
	有序集合类型在某些方面和列表类型有些相似。
1）二者都是有序的。
2）二者都可以获得某一范围的元素。


	但是二者有着很大的区别，这使得他们的应用场景也是不同的。
1）列表类型是通过链表实现的，获取靠近两端的数据速度极快，而当元素增多后，访问中间数据的速度回较慢，所以它更加适合实现如“新鲜事”或“日志”这样很少访问中间元素的应用。
2）有序集合类型是使用散列表和跳跃表实现的，所以即使读取位于中间部分的数据速度也会很快（时间复杂度是O（log（N）））。
3）列表中不能简单地调整某个元素的位置，但是有序集合可以（通过更改这个元素的分数）。
4）有序集合要比列表类型更耗费内存。

1.命令：
zadd scoreboard 89 tom 67 peter 100 david
zscore scoreboard tom
zrange scoreboard 0 2

zrange scoreboard 0 -1 withscores

zrangebyscore scoreboard 80 100  //80 到100之间
zrangebyscore scoreboard 80 (100 //不包括100
zrangebyscore scoreboard (80 +inf  //80以上
zcount scoreboard 90 100
zrank scoreboard peter   //获取peter的排名，升序
zrevrank scoreboard peter  //降序

zrangebyscore scoreboard 60 +inf limit 1 3  //分数高于60的第二个人开始的3个人。
zrenrangebyscore scoreboard 100 0 limit 0 3  //分数小于100

zincrby scoreboard 4 jerry   //jerry 加4
zincrby socreboard -4 jerry  //jerry 减4


- 实践
通过有序集合可以实现文章按照点击量排序。


---------
[1]: https://github.com/benweet/stackedit