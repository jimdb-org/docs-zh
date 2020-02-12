部署
=============================

Master部署
--------------------------

编译代码：
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
切换代码库master分支，进入master-server的cmd目录，执行编译脚本build.sh，生成可执行文件master-server：

.. code-block:: bash

	cd jimdb/master-server/cmd
	./build.sh


启动服务：
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
指定配置文件作为参数，运行master-server在后台：

.. code-block:: bash

	setsid ./master-server -config ms.conf &


配置说明：
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
* node-id: 此master-server在raft复制组中的id，即cluster.peer段中的ip相同的id

::

	例如：
	node-id = 1

* data-dir: master保存集群元数据的存储目录

::

	例如：
	data-dir = “/export/master-server/data”

* cluster段的cluster-id配置：指定此master-server所服务的集群id

:: 

	例如：
	[cluster]
	cluster-id = 10

* cluster段的peer配置：master-server为提供高可用元数据服务能力，使用raft实现元数据复制；peer配置描述的是所有raft复制组成员的信息，peer配置是一个数组, 包括:

  + id（此master在复制组中的编号）

  + host（master ip 地址）

  + http-port（master提供的http服务端口，主要向web管理端服务）

  + rpc-port（master提供的rpc端口，主要向网关服务）

  + raft-ports（master使用的raft心跳端口和复制端口）

::

	例如：

	配置1副本的master-server：

	[[cluster.peer]]

	id = 1 # master-server节点复制组id

	host = “192.168.0.1” # master-server IP地址

	http-port = 8080 # http端口8080

	rpc-port = 8081 # rpc端口8081

	raft-ports = [8082,8083] # raft心跳端口8082，复制端口8083


	再例如:

	如果配置3副本的master-server：

	[[cluster.peer]]

	id = 1 # master-server节点复制组id 1

	host = “192.168.0.1” # master-server IP地址 192.168.0.1

	http-port = 8080

	rpc-port = 8081

	raft-ports = [8082,8083]

	[[cluster.peer]]

	id = 2 # master-server节点复制组id 2

	host = “192.168.0.2” # master-server IP地址192.168.0.2

	http-port = 8080

	rpc-port = 8081

	raft-ports = [8082,8083]

	[[cluster.peer]]

	id = 3 # master-server节点复制组id 3

	host = “192.168.0.3” # master-server IP地址192.168.0.3

	http-port = 8080

	rpc-port = 8081

	raft-ports = [8082,8083]


* log段的配置：指定日志目录，日志文件名前缀和日志级别

::

	例如：

	[log]

	dir = “/export/master-server/log”

	module = “master”

	level = “info” # 日志级别可以是debug info warn error


* replication段的配置：指定创建表时分片的副本数量

::

	例如：

	[replication]

	max-replicas = 1 # 创建1副本range。如果为3则为创建3副本分片


DataServer部署
------------------------

编译代码：
^^^^^^^^^^^^^^^^^^
切换代码库master分支，进入data-server目录，执行编译脚本build.sh，在build目录中生成可执行文件data-server：

启动服务：

.. code-block:: bash

	ulimit -c unlimited
	./data-server ds.conf start


配置说明：
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
* base_path：data-server可执行文件所在目录

::

	例如：

	base_path = /export/data-server/ # 注：在启动命令中，指定配置文件路径是相对于此base_path的


* rocksdb段的配置：指定磁盘存储路径。注：目前mass tree内存版本不是用此配置

::

	例如：

	[rocksdb]

	path = /export/data-server/data/db

* heartbeart段配置：指定此ds的元数据服务master地址和心跳频率：

::

	例如，如果master只有1个副本：

	master_num= 1 # 指定master server服务的副本数量

	master_host= “192.168.0.1:8081” # 指定master server服务的rpc地址

	再例如，如果master server有3个副本，则要依依列出master_host:

	master_num= 3

	master_host= “192.168.0.1:8081”

	master_host= “192.168.0.2:8081”

	master_host= “192.168.0.3:8081”

	node_heartbeat_interval = 10 # data-server node心跳时间间隔

	range_heartbeat_interval= 10 # da ta-server range 心跳时间间隔

* log段配置：指定日志路径和级别

:: 

	log_path = /export/data-server/log

	log_level = info # 可以是debug info warn error

* worker段配置：指定io工作线程服务的端口和线程数量

::

	例如：

	[worker]

	port = 9090 # 工作线程服务的rpc端口，比如sql请求会发到此端口

* manager段配置：指定管理线程服务的端口

::

	例如：

	[manger]

	port.= 9091 # 管理线程服务的rpc端口，比如创建分片请求会发到此端口

* range段配置：指定分片分裂阈值

::

	例如：

	[range]

	check_size = 128MB # 触发range分裂检查的阈值，即大于此阈值后开始检测

	split_size = 256MB # 指定分裂range的大小，通常为max_size的一半

	max_size = 512MB # 指定range分裂的阈值，即等于此阈值开始分裂

* raft段配置:指定raft使用的端口和raft 日志路径

::

	例如：

	[raft]

	port = 9092 # raft 使用的端口

	log_path = /export/data-server/data/raft # raft日志的存储路径
                                                 

Proxy部署
------------------

目录结构
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

::

	├── bin
	│   ├── jim.pid
	│   ├── nohup.out
	│   ├── start.sh
	│   └── stop.sh
	├── conf
	│   ├── jim.properties
	│   ├── log4j2.component.properties
	│   └── log4j2.xml
	└── lib
	    ├── animal-sniffer-annotations-1.14.jar
	    ├── commons-codec-1.12.jar
	    ├── commons-collections-3.2.jar
	    ├── commons-lang3-3.8.1.jar
	    ├── commons-logging-1.2.jar
	    ├── concurrentlinkedhashmap-lru-1.4.2.jar
	    ├── disruptor-3.4.2.jar
	    ├── druid-1.1.20.jar
	    ├── error_prone_annotations-2.0.18.jar
	    ├── fastjson-1.2.58.jar
	    ├── guava-23.0.jar
	    ├── httpclient-4.5.2.jar
	    ├── httpcore-4.4.4.jar
	    ├── j2objc-annotations-1.1.jar
	    ├── jim-common-1.0.0-SNAPSHOT.jar
	    ├── jim-core-1.0.0-SNAPSHOT.jar
	    ├── jim-engine-1.0.0-SNAPSHOT.jar
	    ├── jim-meta-core-1.0.0-SNAPSHOT.jar
	    ├── jim-meta-proto-1.0.0-SNAPSHOT.jar
	    ├── jim-meta-service-1.0.0-SNAPSHOT.jar
	    ├── jim-mysql-model-1.0.0-SNAPSHOT.jar
	    ├── jim-mysql-protocol-1.0.0-SNAPSHOT.jar
	    ├── jim-privilege-1.0.0-SNAPSHOT.jar
	    ├── jim-proto-1.0.0-SNAPSHOT.jar
	    ├── jim-rpc-1.0.0-SNAPSHOT.jar
	    ├── jim-server-1.0.0-SNAPSHOT.jar
	    ├── jim-sql-exec-1.0.0-SNAPSHOT.jar
	    ├── jsr305-3.0.2.jar
	    ├── log4j-api-2.11.2.jar
	    ├── log4j-core-2.11.2.jar
	    ├── log4j-slf4j-impl-2.11.2.jar
	    ├── netty-all-4.1.39.Final.jar
	    ├── reactive-streams-1.0.3.jar
	    ├── reactor-core-3.3.0.RELEASE.jar
	    ├── slf4j-api-1.7.26.jar
	    └── spotbugs-annotations-4.0.0-beta1.jar

conf配置文件
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
jim.properties

::

	opts.memory=-Xms8G -Xmx8G -Xmn3G -XX:SurvivorRatio=8 -XX:MaxDirectMemorySize=4G -XX:MetaspaceSize=64M -XX:MaxMetaspaceSize=512M -Xss256K -server -XX:+TieredCompilation -XX:CICompilerCount=3 -XX:InitialCodeCacheSize=64m -XX:ReservedCodeCacheSize=2048m -XX:CompileThreshold=1000 -XX:FreqInlineSize=2048 -XX:MaxInlineSize=512 -XX:+UseConcMarkSweepGC -XX:+UseCMSCompactAtFullCollection -XX:CMSInitiatingOccupancyFraction=70 -XX:+CMSParallelRemarkEnabled -XX:SoftRefLRUPolicyMSPerMB=0 -XX:CMSMaxAbortablePrecleanTime=100 -XX:+PrintGCDetails -Xloggc:/export/Logs/jimsql/gc.log -XX:+ExplicitGCInvokesConcurrentAndUnloadsClasses -XX:+PrintGCTimeStamps

	#JIM
	jim.outbound.threads=0
	jim.inbound.threads=0
	jim.plugin.metadata=jimMeta
	jim.plugin.sqlengine=mysqlEngine
	jim.plugin.sqlexecutor=jimExecutor
	jim.plugin.storeengine=jimStore

	jim.reactor.debug=false
	#0:DISABLED,1:SIMPLE,2:ADVANCED,3:PARANOID
	jim.netty.leak=1

	jim.aynctask.threads=32
	jim.grpc.threads=8

	#元数据http地址 master地址
	jim.meta.address=http://xx.xx.xx.xx:443
	jim.meta.interval=600000
	jim.cluster=2

	####################### Netty Server ##################################################
	#服务IP
	netty.server.host=0.0.0.0
	#服务端口
	netty.server.port=3306
	#连接请求最大队列长度，如果队列满时收到连接指示，则拒绝该连接。
	netty.server.backlog=65536
	#默认发送数据包超时时间，默认5秒
	netty.server.sendTimeout=5000
	#Selector线程
	netty.server.bossThreads=1
	#IO线程, 0=cpu num
	netty.server.ioThreads=8
	#通道最大空闲时间(毫秒)
	netty.server.maxIdle=1800000
	#socket读超时时间(毫秒)
	netty.server.soTimeout=3000
	#socket缓冲区大小
	netty.server.socketBufferSize=16384
	#使用EPOLL，只支持Linux模式
	netty.server.epoll=true
	#协议packet最大值
	netty.server.frameMaxSize=16778240
	#内存分配器
	netty.server.allocatorFactory=
	#表示是否允许重用Socket所绑定的本地地址
	netty.server.reuseAddress=true
	#关闭时候，对未发送数据包等待时间(秒)，-1,0:禁用,丢弃未发送的数据包>0，等到指定时间，如果还未发送则丢弃
	netty.server.soLinger=-1
	#启用nagle算法，为真立即发送，否则得到确认或缓冲区满发送
	netty.server.tcpNoDelay=true
	#保持活动连接，定期心跳包
	netty.server.keepAlive=true

	####################### Netty Client ##################################################
	#连接池大小
	netty.client.poolSize=32
	#IO线程数, 0=cpu num, -1=共用serverIO线程
	netty.client.ioThreads=4
	#连接超时(毫秒)
	netty.client.connTimeout=3000
	#默认发送数据包超时时间(毫秒)
	netty.client.sendTimeout=5000
	#socket读超时时间(毫秒)
	netty.client.soTimeout=3000
	#通道最大空闲时间(毫秒)
	netty.client.maxIdle=3600000
	#心跳间隔(毫秒)
	netty.client.heartbeat=10000
	#socket缓冲区大小
	netty.client.socketBufferSize=16384
	#协议packet最大值
	netty.client.frameMaxSize=16778240
	#使用EPOLL，只支持Linux模式
	netty.client.epoll=true
	#内存分配器
	netty.client.allocatorFactory=
	#关闭时候，对未发送数据包等待时间(秒)，-1,0:禁用,丢弃未发送的数据包>0，等到指定时间，如果还未发送则丢弃
	netty.client.soLinger=-1
	#启用nagle算法，为真立即发送，否则得到确认或缓冲区满发送
	netty.client.tcpNoDelay=true
	#保持活动连接，定期心跳包
	netty.client.keepAlive=true
	row.id.step.size=100000


log4j2.xml

.. code-block:: xml

	<?xml version='1.0' encoding='UTF-8' ?>
	<Configuration status="OFF">
	    <Properties>
	        <Property name="pattern">%d{yyyy-MM-dd HH:mm:ss.fff} [%level] -- %msg%n</Property>
	    </Properties>
	    <Appenders>
	        <Console name="CONSOLE" target="SYSTEM_OUT">
	            <PatternLayout>
	                <Pattern>${pattern}</Pattern>
	            </PatternLayout>
	        </Console>
	        <RollingRandomAccessFile name="ROLLFILE" immediateFlush="false" bufferSize="256"
	                                 fileName="/export/Logs/jimsql/jim-server.log"
	                                 filePattern="/export/Logs/jimsql/jim-server.log.%d{yyyy-MM-dd}.%i.gz">
	            <PatternLayout>
	                <Pattern>${pattern}</Pattern>
	            </PatternLayout>
	            <Policies>
	                <TimeBasedTriggeringPolicy modulate="true" interval="1"/>
	            </Policies>
	            <DefaultRolloverStrategy max="20">
	                <Delete basePath="/export/Logs/jimsql" maxDepth="1">
	                    <IfFileName glob="*.gz"/>
	                    <IfLastModified age="3d"/>
	                </Delete>
	            </DefaultRolloverStrategy>
	        </RollingRandomAccessFile>
	    </Appenders>
	    <Loggers>
	        <AsyncRoot level="warn" includeLocation="false">
	            <AppenderRef ref="ROLLFILE"/>
	        </AsyncRoot>
	    </Loggers>
	</Configuration>

log4j2.component.properties

::

	log4j2.asyncLoggerRingBufferSize=1048576
	log4j2.asyncLoggerWaitStrategy=Sleep


bin下停启proxy命令
^^^^^^^^^^^^^^^^^^^^

start.sh — proxy启动脚本

::

	# !/bin/sh

	BASEDIR=`dirname $0`/..
	BASEDIR=`(cd "$BASEDIR"; pwd)`

	export JAVA_HOME=/export/servers/jdk1.8.0_60
 
	# If a specific java binary isn't specified search for the standard 'java' binary
	if [ -z "$JAVACMD" ] ; then
	  if [ -n "$JAVA_HOME"  ] ; then
	    if [ -x "$JAVA_HOME/jre/sh/java" ] ; then
	      # IBM's JDK on AIX uses strange locations for the executables
	      JAVACMD="$JAVA_HOME/jre/sh/java"
	    else
	      JAVACMD="$JAVA_HOME/bin/java"
	    fi
	  else
	    JAVACMD=`which java`
	  fi
	fi

	CLASSPATH="$BASEDIR"/conf/:"$BASEDIR"/lib/*
	CONFIG_FILE="$BASEDIR/conf/jim.properties"
	echo "$CLASSPATH"

	if [ ! -x "$JAVACMD" ] ; then
	  echo "Error: JAVA_HOME is not defined correctly."
	  echo "  We cannot execute $JAVACMD"
	  exit 1
	fi


	OPTS_MEMORY=`grep -ios 'opts.memory=.*$' ${CONFIG_FILE} | tr -d '\r'`
	OPTS_MEMORY=${OPTS_MEMORY#*=}

	# DEBUG_OPTS="-Xdebug -Xnoagent -Djava.compiler=NONE -Xrunjdwp:transport=dt_socket,server=y,suspend=n,address=5006"

	nohup "$JAVACMD"\
	  $OPTS_MEMORY $DEBUG_OPTS \
	  -classpath "$CLASSPATH" \
	  -Dbasedir="$BASEDIR" \
	  -Dfile.encoding="UTF-8" \
	  io.jimdb.server.JimBootstrap &
	echo $! > jim.pid


stop.sh — proxy停止脚本

::

	# !/bin/sh
	if [ "$1" == "pid" ]
	then
	    PIDPROC=`cat ./jim.pid`
	else
	    PIDPROC=`ps -ef | grep 'io.jimdb.server.JimBootstrap' | grep -v 'grep'| awk '{print $2}'`
	fi

	if [ -z "$PIDPROC" ];then
	 echo "jim.server is not running"
	 exit 0
	fi

	echo "PIDPROC: "$PIDPROC
	for PID in $PIDPROC
	do
	if kill $PID
	   then echo "process jim.server(Pid:$PID) was force stopped at " `date`
	fi
	done
	echo stop finished.


proxy启动后
^^^^^^^^^^^^^^^^^^^^

proxy启动后进程

.. code-block:: bash 

	[root@79 bin]# ps -ef|grep jim
	root     21234 18113  0 10:10 pts/0    00:00:00 grep --color=auto jim
	root     57810     1 99 Sep30 ?        124-18:30:04 /export/servers/jdk1.8.0_60/bin/java -Xms8G -Xmx8G -Xmn3G -XX:SurvivorRatio=8 -XX:MaxDirectMemorySize=4G -XX:MetaspaceSize=64M -XX:MaxMetaspaceSize=512M -Xss256K -server -XX:+TieredCompilation -XX:CICompilerCount=3 -XX:InitialCodeCacheSize=64m -XX:ReservedCodeCacheSize=2048m -XX:CompileThreshold=1000 -XX:FreqInlineSize=2048 -XX:MaxInlineSize=512 -XX:+UseConcMarkSweepGC -XX:+UseCMSCompactAtFullCollection -XX:CMSInitiatingOccupancyFraction=70 -XX:+CMSParallelRemarkEnabled -XX:SoftRefLRUPolicyMSPerMB=0 -XX:CMSMaxAbortablePrecleanTime=100 -XX:+PrintGCDetails -Xloggc:/export/Logs/jimsql/gc.log -XX:+ExplicitGCInvokesConcurrentAndUnloadsClasses -XX:+PrintGCTimeStamps -classpath /export/App/jim-server/conf/:/export/App/jim-server/lib/* -Dbasedir=/export/App/jim-server -Dfile.encoding=UTF-8 io.jimdb.server.JimBootstrap


提示
^^^^^^^^^^^^^^^^^^^^^^^^^^

proxy一般按流量大小可以部署一到多个节点，重复上面的步骤可部署多个节点。

