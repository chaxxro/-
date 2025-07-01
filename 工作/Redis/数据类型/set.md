# set

Set 是一种无序的字符串集合数据类型，它允许存储唯一的字符串元素

## 命令

```sh
# 将元素添加进集合
SADD key member [member ...]

# 随机移除
SPOP key [count]
# 移除元素
SREM key member [member ...]
# 返回随机元素
# count 正数则不允许重复
# count 负数则允许重复
SRANDMEMBER key [count]
# 将 src 元素 member 移动到 dst
SMOVE source destination member

# 获取集合元素数量
SCARD key
# 检测元素是否在集合中
SISMEMBER key member
SMISMEMBER key member [member ...]
# 获取集合所有元素
SMEMBERS key
# 见 SCAN
SSCAN key cursor [MATCH pattern] [COUNT count]

# 返回第一个 key 与其他 key 的差值
SDIFF key [key ...]
# 将差值存在 destination
SDIFFSTORE destination key [key ...]

# 交集
SINTER key [key ...]
SINTERSTORE destination key [key ...]

# 并集
SUNION key1 key2...
SUNIONSTORE destination key1 key2
```

## 底层编码

- 当 Set 中的元素都是整数且数量较少时，Redis 使用 `intset` 数据结构来实现 Set。`intset` 是一种紧凑的数据结构，它将所有的整数元素存储在一个连续的内存区域中，这样可以节省内存空间。`intset` 支持高效的整数比较和查找操作

- 当 Set 中的元素数量较多或包含非整数元素时，Redis 使用 `dict` 数据结构来实现 Set。`dict` 提供了高效的查找、插入和删除操作，但内存占用相对较高。

- 当 Set 中的元素类型或数量发生变化时，Redis 会自动调整底层数据结构以保持最佳性能

## 使用场景

- 标签系统
- 社交网络关系
