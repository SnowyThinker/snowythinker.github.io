# CentOS6 安装Greenplum

[官方安装文档](https://gpdb.docs.pivotal.io/6-0/install_guide/prep_os.html)

## 节点规划

~~~shell
# 三台机
master  主节点
node1   segment 节点
node2   segment 节点
~~~

## 环境配置

### 配置主机名与host映射（此步省略）

### 关闭防火墙（此步省略）

### 依次修改三个节点 /etc/sysctl.conf

~~~shell
# kernel.shmall = _PHYS_PAGES / 2 # See Note 1
kernel.shmall = 4000000000
# kernel.shmmax = kernel.shmall * PAGE_SIZE # See Note 1
kernel.shmmax = 500000000
kernel.shmmni = 4096
vm.overcommit_memory = 2
vm.overcommit_ratio = 95 # See Note 2
net.ipv4.ip_local_port_range = 10000 65535 # See Note 3
kernel.sem = 500 2048000 200 40960
kernel.sysrq = 1
kernel.core_uses_pid = 1
kernel.msgmnb = 65536
kernel.msgmax = 65536
kernel.msgmni = 2048
net.ipv4.tcp_syncookies = 1
net.ipv4.conf.default.accept_source_route = 0
net.ipv4.tcp_max_syn_backlog = 4096
net.ipv4.conf.all.arp_filter = 1
net.core.netdev_max_backlog = 10000
net.core.rmem_max = 2097152
net.core.wmem_max = 2097152
vm.swappiness = 10
vm.zone_reclaim_mode = 0
vm.dirty_expire_centisecs = 500
vm.dirty_writeback_centisecs = 100
vm.dirty_background_ratio = 0 # See Note 5
vm.dirty_ratio = 0
vm.dirty_background_bytes = 1610612736
vm.dirty_bytes = 4294967296
~~~

~~~shell
sysctl -p
~~~

## 安装

### 下载安装包

~~~shell
curl https://github.com/greenplum-db/gpdb/releases/download/6.0.1/greenplum-db-6.0.1-rhel6-x86_64.rpm
~~~

### 安装RPM包

~~~shell
rpm -ivh greenplum-db-6.0.1-rhel6-x86_64.rpm
~~~

### 配置环境变量

~~~shell
vim /etc/profile

GPHOME=/usr/local/greenplum-db
EXPORT path="$GPHOME/bin:$PATH"

source /etc/profile

vim /usr/local/greenplum-db/greenplum_path.sh

# 修改 GPHOME
GPHOME=$GPHOME
~~~

### 配置节点文件

~~~shell
cd /usr/local/greenplum-db

vi hostfile_exkeys

# 包含Master和Segment
master
node1
node2


vi hostfile_segments

# 只包含Segment
node1
node2
~~~

### Master节点添加gpadmin账户

~~~shell
groupadd gpadmin
useradd gpadmin -r -m -g gpadmin
passwd gpadmin


chown -R gpadmin:gpadmin /usr/local/greenplum-db-6.0.1
chown -R gpadmin:gpadmin /usr/local/greenplum-db
~~~

### 打通各节点 用户 gpadmin SSH登录

~~~shell
# 用 gpadmin 用户登录master节点执行

# 此步骤一定不能省略，如省略会报错（unable to import module: no module named gppylib.commands）
source /usr/local/greenplum-db/greenplum_path.sh
gpssh-exkeys -f hostfile_exkeys
~~~


### 安装后检查

~~~shell
gpssh -f hostfile_exkeys -v -e 'ntpd'
~~~

### 配置数据库

~~~shell
# 切换到 gpadmin用户
su - gpadmin

# 以gpinitsystem_config文件为模板
cp $GPHOME/docs/cli_help/gpconfigs/gpinitsystem_config ./gpinit_config

# 配置gpinit_config
vi gpinit_config

# 修改如下配置，每个segment上1个数据文件夹,没有配置镜像
declare -a DATA_DIRECTORY=(/data/gpdata/primary)
MASTER_DIRECTORY=/data/gpdata/gpmaster
ENCODING=UTF8

# 初始化
gpinitsystem -c gpinit_config -h hostfile_segments
~~~

### 查看集群状态

~~~shell
gpstate -d /data/gpdata/gpmaster/gpseg-1
~~~

## 登录

### 本地登录

~~~shell
#用pgadmin用户登录

psql -d postgres

\l

# 创建用户
CREATE USER test1 WITH PASSWORD 'test1' NOSUPERUSER;

~~~

### 远程登录

~~~shell
cd /data/gpdata/gpmaster/gpseg-1

vim pg_hba.config

# 添加远程访问允许
host    all     test1     all     md5

# 执行命令重新加载访问列表
pg_ctl reload -D /data/gpdata/gpmaster/gpseg-1

# 用DBeaver 或 Greenplum 客户端工具登录

#命令行远程登录
psql -U userName -h hostName dbName
~~~
