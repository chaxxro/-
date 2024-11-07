# stream

## 命令

```sh
# 插入一条消息
XADD key [NOMKSTREAM] [MAXLEN|MINID [=|~] threshold [LIMIT count]] *|ID field value [field value ...]
# id 为 * 表示让 Redis 为插入的数据自动生成一个全局唯一的 id
# 也可以直接在消息队列名称后自行设定一个 id 号，只要保证这个 id 号是全局唯一的就行
# 生成的消息的全局唯一 id 由两部分组成

# 获取总长度
XLEN key

# 范围获取
XRANGE key start end [COUNT count]
XREVRANGE key start end [COUNT count]
# - + 表示获取所有，- 表示最小 ID， + 表示最大 ID

# 删除消息
XDEL key id
# 仅设置标志位，不影响消息总长度

# 删除 Streams
del key

# 获取消息
XREAD [COUNT count] [BLOCK milliseconds] STREAMS key [key ...] ID [ID ...]
# milliseconds 用于设置 XREAD 为阻塞模式下阻塞时长，默认为非阻塞模式
# BLOCK 0 代表永远阻塞
# ID，用于设置由哪个消息开始读取
# 使用 0 表示从第一条消息开始
# 消息队列 ID 是单调递增的，所以通过设置起点，可以向后读取
# 在阻塞模式中，可以使用 $，表示最新的消息 ID
# 如果有多个客户端等待同一个队列，当队列添加一个新消息的时候
# 所有客户端都会收到这个消息，策略为 FIFO

# 创建消费者组
XGROUP CREATE key group-name id
# id 为 $ 表示从新消息开始分发，0 表示从头开始分发
# 初始化 last_delivered_id

# 组内消费
XREADGROUP GROUP group-name consumer-name [COUNT count] [BLOCK milliseconds] [NOACK] STREAMS key [key ...] ID [ID ...]
# count：读取数量
# milliseconds：阻塞时间
# ID 为 > 表示从组内 last_delivered_id 后面开始读
# ID 为 0-0 表示读取所有 PEL 消息及 last_delivered_id 之后的新消息

# 组内消费确认
XACK key group-name id
```

