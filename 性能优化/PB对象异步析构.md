# PB 对象异步析构

对于嵌套层次极深的 pb 而言，析构开销不可小觑。当数据的量级达到一定程度时，析构的时延也会达到毫秒级，但析构其实并不是关键路径

异步析构流程：

1. 业务代码在堆上分配对象，对象生命周期结束时，将指针入列
2. 后台轮训线程不断出列指针，并且执行析构

因为队列通过模板实现，所以只支持一种类型的指针。在启动时，需要这样初始化队列和后台线程。如果想增加新的类型，就必须再注册一次，且再创建至少一个线程

由于 C++ 的多态特性，只需要注册 Message，即可入列所有 pb。但是 pb 的粒度太小，如果为容器内的每个 pb 都入列，这样的并发开销显然没有必要