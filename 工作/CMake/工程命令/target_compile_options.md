# target_compile_options

```cmake
target_compile_options(<target> [BEFORE]
  <INTERFACE|PUBLIC|PRIVATE> [items1...]
  [<INTERFACE|PUBLIC|PRIVATE> [items2...] ...])
```

`target_compile_options` 默认将新属性添加都已有属性后，指定 `BEFORE` 后将新属性添加至已有属性前

`target_compile_options` 可以用来添加任何编译指令，但更推荐使用 `target_compile_definitions` 指定预处理选项，使用 `target_include_directories` 引用目录