---
layout: post
title: efk
categories: efk
description: efk
keywords: efk
---

# 利用EFK（Elasticsearch + Kibana + Filebeat）搭建日志收集展示 #

**拓扑图**

![](https://i.imgur.com/MkJ0jRB.png)

## 一. 环境相关配置 ##

安装 java环境需要JDK1.8以上版本

下载地址：[http://www.oracle.com/technetwork/java/javase/downloads/jdk-netbeans-jsp-142931.html](http://www.oracle.com/technetwork/java/javase/downloads/jdk-netbeans-jsp-142931.html)

下载对应版本并安装

如Centos:

	rpm -ivh jdk-XX--XX.rpm

测试是否安装好了（有版本显示出来表示成功安装并添加相应的环境变量）

	java -verison
		java version "1.8.0_161"
		Java(TM) SE Runtime Environment (build 1.8.0_161-b12)
		Java HotSpot(TM) 64-Bit Server VM (build 25.161-b12, mixed mode)

更改linux相关配置

在 /etc/security/limits.conf 中添加下列配置

	vim /etc/security/limits.conf
	
		*               soft    nofile          655360
		*               hard    nofile          655360
		*               soft    nproc           655350
		*               hard    nproc           655350
		*               soft    memlock         unlimited
		*               hard    memlock         unlimited
              
在 /etc/security/limits.d/90-nproc.conf 中更改为

	vim /etc/security/limits.d/90-nproc.conf

		*          soft    nproc     655350
		root       soft    nproc     unlimited


在 /etc/sysctl.conf 中添加

	vim	/etc/sysctl.conf

		vm.max_map_count=655350

使/etc/sysctl.conf中添加的配置立即生效可用下列命令

	sysctl -p

添加一个普通用户因为elasticsearch只能用普通用户启动

	useradd elk

添加相应的文件并更改权限

	mkdir -p /data/elasticsearch
	mkdir -p /data/kibana
	mkdir -p /data/logs
	chown elk:elk /data -R

## 二 . 下载安装配置Elasticsearch ##

下载Elasticsearch

下载地址：[https://www.elastic.co/downloads/elasticsearch](https://www.elastic.co/downloads/elasticsearch)

下载对应版本(这里用tar包进行配置)

解压-重命名-更改权限
	
	tar -xf elasticsearch-6.2.4.tar.gz -C /usr/local/
	cd /usr/local
	mv elasticsearch-6.2.4 elasticsearch
	chown elk:elk elasticsearch -R
	cd elasticsearch/config

更改配置文件

	1. 将 jvm.options中的配置文件更改

		从：

			-Xms2G
			-Xmx2G
		改为：

			-Xms512m
			-Xmx512m
	
	2. 更改elasticsearch主配置文件elasticsearch.yml
		
		cluster.name: my-application

		node.name: node-1

		path.data: /data/elasticsearch

		path.logs: /data/logs

		bootstrap.memory_lock: false

		bootstrap.system_call_filter: false

		network.host: 0.0.0.0

		http.port: 9200


启动

	sudo -u elk /usr/local/elasticsearch/bin/elasticsearch >> /data/logs/elasticsearch.log 2>&1 &

查看日志
	tail -f	/data/logs/elasticsearch.log

查看对应端口

	netstat -ntlup | grep 9200
		tcp        0      0 :::9200                     :::*                        LISTEN      1862/java
		tcp        0      0 :::9300                     :::*                        LISTEN      1862/java


测试

浏览器打开访问 http://192.168.1.225:9200 

出现下面的表示成功

		{
	  "name" : "node-1",
	  "cluster_name" : "my-application",
	  "cluster_uuid" : "QQOuWf-vQGyduwoP8gupww",
	  "version" : {
	    "number" : "6.2.4",
	    "build_hash" : "ccec39f",
	    "build_date" : "2018-04-12T20:37:28.497551Z",
	    "build_snapshot" : false,
	    "lucene_version" : "7.2.1",
	    "minimum_wire_compatibility_version" : "5.6.0",
	    "minimum_index_compatibility_version" : "5.0.0"
	  },
	  "tagline" : "You Know, for Search"
	}


## 三. 下载安装配置Kibana ##

下载Kibana

下载地址：[https://www.elastic.co/downloads/kibana](https://www.elastic.co/downloads/kibana)

下载对应版本(这里用tar包进行配置)

解压-重命名-更改权限
	
	tar -xf kibana-6.2.4-linux-x86_64.tar.gz -C /usr/local/
	cd /usr/local
	mv kibana-6.2.4-linux-x86_64 kibana
	chown elk:elk kibana -R
	cd kibana/config

更改kibana配置文件

	vim kibana.yml

		server.port: 5601

		server.host: "0.0.0.0"
		
		elasticsearch.url: "http://localhost:9200"

		kibana.index: ".kibana"



启动

	sudo -u elk /usr/local/kibana/bin/kibana >> /data/logs/kibana.log 2>&1 &

查看日志

	tail -f /data/logs/kibana.log

查看对应端口

	netstat -ntlup
		tcp        0      0 0.0.0.0:5601                0.0.0.0:*                   LISTEN      1969/node


浏览器打开访问 http://192.168.1.225:5601 能访问出现kibana页面表示成功


	
	
## 四. 下载安装配置Filebeat ##

下载Filebeat

下载地址：[https://www.elastic.co/downloads/beats/filebeat](https://www.elastic.co/downloads/beats/filebeat)

下载对应版本(这里用rpm 64-bit包进行配置)

安装Filebeat
	
	rpm -ivh filebeat-6.2.4-x86_64.rpm
	
备份更改配置文件（）

	cp -a /etc/filebeat/filebeat.yml /etc/filebeat/filebeat.yml.bak
	
	vim /etc/filebeat/filebeat.yml
		
		filebeat.prospectors:
		- type: log
		  enabled: true
		  paths:
		    - /logdata/*.log
		output.elasticsearch:
		  hosts: ["192.168.1.225:9200"]

注：logdata下的日志是用rsyslog采集到这台服务器上这台服务器为日志服务器rsyslog的配置请查看另一篇文档[ 利用rsyslog搭建日志集中收集配置](https://deepnum.com/2018/04/18/rsyslog-install-config/)

启动-开机自启

	/etc/ini.d/filebeat start

	chkconfig filebeat on


## 五. 添加索引 ##

当日志成功发送时去Kinbana添加索引如图

![](https://i.imgur.com/g3OG2hR.png)


**搭建完成**
