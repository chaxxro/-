# dict

## 数据结构

`dict` 是一个数组，数组的每个元素是一个指向 `dictEntry` 指针

```cpp
typedef struct dictEntry {
    void *key;
    union {
        void *val;
        uint64_t u64;
        int64_t s64;
        double d;
    } v;
    struct dictEntry *next;  // 多个哈希值相同的 key 组成链表
} dictEntry;

// 一些哈希表要用到的函数
typedef struct dictType {
    // 哈希函数
    uint64_t (*hashFunction)(const void *key);
    // 复制键函数
    void *(*keyDup)(void *privdata, const void *key);
    // 复制值函数
    void *(*valDup)(void *privdata, const void *obj);
    // 对比键函数
    int (*keyCompare)(void *privdata, const void *key1, const void *key2);
    // 销毁键函数
    void (*keyDestructor)(void *privdata, void *key);
    // 销毁值函数
    void (*valDestructor)(void *privdata, void *obj);
    // 判断是否需要 rehash
    int (*expandAllowed)(size_t moreMem, double usedRatio);
} dictType;

// 实际的哈希表
typedef struct dictht {
    dictEntry **table;
    unsigned long size;  // 默认值 0
    unsigned long sizemask;  // 哈希表大小掩码，用于计算索引，默认值 0
    unsigned long used;  // 已有节点数，包括 bucket 中的节点，默认值 0
} dictht;

// 暴露的结构
typedef struct dict {
    dictType *type;
    void *privdata;  // 私有数据
    dictht ht[2];  // 两个哈希表用于渐进 rehash
    long rehashidx; // -1 表示没有在进行 rehash，默认值 -1
                    // >= 0 时表示 dictht.table 中的下标
    int16_t pauserehash; // >0 表示 rehash 暂停中，<0 表示错误，默认值 0
} dict;
```

Redis 使用内存分配库 jemalloc，而 jemalloc 在分配内存时，会根据申请的字节数 N，找一个比 N 大且最接近 N 的 2 的幂次数作为分配的空间，这样可以减少频繁分配的次数，所以 `dictEntry` 结构就占用了 32 字节

## 哈希冲突

Redis 使用开链的方式解决哈希冲突，同一个哈希桶中的多个元素用一个链表来保存，它们之间依次用指针连接

![02](dict.assets/02.png)

### rehash

Redis 会对 `dict` 做 rehash 操作增加现有的哈希桶数量，让逐渐增多的 entry 元素能在更多的桶之间分散保存，减少单个桶中的元素数量，从而减少单个桶中的冲突

Redis 会使用装载因子（load factor）来判断是否需要做 rehash

在进行 RDB 生成和 AOF 重写时，`dict` 的 rehash 是被禁止的，这是为了避免对 RDB 和 AOF 重写造成影响

- 装载因子 ≥1，同时哈希表没有进行 RDB 和 AOF 重写，则被允许进行 rehash

- 装载因子 ≥5，则立马开始 rehash

为了使 rehash 操作更高效，`dict` 默认使用两个 `dictht`

当刚插入数据时，默认使用 `dictht`1，此时的 `dictht` 2 并没有被分配空间，随着数据逐步增多，Redis 开始执行 rehash

1. 给 `dictht` 2 分配更大的空间

2. 把 `dictht`1 中的数据重新映射并拷贝到 `dictht` 2 中

3. 释放 `dictht`1 的空间，留作下一次 rehash 扩容备用

第二步涉及大量的数据拷贝，如果一次性把 `dictht`1 中的数据都迁移完，会造成 Redis 线程阻塞，无法服务其他请求，所以 Redis 采用了渐进式 rehash

在第二步拷贝数据时，Redis 仍然正常处理客户端请求，每处理一个请求时，从 `dictht`1  中的第一个索引位置开始，顺带着将这个索引位置上的所有 entries 拷贝到 `dictht` 2 中。等处理下一个请求时，再顺带拷贝 `dictht`1 中的下一个索引位置的 entries。渐进式 rehash 把一次性大量拷贝的开销，分摊到了多次处理请求的过程中，避免了耗时操作，保证了数据的快速访问

![03](dict.assets/03.png)
