# free

查看系统内存使用情况，数据来源于 `/proc/meminfo`

```sh
free [options]
-h 可读方式显示
-b byte
-k kb
-m mb
-g gb

total 总内存大小
used 已使用内存的大小，包含了共享内存
free 未使用内存的大小
shared 共享内存的大小
buff/cache 缓存和缓冲区的大小
available 新进程可用内存的大小
```