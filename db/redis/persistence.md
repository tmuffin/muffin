#### Redis是一个内存数据库，为了保证数据的持久化，redis提供了两种持久化方式RDB和AOF，下面我们就分别来看下这两种持久化方式的实现原理
##### RDB(默认)
- RDB是通过快照方式完成的，当满足一定条件时，redis会自动将内存中的数据持久化到磁盘
- 触发快照的时机
  - 符合自定义配置的快照规则。(在redis.conf中配置，下面会详细介绍)
  - 执行save或者bgsave命令
  - 执行flushall命令
  - 执行主从复制操作(第一次)
##### RDB的优缺点
- 缺点: 使用RDB进行持久化，在redis突然异常退出的时候，会丢失最后一次快照之后的数据。但是，可以根据组合设置自动快照的方式，降低数据损失，确保在接受范围内。如果数据较为重要，可以使用AOF方式
- 优点: 使用RDB方式可以最大化redis性能，在快照过程中，我们可以看到主进程只需要fork出一个子进程即可，剩下的工作全部由子进程完成，父进程无需进行任何的磁盘I/O操作。但是，如果数据集较大，在fork子进程的时候比较耗时，会导致redis在一段时间内停止处理请求。
##### AOF方式
- 默认情况下，redis是没有开启AOF(append only file)的
- 在开始AOF之后，redis每接收到一条更改redis数据的命令，就会将该命令写入硬盘中的AOF文件
- 很明显，该过程会降低redis的性能，但大部分情况下是能够接受的，同时，使用性能较好的硬盘可以提高AOF性能
##### RESP协议
Redis客户端使用RESP协议与Redis服务端进行通信，该协议是专门为redis设计的，但是也可以用于其他C-S项目
- 间隔符号:在linux中是\r\n,在windows中是\n
- 简单字符串以'+'开头
- 错误Errors，以'-'开头
- 整数类型Integer,以':'开头
- 大字符串以'$'开头
- 数组类型Arrays,以'*'开头

##### AOF原理说明
- redis通过将所有的写入命令记录到AOF文件中，来持久化数据
- 而将命令记录到AOF文件的过程，可以分成三个阶段:
  - 命令传播
  - 缓存追加
  - 文件写入和保存
  
....