# Geo

支持存储地理位置信息用来实现诸如附近位置、摇一摇这类依赖于地理位置信息的功能

## 命令

```sh
# 添加地理位置
GEOADD key longitude latitude member [longitude latitude member ...]

# 删除地理位置
ZREM key member

# 获取位置坐标
GEOPOS key member [member ...]

# 计算两地距离
# 单位支持 m、km、mi、ft
GEODIST key member1 member2 [unit]

# 以经纬度为中心查找半径内的成员
GEORADIUS key longitude latitude radius unit [WITHCOORD] [WITHDIST] [WITHHASH] [COUNT count] [ASC|DESC]
# 以成员为中心查找半径内的成员
GEORADIUSBYMEMBER key member radius unit [WITHCOORD] [WITHDIST] [WITHHASH] [COUNT count] [ASC|DESC]
# WITHCOORD：返回成员的经纬度
# WITHDIST：返回与中心点的距离
# COUNT：限制返回结果数量
# ASC/DESC：按距离升序/降序排序

# 替代 GEORADIUS 和 GEORADIUSBYMEMBER
GEOSEARCH key [FROMMEMBER member] [FROMLONLAT longitude latitude] [BYRADIUS radius unit] [BYBOX width height unit] [ASC|DESC] [COUNT count] [WITHCOORD] [WITHDIST] [WITHHASH]
# FROMMEMBER：以成员位置为中心
# FROMLONLAT：以指定经纬度为中心
# BYRADIUS：按圆形范围搜索
# BYBOX：按矩形范围搜索

# 获取地理编码的哈希值
GEOHASH key member [member ...]
```

## 底层编码

GEO 数据通过有序集合 ZSET 存储，成员的分数为经纬度编码后的值

## 使用场景

1. 附近地点搜索
