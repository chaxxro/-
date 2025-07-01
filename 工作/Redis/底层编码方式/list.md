# list

## 双向链表数据结构

```cpp
// 双向链表
typedef struct listNode {
    struct listNode *prev;
    struct listNode *next;
    void *value;
} listNode;

// 迭代器指向下一个节点
typedef struct listIter {
    listNode *next;
    int direction;
} listIter;

// 双向无环
typedef struct list {
    listNode *head;
    listNode *tail;
    void *(*dup)(void *ptr);  // 复制操作
    void (*free)(void *ptr);  // 释放操作
    int (*match)(void *ptr, void *key);  // 比较操作
    unsigned long len;
} list;
```

头节点和尾节点并不相连，它是一个无环链表

## 优缺点

1. 获取某个节点的前置节点或后置节点的时间复杂度只需 O(1)
2. 获取链表的表头节点和表尾节点的时间复杂度只需 O(1)
3. 提供了链表节点数量 `len`，所以获取链表中的节点数量的时间复杂度只需 O(1)
4. 使用 `void*` 指针保存节点值，可以保存各种不同类型的值
5. 内存不连续，无法很好利用 CPU 缓存
6. 内存开销较大