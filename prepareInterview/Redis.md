# Redis

## Redis为什么这么快

Redis 将数据储存在内存里面，读写数据的时候都不会受到硬盘 I/O 速度的限制，所以速度极快

1. 使用简单的数据结构
2. 基于内存
3. 使用IO多路复用模型，且非阻塞
4. 自己实现了VM
5. 单线程，避免了不必要的上下文切换和竞争条件







缓存雪崩：大量的key在一瞬间失效，导致所有请求都打到了db

缓存穿透：使用不存在的key发大量的请求，导致所有请求落到db上

缓存击穿：热点数据忽然失效，导致这个热点数据的查询瞬间落到db上

- Redis 有哪些类型

- - String
  - List
  - Hash
  - Set
  - ZSet
  - **HyperLogLog**
  - **Geo**
  - **Pub/Sub**
  - **Redis Module****：BloomFilter**

- Redis 内部结构

- 聊聊 Redis 使用场景

- Redis 持久化机制

- Redis 如何实现持久化

- Redis 集群方案与实现

- Redis 为什么是单线程的

- 缓存奔溃

- 缓存降级

- 使用缓存的合理性问题