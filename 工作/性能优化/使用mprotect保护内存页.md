# 使用 mprotect 保护内存页

当一个对象的某个属性被越界写坏了，一个常规推断就是与它相邻的属性发生了越界操作，导致当前属性被波及

有很多检测 Cpp 内存越界方案，常见的有：

- 静态代码检查：比如 Cppcheck、Clang Static Analyzer 等，这类工具在编译时能够检测出一些潜在的内存越界方案，但无法检测在运行时发生的越界问题
- 动态分析：比如 Valgrind、AddressSanitizer 等在运行时检测内存越界方案，但缺点是会降低程序的运行速度
- 边界检查库：比如 Safe STL 等，这类对程序是有侵入的

另外一个思路是使用 `mprotect`，在对象间插入内存块并且设置内存页方案权限，一旦发生越界访问可以直接输出相应信息

## mprotect

```cpp
int mprotect(void *addr, size_t len, int prot);
/*
addr 内存区域起始地址（必须页对齐）
len 修改区域长度（自动提升为页大小整数倍）
prot 保护标志的组合
PROT_NONE    0x0   禁止所有访问（触发的访问会导致段错误）
PROT_READ    0x1   允许读取（CPU读取、mmap映射等）
PROT_WRITE   0x2   允许写入（修改内存内容）
PROT_EXEC    0x4   允许执行（可作为代码执行）
*/
```

由于 `mprotect` 保护的内存需要首地址对齐和大小对齐，一般可以搭配 `mmap` 使用

```cpp
class ProtectMem {
 public:
  ProtectMem() {
    // 获取内存页大小
    size_t page_size = sysconf(_SC_PAGESIZE);
    void* ptr = mmap(NULL, 3 * page_size, PROT_READ | PROT_WRITE,
                    MAP_PRIVATE | MAP_ANONYMOUS, -1, 0);
    // 第一页不可访问
    mprotect(ptr, page_size, PROT_NONE);  
    // 最后一页不可方案
    mprotect(ptr + 2 * page_size, page_size, PROT_NONE);  
    // 中间一页是数据
    ...
  }
};
```

## 保护内存页

使用自定义内存分配器，将数据以 `replacement new` 的方式放入被保护内存页的中间

```cpp
std::size_t pagesize() {
    static std::size_t size = ::sysconf(_SC_PAGESIZE);
    return size;
}

std::size_t page_count( std::size_t size) {
    return static_cast< std::size_t >( std::ceil( static_cast< float >(size) / pagesize() ) );
}

template <typename T>
class mprotect_allocator: public std::allocator<T> {
public:
    typedef std::allocator<T> base;
    typedef size_t size_type;
    typedef off_t offset_type;
    typedef T *pointer;
    typedef const T *const_pointer;

    mprotect_allocator() noexcept {}
    mprotect_allocator(const mprotect_allocator& a) noexcept : base(a) {}
    template <typename U>
    mprotect_allocator(const mprotect_allocator<U>& a) noexcept : base(a)
    {
    }
    ~mprotect_allocator() noexcept {}
    template <typename _Other>
    struct rebind {
        typedef mprotect_allocator<_Other> other;
    };

    pointer allocate(size_type n, const void *hint=0) {
        size_type size = n * sizeof(T);
        const std::size_t pages( page_count( size) + 2); // add one guard page
        const std::size_t size_( pages * pagesize() );
        assert( 0 < size && 0 < size_);

        void * limit = mmap( NULL, size_, PROT_READ | PROT_WRITE, MAP_PRIVATE | MAP_ANONYMOUS, -1, 0);
        if (!limit) throw std::bad_alloc();

        // protect head page
        int result( ::mprotect( limit, pagesize(), PROT_NONE) );
        assert( 0 == result);

        // protect tail page
        result = ::mprotect((char*)limit + size_ - pagesize(), pagesize(), PROT_NONE);
        assert( 0 == result);

        return reinterpret_cast<pointer>(static_cast<char*>(limit) + pagesize());
    }

    void deallocate(pointer p, size_type n) {
        assert(NULL != p);

        size_t size = n * sizeof(T);
        const std::size_t pages = page_count( size) + 2;
        const std::size_t size_ = pages * pagesize();
        assert( 0 < size && 0 < size_);
        void * limit = reinterpret_cast< char * >(p) - pagesize();
        ::munmap( limit, size_);
    }
};
```

