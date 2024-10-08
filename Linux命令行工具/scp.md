
# scp

用于跨设备间复制文件和目录

```sh
scp [-BCpqrv] [-c cipher] [-F ssh_config] [-i identity_file] [-l limit] [-P port] [-S program] 
      [[user@]host1:]src_file1 [...] [[user@]host2:]dst_file2

-B 使用批处理模式
-C 允许压缩
-p 保留原文件的修改时间，访问时间和访问权限
-q 不显示传输进度条
-r 递归复制整个目录
-v 详细方式显示输出
-c 将数据传输进行加密
-F 指定一个替代的 ssh 配置文件
-i 从指定文件中读取传输时使用的密钥文件
-l 限定用户所能使用的带宽，以 Kbit/s 为单位。
-P 指定数据传输用到的端口号
-S 指定加密传输时所使用的程序

# 本地文件拷贝至远程
scp -r /data/xxx root@1.1.1.1:/home/xxx
# 远程文件拷贝至本地
scp root@1.1.1.1:/home/xxx /data/xxx
```
