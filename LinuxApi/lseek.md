# lseek

更新文件偏移量，文件偏移量表示对当前文件进行操作的位置

```cpp
off_t lseek(int fd, off_t offset, int whence);
// where 取值
// SEEK_SET 直接设置
// SEEK_CUR 当前偏移量基础上加 offset
// SEEK_END 当前文件大小基础上加 offset 

lseek(fd, (off_t)1250, SEEK_SET); // 将偏移量设置为1250
lseek(fd, 0, SEEK_SET); // 将偏移量设置为文件开头
lseek(fd, 0, SEEK_END); // 将偏移量设置为文件末尾
lseek(fd, 0, SEEK_CUR); // 确定当前文件位置

// 返回结果是以字节为单位从文件开始的偏移位置
```

当偏移超出文件大小后，后续写入数据会产生空洞，空洞填充的是 `\0`