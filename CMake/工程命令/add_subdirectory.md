# add_subdirectory

```cmake
add_subdirectory(source_dir [binary_dir] [EXCLUDE_FROM_ALL])
```

`source_dir` 可以是绝对地址，也可以是相对于当前文件的相对地址

`binary_dir` 指明 `source_dir` 的产物地址，默认是 `source_dir`

`binary_dir` 可以是绝对地址，也可以是相对地址