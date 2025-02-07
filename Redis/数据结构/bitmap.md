# bitmap

Bitmap 是一个以位为单位的数组，数组的每个单元只能存储 0 和 1

## 命令

```sh
# 设置或清除指定偏移量上的位
# 当 key 不存在时，自动生成一个新的字符串值
# 字符串长度会自动增长，以确保它能支持指定偏移处
# offset 范围 [0, 232)
SETBIT key offset value

# 当 offset 比字符串值的长度大，或者 key 不存在时，返回 0
GETBIT key offset
# 返回 1 的数量，key 不存在则为 0
BITCOUNT key [start end]
# 返回 [start, end] 区间内第一个等于 bit 的下表
# bit 只支持 0、1
BITPOS key bit [start [end]]

# 对多个 key 做并、或、异或、非计算，并将结果写入 destkey
# operation 支持 AND、OR、XOR、NOT
BITOP operation destkey key [key ...]
```

