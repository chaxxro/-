
# watch

定期执行程序，展示输出

```sh
watch [options] command
-d 高亮显示变化的区域
-n 刷新间隔

watch -d ls -l
watch -d 'ls -l | fgrep joe'
```