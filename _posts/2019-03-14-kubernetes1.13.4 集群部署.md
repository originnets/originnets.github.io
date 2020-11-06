---
layout: post
title: docker
categories: docker
description: docker
keywords: docker
---


kubernetes 集群安装

# 一、官方文档

	https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG-1.13.md#downloads-for-v1131
	https://kubernetes.io/docs/home/?path=users&persona=app-developer&level=foundational
	https://github.com/etcd-io/etcd
	https://shengbao.org/348.html
	https://github.com/coreos/flannel
	http://www.cnblogs.com/blogscc/p/10105134.html
	https://blog.csdn.net/xiegh2014/article/details/84830880
	https://blog.csdn.net/tiger435/article/details/85002337
	https://www.cnblogs.com/wjoyxt/p/9968491.html
	https://blog.csdn.net/zhaihaifei/article/details/79098564
	http://blog.51cto.com/jerrymin/1898243
	http://www.cnblogs.com/xuxinkun/p/5696031.html

# 二、相关下载链接

	Client Binaries
	https://dl.k8s.io/v1.13.4/kubernetes-client-linux-amd64.tar.gz
	Server Binaries
	https://dl.k8s.io/v1.13.4/kubernetes-server-linux-amd64.tar.gz
	Node Binaries
	https://dl.k8s.io/v1.13.4/kubernetes-node-linux-amd64.tar.gz
	etcd
	https://github.com/etcd-io/etcd/releases/download/v3.3.12/etcd-v3.3.12-linux-amd64.tar.gz
	flannel
	https://github.com/coreos/flannel/releases/download/v0.11.0/flannel-v0.11.0-linux-amd64.tar.gz


​	

- 这里采用kubernetes-v1.13.4, etcd-v3.3.12,flannel-v0.11.0 版本

# 三、角色划分(软件安装)

	master	192.168.1.193	master	etcd、kube-apiserver、kube-controller-manager、kube-scheduler
	node1	192.168.1.195	node	etcd、kubelet、docker、kube_proxy,flannel
	node2	192.168.1.196	node	etcd、kubelet、docker、kube_proxy,flannel

# 四 初始化环境

## 4.1 设置关闭防火墙及SELINUX

	systemctl stop firewalld && systemctl disable firewalld
	setenforce 0
	vi /etc/selinux/config
	SELINUX=disabled

## 4.2 关闭Swap

	swapoff -a && sysctl -w vm.swappiness=0
	vi /etc/fstab
	#UUID=7bff6243-324c-4587-b550-55dc34018ebf swap                    swap    defaults        0 0

## 4.3 设置Docker所需参数

	cat << EOF | tee /etc/sysctl.d/k8s.conf
	net.ipv4.ip_forward = 1
	net.bridge.bridge-nf-call-ip6tables = 1
	net.bridge.bridge-nf-call-iptables = 1
	EOF
	sysctl -p /etc/sysctl.d/k8s.conf

## 4.4 在node1, node2 安装docker

	yum install docker-ce -y
	systemctl start docker && systemctl enable docker

## 4.5 添加hosts

	vim /etc/hosts
	192.168.1.193   master.iotot.cn master
	192.168.1.195   node1.iotot.cn node1
	192.168.1.196   node2.iotot.cn node2

## 4.6 建立互信秘钥(每台主机执行后copy到相关主机上)

	ssh-keygen 
	Generating public/private rsa key pair.
	Enter file in which to save the key (/root/.ssh/id_rsa): 
	Created directory '/root/.ssh'.
	Enter passphrase (empty for no passphrase): 
	Enter same passphrase again: 
	Your identification has been saved in /root/.ssh/id_rsa.
	Your public key has been saved in /root/.ssh/id_rsa.pub.
	The key fingerprint is:
	SHA256:FQjjiRDp8IKGT+UDM+GbQLBzF3DqDJ+pKnMIcHGyO/o root@qas-k8s-master01
	The key's randomart image is:
	+---[RSA 2048]----+
	|o.==o o. ..      |
	|ooB+o+ o.  .     |
	|B++@o o   .      |
	|=X**o    .       |
	|o=O. .  S        |
	|..+              |
	|oo .             |
	|* .              |
	|o+E              |
	+----[SHA256]-----+
	
	# ssh-copy-id 192.168.1.195
	# ssh-copy-id 192.168.1.196

## 4.7 创建相关目录

	mkdir /k8s/etcd/{bin,cfg,ssl} -p
	mkdir /k8s/kubernetes/{bin,cfg,ssl} -p

## 4.8 添加环境变量

	vim /etc/profile
	
	PATH=/k8s/kubernetes/bin:$PATH
	PATH=/k8s/etcd/bin:$PATH
	source /etc/profile

# 五、Master部署

## 5.1 下载软件

	wget https://dl.k8s.io/v1.13.4/kubernetes-server-linux-amd64.tar.gz
	wget https://dl.k8s.io/v1.13.4/kubernetes-client-linux-amd64.tar.gz
	wget https://github.com/etcd-io/etcd/releases/download/v3.3.12/etcd-v3.3.12-linux-amd64.tar.gz
	wget https://github.com/coreos/flannel/releases/download/v0.11.0/flannel-v0.11.0-linux-amd64.tar.gz

## 5.2 cfssl 安装用于生成证书(相关证书只需要生成一次)

	wget https://pkg.cfssl.org/R1.2/cfssl_linux-amd64
	wget https://pkg.cfssl.org/R1.2/cfssljson_linux-amd64
	wget https://pkg.cfssl.org/R1.2/cfssl-certinfo_linux-amd64
	chmod +x cfssl_linux-amd64 cfssljson_linux-amd64 cfssl-certinfo_linux-amd64
	mv cfssl_linux-amd64 /usr/local/bin/cfssl
	mv cfssljson_linux-amd64 /usr/local/bin/cfssljson
	mv cfssl-certinfo_linux-amd64 /usr/bin/cfssl-certinfo

## 5.3 创建用于存证书的目录

	mkdir /root/{etcd_ssl,kubernetes_ssl}

## 5.4 创建认证证书

### 5.4.1 创建 etcd 证书

	cd /root/etcd_ssl
	
	cat << EOF | tee ca-config.json
	{
	  "signing": {
	    "default": {
	      "expiry": "87600h"
	    },
	    "profiles": {
	      "www": {
	         "expiry": "87600h",
	         "usages": [
	            "signing",
	            "key encipherment",
	            "server auth",
	            "client auth"
	        ]
	      }
	    }
	  }
	}
	EOF

### 5.4.2 创建 etcd ca 配置文件

	cat << EOF | tee ca-csr.json
	{
	    "CN": "etcd CA",
	    "key": {
	        "algo": "rsa",
	        "size": 2048
	    },
	    "names": [
	        {
	            "C": "CN",
	            "L": "Shenzhen",
	            "ST": "Shenzhen"
	        }
	    ]
	}
	EOF

### 5.4.3 创建 etcd server 证书

	cat << EOF | tee server-csr.json
	{
	    "CN": "etcd",
	    "hosts": [
	    "192.168.1.193",
	    "192.168.1.195",
	    "192.168.1.196"
	    ],
	    "key": {
	        "algo": "rsa",
	        "size": 2048
	    },
	    "names": [
	        {
	            "C": "CN",
	            "L": "Shenzhen",
	            "ST": "Shenzhen"
	        }
	    ]
	}
	EOF

### 5.4.4 生成 etcd ca 证书和私钥

	cfssl gencert -initca ca-csr.json | cfssljson -bare ca -
	cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=www server-csr.json | cfssljson -bare server

### 5.4.5 创建 Kubernetes 证书

	cd /root/kubernetes_ssl
	
	cat << EOF | tee ca-config.json
	{
	  "signing": {
	    "default": {
	      "expiry": "87600h"
	    },
	    "profiles": {
	      "kubernetes": {
	         "expiry": "87600h",
	         "usages": [
	            "signing",
	            "key encipherment",
	            "server auth",
	            "client auth"
	        ]
	      }
	    }
	  }
	}
	EOF

### 5.4.6 创建 Kubernetes CA 证书

	cat << EOF | tee ca-csr.json
	{
	    "CN": "kubernetes",
	    "key": {
	        "algo": "rsa",
	        "size": 2048
	    },
	    "names": [
	        {
	            "C": "CN",
	            "L": "Shenzhen",
	            "ST": "Shenzhen",
	            "O": "k8s",
	            "OU": "System"
	        }
	    ]
	}
	EOF

### 5.4.7 生成Kubernetes CA 证书

	cfssl gencert -initca ca-csr.json | cfssljson -bare ca -

### 5.4.8 创建API_SERVER 证书

	cat << EOF | tee server-csr.json
	{
	    "CN": "kubernetes",
	    "hosts": [
	      "10.0.0.1",
	      "127.0.0.1",
	      "192.168.1.193",
	      "kubernetes",
	      "kubernetes.default",
	      "kubernetes.default.svc",
	      "kubernetes.default.svc.cluster",
	      "kubernetes.default.svc.cluster.local"
	    ],
	    "key": {
	        "algo": "rsa",
	        "size": 2048
	    },
	    "names": [
	        {
	            "C": "CN",
	            "L": "Shenzhen",
	            "ST": "Shenzhen",
	            "O": "k8s",
	            "OU": "System"
	        }
	    ]
	}
	EOF

### 5.4.9 生成API_SERVER证书

	cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes server-csr.json | cfssljson -bare server

### 5.4.10 创建 Kubernetes Proxy 证书

	cat << EOF | tee kube-proxy-csr.json
	{
	  "CN": "system:kube-proxy",
	  "hosts": [],
	  "key": {
	    "algo": "rsa",
	    "size": 2048
	  },
	  "names": [
	    {
	      "C": "CN",
	      "L": "Shenzhen",
	      "ST": "Shenzhen",
	      "O": "k8s",
	      "OU": "System"
	    }
	  ]
	}
	EOF

### 5.4.11 生成 Kubernetes Proxy 证书

	cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes kube-proxy-csr.json | cfssljson -bare kube-proxy


## 5.5 部署etcd(所有节点都要安装)

### 5.5.1 解压安装文件

	tar -xf etcd-v3.3.12-linux-amd64.tar.gz
	cd etcd-v3.3.12-linux-amd64
	cp etc etcdctl /k8s/etcd/bin

### 5.5.2 编辑配置文件

	vim /k8s/etcd/cfg/etcd
	
	#[Member]
	ETCD_NAME="etcd01"
	ETCD_DATA_DIR="/var/lib/etcd/default.etcd"
	ETCD_LISTEN_PEER_URLS="https://192.168.1.193:2380"
	ETCD_LISTEN_CLIENT_URLS="https://192.168.1.193:2379"
	
	#[Clustering]
	ETCD_INITIAL_ADVERTISE_PEER_URLS="https://192.168.1.193:2380"
	ETCD_ADVERTISE_CLIENT_URLS="https://192.168.1.193:2379"
	ETCD_INITIAL_CLUSTER="etcd01=https://192.168.1.193:2380,etcd02=https://192.168.1.195:2380,etcd03=https://192.168.1.196:2380"
	ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster"
	ETCD_INITIAL_CLUSTER_STATE="new"

### 5.5.3 创建 etcd的 systemd unit 文件

	vim /usr/lib/systemd/system/etcd.service
	
	[Unit]
	Description=Etcd Server
	After=network.target
	After=network-online.target
	Wants=network-online.target
	
	[Service]
	Type=notify
	EnvironmentFile=/k8s/etcd/cfg/etcd
	ExecStart=/k8s/etcd/bin/etcd \
	--name=${ETCD_NAME} \
	--data-dir=${ETCD_DATA_DIR} \
	--listen-peer-urls=${ETCD_LISTEN_PEER_URLS} \
	--listen-client-urls=${ETCD_LISTEN_CLIENT_URLS},http://127.0.0.1:2379 \
	--advertise-client-urls=${ETCD_ADVERTISE_CLIENT_URLS} \
	--initial-advertise-peer-urls=${ETCD_INITIAL_ADVERTISE_PEER_URLS} \
	--initial-cluster=${ETCD_INITIAL_CLUSTER} \
	--initial-cluster-token=${ETCD_INITIAL_CLUSTER_TOKEN} \
	--initial-cluster-state=new \
	--cert-file=/k8s/etcd/ssl/server.pem \
	--key-file=/k8s/etcd/ssl/server-key.pem \
	--peer-cert-file=/k8s/etcd/ssl/server.pem \
	--peer-key-file=/k8s/etcd/ssl/server-key.pem \
	--trusted-ca-file=/k8s/etcd/ssl/ca.pem \
	--peer-trusted-ca-file=/k8s/etcd/ssl/ca.pem
	Restart=on-failure
	LimitNOFILE=65536
	
	[Install]
	WantedBy=multi-user.target

### 5.5.4 拷贝证书

	cd /root/etcd_ssl/
	cp *pem /k8s/etcd/ssl
	
	cd /root/kubernetes_ssl/
	cp *pem /k8s/kubernetes/ssl

### 5.5.5 启动ETCD服务(当其他的etcd没有启动,主的etcd没有办法启动成功)

	systemctl daemon-reload
	systemctl enable etcd
	systemctl start etcd

### 5.5.6 将启动文件、配置文件拷贝到 节点1、节点2

	scp -r /k8s/ node1:/k8s/
	scp -r /k8s/ node2:/k8s/
	scp /usr/lib/systemd/system/etcd.service  node1:/usr/lib/systemd/system/etcd.service
	scp /usr/lib/systemd/system/etcd.service  node2:/usr/lib/systemd/system/etcd.service 

### 5.5.7 更改相应的配置文件

	node1:
	vim /k8s/etcd/cfg/etcd 
	#[Member]
	ETCD_NAME="etcd02"
	ETCD_DATA_DIR="/var/lib/etcd/default.etcd"
	ETCD_LISTEN_PEER_URLS="https://192.168.1.195:2380"
	ETCD_LISTEN_CLIENT_URLS="https://192.168.1.195:2379"
	
	#[Clustering]
	ETCD_INITIAL_ADVERTISE_PEER_URLS="https://192.168.1.195:2380"
	ETCD_ADVERTISE_CLIENT_URLS="https://192.168.1.195:2379"
	ETCD_INITIAL_CLUSTER="etcd01=https://192.168.1.193:2380,etcd02=https://192.168.1.195:2380,etcd03=https://192.168.1.196:2380"
	ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster"
	ETCD_INITIAL_CLUSTER_STATE="new"
	
	node2:
	vim /k8s/etcd/cfg/etcd
	
	#[Member]
	ETCD_NAME="etcd03"
	ETCD_DATA_DIR="/var/lib/etcd/default.etcd"
	ETCD_LISTEN_PEER_URLS="https://192.168.1.196:2380"
	ETCD_LISTEN_CLIENT_URLS="https://192.168.1.196:2379"
	
	#[Clustering]
	ETCD_INITIAL_ADVERTISE_PEER_URLS="https://192.168.1.196:2380"
	ETCD_ADVERTISE_CLIENT_URLS="https://192.168.1.196:2379"
	ETCD_INITIAL_CLUSTER="etcd01=https://192.168.1.193:2380,etcd02=https://192.168.1.195:2380,etcd03=https://192.168.1.196:2380"
	ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster"
	ETCD_INITIAL_CLUSTER_STATE="new"

### 5.5.8 启动所有etcd

	systemctl daemon-reload
	systemctl enable etcd
	systemctl start etcd

### 5.5.9 验证集群是否正常运行

	/k8s/etcd/bin/etcdctl --ca-file=/k8s/etcd/ssl/ca.pem --cert-file=/k8s/etcd/ssl/server.pem --key-file=/k8s/etcd/ssl/server-key.pem --endpoints="https://192.168.1.193:2379,https://192.168.1.195:2379,https://192.168.1.196:2379" cluster-health
	
	member 126f7f6e72e75cc2 is healthy: got healthy result from https://192.168.1.193:2379
	member 544a85d3b9dd3227 is healthy: got healthy result from https://192.168.1.195:2379
	member 6d198760d725d4c8 is healthy: got healthy result from https://192.168.1.196:2379
	cluster is healthy

## 5.6 部署kubernetes server

### 5.6.1 解压文件

	tar -zxvf kubernetes-server-linux-amd64.tar.gz 
	cd kubernetes/server/bin/
	cp kube-scheduler kube-apiserver kube-controller-manager kubectl /k8s/kubernetes/bin/

### 5.6.2 部署kube-apiserver组件 创建TLS Bootstrapping Token

	[root@master bin]# head -c 16 /dev/urandom | od -An -t x | tr -d ' '
	a1e62c2c2be66bf4f7e409107b29df92
	 
	vim /k8s/kubernetes/cfg/token.csv
	a1e62c2c2be66bf4f7e409107b29df92,kubelet-bootstrap,10001,"system:kubelet-bootstrap"

### 5.6.3 创建Apiserver配置文件

	vim /k8s/kubernetes/cfg/kube-apiserver
	
	KUBE_APISERVER_OPTS="--logtostderr=true \
	--v=4 \
	--etcd-servers=https://192.168.1.193:2379,https://192.168.1.195:2379,https://192.168.1.196:2379 \
	--bind-address=192.168.1.193 \
	--secure-port=6443 \
	--advertise-address=192.168.1.193 \
	--allow-privileged=true \
	--service-cluster-ip-range=10.254.0.0/16 \
	--enable-admission-plugins=NamespaceLifecycle,LimitRanger,ServiceAccount,ResourceQuota,NodeRestriction \
	--authorization-mode=RBAC,Node \
	--enable-bootstrap-token-auth \
	--token-auth-file=/k8s/kubernetes/cfg/token.csv \
	--service-node-port-range=30000-50000 \
	--tls-cert-file=/k8s/kubernetes/ssl/server.pem  \
	--tls-private-key-file=/k8s/kubernetes/ssl/server-key.pem \
	--client-ca-file=/k8s/kubernetes/ssl/ca.pem \
	--service-account-key-file=/k8s/kubernetes/ssl/ca-key.pem \
	--etcd-cafile=/k8s/etcd/ssl/ca.pem \
	--etcd-certfile=/k8s/etcd/ssl/server.pem \
	--etcd-keyfile=/k8s/etcd/ssl/server-key.pem"

### 5.6.4 创建apiserver systemd文件

	vim /usr/lib/systemd/system/kube-apiserver.service
	
	[Unit]
	Description=Kubernetes API Server
	Documentation=https://github.com/kubernetes/kubernetes
	 
	[Service]
	EnvironmentFile=-/k8s/kubernetes/cfg/kube-apiserver
	ExecStart=/k8s/kubernetes/bin/kube-apiserver $KUBE_APISERVER_OPTS
	Restart=on-failure
	 
	[Install]
	WantedBy=multi-user.target

### 5.6.5 启动服务并查看

	systemctl daemon-reload
	systemctl enable kube-apiserver
	systemctl start kube-apiserver
	
	systemctl status kube-apiserver
	
	● kube-apiserver.service - Kubernetes API Server
	   Loaded: loaded (/usr/lib/systemd/system/kube-apiserver.service; enabled; vendor preset: disabled)
	   Active: active (running) since Wed 2019-03-13 23:35:16 EDT; 11h ago
	     Docs: https://github.com/kubernetes/kubernetes
	 Main PID: 21192 (kube-apiserver)
	   CGroup: /system.slice/kube-apiserver.service
	           └─21192 /k8s/kubernetes/bin/kube-apiserver --logtostderr=true --v=4 --etcd-servers=https://192.168.1.193:2379,https://192.168.1.195:2379,https://192.168.1.1...
	
	Mar 14 11:16:25 master kube-apiserver[21192]: I0314 11:16:25.546492   21192 wrap.go:47] GET /apis/batch/v1beta1/cronjobs: (1.169888ms) 200 [kube-controller-m...0.1:50476]
	Mar 14 11:16:26 master kube-apiserver[21192]: I0314 11:16:26.098678   21192 wrap.go:47] GET /api/v1/namespaces/kube-system/endpoints/kube-scheduler?timeout=1...0.1:35214]
	Mar 14 11:16:26 master kube-apiserver[21192]: I0314 11:16:26.100975   21192 wrap.go:47] PUT /api/v1/namespaces/

### 5.6.6 部署kube-scheduler组件 创建kube-scheduler配置文件

	vim  /k8s/kubernetes/cfg/kube-scheduler 
	KUBE_SCHEDULER_OPTS="--logtostderr=true --v=4 --master=127.0.0.1:8080 --leader-elect"


参数备注： –address：在 127.0.0.1:10251 端口接收 http /metrics 请求；kube-scheduler 目前还不支持接收 https 请求； –kubeconfig：指定 kubeconfig 文件路径，kube-scheduler 使用它连接和验证 kube-apiserver； –leader-elect=true：集群运行模式，启用选举功能；被选为 leader 的节点负责处理工作，其它节点为阻塞状态；

### 5.6.7 创建kube-scheduler systemd文件

	vim /usr/lib/systemd/system/kube-scheduler.service
	
	[Unit]
	Description=Kubernetes Scheduler
	Documentation=https://github.com/kubernetes/kubernetes
	 
	[Service]
	EnvironmentFile=-/k8s/kubernetes/cfg/kube-scheduler
	ExecStart=/k8s/kubernetes/bin/kube-scheduler $KUBE_SCHEDULER_OPTS
	Restart=on-failure
	 
	[Install]
	WantedBy=multi-user.target

### 5.6.8 启动服务

	systemctl daemon-reload
	systemctl enable kube-scheduler 
	systemctl start kube-scheduler
	
	systemctl status kube-scheduler
	
	● kube-scheduler.service - Kubernetes Scheduler
	   Loaded: loaded (/usr/lib/systemd/system/kube-scheduler.service; enabled; vendor preset: disabled)
	   Active: active (running) since Thu 2019-03-14 10:20:16 EDT; 59min ago
	     Docs: https://github.com/kubernetes/kubernetes
	 Main PID: 24092 (kube-scheduler)
	   CGroup: /system.slice/kube-scheduler.service
	           └─24092 /k8s/kubernetes/bin/kube-scheduler --logtostderr=true --v=4 --master=127.0.0.1:8080 --leader-elect
	
	Mar 14 11:12:36 master kube-scheduler[24092]: I0314 11:12:36.240990   24092 reflector.go:357] k8s.io/client-go/informers/factory.go:132: Watch close - *v1.Pe...s received
	Mar 14 11:12:49 master kube-scheduler[24092]: I0314 11:12:49.336068   24092 reflector.go:357] k8s.io/client-go/informers/factory.go:132: Watch close - *v1.No...s receive

### 5.6.9 部署kube-controller-manager组件 创建kube-controller-manager配置文件

	vim /k8s/kubernetes/cfg/kube-controller-manager
	
	KUBE_CONTROLLER_MANAGER_OPTS="--logtostderr=true \
	--v=4 \
	--master=127.0.0.1:8080 \
	--leader-elect=true \
	--address=127.0.0.1 \
	--service-cluster-ip-range=10.254.0.0/16 \
	--cluster-name=kubernetes \
	--cluster-signing-cert-file=/k8s/kubernetes/ssl/ca.pem \
	--cluster-signing-key-file=/k8s/kubernetes/ssl/ca-key.pem  \
	--root-ca-file=/k8s/kubernetes/ssl/ca.pem \
	--service-account-private-key-file=/k8s/kubernetes/ssl/ca-key.pem"

### 5.6.10 创建kube-controller-manager systemd文件

	vim /usr/lib/systemd/system/kube-controller-manager.service 
	
	KUBE_CONTROLLER_MANAGER_OPTS="--logtostderr=true \
	--v=4 \
	--master=127.0.0.1:8080 \
	--leader-elect=true \
	--address=127.0.0.1 \
	--service-cluster-ip-range=10.254.0.0/16 \
	--cluster-name=kubernetes \
	--cluster-signing-cert-file=/k8s/kubernetes/ssl/ca.pem \
	--cluster-signing-key-file=/k8s/kubernetes/ssl/ca-key.pem  \
	--root-ca-file=/k8s/kubernetes/ssl/ca.pem \
	--service-account-private-key-file=/k8s/kubernetes/ssl/ca-key.pem"
	[root@master k8s]# cat  /usr/lib/systemd/system/kube-controller-manager.service 
	[Unit]
	Description=Kubernetes Controller Manager
	Documentation=https://github.com/kubernetes/kubernetes
	 
	[Service]
	EnvironmentFile=-/k8s/kubernetes/cfg/kube-controller-manager
	ExecStart=/k8s/kubernetes/bin/kube-controller-manager $KUBE_CONTROLLER_MANAGER_OPTS
	Restart=on-failure
	 
	[Install]
	WantedBy=multi-user.target

### 5.6.11 启动服务

	systemctl daemon-reload
	systemctl enable kube-controller-manager
	systemctl start kube-controller-manager
	
	systemctl status kube-controller-manager
	
	● kube-controller-manager.service - Kubernetes Controller Manager
	   Loaded: loaded (/usr/lib/systemd/system/kube-controller-manager.service; enabled; vendor preset: disabled)
	   Active: active (running) since Thu 2019-03-14 10:20:22 EDT; 1h 1min ago
	     Docs: https://github.com/kubernetes/kubernetes
	 Main PID: 24101 (kube-controller)
	   CGroup: /system.slice/kube-controller-manager.service
	           └─24101 /k8s/kubernetes/bin/kube-controller-manager --logtostderr=true --v=4 --master=127.0.0.1:8080 --leader-elect=true --address=127.0.0.1 --service-clust...
	
	Mar 14 11:21:46 master kube-controller-manager[24101]: I0314 11:21:46.725766   24101 resource_quota_controller.go:422] no resource updates from discovery, ski...uota sync
	Mar 14 11:21:52 master kube-controller-manager[24101]: I0314 11:21:52.091962   24101 attach_detach_controller.go:634] processVolumesInUse for node "192.168.1.196"

### 5.6.12 查看master服务状态

	[root@master k8s]# kubectl get cs,nodes
	
	NAME                                 STATUS    MESSAGE             ERROR
	componentstatus/controller-manager   Healthy   ok                  
	componentstatus/scheduler            Healthy   ok                  
	componentstatus/etcd-2               Healthy   {"health":"true"}   
	componentstatus/etcd-1               Healthy   {"health":"true"}   
	componentstatus/etcd-0               Healthy   {"health":"true"}

# 六 Node部署

## 6.1 下载安装

	wget https://dl.k8s.io/v1.13.4/kubernetes-node-linux-amd64.tar.gz
	tar zxvf kubernetes-node-linux-amd64.tar.gz
	cd kubernetes/node/bin/
	cp kube-proxy kubelet kubectl /k8s/kubernetes/bin/

## 6.2 创建kubelet bootstrap kubeconfig文件 通过脚本实现

	#!/bin/bash
	#创建kubelet bootstrapping kubeconfig 
	BOOTSTRAP_TOKEN=a1e62c2c2be66bf4f7e409107b29df92 # 这里表示master中生成的tock(/k8s/kubernetes/cfg/token.csv 中)
	KUBE_APISERVER="https://192.168.1.193:6443"
	#设置集群参数
	kubectl config set-cluster kubernetes \
	  --certificate-authority=/k8s/kubernetes/ssl/ca.pem \
	  --embed-certs=true \
	  --server=${KUBE_APISERVER} \
	  --kubeconfig=bootstrap.kubeconfig
	 
	#设置客户端认证参数
	kubectl config set-credentials kubelet-bootstrap \
	  --token=${BOOTSTRAP_TOKEN} \
	  --kubeconfig=bootstrap.kubeconfig
	 
	# 设置上下文参数
	kubectl config set-context default \
	  --cluster=kubernetes \
	  --user=kubelet-bootstrap \
	  --kubeconfig=bootstrap.kubeconfig
	 
	# 设置默认上下文
	kubectl config use-context default --kubeconfig=bootstrap.kubeconfig
	 
	#----------------------
	 
	# 创建kube-proxy kubeconfig文件
	 
	kubectl config set-cluster kubernetes \
	  --certificate-authority=/k8s/kubernetes/ssl/ca.pem \
	  --embed-certs=true \
	  --server=${KUBE_APISERVER} \
	  --kubeconfig=kube-proxy.kubeconfig
	 
	kubectl config set-credentials kube-proxy \
	  --client-certificate=/k8s/kubernetes/ssl/kube-proxy.pem \
	  --client-key=/k8s/kubernetes/ssl/kube-proxy-key.pem \
	  --embed-certs=true \
	  --kubeconfig=kube-proxy.kubeconfig
	 
	kubectl config set-context default \
	  --cluster=kubernetes \
	  --user=kube-proxy \
	  --kubeconfig=kube-proxy.kubeconfig
	 
	kubectl config use-context default --kubeconfig=kube-proxy.kubeconfig


## 6.3 执行脚本

	[root@node1 ~]# cd /k8s/kubernetes/cfg/
	[root@node1 cfg]#sh environment.sh
	Cluster "kubernetes" set.
	User "kubelet-bootstrap" set.
	Context "default" created.
	Switched to context "default".
	Cluster "kubernetes" set.
	User "kube-proxy" set.
	Context "default" created.
	Switched to context "default".
	[root@node1 cfg]# ls
	bootstrap.kubeconfig  environment.sh  kube-proxy.kubeconfig

## 6.4 创建kubelet参数配置模板文件

	vim /k8s/kubernetes/cfg/kubelet.config
	
	kind: KubeletConfiguration
	apiVersion: kubelet.config.k8s.io/v1beta1
	address: 192.168.1.195
	port: 10250
	readOnlyPort: 10255
	cgroupDriver: cgroupfs
	clusterDNS: ["10.254.0.10"]
	clusterDomain: cluster.local.
	failSwapOn: false
	authentication:
	  anonymous:
	    enabled: true

## 6.5 创建kubelet配置文件

	vim /k8s/kubernetes/cfg/kubelet
	
	KUBELET_OPTS="--logtostderr=true \
	--v=4 \
	--hostname-override=192.168.1.195 \
	--kubeconfig=/k8s/kubernetes/cfg/kubelet.kubeconfig \
	--bootstrap-kubeconfig=/k8s/kubernetes/cfg/bootstrap.kubeconfig \
	--config=/k8s/kubernetes/cfg/kubelet.config \
	--cert-dir=/k8s/kubernetes/ssl \
	--pod-infra-container-image=registry.cn-hangzhou.aliyuncs.com/google-containers/pause-amd64:3.0"

## 6.6 创建kubelet systemd文件

	vim /usr/lib/systemd/system/kubelet.service 
	
	[Unit]
	Description=Kubernetes Kubelet
	After=docker.service
	Requires=docker.service
	 
	[Service]
	EnvironmentFile=/k8s/kubernetes/cfg/kubelet
	ExecStart=/k8s/kubernetes/bin/kubelet $KUBELET_OPTS
	Restart=on-failure
	KillMode=process
	 
	[Install]
	WantedBy=multi-user.target

## 6.7 将kubelet-bootstrap用户绑定到系统集群角色(只需要在master上执行一次的)

	kubectl create clusterrolebinding kubelet-bootstrap \
	  --clusterrole=system:node-bootstrapper \
	  --user=kubelet-bootstrap

注意这个默认连接localhost:8080端口，可以在master上操作

## 6.8 启动kubelet

	systemctl daemon-reload
	systemctl enable kubelet
	systemctl start kubelet
	
	systemctl status kubelet
	
	● kubelet.service - Kubernetes Kubelet
	   Loaded: loaded (/usr/lib/systemd/system/kubelet.service; enabled; vendor preset: disabled)
	   Active: active (running) since Thu 2019-03-14 02:06:01 EDT; 9h ago
	 Main PID: 25070 (kubelet)
	    Tasks: 20
	   Memory: 38.8M
	   CGroup: /system.slice/kubelet.service
	           └─25070 /k8s/kubernetes/bin/kubelet --logtostderr=true --v=4 --hostname-override=192.168.1.195 --kubeconfig=/k8s/kubernetes/cfg/kubelet.kubeconfig --bootstr...
	
	Mar 14 11:33:54 node1 kubelet[25070]: I0314 11:33:54.875181   25070 helpers.go:836] eviction manager: observations: signal=nodefs.available, available: 3650...4.42403586

## 6.9 将node添加到集群(master上添加)

### 6.9.1  Master接受kubelet CSR请求
Master接受kubelet CSR请求可以手动或自动 approve CSR 请求。推荐使用自动的方式，因为从 v1.8 版本开始，可以自动轮转approve csr 后生成的证书，如下是手动 approve CSR请求操作方法 查看CSR列表

	[root@master cfg]#kubectl get csr
	NAME                                                   AGE    REQUESTOR           CONDITION
	node-csr-ij3py9j-yi-eoa8sOHMDs7VeTQtMv0N3Efj3ByZLMdc   102s   kubelet-bootstrap   Pending

### 6.9.2 接受node

	[root@master cfg]# kubectl certificate approve node-csr-ij3py9j-yi-eoa8sOHMDs7VeTQtMv0N3Efj3ByZLMdc
	certificatesigningrequest.certificates.k8s.io/node-csr-ij3py9j-yi-eoa8sOHMDs7VeTQtMv0N3Efj3ByZLMdc approved

### 6.9.3 查看,状态发生改变
	[root@master cfg]#kubectl get csr
	NAME                                                   AGE     REQUESTOR           CONDITION
	node-csr-ij3py9j-yi-eoa8sOHMDs7VeTQtMv0N3Efj3ByZLMdc   5m13s   kubelet-bootstrap   Approved,Issued

## 6.10 部署kube-proxy组件

kube-proxy 运行在所有 node节点上，它监听 apiserver 中 service 和 Endpoint 的变化情况，创建路由规则来进行服务负载均衡 

### 6.10.1 创建 kube-proxy 配置文件

	vim /k8s/kubernetes/cfg/kube-proxy
	
	KUBE_PROXY_OPTS="--logtostderr=true \
	--v=4 \
	--hostname-override=192.168.1.195 \
	--cluster-cidr=10.254.0.0/16 \
	--kubeconfig=/k8s/kubernetes/cfg/kube-proxy.kubeconfig

### 6.10.2 创建kube-proxy systemd文件

	vim /usr/lib/systemd/system/kube-proxy.service
	
	[Unit]
	Description=Kubernetes Proxy
	After=network.target
	 
	[Service]
	EnvironmentFile=-/k8s/kubernetes/cfg/kube-proxy
	ExecStart=/k8s/kubernetes/bin/kube-proxy $KUBE_PROXY_OPTS
	Restart=on-failure
	 
	[Install]
	WantedBy=multi-user.target
	[root@node1 cfg]# cat /usr/lib/systemd/system/kube-proxy.service 
	[Unit]
	Description=Kubernetes Proxy
	After=network.target
	 
	[Service]
	EnvironmentFile=-/k8s/kubernetes/cfg/kube-proxy
	ExecStart=/k8s/kubernetes/bin/kube-proxy $KUBE_PROXY_OPTS
	Restart=on-failure
	 
	[Install]
	WantedBy=multi-user.target


### 6.10.3 启动服务

	systemctl daemon-reload
	systemctl enable kube-proxy
	systemctl start kube-proxy
	
	systemctl status kube-proxy
	
	● kube-proxy.service - Kubernetes Proxy
	   Loaded: loaded (/usr/lib/systemd/system/kube-proxy.service; enabled; vendor preset: disabled)
	   Active: active (running) since Thu 2019-03-14 02:06:04 EDT; 9h ago
	 Main PID: 25153 (kube-proxy)
	    Tasks: 0
	   Memory: 9.1M
	   CGroup: /system.slice/kube-proxy.service
	           ‣ 25153 /k8s/kubernetes/bin/kube-proxy --logtostderr=true --v=4 --hostname-override=192.168.1.195 --cluster-cidr=10.254.0.0/16 --kubeconfig=/k8s/kubernetes/...
	
	Mar 14 11:41:04 node1 kube-proxy[25153]: I0314 11:41:04.459580   25153 config.go:141] Calling handler.OnEndpointsUpdate

## 6.10.4 查看集群状态

	[root@master cfg]# kubectl get nodes
	NAME            STATUS   ROLES    AGE   VERSION
	192.168.1.195   Ready    <none>   11h   v1.13.4

## 6.11 同样操作部署node2 并认证csr，认证后会生成kubelet-client证书

	[root@node1 kubernetes]# ls ssl/
	ca-key.pem  kubelet-client-2019-03-14-00-01-07.pem  kubelet.crt  kube-proxy-key.pem  server-key.pem
	ca.pem      kubelet-client-current.pem              kubelet.key  kube-proxy.pem      server.pem
	
	[root@master cfg]# kubectl get nodes
	NAME            STATUS   ROLES    AGE   VERSION
	192.168.1.195   Ready    <none>   11h   v1.13.4
	192.168.1.196   Ready    <none>   11h   v1.13.4

但需要删除该节点后重新添加需要

	[root@master cfg]# kubectl delete node 192.168.1.195
	[root@node1 ssl ]# rm -rf  kubelet-client* kubelet*
	[root@node1 ssl ]# systemctl restart kubelet

# 七 Flanneld网络部署

默认没有flanneld网络，Node节点间的pod不能通信，只能Node内通信，为了部署步骤简洁明了，故flanneld放在后面安装 flannel服务需要先于docker启动。flannel服务启动时主要做了以下几步的工作： 从etcd中获取network的配置信息 划分subnet，并在etcd中进行注册 将子网信息记录到/run/flannel/subnet.env中

## 7.1 etcd 注册网段(master上执行)

	[root@master cfg]#/k8s/etcd/bin/etcdctl --ca-file=/k8s/etcd/ssl/ca.pem --cert-file=/k8s/etcd/ssl/server.pem --key-file=/k8s/etcd/ssl/server-key.pem --endpoints="https://192.168.1.193:2379,https://192.168.1.195:2379,https://192.168.1.196:2379"  set /coreos.com/network/config  '{ "Network": "10.254.0.0/16", "Backend": {"Type": "vxlan"}}'
	
	
	或者
	
	/k8s/etcd/bin/etcdctl --ca-file=/k8s/etcd/ssl/ca.pem --cert-file=/k8s/etcd/ssl/server.pem --key-file=/k8s/etcd/ssl/server-key.pem --endpoints="https://192.168.1.193:2379,https://192.168.1.195:2379,https://192.168.1.196:2379"  set /coreos.com/network/config  '{ "Network": "10.254.0.0/16", "Backend": {"Type": "vxlan","Directrouting": true}}'

flanneld 当前版本 (v0.10.0) 不支持 etcd v3，故使用 etcd v2 API 写入配置 key 和网段数据； 写入的 Pod 网段 ${CLUSTER_CIDR} 必须是 /16 段地址，必须与 kube-controller-manager 的 –cluster-cidr 参数值一致；

## 7.2 flannel下载解压

	wget https://github.com/coreos/flannel/releases/download/v0.11.0/flannel-v0.11.0-linux-amd64.tar.gz
	tar -xf flannel-v0.11.0-linux-amd64.tar.gz
	mv flanneld mk-docker-opts.sh /k8s/kubernetes/bin/

## 7.3 配置flanneld

### 7.3. 1 创建配置文件

	vim /k8s/kubernetes/cfg/flanneld
	
	FLANNEL_OPTIONS="--etcd-endpoints=https://192.168.1.193:2379,https://192.168.1.195:2379,https://192.168.1.196:2379 -etcd-cafile=/k8s/etcd/ssl/ca.pem -etcd-certfile=/k8s/etcd/ssl/server.pem -etcd-keyfile=/k8s/etcd/ssl/server-key.pem"

### 7.3.2 创建flanneld systemd文件

	vim /usr/lib/systemd/system/flanneld.service
	
	[Unit]
	Description=Flanneld overlay address etcd agent
	After=network-online.target network.target
	Before=docker.service
	 
	[Service]
	Type=notify
	EnvironmentFile=/k8s/kubernetes/cfg/flanneld
	ExecStart=/k8s/kubernetes/bin/flanneld --ip-masq $FLANNEL_OPTIONS
	ExecStartPost=/k8s/kubernetes/bin/mk-docker-opts.sh -k DOCKER_NETWORK_OPTIONS -d /run/flannel/subnet.env
	Restart=on-failure
	 
	[Install]
	WantedBy=multi-user.target

注: mk-docker-opts.sh 脚本将分配给 flanneld 的 Pod 子网网段信息写入 /run/flannel/docker 文件，后续 docker 启动时 使用这个文件中的环境变量配置 docker0 网桥； flanneld 使用系统缺省路由所在的接口与其它节点通信，对于有多个网络接口（如内网和公网）的节点，可以用 -iface 参数指定通信接口; flanneld 运行时需要 root 权限；

### 7.3.3 更改docker配置

配置Docker启动指定子网 修改EnvironmentFile=/run/flannel/subnet.env，ExecStart=/usr/bin/dockerd $DOCKER_NETWORK_OPTIONS即可

	vim /usr/lib/systemd/system/docker.service 
	
	[Service]
	Type=notify
	# the default is not to use systemd for cgroups because the delegate issues still
	# exists and systemd currently does not support the cgroup feature set required
	# for containers run by docker
	EnvironmentFile=/run/flannel/subnet.env
	ExecStart=/usr/bin/dockerd $DOCKER_NETWORK_OPTIONS
	#ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock #注释这个 添加上面两行
	ExecReload=/bin/kill -s HUP $MAINPID
	TimeoutSec=0
	RestartSec=2
	Restart=always

### 7.3.4 启动服务

 注意启动flannel前要关闭docker及相关的kubelet这样flannel才会覆盖docker0网桥

	systemctl daemon-reload
	systemctl stop docker
	systemctl start flanneld
	systemctl enable flanneld
	systemctl start docker
	systemctl restart kubelet
	systemctl restart kube-proxy

### 7.3.5 验证服务(生成该文件)

	[root@node1 kubernetes]#  cat /run/flannel/subnet.env 
	
	DOCKER_OPT_BIP="--bip=10.254.101.1/24"
	DOCKER_OPT_IPMASQ="--ip-masq=false"
	DOCKER_OPT_MTU="--mtu=1450"
	DOCKER_NETWORK_OPTIONS=" --bip=10.254.101.1/24 --ip-masq=false --mtu=1450"
	
	[root@node1 kubernetes]# ifconfig 
	
	docker0: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
	        inet 10.254.101.1  netmask 255.255.255.0  broadcast 10.254.101.255
	        ether 02:42:86:df:43:30  txqueuelen 0  (Ethernet)
	        RX packets 0  bytes 0 (0.0 B)
	        RX errors 0  dropped 0  overruns 0  frame 0
	        TX packets 0  bytes 0 (0.0 B)
	        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
	
	ens32: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
	        inet 192.168.1.195  netmask 255.255.255.0  broadcast 192.168.1.255
	        inet6 fe80::20c:29ff:fe5c:6966  prefixlen 64  scopeid 0x20<link>
	        ether 00:0c:29:5c:69:66  txqueuelen 1000  (Ethernet)
	        RX packets 1969130  bytes 312830914 (298.3 MiB)
	        RX errors 0  dropped 37  overruns 0  frame 0
	        TX packets 1878838  bytes 270822042 (258.2 MiB)
	        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
	
	flannel.1: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1450
	        inet 10.254.101.0  netmask 255.255.255.255  broadcast 0.0.0.0
	        inet6 fe80::68c6:49ff:fe9d:e75  prefixlen 64  scopeid 0x20<link>
	        ether 6a:c6:49:9d:0e:75  txqueuelen 0  (Ethernet)
	        RX packets 3  bytes 252 (252.0 B)
	        RX errors 0  dropped 0  overruns 0  frame 0
	        TX packets 3  bytes 252 (252.0 B)
	        TX errors 0  dropped 8 overruns 0  carrier 0  collisions 0
	
	[root@master cfg]#  kubectl get nodes
	NAME            STATUS   ROLES    AGE   VERSION
	192.168.1.195   Ready    <none>   11h   v1.13.4
	192.168.1.196   Ready    <none>   11h   v1.13.4

在master上面也可以查看到(可以ping通node1 上 docker0网络)

	[root@master cfg]# ifconfig 
	ens32: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
	        inet 192.168.1.193  netmask 255.255.255.0  broadcast 192.168.1.255
	        inet6 fe80::20c:29ff:fe49:1c50  prefixlen 64  scopeid 0x20<link>
	        ether 00:0c:29:49:1c:50  txqueuelen 1000  (Ethernet)
	        RX packets 1344137  bytes 191637030 (182.7 MiB)
	        RX errors 0  dropped 91  overruns 0  frame 0
	        TX packets 1336623  bytes 310854692 (296.4 MiB)
	        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
	
	flannel.1: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1450
	        inet 10.254.64.0  netmask 255.255.255.255  broadcast 0.0.0.0
	        inet6 fe80::4477:caff:fe00:b4fe  prefixlen 64  scopeid 0x20<link>
	        ether 46:77:ca:00:b4:fe  txqueuelen 0  (Ethernet)
	        RX packets 4  bytes 336 (336.0 B)
	        RX errors 0  dropped 0  overruns 0  frame 0
	        TX packets 4  bytes 336 (336.0 B)
	        TX errors 0  dropped 8 overruns 0  carrier 0  collisions 0
	
	[root@master cfg]# ping 10.254.101.1 
	PING 10.254.101.1 (10.254.101.1) 56(84) bytes of data.
	64 bytes from 10.254.101.1: icmp_seq=1 ttl=64 time=0.158 ms
	64 bytes from 10.254.101.1: icmp_seq=2 ttl=64 time=0.108 ms
	64 bytes from 10.254.101.1: icmp_seq=3 ttl=64 time=0.112 ms
	64 bytes from 10.254.101.1: icmp_seq=4 ttl=64 time=0.157 ms

# 八. 完成