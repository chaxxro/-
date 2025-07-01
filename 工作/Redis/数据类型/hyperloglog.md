# hyperloglog

 HyperLogLog 是一种基数算法，可以利用极小的内存空间完成去重计数的统计

- 内存占用量非常小，但是存在错误率
- 不能获取单条数据
- 标准误差是 0.81%

## 命令

```sh
# 添加
# 如果近似基数因为 PFADD 出现了变化，那么命令返回 1，否则返回 0 
PFADD key [element [element ...]]

# 获取近似基数
PFCOUNT key [key ...]

# 合并
PFMERGE destkey sourcekey [sourcekey ...]
```

## 对比

- HashMap：算法简单，统计精度高，对于少量数据建议使用，但是对于大量的数据会占用很大内存空间
- BitMap：虽然内存占用要比 HashMap 少，但是对于大量数据还是会占用较大内存
- HyperLogLog：存在一定误差，占用内存少，稳定占用 12k 左右内存，可以统计 2^64 个元素

## 使用场景

1. 访问统计
2. 独立性分析
