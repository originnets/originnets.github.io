---
layout: post
title: cetus
categories: cetus
description: cetus
keywords: cetus
---

# Cetus #

## 简介 ##

Cetus是由C语言开发的关系型数据库MySQL的中间件，主要提供了一个全面的数据库访问代理功能。Cetus连接方式与MySQL基本兼容，应用程序几乎不用修改即可通过Cetus访问数据库，实现了数据库层的水平扩展和高可用。

## 主要功能特性 ##

Cetus分为读写分离和分库两个版本。

### **针对读写分离版本**： ###

- 单进程无锁提升单个实例效率
- 支持透明的后端连接池
- 支持SQL读写分离
- 增强SQL路由解析与注入
- 支持prepare语句
- 支持结果集压缩
- 支持安全性管理
- 支持状态监控
- 支持tcp stream流式
- 支持域名连接后端
- SSL/TLS支持（正在开发中）
- MGR支持（正在开发中）
- 读强一致性支持（待实现）

### **针对分库版本：** ###

- 单进程无锁提升单个实例效率
- 支持透明的后端连接池
- 支持数据分库
- 支持分布式事务处理
- 支持insert批量操作
- 支持有条件的distinct操作
- 增强SQL路由解析与注入
- 支持结果集压缩
- 具有性能优越的结果集合并算法
- 支持安全性管理
- 支持状态监控
- 支持tcp stream流式
- 支持域名连接后端
- SSL/TLS支持（正在开发中）
- MGR支持（待实现）
- 读强一致性支持（待实现）

## **详细说明** ##

## **Cetus安装与使用** ##


### [一. Cetus 快速入门](https://github.com/Lede-Inc/cetus/blob/master/doc/cetus-quick-try.md) ###

#### 环境说明 ####

MySQL建议使用5.7.16以上版本，若使用读写分离功能则需要搭建MySQL主从关系，若使用sharding功能则需要根据业务进行分库设计；创建用户和密码并确认Cetus可以远程登录MySQL。

#### 安装 ####

Cetus只支持linux系统，安装步骤参考[Cetus 安装说明](https://github.com/Lede-Inc/cetus/blob/master/doc/cetus-install.md)，配置根据安装的不同版本详见[Cetus 读写分离版配置文件说明](https://github.com/Lede-Inc/cetus/blob/master/doc/cetus-rw-profile.md)、[Cetus 分库(sharding)版配置文件说明](https://github.com/Lede-Inc/cetus/blob/master/doc/cetus-shard-profile.md)。

#### 部署 ####

若使用读写分离功能则可以配置一主多从结构，即配置一个主库，0个或多个从库；若使用sharding功能则可以根据分库规则配置多个后端数据库。

#### 启动 ####


	bin/cetus --defaults-file=conf/proxy.conf|shard.conf [--conf-dir＝/home/user/cetus_install/conf/]


#### 连接 ####

Cetus对外暴露两类端口：proxy｜shard端口和admin端口。proxy｜shard端口是Cetus的应用端口，用来与后台数据库和前端应用进行交互；admin端口是Cetus的管理端口，用户可以连接Cetus的管理端口对Cetus的后端状态、配置参数等进行查看和修改。

##### 1. 连接Cetus应用端口 #####


    $ mysql --prompt="proxy> " --comments -h**.**.**.** -P**** -u**** -p***
    proxy> 


在连接Cetus时，使用在配置文件中确认好的用户名和密码登陆，登陆的ip和端口为Cetus监听的proxy-address的ip和端口。

可同时启动监听同一个ip不同端口的Cetus。连接应用端口之后可正常发送Cetus兼容的sql语句。

具体使用说明根据版本情况详见[Cetus 读写分离版使用指南](https://github.com/Lede-Inc/cetus/blob/master/doc/cetus-rw.md)、[Cetus 分库(sharding)版使用指南](https://github.com/Lede-Inc/cetus/blob/master/doc/cetus-sharding.md)

####  2. 连接Cetus管理端口 ####


   $  mysql --prompt="admin> " --comments -h**.**.**.** -P**** -u**** -p***
   admin> select * from backends；


可以使用在配置文件中的admin用户名和密码，登陆地址为admin-address的mysql对Cetus进行管理，例如在查询Cetus的后端详细信息时，可以登录后通过命令 select * from backends，显示后端端口的地址、状态、读写类型，以及读写延迟时间和连接数等信息。

具体使用说明根据版本情况详见[Cetus 读写分离版管理手册](https://github.com/Lede-Inc/cetus/blob/master/doc/cetus-rw-admin.md)、[Cetus 分库(sharding)版管理手册](https://github.com/Lede-Inc/cetus/blob/master/doc/cetus-shard-admin.md)

**注：Cetus读写分离和分库两个版本的使用约束详见[Cetus 使用约束说明](https://github.com/Lede-Inc/cetus/blob/master/doc/cetus-constraint.md)**


### [二. Cetus 安装说明](https://github.com/Lede-Inc/cetus/blob/master/doc/cetus-install.md) ###

#### 安装依赖 ####

编译安装Cetus存在以下依赖：

- cmake
- gcc
- glib2-devel
- flex
- libevent-devel
- mysql-devel／mariadb-devel
- tcmalloc (由于malloc存在着潜在的内存碎片问题，建议采用tcmalloc)   #不需要安装编译器中的选项

请确保在编译安装Cetus前已安装好相应的依赖。

在centos中即
	yum -y install cmake gcc glib2-devel flex libevent-devel mysql-devel 

#### 安装步骤 ####

Cetus利用自动化建构系统CMake进行编译安装，其中描述构建过程的构建文件CMakeLists.txt已经在源码中的主目录和子目录中，下载源码并解压后具体安装步骤如下：

- 创建编译目录：在源码主目录下创建独立的目录build，并转到该目录下

代码：

	mkdir build/

	cd build/


- 编译：利用cmake进行编译，指令如下


读写分离版本：

	cmake ../ -DCMAKE_BUILD_TYPE=Debug -DCMAKE_INSTALL_PREFIX=/home/user/cetus_install -DSIMPLE_PARSER=ON

分库版本：

	cmake ../ -DCMAKE_BUILD_TYPE=Debug -DCMAKE_INSTALL_PREFIX=/home/user/cetus_install -DSIMPLE_PARSER=OFF



其中CMAKE_BUILD_TYPE变量可以选择生成 debug 版和或release 版的程序，CMAKE_INSTALL_PREFIX变量确定软件的实际安装目录的绝对路径；SIMPLE_PARSER变量确定软件的编译版本，设置为ON则编译读写分离版本，否则编译分库版本。

该过程会检查您的系统是否缺少一些依赖库和依赖软件，可以根据错误代码安装相应依赖。

- 安装：执行make install进行安装

代码：
	
	make install


- 配置：Cetus运行前还需要编辑配置文件

代码：

	cd /home/user/cetus_install/conf/
	cp XXX.json.example XXX.json
	cp XXX.conf.example XXX.conf
	vi XXX.json
	vi XXX.conf
	chmod 600 proxy.conf


配置文件在make insatll后存在示例文件，以.example结尾，目录为/home/user/cetus_install/conf/，包括用户设置文件（users.json）、变量处理配置文件（variables.json）、分库版本的分片规则配置文件（sharding.json）、读写分离版本的启动配置文件（proxy.conf）和分库版本的启动配置文件（shard.conf）。

根据具体编译安装的版本编辑相关配置文件，若使用读写分离功能则需配置users.json和proxy.conf，若使用sharding功能则需配置users.json、sharding.json和shard.conf，其中两个版本的variables.json均可选配。

配置文件的具体说明见[Cetus 读写分离版配置文件说明](https://github.com/Lede-Inc/cetus/blob/master/doc/cetus-rw-profile.md)和[Cetus 分库(sharding)版配置文件说明](https://github.com/Lede-Inc/cetus/blob/master/doc/cetus-shard-profile.md)。

- 启动：Cetus可以利用bin/cetus启动

```
	bin/cetus --defaults-file=conf/XXX.conf [--conf-dir＝/home/user/cetus_install/conf/]  #一般XXX.conf表示proxy.conf
```

其中Cetus启动时可以添加命令行选项，--defaults-file选项用来加载启动配置文件，且在启动前保证启动配置文件的权限为660；--conf-dir是可选项，用来加载其他配置文件(.json文件)，默认为当前目录下conf文件夹。

Cetus可起动守护进程后台运行，也可在进程意外终止自动启动一个新进程，可通过启动配置选项进行设置。


### [三. Cetus 读写分离版配置文件说明](https://github.com/Lede-Inc/cetus/blob/master/doc/cetus-rw-profile.md) ###


读写分离版配置文件包括用户配置文件（users.json）、变量处理配置文件（variables.json）和启动配置文件（proxy.conf），具体说明如下：

#### 1.users.json ####


	{
	        "users":        [{
	                        "user": "XXXX",
	                        "client_pwd":   "XXXXXX",
	                        "server_pwd":   "XXXXXX"
	                }, {
	                        "user": "XXXX",
	                        "client_pwd":   "XXXXXX",
	                        "server_pwd":   "XXXXXX"
	                }]
	}


users.json用来配置用户登陆信息，采用键值对的结构，其中键是固定的，值是用户在MySQL创建的登陆用户名和密码。

其中user的值是用户名；client_pwd的值是前端登录Cetus的密码；server_pwd的值是Cetus登录后端的密码。

例如：


	{
	       "users":        [{
	                       "user": "root",
	                       "client_pwd":   "123",
	                       "server_pwd":   "123456"
	               }, {
	                       "user": "test",
	                       "client_pwd":   "456",
	                       "server_pwd":   "123456"
	               }]
	}


我们配置了2个用户名root和test。其中root用户前端登录Cetus的密码是123，Cetus登录后端的密码是123456；test用户前端登录Cetus的密码是456，Cetus登录后端的密码是123456。

#### 2.variables.json ####

Cetus支持部分会话级系统变量的设置，可以通过在variables.json配置允许发送的值和静默处理的值，如下：


	{
	  "variables": [
	    {
	      "name": "XXXXX",
	      "type": "XXXX",
	      "allowed_values": ["XXX"]
	    },
	    {
	      "name": "XXXXX",
	      "type": "XXXX",
	      "allowed_values": ["XXX"],
	      "silent_values": ["XX"]
	    }
	  ]
	}


variables.json同样采用键值对的结构，其中键是固定的，值是用用户自定义的。

其中name的值是需要设置的会话级系统变量的名称；type的值是变量的类型，可以为string或string-csv逗号分隔的字符串值，目前尚未支持int类型；allowed_values的值是指定允许设定的变量值，可以使用通配符\*表示此变量设任意值都允许；silent_values的值是指定静默处理的值，可以使用通配符\*，表示此变量设任意值都静默处理。

**注意：配置过allowed_values才能走到静默处理流程**

例如：

	{
	 "variables": [
	   {
	     "name": "sql_mode",
	     "type": "string-csv",
	     "allowed_values":
	     ["STRICT_TRANS_TABLES",
	       "NO_AUTO_CREATE_USER",
	       "NO_ENGINE_SUBSTITUTION"
	     ]
	   },
	   {
	     "name": "connect_timeout",
	     "type": "string",
	     "allowed_values": ["*"],
	     "silent_values": ["10", "100"]
	   }
	 ]
	}


我们配置了sql_mode变量和connect_timeout变量。其中sql_mode变量的类型是string-csv（逗号分隔的字符串值），指定了允许设定的变量有STRICT_TRANS_TABLES、NO_AUTO_CREATE_USER和NO_ENGINE_SUBSTITUTION；connect_timeout变量的类型是string（字符串），此变量设任意值都允许，指定静默处理的值为10和100。

#### 3.proxy.conf ####


	[cetus]
	# Loaded Plugins
	plugins=XXX,XXX
	
	# Proxy Configuration
	proxy-address=XXX.XXX.XXX.XXX:XXXX
	proxy-backend-addresses=XXX.XXX.XXX.XXX:XXXX
	proxy-read-only-backend-addresses=XXX.XXX.XXX.XXX:XXXX
	
	# Admin Configuration
	admin-address=XXX.XXX.XXX.XXX:XXXX
	admin-username=XXXX
	admin-password=XXXXXX
	
	# Backend Configuration
	default-db=XXXX
	default-username=XXXXX
	
	# File and Log Configuration
	log-file=XXXX
	log-level=XXXX


proxy.conf是读写分离版本的启动配置文件，在启动Cetus时需要加载，配置文件采用key＝value的形式，其中key是固定的，可参考[Cetus 启动配置选项说明](https://github.com/Lede-Inc/cetus/blob/master/doc/cetus-configuration.md)，value是用户自定义的。

例如：


	[cetus]
	# Loaded Plugins
	plugins=proxy,admin
	
	# Proxy Configuration
	proxy-address=127.0.0.1:1234
	proxy-backend-addresses=127.0.0.1:3306
	proxy-read-only-backend-addresses=127.0.0.1:3307
	
	# Admin Configuration
	admin-address=127.0.0.1:5678
	admin-username=admin
	admin-password=admin
	
	# Backend Configuration
	default-db=test
	default-username=dbtest
	
	# File and Log Configuration
	log-file=cetus.log
	log-level=debug


我们配置了读写分离版本的启动选项，其中plugins的值是加载插件的名称，读写分离版本需加载的插件为proxy和admin；

proxy-address的值是Proxy监听的IP和端口，我们设置为127.0.0.1:1234；proxy-backend-addresses的值是读写后端(主库)的IP和端口，我们设置为127.0.0.1:3306，可多项；proxy-read-only-backend-addresses的值是只读后端(从库)的IP和端口，我们设置为127.0.0.1:3307，可多项；

admin-address的值是管理模块的IP和端口，我们设置为127.0.0.1:5678；admin-username的值是管理模块的用户名，我们设置为admin；admin-password的值是管理模块的密码明文，我们设置为admin；

default-db的值是默认数据库，当连接未指定db时，使用的默认数据库名称，我们设置为test；default-username的值是默认登陆用户名，在Proxy启动时自动创建连接使用的用户名，我们设置为dbtest；

log-file的值是日志文件路径，我们设置为当前安装路径下的cetus.log；log-level的值是日志记录级别，可选 info | message | warning | error | critical(default)，我们设置为debug；这些是必备启动选项，其他可选的性能配置详见[Cetus 启动配置选项说明](https://github.com/Lede-Inc/cetus/blob/master/doc/cetus-configuration.md)。

**注：**

**以上配置文件中.json文件名称不可变，.conf文件可自定义名称，并利用命令行加载**

**启动配置文件proxy.conf 常用参数：**

**1）default-pool-size=\<num\>，设置刚启动的连接数量**

**2）max-pool-size=\<num\>，设置最大连接数量**

**3）max-resp-size=\<num\>，设置最大响应大小，一旦超过此大小，则会报错给客户端**

**4）enable-client-compress=\[true\|false\]，支持客户端压缩**

**5）enable-tcp-stream=\[true\|false\]，启动tcp stream，无需等响应收完就发送给客户端**

**6）master-preferred=\[true\|false\]，除非注释强制访问从库，否则一律访问主库**

**7）reduce-connections=\[true\|false\]，自动减少过多的后端连接数量**


### [四. Cetus 分库(sharding)版配置文件说明](https://github.com/Lede-Inc/cetus/blob/master/doc/cetus-shard-profile.md) 

分库(sharding)版配置文件包括用户配置文件（users.json）、变量处理配置文件（variables.json）、分库版本的分片规则配置文件（sharding.json）和分库版本的启动配置文件（shard.conf），具体说明如下：

#### 1. users.json ####


	{
	        "users":        [{
	                        "user": "XXXX",
	                        "client_pwd":   "XXXXXX",
	                        "server_pwd":   "XXXXXX"
	                }, {
	                        "user": "XXXX",
	                        "client_pwd":   "XXXXXX",
	                        "server_pwd":   "XXXXXX"
	                }]
	}


users.json用来配置用户登陆信息，采用键值对的结构，其中键是固定的，值是用户在MySQL创建的登陆用户名和密码。

其中user的值是用户名；client_pwd的值是前端登录Cetus的密码；server_pwd的值是Cetus登录后端的密码。

例如：


	{
	       "users":        [{
	                       "user": "root",
	                       "client_pwd":   "123",
	                       "server_pwd":   "123456"
	               }, {
	                       "user": "test",
	                       "client_pwd":   "456",
	                       "server_pwd":   "123456"
	               }]
	}


我们配置了2个用户名root和test。其中root用户前端登录Cetus的密码是123，Cetus登录后端的密码是123456；test用户前端登录Cetus的密码是456，Cetus登录后端的密码是123456。

#### 2.variables.json ####

Cetus支持部分会话级系统变量的设置，可以通过在variables.json配置允许发送的值和静默处理的值，如下：


	{
	  "variables": [
	    {
	      "name": "XXXXX",
	      "type": "XXXX",
	      "allowed_values": ["XXX"]
	    },
	    {
	      "name": "XXXXX",
	      "type": "XXXX",
	      "allowed_values": ["XXX"],
	      "silent_values": ["XX"]
	    }
	  ]
	}


variables.json同样采用键值对的结构，其中键是固定的，值是用用户自定义的。

其中name的值是需要设置的会话级系统变量的名称；type的值是变量的类型，可以为string或string-csv逗号分隔的字符串值，目前尚未支持int类型；allowed_values的值是指定允许设定的变量值，可以使用通配符\*表示此变量设任意值都允许；silent_values的值是指定静默处理的值，可以使用通配符\*，表示此变量设任意值都静默处理。

**注意：配置过allowed_values才能走到静默处理流程**

例如：


	{
	 "variables": [
	   {
	     "name": "sql_mode",
	     "type": "string-csv",
	     "allowed_values":
	     ["STRICT_TRANS_TABLES",
	       "NO_AUTO_CREATE_USER",
	       "NO_ENGINE_SUBSTITUTION"
	     ]
	   },
	   {
	     "name": "connect_timeout",
	     "type": "string",
	     "allowed_values": ["*"],
	     "silent_values": ["10", "100"]
	   }
	 ]
	}


我们配置了sql_mode变量和connect_timeout变量。其中sql_mode变量的类型是string-csv（逗号分隔的字符串值），指定了允许设定的变量有STRICT_TRANS_TABLES、NO_AUTO_CREATE_USER和NO_ENGINE_SUBSTITUTION；connect_timeout变量的类型是string（字符串），此变量设任意值都允许，指定静默处理的值为10和100。

#### 3.sharding.json ####


	{
	  "vdb": [
	    {
	      "id": X,
	      "type": "XXX",
	      "method": "XXXX",
	      "num": X,
	      "partitions": {"XXXX1": [X,X], "XXXX2": [X,X], "XXXX3": [X,X], "XXXX4": [X,X]}
	    },
	    {
	      "id": X,
	      "type": "XXX",
	      "method": "XXXXX",
	      "num": X,
	      "partitions": {"XXXX1": XXXXXX, "XXXX2": XXXXXX, "XXXX3": XXXXXX,"XXXX4": XXXXXX}
	    }
	  ],
	  "table": [
	    {"vdb": X, "db": "XXXX", "table": "XXX", "pkey": "XX"},
	    {"vdb": X, "db": "XXXX", "table": "XXX", "pkey": "XX"},
	    {"vdb": X, "db": "XXXX", "table": "XXX", "pkey": "XX"},
	    {"vdb": X, "db": "XXXX", "table": "XXX", "pkey": "XX"}
	  ]
	  "single_tables": [
	    {"table": "XXX", "db": "XXXX", "group": "XXXX1"},
	    {"table": "XXX",  "db": "XXXX", "group": "data2"}
	  ]
	}

sharding.json是分库版本的分库规则配置文件，同样采用键值对的结构，其中键是固定的，值是由用户自定义。

其中vdb逻辑db，包含属性有id、type、method、num和partitions，id的值是逻辑db的id，type的值是分片键的类型，method的值是分片方式，num的值是hash分片的底数（range分片的num为0），partitions是分组名和分片范围的键值对,其中键和值都是用户自定义的；table是分片表，包含属性有vdb、db、table和pkey，vdb的值是逻辑db的id，db的值是物理db名，table的是分片表名，pkey的值是分片键；single_tables是单点全局表，包含属性有table、db和group，table的值是表名，db的值是物理db名，group的值是单点全局表的默认分组，可由用户自定义设置。

例如：


	{
	 "vdb": [
	   {
	     "id": 1,
	     "type": "int",
	     "method": "hash",
	     "num": 8,
	     "partitions": {"data1": [0,1], "data2": [2,3], "data3": [4,5], "data4": [6,7]}
	   },
	   {
	     "id": 2,
	     "type": "int",
	     "method": "range",
	     "num": 0,
	     "partitions": {"data1": 124999, "data2": 249999, "data3": 374999,"data4": 499999}
	   }
	  ],
	 "table": [
	   {"vdb": 1, "db": "employees_hash", "table": "dept_emp", "pkey": "emp_no"},
	   {"vdb": 1, "db": "employees_hash", "table": "employees", "pkey": "emp_no"},
	   {"vdb": 2, "db": "employees_range", "table": "dept_emp", "pkey": "emp_no"},
	   {"vdb": 2, "db": "employees_range", "table": "employees", "pkey": "emp_no"},
	 ]
	  "single_tables": [
	    {"table": "regioncode", "db": "employees_hash", "group": "data1"},
	    {"table": "countries",  "db": "employees_range", "group": "data1"}
	  ]
	}


我们配置了两种vbd分片规则，第一种规则的id为1，分片键类型是int，分片方法是hash，hash分片的底数为8，一共分了4组，分组名为data1的分片范围为0和1，分组名为data2的分片范围为2和3，分组名为data3的分片范围为4和5，分组名为data4的分片范围为6和7；第二种规则的id为2，分片键类型是int，分片方法是range，range无底数num设为0，一共分了4组，分组名为data1的分片范围为0-124999，分组名为data2的分片范围为125000-249999，分组名为data3的分片范围为250000-374999，分组名为data4的分片范围为37500-499999；

分片表table涉及两个物理db，为employees_hash和employees_range，其中employees_hash采用第一种分片规则，表dept_emp的分片键为emp_no，表employees的分片键为emp_no，employees_range采用第二种分片规则，表dept_emp的分片键为emp_no，表employees的分片键为emp_no；

单点全局表single_tables有两个，分别为employees_hash的regioncode表和employees_range的countries表，设置默认分给第一组。

#### 4.shard.conf ####


	[cetus]
	# Loaded Plugins
	plugins=XXXX,XXXX
	
	# Proxy Configuration
	proxy-address=XXX.XXX.XXX.XXX:XXXX
	proxy-backend-addresses=XXX.XXX.XXX.XXX:XXXX@XXXX1,XXX.XXX.XXX.XXX:XXXX@XXXX2,XXX.XXX.XXX.XXX:XXXX@XXXX3,XXX.XXX.XXX.XXX:XXXX@XXXX4
	
	# Admin Configuration
	admin-address=XXX.XXX.XXX.XXX:XXXX
	admin-username=XXXX
	admin-password=XXXX
	
	# Backend Configuration
	default-db=XXX
	default-username=XXXX
	
	# Log Configuration
	log-file=XXXX
	log-level=XXXX


shard.conf是分库版本的启动配置文件，在启动Cetus时需要加载，配置文件同样采用key＝value的形式，其中key是固定的，可参考[Cetus 启动配置选项说明](https://github.com/Lede-Inc/cetus/blob/master/doc/cetus-configuration.md)，value是用户自定义的。

例如：


	[cetus]
	# Loaded Plugins
	plugins=shard,admin
	
	# Proxy Configuration
	proxy-address=127.0.0.1:1234
	proxy-backend-addresses=127.0.0.1:3361@data1,127.0.0.1:3362@data2,127.0.0.1:3363@data3,127.0.0.1:3364@data4
	
	# Admin Configuration
	admin-address=127.0.0.1:5678
	admin-username=admin
	admin-password=admin
	
	# Backend Configuration
	default-db=test
	default-username=dbtest
	
	# Log Configuration
	log-file=cetus.log
	log-level=debug


我们配置了分库版本的启动选项，其中plugins的值是加载插件的名称，分库（sharding）版本需加载的插件为shard和admin；

proxy-address的值是Proxy监听的IP和端口，我们设置为127.0.0.1:1234；proxy-backend-addresses的值是后端的IP和端口，需要同时指定group（@group），本例分为4个group，分别data1的127.0.0.1:3361、data2的127.0.0.1:3362、data3的127.0.0.1:3363、data4的127.0.0.1:3364；

admin-address的值是管理模块的IP和端口，我们设置为127.0.0.1:5678；admin-username的值是管理模块的用户名，我们设置为admin；admin-password的值是管理模块的密码明文，我们设置为admin；

default-db的值是默认数据库，当连接未指定db时，使用的默认数据库名称，我们设置为test；default-username的值是默认登陆用户名，在Proxy启动时自动创建连接使用的用户名，我们设置为dbtest；

log-file的值是日志文件路径，我们设置为当前安装路径下的cetus.log；log-level的值是日志记录级别，可选 info | message | warning | error | critical(default)，我们设置为debug；这些是必备启动选项，其他可选性能配置详见[Cetus 启动配置选项说明](https://github.com/Lede-Inc/cetus/blob/master/doc/cetus-configuration.md)。

**注：**

**以上配置文件中.json文件名称不可变，.conf文件可自定义名称，并利用命令行加载**

**启动配置文件shard.conf 常用参数：**

**1）default-pool-size=\<num\>，设置刚启动的连接数量**

**2）max-pool-size=\<num\>，设置最大连接数量**

**3）max-resp-size=\<num\>，设置最大响应大小，一旦超过此大小，则会报错给客户端**

**4）enable-client-compress=\[true\|false\]，支持客户端压缩**

**5）enable-tcp-stream=\[true\|false\]，启动tcp stream，无需等响应收完就发送给客户端**

**6）master-preferred=\[true\|false\]，除非注释强制访问从库，否则一律访问主库**

**7）reduce-connections=\[true\|false\]，自动减少过多的后端连接数量**


### [五. Cetus 启动配置选项说明](https://github.com/Lede-Inc/cetus/blob/master/doc/cetus-configuration.md) ###


#### 常规配置 ####

##### daemon #####

Default: false

通过守护进程启动

> daemon = true

#### user ####

Default: root

启动进程的用户，只有以root身份运行时才能使用

> user = cetus

#### basedir ####

基础路径，其它配置可以以此为基准配置相对路径。(必须是绝对路径)

> basedir = /usr/lib/cetus

#### conf-dir ####

Default: conf

配置文件路径，包括：用户设置文件、变量处理配置文件、分库版本的分片规则配置文件、读写分离版本的启动配置文件和分库版本的启动配置文件。

> conf-dir = /usr/lib/cetus/conf

#### pid-file ####

`必要`

PID文件路径

> pid-file = /var/log/cetus.pid

#### log-file ####

`必要`

日志文件路径

> log-file = /var/log/cetus.log

#### log-level ####

可选值: debug | info | message | warning | error | critical(default)

日志级别

> log-level = info

#### log-use-syslog ####

系统日志文件路径，与log-file不可同时设置。

> log-use-syslog = /var/log/cetus_sys.log

#### log-xa-file ####

xa日志路径（分库中有效）

> log-xa-file = logs/cetus.log

#### log-xa-in-detail ####

Default: false

记录xa日志详情（分库中有效）

> log-xa-in-detail = true

#### plugins ####

`可多项`

加载模块名称

> plugins = admin,proxy

#### plugin-dir ####

库文件路径

> plugin-dir = /usr/lib/cetus/plugins

#### Proxy配置 ####

#### proxy-address ####

Default: :4040

Proxy监听的IP和端口

> proxy-address = 127.0.0.1:4440

#### proxy-allow-ip ####

`可在Admin模块中动态更改`

Proxy允许访问的"用户@IP"

参数未设置时，没有限制；"User@IP"限制特定的用户和IP组合访问；"IP"允许该IP的所有用户访问

> proxy-allow-ip = root@127.0.0.1,10.238.7.6

#### proxy-backend-addresses ####

Default: 127.0.0.1:3306

`可多项`

读写后端(主库)的IP和端口

> proxy-backend-addresses = 10.120.12.12:3306

若是分库模式，需要同时指定group

> proxy-backend-addresses = 10.120.12.12:3306@data1

#### proxy-read-only-backend-addresse ####s

`可多项`

只读后端(从库)的IP和端口

> proxy-read-only-backend-addresses = 10.120.12.13:3307

若是分库模式，需要同时指定group

> proxy-read-only-backend-addresses = 10.120.12.13:3307@data1

#### proxy-connect-timeout ####

Default: : 2 (seconds)

连接Proxy的超时时间

> proxy-connect-timeout = 1

#### proxy-read-timeout ####

Default: : 10 (minutes)

读Proxy的超时时间

> proxy-read-timeout = 1

#### proxy-write-timeout ####

Default: : 10 (minutes)

写Proxy的超时时间

> proxy-write-timeout = 1

#### default-username ####

默认用户名，在Proxy启动时自动创建连接使用的用户名

> default-username = default_user

#### default-db ####

默认数据库，当连接未指定db时，使用的默认数据库名称

> default-db = test

#### default-pool-size ####

Default: 100

当前连接数不足此值时，会自动创建连接

> default-pool-size = 200

#### max-pool-size ####

Default: default-pool-size * 2

连接池的最大连接数，超过此数目的连接不会放入连接池

> max-pool-size = 300

#### max-resp-size ####

Default: 10485760 (10MB)

每个后端返回结果集的最大数量

> max-resp-size = 1024

#### master-preferred ####

`可在Admin模块中动态更改`

Proxy在读写分离时可以指定访问的库

参数未设置时，没有限制；设置为true时仅访问读写后端(主库)，除非利用注释强制走从库

> master-preferred = true

#### read-master-percentag ####e

读取主库的百分比

> read-master-percentage = 50

#### reduce-connections ####

自动减少空闲连接

> reduce-connections = true

#### default-charset ####

默认数据库字符标码方式

> default-charset = gbk

> default-charset = utf8   #未测试

#### enable-client-found-rows ####

Default: false

允许客户端使用FOUND_ROWS标志

> enable-client-found-rows = true

#### worker_id ####

自增guid的worker id，最大值为63最小值为1

> worker_id = 4

#### Admin配置 ####

#### admin-address ####

Default: :4041

管理模块的IP和端口

> admin-address = 127.0.0.1:4441

#### admin-allow-ip ####

`可在Admin模块中动态更改`

参数未设置时，不作限制；仅能限制IP不区分用户

> admin-allow-ip = 127.0.0.1,10.238.7.6

#### admin-username ####

`必要`

管理模块的用户名

> admin-username = admin

#### admin-password ####

`必要`

管理模块的密码明文

> admin-password = admin_pass

#### 远端配置中心 ####

可选择配置远端db，通过配置中心获取分库模式的配置

#### remote-conf-url ####

远端配置中心信息

> remote-conf-url = mysql://dbuser:dbpassword@host:port/schema

或者

> remote-conf-url = sqlite://dbuser:dbpassword@host:port/schema

配置中心端口port可选填，默认3306

#### 辅助线程配置 ####

#### disable-threads ####

Default: false

禁用辅助线程，包括: 配置变更检测、后端存活检测和只读库延迟检测等

> disable-threads = true

#### check-slave-delay ####

Default: false

是否检查从库延迟

> check-slave-delay = true

#### slave-delay-down ####

Default: 60 (seconds)

从库延迟超过该秒，状态将被设置为DOWN

> slave-delay-down = 10

#### slave-delay-recover ####

Default: slave-delay-down / 2  (seconds)

从库延迟少于该秒数，状态将恢复为UP

> slave-delay-recover = 5

#### 其它 ####

#### verbose-shutdown ####

Default: false

程序退出时，记录下退出代码。

> verbose-shutdown = true

#### keepalive ####

Default: false

当Proxy进程意外终止，会自动启动一个新进程

> keepalive = true

#### max-open-files ####

Default: 根据操作系统

最大打开的文件数目(ulimit -n)

> max-open-files = 1024

#### max-allowed-packet ####

Default: 33554432 (32MB)

最大允许报文大小

> max-allowed-packet = 1024

#### disable-dns-cache ####

Default: false

禁用解析连接到后端的域名

> disable-dns-cache = true

#### long-query-time ####

Default: 65536 (millisecond)

慢查询记录阈值(毫秒)

> long-query-time = 500

#### log-backtrace-on-crash ####

Default: false

程序崩溃时启动gdb调试器

> log-backtrace-on-crash = true

#### enable-back-compress ####

Default: false

启用后端传给Cetus的结果集压缩，一般不启用

> enable-back-compress ＝ true

#### merged-output-size ####

Default: 8192

tcp流式结果集合并输出阈值，超过此大小，则输出

> merged-output-size = 2048

#### default-query-cache-timeout ####

Default: 100

设置query cache的默认超时时间，单位为ms

> default-query-cache-timeout = 60

#### enable-query-cache ####

Default: false

开启Proxy请求缓存

> enable-query-cache = true

#### max-header-size ####

Default:  65536

设置响应中header最大大小，供tcp stream使用，如果响应头部特别大，需要设置更大的大小

> max-header-size = 131072

#### enable-tcp-stream ####

Default: false

采用tcp stream来输出响应，规避内存炸裂等问题

> enable-tcp-stream = true


### [六. Cetus 使用约束说明](https://github.com/Lede-Inc/cetus/blob/master/doc/cetus-constraint.md) ###


Cetus分为读写分离和分库两个版本，具有如下使用约束：

#### 针对两个版本 ####

##### 1.不支持批量sql语句的执行 #####

##### 2.不支持TLS #####

##### 3.不支持多租户 #####

##### 4.单进程工作模式，建议在docker容器使用 #####

##### 5.只支持linux系 ####统#

##### 6.不支持客户端ctl+c操作，即不支持kill query操 ####作#

##### 7.set命令的有限支持，不支持global级别的set命令，支持部分session级别的set命令 #####

##### 8.sql语句的有限支持，包括以下几点： #####

##### 1）不支持将LAST_INSERT_ID 嵌套在INSERT或者其他的语句中 #####

##### 2）不支持客户端的change user命令 #####

#### 针对分库版 ####

#### 1.目前分库版不支持动态扩容 ####

#### 2.目前分库版不支持二级分区 ####

#### 3.分库版最多支持64个分库，建议4，8，16个分库 ####

#### 4.分库版不支持跨库join ####

#### 5.分库版的自增主键最好用第三方，比如redis ####

#### 6.分库版的sql限制比读写分离版的要多，除了以上针对两个版本的限制，还包括以下几点： ####


### [七. Cetus 读写分离版使用指南](https://github.com/Lede-Inc/cetus/blob/master/doc/cetus-rw.md) ###


#### 简介 ####

Cetus 读写分离版将前端发来的读请求和写请求分别发送到不同的服务器后端，由于底层的数据库都是Master/Slave架构，做到读写分离能大大提高数据库的处理能力。

#### 安装部署 ####

#### 准备 ####

**1. MySQL**

- 搭建MySQL主从关系

- 若开启主从延迟检测需创建库proxy_heart_beat和表tb_heartbeat：

代码：

    CREATE DATABASE proxy_heart_beat;

    USE proxy_heart_beat;

    CREATE TABLE tb_heartbeat (
    p_id varchar(128) NOT NULL,
    p_ts timestamp(3) NOT NULL DEFAULT CURRENT_TIMESTAMP(3),
    PRIMARY KEY (p_id)
    ) ENGINE = InnoDB DEFAULT CHARSET = utf8;


- 创建用户和密码（默认用户对tb_heartbeat有读写权限）

- 确认Cetus可以远程登录MySQL

**2.Cetus**

- 根据MySQL后端信息配置users.json和proxy.conf（variables.json可选配），具体配置说明详见[Cetus 读写分离版配置文件说明](https://github.com/Lede-Inc/cetus/blob/master/doc/cetus-rw-profile.md)

**3.LVS & keepalived**

- 确定LVS的监听ip和端口

- 根据实际情况和需求确定多个Cetus的LVS分发权重

- 配置keepalived.conf

#### 安装 ####

Cetus只支持linux系统，安装过程详见[Cetus 安装说明](https://github.com/Lede-Inc/cetus/blob/master/doc/cetus-install.md)

#### 部署 ####

Cetus 在部署时架构图如下图所示。

![](https://i.imgur.com/KO2rZRL.png)

Cetus位于应用程序与MySQL数据库之间，作为前端应用与数据库的通讯。其中，前端应用连接LVS节点，LVS节点映射端口到多个Cetus服务，后者通过自身的连接池连接到后端的数据库。

同一个Cetus连接的MySQL数据库后端为一主一从或一主多从，不同Cetus共用后端。

MySQL和Cetus可以部署在同一台服务器上，也可以部署在各自独立的服务器上，但一般LVS & keepalived与MySQL、Cetus分布在不同的服务器上。

#### 启动 ####

Cetus可以利用bin/cetus启动

```
	bin/cetus --defaults-file=conf/proxy.conf [--conf-dir＝/home/user/cetus_install/conf/]
```

其中Cetus启动时可以添加命令行选项，--defaults-file选项用来加载启动配置文件，且在启动前保证启动配置文件的权限为660；--conf-dir是可选项，用来加载其他配置文件(.json文件)，默认为当前目录下conf文件夹。

Cetus可起动守护进程后台运行，也可在进程意外终止自动启动一个新进程，可通过启动配置选项进行设置。

#### 主要功能概述 ####

##### 1.连接池功能 #####

Cetus内置了连接池功能。该连接池会在Cetus启动时，使用默认用户自动创建到后端的连接。创建的连接使用的database是配置的默认database，这些选项是由DBA根据后端数据库的实际情况进行的配置。以确保主要的业务请求过来不需要临时建立连接并切换数据库。

##### 2.读写分离功能 #####

Cetus的读写分离功能可以通过解析SQL，结合请求状态将查询语句下发到后端的只读从库进行查询，从而减少主库的负载。提升系统可用容量。

Cetus能提供对分流策略的自定义。比如可以设置为：把30%的读流量，分流到主节点，70%的读流量分流到只读节点。

由于读写分离需要对SQL进行解析才能实现判断，所以可能出现误判的情况，请大家书写SQL时尽量按照标准语法进行。并且在发现可疑问题时，及时进行抓包，方便开发进行分析、判断。

##### 3.结果集压缩 #####

由于当连接距离较远网络延迟较大时，结果集较大会很大幅度地增加数据传输时长，降低性能，因此针对高延迟场合，Cetus支持对结果集的压缩来提高性能。

网络延迟越大，开启结果集压缩功能的优势越大，当连接距离较近几乎没有网络延迟时，Cetus无形中增加了压缩和解压步骤，反而降低了性能，因此在延迟较小时不建议使用结果集压缩。

##### 4.安全性管理 #####

安全性管理功能包括后端管理、基本配置管理、查看连接信息、用户密码管理、IP许可管理、远程配置中心管理和整体信息查询。

用户可以通过登录管理端口，查看后端状态，增删改指定后端，查看和修改基本配置（包括连接池配置和从库延迟检测配置），查看当前连接的详细信息，用户连接Cetus的密码查询和增删改，查看以及增删改IP许可(可以为每个用户以及管理员用户指定允许访问的来源ip地址，但是无法为来自不同ip的同一用户提供不同的权限设置)，重载远程配置和查看Cetus总体状态等，具体管理操作相见[Cetus 读写分离版管理手册](https://github.com/Lede-Inc/cetus/blob/master/doc/cetus-rw-admin.md)。

##### 5.状态监控 #####

Cetus内置监控功能，可通过配置选择开启或关闭。开启后Cetus会定期和后端进行通信，检测后端数据库的存活状态和主从延迟时间等。可通过登录管理端口SELECT * FROM backends查看。

后端状态包括unknown（后端初始状态，还未建立连接）、up（能与后端正常建立连接，可以正常提供服务）、down（与后端无法联通，无法正常提供服务）、maintaining（后端正在维护，无法建立连接）和delete（后端已被删除）。

主从延迟检测检测设有阈值slave-delay-down和slave-delay-recover，检测到的从库延迟超过slave-delay-down阈值后端状态将被设置为DOWN，从库延迟少于slave-delay-recover阈值后端状态将恢复为UP。

##### 6.TCP流式 #####

针对结果集过大情况，Cetus采用了tcp stream流式，不需要先缓存完整结果集才转发给客户端，以避免内存炸裂问题，降低内存消耗，提高性能。

#### 7.支持prepare语句 #####

支持用户使用prepare语句，方法有两种：A）客户端级别的prepare ；B）server端的prepare支持。

##### 8.域名连接后端 #####

支持利用域名连接数据库后端，Cetus设有相关的启动配置选项disable-dns-cache，选择是否开启解析连接到后端的域名功能，开启后可以通过设置域名并利用域名访问后端。

#### 注意事项 ####

##### 1.连接池使用注意事项 #####

所有的后端MySQL节点，都需要有相同的用户名及口令，在conf/users.json中指定，不在选项中指定的用户，不能用来登录Cetus，不能为每个不同的后端实例单独指定登录信息。用户连接Cetus时的用户名和密码不一定与后端一致，可在conf/users.json中查看。

##### 2.set命令的支持说明 #####

我们支持使用以下session级别的set命令： Set  names/ sql_mode/ autocommit，其中Set sql_mode 仅支持以下 3 个：STRICT_TRANS_TABLES/  NO_AUTO_CREATE_USER/ NO_ENGINE_SUBSTITUTION，支持set session transaction read write/readonly等指令。global级别的set命令统一不支持，其他未列出的set命令不在支持范围内，不建议使用。如果业务确实需要，建议联系DBA将行为配置为默认开启。

支持CLIENT_FOUND_ROWS 全局参数属性的统一设置，同一个Cetus只能选择打开或者关闭，不支持对每个连接单独设置这项属性，可以通过设置启动配置选项来打开，默认关闭。不支持CLIENT_LOCAL_FILES。其他未列出的需要额外设置的特性请测试后再确认。

##### 3.环境变量修改建议 #####

虽然我们支持客户端对连接环境变量进行修改，但是，我们不建议在程序中进行修改。因为一旦变量有改动。我们需要在执行SQL前对连接状态进行复位，会产生额外的请求到服务端，客户端响应的延迟也会增加。

##### 4.使用注释来选择后端 #####

开发需要尽量在处理业务逻辑时，避免对写入的数据进行立即查询。因为主从之间可能存在一定的同步延迟。所以可能出现写入数据和读取不一致的情况，如果应用对这项特别敏感，可以使用注释的方式，提示Cetus将读请求直接发送到主库进行查询，避免延迟。

注释的使用方式为在SQL中SELECT字段之后插入/\*# mode=READWRITE \*/；部分应用可能对数据准确性特别敏感，这种情况下，我们可以设置默认所有请求都走主节点，但是，对于部分后台的统计分析功能，主要分析历史数据时，我们可以通过SQL指定后端到只读节点，来减少批量查询业务对主库的影响，我们可以在SELECT字段后插入/\*# mode=READONLY \*/来指定Cetus将SQL发送到只读从库进行执行。

##### 5.不支持 Kill query #####

不支持在SQL执行过程中 kill query操作，一旦SQL语句开始执行就不能通过这种方式来终止，此时可以连接Cetus管理后端，通过执行 show connectionlist 命令查看正在执行的SQL，从而找到正在执行的后端信息，通过数据库中 kill query的命令进行终止操作。

##### 6.不支持TLS #####

不支持TLS协议，目前不能在某种程度上使主从架构应用程序通讯本身预防窃听、干扰和消息伪造。

##### 7.不支持多租户 #####

目前读写版本还不支持，可以联系dba在数据库设计方面进行设置。

##### 8.不支持分表 #####

TODO

##### 9.SQL支持 #####

由于读写分离只需要对SQL进行路由，不需要进行SQL改写，而且每个数据库后端都是全量的数据，所以，能支持绝大多数的SQL语句。除下述需要注意的用法外，基本都支持。

**1.LAST_INSERT_ID特性有变化**

由于后端的连接存在复用。所以，在查询LAST_INSERT_ID时，会存在错误的情况，为避免这种情况，Cetus实现了缓存上次操作返回LAST_INSERT_ID的功能。目前只支持单独的SQL进行查询。不支持将LAST_INSERT_ID 嵌套在INSERT或者其他的语句中。建议的使用方式是在INSERT完之后。直接获取，或者在事后，直接使用 SELECT LAST_INSERT_ID() 来获取insert id。

**2.事务的处理**

事务中的所有操作都将发送到主库进行执行，避免数据不一致造成的干扰。

**3.尽量避免服务端PREPARE**

虽然Cetus读写分离版支持服务端的PREPARE，但是服务端的PREPARE会降低连接池的使用效率，除非应用程序不兼容。否则，不建议使用服务端PREPARE功能。

**4.DDL语句以及其他语句**

读写分离版本支持DDL语句，所有的非查询语句都将发送到主库，后续的变更需要依赖主从之间的同步保持一致。不建议程序直接调用存储过程，如需调用，需要进行详细的测试，确认满足应用的需求的情况下才能使用。

**5.不支持客户端的change user命令**

#### 应用示例 ####

##### 1.连接Cetus #####

```
    $ mysql --prompt="proxy> " --comments -h**.**.**.** -P**** -u**** -p***
    proxy> 
```

在连接Cetus时，使用在配置文件中确认好的用户名和密码登陆，登陆的ip和端口为Cetus监听的proxy-address的ip和端口。

可同时启动监听同一个ip不同端口的Cetus。

##### 2.管理Cetus #####

```
   $  mysql --prompt="admin> " --comments -h**.**.**.** -P**** -u**** -p***
   admin> select * from backends；
```

可以使用在配置文件中的admin用户名和密码，登陆地址为admin-address的MySQL对Cetus进行管理，例如在查询Cetus的后端详细信息时，可以登录后通过命令 select * from backends，显示后端端口的地址、状态、读写类型，以及读写延迟时间和连接数等信息。

具体使用说明详见[Cetus 读写分离版管理手册](https://github.com/Lede-Inc/cetus/blob/master/doc/cetus-rw-admin.md)

##### 3.查看Cetus参数 #####

```
   $  cd /home/user/cetus_install/
   bin/cetus --help | -h
```

可以查看帮助，获取Cetus启动配置选项的参数及含义。


### [八. Cetus 读写分离版管理手册](https://github.com/Lede-Inc/cetus/blob/master/doc/cetus-rw-admin.md) ###


#### 前言 ####

**有配置修改均能动态生效，配置更改后请务必修改原始配置文件，以确保下次重启时配置能够保留。**

### 查看帮助 ####
 
`select * from help`或
`select help`

查看管理端口用法

| Command                                  | Description                              |
| :--------------------------------------- | :--------------------------------------- |
| select conn_details from backend         | display the idle conns                   |
| select * from backends                   | list the backends and their state        |
| show connectionlist [\<num>]             | show \<num> connections                  |
| show allow_ip \<module>                  | show allow_ip rules of module, currently admin  or proxy or shard |
| show deny_ip \<module>                   | show deny_ip rules of module, currently admin or proxy or shard |
| add allow_ip \<module> \<address>        | add address to white list of module      |
| add deny_ip \<module> \<address>         | add address to black list of module      |
| delete allow_ip \<module> \<address>     | delete address from white list of module |
| delete deny_ip \<module> \<address>      | delete address from black list of module |
| set reduce_conns (true or false)           | reduce idle connections if set to true   |
| reduce memory                            | reduce memory occupied by system         |
| set maintain (true or false)               | close all client connections if set to true |
| show status [like '%\<pattern>%']        | show select/update/insert/delete statistics |
| show variables [like '%\<pattern>%']     | show configuration variables             |
| select version                           | cetus version                            |
| select conn_num from backends where backend_ndx=\<index> and user='\<name>') | display selected backend and its connection number |
| select * from user_pwd [where user='\<name>'] | display server username and password |
| select * from app_user_pwd [where user='\<name>'] | display client username and password |
| update user_pwd set password='xx' where user='\<name>' | update server username and password |
| update app_user_pwd set password='xx' where user='\<name>' | update client username and password |
| delete from user_pwd where user='\<name>' |  delete server username and password    |
| delete from app_user_pwd where user='\<name>' | delete client username and password |
| insert into backends values ('\<ip:port>', '(ro or rw)', '\<state>') or  add mysql instance to backends list      |
| update backends set (type or state)='\<value>' where (backend_ndx=\<index> or address='\<ip:por>') | update mysql instance type or state      |
| delete from backends where (backend_ndx=\<index> or address='\<ip:port>') or  set state of mysql instance to deleted |
| remove backend where (backend_ndx=\<index>\|address='\<ip:port>') or  set state of mysql instance to deleted |
| add master '\<ip:port>'                  | add master                               |
| add slave '\<ip:port>'                   | add slave                                |
| stats get [\<item>]                      | show query statistics                    |
| config get [\<item>]                     | show config                              |
| config set \<key>=\<value>               | set config                               |
| stats reset                              | reset query statistics                   |
| save settings                            | not implemented                          |
| select * from help                       | show this help                           |
| select help                              | show this help                           |
| cetus                                    | Show overall status of Cetus             |

结果说明：

读写分离版本管理端口提供了36条语句对cetus进行管理，具体用法见以下说明。

#### 后端配置 ####

##### 查看后端 #####

`select * from backends`

查看后端信息。

| backend_ndx | address        | state | type | slave delay | uuid | idle_conns | used_conns | total_conns |
| :---------- | :------------- | :---- | :--- | :---------- | :--- | :--------- | :--------- | :---------- |
| 1           | 127.0.0.1:3306 | up    | rw   | NULL        | NULL | 100        | 0          | 100         |
| 2           | 127.0.0.1:3307 | up    | ro   | 0           | NULL | 100        | 0          | 100         |

结果说明：

* backend_ndx: 后端序号，按照添加顺序排列；
* address: 后端地址，IP:PORT格式；
* state: 后端状态(unknown|up|down|maintaining|deleted)；
* type: 读写类型(rw|ro)；
* slave delay: 主从延迟时间(单位：毫秒)；
* uuid: 暂时无用；
* idle_conns: 空闲连接数；
* used_conns: 正在使用的连接数；
* total_conns: 总连接数。


状态说明

	unknown:     后端初始状态，还未建立连接;
	up:          能与后端正常建立连接；
	down:        与后端无法联通(如果开启后端状态检测，能连通后自动变为UP);
	maintaining: 后端正在维护，无法建立连接或自动切换状态(此状态由管理员手动设置);
	deleted:      后端已被删除，无法再建立连接。


### 查看后端连接状态

`select conn_details from backends`

查看每个用户占用和空闲的后端连接数。

| backend_ndx | username | idle_conns | used_used_conns | total_used_conns |
| :---------- | :------- | :--------- | :-------------- | ---------------- |
| 1           | test1    | 2          | 0               | 0                |
| 2           | test2    | 11         | 0               | 0                |

结果说明：

* backend_ndx: 后端序号；
* username: 用户名；
* idle_conns: 空闲连接数；
* used_used_conns：正在使用的连接数。
* total_used_conns: 总的连接数。

##### 添加后端 ####

`add master '<ip:port>'`

添加一个读写类型的后端。

例如

>add master '127.0.0.1:3307'

`add slave '<ip:port>'`

添加一个只读类型的后端。

例如

>add slave '127.0.0.1:3360'

`insert into backends VALUES ('<ip:port>', '(ro|rw)', '<state>')`

添加一个后端，同时指定读写类型。

例如

>insert into backends values ('127.0.0.1:3306', 'rw', 'up');

##### 删除后端 ####

`remove backend <backend_ndx>` 或
`delete from backends where backend_ndx = <backend_ndx>`

删除一个指定序号的后端。

例如

>remove backend 1

`delete from backends where address = '<ip:port>'`

删除一个指定地址的后端。

例如

>delete from backends where address = '127.0.0.1:3306'

##### 修改后端 ####

`update backends set (type|state)='<value>' WHERE (backend_ndx=<index>|address='<ip:port>')`

修改后端类型或状态。

例如

>update backends set type='rw' where address='127.0.0.1:3306'

>update backends set state='up' where backend_ndx=1

```
说明
update后端的state只包括up|down|maintaining三种状态，delete/remove后端可将后端的state设为deleted状态。
```

#### 基本配置 ####

##### 查看连接池/通用配置 #####

`config get [<item>]`

`config get`查看支持的配置类型
   * `pool`连接池配置
   * `common`通用配置

`config get common`查看通用配置
   * `common.check_slave_delay` 是否需要检测从库延迟
   * `common.slave_delay_down_threshold_sec` 若延迟大于此值(秒)，后端状态置为DOWN
   * `common.slave_delay_recover_threshold_sec` 若延迟小于此值(秒)，后端状态置为UP

`config get pool`查看连接池配置
   * `pool.default_pool_size` 默认连接池大小
   * `pool.max_pool_size` 最大连接数量
   * `pool.max_resp_len` 最大结果集长度
   * `pool.master_preferred` 是否只允许走主库

##### 修改连接池/通用配置 #####

`config set [<item>]`

`config set common.[option] = [value]`修改基本配置

例如

>config set common.slave_delay_down = 3

`config set pool.[option] = [value]`修改连接池配置

例如

>config set pool.max_pool_size = 200

##### 查看参数配置 #####

`show variables [like '%<pattern>%']`

查看的参数均为启动配置选项中的参数，详见[Cetus 启动配置选项说明](https://github.com/Lede-Inc/cetus/blob/master/doc/cetus-configuration.md)。

#### 查看/设置连接信息 ####

##### 查看当前连接的详细信息 #####

`show connectionlist`

将当前全部连接的详细内容按表格显示出来。

| User  | Host           | db   | Command | Time | Trans | PS   | State      | Server | Info |
| ----- | -------------- | ---- | ------- | ---- | ----- | ---- | ---------- | ------ | ---- |
| test1 | 127.0.0.1:3306 | test | Sleep   | 0    | N     | N    | READ_QUERY | NULL   | NULL |
| test2 | 127.0.0.1:3307 | test | Sleep   | 0    | N     | N    | READ_QUERY | NULL   | NULL |

结果说明：

* User: 用户名;
* Host: 客户端的IP和端口;
* db: 数据库名称;
* Command: 执行的sql，"Sleep"代表当前空闲;
* Time: 已执行的时间;
* Trans: 是否在事务中;
* PS：是否存在prepare;
* State: 连接当前的状态，"READ_QUERY"代表在等待获取命令;
* Server: 后端地址;
* Info: 暂未知。

##### 查看某用户对某后端的连接数 #####

`select conn_num from backends where backend_ndx=<index> and user='<name>')`

例如

>select conn_num from backends where backend_ndx=2 and user='root');

##### 设置是否减少空闲连接 #####

`set reduce_conns (true|false)`

例如

>set reduce_conns true;

减少空闲连接。

##### 设置是否关闭所有客户端连接 #####

`set maintain (true|false)`

例如

>set maintain true;

关闭所有客户端连接。

#### 用户/密码管理 ####

##### 密码查询 #####

`select * from user_pwd [where user='<name>']`

查询某个用户的后端密码。

**注意由于密码是非明文的，仅能显示字节码。**

>select * from user_pwd where user='root';

`select * from app_user_pwd [where user='<name>']`

查询某个用户连接proxy的密码，同样是非明文。

例如

>select * from app_user_pwd where user='test';

##### 密码添加/修改 #####

`update user_pwd set password='<password>' where user='<name>'`

添加或修改特定用户的后端密码(如果该用户不存在则添加，已存在则覆盖)。

例如

>update user_pwd set password='123456' where user='test'

`update app_user_pwd set password='<password>' where user='<name>'`

添加或修改特定用户连接Proxy的密码(如果该用户不存在则添加，已存在则覆盖)。

例如

>update app_user_pwd set password='123456' where user='root'

##### 密码删除 #####

`delete from user_pwd where user='<name>'`

删除特定用户的后端密码。

例如

>delete from user_pwd where user='root'

`delete from app_user_pwd where user='<name>'`

删除特定用户连接Proxy的密码。

例如

>delete from app_user_pwd where user='root'

#### IP白名单 ####

##### 查看IP白名单 #####

`show allow_ip <module>`

\<module\>：admin|proxy

查看admin／proxy模块的IP白名单。
若列表为空，则代表没有任何限制。

##### 增加IP白名单 #####

`add allow_ip <module> <address>`

向白名单增加一个IP许可。（IP不要加引号）

\<module\>：admin|proxy

\<address\>：[[user@]IP]

```
说明
Admin: 仅能配置IP，不能限制用户(Admin有效用户只有一个)；
Proxy: 仅配置IP，代表允许该IP来源所有用户的访问；配置User@IP，代表允许该IP来源的特定用户访问。
```

例如

>add allow_ip admin 127.0.0.1

>add allow_ip proxy test@127.0.0.1

##### 删除IP白名单 #####

`Ddelete allow_ip <module> <address>`

删除白名单中的一个IP许可。（IP不要加引号）

\<module\>：admin|proxy

\<address\>：[[user@]IP]

例如

>delete allow_ip admin 127.0.0.1

>delete allow_ip proxy test@127.0.0.1

#### IP黑名单 ####

##### 查看IP黑名单 #####

`show deny_ip <module>`

\<module\>：admin|proxy

查看admin／proxy模块的IP黑名单。
若列表为空，则代表没有任何限制。

##### 增加IP黑名单 ######

`add deny_ip <module> <address>`

向黑名单中增加一个IP限制。（IP不要加引号）

\<module\>：admin|proxy

\<address\>：[[user@]IP]

```
说明
Admin: 仅能配置IP，不能限制用户(Admin有效用户只有一个)；
Proxy: 仅配置IP，代表限制该IP来源所有用户的访问；配置User@IP，代表限制该IP来源的特定用户访问。
```

例如

>add deny_ip admin 127.0.0.1

>add deny_ip proxy test@127.0.0.1

##### 删除IP黑名单 #####

`delete deny_ip <module> <address>`

删除黑名单中的一个IP限制。（IP不要加引号）

\<module\>：admin|proxy

\<address\>：[[user@]IP]

例如

>delete deny_ip admin 127.0.0.1

>delete deny_ip proxy test@127.0.0.1

**注意：IP白名单的优先级高于IP黑名单**
 
##### 保存配置到本地文件 ######

`save settings [FILE]`

保存当前配置到指定路径的本地文件中。

例如

>save settings /tmp/proxy.cnf

#### 查看整体信息 #####

##### 查看统计信息 #####

`stats get [<item>]`

`stats get`查看支持的统计类型
   * `client_query` 客户发来的SQL数量
   * `proxyed_query` 发往后端的SQL数量
   * `query_time_table` 查询时间直方图
   * `server_query_details` 每个后端接收的SQL数量
   * `query_wait_table` 等待时间直方图

`stats get client_query` `stats get proxyed_query`查看读/写SQL数量

`stats get server_query_details`查看各个后端读/写SQL数量

`stats get query_time_table` `stats get query_wait_table` 查看各时间值对应的SQL数量，如：

| name               | value |
| :----------------- | :---- |
| query_time_table.1 | 3     |
| query_time_table.2 | 5     |
| query_time_table.5 | 1     |

表示用时1秒的SQL有3条，用时2秒的SQL有5条，用时5秒的SQL有1条

```
说明
stats reset：重置统计信息 
```

##### 查看总体状态 #####

`cetus`

包括程序版本、连接数量、QPS、TPS等信息

##### 查看各类SQL统计 ######

`show status [like '%pattern%']`


	pattern参数说明
	Com_select         总的SELECT数量
	Com_insert         总的INSERT数量
	Com_update         总的UPDATE数量
	Com_delete         总的DELETE数量
	Com_select_shard   走多个节点的SELECT数量
	Com_insert_shard   走多个节点的INSERT数量
	Com_update_shard   走多个节点的UPDATE数量
	Com_delete_shard   走多个节点的DELETE数量
	Com_select_gobal   仅涉及公共表的SELECT数量
	Com_select_bad_key 分库键未识别导致走全库的SELECT数量


### 查看当前cetus版本

`select version`

## 其他

### 减少系统占用的内存

`reduce memory`


### [九. Cetus 分库(sharding)版使用指](https://github.com/Lede-Inc/cetus/blob/master/doc/cetus-sharding.md)南 ###

#### 简介 ####

Cetus sharding版支持对后端的数据库进行分库，可以根据hash/range方式对大表进行分布式部署，提升系统整体的响应和容量。

#### 安装部署 ####

##### 准备 #####

**1.MySQL**

- 5.7.17以上版本（分布式事务功能需要）

- 数据库设计（即分库，根据业务将数据对象分成若干组）

- 创建用户和密码

- 确认Cetus可以远程登录MySQL

**2.Cetus**

- 根据MySQL后端信息配置users.json、sharding.json和shard.conf（variables.json可选配），具体配置说明详见[Cetus 分库(sharding)配置文件说明](https://github.com/Lede-Inc/cetus/blob/master/doc/cetus-shard-profile.md)

**3.LVS & keepalived**

- 确定LVS的监听ip和端口

- 根据实际情况和需求确定多个Cetus的LVS分发权重

- 配置keepalived.conf

##### 安装 ####

Cetus只支持linux系统，安装过程详见[Cetus 安装说明](https://github.com/Lede-Inc/cetus/blob/master/doc/cetus-install.md)

##### 部署 #####

Cetus 在部署时架构图如下图所示。

![](https://i.imgur.com/4aRghrP.png)

Cetus位于应用程序与MySQL数据库之间，作为前端应用与数据库的通讯。其中，前端应用连接LVS节点，LVS节点映射端口到多个Cetus服务，后者通过自身的连接池连接到后端的数据库。

MySQL和Cetus可以部署在同一台服务器上，也可以部署在各自独立的服务器上，但一般LVS & keepalived与MySQL、Cetus分布在不同的服务器上。

##### 启动 #####

Cetus可以利用bin/cetus启动

```
bin/cetus --defaults-file=conf/shard.conf [--conf-dir＝/home/user/cetus_install/conf/]
```

​其中Cetus启动时可以添加命令行选项，--defaults-file选项用来加载启动配置文件，且在启动前保证启动配置文件的权限为660；--conf-dir是可选项，用来加载其他配置文件(.json文件)，默认为当前目录下conf文件夹。

​Cetus可起动守护进程后台运行，也可在进程意外终止自动启动一个新进程，可通过启动配置选项进行设置。

#####  数据库设计 #####

##### 1.专用术语 #####

**1.数据分片**

  按一定的规则，将数据表横向切分成若干部分并将其部署在一个或多个数据节点上。

**2.分片表**

  即Sharding表，将属于同一张表中的数据根据规则拆分成多个分片，分别存入底层不同的存储节点。

**3.分片键**

  即Sharding Key，Sharding表将根据此分片键再加上拆分规则，从而成为sharding表。

**4.拆分方法**

  目前支持的拆分规则有hash、range，其中hash支持数字类型、字符串类型，range支持数字类型、字符串类型、时间类型（date/datetime）。

  Hash分片即数据库中的Hash分区。首先，将Sharding表中Sharding Key的数据进行hash，然后根据制定的分片规则，进行数据存储。Sharding Key常选用数字类型。

  Range分片类似数据库中的Range分区。将Sharding表中Sharding Key的数据进行范围分片，根据规则，进行数据存储。Sharding Key常选用数字类型、时间类型（DATE/DATETIME）。

**5.全局表**

  Public表，即全局表，具有相同VDB的存储节点公共表数据是一致的，即都是全量数据，但不同VDB的存储节点公共表是不同的，如果想具有相同公共表，需要前端再处理，例如配置表conf，VDB1和VDB2都需要，需要前端应用分别写入VDB1、VDB2。

**6.单点全局表**

  Single表，即单点全局表，如果表的写操作频繁，且不合适做分片，也不需要跟其他分片表或单点全局表关联，可以把这种类型的表定义为单点全局表。单点全局表数据放在用户指定的唯一一个后端节点，只涉及单点全局表的写操作是单机事务，不会开启分布式事务，减少了事务的代价；缺点是单点全局表只能跟全局表做join关联。

**7.VDB**

  VDB（Virtual DataBase）非物理DB，即逻辑DB，主要对应分片表，表现在业务数据层，即代表此VDB内的数据有相同属性值，可以根据此属性值，进行数据的进一步拆分，从而构成分片表，例如订单数据和用户数据属于同一个VDB，可以根据用户ID进行分片，而仓储和商品可以放入另一个VDB。不同VDB之间的数据不能进行关联查询，只有在同一个VDB内才支持。

##### 2.设计原则 #####

按业务数据的内在联系（例如，支持同一业务模块，或经常需要一起存取或修改等），将数据对象分成若干个组，使同一分组的数据表高度“内聚”，不同分组之间的表高度“独立”；找出每个组中各表共同的“根元”，以此作为分片键，对数据进行分片。

VDB要在业务设计之初确定，后续的底层数据存储以及上层数据操作都会与此息息相关，所以需要开发、DBA一同来设计。

以下是具体样例：

**用户VDB（VDB_ACCOUNT）**

描述：与用户订单相关联的业务

分片键：ACCOUNT_ID

TB_ORDER（订单表）

TB_ACCOUNT（用户信息表）

TB_PAY_ORDER（支付订单表）

TB_CHARGE_ORDER（充值订单表）

TB_TRANSFER_RECORD（转账记录表）

TB_ACTIVITY_RECORD（活动记录表）

TB_REFUND_ORDER（退款记录表）

**产品VDB（VDB_PRODUCT）**

描述：与产品相关联的业务

分片键：PERIOD_ID

TB_PRODUCT（商品信息表）

TB_PRODUCT_PERIOD（商品期次表）

##### 主要功能概述 ######

##### 1.连接池功能 #####

Cetus内置了连接池功能。该连接池会在Cetus启动时，使用默认用户自动创建到后端的连接。创建的连接使用的database是配置的默认database，这些选项是由DBA根据后端数据库的实际情况进行的配置。以确保主要的业务请求过来不需要临时建立连接并切换数据库。

##### 2.数据分片功能 #####

Cetus支持对后端的数据库进行分库，可以根据hash/range方式对大表进行分布式部署，提升系统整体的响应和容量。由于分片处理后，数据的整体性被破坏，为了保证查询的结果符合预期，我们需要对SQL进行分析，并改写后发送到不同的后端。

##### 3.分布式事务处理 ######

如果前端执行SQL时，开启了事务（start transaction），则统一采用分布式事务处理（除非开启了单点事务的注释功能），如果未开启事务，直接发送SQL指令， Cetus 在处理时会判断是否开启分布式事务。

##### 4.结果集压缩 #####

由于当连接距离较远网络延迟较大时，结果集较大会很大幅度地增加数据传输时长，降低性能，因此针对高延迟场合，Cetus支持对结果集的压缩来提高性能。

网络延迟越大，开启结果集压缩功能的优势越大，当连接距离较近几乎没有网络延迟时，Cetus无形中增加了压缩和解压步骤，反而降低了性能，因此在延迟较小时不建议使用结果集压缩。

##### 5.安全性管理 #####

安全性管理功能包括后端管理、基本配置管理、查看连接信息、用户密码管理、IP许可管理、远程配置中心管理和整体信息查询。

用户可以通过登录管理端口，查看后端状态，增删改指定后端，查看和修改基本配置（包括连接池配置和从库延迟检测配置），查看当前连接的详细信息，用户连接Cetus的密码查询和增删改，查看以及增删改IP许可(可以为每个用户以及管理员用户指定允许访问的来源ip地址，但是无法为来自不同ip的同一用户提供不同的权限设置)，重载远程配置和查看Cetus总体状态等，具体管理操作相见[Cetus 分库(sharding)版管理手册](https://github.com/Lede-Inc/cetus/blob/master/doc/cetus-shard-admin.md)。

##### 6.状态监控 #####

Cetus内置监控功能，可通过配置选择开启或关闭。开启后Cetus会定期和后端进行通信，检测后端数据库的存活状态和主从延迟时间等。可通过登录管理端口SELECT * FROM backends查看。

后端状态包括unknown（后端初始状态，还未建立连接）、up（能与后端正常建立连接，可以正常提供服务）、down（与后端无法联通，无法正常提供服务）、maintaining（后端正在维护，无法建立连接）和delete（后端已被删除）。

主从延迟检测检测设有阈值slave-delay-down和slave-delay-recover，检测到的从库延迟超过slave-delay-down阈值后端状态将被设置为DOWN，从库延迟少于slave-delay-recover阈值后端状态将恢复为UP。

##### 7.TCP流式 #####

针对结果集过大情况，Cetus 采用了tcp stream流式，不需要先缓存完整结果集才转发给客户端，以避免内存炸裂问题，降低内存消耗，提高性能。

##### 8.域名连接后端 #####

支持利用域名连接数据库后端，Cetus设有相关的启动配置选项disable-dns-cache，选择是否开启解析连接到后端的域名功能，开启后可以通过设置域名并利用域名访问后端。

##### 9.insert批量操作 #####
支持在insert语句中写多个value，value之间用","隔开，例如：
INSERT INTO table (field1,field2,field3) VALUES ('a',"b","c"), ('a',"b","c"),('a',"b","c");

#### 注意事项 #####

##### 1.连接池使用注意事项 #####

所有的后端MySQL节点，都需要有相同的用户名及口令，在conf/users.json中指定，不在选项中指定的用户，不能用来登录Cetus，不能为每个不同的后端实例单独指定登录信息。用户连接Cetus时的用户名和密码不一定与后端一致，可在conf/users.json中查看。

##### 2.set命令的支持说明 #####

我们支持使用以下session级别的set命令： Set  names/autocommit。global级别的set命令统一不支持，其他未列出的set命令不在支持范围内，不建议使用。如果业务确实需要，建议联系DBA将行为配置为默认开启。

支持CLIENT_FOUND_ROWS 全局参数属性的统一设置，同一个Cetus只能选择打开或者关闭，不支持对每个连接单独设置这项属性，可以通过设置启动配置选项来打开，默认关闭。不支持CLIENT_LOCAL_FILES。其他未列出的需要额外设置的特性请测试后再确认。

### 3.环境变量修改建议

虽然我们支持客户端对连接环境变量进行修改，但是，我们不建议在程序中进行修改。因为一旦变量有改动。我们需要在执行SQL前对连接状态进行复位，会产生额外的请求到服务端，客户端响应的延迟也会增加。

##### 4.代码处理 #####

能用程序代码实现的函数尽量不用 SQL 函数，最好先在程序中计算出结果，再把值传入 SQL，如时间类函数 curdate()，字符串处理类函数 trim()。因为函数用于分库键将无法路由，导致查询语句需要发送到所有后端，单事务可以搞定的事情就变成了分布式事务，性能变差。

程序做分页任务时，尽量自己记录页偏移，因为 Cetus 做偏移时offset 会重写，SQL 把 offset 置为零，数据全部取回后再截取该页需要的数据，这样一来数据量无形中就变大很多倍，且不能有效利用数据库索引，性能比较差，而且一旦数据超出内存阈值前端将接收到错误。

##### 5.不支持 Kill query #####

不支持在 Sql 执行过程中 kill query操作，一旦 Sql 语句开始执行就不能通过这种方式来终止，此时可以连接Cetus 管理后端，通过执行 show connectionlist 命令查看正在执行的 Sql，从而找到正在执行的后端信息，通过数据库中 kill query的命令进行终止操作。

##### 6.不支持TLS #####

不支持TLS协议，目前不能在某种程度上使主从架构应用程序通讯本身预防窃听、干扰和消息伪造。

##### 7.不支持多租户 #####

目前分片版本还不支持。

##### 8.不支持动态扩容 #####

目前分库版暂不支持动态扩容，需要手工迁移数据，多数情况下需要“停机”扩容。

##### 9.分区限制 #####

目前只支持一级分区，且最多支持64个分库（全局表的更新数量太大，会导致分布式事务性能急剧恶化），建议4，8，16个分库，暂不支持二级分区；分区的自增主键最好用第三方，比如redis。

##### 10.SQL书写规范 #####

由于SQL需要进行完整解析器，建议大家在书写涉及分片的SQL时，要按照标准书写，在测试环境下测试通过后再上线，请遵循以下规范：

1. 在Select语句中尽量为表达式列或函数计算列添加别名，比如“select count(\*) rowcnt from ...”，以利于提高SQL解释器的分析水平。 
2. Sql文本要简洁，针对sharding表，如果有条件，请务必加sharding key做为过滤条件。
3. 开启事务推荐使用start transaction。
4. 少用子查询这种写法，必须用的话，可以用关联查询语法进行替换。
5. Update/delete操作要根据sharding key进行过滤后操作（仅针对分片表）。
6. For update语句不建议使用，锁开销严重，建议在应用端处理该业务逻辑，比如引入分布式锁或者先分配给redis等等。

##### 11.SQL支持 #####

Cetus sharding版能支持大多数的SQL语句，目前限制支持的功能有以下几种：

**不支持项：**

**1.不支持COUNT(DISTINCT)/SUM(DISTINCT)/AVG(DISTINCT)**

  全局表没有限制；针对分片表建议分开操作，即先用 distinct 获取所有后端节点的值，类似 select distinct val from xxx order by val，然后将数据整合到一起做去重计数／去重求和／去重求平均值的工作。

**2.不支持LAST_INSERT_ID**

  目前线上没有发现该用法，如希望获取全局唯一值建议使用 redis 获取，另外Cetus本身也提供了一种方法，select cetus_sequence()，即可返回一个 64 位递增不连续随机数字。

**3.不支持存储过程和视图**

**4.不支持批量sql语句的执行**

**5.不支持客户端的change user命令**

**6.不支持having多个条件**

**7.不支持含有any/all/some的子查询语句**

  不支持含有any/all/some的子查询语句，例如：select dept_no,emp_no from dept_emp where emp_no > any (select emp_no from dept_emp where dept_no='d001');若需要可转成关联查询语句。

**8.不支持load data infile**

**9.不支持handler语法**

**10.不支持lock tables语法**

**11.多个聚合函数情况下，不支持having条件**

**限制支持项：**

**1.ORDER BY的限制**

  针对全局表没限制；针对分片表，排序字段不超过8个列，ORDER BY需要使用列名或者别名，目前暂且不支持使用数字，ORDER BY目前不支持字段为枚举类型的排序。

**2.DISTINCT的限制**

  针对全局表没有限制；针对分片表，仅支持DISCTINCT字段同时也是ORDER BY字段，例如：select distinct col1 from tab1 order by col1，另外为了在使用上更加友好，对于order by未写全的，Cetus会进行补充，例如：selectdistinct col1,col2 from tab1 order by col1，Cetus会改写为 select distinct col1,col2 fromtab1 order by col1,col2，但如果写成select distinct * from tab1，Cetus则会返回错误，为了效率，提倡使用标准写法，以免造成不必要的资源开销。

**3.CASE WHEN/IF 的限制**

  全局表没有限制；针对分片表，不能用于DML语句中，也不能用在GROUP BY后，可以用于SELECT 后，也可以作为过滤条件。

**4.分页查询的限制**

  由于对分片的支持，我们带来的新限制。在结果集大于特定值时分页时，由于性能开销较大，可能无法返回准确值。

**5.JOIN的使用限制**

  不支持跨库的JOIN，非分片表可以在每个分片中都保存一份，以提高join的使用成功率。

**6.Where条件的限制**

   当Where条件中有分区列时，值不能有函数转换，也不能有算术表达式，必须是原子值，否则处理结果会不准确或者强制走全库查询，增加后端数据库的负担。

   目前支持有限的子查询类型以及有限的操作类型：支持子查询作为查询条件使用；支持子查询作为数据源使用。

**7.查询业务的限制**

   在做SQL查询时，应注意以下约束：只支持同一个 VDB 内的关联查询；针对 sharding 表，在查询条件中可以使用 sharding key 的要求加上该过滤条件，
   另外，使用 sharding key 时，不建议使用带有函数转换、算术表达式等逻辑处理, 会严重影响效率。

**8.PREPARE的限制**

  不支持服务器端 PREPARE,可以用客户端的 PREPARE 代替。

**9.中文列名的限制**

  对表列的中文列名或别名的使用有限制，使用中文列名或中文别名时必须加引号｀｀。

##### 12.事务处理限制 #####

跨库事务的有限支持，针对同一分区键分布的事务，我们默认通过分布式事务方式执行，如果需要考虑性能，可以考虑在所有数据操作都在同一分区时，手动通过注释走单机事务提交。 

不跨分区的事务需要在第一条语句中引用分区键，方便Cetus进行SQL转发和路由，并使用单机事务提升效率。

在分布式事务里面，要求用户尽量不要嵌入 select，因为 select 是会加锁的，会导致性能非常差。

能用单事务就不要走分布式事务，所谓单事务就是此事务确定只会根据分片键定位到一个后端节点。

DML语句的限制：Update/Delete 支持子查询，但子查询中不要有嵌套（仅针对分片表）；不支持对 sharding key 列进行 update（仅针对分片表）；Insert 使用时要写全列名，例如：insert into a(col1,col2) values(xx,xx)；Insert 不支持子查询,如有特殊业务需要用到,可以使用注释（仅针对分片表，详见注释功能）；Insert 支持多 value 语句，例如：insert into a(col1,col2) values(x,x),(xx,xx)；支持 replace into/insert on duplicate key 语法。


##### 13.分区键的类型 #####

用于分区的列，可以是“int”或“char”类型，“int”对应到MySQL中各种整数类型，“char”对应到各种定长和变长字符串类型，日期类型在SQL中按字符串处理的话可以支持。 后续会支持特定格式的字符型表示的时间类型,如“YYYY－MM－DD” 和 “YYYY－MM－DD HH24:MI:SS” 格式，时间格式不支持针对时区进行转换，统一使用本地时间。

##### 14.分区相关注释 #####

Cetus提供注释功能，用以解决日常维护时的需求（DBA同学经常使用）和一些针对前端业务的特殊需求（例如，强制该SQL只通过主库或者从库进行操作）。

注释书写样式 /\*# key=value \*/。

其中，以“/\*#”号开头（“/\*” 与“#”之间不允许有空格），“\*/”结尾， 中间以键值对形式书写，如果value包含[a-zA-Z0-9_-.]以外的其它特殊字符，需加双引号 。Key/value的值大小写均可，建议统一小写。

Sharding版支持的key类型：table|group|mode|transaction，支持的value包括all/readwrite/readonly/single_node。

使用示例如下：

**1.Key类型为table的用法**

  用法：/\*#table=employee\*/

  SQL: select /\*# table=employee key=123\*/emp_no,emp_name from employee;

  说明：查询表employee中分区键的值是123的记录。

**2.Key类型为group的用法**

  用法：/\*# group=dataA\*/

  SQL: select /\*# group=dataA \*/ count(\*) from employee;

  说明：查询后端节点dataA中，表employee的记录数。

  用法：/\*# group=all \*/

  SQL: create /\*# group=all \*/ table employee xxxx;

  说明：在后端所有节点均创建此表。

**3.Key类型为mode的用法**

  用法：/\*# mode=readwrite \*/

  SQL: select /\*# mode=readwrite \*/ count(\*) fromemployee;

  说明：此查询语句强制选择主库执行，查询操作默认选择从库执行。

**4.Key类型为transaction的用法**

  用法：/\*# transaction=single_node \*/

  SQL: update /\*# transaction=single_node \*/ departmentsset dept_name='ecbj' where dept_no='d010';

  说明：此dml语句将强制采用非分布式事务，一旦Cetus在执行时判断应该采用分布式事务，会返回错误。

**5.复合用法**

  用法：/\*# table=employee key=123\*/ /\*#mode=readwrite\*/

  SQL: select /\*# table=employee key=123\*/ /\*#mode=readwrite\*/ emp_no,emp_name from employee;

  说明：查询表employee中分区键的值是123的记录，且强制从主库读取。

  注意：table、group这两个key是互斥的，即table或者group分别可以和mode/transaction共用，但它俩不能同时出现，否则会返回错误。另外，请将注释部分写到第一个关键字之后。注释一旦使用，其优先级高于后续where条件（如果有的话）中的分区路由信息。

#### Cetus应用示例 ####

##### 1.连接Cetus #####

```
    $ mysql --prompt="proxy> " --comments -h**.**.**.** -P**** -u**** -p***
    proxy> 
```

在连接Cetus时，使用在配置文件中确认好的用户名和密码登陆，登陆的ip和端口为Cetus监听的ip和端口。

可同时启动监听同一个ip不同端口的Cetus。

##### 2.管理Cetus #####

```
   $  mysql --prompt="admin> " --comments -h**.**.**.** -P**** -u**** -p***
   admin>  show connectionist；
```

可以使用在配置文件中的admin用户名和密码，登陆地址为admin-address的MySQL对Cetus进行管理，例如在查询Cetus的连接详细信息时，可以登录后通过命令 show connectionlist，显示从前端到后端端口映射关系，以及涉及到的 xa 信息。

具体使用说明详见[Cetus 分库(sharding)版管理手册](https://github.com/Lede-Inc/cetus/blob/master/doc/cetus-shard-admin.md)

##### 3.查看Cetus参数 #####

```
   $  cd /home/user/cetus_install/
   bin/cetus --help | -h
```

可以查看帮助，获取Cetus启动配置选项的参数及含义。


### [十. Cetus 分库(sharding)版管理手册](https://github.com/Lede-Inc/cetus/blob/master/doc/cetus-shard-admin.md) ###


#### 前言 

**有配置修改均能动态生效，配置更改后请务必修改原始配置文件，以确保下次重启时配置能够保留。**

#### 查看帮助

`select * from help`或
`select help`

查看管理端口用法。

| Command                                  | Description                              |
| :--------------------------------------- | :--------------------------------------- |
| select conn_details from backend         | display the idle conns                   |
| select * from backends                   | list the backends and their state        |
| select * from groups                     | list the backends and their groups       |
| show connectionlist [\<num>]             | show \<num> connections                  |
| show allow_ip \<module>                  | show allow_ip rules of module, currently admin or proxy or shard |
| show deny_ip \<module>                   | show deny_ip rules of module, currently admin or proxy or shard |
| add allow_ip \<module> \<address>        | add address to white list of module      |
| add deny_ip \<module> \<address>         | add address to black list of module      |
| delete allow_ip \<module> \<address>     | delete address from white list of module |
| delete deny_ip \<module> \<address>      | delete address from black list of module |
| set reduce_conns (true or false)           | reduce idle connections if set to true   |
| reduce memory                            | reduce memory occupied by system         |
| set maintain (true or false)               | close all client connections if set to true |
| reload shard                             | reload sharding config from remote db    |
| show status [like '%\<pattern>%']        | show select/update/insert/delete statistics |
| show variables [like '%\<pattern>%']     | show configuration variables             |
| select version                           | cetus version                            |
| select conn_num from backends where backend_ndx=\<index> and user='\<name>') | display selected backend and its connection number |
| select * from user_pwd [where user='\<name>'] | display server username and password |
| select * from app_user_pwd [where user='\<name>'] |  display client username and password |
| update user_pwd set password='xx' where user='\<name>' |  update server username and password |
| update app_user_pwd set password='xx' where user='\<name>' | update client username and password |
| delete from user_pwd where user='\<name>' | delete server username and password      |
| delete from app_user_pwd where user='\<name>' | delete client username and password  |
| insert into backends values ('\<ip:port@group>', '(ro or rw)', '\<state>') | add mysql instance to backends list |
| update backends set (type or state)='\<value>' where (backend_ndx=\<index> or address='\<ip:port>') | update mysql instance type or state |
| delete from backends where (backend_ndx=\<index> or address='\<ip:port>') | set state of mysql instance to deleted |
| remove backend where (backend_ndx=\<index> or address='\<ip:port>') | set state of mysql instance to deleted |
| add master '\<ip:port@group>'            | add master                               |
| add slave '\<ip:port@group>'             | add slave                                |
| stats get [\<item>]                      | show query statistics                    |
| config get [\<item>]                     | show config                              |
| config set \<key>=\<value>               | set config                               |
| stats reset                              | reset query statistics                   |
| save settings                            | not implemented                          |
| select * from help                       | show this help                           |
| select help                              | show this help                           |
| cetus                                    | Show overall status of Cetus             |

结果说明：

sharding版本管理端口提供了38条语句对cetus进行管理，具体用法见以下说明。

#### 后端配置

##### 查看后端

`select * from backends`

查看后端信息。

| backend_ndx | address        | state | type | slave delay | uuid | idle_conns | used_conns | total_conns | group  |
| :---------- | :------------- | :---- | :--- | :---------- | :--- | :--------- | :--------- | :---------- | :----- |
| 1           | 127.0.0.1:3306 | up    | rw   | NULL        | NULL | 100        | 0          | 100         | group1 |
| 2           | 127.0.0.1:3307 | up    | rw   | NULL        | NULL | 100        | 0          | 100         | group2 |
| 3           | 127.0.0.1:3308 | up    | rw   | NULL        | NULL | 100        | 0          | 100         | group3 |
| 4           | 127.0.0.1:3309 | up    | rw   | NULL        | NULL | 100        | 0          | 100         | group4 |

结果说明：

* backend_ndx: 后端序号，按照添加顺序排列；
* address: 后端地址，IP:PORT格式；
* state: 后端状态(unknown|up|down|maintaining|deleted)；
* type: 读写类型(rw|ro)；
* slave delay: 主从延迟时间(单位：毫秒)；
* uuid: 暂时无用；
* idle_conns: 空闲连接数；
* used_conns: 正在使用的连接数；
* total_conns: 总连接数；
* group: 后端分组。


状态说明

	unknown:     后端初始状态，还未建立连接；
	up:          能与后端正常建立连接；
	down:        与后端无法联通(如果开启后端状态检测，能连通后自动变为UP)；
	maintaining: 后端正在维护，无法建立连接或自动切换状态(此状态由管理员手动设置)；
	deleted:      后端已被删除，无法再建立连接。


##### 查看后端连接状态

`select conn_details from backends`

查看每个用户占用和空闲的后端连接数。

| backend_ndx | username | idle_conns | used_used_conns | total_used_conns |
| :---------- | :------- | :--------- | :-------------- | ---------------- |
| 1           | test1    | 2          | 0               | 0                |
| 2           | test2    | 11         | 0               | 0                |

结果说明：

* backend_ndx: 后端序号；
* username: 用户名；
* idle_conns: 空闲连接数；
* used_used_conns：正在使用的连接数。
* total_used_conns: 总的连接数。

##### 查看后端分组情况

`select * from groups`

查看后端分组的详细信息。

| group | master         | slaves         |
| :---- | :------------- | :------------- |
| data1 | 127.0.0.1:3306 | 127.0.0.1:3316 |
| data2 | 127.0.0.1:3307 | 127.0.0.1:3317 |
| data3 | 127.0.0.1:3308 | 127.0.0.1:3318 |
| data4 | 127.0.0.1:3309 | 127.0.0.1:3319 |

结果说明：

* group: 后端分组序号；
* master: 读写后端；
* slaves: 只读后端。

##### 添加后端

`add master '<ip:port@group>'`

添加一个读写类型的后端。

例如

>add master '127.0.0.1:3307@group1'

`add slave '<ip:port@group>'`

添加一个只读类型的后端。

例如

>add slave '127.0.0.1:3306@group1'

`insert into backends values ('<ip:port@group>', '(ro|rw)', '<state>')`

添加一个后端，同时指定读写类型。

例如

>insert into backends values ('127.0.0.1:3306@group1', 'rw', 'up');

##### 删除后端

`remove backend <backend_ndx>` 或
`delete from backends where backend_ndx = <backend_ndx>`

删除一个指定序号的后端。

例如

>remove backend 1

`delete from backends where address = '<ip:port>'`

删除一个指定地址的后端。

例如

>delete from backends where address = '127.0.0.1:3306'

##### 修改后端

`update backends se (type|state)='<value>' where (backend_ndx=<index>|address='<ip:port>')`

修改后端类型或状态。

例如

>update backends set type='rw' where address='127.0.0.1:3306'

>update backends set state='up' where backend_ndx=1


说明

	update后端的state只包括up|down|maintaining三种状态，delete/remove后端可将后端的state设为deleted状态。


#### 基本配置

##### 查看连接池/通用配置

	config get [<item>]
	
	config get #查看支持的配置类型
	   * pool #连接池配置
	   * common #通用配置
	
	config get common #查看通用配置
	   * common.check_slave_delay #是否需要检测从库延迟
	   * common.slave_delay_down_threshold_sec #若延迟大于此值(秒)，后端状态置为DOWN
	   * common.slave_delay_recover_threshold_sec #若延迟小于此值(秒)，后端状态置为UP
	
	config get pool #查看连接池配置
	   * pool.default_pool_size #默认连接池大小
	   * pool.max_pool_size #最大连接数量
	   * pool.max_resp_len #最大结果集长度
	   * pool.master_preferred #是否只允许走主库

##### 修改连接池/通用配置

`config set [<item>]`

`config set common.[option] = [value]`  修改基本配置

例如

>config set common.slave_delay_down = 3

`config set pool.[option] = [value]`  修改连接池配置

例如

>config set pool.max_pool_size = 200

### 查看参数配置

`show variables [like '%<pattern>%']`

查看的参数均为启动配置选项中的参数，详见[Cetus 启动配置选项说明](https://github.com/Lede-Inc/cetus/blob/master/doc/cetus-configuration.md)。

## 查看/设置连接信息

### 查看当前连接的详细信息

`show connectionlist`

将当前全部连接的详细内容按表格显示出来。

| User  | Host           | db   | Command | Time | Trans | PS   | State      | Xa   | Xid  | Server | Info |
| ----- | -------------- | ---- | ------- | ---- | ----- | ---- | ---------- | ---- | ---- | ------ | ---- |
| test1 | 127.0.0.1:3306 | test | Sleep   | 0    | N     | N    | READ_QUERY | NX   | NULL | NULL   | NULL |
| test2 | 127.0.0.1:3307 | test | Sleep   | 0    | N     | N    | READ_QUERY | NX   | NULL | NULL   | NULL |

结果说明：

* User: 用户名;
* Host: 客户端的IP和端口;
* db: 数据库名称;
* Command: 执行的sql，"Sleep"代表当前空闲;
* Time: 已执行的时间;
* Trans: 是否在事务中（Y｜N）;
* PS：是否存在prepare（Y｜N）;
* State: 连接当前的状态，"READ_QUERY"代表在等待获取命令;
* Xa：分布式事务状态（NX|XS|XQ|XE|XP|XC|XR|XCO|XO）;
* Xid：分布式事务的xid;
* Server: 后端地址;
* Info: 暂未知。


Xa状态说明

	NX:     未处于分布式事务状态中；
	XS:     处于XA START状态；
	XQ:     处于XA QUERY状态；
	XE:     处于XA END状态；
	XP:     处于XA PREPARE状态；
	XC:     处于XA COMMIT状态；
	XR:     处于XA ROLLBACK状态；
	XCO:    处于XA CANDIDATE OVER状态；
	XO:     处于XA OVER状态。


##### 查看某用户对某后端的连接数

`select conn_num from backends where backend_ndx=<index> and user='<name>')`

例如

>select conn_num from backends where backend_ndx=2 and user='root');

##### 设置是否减少空闲连接

`set reduce_conns (true|false)`

例如

>set reduce_conns true;

减少空闲连接。

##### 设置是否关闭所有客户端连接

`set maintain (true|false)`

例如

>set maintain true;

关闭所有客户端连接。

#### 用户/密码管理

##### 密码查询

`select * from user_pwd [where user='<name>']`

查询某个用户的后端密码。

**注意由于密码是非明文的，仅能显示字节码。**

>select * from user_pwd where user='root';

`select * from app_user_pwd [where user='<name>']`

查询某个用户连接proxy的密码，同样是非明文。

例如

>select * from app_user_pwd where user='test';

##### 密码添加/修改

`update user_pwd set password='<password>' where user='<name>'`

添加或修改特定用户的后端密码(如果该用户不存在则添加，已存在则覆盖)。

例如

>update user_pwd set password='123456' where user='test'

`update app_user_pwd set password='<password>' where user='<name>'`

添加或修改特定用户连接Proxy的密码(如果该用户不存在则添加，已存在则覆盖)。

例如

>update app_user_pwd set password='123456' where user='root'

##### 密码删除

`delete from user_pwd where user='<name>'`

删除特定用户的后端密码。

例如

>delete from user_pwd where user='root'

`delete from app_user_pwd where user='<name>'`

删除特定用户连接Proxy的密码。

例如

>delete from app_user_pwd where user='root'

#### IP白名单

##### 查看IP白名单

`show allow_ip <module>`

\<module\>：admin|shard

查看admin／shard模块的IP白名单。

若列表为空，则代表没有任何限制。

##### 增加IP白名单

`add allow_ip <module> <address>`

向白名单增加一个IP许可。(IP不要加引号)

\<module\>：admin|shard

\<address\>：[[user@]IP]

```
说明
Admin: 仅能配置IP，不能限制用户(Admin有效用户只有一个)；
Shard: 仅配置IP，代表允许该IP来源所有用户的访问；配置User@IP，代表允许该IP来源的特定用户访问。
```

例如

>add allow_ip admin 127.0.0.1

>add allow_ip shard test@127.0.0.1

##### 删除IP白名单

`delete allow_ip <module> <address>`

删除白名单中的一个IP许可。(IP不要加引号)

\<module\>：admin|shard

\<address\>：[[user@]IP]

例如

>delete allow_ip admin 127.0.0.1

>delete allow_ip shard test@127.0.0.1

#### IP黑名单

##### 查看IP黑名单

`show deny_ip <module>`

\<module\>：admin|shard

查看admin／shard模块的IP黑名单。

若列表为空，则代表没有任何限制。

##### 增加IP黑名单

`add deny_ip <module> <address>`

向黑名单增加一个IP限制。(IP不要加引号)

\<module\>：admin|shard

\<address\>：[[user@]IP]

```
说明
Admin: 仅能配置IP，不能限制用户(Admin有效用户只有一个)；
Shard: 仅配置IP，代表限制该IP来源所有用户的访问；配置User@IP，代表限制该IP来源的特定用户访问。
```

例如

>add deny_ip admin 127.0.0.1

>add deny_ip shard test@127.0.0.1

##### 删除IP黑名单

`delete deny_ip <module> <address>`

删除黑名单中的一个IP限制。(IP不要加引号)

\<module\>：admin|shard

\<address\>：[[user@]IP]

例如

>delete deny_ip admin 127.0.0.1

>delete deny_ip shard test@127.0.0.1

**注意：IP白名单的优先级高于IP黑名单**

#### 远程管理

##### 重载分库配置

`reload shard`

需要"remote-conf-url ＝ \<url>"和"disable-threads = false"启动选项。
从远端配置库中重载Shard配置。

##### 保存配置到本地文件

`save settings [FILE]`

保存当前配置到指定路径的本地文件中。

例如

>save settings /tmp/shard.cnf

#### 查看整体信息

##### 查看统计信息
	
	stats get [<item>]
	
	stats get #查看支持的统计类型
	   * client_query  #客户发来的SQL数量
	   * proxyed_query  #发往后端的SQL数量
	   * query_time_table #查询时间直方图
	   * server_query_details  #每个后端接收的SQL数量
	   * query_wait_table #等待时间直方图
	
	stats get client_query #stats get proxyed_query`查看读/写SQL数量
	
	stats get server_query_details #查看各个后端读/写SQL数量
	
	stats get query_time_table 

	stats get query_wait_table #查看各时间值对应的SQL数量，如：

| name               | value |
| :----------------- | :---- |
| query_time_table.1 | 3     |
| query_time_table.2 | 5     |
| query_time_table.5 | 1     |

表示用时1秒的SQL有3条，用时2秒的SQL有5条，用时5秒的SQL有1条

```
说明
stats reset：重置统计信息 
```

##### 查看总体状态

`cetus`

包括程序版本、连接数量、QPS、TPS等信息

##### 查看各类SQL统计

	show status [like '%<pattern>%']
	
	
	pattern参数说明
	Com_select         总的SELECT数量
	Com_insert         总的INSERT数量
	Com_update         总的UPDATE数量
	Com_delete         总的DELETE数量
	Com_select_shard   走多个节点的SELECT数量
	Com_insert_shard   走多个节点的INSERT数量
	Com_update_shard   走多个节点的UPDATE数量
	Com_delete_shard   走多个节点的DELETE数量
	Com_select_gobal   仅涉及公共表的SELECT数量
	Com_select_bad_key 分库键未识别导致走全库的SELECT数量

### 查看当前cetus版本

`select version`

#### 其他

##### 减少系统占用的内存

`reduce memory`

## Cetus架构与设计 ##

### [Cetus 架构和实现](https://github.com/Lede-Inc/cetus/blob/master/doc/cetus-architecture.md) ###


#### 1.整体架构

Cetus 网络架构图如下所示：

![](https://i.imgur.com/xUd5XDQ.png)

Cetus位于应用程序与MySQL数据库之间，作为前端应用与数据库的通讯。其中，前端应用连接LVS节点，LVS节点映射端口到多个Cetus服务，后者通过自身的连接池连接到后端的数据库。

#### 2.功能实现

##### 1. 功能模块

Cetus 主要的功能模块包括以下五个部分：

1.读写分离

2.分库

3.SQL解析

4.连接池

5.管理功能

功能模块间的交互关系如下：

![](https://i.imgur.com/AMmFUPA.png)

其中，SQL解析模块为后续读写分离和数据分片等功能解析出SQL类型、表名和查询条件等关键信息；连接池模块是自维护连接池，支持Cetus根据需求查询和检测后端，维护连接数，具有高效连接共享性、事务与Prepare的前后端绑定功能和热点连接重用与连接等待机制；管理功能模块通过用户在管理界面输入，独立认证并转到下一状态，给用户回复状态查询结果或调整参数。


##### 2. 工作流程

Cetus 整体工作流程图如下：

![](https://i.imgur.com/NGCh1hU.png)

其整体工作流程如下所述：

1.Cetus读取启动配置文件和其他配置并启动，监听客户端请求；

2.收到客户端新建连接请求后，Cetus经过用户鉴权和连接池判断连接数是否达到上限，确定是否新建连接；

3.连接建立和认证通过后，Cetus接收客户端发送来的SQL语句，并进行词法和语义分析，对SQL语句进行解析，分析SQL的请求类型，必要时改写SQL，然后选取相应的DB并转发；

4.等待后端处理查询，接收处理查询结果集，进行合并和修改，然后转发给客户端；

5.如收到客户端关闭连接的请求，Cetus判断是否需要关闭后端连接，关闭连接。


## Cetus发现的MySQL xa事务问题 ##

### [MySQL xa事务问题说明](https://github.com/Lede-Inc/cetus/blob/master/doc/mysql-xa-bug.md) ###


#### 发现的问题
cetus测试过程中，发现MySQL的xa 事务有bug，MySQL版本为5.7.21。测试方法为：开启一个模拟银行账户转账的测试任务，随机kill -9 杀掉后端MySQL实例，并重启实例。发现有两种悬挂事务的情况：

#### 问题的现象
##### 1、只有主库存在悬挂事务

MySQL主库已接受到xa commit通知，xa commit未完成前，kill -9 杀掉MySQL主库,再启动MySQL主库，主库出现悬挂事务，而从库该分布式事务已提交。主库此时需要执行xa commit语句，提交分布式事务，这个操作同步到从库后，会导致从库sql应用进程报错，提示找不到该分布式事务。

##### 2、只有从库出现悬挂事务

cetus向后端分片发送xa prepare，分片MySQL主库接收到xa prepare，xa prepare未完成前，kill -9 杀掉MySQL主库，再启动MySQL主库，用xa事务已回滚，主库未出现悬挂事务；从库出现悬挂事务。这种情况下，从库需要回滚xa事务，才能保证数据的一致性。

以上两种情况，主库的xa事务状态，跟binlog记录的事务状态不一致。在MySQL官方文档找到解释，MySQL异常关闭，有可能导致数据库状态和binlog不一致。这些bug，在非正常关闭MySQL时才出现，正常关闭mysql不会出现这个问题。如果出现xa事务悬挂，可以用[Cetus xa悬挂处理工具](https://github.com/Lede-Inc/cetus/blob/master/doc/cetus-xa.md)自动处理。
      
#### 官方的描述

XA is not fully resilient to an unexpected halt with respect to the binary log (on the master). If there is an unexpected halt before XA PREPARE, between XA PREPARE and XA COMMIT (or XA ROLLBACK), or after XA COMMIT (or XA ROLLBACK), the server and binary log are correctly recovered and taken to a consistent state. However, if there is an unexpected halt in the middle of the execution of one of these statements, the server may not be able to recover to a correct state, leaving the server and the binary log in an inconsistent state.
已提交至官方的xa bug：
XA prepare is logged ahead of engine prepare
https://bugs.mysql.com/bug.php?id=76233

#### 详细的验证

我们对MySQL 5.7.21 XA bug的详细验证主要包括从外部看到的现象和代码层分析两部分：

[MySQL 5.7.21 XA bug 外部现象](https://github.com/Lede-Inc/cetus/blob/master/doc/MySQL-5.7.21-XA-bug-phenomena.pdf)

[MySQL 5.7.21 XA bug 代码层分析](https://github.com/Lede-Inc/cetus/blob/master/doc/MySQL-5.7.21-XA-bug-code-analysis.pdf)


## Cetus辅助 ##

### [一. Cetus xa悬挂处理工具](https://github.com/Lede-Inc/cetus/blob/master/doc/cetus-xa.md) ###


####  简介

中间件Cetus处理分布式事务可能会由于网络、节点错误而中断，导致xa事务悬挂。Cetus xa悬挂处理工具是由python语言开发的，主要是在Cetus遇到分布式事务悬挂时自动处理悬挂事务的修复工具。

#### 原理

该工具主要包括悬挂事务查找模块和悬挂事务处理模块。其中悬挂事务查找模块是通过读取MySQL中xa recover的结果，获取长时间处于悬挂的事务xid列表，将所有后端的xa悬挂事务对应的xid汇总并去重，再读取后端binlog日志的内容获得所有后端xa悬挂事务的xid对应的最终状态；悬挂事务处理模块主要是根据悬挂事务查找模块获取的最终状态，对悬挂事务进行简单的处理，即当悬挂事务的最终状态为PREPARE、ROLLBACK、END或START时进行回滚操作，当悬挂事务的最终状态为COMMIT时进行提交操作。

#### 安装启动步骤

##### 安装依赖

- python

- python需要的模块Pool

- python需要的模块MySQLdb

请确保在使用Cetus xa悬挂处理工具前已安装好相应的依赖。

##### 启动步骤

- 配置：将xa悬挂处理工具记录的日志路径、Cetus的启动配置文件路径、Cetus的用户设置文件和临时文件的存放路径等信息填入xa悬挂处理工具前部的CONFIG中，如下：

```
CONFIG = {"logs": "/data/cetus/xa_suspension_logs/xa-suspension.log",
          "backend": "/home/mysql-cetus/cetus_install/conf/cetus.conf",
          "user": "/home/mysql-cetus/cetus_install/conf/users.json",
          "temp_file": "/data/cetus/xa_suspension_logs/"
         }
```

- 启动：将xa悬挂处理工具设置为可执行文件，并在后台运行，指令如下：

```
chmod +x xa-suspension.py
nohup ./xa-suspension.py &
```

#### 注意事项

- 由于该工具主要是结合Cetus软件处理xa悬挂事务的，因此请确保使用该工具前已运行Cetus。
- 由于该工具主要是针对当天的悬挂事务进行处理，若需要在开启Cetus软件的同时处理悬挂事务，请确保及时开启该工具。


### [二. Cetus + mha高可用方案](https://github.com/Lede-Inc/cetus/blob/master/doc/cetus-mha.md) ###

#### 简介
cetus使用改进过的mha实现高可用。乐得DBA在原生mha中，加入修改cetus状态的模块和mha操作过程的短信、邮件通知功能。修改过的mha版本简称mha_ld。

mha切换包括故障切换和在线手工切换，以故障切换(failover)为例，简单介绍一下mha_ld的工作流程，未提及的操作请参考官方文档：
https://github.com/yoshinorim/mha4mysql-manager/wiki

![](https://i.imgur.com/fuaty6l.jpg)

1、检测当前主库不达，准备开始切换

2、在cetus管理端修改当前主库的状态，将当前主库状态修改为“维护(maintaining)”，类型修改为“只读(ro)”，发送修改的通知短信

update backends set state='maintaining' , type='ro' where address='172.0.0.1:3306';

3、提升MySQL从库为新的主库

4、在cetus管理端修改新主库的状态，将当前主库状态修改为“维护(unknown)”，类型修改为“只读(rw)”，发送修改的通知短信

update backends set state='unknown' , type='rw' where address='172.0.0.2:3306';

#### 安装
在master和node主机节点yum安装rpm包

yum install -y  perl-DBD-MySQL perl-Config-Tiny perl-Log-Dispatch perl-Parallel-ForkManager perl-Config-IniFiles


master和node主机节点，安装mha4mysql-node-0.56-0.el6.noarch.rpm包

rpm -ivh mha4mysql-node-0.56-0.el6.noarch.rpm 

master主机节点，安装mha4mysql-manager-0.56-0.el6.noarch.rpm包

rpm -ivh mha4mysql-manager-0.56-0.el6.noarch.rpm

使用 mha_ld/src 替换所有文件/usr/share/perl5/vendor_perl/MHA/目录的所有同名文件

使用 mha_ld/masterha_secondary_check替换masterha_secondary_check命令
 which masterha_secondary_check

/usr/bin/masterha_secondary_check

rm /usr/bin/masterha_secondary_check

cd /usr/bin/

上传修改后的masterha_secondary_check

chmod +x /usr/bin/masterha_secondary_check

#### 配置
安装好mha_ld后，配置启动mha的cnf文件请参考mha_ld/sample.cnf，参数部分可以参考mha githup官方文档
https://github.com/yoshinorim/mha4mysql-manager/wiki/Configuration

配置cnf后，有一个变量proxy_conf（变量需要写绝对路径），文件内容参考mha_ld/cetus.cnf：

这个文件记录的是cetus的连接信息，含义解释如下：

middle_ipport=127.0.0.1:4306,127.0.0.14:4307

middle_user=admin

middle_pass=xxxxxxx

middle_ipport记录多组cetus。一组cetus由ip和端口组成，ip和端口用英文冒号分隔；每组cetus用英文逗号分隔。

middle_user记录每组cetus管理入口的用户名，与cetus配置文件中的admin-username变量值一致。同一个mha_ld维护的多组cetus，管理入口用户名必须一致。

middle_pass记录每组cetus管理入口的用户密码，与cetus配置文件中的admin-password变量值一致。同一个mha_ld维护的多组cetus，管理入口用户密码必须一致。



#### 修改切换时的通知

在/usr/share/perl5/vendor_perl/MHA/ManagerConst.pm文件中，有一个变量MOBILE_PHONES，更改该变量，可以将需要收到短信通知的人加入列表中，例如：

our @MOBILE_PHONES = (

  1234567890,  # zhang

  2345678901,  # wang

);

另外，还有几个文件涉及到发送短信时使用的url，分别为HealthCheck.pm/MasterFailover.pm/MasterMonitor.pm/ProxyManager.pm，找到这些文件中send_alert函数，修改 curl调用即可。


### [三. Cetus rpm说明](https://github.com/Lede-Inc/cetus/blob/master/doc/cetus-rpm.md) ###


####  简介

Cetus rpm打包流程及安装

#### 打包流程

##### 1. 创建打包环境

```
mkdir rpmbuild/
cd rpmbuild/ 
mkdir -pv {BUILD,BUILDROOT,RPMS,SOURCES,SPECS,SRPMS} 
```

##### 2. 下载源码，压缩成tar.gz格式，放入SOURCES中

```
git clone https://github.com/Lede-Inc/cetus.git
tar zcvf cetus-version.tar.gz cetus/
```

##### 3. 将cetus的原spec文件放入SPECS中,编辑sepc文件，修改版本号和释出号等信息

```
#版本号,与tar.gz包的一致
Version:        version
#释出号，也就是第几次制作rpm
Release:        release%{?dist}
```

##### 4.增加日志段

```
%changelog
* Week month day year packager<email> - cetus-version-release
- do something
```

#### 安装

安装命令如下：

```
rpm -ivh --prefix /home/user/cetus_install cetus-version-release.el7.x86_64.rpm
```


## Cetus测试 ##

### [Cetus 测试报告](https://github.com/Lede-Inc/cetus/blob/master/doc/cetus-test.md) ###


####  简介

Cetus 测试报告主要是Cetus的性能测试，针对单Cetus、集群部署Cetus与原生mysql的性能对比情况，出于简单性能对比的目的，我们也对友商的oneproxy和mycat进行了部署和简单测试。

#### 测试报告

##### 1. [Cetus性能测试报告20160901](https://github.com/Lede-Inc/cetus/blob/master/doc/cetus-test-20160901.pdf)

##### 2. [Cetus性能测试报告20170525](https://github.com/Lede-Inc/cetus/blob/master/doc/cetus-test-20170525.pdf)

**Cetus测试报告持续更新中**