# sortedset

Redis 中的  SortedSet 是一个有序集合数据类型，它允许存储唯一的字符串元素，并且每个元素都有一个分数 score 与之关联，并且这个 score 是一个 `double` 类型

1. 元素是有序的，它们根据分数进行递增排序
2. 元素必须是唯一的，不能有重复的元素
3. 支持添加、删除、查找和检查元素是否存在等操作
4. 当不同元素有相同 score 时，会按字典序排列

## 命令

```sh
# 添加元素，score 可以是整数和双精度浮点数
# 如果 member 已存在则更新 score 并重新排序
# XX 只在 member 已存在时生效
# NX 只在 member 不存在时生效
# LT 只在 member 已存在时生效，并且新 score 必须小于旧 score
# GT 只在 member 已存在时生效，并且新 score 必须大于旧 score
# CH 命令返回值由设置的元素个数变成更新了 score 的元素个数
# INCR 等同于 ZINCRBY 且只支持一个 member
ZADD key [NX|XX] [GT|LT] [CH] [INCR] score member [score member ...]
# 权重递增，increment 可以是负数
# member 不存在时，权重默认为 0
ZINCRBY key increment member

# 移除
ZREM key member [member ...]
ZREMRANGEBYLEX key min max
ZREMRANGEBYRANK key start stop
ZREMRANGEBYSCORE key min max
# 删除最大、最小 score 的元素
ZPOPMAX key [count]
ZPOPMIN key [count]
# 阻塞版本
BZPOPMAX key [count]
BZPOPMIN key [count]

# 获取权重
ZSCORE key member
ZMSCORE key member [member ...]
# 获取递增序列索引
ZRANK key member
# 获取递减序列索引
ZREVRANK key member
# 获取数量
ZCARD key
# 获取闭区间内元素个数
ZCOUNT key min max
# 获取字典序区间个数，只能用于所有元素的 score 相同的情况
# min、max 同 ZRANGE
ZLEXCOUNT key min max
# 获取随机元素
# count > 0 则不允许重复数据
# count < 0 允许重复数据
ZRANDMEMBER key [count [WITHSCORES]]

# 区间获取
# 默认情况下 [min, max] 表示的是索引区间，索引区间支持负值，-1 表示最后一个元素
# BYSCORE 指明获取 score 在 [min, max] 区间内的元素，此时 min、max 支持 -inf 和 +inf
# BYSCORE 区间默认是闭区间，但可以使用 ( 表示开区间，(1 5 表示  1 < score <= 5，(5 (10 表示 5 < score < 10
# BYLEX 只能用于所有元素的 score 相同的情况，否则返回结果未定义
# 合法的 min、max 必须以 ( 或 [ 开头，以指明区间开闭
# 特殊符号支持 - 和 +，- 表示 -inf，+ 表示 +inf
# REV 反转输出结果
# LIMIT 等同于 sql 中的 limit，如果 count 小于 0 表示获取从 offset 开始的全部元素
# WITHSCORES 表示返回结果时带上 score
ZRANGE key min max [BYSCORE|BYLEX] [REV] [LIMIT offset count] [WITHSCORES]
# 结果存储在 dst
ZRANGESTORE dst src min max [BYSCORE|BYLEX] [REV] [LIMIT offset count]
# 递减输出
ZREVRANGE key start stop [WITHSCORES]
# 等同于 ZRANGE key min max BYSCORE...
ZRANGEBYSCORE key min max [WITHSCORES] [LIMIT offset count]
ZREVRANGEBYSCORE key max min [WITHSCORES] [LIMIT offset count]
# 等同于 ZRANGE key min max BYLEX...
ZRANGEBYLEX key min max [LIMIT offset count]
ZREVRANGEBYLEX key max min [LIMIT offset count]

# 并集
# numkeys 参数指定 key 的数量
# 默认并集结果中某个成员的 score 是所有输入集和下该成员 score 值之和
# WEIGHTS 为每个输入集和指定一个乘法因子，在传递给聚合函数之前，每个输入排序集中的每个元素的分数都要乘以这个因子。当权重未给定时，乘法因子默认为1
# AGGREGATE 指定聚合方式
ZUNION numkeys key [key ...] [WEIGHTS weight [weight ...]] [AGGREGATE SUM|MIN|MAX] [WITHSCORES]
# 将交集存储在 destination
ZUNIONSTORE destination numkeys key [key ...] [WEIGHTS weight [weight ...]] [AGGREGATE SUM|MIN|MAX]

# 交集
ZINTER numkeys key [key ...] [WEIGHTS weight [weight ...]] [AGGREGATE SUM|MIN|MAX] [WITHSCORES]
# 将交集存储在 destination
ZINTERSTORE destination numkeys key [key ...] [WEIGHTS weight [weight ...]] [AGGREGATE SUM|MIN|MAX]
# 只返回交集数量
ZINTERCARD numkeys key [key ...]

# 差集
ZDIFF numkeys key [key ...] [WITHSCORES]
# 将差集存储在 destination
ZDIFFSTORE destination numkeys key [key ...]

# 见 SCAN
ZSCAN key cursor [MATCH pattern] [COUNT count]
```

## 底层编码

- 当 SortedSet 中的元素数量较少且每个元素的分数和值都较小时，Redis 使用 `ziplist` 数据结构来实现 
- 当 SortedSet 中的元素数量较多或每个元素的分数和值都较大时，Redis 使用 `skiplist` 数据结构来实现 
- 当 SortedSet 中的元素数量或大小发生变化时，Redis 会自动调整底层数据结构以保持最佳性能
