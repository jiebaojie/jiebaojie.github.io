---
layout: post
notes: true
subtitle: "【开源代码】ZEBRA"
comments: false
author: "大众点评网"
date: 2018-10-03 00:00:00

---

GitHub地址：[https://github.com/dianping/zebra/](https://github.com/dianping/zebra)


*   目录
{:toc }

DianPing data access layer (DAL) for MySQL.

# 简介

Zebra是点评内部使用的数据库访问层中间件，它具有以下的功能点：

1.	配置集中管理，动态刷新
2.	支持读写分离、分库分表
3.	丰富的监控信息在CAT上展现

其中的三个组件的功能分别是：

*	zebra-api : 最主要的访问层中间件
*	zebra-ds-monitor-client：基于CAT的监控(可选)
*	zebra-dao：基于MyBatis的异步化的DAO组件(可选)

# 编译

1.	git clone https://github.com/dianping/zebra.git
2.	git checkout mvn-repo
3.	拷贝里面的mvn依赖到本地仓库（第2和3步骤主要是为了使用zebra-ds-monitor-client中的CAT监控）
4.	mvn clean install -DskipTests

# zebra-api用户手册

[zebra-api用户手册](https://github.com/dianping/zebra/blob/master/zebra-api/README.md)

## 简介

Zebra是在jdbc协议层上开发的数据库访问层中间件，它具有以下的功能点：

1.	配置集中管理，动态刷新
2.	支持读写分离、分库分表
3.	SQL流控

## 使用说明

### 第一步：添加POM依赖

目前的最新版本为2.8.3，并配合数据监控组件zebra-ds-monitor-client一起使用

	<dependency>
		<groupId>com.dianping.zebra</groupId>
		<artifactId>zebra-api</artifactId>
		<version>${version}
		</version>
	</dependency>
	<dependency>
		<groupId>com.dianping.zebra</groupId>
		<artifactId>zebra-ds-monitor-client</artifactId>
		<version>${version}</version>
	</dependency>
	
### 第二步：通过Spring方式使用

#### SingleDataSource（单数据源）配置

	<bean id="dataSource" class="com.dianping.zebra.single.jdbc.SingleDataSource" init-method="init" destroy-method="close">
		<!--必填，填写atlas的地址-->
		<property name="jdbcUrl" value="{jdbcUrl}"/>
		<!--必填，填写atlas的用户名-->
		<property name="user" value="{user}" />
		<!--必填，填写atlas的密码-->
		<property name="password" value="{password}" />
		<!-- 选填，默认值为"c3p0",还支持"tomcat-jdbc"或者"druid"-->
		<property name="driverClass" value="com.mysql.jdbc.Driver" />
		<property name="poolType" value="c3p0" />
		<!-- c3p0的minPoolSize,该值对应tomcat-jdbc或druid的"minIdle" -->
		<property name="minPoolSize" value="5" />
		<!-- c3p0的maxPoolSize,该值对应tomcat-jdbc或druid的"maxActive" -->
		<property name="maxPoolSize" value="30" />
		<!-- c3p0的initialPoolSize,该值对应tomcat-jdbc或druid的"initialSize" -->
		<property name="initialPoolSize" value="5" />
		<!-- c3p0的checkoutTimeout,该值对应tomcat-jdbc或druid的"maxWait" -->
		<property name="checkoutTimeout" value="1000" />
		<property name="maxIdleTime" value="1800" />
		<property name="idleConnectionTestPeriod" value="60" />
		<property name="acquireRetryAttempts" value="3" />
		<property name="acquireRetryDelay" value="300" />
		<property name="maxStatements" value="0" />
		<property name="maxStatementsPerConnection" value="100" />
		<property name="numHelperThreads" value="6" />
		<property name="maxAdministrativeTaskTime" value="5" />
		<property name="preferredTestQuery" value="SELECT 1" />
	</bean>
	
### GroupDataSource（读写分离数据源）配置

	<bean id="dataSource" class="com.dianping.zebra.group.jdbc.GroupDataSource" init-method="init">
		<!-- jdbcRef决定需要访问的库的名字，这里指访问的是test库 -->
		<property name="jdbcRef" value="test" />
		<!-- 选择使用背后使用哪种数据源，"c3p0"或者"tomcat-jdbc"，可以不配，默认值为"c3p0" -->
		<property name="poolType" value="c3p0" />
		<!-- 选择配置源，默认是remote方式，这里使用的是local方式，意味着配置是本地文件 -->
		<property name="configManagerType" value="local" />
		<property name="maxIdleTime" value="1800" />
		<property name="idleConnectionTestPeriod" value="60" />
		<property name="acquireRetryAttempts" value="3" />
		<property name="acquireRetryDelay" value="300" />
		<property name="maxStatements" value="0" />
		<property name="maxStatementsPerConnection" value="100" />
		<property name="numHelperThreads" value="6" />
		<property name="maxAdministrativeTaskTime" value="5" />
		<property name="preferredTestQuery" value="SELECT 1" />
	</bean>
	
创建 test.properties 到 src/main/resources下，文件名必须是以jdbcRef.properties命名

	groupds.dbname.mapping=(db2:1,db3:1),(db1,db2)

	#zebra.ds.db1.jdbc
	ds.db1.jdbc.active=true
	ds.db1.jdbc.url=jdbc:mysql://192.168.0.1:3306/test?characterEncoding=UTF8&socketTimeout=60000
	ds.db1.jdbc.username=root
	ds.db1.jdbc.password=123456
	ds.db1.jdbc.driverClass=com.mysql.jdbc.Driver
	#other properties
	ds.db1.jdbc.properties=idleConnectionTestPeriod=60&acquireRetryAttempts=50&acquireRetryDelay=300&maxStatements=0
	ds.db1.jdbc.testReadOnlySql=SELECT 1

	#zebra.ds.db2.jdbc
	ds.db2.jdbc.active=true
	ds.db2.jdbc.url=jdbc:mysql://192.168.0.1:3306/test?characterEncoding=UTF8&socketTimeout=60000
	ds.db2.jdbc.username=sa
	ds.db2.jdbc.password=123456
	ds.db2.jdbc.driverClass=com.mysql.jdbc.Driver
	ds.db2.jdbc.testReadOnlySql=SELECT 1

	#zebra.ds.db3.jdbc
	ds.db3.jdbc.active=true
	ds.db3.jdbc.url=jdbc:mysql://192.168.0.1:3306/test?characterEncoding=UTF8&socketTimeout=60000
	ds.db3.jdbc.username=root
	ds.db3.jdbc.password=123456
	ds.db3.jdbc.driverClass=com.mysql.jdbc.Driver
	ds.db3.jdbc.properties=idleConnectionTestPeriod=80&acquireRetryAttempts=50&acquireRetryDelay=300&maxStatements=1
	ds.db3.jdbc.testReadOnlySql=call readonly()
	
#### 本地(local)配置和远端(remote)配置

如果要实现配置的集中化管理和动态刷新功能，请自行实现RemoteConfigService中的相应方法。点评内部是基于内部的配置系统来进行配置的，在开源版本中剥离了这个依赖，所以请自行实现。 目前仅支持local方式的使用GroupDataSource

#### 事务处理

zebra是一个读写分离的数据源，如果在事务中，那默认所有在事务中的操作均将路由到主库进行操作。

#### 特殊配置介绍

如果业务对主从延迟要求很高，不能容忍一点延迟，比如支付等业务，可以根据需要配置两个数据源，其中一个只走读库，另外一个只走写库，可以在spring的配置中加入如下的property。一般情况下，除非对主从延迟要求很高，一般应用不建议使用该配置。默认值是master-slave。

	<bean id="readDs" class="com.dianping.zebra.group.jdbc.GroupDataSource" init-method="init">
		<property name="jdbcRef" value="tuangou2010" />
		<property name="routerType" value="slave-only" /> <!-- 只取已配的读账号连接，如需使用请升级到2.8.3版本以上 -->
	</bean>

	<bean id="writeDs" class="com.dianping.zebra.group.jdbc.GroupDataSource" init-method="init">
		<property name="jdbcRef" value="tuangou2010" />
		<property name="routerType" value="master-only" /><!-- 只取已配的写账号连接，如需使用请升级到2.8.3版本以上 -->
	</bean>
	
### ShardDataSource（分库分表数据源）配置

	<bean id="zebraDs0" class="com.dianping.zebra.single.jdbc.SingleDataSource" init-method="init" destroy-method="close">
		<property name="jdbcUrl" value="jdbc:mysql://192.168.0.1:6002/zebraatlas?characterEncoding=UTF8"/>
		<property name="user" value="zebraatlas_wr" />
		<property name="password" value="FsdRg5d3j01" />
		<property name="driverClass" value="com.mysql.jdbc.Driver" />
		<property name="poolType" value="c3p0" />
		<property name="minPoolSize" value="2" />
		<property name="maxPoolSize" value="10" />
		<property name="initialPoolSize" value="2" />
		<property name="checkoutTimeout" value="2000" />
		<property name="maxIdleTime" value="1800" />
		<property name="idleConnectionTestPeriod" value="60" />
		<property name="acquireRetryAttempts" value="3" />
		<property name="acquireRetryDelay" value="300" />
		<property name="maxStatements" value="0" />
		<property name="maxStatementsPerConnection" value="100" />
		<property name="numHelperThreads" value="6" />
		<property name="maxAdministrativeTaskTime" value="5" />
		<property name="preferredTestQuery" value="SELECT 1" />
	</bean>
	<bean id="zebraDs1" class="com.dianping.zebra.single.jdbc.SingleDataSource" init-method="init" destroy-method="close">
		<property name="jdbcUrl" value="jdbc:mysql://192.168.0.1:6002/zebraatlas?characterEncoding=UTF8"/>
		<property name="user" value="zebraatlas_wr" />
		<property name="password" value="FsdRg5d3j01" />
		<property name="driverClass" value="com.mysql.jdbc.Driver" />
		<property name="poolType" value="c3p0" />
		<property name="minPoolSize" value="2" />
		<property name="maxPoolSize" value="10" />
		<property name="initialPoolSize" value="2" />
		<property name="checkoutTimeout" value="2000" />
		<property name="maxIdleTime" value="1800" />
		<property name="idleConnectionTestPeriod" value="60" />
		<property name="acquireRetryAttempts" value="3" />
		<property name="acquireRetryDelay" value="300" />
		<property name="maxStatements" value="0" />
		<property name="maxStatementsPerConnection" value="100" />
		<property name="numHelperThreads" value="6" />
		<property name="maxAdministrativeTaskTime" value="5" />
		<property name="preferredTestQuery" value="SELECT 1" />
	</bean>
	<bean id="zebraDs2" class="com.dianping.zebra.single.jdbc.SingleDataSource" init-method="init" destroy-method="close">
		<property name="jdbcUrl" value="jdbc:mysql://192.168.0.1:6002/zebraatlas?characterEncoding=UTF8"/>
		<property name="user" value="zebraatlas_wr" />
		<property name="password" value="FsdRg5d3j01" />
		<property name="driverClass" value="com.mysql.jdbc.Driver" />
		<property name="poolType" value="c3p0" />
		<property name="minPoolSize" value="2" />
		<property name="maxPoolSize" value="10" />
		<property name="initialPoolSize" value="2" />
		<property name="checkoutTimeout" value="2000" />
		<property name="maxIdleTime" value="1800" />
		<property name="idleConnectionTestPeriod" value="60" />
		<property name="acquireRetryAttempts" value="3" />
		<property name="acquireRetryDelay" value="300" />
		<property name="maxStatements" value="0" />
		<property name="maxStatementsPerConnection" value="100" />
		<property name="numHelperThreads" value="6" />
		<property name="maxAdministrativeTaskTime" value="5" />
		<property name="preferredTestQuery" value="SELECT 1" />
	</bean>
	<bean id="zebraDs3" class="com.dianping.zebra.single.jdbc.SingleDataSource" init-method="init" destroy-method="close">
		<property name="jdbcUrl" value="jdbc:mysql://192.168.0.1:6002/zebraatlas?characterEncoding=UTF8"/>
		<property name="user" value="zebraatlas_wr" />
		<property name="password" value="FsdRg5d3j01" />
		<property name="driverClass" value="com.mysql.jdbc.Driver" />
		<property name="poolType" value="c3p0" />
		<property name="minPoolSize" value="2" />
		<property name="maxPoolSize" value="10" />
		<property name="initialPoolSize" value="2" />
		<property name="checkoutTimeout" value="2000" />
		<property name="maxIdleTime" value="1800" />
		<property name="idleConnectionTestPeriod" value="60" />
		<property name="acquireRetryAttempts" value="3" />
		<property name="acquireRetryDelay" value="300" />
		<property name="maxStatements" value="0" />
		<property name="maxStatementsPerConnection" value="100" />
		<property name="numHelperThreads" value="6" />
		<property name="maxAdministrativeTaskTime" value="5" />
		<property name="preferredTestQuery" value="SELECT 1" />
	</bean>

	<bean id="zebraDS" class="com.dianping.zebra.shard.jdbc.ShardDataSource" init-method="init">
		<property name="dataSourcePool">
			<map>
				<entry key="id0" value-ref="zebraDs0"/>
				<entry key="id1" value-ref="zebraDs1"/>
				<entry key="id2" value-ref="zebraDs2"/>
				<entry key="id3" value-ref="zebraDs3"/>
			</map>   
		</property>
		<property name="routerFactory">
			<bean class="com.dianping.zebra.shard.router.builder.XmlResourceRouterBuilder">
				<constructor-arg value="spring/shard/router-local-rule.xml"/>
			</bean>
		</property>
		<!--业务自行调整并发查询的线程池参数 -->
		<property name="parallelCorePoolSize" value="16" />
		<!--业务自行调整并发查询的线程池参数 -->
		<property name="parallelMaxPoolSize" value="32" />
		<!--业务自行调整并发查询的线程池参数 -->
		<property name="parallelWorkQueueSize" value="500" />
		<!--业务自行调整逻辑SQL在线程池里面的超时时间，可以在beta环境设置的大一点 -->
		<property name="parallelExecuteTimeOut" value="3000" />
	</bean>
	
在src/main/resources目录下创建spring/shard/router-local-rule.xml文件

	<?xml version="1.0" encoding="UTF-8"?>
	<router-rule>
		<table-shard-rule table="Feed" generatedPK="id">
			<shard-dimension dbRule="(#id#.intValue() % 8).intdiv(2)"
				dbIndexes="id0,id1,id2,id3"
				tbRule="#id#.intValue() % 2"
				tbSuffix="alldb:[0,7]"
				isMaster="true">
			</shard-dimension>
		</table-shard-rule>
	</router-rule>
	
# zebra-api分库分表

[zebra-api分库分表](https://github.com/dianping/zebra/blob/master/zebra-api/README_SHARD.md)

## 简介

在数据库单表记录行数达到很大的量的时候，需要对这个单表进行水平拆分。zebra-api就是数据库水平拆分的解决方案。

1.	对业务透明，业务写的SQL只知道逻辑的库名和逻辑的表名
2.	对大部分SQL都支持

目前，暂不支持路由配置动态变更，因为分库分表的路由逻辑一定是一开始设计好的，并且容量够大的，所以无需动态变更。

## 原理

单表水平拆分时，需要指定对某一个维度来进行拆分。那么什么是维度？举个例子，例如订单库中的订单表中，每一条订单记录必定是某个用户下的订单，那么订单表可以按照用户这个维度来进行划分。拆分之后，物理上订单库会在多个数据库集群中，每个集群中都可能会有多个订单表。但是，zebra使得业务在逻辑上看到的是单库单表。那么对于以下这条语句，zebra的执行顺序是什么呢？

	select * from Order where userId = "xxxxx"
	
执行顺序：

1.	zebra会对上面的SQL进行解析，按照分库分表的配置配置的维度解析出该SQL需要按照分库分表的维度(userId)和值xxxxx。
2.	按照分库分表的配置配置的路由策略，得出该SQL会最终路由到哪个物理数据库(假设是OrderDB0)和物理表(假设是Order10)。
3.	改写SQL语句，替换表名。
4.	将该SQL语句放到真正的数据库上OrderDB0上执行。
5.	将以上执行返回的结果，进行各种整理(此处省略其余各种复杂细节)返回

## 注意事项

1.	分库分表后，业务的SQL必须要带上维度，因为如果没有维度，那么如果是Insert语句，则会直接报错，因为无法知道该行语句将要往哪个数据库上执行。如果是Select、Update或者Delete语句，那么会将该SQL对所有的库和表进行执行，试想一下，如果表被拆成了1024张表，那么，这个不带维度的SQL可能会执行1024次，性能极差。
2.	分库分表后，业务的SQL最好要只有一个维度。因为如果有多个维度，就涉及到将主维度的数据同步到其他维度中去。这里会有一定的延迟。
3.	分库分表后，不能写诸如join，group by，limit等复杂的SQL。一来，性能差；二来，zebra未必支持。

## 接入之前的准备工作

1.	开发需要排查业务所有的SQL，找出唯一的那个维度，如果不唯一，尽可能的优化业务，使得维度尽量的少。
2.	和DBA确定需要分的表，以及需要分多少张表。确认好之后，让DBA在各个环境进行建库建表。
3.	指定分库分表配置。每个接入的分库分表规则都有一个名字，一般以需要分库的库名小写来起，比如welife，该规则可以在lion上的shardds.welife.shard看到。

规则的json格式如下：

	{
		"tableShardConfigs":[
			{
				"tableName":"welife_users",
				"dimensionConfigs":[
					{
						"dbRule":"crc32(#bid#)%10",
						"dbIndexes":"welife[0-9]",
						"tbRule":"(crc32(#bid#)/10).toLong()%10",
						"tbSuffix":"everydb:[0,9]",
						"isMaster":true
					}],
				"generatedPK":"uid"
			}
		]
	}
	
这里具体解释一下这个配置的意思：

1.	tableName: 需要分库分表的表名，该例中需要水平拆分的表是welife_users。
2.	dimensionConfigs: 指定这个表的维度规则，可以有多个维度规则，该例中仅有单维度。这是推荐的用法。
3.	dbRule: 指定库名的路由规则表达式。zebra会解析出SQL中#bid#维度和值，并将该值带入该表达式计算出最终落到的数据库的index。
4.	dbIndexes: 指定所有拆分的库的GroupDataSource的jdbcRef。zebra会将第3步计算出的index带入该表达式，得出最终落到的数据库的jdbcRef值。dbIndexes可以有以下几种等价的写法，它们的index都是从0开始按照写的顺序呢进行排列。
5.	tbRule: 指定是分表的路由规则表达式。zebra会解析出SQL中#bid#维度和值，并将该值带入该表达式计算出最终落到的数据库中表名的后缀index。表达式内置了crc32和md5两个函数，同时可以支持多值组成的单维度。
6.	tbSuffix: 是表的后缀命名规则，分everydb和alldb。everydb指任何库上的表名都相同。例如everydb:[0,9]，意味着分着的10个库上，每个库上的表名均为welife_users0到welife_users9。而alldb是指10表名在十个库上都不一样，同样的例子如果使用alldb的方式应该配成alldb:[0,99]。此时在库welife0上的表名为welife_users[0-9]，在welife1上的表名为welife_users[10-19]......在welife9上的表名为welife_users[89-99]。建议使用alldb的方式，这样清晰，方便定位。
7.	generatedPK: 表中唯一识别一行的主键字段，这里是uid。
8.	isMaster: 表明该维度是否是主维度，也就说说一个分表可以有多个维度，但只有主维度支持写，其他辅助维度只能进行读。点评内部是通过binlog的方式进行复制的方式，将主维度的数据自动的复制到辅助维度上去。

dbIndexes支持的写法：

	welife[0-4]			
	welife0,welife1,welife2,welife3,welife4
	welife0,welife[1-3],welife4
	
例如下面tbRule示例，zebra会解析SQL，得到#bid#和#uid#两个值带入表达式得出index。

	(crc32(md5(#bid# + "_" + #uid#))).toLong()%10
	
# ShardDataSource SQL支持列表

[SQL支持列表](https://github.com/dianping/zebra/blob/master/zebra-api/README_SHARD_SQL.md)

假设有一张表：User，主维度是：Id，辅维度是：CityId

## INSERT

**插入时必须指定主键，暂不支持自动生成主键**

*	INSERT INTO User WHERE (Id, Name, CityId) VALUES (1, 'test', 2)

## Batch INSERT

仅支持Values后的多行记录是确定到某个库某张表上的。暂不支持Values后的多条记录落到不同的库和表。

## UPDATE & DELETE

**更新操作最好指定主维度，这样性能高**

*	UPDATE User SET Name = 'new name' WHERE Id = 1
*	DELETE FROM User WHERE Id = 1

**这里虽然有主维度，但是这类操作会操作所有的主维度表，性能较差**

*	UPDATE User SET Name = 'new name' WHERE Id > 1
*	UPDATE User SET Name = 'new name' WHERE Id <> 1
*	DELETE FROM User WHERE Id > 1
*	DELETE FROM User WHERE Id <> 1

**使用辅维度，但是也会操作所有主维度，性能较差**

*	UPDATE User SET Name = 'new name' WHERE CityId = 2
*	DELETE FROM User WHERE CityId = 2

**使用其他字段或者不加条件，都会操作所有主维度**

*	UPDATE User SET Name = 'new name' WHERE Name = 'test'
*	DELETE FROM User WHERE Name = 'test'
	
## SELECT

**指定主维度或者辅维度，都可以快速查询到数据，性能高**

*	SELECT Id, Name, CityId WHERE Id = 1
*	SELECT Id, Name, CityId WHERE CityId = 1
*	SELECT Id, Name, CityId WHERE Id = 2 AND CityId = 1
*	SELECT Id, Name, CityId WHERE Id IN (1,2,3,4)
*	SELECT Id, Name, CityId WHERE Id = 1 OR Id = 2

**不指定主维度或者辅维度，会查询所有主维度的表，性能差**

*	SELECT Id, Name, CityId FROM User
*	SELECT Id, Name, CityId FROM User WHERE Id > 1
*	SELECT Id, Name, CityId FROM User WHERE Id <> 1
*	SELECT Id, Name, CityId FROM User WHERE Name LIKE '%test%'

**LIMIT, OFFSET, ORDER BY 都支持，性能差**

*	SELECT Id, Name, CityId FROM User ORDER BY CityId LIMIT 1 OFFSET 1

**支持子查询，但无法识别子查询中的分区字段，性能差**

*	SELECT Id, Name, CityId FROM User WHERE Id IN (SELECT Id FROM User WHERE CityId = 1)

**支持 GROUP BY, COUNT, MAX, MIN 都支持，AVG 不支持，性能差，另外必须加字段别名**

*	SELECT CityId, MAX(Id) as MaxId FROM User GROUP BY CityId
*	SELECT CityId, MIN(Id) as MinId FROM User GROUP BY CityId
*	SELECT CityId, COUNT(Id) as AllId FROM User GROUP BY CityId

**IN语句支持

指定主维度在IN语句内，因为使用了并发查询，性能极佳。不建议IN语句里面有太多的数据。

*	SELECT Id, Name, CityId FROM User WHERE Id IN (1,2,3,4)
*	UPDATE User SET Name = 'new name' WHERE Id IN (1,2,3,4,5)
*	DELETE FROM User WHERE Id IN (1,2,3,4,5)