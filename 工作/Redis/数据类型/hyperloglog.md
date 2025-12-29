# hyperloglog

 HyperLogLog 是一种基数算法，可以利用极小的内存空间完成去重计数的统计

- 内存占用量非常小，固定 12KB，可以统计 2^64 个元素
- 存在错误率，标准误差是 0.81%
- 不能查询单条数据，不能删除元素

## 命令

```sh
# 添加
# 如果近似基数因为 PFADD 出现了变化，那么命令返回 1，否则返回 0 
PFADD key [element [element ...]]

# 获取近似基数（去重后的数量）
PFCOUNT key [key ...]
# 当使用多个键调用时，会将所有 key 内部合并到一个临时 HyperLogLog 中，返回所传递的 HyperLogLog 联合的近似基数

# 合并
PFMERGE destkey sourcekey [sourcekey ...]
```

## 对比

- HashMap：算法简单，统计精度高，对于少量数据建议使用，但是对于大量的数据会占用很大内存空间
- BitMap：虽然内存占用要比 HashMap 少，但是对于大量数据还是会占用较大内存，并且要求 ID 连续且范围可控
- HyperLogLog：存在一定误差，占用内存少，稳定占用 12k 左右内存，可以统计 2^64 个元素

## 使用场景

1. 访问统计
2. 独立性分析
