# MyCat
【Mycat关键特性】

* 支持SQL92标准
* 支持MySQL、Oracle、DB2、SQL Server、PostgreSQL等DB的常见SQL语法
* 遵守Mysql原生协议，跨语言，跨平台，跨数据库的通用中间件代理。
* 基于心跳的自动故障切换，支持读写分离，支持MySQL主从，以及galera cluster集群。
* 支持Galera for MySQL集群，Percona Cluster或者MariaDB cluster
* 基于Nio实现，有效管理线程，解决高并发问题。
* 支持数据的多片自动路由与聚合，支持sum,count,max等常用的聚合函数,支持跨库分页。
* 支持单库内部任意join，支持跨库2表join，甚至基于caltlet的多表join。
* 支持通过全局表，ER关系的分片策略，实现了高效的多表join查询。
* 支持多租户方案。
* 支持分布式事务（弱xa）。
* 支持XA分布式事务（1.6.5）。
* 支持全局序列号，解决分布式下的主键生成问题。
* 分片规则丰富，插件化开发，易于扩展。
* 强大的web，命令行监控。
* 支持前端作为MySQL通用代理，后端JDBC方式支持Oracle、DB2、SQL Server 、 mongodb 、巨杉。
* 支持密码加密
* 支持服务降级
* 支持IP白名单
* 支持SQL黑名单、sql注入攻击拦截
* 支持prepare预编译指令（1.6）
* 支持非堆内存(Direct Memory)聚合计算（1.6）
* 支持PostgreSQL的native协议（1.6）
* 支持mysql和oracle存储过程，out参数、多结果集返回（1.6）
* 支持zookeeper协调主从切换、zk序列、配置zk化（1.6）
* 支持库内分表（1.6）
* 集群基于ZooKeeper管理，在线升级，扩容，智能优化，大数据处理（2.0开发版）。

1.MyCat概念

1.1 MyCAT的架构如下图所示：

<image   src='\Images\o_000000000000000001.png'></image>

MyCAT使用MySQL的通讯协议模拟成一个mysql服务器，并建立了完整的Schema（数据库）、Table（数据表）、User（用户）的逻辑模型，并将这套逻辑模型映射到后端的存储节点DataNode（MySQL Instance）上的真实物理库中，这样一来，所有能使用MySQL的客户端以及编程语言都能将MyCAT当成是MySQLServer来使用，不必开发新的客户端协议。
当MyCAT收到一个客户端发送的SQL请求时，会先对SQL进行语法分析和检查，分析的结果用于SQL路由，SQL路由策略支持传统的基于表格的分片字段方式进行分片，也支持独有的基于数据库E-R关系的分片策略，对于路由到多个数据节点（DataNode）的SQL，则会对收到的数据集进行“归并”然后输出到客户端。

SQL执行的过程，简单的说，就是把SQL通过网络协议发送给后端的真正的数据库上进行执行，对于MySQL Server来说，是通过MySQL网络协议发送报文，并解析返回的结果，若SQL不涉及到多个分片节点，则直接返回结果，写入客户端的SOCKET流中，这个过程是非阻塞模式（NIO）。

DataNode是MyCAT的逻辑数据节点，映射到后端的某一个物理数据库的一个Database，为了做到系统高可用，每个DataNode可以配置多个引用地址（DataSource），当主DataSource被检测为不可用时，系统会自动切换到下一个可用的DataSource上，这里的DataSource即可认为是Mysql的主从服务器的地址。

1.2 逻辑库

与任何一个传统的关系型数据库一样，MyCAT也提供了“数据库”的定义，并有用户授权的功能，下面是MyCAT逻辑库相关的一些概念：
* schema:逻辑库，与MySQL中的Database（数据库）对应，一个逻辑库中定义了所包括的Table。
* table：表，即物理数据库中存储的某一张表，与传统数据库不同，这里的表格需要声明其所存储的逻辑数据节点DataNode，这是通过表格的分片规则定义来实现的，table可以定义其所属的“子表(childTable)”，子表的分片依赖于与“父表”的具体分片地址，简单的说，就是属于父表里某一条记录A的子表的所有记录都与A存储在同一个分片上。
* 分片规则：是一个字段与函数的捆绑定义，根据这个字段的取值来返回所在存储的分片（DataNode）的序号，每个表格可以定义一个分片规则，分片规则可以灵活扩展，默认提供了基于数字的分片规则，字符串的分片规则等。
* dataNode: MyCAT的逻辑数据节点，是存放table的具体物理节点，也称之为分片节点，通过DataSource来关联到后端某个具体数据库上，一般来说，为了高可用性，每个DataNode都设置两个DataSource，一主一从，当主节点宕机，系统自动切换到从节点。
* dataHost：定义某个物理库的访问地址，用于捆绑到dataNode上。

MyCAT目前通过配置文件的方式来定义逻辑库和相关配置：

*    MYCAT_HOME/conf/schema.xml中定义逻辑库，表、分片节点等内容；

*    MYCAT_HOME/conf/rule.xml中定义分片规则；

*    MYCAT_HOME/conf/server.xml中定义用户以及系统相关变量，如端口等。

下图给出了MyCAT一个可能的逻辑库到物理库（MySQL的完整映射关系），可以看出强大的分片能力以及灵活的Mysql集群整合能

<image   src='\Images\r_000000012.png'></image>

2.MyCat基本使用教程

 MyCAT使用Java开发，因为用到了JDK 7的部分功能，所以在使用前请确保安装了JDK 7.0，要求是JDK 7.0以上，并设置了正确的Java环境变量

目前下载的版本是免安装，解压在任意磁盘、根目录下，避免路径中出现中文。

目录下的“Mycat-server-1.2-GA-win.tar.gz”文件，解压后的目录结构如下图所示：

<image   src='\Images\r_MyCAT目录结构.png'></image>

目录说明见下表所示：


<image   src='\Images\QQ截图20170719095646.png'></image>

2.2 启动和停止

安装mycat服务 ：mycate install 

启动mycat服务 ：mycate start

停止mycat服务 ：mycate stop

注意：当修改配置文件后，需要重启mycat服务

3、使用教程

3.1  硬件配置和安装数据库

        本地          mycat    192.168.1.5

        服务器A    mysql    192.168.1.201

        服务器A    mysql    192.168.1.202

 安装MySQL服务器和MySQL客户端，笔者使用的MySQL服务器是免安装版本：mysql-noinstall-5.1.73-winx64，MySQL客户端是：Navicat for MySQL，免安装版本安装方法请参考：http://blog.csdn.net/q98842674/article/details/12094777

 3.2 创建数据库

分别在服务器A、服务器B创建所用的分片数据库；

CREATE database db1;

3.3 配置文件

schema.xml配置文件，因为分库在不同的服务器，因此配置两个datahost；如果在一个datahost中配置多个writeHost，则为主从配置。type="global"时，为全局表

<?xml version="1.0"?>
 <!DOCTYPE mycat:schema SYSTEM "schema.dtd">
 <mycat:schema xmlns:mycat="http://org.opencloudb/">

    <schema name="TESTDB" checkSQLschema="false" sqlMaxLimit="100">
         <!-- auto sharding by id (long) -->
         <table name="travelrecord" dataNode="dn1,dn2" rule="auto-sharding-long" />

         <!-- global table is auto cloned to all defined data nodes ,so can join 
             with any table whose sharding node is in the same data node -->
         <table name="company" primaryKey="ID" type="global" dataNode="dn1,dn2" />
         <table name="goods" primaryKey="ID" type="global" dataNode="dn1,dn2" />

         <!-- random sharding using mod sharind rule -->
         <table name="hotnews" primaryKey="ID" dataNode="dn1,dn2"
             rule="mod-long" />
         <!-- <table name="dual" primaryKey="ID" dataNode="dnx,dnoracle2" type="global"
             needAddLimit="false"/> <table name="worker" primaryKey="ID" dataNode="jdbc_dn1,jdbc_dn2,jdbc_dn3"
             rule="mod-long" /> -->
         <table name="employee" primaryKey="ID" dataNode="dn1,dn2"
             rule="sharding-by-intfile" />
         <table name="customer" primaryKey="ID" dataNode="dn1,dn2"
             rule="sharding-by-intfile">
             <childTable name="orders" primaryKey="ID" joinKey="customer_id"
                 parentKey="id">
                 <childTable name="order_items" joinKey="order_id"
                     parentKey="id" />
             </childTable>
             <childTable name="customer_addr" primaryKey="ID" joinKey="customer_id"
                 parentKey="id" />
         </table>
     </schema>
     <dataNode name="dn1" dataHost="localhost1" database="db1" />
     <dataNode name="dn2" dataHost="localhost2" database="db1" />
     <dataHost name="localhost1" maxCon="1000" minCon="10" balance="0"
         writeType="0" dbType="mysql" dbDriver="native" switchType="1"  slaveThreshold="100">
         <heartbeat>select user()</heartbeat>
         <writeHost host="hostM1" url="192.168.1.201:3306" user="shopuser"
             password="123456">
         </writeHost>
     </dataHost>
     <dataHost name="localhost2" maxCon="1000" minCon="10" balance="0"
         writeType="0" dbType="mysql" dbDriver="native" switchType="1"  slaveThreshold="100">
         <heartbeat>select user()</heartbeat>
         <writeHost host="hostM1" url="192.168.1.202:3306" user="shopuser"
             password="123456">
         </writeHost>
     </dataHost>
 </mycat:schema>

server.xml配置文件，本实例很简单，就只定user，

name：用户名

password：密码

schemas：实例名，和schema.xml定义的schema对应，这里的实例名是虚拟名，也就是对mycat服务的一种别名，是 应用程序以及客户端连接的入口。

<?xml version="1.0" encoding="UTF-8"?>
 <!-- - - Licensed under the Apache License, Version 2.0 (the "License"); 
     - you may not use this file except in compliance with the License. - You 
     may obtain a copy of the License at - - http://www.apache.org/licenses/LICENSE-2.0
     - - Unless required by applicable law or agreed to in writing, software - 
    distributed under the License is distributed on an "AS IS" BASIS, - WITHOUT 
     WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. - See the 
     License for the specific language governing permissions and - limitations 
     under the License. -->
 <!DOCTYPE mycat:server SYSTEM "server.dtd">
 <mycat:server xmlns:mycat="http://org.opencloudb/">
     <system>
     <property name="defaultSqlParser">druidparser</property>
       <!--  <property name="useCompression">1</property>--> <!--1为开启mysql压缩协议-->
     <!-- <property name="processorBufferChunk">40960</property> -->
     <!-- 
    <property name="processors">1</property> 
     <property name="processorExecutor">32</property> 
      -->
         <!--默认是65535 64K 用于sql解析时最大文本长度 -->
         <!--<property name="maxStringLiteralLength">65535</property>-->
         <!--<property name="sequnceHandlerType">0</property>-->
         <!--<property name="backSocketNoDelay">1</property>-->
         <!--<property name="frontSocketNoDelay">1</property>-->
         <!--<property name="processorExecutor">16</property>-->
         <!-- 
            <property name="mutiNodeLimitType">1</property> 0：开启小数量级（默认） ；1：开启亿级数据排序
            <property name="mutiNodePatchSize">100</property> 亿级数量排序批量
            <property name="processors">32</property> <property name="processorExecutor">32</property>
             <property name="serverPort">8066</property> <property name="managerPort">9066</property>
             <property name="idleTimeout">300000</property> <property name="bindIp">0.0.0.0</property>
             <property name="frontWriteQueueSize">4096</property> <property name="processors">32</property> -->
     </system>
     <user name="test">
         <property name="password">test</property>
         <property name="schemas">TESTDB</property>
     </user>
 </mycat:server>

 3.4 登录mycat
在任意有mysql的客户端的机器连接Mycat, 执行以下命令：

 mysql -utest -ptest -h192.168.1.5 -P8066 -DTESTDB   注意：8066登录mycat数据端口，9066登录mycat管理端口(能看到mycat内的配置、以及各个数据库连接情况，很有用)

 3.5 测试
 全局表：company

 mysql> create table company(id int not null primary key,name varchar(100),sharding_id int not null);
       Query OK, 0 rows affected (0.30 sec)
       mysql> explain create table company(id int not null primary key,name varchar(100),sharding_id int not null);
      +-----------+------------------------------------------------------------------------------------------------+
      | DATA_NODE | SQL                                                                                            |
      +-----------+------------------------------------------------------------------------------------------------+
      | dn1       | create table company(id int not null primary key,name varchar(100),sharding_id int not null) |
      | dn2       | create table company(id int not null primary key,name varchar(100),sharding_id int not null) |
      +-----------+------------------------------------------------------------------------------------------------+
      2 rows in set (0.04 sec)

 mysql> insert into company(id,name,sharding_id) values(1,'leader us',10000);
       ERROR 2006 (HY000): MySQL server has gone away
       No connection. Trying to reconnect...
       Connection id:    6
       Current database: TESTDB

  Query OK, 1 row affected (0.03 sec)
        mysql> explain insert into company(id,name,sharding_id) values(1,'leader us',10000);
        +-----------+-----------------------------------------------------------------------+
        | DATA_NODE | SQL                                                                   |
        +-----------+-----------------------------------------------------------------------+
       | dn1       | insert into company(id,name,sharding_id) values(1,'leader us',10000) |
       +-----------+-----------------------------------------------------------------------+
       1 row in set (0.00 sec) 

水平分表：travelrecord

mysql> explain create table travelrecord(id int not null primary key,name varchar(100));
       +-----------+---------------------------------------------------------------------+
       | DATA_NODE | SQL                                                                 |
      +-----------+---------------------------------------------------------------------+
      | dn1       | create table travelrecord(id int not null primary key,name varchar(100)) |
      | dn2       | create table travelrecord(id int not null primary key,name varchar(100)) |
      | dn3       | create table travelrecord(id int not null primary key,name varchar(100)) |
      +-----------+---------------------------------------------------------------------+
      3 rows in set (0.01 sec)

  (7) 三个分片上都插入了3条数据
     mysql> explain insert into travelrecord(id,name) values(1,'hp');
      +-----------+---------------------------------------------+
      | DATA_NODE | SQL                                         |
      +-----------+---------------------------------------------+
      | dn1       | insert into travelrecord(id,name) values(1,'hp') | 
      | dn2       | insert into travelrecord(id,name) values(1,'hp') | 
      | dn3       | insert into travelrecord(id,name) values(1,'hp') | 
      +-----------+---------------------------------------------+
      3 rows in set (0.00 sec)









