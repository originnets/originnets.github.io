---
layout: post
title: rsyslog
categories: rsyslog
description: rsyslog
keywords: rsyslog
---


# Rsyslog搭建日志集中收集中心 #

**拓补图：**

![](https://i.imgur.com/aZyVaUY.png)

## 安装配置 ##


**Server和Client都安装**

	yum -y install rsyslog 

### Server的配置 ###

备份配置文件

	cp -a /etc/rsyslog.conf /etc/rsyslog.conf.bak

 

编辑Server的配置文件


	vim /etc/rsyslog.conf
	
		$ModLoad imuxsock 
		$ModLoad imjournal
		$ModLoad imudp			#开启upd
		$UDPServerRun 514		#开启udp514端口
		
		$WorkDirectory /var/lib/rsyslog
		$ActionFileDefaultTemplate RSYSLOG_TraditionalFileFormat
		$IncludeConfig /etc/rsyslog.d/*.conf		#加载/etc/rsyslog.d/下的所有配置文件
		$OmitLocalLogging on
		$IMJournalStateFile imjournal.state
		
		*.info;mail.none;authpriv.none;cron.none                /var/log/messages
		
		authpriv.*                                              /var/log/secure
		
		mail.*                                                  -/var/log/maillog
		
		cron.*                                                  /var/log/cron
		
		*.emerg                                                 :omusrmsg:*
		
		local7.*                                                /var/log/boot.log



添加Server子配置文件

	vim /etc/rsyslog.d/synclog.conf

		if ($fromhost-ip == '192.168.1.215' ) then /var/log/hosts/192.168.1.215.log
		if ($fromhost-ip == '192.168.1.216' ) then /var/log/hosts/192.168.1.216.log
		if ($fromhost-ip == '192.168.1.217' ) then /var/log/hosts/192.168.1.217.log
		if ($fromhost-ip == '192.168.1.218' ) then /var/log/hosts/192.168.1.216.log
		#判断Client的IP，即当ip为192.168.1.215时是将该主机发送过来的日志写入到Server的/var/log/hosts/192.168.1.215.log，中其他同理

创建相关目录
	
	mkdir -p /var/log/hosts

重启服务, 添加开机启动 

Centos6.X

	/etc/init.d/rsyslog restart
	chkconfig rsyslog on

Centos7.X

	systemctl restart rsyslog
	systemctl enable rsyslog

### Client配置 ###


备份配置文件

	cp -a /etc/rsyslog.conf /etc/rsyslog.conf.bak


编辑Client的配置文件


	vim /etc/rsyslog.conf
		
		$ModLoad imuxsock
		$ModLoad imklog
		$ModLoad imfile			#添加自定义日志
		
		$ActionFileDefaultTemplate RSYSLOG_TraditionalFileFormat
		
		$IncludeConfig /etc/rsyslog.d/*.conf
		
		*.info;mail.none;authpriv.none;cron.none                /var/log/messages
		authpriv.*                                              /var/log/secure
		mail.*                                                  -/var/log/maillog
		cron.*                                                  /var/log/cron
		*.emerg                                                 *
		uucp,news.crit                                          /var/log/spooler
		local7.*                                                /var/log/boot.log
		
		$InputFileName /var/log/nginx/lemon.access.log 				#指定监控日志文件
		$InputFilePollInterval 5 									#指定每5秒轮询一次文件
		$InputFileTag nginx 										#指定文件的tag
		$InputFileStateFile /var/lib/rsyslog/lemon.access-test.log 	#指定状态文件存放位置，如不指定会报错。
		$InputFileSeverity info 									#设置监听日志级别
		$InputFileFacility local5 									#指定设备
		$InputRunFileMonitor										#激活读取，可以设置多组日志读取，每组结束时设置本参数
		
		local5.* @192.168.1.218:514  								#发送给rsyslog-server服务器


重启服务, 添加开机启动 

Centos6.X

	/etc/init.d/rsyslog restart
	chkconfig rsyslog on

Centos7.X

	systemctl restart rsyslog
	systemctl enable rsyslog


### 测试 ###


在Client上输入 (该主机的ip是192.168.1.215)
	
	logger -p local5.info "hello world"   	#local5.info 表示上面定义发送给rsyslog-server的设备
	echo "-----------test------------" >> /var/log/nginx/lemon.access.log

此时应该在rsyslog-server中生成 /var/log/host/192.1618.1.215.log 该文件

查看该文件
 
	cat /var/log/host/192.1618.1.215.log
		hello world
		nginx -----------test------------

有该内容表示成功

