# const 细节

## 定义常量

定义时必须初始化，因为常量在定义后不能被修改

```cpp
const int a = 10;
```

`const` 并未区分出编译期常量和运行期常量，`const` 只保证了运行时不直接被修改

## 类型检查

`const` 常量具有类型，编译器可以进行安全检查

`#define` 宏定义没有数据类型，只是简单的字符串替换，不能进行安全检查

## 节省空间，避免不必要的内存分配

`const` 定义的常量在程序运行过程中只有一份拷贝，而 `#define` 定义的常量在内存中有若干拷贝

## const 对象默认为文件局部变量

非 `const` 变量默认为 `extern`，而 `const` 变量默认为局部变量，要使其他文件访问变量，必须显式指定为 `extern`

```cpp
// 非 const 变量
// file1
int a;
// file2
extern int a;

// const 变量
// file1
extern const int b = 1;
// file2
extern const int b;
```

## 指针与 const 

```cpp
const char * a;  // 常量指针
char cosnt * b;  // 同上
char * const c;  // 指针常量
const char * const d;  // 指向常量的指针常量
```

允许将非 `const` 对象地址赋值给常量指针，但仍然不能通过常量指针修改该对象

## 类中的 const 

- 常成员函数：在一个类中，任何不会修改成员数据的函数都应该声明为 `const`，只有常成员函数才有资格操作常量或常对象，非常成员函数不能用来操作常对象

- 常成员变量初始化：常成员变量必须通过初始化列表进行初始化，或者声明为 `static`，或在类外面初始化