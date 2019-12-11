### CentOS7 安装K8S

#### 设置源
~~~
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
exclude=kube*
EOF
~~~


#### 关闭SeLinux
~~~
# Set SELinux in permissive mode (effectively disabling it)
setenforce 0
sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config
~~~


#### 安装Docker
~~~
yum install -y yum-utils device-mapper-persistent-data lvm2
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
yum list docker-ce --showduplicates | sort -r
yum -y install docker-ce
docker --version
systemctl start docker
systemctl status docker
systemctl enable docker
~~~

#### 安装etcd
~~~
wget https://storage.googleapis.com/etcd/v3.2.9/etcd-v3.2.9-linux-amd64.tar.gz
tar -zxvf etcd-v3.2.9-linux-amd64.tar.gz
cp etcd-v3.2.9-linux-amd64/etcd* /usr/bin/

etcd --version
etcdctl --version
~~~

#### 安装Kubernetes
~~~
wget https://dl.k8s.io/v1.13.0/kubernetes-server-linux-amd64.tar.gz
tar -zxvf kubernetes-server-linux-amd64.tar.gz
cd kubernetes/server/bin/
cp kubectl kube-apiserver kube-scheduler kube-controller-manager kubelet kube-proxy /usr/bin/       
kube-apiserver --version
~~~

#### 安装 flanneld
~~~
curl -L https://github.com/coreos/flannel/releases/download/v0.9.0/flannel-v0.9.0-linux-amd64.tar.gz -o flannel-v0.9.0-linux-amd64.tar.gz
tar -zxvf flannel-v0.9.0-linux-amd64.tar.gz
mv flanneld /usr/bin/
mkdir /usr/libexec/flannel/
mv mk-docker-opts.sh /usr/libexec/flannel/
flanneld --version
~~~


#### 配置并启用 etcd

A. 配置启动项
~~~
vim /usr/lib/systemd/system/etcd.service

[Unit]
Description=etcd
After=network.target
After=network-online.target
Wants=network-online.target
Documentation=https://github.com/coreos/etcd
[Service]
Type=notify
WorkingDirectory=/var/lib/etcd
EnvironmentFile=-/etc/etcd/etcd.conf
ExecStart=/usr/bin/etcd --config-file /etc/etcd/etcd.conf
Restart=on-failure
LimitNOFILE=65536
Restart=on-failure
RestartSec=5
LimitNOFILE=65536
[Install]
WantedBy=multi-user.target
~~~

B. 配置各节点 etcd.conf 配置文件
~~~
mkdir -p /var/lib/etcd/
mkdir -p /etc/etcd/
export ETCD_NAME=etcd
export INTERNAL_IP=192.168.100.104

cat << EOF > /etc/etcd/etcd.conf 
name: '${ETCD_NAME}'
data-dir: "/var/lib/etcd/"
listen-peer-urls: http://${INTERNAL_IP}:2380
listen-client-urls: http://${INTERNAL_IP}:2379,http://127.0.0.1:2379
initial-advertise-peer-urls: http://${INTERNAL_IP}:2380
advertise-client-urls: http://${INTERNAL_IP}:2379
initial-cluster: "etcd=http://${INTERNAL_IP}:2380"
initial-cluster-token: 'etcd-cluster'
initial-cluster-state: 'new'
EOF
~~~
注:
   new-----初始化集群安装时使用该选项；
   existing-----新加入集群时使用该选项。

C. 启动 etcd
~~~
systemctl start etcd
systemctl status etcd
systemctl enable etcd

# 查看集群成员
etcdctl member list

#查看集群健康状况
etcdctl cluster-health
~~~


#### 配置并启用flanneld


A. 配置启动项
~~~
vim /usr/lib/systemd/system/flanneld.service

[Unit]
Description=Flanneld overlay address etcd agent
After=network.target
After=network-online.target
Wants=network-online.target
After=etcd.service
Before=docker.service
[Service]
Type=notify
EnvironmentFile=/etc/sysconfig/flanneld
EnvironmentFile=-/etc/sysconfig/docker-network
ExecStart=/usr/bin/flanneld-start $FLANNEL_OPTIONS
ExecStartPost=/usr/libexec/flannel/mk-docker-opts.sh -k DOCKER_NETWORK_OPTIONS -d /run/flannel/docker
Restart=on-failure
[Install]
WantedBy=multi-user.target
RequiredBy=docker.service


vim /usr/bin/flanneld-start 

#!/bin/sh
exec /usr/bin/flanneld \
        -etcd-endpoints=${FLANNEL_ETCD_ENDPOINTS:-${FLANNEL_ETCD}} \
        -etcd-prefix=${FLANNEL_ETCD_PREFIX:-${FLANNEL_ETCD_KEY}} \
        "$@"

chmod 755 /usr/bin/flanneld-start
~~~


B. 配置 flannel 配置文件
~~~
etcdctl mkdir /kube/network
etcdctl set /kube/network/config '{ "Network": "10.254.0.0/16" }'

vim /etc/sysconfig/flanneld

FLANNEL_ETCD_ENDPOINTS="http://192.168.100.104:2379"
FLANNEL_ETCD_PREFIX="/kube/network"
~~~

C. 启动 flanneld
~~~
systemctl start flanneld
systemctl status flanneld
systemctl enable flanneld
~~~

D. 查看各节点网段
~~~
cat /var/run/flannel/subnet.env
~~~

E. 更改 docker 网段为 flannel 分配的网段
~~~
export FLANNEL_SUBNET=10.254.26.1/24

cat << EOF > /etc/docker/daemon.json
{
  "bip" : "$FLANNEL_SUBNET"
}
EOF

systemctl daemon-reload
systemctl restart docker
~~~

F. 查看是否已分配相应网段
~~~
ip route show
~~~

G. 使用 etcdctl 命令查看 flannel 的相关信息

~~~
etcdctl ls /kube/network/subnets

etcdctl -o extended  get /kube/network/subnets/10.254.26.0-24
~~~

H. 测试网络是否正常
~~~
ping -c 4 10.254.26.1
~~~


##

#### 配置并启用 Kubernetes Master 节点

Kubernetes Master 节点包含的组件:
	kube-apiserver
	kube-scheduler
	kube-controller-manager


A. 配置 config 文件
~~~
mkdir -p /etc/kubernetes/

vim /etc/kubernetes/config

KUBE_LOGTOSTDERR="--logtostderr=true"
KUBE_LOG_LEVEL="--v=0"
KUBE_ALLOW_PRIV="--allow-privileged=false"
KUBE_MASTER="--master=http://192.168.100.104:8080"
~~~

B. 配置 kube-apiserver 启动项
~~~
vim /usr/lib/systemd/system/kube-apiserver.service 

[Unit]
Description=Kubernetes API Server
Documentation=https://github.com/kubernetes/kubernetes
After=network.target
After=etcd.service
[Service]
EnvironmentFile=-/etc/kubernetes/config
EnvironmentFile=-/etc/kubernetes/apiserver
ExecStart=/usr/bin/kube-apiserver \
            $KUBE_LOGTOSTDERR \
            $KUBE_LOG_LEVEL \
            $KUBE_ETCD_SERVERS \
            $KUBE_API_ADDRESS \
            $KUBE_API_PORT \
            $KUBELET_PORT \
            $KUBE_ALLOW_PRIV \
            $KUBE_SERVICE_ADDRESSES \
            $KUBE_ADMISSION_CONTROL \
            $KUBE_API_ARGS
Restart=on-failure
Type=notify
LimitNOFILE=65536
[Install]
WantedBy=multi-user.target
~~~


C. 配置 apiserver 配置文件
~~~
vim /etc/kubernetes/apiserver

KUBE_API_ADDRESS="--advertise-address=192.168.100.104 --bind-address=192.168.100.104 --insecure-bind-address=0.0.0.0"
KUBE_ETCD_SERVERS="--etcd-servers=http://192.168.100.104:2379"
KUBE_SERVICE_ADDRESSES="--service-cluster-ip-range=10.254.0.0/16"
KUBE_ADMISSION_CONTROL="--admission-control=NamespaceLifecycle,NamespaceExists,LimitRanger,SecurityContextDeny,ResourceQuota"
KUBE_API_ARGS="--enable-swagger-ui=true --apiserver-count=3 --audit-log-maxage=30 --audit-log-maxbackup=3 --audit-log-maxsize=100 --audit-log-path=/var/log/apiserver.log"
~~~

注:使用 HTTP 和 使用 HTTPS 的最大不同就是--admission-control=ServiceAccount选项。


D. 启动 kube-apiserver
~~~
systemctl start kube-apiserver
systemctl status kube-apiserver
systemctl enable kube-apiserver
~~~

E. 配置 kube-controller-manager 启动项
~~~
vim /usr/lib/systemd/system/kube-controller-manager.service

[Unit]
Description=Kubernetes Controller Manager
Documentation=https://github.com/kubernetes/kubernetes
[Service]
EnvironmentFile=-/etc/kubernetes/config
EnvironmentFile=-/etc/kubernetes/controller-manager
ExecStart=/usr/bin/kube-controller-manager \
            $KUBE_LOGTOSTDERR \
            $KUBE_LOG_LEVEL \
            $KUBE_MASTER \
            $KUBE_CONTROLLER_MANAGER_ARGS
Restart=on-failure
LimitNOFILE=65536
[Install]
WantedBy=multi-user.target
~~~

F. 配置 kube-controller-manager 配置文件
~~~
vim /etc/kubernetes/controller-manager

KUBE_CONTROLLER_MANAGER_ARGS="--address=0.0.0.0 --service-cluster-ip-range=10.254.0.0/16 --cluster-name=kubernetes"
~~~


G.启动 kube-controller-manager
~~~
systemctl start kube-controller-manager
systemctl status kube-controller-manager
systemctl enable kube-controller-manager
~~~


H. 配置 kube-scheduler 启动项
~~~
vim /usr/lib/systemd/system/kube-scheduler.service 

[Unit]
Description=Kubernetes Scheduler Plugin
Documentation=https://github.com/kubernetes/kubernetes
[Service]
EnvironmentFile=-/etc/kubernetes/config
EnvironmentFile=-/etc/kubernetes/scheduler
ExecStart=/usr/bin/kube-scheduler \
            $KUBE_LOGTOSTDERR \
            $KUBE_LOG_LEVEL \
            $KUBE_MASTER \
            $KUBE_SCHEDULER_ARGS
Restart=on-failure
LimitNOFILE=65536
[Install]
WantedBy=multi-user.target
~~~

I. 配置 kube-scheduler 配置文件
~~~
vim /etc/kubernetes/scheduler
KUBE_SCHEDULER_ARGS="--address=0.0.0.0"
~~~

J. 启动 kube-scheduler
~~~
systemctl start kube-scheduler
systemctl status kube-scheduler
systemctl enable kube-scheduler
~~~

K. 验证 Master 节点
~~~
kubectl get componentstatuses
kubectl get cs
~~~


#### 配置并启用 Kubernetes Node 节点 

Kubernetes Node 节点包含如下组件:
  kubelet
  kube-proxy

A. 配置 kubelet 启动项
~~~
vim /usr/lib/systemd/system/kubelet.service 

[Unit]
Description=Kubernetes Kubelet Server
Documentation=https://github.com/kubernetes/kubernetes
After=docker.service
Requires=docker.service
[Service]
WorkingDirectory=/var/lib/kubelet
EnvironmentFile=-/etc/kubernetes/config
EnvironmentFile=-/etc/kubernetes/kubelet
ExecStart=/usr/bin/kubelet \
            $KUBE_LOGTOSTDERR \
            $KUBE_LOG_LEVEL \
            $KUBELET_ADDRESS \
            $KUBELET_PORT \
            $KUBELET_HOSTNAME \
            $KUBE_ALLOW_PRIV \
            $KUBELET_POD_INFRA_CONTAINER \
            $KUBELET_ARGS
Restart=on-failure
[Install]
WantedBy=multi-user.target
~~~

B. 配置 kubelet 配置文件
~~~
mkdir -p /var/lib/kubelet
export MASTER_ADDRESS=192.168.100.104
export KUBECONFIG_DIR=/etc/kubernetes

cat <<EOF > "${KUBECONFIG_DIR}/kubelet.kubeconfig"
apiVersion: v1
kind: Config
clusters:
  - cluster:
      server: http://${MASTER_ADDRESS}:8080/
    name: local
contexts:
  - context:
      cluster: local
    name: local
current-context: local
EOF


vim /etc/kubernetes/kubelet

KUBELET_ADDRESS="--address=0.0.0.0"
KUBELET_PORT="--port=10250"
KUBELET_HOSTNAME="--hostname-override=XXXXX"
KUBELET_POD_INFRA_CONTAINER="--pod-infra-container-p_w_picpath=hub.c.163.com/k8s163/pause-amd64:3.0"
KUBELET_ARGS="--kubeconfig=/etc/kubernetes/kubelet.kubeconfig --fail-swap-on=false --cluster-dns=10.254.0.2"

#--cluster-dns=10.254.0.2 此处DNS 一定是以.2为结尾的

# 下载镜像
docker pull daocloud.io/daocloud/google_containers_pause-amd64:3.1
~~~

C. 启动 kubelet
~~~
systemctl start kubelet
systemctl status kubelet
systemctl enable kubelet
~~~


D. 配置 kube-proxy 启动项
~~~
vim /usr/lib/systemd/system/kube-proxy.service 

[Unit]
Description=Kubernetes Kube-Proxy Server
Documentation=https://github.com/kubernetes/kubernetes
After=network.target
[Service]
EnvironmentFile=-/etc/kubernetes/config
EnvironmentFile=-/etc/kubernetes/proxy
ExecStart=/usr/bin/kube-proxy \
            $KUBE_LOGTOSTDERR \
            $KUBE_LOG_LEVEL \
            $KUBE_MASTER \
            $KUBE_PROXY_ARGS
Restart=on-failure
LimitNOFILE=65536
[Install]
WantedBy=multi-user.target
~~~


E. 配置 kube-proxy 配置文件
~~~
vim /etc/kubernetes/proxy 
KUBE_PROXY_ARGS="--bind-address=192.168.100.104 --hostname-override=192.168.100.104 --cluster-cidr=10.254.0.0/16"
~~~

F. 启动 kube-proxy
~~~
systemctl start kube-proxy
systemctl status kube-proxy
systemctl enable kube-proxy
~~~



##部署 Kubernetes Dashboard
参考： https://www.wuweixin.com/2017/05/20/kubernetes-dashboard-deploy/

~~~

wget https://raw.githubusercontent.com/kubernetes/dashboard/v1.5.1/src/deploy/kubernetes-dashboard.yaml

# 修改 apiserver-host
- --apiserver-host=http://192.168.0.1:8080

# 创建实例
kubectl create -f kubernetes-dashboard.yaml
~~~

# 创建错误可通过删除命令
kubectl delete -f kubernetes-dashboard.yaml

~~~
# 查看所有实例
kubectl get all --all-namespaces

# 描述容器
kubectl describe pods deploy/kubernetes-dashboard
kubectl describe service deploy/kubernetes-dashboard
~~~



#获取所有namespace
kubectl get namespace

#获取指定namespace下面的pods
kubectl --namespace=kube-public get pods

# 描述指定pod
kubectl describe pod <PodName> --namespace=<NAMESPACE>

# 获取proxy对外映射的访问地址
kubectl cluster-info




#Master Node上
6443*，Kubernetes API server
2379-2380， etcd server client API
10250， Kubelet API
10251，kube-scheduler
10252，kube-controller-manager
10255，Read-only Kubelet API

#worker node上需要以下TCP端口：
10250，Kubelet API
10255，Read-only Kubelet API
30000-32767，NodePort Services**
