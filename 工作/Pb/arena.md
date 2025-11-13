# arena

arena 是 protobuf 3.0 版本开始支持的一个特性，它用来接管pb对象的生命周期,使用它可以帮助你优化程序的内存使用、提高性能

arena 就是由 protbuf 库去接管 pb 对象的内存管理。它预先分配一个内存块，解析消息和构建消息等触发对象创建时是在已分配好的内存块上 `placement new`出来。arena 对象析构时会释放所有内存，理想情况下不需要运行任何被包含对象的析构函数

- 减少复杂的pb对象中多次 `malloc/free` 和析构带来的系统开销
- 减少内存碎片
- pb 对象的内存连续，cache line 友好、读取性能高

## 使用

```sh
option cc_enable_arenas = true;
```

```cpp
#include <google/protobuf/arena.h>
{
  // 栈对象
  google::protobuf::Arena arena;
  MyMessage* message = google::protobuf::Arena::CreateMessage<MyMessage>(&arena);
  // ...
}

{
  // 堆对象
  auto arena = std::make_shared<google::protobuf::Arena>();
  auto* message = google::protobuf::Arena::CreateMessage<MyMessage>(arena.get());
  auto message_sptr = std::shared_ptr<MyMessage>(message, [arena](void*) {});
}
```

不要让 arena 永久存在，不然内存会不断累加，可能造成内存泄漏

## 踩坑

```cpp
template<typename T> 
static T* CreateMessage(Arena* arena)
```

- 如果 `arena` 不为 `null`，则新对象在 `arena` 分配，其内部存储和子消息也将在同一 `arena` 上分配，新对象不能手动删除、释放
- 如果 `arena` 为 `null`，新对象在 heap 上分配，需要手动删除、释放

```cpp
Arena* GetArena()
```

- 如果此消息是在 `arena` 上分配的，返回分配此消息对象的 `arena` 指针
- 如果此消息不是在 `arena` 上分配的，返回 `nullptr`
- 使用 `GetArena` 一定要判断返回的指针，如果一个 pb message 不是在 `arena` 上创建的，那么如果直接使用这个message中 `GetArena()` 返回的 `arena` 去创建新的 pb 对象，实际上是在 heap 中分配的，使用完成后需要手动释放内存

```cpp
void Swap(Message* other)
```

- 如果要交换的两个 message 不在 `arena` 或者在同一个 `arena`，则行为与未启用 `arena` 时相同，浅拷贝，指针交换，避免复制
- 如果其中一个不在 `arena` 或者不在同一个 `arena`，则深拷贝