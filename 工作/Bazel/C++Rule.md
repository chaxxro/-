# C++Rule

## cc_binary

```
cc_binary(name, deps, srcs, data, additional_linker_inputs, args, compatible_with, copts, defines, deprecation, distribs, dynamic_deps, env, exec_compatible_with, exec_properties, features, hdrs_check, includes, licenses, link_extra_lib, linkopts, linkshared, linkstatic, local_defines, malloc, nocopts, output_licenses, reexport_deps, restricted_to, stamp, tags, target_compatible_with, testonly, toolchains, visibility, win_def_file)
```

- name 目标名称，最好跟程序主入口文件名称相同

- deps 需要链接的其它库的列表

- srcs 源文件列表，可以是自动生成的或人工编写的。如果某个声明的文件在其它规则的 outs 列表中，则当前规则自动依赖于那个规则。可以包含 .h 文件，.h 文件不会被编译仅被引用。任何引用的头文件要么在此属性中声明，要么在依赖的 `cc_library` 的 hdrs 属性中声明

- copts 为 C++ 编译器提供的选项，在编译目标之前，这些选项按顺序添加到 COPTS。这些选项仅仅影响当前目标的编译，而不影响其依赖。选项中的任何路径都相对于当前工作空间而非当前包。也可以在 `bazel build` 时通过 --copts 选项传入

- defines 为 C++ 编译器传递宏定义，实际上会前缀以 `-D` 并添加到 COPTS，这些宏定义会添加到当前目标，以及所有依赖它的目标

- includes 头文件包含目录，实际上会前缀以 `-isystem` 并添加到 COPTS。与 copts 属性不同，这些头文件包含会影响当前目标，以及所有依赖它的目标

- linkopts 为 C++ 链接器传递选项，在链接二进制文件之前，此属性中的每个字符串被添加到 LINKOPTS。

- linkshared 默认 False，用于创建共享库，但它永远不会被链接到依赖的二进制文件中，需要由其他程序手动加载，因此它不应该被视为 `cc_library` 规则的替代品

- linkstatic 默认 True，表示链接用户库的方式。True 表示仅允许静态链接，False 表示使用动态链接

- nocopts 从 C++ 编译命令中移除匹配的选项，此属性的值是正则式，任何匹配正则式的、已经存在的 COPTS 被移除 

- toolchains 提供构建变量的工具链的标签列表 

## cc_import

导入编译好的 C/C++ 库

```
cc_import(name, deps, data, hdrs, alwayslink, compatible_with, deprecation, distribs, exec_compatible_with, exec_properties, features, includes, interface_library, linkopts, objects, pic_objects, pic_static_library, restricted_to, shared_library, static_library, system_provided, tags, target_compatible_with, testonly, toolchains, visibility)

```

- hdrs 预编译库对外发布的头文件列表，依赖此库的规则（dependent rule）会直接将这些头文件包含在源码列表中

- alwayslink 默认 False，如果为 True 则依赖此库的二进制文件会将此静态库归档中的对象文件链接进去，就算某些对象文件中的符号并没有被二进制文件使用

## cc_library

```
cc_library(name, deps, srcs, data, hdrs, additional_compiler_inputs, additional_linker_inputs, alwayslink, compatible_with, copts, defines, deprecation, distribs, exec_compatible_with, exec_properties, features, hdrs_check, implementation_deps, include_prefix, includes, licenses, linkopts, linkstamp, linkstatic, local_defines, restricted_to, strip_include_prefix, tags, target_compatible_with, testonly, textual_hdrs, toolchains, visibility, win_def_file)
```

在 hdrs 声明的头文件构成库的公共接口，`cc_binary` 和 `cc_test` 不会暴露接口，因此它们没有 hdrs 属性

- name 名称

- deps 需要链接到当前库的其它库

- srcs 头文件和源码列表

- hdrs 导出的头文件列表

- copts/nocopts 传递给 C++ 编译命令的参数

- defines 宏定义列表

- include_prefix hdrs 中头文件的路径前缀

- includes 需要添加到编译命令的包含文件列表

- linkopts 链接选项

- linkstatic 是否生成动态库

## cc_proto_library

```
cc_proto_library(name, deps, compatible_with, deprecation, distribs, exec_compatible_with, exec_properties, features, restricted_to, tags, target_compatible_with, testonly, toolchains, visibility)
```

从 .proto 文件生成 C++ 代码

`deps` 必须引用 `proto_library`

```
cc_library(
    name = "lib",
    deps = [":foo_cc_proto"],
)

cc_proto_library(
    name = "foo_cc_proto",
    deps = [":foo_proto"],
)

proto_library(
    name = "foo_proto",
)
```

## cc_shared_library

```
cc_shared_library(name, deps, additional_linker_inputs, compatible_with, deprecation, distribs, dynamic_deps, exec_compatible_with, exec_properties, experimental_disable_topo_sort_do_not_use_remove_before_7_0, exports_filter, features, restricted_to, roots, shared_lib_name, static_deps, tags, target_compatible_with, testonly, toolchains, user_link_flags, visibility, win_def_file)
```