# bloomfilter

布隆过滤器是一种非常节省空间的概率数据结构，运行速度快，占用内存小，但是有一定的误判率且无法删除元素

## 命令

```sh
# 初始化布隆过滤器
# 指定误判率和容量
BF.RESERVE key error_rate capacity

# 将元素添加到布隆过滤器
BF.ADD key item	
# 批量添加元素
BF.MADD key item1 item2 ...

# 检查元素是否存在
BF.EXISTS key item	
# 批量检查元素是否存在
BF.MEXISTS key item1 item2 ...
```

## 使用场景

- 缓存穿透防护

- 去重

  