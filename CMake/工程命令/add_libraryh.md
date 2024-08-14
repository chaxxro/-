
# title: add_library

```cmake
add_library(<name> [STATIC | SHARED | MODULE]
            [EXCLUDE_FROM_ALL]
            [<source>...])
```

默认构建类型取决于 `BUILD_SHARED_LIBS` 的值