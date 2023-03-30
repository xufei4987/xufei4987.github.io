---
layout: post
title:  "golang-migrate使用指南"
subtitle:   " \"golang-migrate\""
date:   2023-03-30 14:04:30 +0800
tags:
- Mysql
author: Youux
header-img: "img/post-bg-2015.jpg"
catalog: true
---
## 简介
随着系统版本的迭代，难免会有对数据库表结构的调整，migrate工具可以明确的标识对于数据库的某次修改，方便对这些修改做部署和回滚

## 安装
```shell
go install -tags 'mysql' github.com/golang-migrate/migrate/v4/cmd/migrate@latest
```

## 使用说明
```shell
# 指定migrations目录，并生成部署和回滚的sql文件，文件内容需要自己补充
migrate create -ext sql -dir ./migrations -seq init_schema

# 在migrations目录下添加新的记录
migrate create -ext sql -dir ./migrations -seq add_user_table

# 对数据库进行表结构升级
migrate -database 'mysql://${user}:${passwd}@tcp(${ip}:${port})/${database}?multiStatements=true' -source file://sql/migrations up

# 查看当前数据库版本
migrate -database 'mysql://${user}:${passwd}@tcp(${ip}:${port})/${database}?multiStatements=true' -source file://sql/migrations version

# 数据库迁移到指定版本
migrate -database 'mysql://${user}:${passwd}@tcp(${ip}:${port})/${database}?multiStatements=true' -source file://sql/migrations goto 1

# 数据库回滚1个版本
migrate -database 'mysql://${user}:${passwd}@tcp(${ip}:${port})/${database}?multiStatements=true' -source file://sql/migrations down 1

# 强制将数据库指定为版本1，不执行任何sql
migrate -database 'mysql://${user}:${passwd}@tcp(${ip}:${port})/${database}?multiStatements=true' -source file://sql/migrations force 1
```

## 参考资料
[golang-migrate][1]

[1]: https://github.com/golang-migrate/migrate