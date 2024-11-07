# list

## 命令

```sh
# 将一个或多个值插入到列表
# 多个 value 则按从左到右的顺序依次插入
# key 不存在则创建一个空列表
# key 存在但不是列表类型则返回错误
LPUSH key element [element ...]
RPUSH key element [element ...]
# 只有当 key 存在且为 list 时执行 push
LPUSHX key value
RPUSHX key value
# 将列表 key 下标为 index 的元素的值设置为 value
# 当 key 不存在或 index 超出范围时返回错误
LSET key index value
# 在 pivot 处插入，没有找到 pivot 返回 -1
LINSERT key BEFORE|AFTER pivot value

# 弹出
LPOP key [count]
RPOP key [count]
# LPOP RPOP 阻塞版本，直到从多个 key 中弹出一个元素结束
BLPOP key [key ...] timeout
BRPOP key [key ...] timeout
# 接受多个 key 和弹出多个元素
LMPOP numkeys [key [key ...]] LEFT|RIGHT [COUNT count]
# LMPOP 阻塞版
BLMPOP timeout numkeys [key [key ...]] LEFT|RIGHT [COUNT count]
# 弹出 src 最后一个元素，并返回给客户端
# 弹出的元素插入到 dst 列表，作为头元素
# src 不存在则报错
# src = dst 则将表尾元素移动到表头
RPOPLPUSH source destination
# RPOPLPUSH 阻塞版
BRPOPLPUSH source destination timeout
# 根据参数 count 的值，移除列表中与参数 value 相等的元素
# count > 0: 从表头开始向表尾搜索，移除与 value 相等的元素，数量为 count
# count < 0: 从表尾开始向表头搜索，移除与 value 相等的元素，数量为 count 的绝对值
# count = 0: 移除表中所有与 value 相等的值
LREM key count value
# 修剪，只保留区间内的元素
LTRIM key start stop
# 将 source 的 SLEFT|SRIGHT 第一个元素移动到 destination 的 DLEFT|DRIGHT 第一个元素
LMOVE source destination SLEFT|SRIGHT DLEFT|DRIGHT
# LMOVE 的阻塞版本
BLMOVE source destination LEFT|RIGHT LEFT|RIGHT timeout

# 返回元素索引
# RANK 表示当多个元素等于 element 时返回第几个
# COUNT 返回 element 个数，0 表示全部返回
# MAXLEN 表示比较次数
LPOS key element [RANK rank] [COUNT num-matches] [MAXLEN len]

# 获取链表长度，不存在返回 0
LLEN key
# 返回下标为 index 的元素
LINDEX key index
# 范围查询
LRANGE key start stop
```

## 底层编码

在 Redis 3.0 之前，List 对象的底层数据结构是 `list` 或者 `ziplist`，然后在 Redis 3.2 之后，List 对象的底层改由 `quicklist` 数据结构实现

## 使用场景

1. `lpush` + `lpop` 模拟栈
2. `lpush` + `rpop` 模拟队列
3. `lpush` + `ltrim` 模拟有限集合
4. `lpush` + `brpop`模拟消息队列
