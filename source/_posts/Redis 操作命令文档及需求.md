---
title: Redis 操作命令文档及需求
date: 2025-04-06 21:35:10
categories: redis
tags: 
 - redis
---

# Redis 操作命令文档

Redis 是一个高性能的键值存储数据库，支持多种数据结构，包括字符串（String）、哈希（Hash）、列表（List）、集合（Set）、有序集合（Sorted Set），以及位图（Bitmap）、HyperLogLog 和地理空间索引（Geospatial）等扩展数据结构。本文档将详细列出这些数据结构的常用命令，并探讨它们在生产环境中的典型应用场景。

---

## 1. 字符串（String）
字符串是 Redis 最基本的数据类型，可以存储文本、数字或二进制数据。常用于缓存、计数器、分布式锁等场景。

| 命令                       | 描述                       | 示例                        | 生产应用场景            |
|----------------------------|----------------------------|-----------------------------|-------------------------|
| `SET key value`            | 设置键值对                 | `SET name "Alice"`          | 缓存用户信息、配置项    |
| `GET key`                  | 获取键的值                 | `GET name`                  | 读取缓存数据            |
| `APPEND key value`         | 追加值到键的末尾           | `APPEND name " Smith"`      | 日志记录、消息拼接      |
| `INCR key`                 | 将键的值加 1（适用于整数） | `INCR counter`              | 计数器（如页面访问量）  |
| `DECR key`                 | 将键的值减 1               | `DECR counter`              | 库存扣减                |
| `INCRBY key increment`     | 增加指定增量               | `INCRBY counter 10`         | 批量增加计数            |
| `DECRBY key decrement`     | 减少指定减量               | `DECRBY counter 5`          | 批量减少计数            |
| `GETRANGE key start end`   | 获取子字符串               | `GETRANGE name 0 4`         | 提取部分数据            |
| `SETNX key value`          | 仅在键不存在时设置值       | `SETNX name "Bob"`          | 分布式锁                |
| `MSET key1 value1 ...`     | 批量设置键值对             | `MSET k1 "v1" k2 "v2"`      | 批量缓存数据            |
| `MGET key1 key2 ...`       | 批量获取键的值             | `MGET k1 k2`                | 批量读取缓存            |
| `STRLEN key`               | 获取值的长度               | `STRLEN name`               | 验证数据完整性          |

**生产应用场景**：
- **缓存**：存储数据库查询结果、API 响应等，减少数据库压力。
- **分布式锁**：使用 `SETNX` 实现互斥锁，防止并发冲突。
- **计数器**：如网站访问量、点赞数，Redis 的原子操作保证计数的准确性。

---

## 2. 哈希（Hash）
哈希适合存储对象，键值对形式类似于字段和值的映射。常用于用户信息、配置信息等。

| 命令                       | 描述                       | 示例                          | 生产应用场景            |
|----------------------------|----------------------------|-------------------------------|-------------------------|
| `HSET key field value`     | 设置哈希字段的值           | `HSET user:1 name "Alice"`    | 存储用户对象            |
| `HGET key field`           | 获取哈希字段的值           | `HGET user:1 name`            | 读取用户属性            |
| `HMSET key field1 value1 ...` | 批量设置字段值          | `HMSET user:1 name "Alice" age "25"` | 批量更新对象     |
| `HMGET key field1 field2 ...` | 批量获取字段值          | `HMGET user:1 name age`       | 批量读取属性            |
| `HGETALL key`              | 获取哈希所有字段和值       | `HGETALL user:1`              | 获取完整对象            |
| `HDEL key field1 ...`      | 删除指定字段               | `HDEL user:1 age`             | 删除对象属性            |
| `HEXISTS key field`        | 检查字段是否存在           | `HEXISTS user:1 name`         | 验证属性存在            |
| `HKEYS key`                | 获取哈希所有字段名         | `HKEYS user:1`                | 列出对象字段            |
| `HVALS key`                | 获取哈希所有值             | `HVALS user:1`                | 列出对象值              |
| `HLEN key`                 | 获取哈希字段数量           | `HLEN user:1`                 | 获取对象字段数          |

**生产应用场景**：
- **用户 Session**：存储用户会话数据，支持快速访问和更新。
- **配置管理**：存储应用的配置项，支持动态修改。
- **购物车**：字段为商品 ID，值为数量，便于管理。

---

## 3. 列表（List）
列表是一个有序的字符串集合，支持从两端操作（双端队列）。常用于消息队列、任务队列等。

| 命令                       | 描述                       | 示例                        | 生产应用场景            |
|----------------------------|----------------------------|-----------------------------|-------------------------|
| `LPUSH key value1 ...`     | 从左侧插入元素             | `LPUSH list "a" "b"`        | 消息队列入队            |
| `RPUSH key value1 ...`     | 从右侧插入元素             | `RPUSH list "c" "d"`        | 任务队列入队            |
| `LPOP key`                 | 从左侧弹出元素             | `LPOP list`                 | 消息队列出队            |
| `RPOP key`                 | 从右侧弹出元素             | `RPOP list`                 | 任务队列出队            |
| `LRANGE key start end`     | 获取指定范围的元素         | `LRANGE list 0 -1`          | 获取队列内容            |
| `LLEN key`                 | 获取列表长度               | `LLEN list`                 | 检查队列长度            |
| `LINDEX key index`         | 获取指定索引的元素         | `LINDEX list 1`             | 访问特定元素            |
| `LREM key count value`     | 删除指定值的元素           | `LREM list 1 "a"`           | 移除重复消息            |
| `LSET key index value`     | 设置指定索引的值           | `LSET list 0 "x"`           | 更新队列元素            |
| `LTRIM key start end`      | 修剪列表到指定范围         | `LTRIM list 1 2`            | 保留最新 N 条记录       |

**生产应用场景**：
- **消息队列**：使用 `LPUSH` 和 `RPOP` 实现 FIFO 队列。
- **任务队列**：存储待处理任务，工作者从队列中取任务。
- **最新列表**：如最新评论，保持固定长度，移除旧数据。

---

## 4. 集合（Set）
集合是无序、不重复的字符串集合。常用于去重、关系运算、随机抽取等。

| 命令                       | 描述                       | 示例                        | 生产应用场景            |
|----------------------------|----------------------------|-----------------------------|-------------------------|
| `SADD key member1 ...`     | 添加元素到集合             | `SADD set "a" "b"`          | 添加用户标签            |
| `SREM key member1 ...`     | 删除集合中的元素           | `SREM set "a"`              | 移除用户标签            |
| `SMEMBERS key`             | 获取集合所有元素           | `SMEMBERS set`              | 列出用户标签            |
| `SISMEMBER key member`     | 检查元素是否在集合中       | `SISMEMBER set "b"`         | 检查用户标签            |
| `SCARD key`                | 获取集合元素数量           | `SCARD set`                 | 统计标签数量            |
| `SINTER key1 key2 ...`     | 求多个集合的交集           | `SINTER set1 set2`          | 共同好友                |
| `SUNION key1 key2 ...`     | 求多个集合的并集           | `SUNION set1 set2`          | 合并标签                |
| `SDIFF key1 key2 ...`      | 求多个集合的差集           | `SDIFF set1 set2`           | 独有标签                |
| `SPOP key`                 | 随机弹出一个元素           | `SPOP set`                  | 随机抽奖                |
| `SRANDMEMBER key [count]`  | 随机获取元素               | `SRANDMEMBER set 2`         | 随机推荐                |

**生产应用场景**：
- **去重**：存储用户 ID，防止重复操作。
- **关系运算**：计算共同关注、共同好友等。
- **随机抽取**：用于抽奖系统或随机推荐。

---

## 5. 有序集合（Sorted Set）
有序集合是有序、不重复的字符串集合，每个元素关联一个分数（score）。常用于排行榜、延迟任务等。

| 命令                       | 描述                       | 示例                          | 生产应用场景            |
|----------------------------|----------------------------|-------------------------------|-------------------------|
| `ZADD key score1 member1 ...` | 添加元素及分数          | `ZADD zset 1 "a" 2 "b"`       | 添加用户分数            |
| `ZRANGE key start end [WITHSCORES]` | 按分数升序获取元素 | `ZRANGE zset 0 -1 WITHSCORES` | 获取排行榜             |
| `ZREVRANGE key start end [WITHSCORES]` | 按分数降序获取元素 | `ZREVRANGE zset 0 -1`        | 获取倒序排行            |
| `ZREM key member1 ...`     | 删除元素                   | `ZREM zset "a"`              | 移除用户                |
| `ZCARD key`                | 获取元素数量               | `ZCARD zset`                 | 统计用户数              |
| `ZSCORE key member`        | 获取元素的分数             | `ZSCORE zset "b"`            | 查询用户分数            |
| `ZINCRBY key increment member` | 增加元素的分数         | `ZINCRBY zset 1.5 "b"`        | 更新用户分数            |
| `ZRANK key member`         | 获取元素升序排名           | `ZRANK zset "b"`             | 查询用户排名            |
| `ZREVRANK key member`      | 获取元素降序排名           | `ZREVRANK zset "b"`          | 查询倒序排名            |
| `ZRANGEBYSCORE key min max [WITHSCORES]` | 按分数范围获取元素 | `ZRANGEBYSCORE zset 1 5`     | 范围查询               |

**生产应用场景**：
- **排行榜**：如游戏积分榜、商品销量榜。
- **延迟任务**：score 为执行时间，定时轮询。
- **范围查询**：按时间或分数段查询数据。

---

## 6. 位图（Bitmap）
位图通过字符串的位操作实现，适合存储大量布尔值，节省空间。常用于在线状态、签到记录等。

| 命令                       | 描述                       | 示例                        | 生产应用场景            |
|----------------------------|----------------------------|-----------------------------|-------------------------|
| `SETBIT key offset value`  | 设置指定偏移量的位值       | `SETBIT bitmap 7 1`         | 设置用户在线状态        |
| `GETBIT key offset`        | 获取指定偏移量的位值       | `GETBIT bitmap 7`           | 检查用户在线状态        |
| `BITCOUNT key [start end]` | 统计值为 1 的位数          | `BITCOUNT bitmap`           | 统计在线用户数          |
| `BITOP operation destkey key1 ...` | 位运算（如 AND、OR） | `BITOP AND result k1 k2`    | 用户行为分析            |

**生产应用场景**：
- **在线状态**：offset 为用户 ID，记录在线状态。
- **签到记录**：offset 为天数，记录签到情况。
- **布隆过滤器**：实现布隆过滤器，快速判断元素是否存在。

---

## 7. HyperLogLog
HyperLogLog 用于基数统计（近似去重计数），占用固定空间，适合大数据量去重。

| 命令                       | 描述                       | 示例                        | 生产应用场景            |
|----------------------------|----------------------------|-----------------------------|-------------------------|
| `PFADD key element1 ...`   | 添加元素                   | `PFADD hll "a" "b"`         | 记录用户访问            |
| `PFCOUNT key1 key2 ...`    | 获取基数估计值             | `PFCOUNT hll`               | 统计独立用户数          |
| `PFMERGE destkey key1 ...` | 合并多个 HyperLogLog       | `PFMERGE hll3 hll1 hll2`    | 合并统计数据            |

**生产应用场景**：
- **UV 统计**：统计网站独立访客数。
- **去重计数**：如独立 IP 数、设备数。
- **大数据去重**：在内存受限时进行大规模去重。

---

## 8. 地理空间索引（Geospatial）
用于存储地理位置并计算距离，适合 LBS（基于位置的服务）应用。

| 命令                       | 描述                       | 示例                          | 生产应用场景            |
|----------------------------|----------------------------|-------------------------------|-------------------------|
| `GEOADD key longitude latitude member` | 添加地理位置   | `GEOADD cities 13.36 52.52 "Berlin"` | 添加城市位置     |
| `GEOPOS key member1 ...`   | 获取坐标                   | `GEOPOS cities "Berlin"`     | 查询城市坐标            |
| `GEODIST key member1 member2 [unit]` | 计算两点距离     | `GEODIST cities "Berlin" "Paris" km` | 计算城市间距离   |
| `GEORADIUS key longitude latitude radius unit` | 查找范围内的位置 | `GEORADIUS cities 13.36 52.52 100 km` | 查找附近城市   |

**生产应用场景**：
- **附近的人**：查找附近的用户。
- **门店推荐**：根据位置推荐附近门店。
- **路径规划**：计算两点距离，辅助规划。

---

## 9. 通用命令（适用于所有数据结构）
这些命令用于管理 Redis 键的生命周期和基本操作。

| 命令                       | 描述                       | 示例                        | 生产应用场景            |
|----------------------------|----------------------------|-----------------------------|-------------------------|
| `DEL key1 key2 ...`        | 删除键                     | `DEL name`                  | 清理缓存                |
| `EXISTS key`               | 检查键是否存在             | `EXISTS name`               | 验证数据存在            |
| `TYPE key`                 | 获取键的数据类型           | `TYPE name`                 | 调试和监控              |
| `EXPIRE key seconds`       | 设置键的过期时间           | `EXPIRE name 60`            | 设置缓存过期            |
| `TTL key`                  | 获取键的剩余生存时间       | `TTL name`                  | 检查缓存有效期          |
| `PERSIST key`              | 移除键的过期时间           | `PERSIST name`              | 取消过期设置            |
| `RENAME key newkey`        | 重命名键                   | `RENAME name new_name`      | 更新键名                |
| `KEYS pattern`             | 查找匹配模式的键           | `KEYS user:*`               | 批量操作键              |

**生产应用场景**：
- **缓存管理**：设置过期时间，自动清理数据。
- **数据清理**：定期删除无用数据。
- **监控**：检查键的状态和类型。

---

# 生产中的高级需求和解决方案

以下是 Redis 在生产环境中常见的高级需求及其解决方案：

### 1. 缓存穿透
- **问题**：恶意请求查询不存在的键，导致数据库压力过大。
- **解决方案**：使用布隆过滤器（基于位图实现）预先过滤不存在的键。

### 2. 缓存雪崩
- **问题**：大量缓存同时过期，导致数据库压力剧增。
- **解决方案**：设置随机过期时间，避免同时失效；使用多级缓存。

### 3. 分布式锁
- **问题**：分布式系统中需要互斥访问资源。
- **解决方案**：使用 `SETNX` 实现锁，结合 `EXPIRE` 设置超时。

### 4. 消息队列
- **问题**：需要异步处理任务或消息。
- **解决方案**：使用列表实现队列，支持阻塞操作（如 `BLPOP`）。

### 5. 排行榜
- **问题**：需要实时更新和查询排行榜。
- **解决方案**：使用有序集合，score 作为排名依据。

### 6. 限流
- **问题**：限制接口访问频率，防止攻击。
- **解决方案**：使用计数器或滑动窗口算法实现。

### 7. 会话管理
- **问题**：分布式系统中管理用户会话。
- **解决方案**：将 session 存储在 Redis 中，支持过期管理。
