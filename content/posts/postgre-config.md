---
title: "postgresql配置远程访问、账号授权"
date: 2021-04-06T10:31:03+08:00
tags: ["database", "postgresql"]
draft: false
---

## 前言

因为工作中用到PostgreSQL。数据库装在物理机中，对外提供服务。



## 远程访问

1. 查看配置文件位置

   ```shell
   # 连接pgsql
   psql -U <用户名>
   # 查看配置文件位置
   select name,setting from pg_settings where category='File Locations';
   ```

2. 修改服务监听地址

   ```shell
   # 修改配置文件postgresql.conf。上述步骤获取
   vim <config_file 配置文件地址>
   # 修改监听地址。打开配置项  listen_addresses
   listen_addresses = '*'
   ```

3. 修改`Client Authentication`

   ```shell
   # 修改配置文件pg_hba.conf。第一步获取
   vim <hba_file 配置文件地址>
   # 修改`IPv4 local connections:`行配置。允许所有用户所有IP以MD5加密方式访问。
   IPv4 local connections:
   host    all             all             0.0.0.0/0               md5
   ```

4. 重启pgsql服务

   `sudo systemctl restart postgresql.service`



## 账号授权

1. 创建新的数据库角色

   `create user <user_name> with password '******';`

2. 更新用户默认为只读事务

   `alter user <user_name> set default_transaction_read_only=on;`

3. 给用户赋予所有库的USAGE权限

   `GRANT USAGE ON SCHEMA public to <user_name>;`

4. 进入具体数据库，赋予用户该库具体表的读取权限。

   `\c <database>`

   `grant select on all tables in schema public to <user_name>;`



## 参考:

[postgres文档](http://postgres.cn/docs/12/sql-grant.html)