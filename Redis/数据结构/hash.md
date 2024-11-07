# hash

## 命令

```sh
# 将 hash 表中 filed 设置为 value
# 如果给定的哈希表并不存在， 那么创建一个新的哈希表
# 如果 field 已经存在于哈希表中， 那么它的旧值将被新值覆盖
HSET key field value [field value ...]
HMSET key field value [field value ...]
# 不存在则创建
HSETNX key field value
# 为 field 的值加上增量，增量可以为负数
# hash 不存在则新建并初始化为 0
HINCRBY key field num
HINCRBYFLOAT key field num

HGET key field
HMGET key field [field ...]
# 获取 hash 表中 field 对应的字符串长度
HSTRLEN key field
# 获取所有 field
HKEYS key
# 获取 hash 中所有 value
HVALS key
# 获取所有 field-value
HGETALL key
# 获取 hash 表中 field 数量
HLEN key
# 同 SCAN
HSCAN key cursor [MATCH pattern] [COUNT count]
# 当只使用 key 参数时返回一个随机 key
# 如果 count 是正值则返回不同的 key
# 如果 count 是负值则允许返回相同的 key
# WITHVALUES 会将 k-v 同时返回
HRANDFIELD key [count [WITHVALUES]]

HEXISTS key field
HDEL key field [field ...]
```

## 底层编码

Hash 对象底层使用 `ziplist` 和 `dict`

- 当 Hash 对象中的元素数量较少且每个元素的键值对大小较小时，会使用 `ziplist` 作为底层数据结构。但是 `ziplist` 的查找效率相对较低，因为它需要遍历整个列表来查找特定的键值对
- 当 Hash 对象中的元素数量较多或每个元素的键值对较大时，会使用 `dict` 作为底层数据结构。`dict`提供了较高的查找效率，但是内存占用相对较高
- 当 Hash 对象中的元素数量或大小发生变化时，Redis 会自动调整底层数据结构以保持最佳性能

## 使用场景

1. 存储对象，不同于字符串需要序列化后才能存储对象，Hash 可以将对象各个字段单独存储
