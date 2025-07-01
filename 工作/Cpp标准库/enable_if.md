# enable_if

`std::enable_if` 是 C++ 标准库中的一个模板元编程工具，用于在编译时基于条件启用或禁用函数模板或类模板的特化。它利用了 SFINAE（Substitution Failure Is Not An Error）原则，即当模板参数替换导致无效类型或表达式时，该模板特化或重载被丢弃而不视为错误

```cpp
template<bool B, class T = void>
struct enable_if {};
 
template<class T>
struct enable_if<true, T> { typedef T type; };

template< bool B, class T = void >
using enable_if_t = typename enable_if<B,T>::type;
```

`enable_if` 的核心是一个编译时的条件开关

- 如果第一个模板参数 `B` 为 `true`，则 `enable_if<true, T>` 特化定义了嵌套类型 `type`（即 `T` 本身）。
- 如果 `B` 为 `false`，则 `enable_if<false, T>` 是基础模板（没有定义 `type`），这将导致依赖它的类型表达式替换失败

## 使用场景

### 控制函数模板的存在性

```cpp
// 仅当 T 是整数类型时存在
template<typename T>
typename std::enable_if<std::is_integral<T>::value, void>::type
process(T value) {
    // 整数处理逻辑
}

// 仅当 T 是浮点类型时存在
template<typename T>
std::enable_if_t<std::is_floating_point_v<T>, void>
process(T value) {
    // 浮点数处理逻辑
}

process(10);      // 调用整数版本
process(3.14);    // 调用浮点数版本
process("hello"); // 编译错误：无匹配函数
```

### 类模板特化控制

```cpp
// 主模板
template<typename T, typename = void>
struct Processor {
    void operator()(T) { /* 通用实现 */ }
};

// 针对字符串类型的特化
template<typename T>
struct Processor<T, std::enable_if_t<std::is_convertible_v<T, std::string>>> {
    void operator()(T str) {
        // 字符串处理逻辑
    }
};

```

### 构造函数控制

```cpp
class SmartPtr {
public:
    // 只允许指针类型的构造
    template<typename T>
    SmartPtr(T* ptr, std::enable_if_t<std::is_pointer_v<T>>* = nullptr)
        : data(ptr) {}
    
    // 防止隐式转换（非指针）
    template<typename T>
    SmartPtr(T, std::enable_if_t<!std::is_pointer_v<T>>* = nullptr) = delete;
};
```

