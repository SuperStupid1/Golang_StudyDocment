# 📚 Redis 面试题精选

## 🔑 基础概念与数据类型

### 1. Redis 的特点和应用场景？
**答案：**
Redis 是一个开源的内存数据结构存储系统，具有以下特点：
1. 高性能：基于内存，单线程模型，10w+ QPS
2. 数据结构丰富：支持 String、Hash、List、Set、Sorted Set 等
3. 原子性：所有操作都是原子性的
4. 持久化：支持 RDB 和 AOF
5. 主从复制：支持数据备份和读写分离

**应用场景：**
1. 缓存系统（热点数据、会话信息）
2. 计数器（文章阅读量、用户点赞数）
3. 消息队列（基于 List 或 Stream）
4. 排行榜（基于 Sorted Set）
5. 分布式锁（基于 SETNX）

### 2. Redis 数据类型的最佳实践？
**答案：**

| 数据类型 | 使用场景 | 实现原理 | 注意事项 |
|----------|----------|----------|----------|
| String | 缓存、计数器 | 动态字符串 | 大 key 问题 |
| Hash | 对象存储 | 哈希表 | 字段数量 |
| List | 消息队列 | 双向链表 | 长度控制 |
| Set | 去重、关系 | 哈希表 | 集合运算开销 |
| Sorted Set | 排行榜 | 跳表+哈希表 | score 精度 |

**代码示例：**
```bash
# String 计数器
INCR page_view

# Hash 存储用户信息
HMSET user:1001 name "Tom" age "20" 

# List 实现队列
LPUSH queue:tasks "task1"
RPOP queue:tasks

# Set 存储关注关系
SADD following:1001 1002 1003

# Sorted Set 实现排行榜
ZADD leaderboard 99.5 user:1001
```

## 💾 持久化机制

### 3. Redis 持久化方案的选择？
**答案：**

**RDB（快照）：**
1. 原理：某一时刻的完整数据快照
2. 优点：
   - 文件紧凑，适合备份
   - 恢复速度快
   - 子进程备份，影响小
3. 缺点：
   - 可能丢失最后一次快照后的数据
   - 大数据集时 fork() 耗时

**AOF（日志）：**
1. 原理：记录所有写操作命令
2. 优点：
   - 数据安全性高
   - 可读性强，便于恢复
3. 缺点：
   - 文件体积大
   - 恢复速度慢

**最佳实践：**
```bash
# redis.conf
save 900 1      # 900秒内至少1个key变化
save 300 10     # 300秒内至少10个key变化
save 60 10000   # 60秒内至少10000个key变化

appendonly yes
appendfsync everysec
```

## 🔄 缓存策略

### 4. Redis 缓存更新策略？
**答案：**

**Cache-Aside（旁路缓存）：**
```python
def get_user(user_id):
    user = redis.get(f"user:{user_id}")
    if not user:
        user = db.query(user_id)
        redis.setex(f"user:{user_id}", 3600, user)
    return user
```

**Write-Through（读写穿透）：**
```python
def update_user(user_id, data):
    db.update(user_id, data)
    redis.setex(f"user:{user_id}", 3600, data)
```

**Write-Behind（异步写入）：**
```python
def update_user(user_id, data):
    redis.setex(f"user:{user_id}", 3600, data)
    async_task.delay(update_db, user_id, data)
```

### 5. 缓存穿透、击穿、雪崩的解决方案？
**答案：**

| 问题 | 描述 | 解决方案 |
|------|------|----------|
| 缓存穿透 | 查询不存在的数据 | 布隆过滤器、空值缓存 |
| 缓存击穿 | 热点 key 过期 | 互斥锁、永不过期 |
| 缓存雪崩 | 大量 key 同时过期 | 随机过期时间、多级缓存 |

**布隆过滤器示例：**
```python
from bloom_filter import BloomFilter

bloom = BloomFilter(capacity=10000, error_rate=0.001)
bloom.add("user:1001")

def get_user(user_id):
    if f"user:{user_id}" not in bloom:
        return None
    # 继续查询缓存和数据库
```

## 🔒 分布式锁

### 6. Redis 分布式锁的实现？
**答案：**

**基本实现：**
```python
def acquire_lock(lock_key, expire_seconds):
    return redis.set(lock_key, uuid.uuid4(), nx=True, ex=expire_seconds)

def release_lock(lock_key, lock_value):
    lua_script = """
    if redis.call('get', KEYS[1]) == ARGV[1] then
        return redis.call('del', KEYS[1])
    else
        return 0
    end
    """
    return redis.eval(lua_script, 1, lock_key, lock_value)
```

**Redlock 算法：**
1. 获取当前时间
2. 依次向 N 个节点获取锁
3. 计算获取锁消耗的时间
4. 锁的有效性检查
5. 释放锁（向所有节点发送释放命令）

## 🌐 集群架构

### 7. Redis 集群方案对比？
**答案：**

**主从复制：**
1. 配置简单：`slaveof master_ip master_port`
2. 数据一致性：异步复制
3. 故障转移：需要外部支持

**哨兵模式：**
1. 自动故障检测和转移
2. 客户端支持
3. 配置：
```bash
sentinel monitor mymaster 127.0.0.1 6379 2
sentinel down-after-milliseconds mymaster 5000
```

**Cluster 模式：**
1. 数据自动分片
2. 去中心化
3. 配置：
```bash
cluster-enabled yes
cluster-node-timeout 5000
cluster-config-file nodes.conf
```

### 8. Redis Cluster 数据分片原理？
**答案：**
1. 槽位分配：16384 个槽位
2. 节点负责一定范围的槽位
3. CRC16 计算 key 的槽位
4. 跨槽操作限制

**示例：**
```python
def get_slot(key):
    return crc16(key) % 16384

def get_node(key):
    slot = get_slot(key)
    return slot_node_mapping[slot]
```

## 📈 性能优化

### 9. Redis 性能优化方案？
**答案：**

**内存优化：**
1. 合理的数据结构
2. 设置过期时间
3. maxmemory 和淘汰策略

**命令优化：**
1. 批量操作（Pipeline）
2. 禁用慢命令（keys、flushall）
3. Lua 脚本优化

**架构优化：**
1. 读写分离
2. 数据分片
3. 使用连接池

**监控指标：**
```bash
# 内存指标
info memory

# 性能指标
info stats

# 慢查询
slowlog get 10
```

### 10. Redis 大 key 处理方案？
**答案：**

**发现大 key：**
```bash
redis-cli --bigkeys
debug object <key>
```

**处理方案：**
1. 拆分大 key：
```python
# 拆分大 hash
for field in redis.hscan_iter("big_hash"):
    redis.hset(f"small_hash:{field}", field, value)
```

2. 压缩数据：
```python
import zlib
compressed_data = zlib.compress(raw_data)
redis.set("compressed_key", compressed_data)
```

3. 采用合适的数据结构

## 🔧 运维实践

### 11. Redis 常见问题排查？
**答案：**

**延迟问题：**
1. 使用 latency monitor
2. 检查网络延迟
3. 排查慢命令

**内存问题：**
1. 监控内存使用
2. 分析内存碎片
3. 定期清理过期 key

**CPU 问题：**
1. 排查慢查询
2. 优化命令使用
3. 控制并发连接

### 12. Redis 备份与恢复策略？
**答案：**

**备份策略：**
1. RDB 文件备份
2. AOF 文件备份
3. 从节点备份

**恢复流程：**
```bash
# RDB 恢复
cp dump.rdb /var/lib/redis/
redis-server restart

# AOF 恢复
redis-check-aof --fix appendonly.aof
redis-server restart
```

## 🚀 高级特性

### 13. Redis 事务实现原理？
**答案：**

**MULTI/EXEC 实现：**
```python
def transfer(from_user, to_user, amount):
    pipe = redis.pipeline(transaction=True)
    pipe.multi()
    try:
        pipe.hincrby(f"user:{from_user}", "balance", -amount)
        pipe.hincrby(f"user:{to_user}", "balance", amount)
        pipe.execute()
        return True
    except Exception as e:
        pipe.discard()
        return False
```

**特点：**
1. 命令排队执行
2. 不支持回滚
3. 乐观锁实现（WATCH）

### 14. Redis Stream 消息队列？
**答案：**

**特点：**
1. 持久化消息队列
2. 消费组模式
3. 消息确认机制

**示例：**
```python
# 生产者
redis.xadd("mystream", {"sensor_id": 1234, "temperature": 19.8})

# 消费者组
redis.xgroup_create("mystream", "mygroup", "$")
while True:
    messages = redis.xreadgroup("mygroup", "consumer1", {"mystream": ">"}, count=1)
    process_messages(messages)
    redis.xack("mystream", "mygroup", message_id)
```

## 🔮 Redis 6.0+ 新特性

### 15. Redis 6.0 多线程模型？
**答案：**

**特点：**
1. IO 多线程
2. 命令执行仍然单线程
3. 提升网络性能

**配置：**
```bash
io-threads 4
io-threads-do-reads yes
```

**最佳实践：**
1. 根据 CPU 核心数配置线程
2. 监控线程性能
3. 适用于网络 IO 密集场景

### 16. Redis 新数据类型使用？
**答案：**

**Streams：**
- 消息队列的完整实现
- 消费组管理
- 消息持久化

**RedisJSON：**
- 原生 JSON 支持
- 路径查询
- 原子操作

**RedisTimeSeries：**
- 时间序列数据
- 自动降采样
- 聚合计算

## 📊 监控与告警

### 17. Redis 监控指标有哪些？
**答案：**

**核心指标：**
1. 性能指标：
   - 操作延迟
   - QPS
   - 连接数

2. 内存指标：
   - 使用率
   - 碎片率
   - 淘汰次数

3. 复制指标：
   - 复制延迟
   - 连接状态

**监控工具：**
```bash
# Prometheus + Redis Exporter
grafana_dashboard:
  - redis_memory
  - redis_performance
  - redis_replication
```

## 🎯 实战案例

### 18. 如何设计一个商品库存系统？
**答案：**

**架构设计：**
1. Redis + MySQL 双写
2. 预扣库存
3. 异步确认

**代码实现：**
```python
def deduct_stock(product_id, quantity):
    lua_script = """
    local stock = redis.call('get', KEYS[1])
    if not stock or tonumber(stock) < tonumber(ARGV[1]) then
        return 0
    end
    redis.call('decrby', KEYS[1], ARGV[1])
    return 1
    """
    return redis.eval(lua_script, 1, f"stock:{product_id}", quantity)
```

### 19. 如何实现一个排行榜系统？
**答案：**

**功能特点：**
1. 实时更新
2. 分数排名
3. 获取前 N 名

**实现方案：**
```python
# 更新分数
redis.zadd("leaderboard", {"player:1001": 89.5})

# 获取排名
rank = redis.zrevrank("leaderboard", "player:1001")

# 获取前 N 名
top_n = redis.zrevrange("leaderboard", 0, 9, withscores=True)
```

### 20. 如何实现一个限流系统？
**答案：**

**算法选择：**
1. 固定窗口
2. 滑动窗口
3. 令牌桶

**实现示例：**
```python
def sliding_window_ratelimit(user_id, window_size=60, max_requests=10):
    lua_script = """
    local key = KEYS[1]
    local now = tonumber(ARGV[1])
    local window = tonumber(ARGV[2])
    local max_req = tonumber(ARGV[3])
    
    redis.call('zremrangebyscore', key, 0, now - window)
    local count = redis.call('zcard', key)
    
    if count < max_req then
        redis.call('zadd', key, now, now .. '-' .. math.random())
        return 1
    end
    return 0
    """
    return redis.eval(lua_script, 1, f"ratelimit:{user_id}", time.time(), window_size, max_requests)
```

