# Script specification

<!-- **脚本已经迁移到：[Thirdparty-OPS](https://github.com/pingcap/thirdparty-ops)** -->

## 1. split_hot_region.py
- 脚本说明
  - 主要是为了快速打散读/写热点
  - `TiDB 3.0` 版本开始已经有了打散表的命令，可结合使用。

- 使用说明
  - 需要将整个项目 `git clone` 到 `tidb-ansible/` 目录下，脚本中使用的相对路径调用 `pd-ctl`
  - 可以使用 `split_hot_region.py -h` 获取帮助

* 注意事项
  + 3.0 较新版本，已经有 `INFORMATION_SCHEMA.TIDB_HOT_REGIONS` 视图可以查看集群热点信息。
  + 在 3.0.10 以上的版本，可以使用浏器打开：`http://{{pd_leader_ip}}:{{pd_status_port}}/dashboard/#/keyvis`, 查看热点图，更加方便直观。

- 使用演示
```shell
# 查看脚本说明
[tidb@ip-172-16-4-51 scripts]$ ./split_hot_region.py -h
usage: split.py [-h] [--th TIDB] [--ph PD] top

Show the hot region details and splits

positional arguments:
  top         the top read/write region number

optional arguments:
  -h, --help  show this help message and exit
  --th TIDB   tidb status url, default: 127.0.0.1:10080
  --ph PD     pd status url, default: 127.0.0.1:2379

# 脚本使用
[tidb@ip-172-16-4-51 scripts]$ ./split_hot_region.py --th 127.0.0.1:10080 --ph 172.16.4.51:2379 1
--------------------TOP 1 Read region messegs--------------------
leader and region id is [53] [27], Store id is 7 and IP is 172.16.4.58:20160, and Flow valuation is 11.0MB, DB name is mysql, table name is stats_buckets
--------------------TOP 1 Write region messegs--------------------
leader and region id is [312] [309], Store id is 6 and IP is 172.16.4.54:20160, and Flow valuation is 61.0MB, DB name is mysql, table name is stats_buckets
The top Read 1 Region is 27
The top Write 1 Region is 309
Please Enter the region you want to split(Such as 1,2,3, default is None):27,26
Split Region 27 Command executed 
Please check the Region 26 is in Top
```

**注意:** 在脚本使用过程中，如果不进行内容输入，将会退出脚本。如果想选择性分裂 `region`，请严格按照提醒输入，如：1,2。

## 2. split_table_region.py
- 脚本说明
  - 主要为了分裂小表 `region`
  - `TiDB 3.0` 版本开始已经有了打散表的命令，新版本可以忽略该脚本

- 使用说明
  - 需要将整个项目 `git clone` 到 `tidb-ansible/` 目录下，脚本中使用的相对路径调用 `pd-ctl`
  - 可以使用 `split_table_region.py -h` 获取帮助

- 使用演示
```shell
# 查看帮助
[tidb@ip-172-16-4-51 scripts]$ ./split_table_region.py -h
usage: table_split.py [-h] [--th TIDB] [--ph PD] database table

Show the hot region details and splits

positional arguments:
  database    database name
  table       table name

optional arguments:
  -h, --help  show this help message and exit
  --th TIDB   tidb status url, default: 127.0.0.1:10080
  --ph PD     pd status url, default: 127.0.0.1:2379

# 分裂小表
[tidb@ip-172-16-4-51 scripts]$ ./split_table_region.py --th 127.0.0.1:10080 --ph 172.16.4.51:2379 test t2
Table t2 Info:
  Region id is 26627, leader id is 26628, Store id is 8 and IP is 172.16.4.59:20160

Table t2 Index info:
Index IN_name info, id is 1:
  Region id is 26627 and leader id is 26628, Store id is 8 and IP is 172.16.4.59:20160
Index In_age info, id is 2:
  Region id is 26627 and leader id is 26628, Store id is 8 and IP is 172.16.4.59:20160
We will Split region: ['26627'], y/n(default is yes): y
Split Region 26627 Command executed 
```

## 3. Stats_dump.py

* 脚本目的
  + 快速拿到相关表的统计信息、表结构、版本信息、生成导入统计信息语句
  + 并且内部方便快速导入表结构和统计信息

* 使用说明
  + 该脚本需要访问 `TIDB` 数据库和 `TiDB status url`，需要安装 `pymysql` 包：`sudo pip install pymysql`
  + 可以使用 `Stats_dump.py -h` 获取帮助
  + 最终会生成一个 `tar` 包，解压后，里面有一个 `schema.sql` 文件，里面有 TiDB 的集群信息。
  + 还原统计信息和表结构可以: `mysql -uroot -P4000 -h127.0.0.1 <schema.sql` (注意要在解压缩的目录中执行还原命令)

* 使用演示

```shell
./Stats_dump.py -h
usage: Stats_dump.py [-h] [-tu TIDB] [-H MYSQL] [-u USER] [-p PASSWORD]
                     [-d DATABASE] [-t TABLES]

Export statistics and table structures

optional arguments:
  -h, --help   show this help message and exit
  -tu TIDB     tidb status url, default: 127.0.0.1:10080
  -H MYSQL     Database address and port, default: 127.0.0.1:4000
  -u USER      Database account, default: root
  -p PASSWORD  Database password, default: null
  -d DATABASE  Database name, for example: test,test1, default: None
  -t TABLES    Table name (database.table), for example: test.test,test.test2,
               default: None
```

* 参数说明
  + `-tu` 后填 TIDB 的 IP 地址和 status 端口，端口默认为 10080
  + `-H` 后填 TiDB 的 IP 地址和连接端口，端口默认是 4000
  + `-u` 为数据库登录账户
  + `-p` 为数据库登录密码
  + `-d` 为需要导出统计信息的库，如果使用该参数，就是代表将会导出对应库所有表的统计信息和表结构。比如填 `-d test1,test2`，就是讲 `test1` 和 `test2` 库下的表的统计信息和表结构导出
  + `-t` 导出对应表的统计信息、表结构。需要注意格式：`database_name.table_name`。比如填 `-t test1.t1,test2.t2`，代表将会导出 test1 库 t1 表和 test2 库 t2 表的表结构和统计信息。

* 注意
  + 如果 `-d` 和 `-t` 都没有指定，默认是导出除了系统表以外所有表的统计信息和表结构。
  + 不会导出 `"INFORMATION_SCHEMA", "PERFORMANCE_SCHEMA","mysql", "default"` 库的表结构和统计信息。

## 4. Cluster_Region.py

* 脚本目的
  + 快速分析当前集群 `region` 是否合并完成
  + 输出当前可以合并的 `region` 个数

* 使用说明
  + 注意相对路径，推荐放入 `tidb-ansible/scripts` 目录下
  + 可以使用 `Cluster_Region.py -h` 获取帮助
  
* 使用演示

```shell
[tidb@xiaohou-vm1 scripts]$ ./Cluster_Region.py -h
usage: Cluster_Region.py [-h] [-pd PD] [-s SIZE] [-k KEYS] [-file FILE]

Show the hot region details and splits

optional arguments:
  -h, --help  show this help message and exit
  -pd PD      pd status url, default: 127.0.0.1:2379
  -s SIZE     Region size(MB), default: 20
  -k KEYS     Region keys, default: 200000
  -file FILE  Files to parse, default: None

[tidb@xiaohou-vm1 scripts]$ python Cluster_Region.py -pd 10.0.1.16:2379 -s 20 -k 200000
Total: 33 
Number of empty regions: 17 
The regions that can be merged: 16 
Size <= 20 and Keys > 200000: 0 
Size > 20 and Keys <= 200000: 0 
Size > 20 and Keys > 200000: 0 
Parser errors: 0

[tidb@xiaohou-vm1 scripts]$ python2 Cluster_Region.py -file region.json 
Total: 33 
Number of empty regions: 17 
The regions that can be merged: 16 
Size <= 20 and Keys > 200000: 0 
Size > 20 and Keys <= 200000: 0 
Size > 20 and Keys > 200000: 0 
Parser errors: 0
```

* 参数说明
  + `-pd` 后填 PD 的 IP 地址和 status 端口，端口默认是 2379
  + `-s` 为 region merge 配置 `max-merge-region-size` 大小
  + `-k` 为 region merge 配置 `max-merge-region-keys` 大小
  + `-file` 用来指定需要解析的 region 信息文件，优先级高于 `-pd` 配置，如果配置，将优先分析 file 文件内容，不访问 pd

* 注意
  + `Number of empty regions` 代表空 region(比如 drop 或者 truncate 之后存留 region，如果很多，则需要开启跨表 region merge，请联系官方)。
  + 如果 `The regions that can be merged` 有值，代表着符合 merge 条件，一般是不同表的 region(默认是不会合并)，或者是还没有来得及合并。
  + 如果 `Size <= 20 and Keys > 200000` 或者 `Size > 20 and Keys <= 200000` 有值，代表 region merge 配置的 `max-merge-region-keys` 或者 `max-merge-region-size` 配置小了，可以考虑调整。
  + `Size > 20 and Keys > 200000` 代表 region merge 条件都不符合
  + `Parser errors` 代表解析异常，请联系官方。
  + 如果服务器上有 `jq` 命令，也可以使用 `jq` 解析：```./bin/pd-ctl -d region | jq ".regions | map(select(.approximate_size < 20 and .approximate_keys < 200000)) | length"```


## 5. Outfile_TiDB.py

* 脚本目的
  + 方便导出 TIDB 数据为 CSV 格式

* 使用说明
  + 需要安装 `MySQLdb`: ` sudo yum -y install mysql-devel;pip install mysql`
  + 可以使用 `Outfile_TiDB.py -h` 获取帮助
  
* 使用演示

```shell
[tidb@xiaohou-vm1 scripts]$ ./Outfile_TiDB.py -h
usage: outfile.py [-h] [-tp MYSQL] [-u USER] [-p PASSWORD] [-d DATABASE]
                  [-t TABLE] [-k FIELD] [-T THREAD] [-B BATCH] [-w WHERE]
                  [-c COLUMN]

Export data to CSV

optional arguments:
  -h, --help   show this help message and exit
  -tp MYSQL    TiDB Port, default: 127.0.0.1:4000
  -u USER      TiDB User, default: root
  -p PASSWORD  TiDB Password, default: null
  -d DATABASE  database name, default: test
  -t TABLE     Table name, default: test
  -k FIELD     Table primary key, default: _tidb_rowid
  -T THREAD    Export thread, default: 20
  -B BATCH     Export batch size, default: 3000
  -w WHERE     Filter condition， for example: where id >= 1, default: null
  -c COLUMN    Table Column, for example: id,name, default: all

[tidb@xiaohou-vm1 scripts]$ python Outfile_TiDB.py
...
Exiting Main Thread, Total cost time is 17.1548991203

[tidb@xiaohou-vm1 scripts]$ cat test.test.0.csv
"id","name"
1,"aa"
2,"bb"
3,""

[tidb@xiaohou-vm1 scripts]$ ./Outfile_TiDB.py -c 'id'
Write test.test.0.csv is Successful, Cost time is 0.000797271728515625
Retrieved select id from test where _tidb_rowid >= 1 and _tidb_rowid < 100001
Exiting Main Thread

[tidb@xiaohou-vm1 scripts]$ cat test.test.0.csv
"id"
1
2
3
```

* 参数说明
  + `-tp` 后填 TIDB 的 IP 地址和 `status` 端口，端口默认是 `10080`
  + `-u` 为 TIDB 的用户，默认为 `root`
  + `-p` 为 TiDB 的密码，默认为空
  + `-d` 指定库名，默认为 `test`
  + `-t` 指定表名，默认为 `test`
  + `-c` 指定表字段，默认为 `all`，导出所有
  + `-k` 指定主键名，默认使用 `_tidb_rowid`
  + `-T` 指定并发数，默认使用 20
  + `-B` 指定每次批量导出数据大小，默认使用 3000
  + `-w` 可加入判断条件，比如 `where id >= 1`，默认为空。
  + `-c` 可添加导出数据的的列

* 注意
  + 多少并发就会生成多少文件，如果数据量很少，`-B` 较大，只有一个文件是正常的
  + 当前只支持单表导出。

# 6. analyze 表相关脚本

* 脚本目的
  + 当我们全量导入一次数据后，表统计信息可能不会非常准确，这个时候最好进行一次全量的 analyze。

* 为了避免安装过多插件，可以使用 shell 脚本 `analyze.sh` 脚本进行统计信息收集，该脚本会对配置的库中所有的表进行 analyze。

* 相关配置说明

| 相关参数    | 说明                                     |
| ----------- | ---------------------------------------- |
| db_name     | 需要 analyze 的库名。                    |
| db_user     | 数据库登录用户名，默认 root              |
| db_port     | 数据库登录端口，默认 4000                |
| db_password | 数据库登录密码，默认 123，注意不能为空。 |
| db_ip       | 数据库登录 IP，默认为 "127.0.0.1"        |
| mysql_path  | MySQL 命令的绝对路径                     |

* 使用演示

```shell
nohup ./analyze.sh >>& analyze.log &

cat analyze.log
mysql: [Warning] Using a password on the command line interface can be insecure.
Analyze table t1
mysql: [Warning] Using a password on the command line interface can be insecure.
Analyze table t2
mysql: [Warning] Using a password on the command line interface can be insecure.
Analyze table t3
mysql: [Warning] Using a password on the command line interface can be insecure.
```

**如果习惯使用 Python 脚本，可以使用 Analyze.py 脚本**

* Analyze.py 脚本和 Analyze.sh 脚本目的一样，功能会更加丰富一点，可以指定表进行 Analyze。

* 使用说明
  * 需要安装 `pymysql` 的包：`sudo pip install pymysql`

* 参数说明：

| 参数 | 说明                                   |
| ---- | -------------------------------------- |
| -h   | 显示脚本使用方式                       |
| -P   | 数据库连接端口，默认 4000              |
| -p   | 数据库密码，默认为空                   |
| -H   | 数据库连接 IP，默认为 127.0.0.1        |
| -u   | 数据库连接账户，默认为 root            |
| -d   | 需要收集统计信息的库，多个使用逗号隔开 |
| -t   | 需要收集统计信息的表，多个使用逗号隔开 |

* 使用演示

```shell
# 可以使用 -h 进行帮助查看
$ ./Analyze.py -h
usage: Analyze.py [-h] [-P PORT] [-H MYSQL] [-u USER] [-p PASSWORD]
                  [-d DATABASE] [-t TABLES]

Update table statistics manually

optional arguments:
  -h, --help   show this help message and exit
  -P PORT      tidb port, default: 4000
  -H MYSQL     Database address, default: 127.0.0.1
  -u USER      Database account, default: root
  -p PASSWORD  Database password, default: null
  -d DATABASE  Database name, for example: test,test1, default: None
  -t TABLES    Table name (database.table), for example: test.test,test.test2,
               default: None

# 更新 test、hdata_migrate 两个库中所有表的统计信息
$ ./Analyze.py -p 123 -d test,hdata_migrate
Analyze table test.t1 Sucessful
Analyze table test.t2 Sucessful
Analyze table test.t3 Sucessful
Statistics for all tables in Analyze test library succeeded~

Analyze table hdata_migrate.ods_risk_operaterecord Sucessful
Statistics for all tables in Analyze hdata_migrate library succeeded~

# 更新 test.t1、hdata_migrate.ods_risk_operaterecord 两张表的统计信息
$ ./Analyze.py -p 123 -t test.t1,hdata_migrate.ods_risk_operaterecord
Analyze table test.t1 Sucessful
Success Analyze all tables
Analyze table hdata_migrate.ods_risk_operaterecord Sucessful
Success Analyze all tables
```

# 7. 导出数据库账号密码相关脚本

* 脚本目的
  + 一次性备份 TiDB/MySQL 所有账户、密码权限

* 使用说明
  * 需要安装 `pymysql` 的包：`sudo pip install pymysql`

* 参数说明：

| 参数 | 说明                                   |
| ---- | -------------------------------------- |
| -h   | 显示脚本使用方式                       |
| -P   | 数据库连接端口，默认 4000              |
| -p   | 数据库密码，默认为空                   |
| -H   | 数据库连接 IP，默认为 127.0.0.1        |
| -u   | 数据库连接账户，默认为 root            |
| -d   | 需要收集统计信息的库，多个使用逗号隔开 |
| -t   | 需要收集统计信息的表，多个使用逗号隔开 |

* 使用演示

```shell
./out_user.py -h
usage: out_user.py [-h] [-P PORT] [-H MYSQL] [-u USER] [-p PASSWORD]

Update table statistics manually

optional arguments:
  -h, --help   show this help message and exit
  -P PORT      tidb port, default: 4000
  -H MYSQL     Database address, default: 127.0.0.1
  -u USER      Database account, default: root
  -p PASSWORD  Database password, default: null

# 导出所有账户密码：
./out_user.py   
User: 'root'@'%' is OK
User: 'tidb'@'%' is OK
User: 'tidb1'@'%' is OK

# 结果：
cat tidb_users.sql       
-- 'root'@'%' 
create user 'root'@'%'; 
update mysql.user set `authentication_string`='' where user='root' and host='%'; 
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' WITH GRANT OPTION;

-- 'tidb'@'%' 
create user 'tidb'@'%'; 
update mysql.user set `authentication_string`='' where user='tidb' and host='%'; 
GRANT USAGE ON *.* TO 'tidb'@'%';

-- 'tidb1'@'%' 
create user 'tidb1'@'%'; 
update mysql.user set `authentication_string`='*445AD2DF92D86C174C647766EB0146B6E70341CB' where user='tidb1' and host='%'; 
GRANT USAGE ON *.* TO 'tidb1'@'%';

```

# 8. 连接指定 tidb 节点执行 kill session 操作

* 脚本目的
  + 由于 TiDB 有多个 TiDB-server 节点，在某些情况下，会有只能通过负载均衡的方式连接数据库的情况。 
  + 有些操作，需要登录对应服务器进行执行，比如发现某个节点有一个大 SQL，这个时候就需要登录对应节点，使用 `kill tidb session_id;` 来 kill 对应 SQL。
  + 本脚本就是为了解决相关情况下，可以快速链接对应节点来执行 kill 命令。

* 使用说明
  * 需要安装 `pymysql` 的包：`sudo pip install pymysql`
  * 默认最多重试 100 次，如果 100 次尝试连接都没有连接到对应节点，则自动退出。

* 参数说明：

| 参数 | 说明                                                     |
| ---- | -------------------------------------------------------- |
| -h   | 显示脚本使用方式                                         |
| -P   | 数据库连接端口，默认 4000                                |
| -p   | 数据库密码，默认为空                                     |
| -H   | 数据库连接 IP，默认为 127.0.0.1                          |
| -u   | 数据库连接账户，默认为 root                              |
| -a   | 需要执行 kill 命令的 TiDB server 节点 IP，默认为空       |
| -ap  | 需要执行 kill 命令的 TiDB server 节点的端口，默认为 4000 |
| -id  | 需要 kill 的 session id                                  |

* 使用示例

```shell
# help 命令
./TiDB_Kill.py -h
usage: TiDB_Kill.py [-h] [-P PORT] [-H MYSQL] [-u USER] [-p PASSWORD]
                    [-a ADVERTISEADDRESS] [-ap ADVERTISEPORT] [-id SESSIONID]

Kill TiDB session by id

optional arguments:
  -h, --help           show this help message and exit
  -P PORT              tidb port, default: 4000
  -H MYSQL             Database address, default: 127.0.0.1
  -u USER              Database account, default: root
  -p PASSWORD          Database password, default: null
  -a ADVERTISEADDRESS  TiDB Address, default: null
  -ap ADVERTISEPORT    TiDB Port, default: 4000
  -id SESSIONID        session id, default: null

# 使用
./TiDB_Kill.py -p 123456 -a 172.16.5.120 -ap 4000 -id 1
172.16.5.120 4000
The TiDB IP is 172.16.5.120
The TiDB IP Port is 4000
Will execute: kill tidb 1, y/n (default:yes)
Connection retries 1
```

# 9. 数据库同步，数据对比

* 脚本目的
  + 当 TiDB 集群做了主从同步，有时候需要对上下游数据做数据对比

* 使用说明
  * 需要安装 `pymysql` 的包：`sudo pip install pymysql`
  * 默认对比内容

* 参数说明：

| 参数 <div style="width:30px"> | 说明                                                                                                                             |
| :----: | -------------------------------------------------------------------------------------------------------------------------------- |
| -h   | 显示脚本使用方式                                                                                                                 |
| -hf  | 上游数据库 IP 和 端口，默认 127.0.0.1:4000                                                                                       |
| -uf  | 上游数据库密码，默认为 root                                                                                                      |
| -pf  | 上游数据库密码，默认为空                                                                                                         |
| -ht  | 下游数据库 IP 和 端口，默认 127.0.0.1:4000                                                                                       |
| -ut  | 下游数据库密码，默认为 root                                                                                                      |
| -pt  | 下游数据库密码，默认为空                                                                                                         |
| -d   | 需要对比的库，逗号隔开，比如 tmp,test                                                                                            |
| -t   | 需要对比的表，比如 tmp.t1,tmp.t2，当前未开发                                                                                     |
| -T   | 数据对比并行度，默认 200，约小越慢，约大，TiKV CPU 使用率越高                                                                    |
| -m   | 同步类型，比如，tidb,tidb: 第一个是上游数据库类型，逗号后是下游数据库类型。tidb,tidb 可以直接对比，有一个为 mysql 的，需要无 udi |
| -v   | 对比算法，默认为 xor，对比的是数据内容，可以配置 count，对比的是 kv 数。只能用 `-m tidb,tidb` 模式下                             |

* 使用示例

```shell
# help 命令
 ./sync_diff.py -h
usage: sync_diff.py [-h] [-hf FMYSQL] [-uf FUSER] [-pf FPASSWORD] [-ht TMYSQL]
                    [-ut TUSER] [-pt TPASSWORD] [-d DATABASE] [-t TABLES]
                    [-T THREAD] [-m MODE] [-v VERIFICATION]

Check tables

optional arguments:
  -h, --help       show this help message and exit
  -hf FMYSQL       Source database address and port, default: 127.0.0.1:4000
  -uf FUSER        Source database account, default: root
  -pf FPASSWORD    Source database password, default: null
  -ht TMYSQL       Target database address and port, default: 127.0.0.1:4000
  -ut TUSER        Target database account, default: root
  -pt TPASSWORD    Target database password, default: null
  -d DATABASE      Database name, for example: test,tmp, default: None
  -t TABLES        Table name, for example: tmp.t,tmp.t1, default: None
  -T THREAD        set tidb_distsql_scan_concurrency, for example: 200,
                   default: 200
  -m MODE          Compare database types, for example: tidb,mysql, default:
                   tidb,tidb
  -v VERIFICATION  Verification method, for example: checksum, default: xor

# 使用
./sync_diff.py -d tmp,tmp1
Check sucessfull, Cost time is 0.014266729354858398s, DB name is: tmp, Table name is:t, bit xor:2175547901
Check sucessfull, Cost time is 0.012514114379882812s, DB name is: tmp, Table name is:t1, bit xor:2393996321
Check sucessfull, Cost time is 0.012146711349487305s, DB name is: tmp, Table name is:t2, bit xor:1665511314
Check sucessfull, Cost time is 0.012310504913330078s, DB name is: tmp, Table name is:t_json, bit xor:0
Check sucessfull, Cost time is 0.01268911361694336s, DB name is: tmp, Table name is:order, bit xor:0
Check sucessfull, Cost time is 0.013577938079833984s, DB name is: tmp1, Table name is:test, bit xor:0
```