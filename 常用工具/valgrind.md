# valgrind

```sh
valgrind [options] prog-and-args [options]
# --tool=[name]，默认为 memcheck
# -q --quiet 只打印错误信息
# --verbose 更详细的信息
# --trace-children=<yes|no> 跟踪子线程，默认 no
#--track-fds=<yes|no> 跟踪打开的文件描述，默认 no
#--time-stamp=<yes|no> 增加时间戳到LOG信息，默认 no
#--log-fd=<number> 输出 LOG 到描述符文件，默认 2=stderr
# --log-file=<file> 将输出的信息写入到 filename.PID 的文件里
```

## memcheck

探测程序中内存管理存在的问题：

1. 使用未初始化的内存

2. 检查内存读写越界

3. 内存泄露

4. 读写已经释放的内存

5. 使用 `malloc`/`new`/`new[]` 和 `free`/`delete` /`delete[]` 不匹配

6. src 和 dst 重叠

7. 读写不恰当的内存栈空间

首先，在编译程序的时候打开调试模式 `-g`，最好还使用 `-fno-inline` ，最好不要使用优化选项

## callgrind

`valgrind --tool=callgrind ./test` 执行完毕后会在当前目录下生成一个文件，文件名为 callgrind.out.pid

对于服务类进程的调试，不可以通过 `kill -9` 方式停止，要用 `kill -s SIGINT pid` 或者 Ctrl + C 的方式才会生成 callgrind.out 文件

生成的 callgrind.out 文件有两种查看方式，一种为转换为 png 图片查看，另一种使用 cachegrind 查看

### 转换成 png

1. 把 callgrind 生成的性能数据转换成 dot 格式数据，使用 gprof2dot

2. 通过 dot 工具将 dot 格式数据转成 png 图片，`dot -Tpng valgrind.dot -o valgrind.png`

### cachegrind

mac 下可以使用 qcachegrind