---
layout: post
title: kubernetes 1.10
categories: kubernetes
description: kubernetes 1.10
keywords: kubernetes
---


# Kubernetes v1.10.x HA 全手动安装教程(TL;DR) #

本篇延续过往手动安装方式来部署 Kubernetes v1.10.x 版本的 High Availability 集群，主要目的是学习 Kubernetes 安装的一些元件关析与流程。若不想这么累的话，可以参考 Picking the Right Solution 来选择自己最喜欢的方式。

**本次安装的软件版本：**

- Kubernetes v1.10.0
- CNI v0.6.0
- Etcd v3.1.13
- Calico v3.0.4
- Docker CE latest version

![](https://i.imgur.com/bzKLvS9.png)

## 节点信息 ##

**本教学将以下列节点数与规格来进行部署 Kubernetes 集群，操作系统可采用Ubuntu 16.x与CentOS 7.x：**

IP |	Address	|	Hostname	|	CPU	Memory
---|:----------:|:-------------:|------------
192.16.35.11	|	k8s-m1	|	1	|	4G
192.16.35.12	|	k8s-m2	|	1	|	4G
192.16.35.13	|	k8s-m3	|	1	|	4G
192.16.35.14	|	k8s-n1	|	1	|	4G
192.16.35.15	|	k8s-n2	|	1	|	4G
192.16.35.16	|	k8s-n2	|	1	|	4G
 

另外由所有 master 节点提供一组 VIP 192.16.35.10。

- 这边m为主要控制节点，n为应用程序工作节点。
- 所有操作全部用root使用者进行(方便用)，以 SRE 来说不推荐。
- 可以下载Vagrantfile 来建立 Virtualbox 虚拟机集群。不过需要注意机器资源是否足够。

## 事前准备 ##

**开始安装前需要确保以下条件已达成：**

所有节点彼此网络互通，并且k8s-m1SSH 登入其他节点为 passwdless。

所有防火墙与 SELinux 已关闭。

如 CentOS：

    $ systemctl stop firewalld && systemctl disable firewalld
    $ setenforce 0
    $ vim /etc/selinux/config
    SELINUX=disabled

所有节点需要设定/etc/hosts解析到所有集群主机。

	...
	192.16.35.11 k8s-m1
	192.16.35.12 k8s-m2
	192.16.35.13 k8s-m3
	192.16.35.14 k8s-n1
	192.16.35.15 k8s-n2
	192.16.35.16 k8s-n3

所有节点需要安装 Docker CE 版本的容器引擎：

	$ curl -fsSL "https://get.docker.com/" | sh

不管是在 Ubuntu 或 CentOS 都只需要执行该指令就会自动安装最新版 Docker。
CentOS 安装完成后，需要再执行以下指令：

	$ systemctl enable docker && systemctl start docker

所有节点需要设定/etc/sysctl.d/k8s.conf的系统参数。

	$ cat <<EOF > /etc/sysctl.d/k8s.conf

	net.ipv4.ip_forward = 1

	net.bridge.bridge-nf-call-ip6tables = 1

	net.bridge.bridge-nf-call-iptables = 1

	EOF

	$ sysctl -p /etc/sysctl.d/k8s.conf

Kubernetes v1.8+ 要求关闭系统 Swap，若不关闭则需要修改 kubelet 设定参数，在所有节点利用以下指令关闭：

	$ swapoff -a && sysctl -w vm.swappiness=0

**记得/etc/fstab也要注解掉SWAP挂载。**

在所有节点下载 Kubernetes 二进制执行档：

	$ export KUBE_URL="https://storage.googleapis.com/kubernetes-release/release/v1.10.0/bin/linux/amd64"
	
	$ wget "${KUBE_URL}/kubelet" -O /usr/local/bin/kubelet
	
	$ chmod +x /usr/local/bin/kubelet
	
	# node 请忽略下载 kubectl
	
	$ wget "${KUBE_URL}/kubectl" -O /usr/local/bin/kubectl
	
	$ chmod +x /usr/local/bin/kubectl

在所有节点下载 Kubernetes CNI 二进制文件：

	$ mkdir -p /opt/cni/bin && cd /opt/cni/bin
	
	$ export CNI_URL="https://github.com/containernetworking/plugins/releases/download"
	
	$ wget -qO- --show-progress "${CNI_URL}/v0.6.0/cni-plugins-amd64-v0.6.0.tgz" | tar -zx

在k8s-m1需要安装CFSSL工具，这将会用来建立 TLS Certificates。

	$ export CFSSL_URL="https://pkg.cfssl.org/R1.2"
	
	$ wget "${CFSSL_URL}/cfssl_linux-amd64" -O /usr/local/bin/cfssl
	
	$ wget "${CFSSL_URL}/cfssljson_linux-amd64" -O /usr/local/bin/cfssljson
	
	$ chmod +x /usr/local/bin/cfssl /usr/local/bin/cfssljson

## 建立集群 CA keys 与 Certificates ##

在这个部分，将需要产生多个元件的 Certificates，这包含 Etcd、Kubernetes 元件等，并且每个集群都会有一个根数位凭证认证机构(Root Certificate Authority)被用在认证 API Server 与 Kubelet 端的凭证。

P.S. 这边要注意 CA JSON 档的CN(Common Name)与O(Organization)等内容是会影响 Kubernetes 元件认证的。

### Etcd ###

首先在k8s-m1建立/etc/etcd/ssl资料夹，然后进入目录完成以下操作。

	$ mkdir -p /etc/etcd/ssl && cd /etc/etcd/ssl
	
	$ export PKI_URL="https://kairen.github.io/files/manual-v1.10/pki"

下载ca-config.json与etcd-ca-csr.json文件，并从 CSR json 产生 CA keys 与 Certificate：

	$ wget "${PKI_URL}/ca-config.json" "${PKI_URL}/etcd-ca-csr.json"
	
	$ cfssl gencert -initca etcd-ca-csr.json | cfssljson -bare etcd-ca

下载etcd-csr.json文件，并产生 Etcd 证书：

	$ wget "${PKI_URL}/etcd-csr.json"
	
	$ cfssl gencert \
	
	  -ca=etcd-ca.pem \
	
	  -ca-key=etcd-ca-key.pem \
	
	  -config=ca-config.json \
	
	  -hostname=127.0.0.1,192.16.35.11,192.16.35.12,192.16.35.13 \
	
	  -profile=kubernetes \
	
	  etcd-csr.json | cfssljson -bare etcd

-hostnam表示需修改成所有 masters 节点。

完成后删除不必要文件：

	$ rm -rf *.json *.csr

确认/etc/etcd/ssl有以下文件：

	$ ls /etc/etcd/ssl

	etcd-ca-key.pem  etcd-ca.pem  etcd-key.pem  etcd.pem

复制相关文件至其他 Etcd 节点，这边为所有master节点：

	$ for NODE in k8s-m2 k8s-m3; do
	
	    echo "--- $NODE ---"
	
	    ssh ${NODE} "mkdir -p /etc/etcd/ssl"
	
	    for FILE in etcd-ca-key.pem  etcd-ca.pem  etcd-key.pem  etcd.pem; do
	
	      scp /etc/etcd/ssl/${FILE} ${NODE}:/etc/etcd/ssl/${FILE}
	
	    done
	
	  done

## Kubernetes ##

在k8s-m1建立pki资料夹，然后进入目录完成以下章节操作。

	$ mkdir -p /etc/kubernetes/pki && cd /etc/kubernetes/pki
	
	$ export PKI_URL="https://kairen.github.io/files/manual-v1.10/pki"
	
	$ export KUBE_APISERVER="https://192.16.35.10:6443"

下载ca-config.json与ca-csr.json文件，并产生 CA 金钥：

	$ wget "${PKI_URL}/ca-config.json" "${PKI_URL}/ca-csr.json"
	
	$ cfssl gencert -initca ca-csr.json | cfssljson -bare ca
	
	$ ls ca*.pem
	
	ca-key.pem  ca.pem

### API Server Certificate ###

下载apiserver-csr.json文件，并产生 kube-apiserver 凭证：

	$ wget "${PKI_URL}/apiserver-csr.json"
	
	$ cfssl gencert \
	
	  -ca=ca.pem \
	
	  -ca-key=ca-key.pem \
	
	  -config=ca-config.json \
	
	  -hostname=10.96.0.1,192.16.35.10,127.0.0.1,kubernetes.default \
	
	  -profile=kubernetes \
	
	  apiserver-csr.json | cfssljson -bare apiserver
	
	
	
	$ ls apiserver*.pem
	
	apiserver-key.pem  apiserver.pem

这边-hostname的96.0.1是 Cluster IP 的 Kubernetes 端点;

16.35.10为虚拟 IP 位址(VIP);

default为 Kubernetes DN。

### Front Proxy Certificate ###

下载front-proxy-ca-csr.json文件，并产生 Front Proxy CA 金钥，Front Proxy 主要是用在 API aggregator 上:

	$ wget "${PKI_URL}/front-proxy-ca-csr.json"
	
	$ cfssl gencert \
	
	  -initca front-proxy-ca-csr.json | cfssljson -bare front-proxy-ca
	
		
	$ ls front-proxy-ca*.pem
	
	front-proxy-ca-key.pem  front-proxy-ca.pem

下载front-proxy-client-csr.json文件，并产生 front-proxy-client 证书：

	$ wget "${PKI_URL}/front-proxy-client-csr.json"
	
	$ cfssl gencert \
	
	  -ca=front-proxy-ca.pem \
	
	  -ca-key=front-proxy-ca-key.pem \
	
	  -config=ca-config.json \
	
	  -profile=kubernetes \
	
	  front-proxy-client-csr.json | cfssljson -bare front-proxy-client
	
	
	
	$ ls front-proxy-client*.pem
	
	front-proxy-client-key.pem  front-proxy-client.pem

### Admin Certificate ###

下载admin-csr.json文件，并产生 admin certificate 凭证：

	$ wget "${PKI_URL}/admin-csr.json"
	
	$ cfssl gencert \
	
	  -ca=ca.pem \
	
	  -ca-key=ca-key.pem \
	
	  -config=ca-config.json \
	
	  -profile=kubernetes \
	
	  admin-csr.json | cfssljson -bare admin
	
	
	
	$ ls admin*.pem
	
	admin-key.pem  admin.pem

接着通过以下指令产生名称为 admin.conf 的 kubeconfig 档：

	# admin set cluster
	
	$ kubectl config set-cluster kubernetes \
	
	    --certificate-authority=ca.pem \
	
	    --embed-certs=true \
	
	    --server=${KUBE_APISERVER} \
	
	    --kubeconfig=../admin.conf
	
	# admin set credentials
	
	$ kubectl config set-credentials kubernetes-admin \
	
	    --client-certificate=admin.pem \
	
	    --client-key=admin-key.pem \
	
	    --embed-certs=true \
	
	    --kubeconfig=../admin.conf
	
	# admin set context
	
	$ kubectl config set-context kubernetes-admin@kubernetes \
	
	    --cluster=kubernetes \
	
	    --user=kubernetes-admin \
	
	    --kubeconfig=../admin.conf
	
	# admin set default context
	
	$ kubectl config use-context kubernetes-admin@kubernetes \
	
	    --kubeconfig=../admin.conf

### Controller Manager Certificate ###

下载manager-csr.json文件，并产生 kube-controller-manager certificate 凭证：

	$ wget "${PKI_URL}/manager-csr.json"
	
	$ cfssl gencert \
	
	  -ca=ca.pem \
	
	  -ca-key=ca-key.pem \
	
	  -config=ca-config.json \
	
	  -profile=kubernetes \
	
	  manager-csr.json | cfssljson -bare controller-manager
	
	
	
	$ ls controller-manager*.pem
	
	controller-manager-key.pem  controller-manager.pem

若节点 IP 不同，需要修改manager-csr.json的hosts。

接着通过以下指令产生名称为controller-manager.conf的 kubeconfig 档：

	# controller-manager set cluster
	
	$ kubectl config set-cluster kubernetes \
	
	    --certificate-authority=ca.pem \
	
	    --embed-certs=true \
	
	    --server=${KUBE_APISERVER} \
	
	    --kubeconfig=../controller-manager.conf
	
	# controller-manager set credentials
	
	$ kubectl config set-credentials system:kube-controller-manager \
	
	    --client-certificate=controller-manager.pem \
	
	    --client-key=controller-manager-key.pem \
	
	    --embed-certs=true \
	
	    --kubeconfig=../controller-manager.conf
	
	# controller-manager set context
	
	$ kubectl config set-context system:kube-controller-manager@kubernetes \
	
	    --cluster=kubernetes \
	
	    --user=system:kube-controller-manager \
	
	    --kubeconfig=../controller-manager.conf
	
	# controller-manager set default context
	
	$ kubectl config use-context system:kube-controller-manager@kubernetes \
	
	    --kubeconfig=../controller-manager.conf

### Scheduler Certificate ###

下载scheduler-csr.json文件，并产生 kube-scheduler certificate 凭证：

	$ wget "${PKI_URL}/scheduler-csr.json"
	
	$ cfssl gencert \
	
	  -ca=ca.pem \
	
	  -ca-key=ca-key.pem \
	
	  -config=ca-config.json \
	
	  -profile=kubernetes \
	
	  scheduler-csr.json | cfssljson -bare scheduler
	
	
	
	$ ls scheduler*.pem
	
	scheduler-key.pem  scheduler.pem

若节点 IP 不同，需要修改scheduler-csr.json的hosts。

接着通过以下指令产生名称为 scheduler.conf 的 kubeconfig 档：

	# scheduler set cluster
	
	$ kubectl config set-cluster kubernetes \
	
	    --certificate-authority=ca.pem \
	
	    --embed-certs=true \
	
	    --server=${KUBE_APISERVER} \
	
	    --kubeconfig=../scheduler.conf
	
	# scheduler set credentials
	
	$ kubectl config set-credentials system:kube-scheduler \
	
	    --client-certificate=scheduler.pem \
	
	    --client-key=scheduler-key.pem \
	
	    --embed-certs=true \
	
	    --kubeconfig=../scheduler.conf
	
	# scheduler set context
	
	$ kubectl config set-context system:kube-scheduler@kubernetes \
	
	    --cluster=kubernetes \
	
	    --user=system:kube-scheduler \
	
	    --kubeconfig=../scheduler.conf
	
	# scheduler use default context
	
	$ kubectl config use-context system:kube-scheduler@kubernetes \
	
	    --kubeconfig=../scheduler.conf

### Master Kubelet Certificate ###

接着在所有k8s-m1节点下载kubelet-csr.json文件，并产生凭证：

	$ wget "${PKI_URL}/kubelet-csr.json"
	
	$ for NODE in k8s-m1 k8s-m2 k8s-m3; do
	
	    echo "--- $NODE ---"
	
	    cp kubelet-csr.json kubelet-$NODE-csr.json;
	
	    sed -i "s/\$NODE/$NODE/g" kubelet-$NODE-csr.json;
	
	    cfssl gencert \
	
	      -ca=ca.pem \
	
	      -ca-key=ca-key.pem \
	
	      -config=ca-config.json \
	
	      -hostname=$NODE \
	
	      -profile=kubernetes \
	
	      kubelet-$NODE-csr.json | cfssljson -bare kubelet-$NODE
	
	  done
	
	
	
	$ ls kubelet*.pem
	
	kubelet-k8s-m1-key.pem  kubelet-k8s-m1.pem  kubelet-k8s-m2-key.pem  kubelet-k8s-m2.pem  kubelet-k8s-m3-key.pem  kubelet-k8s-m3.pem

这边需要依据节点修改-hostname与$NODE。

完成后复制 kubelet 凭证至其他master节点：

	$ for NODE in k8s-m2 k8s-m3; do
	
	    echo "--- $NODE ---"
	
	    ssh ${NODE} "mkdir -p /etc/kubernetes/pki"
	
	    for FILE in kubelet-$NODE-key.pem kubelet-$NODE.pem ca.pem; do
	
	      scp /etc/kubernetes/pki/${FILE} ${NODE}:/etc/kubernetes/pki/${FILE}
	
	    done
	
	  done

接着执行以下指令产生名称为kubelet.conf的 kubeconfig 档：

	$ for NODE in k8s-m1 k8s-m2 k8s-m3; do
	
	    echo "--- $NODE ---"
	
	    ssh ${NODE} "cd /etc/kubernetes/pki && \
	
	      kubectl config set-cluster kubernetes \
	
	        --certificate-authority=ca.pem \
	
	        --embed-certs=true \
	
	        --server=${KUBE_APISERVER} \
	
	        --kubeconfig=../kubelet.conf && \
	
	      kubectl config set-cluster kubernetes \
	
	        --certificate-authority=ca.pem \
	
	        --embed-certs=true \
	
	        --server=${KUBE_APISERVER} \
	
	        --kubeconfig=../kubelet.conf && \
	
	      kubectl config set-credentials system:node:${NODE} \
	
	        --client-certificate=kubelet-${NODE}.pem \
	
	        --client-key=kubelet-${NODE}-key.pem \
	
	        --embed-certs=true \
	
	        --kubeconfig=../kubelet.conf && \
	
	      kubectl config set-context system:node:${NODE}@kubernetes \
	
	        --cluster=kubernetes \
	
	        --user=system:node:${NODE} \
	
	        --kubeconfig=../kubelet.conf && \
	
	      kubectl config use-context system:node:${NODE}@kubernetes \
	
	        --kubeconfig=../kubelet.conf && \
	
	      rm kubelet-${NODE}.pem kubelet-${NODE}-key.pem"
	
	  done

### Service Account Key ###

Service account 不是通过 CA 进行认证，因此不要通过 CA 来做 Service account key 的检查，这边建立一组 Private 与 Public 金钥提供给 Service account key 使用：

	$ openssl genrsa -out sa.key 2048
	
	$ openssl rsa -in sa.key -pubout -out sa.pub
	
	$ ls sa.*
	
	sa.key  sa.pub

删除不必要文件

所有信息准备完成后，就可以将一些不必要文件删除：

	$ rm -rf *.json *.csr scheduler*.pem controller-manager*.pem admin*.pem kubelet*.pem

复制文件至其他节点

复制凭证文件至其他master节点：

	$ for NODE in k8s-m2 k8s-m3; do
	
	    echo "--- $NODE ---"
	
	    for FILE in $(ls /etc/kubernetes/pki/); do
	
	      scp /etc/kubernetes/pki/${FILE} ${NODE}:/etc/kubernetes/pki/${FILE}
	
	    done
	
	  done


复制 Kubernetes config 文件至其他master节点：

	$ for NODE in k8s-m2 k8s-m3; do
	
	    echo "--- $NODE ---"
	
	    for FILE in admin.conf controller-manager.conf scheduler.conf; do
	
	      scp /etc/kubernetes/${FILE} ${NODE}:/etc/kubernetes/${FILE}
	
	    done
	
	  done
 

## Kubernetes Masters ##

本部分将说明如何建立与设定 Kubernetes Master 角色，过程中会部署以下元件：

- kube-apiserver：提供 REST APIs，包含授权、认证与状态储存等。
- kube-controller-manager：负责维护集群的状态，如自动扩展，滚动更新等。
- kube-scheduler：负责资源排程，依据预定的排程策略将 Pod 分配到对应节点上。
- Etcd：储存集群所有状态的 Key/Value 储存系统。
- HAProxy：提供负载平衡器。
- Keepalived：提供虚拟网络位址(VIP)。

## 部署与设定 ##

首先在所有 master 节点下载部署元件的 YAML 文件，这边不采用二进制执行档与 Systemd 来管理这些元件，全部采用 Static Pod 来达成。这边将文件下载至/etc/kubernetes/manifests目录：

	$ export CORE_URL="https://kairen.github.io/files/manual-v1.10/master"
	
	$ mkdir -p /etc/kubernetes/manifests && cd /etc/kubernetes/manifests
	
	$ for FILE in kube-apiserver kube-controller-manager kube-scheduler haproxy keepalived etcd etcd.config; do
	
	    wget "${CORE_URL}/${FILE}.yml.conf" -O ${FILE}.yml
	
	    if [ ${FILE} == "etcd.config" ]; then
	
	      mv etcd.config.yml /etc/etcd/etcd.config.yml
	
	      sed -i "s/\${HOSTNAME}/${HOSTNAME}/g" /etc/etcd/etcd.config.yml
	
	      sed -i "s/\${PUBLIC_IP}/$(hostname -i)/g" /etc/etcd/etcd.config.yml
	
	    fi
	
	  done
	
	
	
	$ ls /etc/kubernetes/manifests
	
	etcd.yml  haproxy.yml  keepalived.yml  kube-apiserver.yml  kube-controller-manager.yml  kube-scheduler.yml

- 若IP与教学设定不同的话，请记得修改 YAML 文件。
- kube-apiserver 中的NodeRestriction 请参考 Using Node Authorization。

产生一个用来加密 Etcd 的 Key：

	$ head -c 32 /dev/urandom | base64SUpbL4juUYyvxj3/gonV5xVEx8j769/99TSAf8YT/sQ=

注意每台master节点需要用一样的 Key。

在/etc/kubernetes/目录下，建立encryption.yml的加密 YAML 文件：

	$ cat <<EOF > /etc/kubernetes/encryption.yml
	
	kind: EncryptionConfig
	
	apiVersion: v1
	
	resources:
	
	  - resources:
	
	      - secrets
	
	    providers:
	
	      - aescbc:
	
	          keys:
	
	            - name: key1
	
	              secret: SUpbL4juUYyvxj3/gonV5xVEx8j769/99TSAf8YT/sQ=
	
	      - identity: {}
	
	EOF

Etcd 资料加密可参考这篇 [Encrypting data at rest](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/)。

在/etc/kubernetes/目录下，建立audit-policy.yml的进阶稽核策略 YAML 档：

	$ cat <<EOF > /etc/kubernetes/audit-policy.yml
	
	apiVersion: audit.k8s.io/v1beta1
	
	kind: Policy
	
	rules:- level: Metadata
	
	EOF

Audit Policy 请参考这篇 [Auditing](https://kubernetes.io/docs/tasks/debug-application-cluster/audit/)。

下载haproxy.cfg文件来提供给 HAProxy 容器使用：

	$ mkdir -p /etc/haproxy/
	
	$ wget "${CORE_URL}/haproxy.cfg" -O /etc/haproxy/haproxy.cfg

若与本教学 IP 不同的话，请记得修改设定档。

下载kubelet.service相关文件来管理 kubelet：

	$ mkdir -p /etc/systemd/system/kubelet.service.d
	
	$ wget "${CORE_URL}/kubelet.service" -O /lib/systemd/system/kubelet.service
	
	$ wget "${CORE_URL}/10-kubelet.conf" -O /etc/systemd/system/kubelet.service.d/10-kubelet.conf

若 cluster dns或domain有改变的话，需要修改10-kubelet.conf。

最后建立 var 存放信息，然后启动 kubelet 服务:

	$ mkdir -p /var/lib/kubelet /var/log/kubernetes /var/lib/etcd
	
	$ systemctl enable kubelet.service && systemctl start kubelet.service

完成后会需要一段时间来下载镜像档与启动元件，可以利用该指令来监看：

	$ watch netstat -ntlpActive Internet connections (only servers)Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
	
	tcp        0      0 127.0.0.1:10248         0.0.0.0:*               LISTEN      10344/kubelet
	
	tcp        0      0 127.0.0.1:10251         0.0.0.0:*               LISTEN      11324/kube-schedule
	
	tcp        0      0 0.0.0.0:6443            0.0.0.0:*               LISTEN      11416/haproxy
	
	tcp        0      0 127.0.0.1:10252         0.0.0.0:*               LISTEN      11235/kube-controll
	
	tcp        0      0 0.0.0.0:9090            0.0.0.0:*               LISTEN      11416/haproxy
	
	tcp6       0      0 :::2379                 :::*                    LISTEN      10479/etcd
	
	tcp6       0      0 :::2380                 :::*                    LISTEN      10479/etcd
	
	tcp6       0      0 :::10255                :::*                    LISTEN      10344/kubelet
	
	tcp6       0      0 :::5443                 :::*                    LISTEN      11295/kube-apiserve

若看到以上信息表示服务正常启动，若发生问题可以用docker指令来查看。

## 验证集群 ##

完成后，在任意一台master节点复制 admin kubeconfig 文件，并通过简单指令验证：

	$ cp /etc/kubernetes/admin.conf ~/.kube/config
	
	$ kubectl get cs
	
	NAME                 STATUS    MESSAGE              ERROR
	
	controller-manager   Healthy   ok
	
	scheduler            Healthy   ok
	
	etcd-2               Healthy   {"health": "true"}
	
	etcd-1               Healthy   {"health": "true"}
	
	etcd-0               Healthy   {"health": "true"}
	
	
	
	$ kubectl get node
	
	NAME      STATUS     ROLES     AGE       VERSION
	
	k8s-m1    NotReady   master    52s       v1.10.0
	
	k8s-m2    NotReady   master    51s       v1.10.0
	
	k8s-m3    NotReady   master    50s       v1.10.0
	
	
	
	$ kubectl -n kube-system get po
	
	NAME                             READY     STATUS    RESTARTS   AGE
	
	etcd-k8s-m1                      1/1       Running   0          7s
	
	etcd-k8s-m2                      1/1       Running   0          57s
	
	haproxy-k8s-m3                   1/1       Running   0          1m...

接着确认服务能够执行 logs 等指令：

	$ kubectl -n kube-system logs -f kube-scheduler-k8s-m2Error from server (Forbidden): Forbidden (user=kube-apiserver, verb=get, resource=nodes, subresource=proxy) ( pods/log kube-scheduler-k8s-m2)

这边会发现出现 403 Forbidden 问题，这是因为 kube-apiserver user 并没有 nodes 的资源存取权限，属于正常。

由于上述权限问题，必需建立一个apiserver-to-kubelet-rbac.yml来定义权限，以供对 Nodes 容器执行 logs、exec 等指令。在任意一台master节点执行以下指令：

	$ kubectl apply -f "${CORE_URL}/apiserver-to-kubelet-rbac.yml.conf"
	
	clusterrole.rbac.authorization.k8s.io "system:kube-apiserver-to-kubelet" configured
	
	clusterrolebinding.rbac.authorization.k8s.io "system:kube-apiserver" configured
	
	# 测试 logs
	
	$ kubectl -n kube-system logs -f kube-scheduler-k8s-m2...
	
	I0403 02:30:36.375935       1 server.go:555] Version: v1.10.0
	
	I0403 02:30:36.378208       1 server.go:574] starting healthz server on 127.0.0.1:10251
	
	设定master节点允许 Taint：
	
	$ kubectl taint nodes node-role.kubernetes.io/master="":NoSchedule --all
	
	node "k8s-m1" tainted
	
	node "k8s-m2" tainted
	
	node "k8s-m3" tainted

[Taints and Tolerations](https://kubernetes.io/docs/concepts/configuration/taint-and-toleration/)。

## 建立 TLS Bootstrapping RBAC 与 Secret ##

由于本次安装启用了 TLS 认证，因此每个节点的 kubelet 都必须使用 kube-apiserver 的 CA 的凭证后，才能与 kube-apiserver 进行沟通，而该过程需要手动针对每台节点单独签署凭证是一件繁琐的事情，且一旦节点增加会延伸出管理不易问题; 而 TLS bootstrapping 目标就是解决该问题，通过让 kubelet 先使用一个预定低权限使用者连接到 kube-apiserver，然后在对 kube-apiserver 申请凭证签署，当授权 Token 一致时，Node 节点的 kubelet 凭证将由 kube-apiserver 动态签署提供。具体作法可以参考 [TLS Bootstrapping](https://kubernetes.io/docs/admin/kubelet-tls-bootstrapping/) 与 [Authenticating with Bootstrap Tokens](https://kubernetes.io/docs/admin/bootstrap-tokens/)。

首先在k8s-m1建立一个变量来产生BOOTSTRAP_TOKEN，并建立bootstrap-kubelet.conf的 Kubernetes config 档：

	$ cd /etc/kubernetes/pki
	
	$ export TOKEN_ID=$(openssl rand 3 -hex)
	
	$ export TOKEN_SECRET=$(openssl rand 8 -hex)
	
	$ export BOOTSTRAP_TOKEN=${TOKEN_ID}.${TOKEN_SECRET}
	
	$ export KUBE_APISERVER="https://192.16.35.10:6443"
	
	# bootstrap set cluster
	
	$ kubectl config set-cluster kubernetes \
	
	    --certificate-authority=ca.pem \
	
	    --embed-certs=true \
	
	    --server=${KUBE_APISERVER} \
	
	    --kubeconfig=../bootstrap-kubelet.conf
	
	# bootstrap set credentials
	
	$ kubectl config set-credentials tls-bootstrap-token-user \
	
	    --token=${BOOTSTRAP_TOKEN} \
	
	    --kubeconfig=../bootstrap-kubelet.conf
	
	# bootstrap set context
	
	$ kubectl config set-context tls-bootstrap-token-user@kubernetes \
	
	    --cluster=kubernetes \
	
	    --user=tls-bootstrap-token-user \
	
	    --kubeconfig=../bootstrap-kubelet.conf
	
	# bootstrap use default context
	
	$ kubectl config use-context tls-bootstrap-token-user@kubernetes \
	
	    --kubeconfig=../bootstrap-kubelet.conf

若想要用手动签署凭证来进行授权的话，可以参考 [Certificate](https://kubernetes.io/docs/concepts/cluster-administration/certificates/)。

接着在k8s-m1建立 TLS bootstrap secret 来提供自动签证使用：

	$ cat <<EOF | kubectl create -f -
	
	apiVersion: v1
	
	kind: Secret
	
	metadata:
	
	  name: bootstrap-token-${TOKEN_ID}
	
	  namespace: kube-system
	
	type: bootstrap.kubernetes.io/token
	
	stringData:
	
	  token-id: ${TOKEN_ID}
	
	  token-secret: ${TOKEN_SECRET}
	
	  usage-bootstrap-authentication: "true"
	
	  usage-bootstrap-signing: "true"
	
	  auth-extra-groups: system:bootstrappers:default-node-token
	
	EOF
	
	
	
	secret "bootstrap-token-65a3a9" created
	
	在k8s-m1建立 TLS Bootstrap Autoapprove RBAC：
	
	$ kubectl apply -f "${CORE_URL}/kubelet-bootstrap-rbac.yml.conf"
	
	clusterrolebinding.rbac.authorization.k8s.io "kubelet-bootstrap" created
	
	clusterrolebinding.rbac.authorization.k8s.io "node-autoapprove-bootstrap" created
	
	clusterrolebinding.rbac.authorization.k8s.io "node-autoapprove-certificate-rotation" created

## Kubernetes Nodes ##

本部分将说明如何建立与设定 Kubernetes Node 角色，Node 是主要执行容器实例(Pod)的工作节点。

在开始部署前，先在k8-m1将需要用到的文件复制到所有node节点上：

	$ cd /etc/kubernetes/pki
	
	$ for NODE in k8s-n1 k8s-n2 k8s-n3; do
	
	    echo "--- $NODE ---"
	
	    ssh ${NODE} "mkdir -p /etc/kubernetes/pki/"
	
	    ssh ${NODE} "mkdir -p /etc/etcd/ssl"
	
	    # Etcd
	
	    for FILE in etcd-ca.pem etcd.pem etcd-key.pem; do
	
	      scp /etc/etcd/ssl/${FILE} ${NODE}:/etc/etcd/ssl/${FILE}
	
	    done
	
	    # Kubernetes
	
	    for FILE in pki/ca.pem pki/ca-key.pem bootstrap-kubelet.conf; do
	
	      scp /etc/kubernetes/${FILE} ${NODE}:/etc/kubernetes/${FILE}
	
	    done
	
	  done

## 部署与设定 ##

在每台node节点下载kubelet.service相关文件来管理 kubelet：

	$ export CORE_URL="https://kairen.github.io/files/manual-v1.10/node"
	
	$ mkdir -p /etc/systemd/system/kubelet.service.d
	
	$ wget "${CORE_URL}/kubelet.service" -O /lib/systemd/system/kubelet.service
	
	$ wget "${CORE_URL}/10-kubelet.conf" -O /etc/systemd/system/kubelet.service.d/10-kubelet.conf

若 cluster dns或domain有改变的话，需要修改10-kubelet.conf。

最后建立 var 存放信息，然后启动 kubelet 服务:

	$ mkdir -p /var/lib/kubelet /var/log/kubernetes
	
	$ systemctl enable kubelet.service && systemctl start kubelet.service

## 验证集群 ##

完成后，在任意一台master节点并通过简单指令验证：

	$ kubectl get csr
	
	NAME                                                   AGE       REQUESTOR                 CONDITION
	
	csr-bvz9l                                              11m       system:node:k8s-m1        Approved,Issued
	
	csr-jwr8k                                              11m       system:node:k8s-m2        Approved,Issued
	
	csr-q867w                                              11m       system:node:k8s-m3        Approved,Issued
	
	node-csr-Y-FGvxZWJqI-8RIK_IrpgdsvjGQVGW0E4UJOuaU8ogk   17s       system:bootstrap:dca3e1   Approved,Issued
	
	node-csr-cnX9T1xp1LdxVDc9QW43W0pYkhEigjwgceRshKuI82c   19s       system:bootstrap:dca3e1   Approved,Issued
	
	node-csr-m7SBA9RAGCnsgYWJB-u2HoB2qLSfiQZeAxWFI2WYN7Y   18s       system:bootstrap:dca3e1   Approved,Issued
	
	
	
	$ kubectl get nodes
	
	NAME      STATUS     ROLES     AGE       VERSION
	
	k8s-m1    NotReady   master    12m       v1.10.0
	
	k8s-m2    NotReady   master    11m       v1.10.0
	
	k8s-m3    NotReady   master    11m       v1.10.0
	
	k8s-n1    NotReady   node      32s       v1.10.0
	
	k8s-n2    NotReady   node      31s       v1.10.0
	
	k8s-n3    NotReady   node      29s       v1.10.0

## Kubernetes Core Addons 部署 ##

当完成上面所有步骤后，接着需要部署一些插件，其中如Kubernetes DNS与Kubernetes Proxy等这种 Addons 是非常重要的。

## Kubernetes Proxy ##

[Kube-proxy](https://github.com/kubernetes/kubernetes/tree/master/cluster/addons/kube-proxy) 是实现 Service 的关键插件，kube-proxy 会在每台节点上执行，然后监听 API Server 的 Service 与 Endpoint 资源物件的改变，然后来依据变化执行 iptables 来实现网络的转发。这边我们会需要建议一个 DaemonSet 来执行，并且建立一些需要的 Certificates。

在k8s-m1下载kube-proxy.yml来建立 Kubernetes Proxy Addon：

	$ kubectl apply -f "https://kairen.github.io/files/manual-v1.10/addon/kube-proxy.yml.conf"
	
	serviceaccount "kube-proxy" created
	
	clusterrolebinding.rbac.authorization.k8s.io "system:kube-proxy" created
	
	configmap "kube-proxy" created
	
	daemonset.apps "kube-proxy" created
	
	
	
	$ kubectl -n kube-system get po -o wide -l k8s-app=kube-proxy
	
	NAME               READY     STATUS    RESTARTS   AGE       IP             NODE
	
	kube-proxy-8j5w8   1/1       Running   0          29s       192.16.35.16   k8s-n3
	
	kube-proxy-c4zvt   1/1       Running   0          29s       192.16.35.11   k8s-m1
	
	kube-proxy-clpl6   1/1       Running   0          29s       192.16.35.12   k8s-m2...

## Kubernetes DNS ##

[Kube DNS](https://github.com/kubernetes/kubernetes/tree/master/cluster/addons/dns) 是 Kubernetes 集群内部 Pod 之间互相沟通的重要 Addon，它允许 Pod 可以通过 Domain Name 方式来连接 Service，其主要由 Kube DNS 与 Sky DNS 组合而成，通过 Kube DNS 监听 Service 与 Endpoint 变化，来提供给 Sky DNS 信息，已更新解析位址。

在k8s-m1下载kube-proxy.yml来建立 Kubernetes Proxy Addon：

	$ kubectl apply -f "https://kairen.github.io/files/manual-v1.10/addon/kube-dns.yml.conf"
	
	serviceaccount "kube-dns" created
	
	service "kube-dns" created
	
	deployment.extensions "kube-dns" created
	
	
	
	$ kubectl -n kube-system get po -l k8s-app=kube-dns
	
	NAME                        READY     STATUS    RESTARTS   AGE
	
	kube-dns-654684d656-zq5t8   0/3       Pending   0          1m

这边会发现处于Pending状态，是由于 Kubernetes Pod Network 还未建立完成，因此所有节点会处于NotReady状态，而造成 Pod 无法被排程分配到指定节点上启动，由于为了解决该问题，下节将说明如何建立 Pod Network。

## Calico Network 安装与设定 ##

Calico 是一款纯 Layer 3 的资料中心网络方案(不需要 Overlay 网络)，Calico 好处是它整合了各种云原生平台，且 Calico 在每一个节点利用 Linux Kernel 实现高效的 vRouter 来负责资料的转发，而当资料中心复杂度增加时，可以用 BGP route reflector 来达成。

本次不采用手动方式来建立 Calico 网络，若想了解可以参考 [Integration Guide](https://docs.projectcalico.org/v3.0/getting-started/kubernetes/installation/integration)。

在k8s-m1下载calico.yaml来建立 Calico Network：
	
	$ kubectl apply -f "https://kairen.github.io/files/manual-v1.10/network/calico.yml.conf"
	
	configmap "calico-config" created
	
	daemonset "calico-node" created
	
	deployment "calico-kube-controllers" created
	
	clusterrolebinding "calico-cni-plugin" created
	
	clusterrole "calico-cni-plugin" created
	
	serviceaccount "calico-cni-plugin" created
	
	clusterrolebinding "calico-kube-controllers" created
	
	clusterrole "calico-kube-controllers" created
	
	serviceaccount "calico-kube-controllers" created
	
	
	
	$ kubectl -n kube-system get po -l k8s-app=calico-node -o wide
	
	NAME                READY     STATUS    RESTARTS   AGE       IP             NODE
	
	calico-node-22mbb   2/2       Running   0          1m        192.16.35.12   k8s-m2
	
	calico-node-2qwf5   2/2       Running   0          1m        192.16.35.11   k8s-m1
	
	calico-node-g2sp8   2/2       Running   0          1m        192.16.35.13   k8s-m3
	
	calico-node-hghp4   2/2       Running   0          1m        192.16.35.14   k8s-n1
	
	calico-node-qp6gf   2/2       Running   0          1m        192.16.35.15   k8s-n2
	
	calico-node-zfx4n   2/2       Running   0          1m        192.16.35.16   k8s-n3

这边若节点 IP 与网卡不同的话，请修改calico.yml文件。

在k8s-m1下载 Calico CLI 来查看 Calico nodes:

	$ wget https://github.com/projectcalico/calicoctl/releases/download/v3.1.0/calicoctl -O /usr/local/bin/calicoctl
	
	$ chmod u+x /usr/local/bin/calicoctl
	
	$ cat <<EOF > ~/calico-rcexport ETCD_ENDPOINTS="https://192.16.35.11:2379,https://192.16.35.12:2379,https://192.16.35.13:2379"export ETCD_CA_CERT_FILE="/etc/etcd/ssl/etcd-ca.pem"export ETCD_CERT_FILE="/etc/etcd/ssl/etcd.pem"export ETCD_KEY_FILE="/etc/etcd/ssl/etcd-key.pem"
	
	EOF
	
	
	
	$ . ~/calico-rc
	
	$ calicoctl node statusCalico process is running.
	
	IPv4 BGP status+--------------+-------------------+-------+----------+-------------+| PEER ADDRESS |     PEER TYPE     | STATE |  SINCE   |    INFO     |+--------------+-------------------+-------+----------+-------------+| 192.16.35.12 | node-to-node mesh | up    | 04:42:37 | Established || 192.16.35.13 | node-to-node mesh | up    | 04:42:42 | Established || 192.16.35.14 | node-to-node mesh | up    | 04:42:37 | Established || 192.16.35.15 | node-to-node mesh | up    | 04:42:41 | Established || 192.16.35.16 | node-to-node mesh | up    | 04:42:36 | Established |+--------------+-------------------+-------+----------+-------------+...

查看 pending 的 pod 是否已执行：
	
	$ kubectl -n kube-system get po -l k8s-app=kube-dns
	
	kubectl -n kube-system get po -l k8s-app=kube-dns
	
	NAME                        READY     STATUS    RESTARTS   AGE
	
	kube-dns-654684d656-j8xzx   3/3       Running   0          10m

## Kubernetes Extra Addons 部署 ##

本节说明如何部署一些官方常用的 Addons，如 Dashboard、Heapster 等。

### Dashboard ###

[Dashboard](https://github.com/kubernetes/dashboard) 是 Kubernetes 社区官方开发的仪表板，有了仪表板后管理者就能够通过 Web-based 方式来管理 Kubernetes 集群，除了提升管理方便，也让资源视觉化，让人更直觉看见系统信息的呈现结果。

在k8s-m1通过 kubectl 来建立 kubernetes dashboard 即可：

	$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/master/src/deploy/recommended/kubernetes-dashboard.yaml
	
	$ kubectl -n kube-system get po,svc -l k8s-app=kubernetes-dashboard
	
	NAME                                    READY     STATUS    RESTARTS   AGE
	
	kubernetes-dashboard-7d5dcdb6d9-j492l   1/1       Running   0          12s
	
		
	NAME                   TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
	
	kubernetes-dashboard   ClusterIP   10.111.22.111   <none>        443/TCP   12s

这边会额外建立一个名称为open-api Cluster Role Binding，这仅作为方便测试时使用，在一般情况下不要开启，不然就会直接被存取所有 API:

	$ cat <<EOF | kubectl create -f -
	
	apiVersion: rbac.authorization.k8s.io/v1
	
	kind: ClusterRoleBinding
	
	metadata:
	
	  name: open-api
	
	  namespace: ""
	
	roleRef:
	
	  apiGroup: rbac.authorization.k8s.io
	
	  kind: ClusterRole
	
	  name: cluster-admin
	
	subjects:
	
	  - apiGroup: rbac.authorization.k8s.io
	
	    kind: User
	
	    name: system:anonymous
	
	EOF

注意!管理者可以针对特定使用者来开放 API 存取权限，但这边方便使用直接绑在 cluster-admin cluster role。

完成后，就可以通过浏览器存取 Dashboard。

在 1.7 版本以后的 Dashboard 将不再提供所有权限，因此需要建立一个 service account 来绑定 cluster-admin role：

	$ kubectl -n kube-system create sa dashboard
	
	$ kubectl create clusterrolebinding dashboard --clusterrole cluster-admin --serviceaccount=kube-system:dashboard
	
	$ SECRET=$(kubectl -n kube-system get sa dashboard -o yaml | awk '/dashboard-token/ {print $3}')
	
	$ kubectl -n kube-system describe secrets ${SECRET} | awk '/token:/{print $2}'
	
	eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJkYXNoYm9hcmQtdG9rZW4tdzVocmgiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC5uYW1lIjoiZGFzaGJvYXJkIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQudWlkIjoiYWJmMTFjYzMtZjRlYi0xMWU3LTgzYWUtMDgwMDI3NjdkOWI5Iiwic3ViIjoic3lzdGVtOnNlcnZpY2VhY2NvdW50Omt1YmUtc3lzdGVtOmRhc2hib2FyZCJ9.Xuyq34ci7Mk8bI97o4IldDyKySOOqRXRsxVWIJkPNiVUxKT4wpQZtikNJe2mfUBBD-JvoXTzwqyeSSTsAy2CiKQhekW8QgPLYelkBPBibySjBhJpiCD38J1u7yru4P0Pww2ZQJDjIxY4vqT46ywBklReGVqY3ogtUQg-eXueBmz-o7lJYMjw8L14692OJuhBjzTRSaKW8U2MPluBVnD7M2SOekDff7KpSxgOwXHsLVQoMrVNbspUCvtIiEI1EiXkyCNRGwfnd2my3uzUABIHFhm0_RZSmGwExPbxflr8Fc6bxmuz-_jSdOtUidYkFIzvEWw2vRovPgs3MXTv59RwUw

复制token，然后贴到 Kubernetes dashboard。注意这边一般来说要针对不同 User 开启特定存取权限。



### Heapster ###

[Heapster](https://github.com/kubernetes/heapster) 是 Kubernetes 社区维护的容器集群监控与效能分析工具。Heapster 会从 Kubernetes apiserver 取得所有 Node 信息，然后再通过这些 Node 来取得 kubelet 上的资料，最后再将所有收集到资料送到 Heapster 的后台储存 InfluxDB，最后利用 Grafana 来抓取 InfluxDB 的资料源来进行视觉化。

在k8s-m1通过 kubectl 来建立 kubernetes monitor 即可：

	$ kubectl apply -f "https://kairen.github.io/files/manual-v1.10/addon/kube-monitor.yml.conf"
	
	$ kubectl -n kube-system get po,svc
	
	NAME                                           READY     STATUS    RESTARTS   AGE...
	
	po/heapster-74fb5c8cdc-62xzc                   4/4       Running   0          7m
	
	po/influxdb-grafana-55bd7df44-nw4nc            2/2       Running   0          7m
	
	
	
	NAME                       TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)             AGE...
	
	svc/heapster               ClusterIP   10.100.242.225   <none>        80/TCP              7m
	
	svc/monitoring-grafana     ClusterIP   10.101.106.180   <none>        80/TCP              7m
	
	svc/monitoring-influxdb    ClusterIP   10.109.245.142   <none>        8083/TCP,8086/TCP   7m···

完成后，就可以通过浏览器存取 Grafana Dashboard。



### Ingress Controller ###
[Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/)是利用 Nginx 或 HAProxy 等负载平衡器来曝露集群内服务的元件，Ingress 主要通过设定 Ingress 规格来定义 Domain Name 映射 Kubernetes 内部 Service，这种方式可以避免掉使用过多的 NodePort 问题。

在k8s-m1通过 kubectl 来建立 Ingress Controller 即可：

	$ kubectl create ns ingress-nginx
	
	$ kubectl apply -f "https://kairen.github.io/files/manual-v1.10/addon/ingress-controller.yml.conf"
	
	$ kubectl -n ingress-nginx get po
	
	NAME                                       READY     STATUS    RESTARTS   AGEdefault-http-backend-5c6d95c48-rzxfb       1/1       Running   0          7m
	
	nginx-ingress-controller-699cdf846-982n4   1/1       Running   0          7m

这里也可以选择 [Traefik](https://github.com/containous/traefik) 的 Ingress Controller。

测试 Ingress 功能
这边先建立一个 Nginx HTTP server Deployment 与 Service：

	$ kubectl run nginx-dp --image nginx --port 80
	
	$ kubectl expose deploy nginx-dp --port 80
	
	$ kubectl get po,svc
	
	$ cat <<EOF | kubectl create -f -
	
	apiVersion: extensions/v1beta1
	
	kind: Ingress
	
	metadata:
	
	  name: test-nginx-ingress
	
	  annotations:
	
	    ingress.kubernetes.io/rewrite-target: /
	
	spec:
	
	  rules:
	
	  - host: test.nginx.com
	
	    http:
	
	      paths:
	
	      - path: /
	
	        backend:
	
	          serviceName: nginx-dp
	
	          servicePort: 80
	
	EOF

通过 curl 来进行测试：

	$ curl 192.16.35.10 -H 'Host: test.nginx.com'<!DOCTYPE html><html><head><title>Welcome to nginx!</title>...

# 测试其他 domain name 是否会回传 404

	$ curl 192.16.35.10 -H 'Host: test.nginx.com1'default backend - 404

### Helm Tiller Server ###

[Helm](https://github.com/kubernetes/helm) 是 Kubernetes Chart 的管理工具，Kubernetes Chart 是一套预先组态的 Kubernetes 资源套件。其中Tiller Server主要负责接收来至 Client 的指令，并通过 kube-apiserver 与 Kubernetes 集群做沟通，根据 Chart 定义的内容，来产生与管理各种对应 API 物件的 Kubernetes 部署文档(又称为 Release)。

首先在k8s-m1安装 Helm tool：

	$ wget -qO- https://kubernetes-helm.storage.googleapis.com/helm-v2.8.1-linux-amd64.tar.gz | tar -zx
	
	$ sudo mv linux-amd64/helm /usr/local/bin/

另外在所有node节点安装 socat：

	$ sudo apt-get install -y socat

接着初始化 Helm(这边会安装 Tiller Server)：

	$ kubectl -n kube-system create sa tiller
	
	$ kubectl create clusterrolebinding tiller --clusterrole cluster-admin --serviceaccount=kube-system:tiller
	
	$ helm init --service-account tiller...Tiller (the Helm server-side component) has been installed into your Kubernetes Cluster.Happy Helming!
	
	
	
	$ kubectl -n kube-system get po -l app=helm
	
	NAME                             READY     STATUS    RESTARTS   AGE
	
	tiller-deploy-5f789bd9f7-tzss6   1/1       Running   0          29s
	
	
	
	$ helm versionClient: &version.Version{SemVer:"v2.8.1", GitCommit:"6af75a8fd72e2aa18a2b278cfe5c7a1c5feca7f2", GitTreeState:"clean"}Server: &version.Version{SemVer:"v2.8.1", GitCommit:"6af75a8fd72e2aa18a2b278cfe5c7a1c5feca7f2", GitTreeState:"clean"}

测试 [Helm](https://www.kubernetes.org.cn/tags/helm) 功能
这边部署简单 Jenkins 来进行功能测试：

	$ helm install --name demo --set Persistence.Enabled=false stable/jenkins
	
	$ kubectl get po,svc  -l app=demo-jenkins
	
	NAME                           READY     STATUS    RESTARTS   AGE
	
	demo-jenkins-7bf4bfcff-q74nt   1/1       Running   0          2m
	
	
	
	NAME                 TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
	
	demo-jenkins         LoadBalancer   10.103.15.129    <pending>     8080:31161/TCP   2m
	
	demo-jenkins-agent   ClusterIP      10.103.160.126   <none>        50000/TCP        2m

# 取得 admin 账号的密码

	$ printf $(kubectl get secret --namespace default demo-jenkins -o jsonpath="{.data.jenkins-admin-password}" | base64 --decode);echo
	
	r6y9FMuF2u

完成后，就可以通过浏览器存取 Jenkins Web。



测试完成后，即可删除：

	$ helm ls
	
	NAME    REVISION    UPDATED                     STATUS      CHART             NAMESPACE
	
	demo    1           Tue Apr 10 07:29:51 2018    DEPLOYED    jenkins-0.14.4    default
	
	
	
	$ helm delete demo --purge
	
	release "demo" deleted

更多 Helm Apps 可以到 [Kubeapps Hub](https://hub.kubeapps.com/) 寻找。

## 测试集群 ##

SSH 进入k8s-m1节点，然后关闭该节点：

	$ sudo poweroff

接着进入到k8s-m2节点，通过 kubectl 来检查集群是否能够正常执行：

	# 先检查 etcd 状态，可以发现 etcd-0 因为关机而中断
	
	$ kubectl get cs
	
	NAME                 STATUS      MESSAGE                                                                                                                                          ERROR
	
	scheduler            Healthy     ok
	
	controller-manager   Healthy     ok
	
	etcd-1               Healthy     {"health": "true"}
	
	etcd-2               Healthy     {"health": "true"}
	
	etcd-0               Unhealthy   Get https://192.16.35.11:2379/health: net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers)
	
	# 测试是否可以建立 Pod
	
	$ kubectl run nginx --image nginx --restart=Never --port 80
	
	$ kubectl get po
	
	NAME      READY     STATUS    RESTARTS   AGE
	
	nginx     1/1       Running   0          22s


**注：该文章由[k8s中文网](https://www.kubernetes.org.cn/3814.html)搬运而来**