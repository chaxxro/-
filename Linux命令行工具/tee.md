# tee

用于将一个命令的输出同时显示在终端上（或重定向到一个文件），并复制一份输出到另一个文件

- 显示并保存输出：将一个命令的输出同时显示在终端上，并将输出复制到一个或多个文件中。这对于需要查看命令输出并进行日志记录的场景非常有用
- 管道操作：与管道 `|` 结合使用，以便在一个命令序列中同时处理和保存输出

```sh
command | tee output_file [output_file2 ...]
# tee 命令会覆盖目标文件中的现有内容。如果需要在文件末尾追加输出，可以使用 -a 选项
```

