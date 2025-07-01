
# add_executable

```cmake
add_executable(<name> [WIN32] [MACOSX_BUNDLE]
               [EXCLUDE_FROM_ALL]
               [source1] [source2 ...])
```

利用源码文件生成目标可执行程序

产物地址依赖于 `RUNTIME_OUTPUT_DIRECTORY`

```cmake
add_excutable(<name> IMPORTED [GLOBAL])
```

```cmake
add_executable(<name> ALIAS <target>)
```

别名产物，`name` 可以被后续的命令使用，比如使用 `if(name)` 来检测，但不能用于修改 `target`，比如 `set_property`、`target_link_libraries` 等