# intset

`intset` 是 Set 对象的底层实现之一，当一个 Set 对象只包含整数值元素，并且元素数量不时，就会使用整数集这个数据结构作为底层实现

`intset` 本质上是一块连续内存空间，支持高效的插入、删除和查找操作

## 数据结构

```cpp
#define INTSET_ENC_INT16 (sizeof(int16_t))
#define INTSET_ENC_INT32 (sizeof(int32_t))
#define INTSET_ENC_INT64 (sizeof(int64_t))

typedef struct intset {
    // 支持 INTSET_ENC_INT16、INTSET_ENC_INT32、INTSET_ENC_INT164
    uint32_t encoding;
    uint32_t length;    //集合包含的元素数量
    int8_t contents[];  //保存元素的数组
} intset;
```

## 升级操作

当一个新元素加入到 `intset` 里面时，如果新元素的类型比现有编码方式要长时，`intset` 需要先进行升级

1. 按新类型扩展 `contents` 数组的空间大小
2. 将 `contents` 现有的所有元素都转换成新的编码格式，并将转换后的元素放到正确的位置
3. 插入新元素且保持数组的有序性

`intset` 不支持降级操作

## 优缺点

- 使用连续的内存空间存储整数，这使得它在内存占用方面非常紧凑
- 使用数组来存储整数，因此它的插入、删除和查找操作的时间复杂度都是 O(1)
- 整数是有序存储的，这使得它支持范围查询操作
- 当 `intset` 中的整数超出当前编码范围时，它会自动升级到更大的编码类型，以确保能够存储更大的整数
