---
layout: post
title:  "RedisShake数据同步"
date:   2023-02-01 20:36:30 +0800
categories:
- redis
author: Youux
---
## redis-shake介绍
redis-shake是阿里云用于Redis数据传输与过滤的开源工具，该工具易于部署，灵活高效。
- Sync（同步）模式支持全量数据迁移和增量数据迁移； 
- Restore（恢复）模式支持将本地RDB文件中的数据恢复至目标端；
- Scan模式是使用SCAN命令来获取源端Redis中的Key，然后使用DUMP命令获取Key的值，最终使用RESTORE命令将Key（与其值）恢复至目标端。
  
不同模式的同步流程如下图所示：

![redis-shake模式介绍](/img/blog/redis-shake.png)

## redis-shake安装
redis-shake下载地址 https://github.com/alibaba/RedisShake/releases/tag/v3.1.8
### 二进制安装
```shell
wget https://github.com/alibaba/RedisShake/releases/download/v3.1.8/redis-shake-linux-amd64.tar.gz
tar -zxvf redis-shake-linux-amd64.tar.gz -C redis-shake-linux-amd64
```
### 源码编译安装
```shell
wget -c https://github.com/alibaba/RedisShake/archive/refs/tags/v3.1.8.tar.gz -O RedisShake-3.1.8.tar.gz
tar -zxvf RedisShake-3.1.8.tar.gz
cd RedisShake-3.1.8 && sh build.sh
```

## 数据同步
### Sync模式
```shell
# 编辑sync.toml，修改源和目标redis的版本、地址、用户名、密码等
type = "sync"
[source]
version = 7.0 # redis version, such as 2.8, 4.0, 5.0, 6.0, 6.2, 7.0, ...
address = "127.0.0.1:36379"
username = "" # keep empty if not using ACL
password = "" # keep empty if no authentication is required
tls = false
elasticache_psync = "" # using when source is ElastiCache. ref: https://github.com/alibaba/RedisShake/issues/373
[target]
type = "standalone" # "standalone" or "cluster"
version = 7.0 # redis version, such as 2.8, 4.0, 5.0, 6.0, 6.2, 7.0, ...
address = "127.0.0.1:46379"
username = "" # keep empty if not using ACL
password = "" # keep empty if no authentication is required
tls = false
[advanced]
dir = "data"
ncpu = 4
pprof_port = 0
metrics_port = 0
log_file = "redis-shake.log"
log_level = "info" # debug, info or warn
log_interval = 5 # in seconds
rdb_restore_command_behavior = "rewrite" # panic, rewrite or skip
pipeline_count_limit = 1024
target_redis_client_max_querybuf_len = 1024_000_000
target_redis_proto_max_bulk_len = 512_000_000

# 然后执行
./bin/redis-shake sync.toml
```
### Restore模式
```shell
redis-cli -h 127.0.0.1 -p 36379
# 查看备份目录
127.0.0.1:36379> CONFIG GET dir
1) "dir"
2) "/data"
# 备份redis，备份文件为/data/dump.rdb
127.0.0.1:36379> bgsave
Background saving started
# 编辑restore.toml，修改目标redis的版本、地址、用户名、密码等
type = "restore"
[source]
version = 7.0 # redis version, such as 2.8, 4.0, 5.0, 6.0, 6.2, 7.0, ...
rdb_file_path = "/data/dump.rdb"
[target]
type = "standalone" # standalone or cluster
version = 7.0 # redis version, such as 2.8, 4.0, 5.0, 6.0, 6.2, 7.0, ...
address = "127.0.0.1:46379"
username = "" # keep empty if not using ACL
password = "" # keep empty if no authentication is required
tls = false
[advanced]
dir = "data"
ncpu = 3
pprof_port = 0
metrics_port = 0
log_file = "redis-shake.log"
log_level = "info" # debug, info or warn
log_interval = 5 # in seconds
rdb_restore_command_behavior = "rewrite" # panic, rewrite or skip
pipeline_count_limit = 1024
target_redis_client_max_querybuf_len = 1024_000_000
target_redis_proto_max_bulk_len = 512_000_000

# 然后执行
./bin/redis-shake restore.toml
```

## 参考资料
[通过redis-shake将备份数据恢复至阿里云][1]

[redis-shake github wiki][2]

[1]: https://help.aliyun.com/document_detail/116378.htm
[2]: https://github.com/alibaba/RedisShake/wiki