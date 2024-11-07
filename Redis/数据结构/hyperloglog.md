# hyperloglog

 HyperLogLog 是一种基数算法，可以利用极小的内存空间完成不精确的去重计数

## 命令

```sh
# 添加
PFADD key [element [element ...]]



# 获取近似基数
PFCOUNT key [key ...]



# 合并
PFMERGE destkey sourcekey [sourcekey ...]
```

