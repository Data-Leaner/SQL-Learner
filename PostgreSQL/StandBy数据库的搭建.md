## Standby数据库
通常是一台主数据库提供读写，然后把数据同步到另一台备数据库上，这台备数据库不断应用（apply）从主数据库发来的变化数据。这台备服务器不能提供写服务，只提供只读服务。在PostgreSQL中能提供读写全功能的服务器，称为Primary database或masterdatabase ；若备份数据库在接收主数据库同步数据和应用同步数据时，不能提供只读服务，则该备份数据库称之为warm standby server ;而如果在接收同步数据和应用同步数据时也能提供只读操作，则称该备份数据库为hot standby servero hot standby功能是PostgreSQL 9.X版本之后提供

### PITR原理
PostgreSQL在数据目录的pg_xlog子目录中始终维护着一个WAL日志文件。这个日志文件用于记录数据库数据文件的每次改变。当初设计这个日志文件的主要目的是为了在数据库异常崩溃后，能够通过重放最后一次checkpoint点之后的日志文件，把数据库推到一致状态。事实上，这种日志文件机制，也提供了一种数据库热备份方案：在把数据库使用文件系统的方式备份出来的同时把相应的WAL日志也备份出来。虽然直接拷贝数据库数据文件会导致拷贝岀来的文件不一致（比如拷贝的多个数据文件不是同一个时间点文件；拷贝一个8KB的数据块时，也存在不一致的情况：假设刚拷贝完前4KB的块，数据库又写了后4KB的块内容,那么所拷贝的块就不是一个完整的数据块），但因为有了 WAL日志，即使备份出来的数据块不一致，也可以重放备份开始后的WAL日志，把备份的内容推到一致状态。由此可见，有了 WAL日志之后，备份数据库时不再需要完美的一致性备份了，备份中任何数据的非一致性都会被重放WAL日志文件进行纠正，所以在备份数据库时可以通过简单的cp命令或tar等拷贝、备份文件来实现数据库的在线备份。不停地重放WAL日志就可以把数据推到备份结束后的任意一个时间点，这就是基于时间点的备份，英文为“Point-in-Time Recovery”，缩写为PITR。

### WAL日志归档
所谓把WAL日志归档，其实就是把在线的WAL H志备份出来。在PostgreSQL中配置归档的方法是配置参数`archive command`,参数的配置值可以是一个Unix命令，此命令把WAL

**参考全局参数archive_mode & archive_command**

### 流复制
#### 同步方式
在Primary数据库提交事务时，一定会等到WAL日志传递到Standby后才会返回，这样可以做到Standby数据库完全与Primary数据库同步，没有一点落后，当主备库切换时使用同步方式可以做到零数据丢失。
#### 异步方式
异步方式，则是事务提交后不必等日志传递到Standby就即可返回，所以Standby数据库通常也只比Primary数据库落后很少（如几秒）的时间。

### Standby的运行原理
PostgreSQL数据库异常中止后，数据库刚重启时，会重放停机前最后一个checkpoint点之后的WAL日志，在把数据库恢复到停机的状态后，自动进入正常的状态，可以接收其他用户
的查询和修改。

### 创建Standby的步骤
#### 生成一个基础备份
1.以数据库超级用户身份连接到数据库
```SQL
select pg_start_backup('babel')
```
- 设置写日志标志
`XLogCtl->Insert.fbrcePageWrites = true`,也就是把这个标志设置为true后，数据库会把变化的整个数据块都记录到数据库中，而不仅仅是块中记录的变化。
- 强制发生一次`checkpoint`点。
强制发生一次`checkpoint`,也是为了把前面的脏数据都刷到磁盘中，这样从这之后产生的日志就记录整个数据块，可以保证恢复的正确。
2. 执行备份
3. 以数据库超级用户身份连接到数据库
```SQL
select pg_stop_backup()
```

#### 基本备份拷贝到备机上
配置`recovery.conf`把备库启动在Standby模式下

### pg_basebackup命令行工具
这个工具会把整个数据库实例的数据都拷贝出来，而不只是把实例中的部分（如某个数据库或某些表）单独备份出来。该工具使用`replication`协议连接到数据库实例上，所以主数据库中的`pg_hba.conf`必须允许replication连接，也就是在pg_hba.conf中必须有类似下面的内容：
```bash
local replication  osdba              trust
local replication  osdba              ident
host  replication  osdba  0.0.0.0/0   md5
```

```bash
pg_basebackup [option...]
```

-	`-D directory`或`--pgdata=directory` :指定把备份写到哪个目录。如果这个目录或这个目录路径中的各级父目录不存在，则pg_basebackup就会自动创建这个目录。如果这个目录存在，但目录不为空，则会导致pg_basebackup执行失败。如果备份的输出是tar结果（指定-F tar,后面会介绍这个选项），而-D参数后的目录名写成了	（即中划线），则备份会输出到标准输出中，以便管道与其他工具配合使用。

-	`-F format`或`--format=format`：指定输出的格式。`format`指一种格式，它目前又支持两种格式，一种是原样输出，即把主数据库中的各个数据文件、配置文件、目录结构都完全一样地写到备份目录，这种情况下`format`指定为`p`或`plain` ;另一种是tar格式，相当于把输出的备份文件打包到一个tar文件中，这种情下`format`应为`t`或`tar`。

- `-X`或`--xlog` :备份时会把备份中产生的xlog文件也自动备份出来，这样才能在恢复数据库时，应用这些Xlog文件把数据库推到一个一致点，然后真正打开这个备份的数据库。这个选项与选项`-X fetch`是完全L样的。使用这个选项，需要设置`wal_keep_segments`参数，以保证在备份过程中，需要的WAL日志文件不会被覆盖。

-	`-X method` 或`--xlog-method=method` : method 可以取的值为`f`、`ffetch`、`s`、`stream`，`f`与`fetch相同`，其意义与`-x`参数是一样的。`s`与`stream`表示的意思也相同，表示备份开始后，启动另一个流复制连接从主库接收WAL日志。这种方式避免了使用`-Xf`时，主库上的WAL日志有可能被覆盖而导致失败的问题。但这种方式需要与主库建两个连接，因此使用这种方式时，主库的`max_wal_senders`参数至少要设置为2或大于2的值。

-	`-z`或`-gzip`:仅能与tar输出模式配合使用，表明输出的tar备份包是经过gzip压缩的，相当于生成了一个*.tar.gz的备份包。

-	`-Z level`或`--compress=level` ：指定gzip的压缩级别，可以选1〜9的数字，与gzip命令中的压缩级别是一样的，9表示最高压缩率，但也最耗CPU。

-	`-c fast|spread` 或 `--checkpoint=fast|spread`:设置 checkpoint 的模式是`fast`还是 `spread`

-	`-1 label`或`--label=label` ：指定备份的一个标识，备份的标识是一个任意字符串，便于今后维护人员识别这个备份，该标识就是手工做基础备份时运行`select pg_start_backup('lable')`传递给pg_start_backup函数的参数。在备份集中有一个文件叫`backup_label`,这里面除了记录开始备份时WAL日志的起始位置、Checkpoint的WAL日志位置、备份的开始时间外，也记录了这个标识串的信息。

-	`-P`或`--progress`：允许在备份过程中实时地打印备份的进度。当然这个打印的进度不是百分之百精确的，因为在备份过程中，数据库的数据还会发生变化，还会不断产生一些WAL日志。
-	`-V`或`--verbose`：详细模式，使用了-P参数后，还会打印出正在备份的具体文件的信息。

-	`-V`或`--version`：打印pg_basebackup的版本后退出。

-	`-?`或`--help`：显示帮助信息后退出。下面一些参数控制连接数据库的参数。

-	`-h host`或`--host=host`：指定连接的数据库的主机名或IP地址。

-	`-p port`或`--port=port`：指定连接的端口。

-	`-s interval`或`--status-interval=interval` ：指定向服务器端周期反馈状态的秒数，如果服务器上配置了流复制的超时，在使用`-xlog=stream`选项时，则需要设置这个参数，默认值为10秒。如果设置为0,表示不向服务器反馈状态。

-	`-U username`或`--usemame=usemame`：指定连接的用户名。

-	`-w`或`--no-password`：指定从来不提示输入密码。

-	`-W`或`--password`,强制让pg_basebackup岀现输入密码的提示。
