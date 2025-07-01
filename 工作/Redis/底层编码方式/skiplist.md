# skiplist

在链表的基础上，增加了多级索引，通过索引位置的几个跳转，实现数据的快速定位

![01](skiplist.assets/01.png)

## 数据结构

```cpp
typedef struct zskiplistNode {
    // Zset 对象的元素值
    sds ele;
    // 元素权重值
    double score;
    // 后向指针
    struct zskiplistNode *backward;
    // 节点的 level 数组，保存每层的前向指针和跨度
    struct zskiplistLevel {
        struct zskiplistNode *forward;
        unsigned long span;
    } level[];
} zskiplistNode;

typedef struct zskiplist {
    struct zskiplistNode *header, *tail;
    unsigned long length;
    int level;
} zskiplist;
```

