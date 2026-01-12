# stream

主要用于消息队列的场景，支持多消费者组、消息持久化、ACK 确认机制，这些是 Pub/Sub 所不具备的

- 每条消息由唯一 ID 和键值对数据组成，格式为 `ID: {field1 value1, field2 value2 ...}`
- 消费者组允许多个消费者组独立消费同一 Stream，每个组维护自己的消费进度
- 消息 ID 支持自动生成（如 `*` 表示由 Redis 生成）或手动指定（格式为 `<时间戳>-<序号>`，如 `1630000000000-0`）
- Stream 数据持久化在 Redis 中，重启后不丢失

## 命令

```sh
# 插入一条消息
XADD key [NOMKSTREAM] [MAXLEN|MINID [=|~] threshold [LIMIT count]] *|ID field value [field value ...]
# key：Stream 的名称
# [NOMKSTREAM] 如果指定，当 Stream 不存在时，不会自动创建

# 基本用法：自动生成 ID
XADD mystream * field1 value1 field2 value2
# 返回：1703845200000-0（时间戳-序号）

# 手动指定 ID（必须大于上一条消息 ID）
XADD orders 1703845300000-0 order_id 1002 user_id 5002 amount 199.99
# 返回：1703845300000-0

# 限制 Stream 长度（保留最新 1000 条）
XADD orders MAXLEN ~ 1000 * order_id 1003 user_id 5003 amount 399.99
# ~ 表示近似裁剪（性能更好）
# = 表示精确裁剪

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

## 使用案例

以订单处理系统举例

```sh
# 创建第一个订单事件
XADD orders:events * order_id 1001 user_id 2001 action "order_created" items "item1,item2" timestamp "1672531200"
# 返回消息ID
"1672531200000-0"

# 创建更多订单
XADD orders:events * order_id 1002 user_id 2002 action "order_created" items "item3"
XADD orders:events * order_id 1003 user_id 2003 action "order_created" items "item4,item5,item6"

# 创建库存处理消费者组（从流的开头开始消费）
XGROUP CREATE orders:events inventory-group 0
# 创建通知处理消费者组
XGROUP CREATE orders:events notification-group 0
# 创建积分处理消费者组
XGROUP CREATE orders:events points-group 0

# 库存扣减服务
XREADGROUP GROUP inventory-group stock-consumer-1 COUNT 2 STREAMS orders:events >
# 输出：读取到订单1001, 1002的消息
# 处理消息后确认
XACK orders:events inventory-group 1672531200000-0
XACK orders:events inventory-group 1672531201000-0

# 通知服务
XREADGROUP GROUP notification-group notify-consumer-1 COUNT 1 STREAMS orders:events >
# 输出：读取到订单1001的消息

# 积分服务
XREADGROUP GROUP points-group points-consumer-1 COUNT 3 STREAMS orders:events >
# 输出：读取到所有3个订单的消息

# 确认处理完成
XACK orders:events points-group 1672531200000-0
XACK orders:events points-group 1672531201000-0
XACK orders:events points-group 1672531202000-0
```

