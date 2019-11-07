## Postgres-XC
Postgres-XC是基于PostgreSQL数据库实现的真正的数据水平拆分的分布式数据库。

- Coordinators
协调器。应用程序连接Postgres-XC,实际连接的就是Coordinators。它对SQL进行分析，并生成全局的执行计划。为了节省机器，通常此服务与数据节点部署在一起。

- Datanodes
数据节点，或简单地称之为Nodes,这是实际数据存储的节点，执行本地的 SQL。

- GTM
Global Transaction Manage 的缩写，即全局事务管理器，它管理全局的事务ID(GXID即Global Transaction ID）。PostgreSQL使用事务ID来控制多版本,在Postgres-XC中由GTM提供统一的事务ID的分配和管理。

- GTM Standby
GTM的备节点。

- GTM-Proxy
GTM需要与所有的Coordinators通信，为了降低压力，可以在每台Coordinator 机器上部署1个 GTM-Proxy。

Coordinators和Datanodes的主程序，实际上Coordinators和Datanodes的主程序都是postgres,只是通过不同的启动参数来表示不同的角色。

### 命令
-	initdb:
用于创建 Coordinators 和 Datanodes 实例。
    -	--coordinator：作为 Coordinator 启动。
    - --datanode：作为 Datanode 启动。
    - --restoremode：启动到一种特有的恢复模式。

-	postgres
Coordinators 和 Datanodes 的主程序。

- pg_ctl
    - -Z NODE-TYPE :指定管理的节点类型，类型可以为“coordinator”或“datanode”

Datanodes基本上就是一个单机版本的PostgreSQL实例，可以单独连接到Datanodes上执行一些本地操作。Coordinators也相当于一个单机版本的PostgreSQL实例，只是它上面没有数据，它把SQL请求解析为全局的执行计划，然后分发到各个Datanodes上执行。
