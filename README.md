# MyCat

### 关键特性 

1. 支持 SQL 92标准 支持Mysql集群，可以作为Proxy使用 支持JDBC连接ORACLE、DB2、SQL Server，将其模拟为MySQL Server使用 支持galera for mysql集群，percona-cluster或者mariadb cluster，提供高可用性数据分片集群，自动故障切换，高可用性 。 

2. 支持读写分离。 

3. 支持Mysql双主多从，以及一主多从的模式 。 

4. 支持全局表。 

5. 支持数据自动分片到多个节点，用于高效表关联查询 。 

6. 跨库join，支持独有的基于E-R 关系的分片策略，实现了高效的表关联查询多平台支持，部署和实施简单。

# Ctrip Dal

### 关键特性

1. 支持2种主流编程语言：Java和C#。

2. 支持2种主流数据库Mysql和MS SqlServer。

3. Ctrip DAL支持流行的分库分表操作。

4. 整个框架包括代码生成器 和客户端。工作模式是使用代码生成器在线生成代码，通过DAL客户端完成数据库操作。生成器具有丰富的向导指引，操作简单清晰，即可以批量生成标准 DAO，也可以在方法级别定制数据库访问。客户端则可以简单的通过标准的maven方式添加依赖


更多数据库中间件链接：http://blog.csdn.net/lichangzhen2008/article/details/44708227


