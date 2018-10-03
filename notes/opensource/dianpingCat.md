---
layout: post
notes: true
subtitle: "【开源代码】CAT"
comments: false
author: "大众点评网"
date: 2018-10-01 00:00:00

---

GitHub地址：[https://github.com/dianping/cat/](https://github.com/dianping/cat/)

Demo地址：[http://unidal.org/cat/r/home?op=view&docName=index](http://unidal.org/cat/r/home?op=view&docName=index)


*   目录
{:toc }

**CAT基于Java开发的实时应用监控平台，包括实时应用监控，业务监控。**

**CAT支持的监控消息类型包括：**

*	**Transaction**适合记录跨越系统边界的程序访问行为,比如远程调用，数据库调用，也适合执行时间较长的业务逻辑监控，Transaction用来记录一段代码的执行时间和次数。
*	**Event**用来记录一件事发生的次数，比如记录系统异常，它和transaction相比缺少了时间的统计，开销比transaction要小。
*	**Heartbeat**表示程序内定期产生的统计信息, 如CPU%, MEM%, 连接池状态, 系统负载等。
*	**Metric**用于记录业务指标、指标可能包含对一个指标记录次数、记录平均值、记录总和，业务指标最低统计粒度为1分钟。

# 消息树

CAT监控系统将每次URL、Service的请求内部执行情况都封装为一个完整的消息树、消息树可能包括Transaction、Event、Heartbeat、Metric和Trace信息。

## 完整的消息树

![](/img/notes/opensource/dianpingCat/total_message_tree.png)

## 可视化消息树

![](/img/notes/opensource/dianpingCat/view_message_tree.png)

## 分布式消息树（一台机器调用另外一台机器）

![](/img/notes/opensource/dianpingCat/distribute_message_tree.png)

# Quick Start

## CAT安装环境

*	Linux 2.6以及之上（2.6内核才可以支持epoll），线上服务端部署请使用Linux环境，Mac以及Windows环境可以作为开发环境，美团点评内部CentOS 6.5
*	Java 6，7，8，服务端推荐是用jdk7的版本，客户端jdk6、7、8都支持
*	Maven 3.3.3
*	MySQL 5.6，5.7，更高版本MySQL都不建议使用，不清楚兼容性
*	J2EE容器建议使用tomcat，建议版本7.0.70，高版本tomcat默认了get字符串限制，需要修改一些配置才可以生效，不然提交配置可能失败。
*	Hadoop环境可选，一般建议规模较小的公司直接使用磁盘模式，可以申请CAT服务端，500GB磁盘或者更大磁盘，这个磁盘挂载在/data/目录上

## 安装CAT集群大致步骤

1.	初始化Mysql数据库，一套CAT集群部署一个数据库，初始化脚本在script下的Cat.sql
2.	准备三台CAT服务器，IP比如为10.1.1.1，10.1.1.2，10.1.1.3。
3.	初始化/data/目录，配置几个配置文件/data/appdatas/cat/*.xml 几个配置文件，具体下面有详细说明
4.	打包cat.war 放入tomcat容器
5.	修改一个路由配置，重启tomcat

## 1、tomcat启动参数调整，修改catalina.sh文件（服务端）

需要每台CAT集群10.1.1.1，10.1.1.2，10.1.1.3都进行部署

建议使用cms gc策略

建议cat的使用堆大小至少10G以上，开发环境启动2G堆启动即可

	CATALINA_OPTS="$CATALINA_OPTS -server -Djava.awt.headless=true -Xms25G -Xmx25G -XX:PermSize=256m -XX:MaxPermSize=256m -XX:NewSize=10144m -XX:MaxNewSize=10144m -XX:SurvivorRatio=10 -XX:+UseParNewGC -XX:ParallelGCThreads=4 -XX:MaxTenuringThreshold=13 -XX:+UseConcMarkSweepGC -XX:+DisableExplicitGC -XX:+UseCMSInitiatingOccupancyOnly -XX:+ScavengeBeforeFullGC -XX:+UseCMSCompactAtFullCollection -XX:+CMSParallelRemarkEnabled -XX:CMSFullGCsBeforeCompaction=9 -XX:CMSInitiatingOccupancyFraction=60 -XX:+CMSClassUnloadingEnabled -XX:SoftRefLRUPolicyMSPerMB=0 -XX:-ReduceInitialCardMarks -XX:+CMSPermGenSweepingEnabled -XX:CMSInitiatingPermOccupancyFraction=70 -XX:+ExplicitGCInvokesConcurrent -Djava.nio.channels.spi.SelectorProvider=sun.nio.ch.EPollSelectorProvider -Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager -Djava.util.logging.config.file="%CATALINA_HOME%\conf\logging.properties" -XX:+PrintGCDetails -XX:+PrintGCTimeStamps -XX:+PrintGCApplicationConcurrentTime -XX:+PrintHeapAtGC -Xloggc:/data/applogs/heap_trace.txt -XX:-HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/data/applogs/HeapDumpOnOutOfMemoryError -Djava.util.Arrays.useLegacyMergeSort=true"

修改中文乱码tomcat conf目录下server.xml

	<Connector port="8080" protocol="HTTP/1.1"
           URIEncoding="utf-8"    connectionTimeout="20000"
               redirectPort="8443" />  增加  URIEncoding="utf-8"
			   
## 2、程序对于/data/目录具体读写权限（包括客户端&服务端）

*	注意无论是CAT客户端和服务端都要求/data/目录能进行读写操作，如果/data/目录不能写，建议使用linux的软链接链接到一个固定可写的目录，软链接的基本命令请自行搜索google
*	此目录会存一些CAT必要的配置文件，运行时候的缓存文件，建议不要修改，如果想改，请自行研究好源码里面的东西，在酌情修改，此目录不支持进行配置化
*	mkdir /data
*	chmod 777 /data/ -R
*	如果是Windows开发环境则是对程序运行盘下的/data/appdatas/cat和/data/applogs/cat有读写权限,如果cat服务运行在e盘的tomcat中，则需要对e:/data/appdatas/cat和e:/data/applogs/cat有读写权限
*	如果windows实在不知道哪个盘，就所有盘都建好，最后看哪个盘多文件，就知道哪个了

## 3、配置/data/appdatas/cat/client.xml（包括客户端&服务端）

*	此配置文件的作用是所有的客户端都需要一个地址指向CAT的服务端，比如CAT服务端有三个IP，10.1.1.1，10.1.1.2，10.1.1.3，2280是默认的CAT服务端接受数据的端口，不允许修改，http-port是Tomcat启动的端口，默认是8080，建议使用默认端口。
*	此文件可以通过运维统一进行部署和维护，比如使用puppert等运维工具。
*	不同环境这份文件不一样，比如区分prod环境以及test环境，在美团点评内部一共是2套环境的CAT，一份是生产环境，一份是测试环境

配置如下：

	<?xml version="1.0" encoding="utf-8"?>
	<config mode="client">
		<servers>
			<server ip="10.1.1.1" port="2280" http-port="8080"/>
			<server ip="10.1.1.2" port="2280" http-port="8080"/>
			<server ip="10.1.1.3" port="2280" http-port="8080"/>
		</servers>
	</config>

## 4、安装CAT的数据库

*	数据库的脚本文件 script/Cat.sql
*	MySQL的一个系统参数：max_allowed_packet，其默认值为1048576(1M)，修改为1000M，修改完需要重启mysql
*	注意：一套独立的CAT集群只需要一个数据库（之前碰到过个别同学在每台cat的服务端节点都安装了一个数据库）

## 5、配置/data/appdatas/cat/datasources.xml【服务端配置】

**需要每台CAT集群10.1.1.1，10.1.1.2，10.1.1.3都进行部署**

注意：此xml仅仅为模板，请根据自己实际的情况替换jdbc.url,jdbc.user,jdbc.password的实际值。 app数据库和cat数据配置为一样，app库不起作用，为了运行时候代码不报错。

	<?xml version="1.0" encoding="utf-8"?>

	<data-sources>
		<data-source id="cat">
			<maximum-pool-size>3</maximum-pool-size>
			<connection-timeout>1s</connection-timeout>
			<idle-timeout>10m</idle-timeout>
			<statement-cache-size>1000</statement-cache-size>
			<properties>
				<driver>com.mysql.jdbc.Driver</driver>
				<url><![CDATA[${jdbc.url}]]></url>
				<user>${jdbc.user}</user>
				<password>${jdbc.password}</password>
				<connectionProperties><![CDATA[useUnicode=true&characterEncoding=UTF-8&autoReconnect=true&socketTimeout=120000]]></connectionProperties>
			</properties>
		</data-source>
		<data-source id="app">
			<maximum-pool-size>3</maximum-pool-size>
			<connection-timeout>1s</connection-timeout>
			<idle-timeout>10m</idle-timeout>
			<statement-cache-size>1000</statement-cache-size>
			<properties>
				<driver>com.mysql.jdbc.Driver</driver>
				<url><![CDATA[${jdbc.url}]]></url>
				<user>${jdbc.user}</user>
				<password>${jdbc.password}</password>
				<connectionProperties><![CDATA[useUnicode=true&characterEncoding=UTF-8&autoReconnect=true&socketTimeout=120000]]></connectionProperties>
			</properties>
		</data-source>
	</data-sources>
	
## 6、配置/data/appdatas/cat/server.xml【服务端】

**需要每台CAT集群10.1.1.1，10.1.1.2，10.1.1.3都进行部署**

CAT节点一共有四个职责：

1.	控制台 - 提供给业务人员进行数据查看【默认所有的cat节点都可以作为控制台，不可配置】
2.	消费机 - 实时接收业务数据，实时处理，提供实时分析报表【默认所有的cat节点都可以作为消费机，不可配置】
3.	告警端 - 启动告警线程，进行规则匹配，发送告警（目前仅支持单点部署）【可以配置】
4.	任务机 - 做一些离线的任务，合并天、周、月等报表 【可以配置】

线上做多集群部署，比如说10.1.1.1，10.1.1.2，10.1.1.3这三台机器

1.	建议选取一台10.1.1.1 负责角色有控制台、告警端、任务机，建议配置域名访问CAT，就配置一台机器10.1.1.1一台机器挂在域名下面
2.	10.1.1.2，10.1.1.3 负责消费机处理，这样能做到有效隔离，任务机、告警等问题不影响实时数据处理

默认script下的server.xml为

	<?xml version="1.0" encoding="utf-8"?>
	<config local-mode="false" hdfs-machine="false" job-machine="true" alert-machine="false">
		<storage  local-base-dir="/data/appdatas/cat/bucket/" max-hdfs-storage-time="15" local-report-storage-time="7" local-logivew-storage-time="7">
			<hdfs id="logview" max-size="128M" server-uri="hdfs://10.1.77.86/user/cat" base-dir="logview"/>
			<hdfs id="dump" max-size="128M" server-uri="hdfs://10.1.77.86/user/cat" base-dir="dump"/>
			<hdfs id="remote" max-size="128M" server-uri="hdfs://10.1.77.86/user/cat" base-dir="remote"/>
		</storage>
		<console default-domain="Cat" show-cat-domain="true">
			<remote-servers>127.0.0.1:8080</remote-servers>
		</console>
	</config>
	
配置说明：

*	local-mode : 建议在开发环境以及生产环境时，都设置为false
*	hdfs-machine : 定义是否启用HDFS存储方式，默认为 false
*	job-machine : 定义当前服务是否为报告工作机（开启生成汇总报告和统计报告的任务，只需要一台服务机开启此功能），默认为 false
*	alert-machine : 定义当前服务是否为报警机（开启各类报警监听，只需要一台服务机开启此功能），默认为 false；
*	storage : 定义数据存储配置信息
*	local-report-storage-time : 定义本地报告文件存放时长，单位为（天）
*	local-logivew-storage-time : 定义本地日志文件存放时长，单位为（天）
*	local-base-dir : 定义本地数据存储目录，建议直接使用/data/appdatas/cat/bucket目录
*	hdfs : 定义HDFS配置信息
*	server-uri : 定义HDFS服务地址
*	console : 定义服务控制台信息
*	remote-servers : 定义HTTP服务列表，（远程监听端同步更新服务端信息即取此值）
*	ldap : 定义LDAP配置信息（这个可以忽略）
*	ldapUrl : 定义LDAP服务地址（这个可以忽略）

按照如上的说明，10.1.1.1 机器/data/appdatas/cat/serverm.xml配置，注意hdfs配置就随便下了一个，请忽略

	<?xml version="1.0" encoding="utf-8"?>
	<config local-mode="false" hdfs-machine="false" job-machine="true" alert-machine="true">
		<storage  local-base-dir="/data/appdatas/cat/bucket/" max-hdfs-storage-time="15" local-report-storage-time="7" local-logivew-storage-time="7">
		<hdfs id="logview" max-size="128M" server-uri="hdfs://10.1.77.86/user/cat" base-dir="logview"/>
			<hdfs id="dump" max-size="128M" server-uri="hdfs://10.1.77.86/user/cat" base-dir="dump"/>
			<hdfs id="remote" max-size="128M" server-uri="hdfs://10.1.77.86/user/cat" base-dir="remote"/>
		</storage>
		<console default-domain="Cat" show-cat-domain="true">
			<remote-servers>10.1.1.1:8080,10.1.1.2:8080,10.1.1.3:8080</remote-servers>
		</console>
	</config>
	
10.1.1.2，10.1.1.3 机器/data/appdatas/cat/serverm.xml配置如下，仅仅job-machine&alert-machine修改为false

	<?xml version="1.0" encoding="utf-8"?>
	<config local-mode="false" hdfs-machine="false" job-machine="false" alert-machine="false">
		<storage  local-base-dir="/data/appdatas/cat/bucket/" max-hdfs-storage-time="15" local-report-storage-time="7" local-logivew-storage-time="7">
		<hdfs id="logview" max-size="128M" server-uri="hdfs://10.1.77.86/user/cat" base-dir="logview"/>
			<hdfs id="dump" max-size="128M" server-uri="hdfs://10.1.77.86/user/cat" base-dir="dump"/>
			<hdfs id="remote" max-size="128M" server-uri="hdfs://10.1.77.86/user/cat" base-dir="remote"/>
		</storage>
		<console default-domain="Cat" show-cat-domain="true">
			<remote-servers>10.1.1.1:8080,10.1.1.2:8080,10.1.1.3:8080</remote-servers>
		</console>
	</config>
	
## 7、war打包

1.	在cat的源码目录，执行mvn clean install -DskipTests
2.	如果发现cat的war打包不通过，CAT所需要依赖jar都部署在 http://unidal.org/nexus/
3.	可以配置这个公有云的仓库地址到本地的settings路径，理论上不需要配置即可，可以参考cat的pom.xml配置
4.	如果自行打包仍然问题，请使用下面链接进行下载 http://unidal.org/nexus/service/local/repositories/releases/content/com/dianping/cat/cat-home/2.0.0/cat-home-2.0.0.war
5.	官方的cat的master版本，重命名为cat.war进行部署，注意此war是用jdk8，服务端请使用jdk8版本

## 8、war部署

1.	将cat.war部署到10.1.1.1的tomcat的webapps下，启动tomcat，注意webapps下只允许放一个war，仅仅为cat.war
2.	如果发现重启报错，里面有NPE等特殊情况，可以检查当前java进程，ps aux | grep java，可能存在之前的tomcat的进程没有关闭，又新启动了一个，导致出问题，建议kill -9 干掉所有的java进程
3.	打开控制台的URL，http://10.1.1.1:8080/cat/s/config?op=routerConfigUpdate
4.	注意10.1.1.1这个IP需要替换为自己实际的IP链接，修改路由配置只能修改一次即可
5.	修改路由配置为如下，当为如下配置时，10.1.1.1 正常不起消费数据的作用，仅当10.1.1.2以及10.1.1.3都挂掉才会进行实时流量消费

	<?xml version="1.0" encoding="utf-8"?>
	<router-config backup-server="10.1.1.1" backup-server-port="2280">
	   <default-server id="10.1.1.2" weight="1.0" port="2280" enable="true"/>
	   <default-server id="10.1.1.3" weight="1.0" port="2280" enable="true"/>
	</router-config>

6.	重启10.1.1.1的机器的tomcat
7.	将cat.war部署到10.1.1.2，10.1.1.3这两台机器中，启动tomcat
8.	cat集群部署完毕，如果有问题，欢迎在微信群咨询，如果文档有误差，欢迎指正以及提交pullrequest

## 9、开发环境CAT的部署

1.	请按照如上部署/data/环境目录，数据库配置client.xml ,datasources.xml,server.xml这三个配置文件，注意server.xml里面的节点角色，job-machine&alert-machine都可以配置为true
2.	在cat目录中执行 mvn eclipse:eclipse，此步骤会生成一些代码文件，直接导入到工程会发现找不到类
3.	将源码以普通项目到入eclipse中，注意不要以maven项目导入工程
4.	运行com.dianping.cat.TestServer 这个类，即可启动cat服务器
5.	这里和集群版本唯一区别就是服务端部署单节点，client.xml server.xml以及路由地址配置为单台即可

## 10、客户端的集成

1.	参考 http://unidal.org/cat/r/home?op=view&docName=integration
2.	一些埋点的DEMO可以参考cat-home下的testcase，TestSendMessage.java,注意所有埋点cat不支持中文，cat后端存储会过滤掉所有的中文，请使用英文以及简单的符号比如. 来做埋点
3.	一些默认框架埋点的可以参考，cat目录下框架埋点方案集成的文件夹
4.	jar包的集成如下方案
5.	将cat的客户端以及client的依赖包部署到公司私有仓库，检查cat的依赖包可以使用mvn dependency:tree命令
6.	如果公司没有私有仓库，可以请使用cat提供的公有云仓库，http://unidal.org/nexus/
7.	项目的pom可以配置参考cat资源文件的pom.xml文件

	<repositories>
		<repository>
			<id>central</id>
			<name>Maven2 Central Repository</name>
			<layout>default</layout>
			<url>http://repo1.maven.org/maven2</url>
		</repository>
		<repository>
			<id>unidal.releases</id>
			<url>http://unidal.org/nexus/content/repositories/releases/</url>
		</repository>
	</repositories>
	<pluginRepositories>
		<pluginRepository>
			<id>central</id>
			<url>http://repo1.maven.org/maven2</url>
		</pluginRepository>
		<pluginRepository>
			<id>unidal.releases</id>
			<url>http://unidal.org/nexus/content/repositories/releases/</url>
		</pluginRepository>
	</pluginRepositories>
	
# 用户文档

## 实时导航介绍

![](/img/notes/opensource/dianpingCat/real.png)

## 历史导航介绍

点击导航中间的“History Mode”便可进入相应的历史报表界面。

![](/img/notes/opensource/dianpingCat/history.png)

历史报表目前分为三类：日报表、周报表、月报表。当首次选择day、week、month时，默认为当前最近的一天、一周、一个月。

注：页面暂时不支持特定时间区间的报表查询，如果想查询特定连续时间的统计情况，可以加入URL参数输入条件，参数为 &startDate=20120712&endDate=20120715，它表示查询7月12号0点-7月15号0点这段期间的统计数据。

# 告警文档

## 1. 告警模块介绍

告警模块按照规则对收集的信息进行监控，并在规则被触发时通知相应的联系人。

能否有效的使用告警模块直接影响着监控的质量。通过对监控规则、告警策略、默认联系人等元素进行合理的配置，告警模块能够更快、更准确、更灵活的发现线上故障，并更有效的通知对应联系人。

## 2. 监控规则配置

合理、灵活的监控规则可以帮助更快、更精确的发现业务线上故障。目前Cat的监控规则有五个要素，请按照以下五点要素制定规则：

1.	时间段。同一项业务指标在每天不同的时段可能有不同的趋势。设定该项，可让Cat在每天不同的时间段执行不同的监控规则。
2.	规则组合。在一个时间段中，可能指标触发了多个监控规则中的一个规则就要发出警报，也有可能指标要同时触发了多个监控规则才需要发出警报。这种关系好比电路图中的并联和串联。规则的组合合理有助于提高监控的准确度。
3.	监控规则类型。通过以下八种类型对指标进行监控：下降百分比、下降数值、上升百分比、上升数值、最大值、最小值、波动百分比最大值、波动百分比最小值。
4.	持续时间。设定时间后（单位为分钟），当指标在设定的时间长度内连续触发了监控规则，才会发出警报。
5.	规则与被监控指标的匹配。监控规则可以按照名称、正则表达式与监控的对象（指标）进行匹配。

监控规则模型如下图所示：

![](/img/notes/opensource/dianpingCat/alert_config.png)

具体解释如下：

1.	一个rule元素为规则的基本单位，由唯一的id标示
2.	rule元素由两个部分组成：监控对象与监控规则
	1.	监控对象：由metric－item元素匹配，与图中的匹配对象相对应。匹配对象是两级的，每一级都支持正则匹配
	2.	.监控条件配置：由config元素组成，与图中监控规则相对应。每个config代表一个时间段的规则，由starttime和endtime两个属性确定。时间的配置格式为：“hh:mm”，请注意hh为24小时制。config元素由多个监控条件组成，条件由condition元素表示。一个config下的多个condition为并联关系，当一个condition被触发，conditon所在的整个rule就被触发。condition元素中的minute属性表示该条件的持续时间。设定时间单位为分钟。当指标在设定的时间长度内连续触发了该条规则，才会触发该condition。condition由subcondition组成。subcondition与图中的子条件相对应。一个condition下的多个subcondition为串联关系，只有当一个condition下的全部subcondition被触发，该condition才被触发。subcondition有八种类型，由type属性指定。subcondition的内容为对应的阈值，请注意阈值只能由数字组成，当阈值表达百分比时，不能在最后加上百分号。
	
八种类型如下：

| 类型 | 说明 |
| ---- | ---- |
| DescPer | 下降百分比 |
| DescVal | 下降数值 |
| AscPer | 上升百分比 |
| AscVal | 上升数值 |
| MaxVal | 最大值 |
| MinVal | 最小值 |
| FluAscPer | 波动百分比最大值。即当前分钟值比监控周期内其它分钟值的增加百分比都大于设定的百分比时触发警报 |
| FluDescPer | 波动百分比最小值。即当前分钟值比监控周期内其它分钟值的减少百分比都大于设定的百分比时触发警报 |
| SumMaxVal | 总和最大值，请与告警分钟总和考虑 |
| SumMinVal | 综合最小值，请与告警分钟总和考虑 |

点击"如何使用?"按钮，将会出现信息介绍设置规则的流程

## 3. 告警策略

为了将告警信息更有效的发送给对应联系人，请考虑以下五个要素制定告警策略：

1.	告警类型。Cat将告警分为六种类型：业务告警(项目指标的监控)、网络告警(网络设备监控)、系统告警(服务器状态监控)、异常告警(Exception数量监控)、第三方监控(对给定的网址，根据HTTP请求的返回码监控)、前端监控。由于告警策略是按照类型划分的，制定告警策略前首先请确定目前采用的是哪种类型的监控。
2.	告警级别。告警级别即为该告警的优先级。不同级别的告警在通知渠道、暂停告警时间上可以有所差别。对告警进行合理的分级能够帮助我们将更多的精力放在更重要的问题上。
3.	告警渠道。目前有三种告警渠道：邮件、微信、短信。
4.	暂停告警时间。设定暂停告警时间(suspendMinute)后，某一指标在一次告警之后的指定时间段内不会再次发送告警信息。
5.	.恢复通知。设定恢复通知时间(recoverMinute)后，当一个指标在某一分钟告警并且在以后的指定时间段内没有再次告警时，Cat会发出恢复通知，表明该指标在这个时间段的状态是正常的。默认的恢复通知时间段为一分钟。

告警策略模型如下图所示：

![](/img/notes/opensource/dianpingCat/alert_policy.png)

具体解释如下：

1.	alert-policy元素对应着Cat上的全部告警策略信息。每个type元素对应着一种告警类型，由id可以得知type与告警类型的对应关系
2.	type元素下每一个group元素对应着一个项目或是一个产品线的告警策略。对于异常监控以及第三方监控，此处的group请填写项目名；其它类型请填写产品线名；当group元素的id为default时，该group元素即为该告警类型的默认告警策略。当没有其它group命中时，会采用默认告警策略。
3.	group元素下的level元素对应着告警级别。level元素与监控规则中的alertType属性是对应的，请两者配合使用。level元素有两个属性：

	*	send属性。该属性对应着发送渠道，发送渠道之间用逗号分割
	*	suspendMinute属性。该属性为暂停告警时间，单位为分钟
	
配置方法

1.	点击导航栏Config－－监控告警配置－－告警类型设置
2.	编辑文本框的内容，点击提交
3.	当出现"操作成功"提示时表明策略已经生效

## 4. 默认联系人设置

此处仅建议Cat开发者使用。主要有以下两个功能：

1.	控制某一个类型的所有告警信息是否发送
2.	添加默认通知人。该通知人会收到某类型的所有告警

默认发送人模型如下图所示：

![](/img/notes/opensource/dianpingCat/default_receiver.png)

配置方法：

1.	点击导航栏Config－－监控告警配置－－默认告警配置
2.	编辑文本框的内容，点击提交
3.	当出现"操作成功"提示时表明规则已经生效

# 集成文档

## 1. Web.xml中新增filter

注：如果项目是对外不提供URL访问，比如GroupService，仅仅提供Pigeon服务，则不需要。

**Filter放在url-rewrite-filter 之后的第一个，如果不是会导致URL的个数无限多，比如search/1/2,search/2/3等等，无法监控，后端存储压力也变大。**

	<filter>
        <filter-name>cat-filter</filter-name>
        <filter-class>com.dianping.cat.servlet.CatFilter</filter-class>
    </filter>
    <filter-mapping>
        <filter-name>cat-filter</filter-name>
        <url-pattern>/*</url-pattern>
        <dispatcher>REQUEST</dispatcher>
        <dispatcher>FORWARD</dispatcher>
    </filter-mapping>
	
struts会吃掉URL中的ERROR信息，请在配置中加

	<constant name="struts.handle.exception" value="false" /> 
	
解决URL中很多重复的问题，比如restfull的url

	CAT 提供了自定义的URL的name功能，只要在HttpServletRequest的设置一个Attribute，
	在业务运行代码中加入如下code可以自定义URL下name，这样可以进行自动聚合。
	HttpServletRequest req ;
	req.setAttribute("cat-page-uri", "myPageName");
	
## 2. Pom.xml中更新jar包(点评内部公共组件，外部公司可以忽略)

	<dependency>
		<groupId>com.dianping.cat</groupId>
		<artifactId>cat-core</artifactId>   
		<version>1.2.7</version>
	</dependency>
   
## 3. 配置domain (cat-core 1.1.3之后版本，优先读取A配置)

A) 在资源文件中新建app.properties文件

在resources资源文件META-INF下，注意是src/main/resources/META-INF/文件夹， 而不是webapps下的那个META-INF,添加app.properties，加上domain配置，如：app.name=tuangou-web

B) 在资源文件中新建client.xml文件

在resources资源文件META-INF下，新建cat文件夹，注意是src/main/resources/META-INF/cat/client.xml文件， 而不是webapps下的那个META-INF,domain id表示项目名称此处为CMDB申请的名字，比如：

	<config mode="client">
		<domain id="tuangou-web"/>
	</config>
	
## 4. /data/appdatas/cat/目录下，新建一个client.xml文件(线上环境是OP配置)

如果系统是windows环境，则在eclipse运行的盘，比如D盘，新建/data/appdatas/cat/目录，新建client.xml文件

项目文件中srouce中的client.xml,此文件代表了这个项目我是谁,比如项目的名字Cat。

/data/appdatas/cat/client.xml,此文件有OP控制,这里的Domain名字用来做开关，如果一台机器上部署了多个应用，可以指定把一个应用的监控关闭。

	<config mode="client">
		<servers>
			<server ip="192.168.213.115" port="2280" />
		</servers>
	</config>
	
alpha、beta这个配置需要自己在此目录添加

预发以及生产环境这个配置需要通知到对应OP团队，让他们统一添加，自己上线时候做下检查即可

a、192.168.213.115:2280端口是指向测试环境的cat地址

b、配置可以加入CAT的开关，用于关闭CAT消息发送,将enabled改为false，如下表示将mobile-api这个项目关闭

	<config mode="client">
		<servers>
			<server ip="192.168.213.115" port="2280" />
		</servers>
		<domain id="mobile-api" enabled="false"/>
	</config>
	
## 5. CAT的Log4j集成 【建议所有Log都打到CAT，这样才能更快发现问题】

业务程序的所有异常都通过记录到CAT中，方便看到业务程序的问题，建议在Root节点中添加次appendar

a）在Log4j的xml中，加入Cat的Appender>

	<appender name="catAppender" class="com.dianping.cat.log4j.CatAppender"></appender>
	
b）在Root的节点中加入catAppender

	 <root>
       <level value="error" />
       <appender-ref ref="catAppender" />
     </root>
	 
c）注意有一些Log的是不继承root的，需要如下配置

	<logger name="com.dianping.api.location" additivity="false">
        <level value="INFO"/>
        <appender-ref ref="locationAppender"/>
        <appender-ref ref="catAppender"/>
    </logger>
	
# 开发者文档

## 1. CAT消息协议

CAT客户端可以向服务端发送Transaction, Event, Heartbeat三种消息. 消息的传输格式如下：

	Class Timestamp Type Name Status  Duration  Data
	
说明：

*	Timestamp：记录消息产生的时刻, 格式"yyyy-mm-dd HH:MM:SS.sss"
*	Type：大小写敏感的字符串. 常见的Transaction type有 "URL", "SQL", "Email", "Exec"等. 常见的Event type有 "Info", "Warn", "Error", 还有"Cat"用来表示Cat内部的消息
*	Name：大小写敏感的字符串. type和name的组合要满足全局唯一性. 常见的URL transaction type的name如 "ViewItem", "MakeBid", "SignIn"等. SQL transaction type的name如 "AddFeedback", "GetAccountDetailUnit4", "IncrementFeedbackAndTotalScore"等
*	Status：大小写敏感的字符串. 0表示成功, 非零表示失败。建议不要使用太长的字符串. Transaction start没有status字段
*	Duration：精确到0.1毫秒。表示transaction start和transaction end之间的时间长度。仅出现在Transaction end或者Atomic Transaction。Event和Heartbeat没有duration字段
*	Data：建议使用以&字符分割的name=value对组成的字符串列表. Transaction start没有data字段

## 2. Transaction

1.	transaction适合记录跨越系统边界的程序访问行为，比如远程调用，数据库调用，也适合执行时间较长的业务逻辑监控
2.	某些运行期单元要花费一定时间完成工作, 内部需要其他处理逻辑协助, 我们定义为Transaction
3.	Transaction可以嵌套(如http请求过程中嵌套了sql处理)
4.	大部分的Transaction可能会失败, 因此需要一个结果状态码。
5.	如果Transaction开始和结束之间没有其他消息产生, 那它就是Atomic Transaction(合并了起始标记)

### Transaction API

	com.dianping.cat.message.MessageProducer:
	Transaction newTransaction(String type, String name);
	com.dianping.cat.message.Transaction:
	void addData(String keyValuePairs);
	void addData(String key, Object value);
	void setStatus(String status);
	void complete();
	
### 代码示例

	Transaction t = Cat.getProducer().newTransaction("your transaction type", "your transaction name");
	try {
		yourBusinessOperation();
		Cat.getProducer().logEvent("your event type", "your event name", Event.SUCCESS, "keyValuePairs")
		t.setStatus(Transaction.SUCCESS);
	} catch (Exception e) {
		Cat.getProducer().logError(e);//用log4j记录系统异常，以便在Logview中看到此信息
		t.setStatus(e);
		throw e; 
			(CAT所有的API都可以单独使用，也可以组合使用，比如Transaction中嵌套Event或者Metric。)
			(注意如果这里希望异常继续向上抛，需要继续向上抛出，往往需要抛出异常，让上层应用知道。)
			(如果认为这个异常在这边可以被吃掉，则不需要在抛出异常。)
	} finally {
		t.complete();
	}
	
## 3. Event

Event用来记录次数，表名单位时间内消息发生次数，比如记录系统异常，它和transaction相比缺少了时间的统计，开销比transaction要小
	
## 4. Metric

Metric一共有三个API，分别用来记录次数、平均、总和，统一粒度为一分钟
	
1.	logMetricForCount用于记录一个指标值出现的次数
2.	logMetricForDuration用于记录一个指标出现的平均值
3.	logMetricForSum用于记录一个指标出现的总和
4.	修改指标一些图形描述信息，请到Config进行配置

## 5. Heartbeat

**这个是系统CAT客户端使用，应用程序不使用此API。**

Heartbeta表示程序内定期产生的统计信息, 如CPU%, MEM%, 连接池状态, 系统负载等。

## 6. 一份埋点的样例

*	Transaction用来记录一段程序响应时间
*	Event用来记录一行code的执行次数
*	Metric用来记录一个业务指标

这些指标都是独立的，可以单独使用，主要看业务场景。

下面的埋点代码里面表示需要记录一个页面的响应时间，并且记录一个代码执行次数，以及记录两个业务指标,所有用了一个Transaction，一个Event，两个Metric

Transaction的埋点一定要complete，切记放在finally里面。

![](/img/notes/opensource/dianpingCat/cat_example.png)

# 设计文档

## 监控领域建模

![](/img/notes/opensource/dianpingCat/domain_model.png)

## 客户端设计

![](/img/notes/opensource/dianpingCat/client.png)

## 服务端设计

![](/img/notes/opensource/dianpingCat/server.png)

## 应用程序部署

![](/img/notes/opensource/dianpingCat/deploy.png)
