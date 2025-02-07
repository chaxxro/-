# string

## 命令

值可以是任何种类的字符串（包括二进制数据），只需要注意不要超过 512 MB 的最大限度

```sh
# 当 key 存在时，SET 命令会覆盖掉你上一次设置的值
# EX EXAT 过期时间，单位秒
# PX PXAT 过期时间，单位毫秒
# GET 设置新值并返回老值
SET key value [EX seconds|PX milliseconds|EXAT timestamp|PXAT milliseconds-timestamp|KEEPTTL] [NX|XX] [GET]
MSET key value [key value ...]
SETNX key value
# 等价于  SET + EXPIRE
SETEX key seconds value
# 等价于 SETEX，不同的是过期时间单位
PSETEX key milliseconds value
# 从偏移量开始，用 value 覆写键 key 储存的字符串值
# 不存在的键 key 当作空白字符串处理
# 需要确保字符串足够长以便将 value 设置到指定的偏移量上
# 如果键 key 原来储存的字符串长度比偏移量小，原字符和偏移量之间的空白将用零字节填充
SETRANGE key offset value
# 获取字符串长度
STRLEN key

GET key
MGET key [key ...]
# 设置新的 value 并返回旧值
GETSET key value
# 获取值并设置过期时间
GETEX key [EX seconds|PX milliseconds|EXAT timestamp|PXAT milliseconds-timestamp|PERSIST]
# 获取 [start, end] 子串，允许负值
GETRANGE key start end
# 获取值并删除 key
GETDEL key

# 追加，如果 key 存在则将 value 追加到末尾，如果 key 不存在则将 key 设置为 value
APPEND key value
# 如果 key 的 value 是数字，则做数学运算
INCR key
INCRBY key increment
INCRBYFLOAT key increment
DECR key
DECRBY key decrement
```

如果 value 是一个整数，还可以对它使用 `INCR`  命令进行原子性的自增操作

```sh
SET connections 10
INCR connections # 11
DEL connections
INCR connections # 1
INCRBY connections 100 # 101
DECR connections # 100
DECRBY connections # 90
```

## 底层编码

字符串类型有三种编码方式：

- `OBJ_ENCODING_INT` 整数编码，适用于存储可以表示为 64 位有符号整数的字符串值
- `OBJ_ENCODING_RAW` sds 编码 ，用于存储较长的字符串
- `OBJ_ENCODING_EMBSTR` 嵌入式字符串编码，适用于存储较短的字符串（长度小于等于 44 字节）

```sh
# int 编码
SET mykey 12345
# raw 编码
SET mykey "11111111111111111111111111111111111111111111111111"
# embstr 编码
SET mykey "this is a data"
```

当保存的是 64 位有符号整数时，`RedisObject` 中的指针就直接赋值为整数数据了，这样就不用额外的指针再指向整数了，节省了指针的空间开销

当保存的是字符串且字符串小于等于 44 字节时，字符串数据以 sds 方式编码，并且和 `RedisObject` 存放在同一片内存区域，这样就可以避免内存碎片

当字符串大于 44 字节时，字符串数据以 sds 方式编码，但 Redis 就不再把 sds 和 `RedisObject` 布局在一起了，而是会给 sds 分配独立的空间，并用指针指向 SDS 结构

![01](string.assets/01.png)



## 使用场景

1.  缓存数据，提高查询性能
2. 计数器
3. 共享 Session
