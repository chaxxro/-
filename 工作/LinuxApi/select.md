# select

`select` 允许应用程序监视一组文件描述符，等待一个或者多个描述符成为就绪状态，从而完成 IO 操作

`select()` 将监听的文件描述符分为三组，每一组监听不同的需要进行的 IO 操作，`readfds` 是需要进行读操作的文件描述符，`writefds` 是需要进行写操作的文件描述符，`exceptfds` 是需要进行异常事件处理的文件描述符

- 读事件：

1. 在 socket 内核中，接受缓冲区中的字节数大于或等于低水位标志 `SO_RCVLOWAT`，此时调用 `recv` 或 `read` 可以无阻塞读

2. TCP 对端关闭连接，此时调用 `recv` 或 `read` 返回 0

3. 监听 socket 上有新的连接请求

4. socket 上有未处理的错误

- 写事件：

1. 在 socket 内核中，发送缓冲区中的可用字节数大于或等于低水位标记 `SO_SNDLOWAT`，此时可以无阻塞地写

2. socket 写操作被关闭，如调用了 `close` 或 `shutdown`，对一个写操作被关闭的 socket 进行写操作，会出发 `SIGPIPE`

3. socket 使用非阻塞 `connect` 连接成功或失败，对于没有建立连接成功的 socket，也会得到可写的结果；对于非阻塞的 socket，不仅要调用 `select` 检测是否可写，还需要调用 `getsockopt` 检测 socket 是否出错，通过错误码来判断是否连接成功，错误码为 0 时连接成功

```cpp
#include <sys/select.h>
int select(int nfds, fd_set *readfds, fd_set *writefds, fd_set *exceptfds, struct timeval *timeout);

struct timeval {
    long tv_sec;   // second
    long tv_usec;  // microsecond
}
/*
nfds 表示需要使用 select 检测事件中的 fd 中的最大值加 1

fd_set 使用数组实现，数组大小使用 FD_SETSIZE 定义，所以只能监听少于 FD_SETSIZE 数量的描述符

readset、writeset、exceptset，分别对应读、写、异常条件的描述符集合

timeout 为超时参数，调用 select 会一直阻塞直到有描述符的事件到达或者等待的时间超过 timeout

timeout 为空时表示永不超时，超时之后 timeout 参数都会被置 0

select 函数也会修改 timeval 结构体的值

成功调用返回结果大于 0，出错返回结果为 -1，超时返回结果为 0
*/

void FD_ZERO(fd_set *fdset);  
void FD_SET(int fd, fd_set *fdset);
void FD_CLR(int fd, fd_set *fdset);
int FD_ISSET(int fd,fd_set *fdset);
/*
FD_ZERO：指定的文件描述符集清空，在对文件描述符集合进行设置前，必须对其进行初始化，如果不清空，由于在系统分配内存空间后，通常并不作清空处理，所以结果是不可知的

FD_SET：用于在文件描述符集合中增加一个新的文件描述符

FD_CLR：用于在文件描述符集合中删除一个文件描述符

FD_ISSET：用于测试指定的文件描述符是否在该集合中
*/

int svr_fd = get_svr_fd(10000);

fd_set fd_reads;
std::vector<int> fds {svr_fd};

struct timeval tv;
tv.tv_sec = 10;
tv.tv_usec = 0;

while (1) {
  FD_ZERO(&fd_reads);
  int ns = 0;
  std::for_each(fds.begin(), fds.end(), [&fd_reads, &ns](const int& fd) {ns = std::max(ns, fd); FD_SET(fd, &fd_reads);});

  int ret = select(ns + 1, &fd_reads, nullptr, nullptr, &tv);
  if (ret == -1) {
    // 错误
    OUTPUT_ERRNO_EXIT;
  } else if (ret == 0) {
    // 超时
  } else {
    for (const int& fd : fds) {
      if (FD_ISSET(fd, &fd_reads)) {
        if (fd == svr_fd) {
          int cli = accept_cli(svr_fd);
          send_msg(cli);
          fds.push_back(cli);
        } else {
          recv_msg(fd);
        }
      }
    }
  }
}
```

`select` 底层使用 `poll`

## 缺点

1. 性能开销大

- 调用 `select` 时需要将 `fd_set` 拷贝到内核空间，函数返回时又需要将 `fd_set` 拷回用户空间

- 函数只返回就绪连接数量，需要遍历所有 `fd_set` 逐一检查是否就绪

2. 支持监听数量有限

## 支持的最大数量连接数

32 位系统是 1024，64 位系统是 2048

```cpp
// fd_set 是一个 long 数组
typedef long int __fd_mask;
// __fd_mask 所占位数，64
#define __NFDBITS     (8 * (int) sizeof (__fd_mask))

typedef struct {
    // __FD_SETSIZE = 1024
    __fd_mask __fds_bits[__FD_SETSIZE / __NFDBITS];
} fd_set
```

- `__fds_bits` 占 `__FD_SETSIZE` 位，表示 `select` 支持最大的 fd 数量

- 每一位对应一个 fd 的事件状态，0 表示无事件，1 表示有事件