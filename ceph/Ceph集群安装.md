# Ceph集群安装

## 前置条件

1. 操作系统：Ubuntu-Server22.04, `Ceph Version 18.2.1 reef (stable)`

2. 更新国内源，提高软件包下载速度

```bash
# 备份官方源
sudo cp /etc/apt/sources.list /etc/apt/sources.list.bak
# 使用阿里云镜像
sudo bash -c "cat << EOF > /etc/apt/sources.list && apt update
deb http://mirrors.aliyun.com/ubuntu/ jammy main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ jammy main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ jammy-security main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ jammy-security main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ jammy-updates main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ jammy-updates main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ jammy-proposed main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ jammy-proposed main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ jammy-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ jammy-backports main restricted universe multiverse
EOF"
```

3. Ceph集群运行需要容器环境，安装Docker

```bash
sudo apt install apt-transport-https ca-certificates curl software-properties-common

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

# 根据服务器硬件选择 x86还是ARM64
# x86_64/amd64架构的系统
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# ARM64架构的系统
echo "deb [arch=arm64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update

sudo apt install docker-ce docker-ce-cli containerd.io

sudo systemctl enable docker

sudo systemctl start docker

sudo docker -v
```

### NTP时间同步服务器

> 集群各节点间需要保证时钟同步

1. 修改服务器时区

`sudo timedatectl set-timezone Asia/Shanghai`

2. 主节点下载ntp

`sudo apt install ntp`

3.  修改授时中心 

```bash
sudo vim /etc/ntp.conf
# 中国国家授时中心
server ntp.ntsc.ac.cn
```

## 初始化Ceph集群

1. `apt install -y cephadm`
2. `sudo apt install ceph-volume`
3. `sudo cephadm bootstrap --mon-ip <mon-ip>` 初始化集群
4. 开启CEPH CLI

```bash
cephadm add-repo --release reef
# 改用阿里云镜像
ceph_admin@ceph1:/etc/apt/sources.list.d$ vim ceph.list
#deb https://download.ceph.com/debian-reef/ jammy main
deb https://mirrors.aliyun.com/ceph/debian-reef/ jammy main
```

```Bash
sudo apt install ceph-common

cephadm shell
# Example
cephadm shell -- ceph -s

# 确认ceph命令可用
ceph -v

# 确认ceph命令能够连接到集群
ceph status

# 取消集群自动创建OSD
sudo ceph orch apply osd --all-available-devices --unmanaged=true
```

## 配置文件与常用命令

配置文件示例：

```ini
[global]
fsid = {cluster-id}
mon_initial_members = {hostname}[, {hostname}]
mon_host = {ip-address}[, {ip-address}]

#All clusters have a front-side public network.
#If you have two network interfaces, you can configure a private / cluster 
#network for RADOS object replication, heartbeats, backfill,
#recovery, etc.
public_network = {network}[, {network}]
#cluster_network = {network}[, {network}] 

#Clusters require authentication by default.
auth_cluster_required = cephx
auth_service_required = cephx
auth_client_required = cephx

#Choose reasonable number of replicas and placement groups.
osd_journal_size = {n}
osd_pool_default_size = {n}  # Write an object n times.
osd_pool_default_min_size = {n} # Allow writing n copies in a degraded state.
osd_pool_default_pg_autoscale_mode = {mode} # on, off, or warn
# Only used if autoscaling is off or warn:
osd_pool_default_pg_num = {n}

#Choose a reasonable crush leaf type.
#0 for a 1-node cluster.
#1 for a multi node cluster in a single rack
#2 for a multi node, multi chassis cluster with multiple hosts in a chassis
#3 for a multi node cluster with hosts across racks, etc.
osd_crush_chooseleaf_type = {n}
```

```bash
# 查看集群状态详细信息
sudo ceph health detail
```



## [添加主机](https://docs.ceph.com/en/reef/cephadm/host-management/#adding-hosts)

> [!NOTE]
>
> 添加主机前需要满足<a href="#前置条件">前置条件</a>

1. 在新主机中执行以下命令

```bash
apt install -y cephadm
sudo apt install ceph-common

#确认ceph命令可用
ceph -v
```

2. 添加Ceph集群SSH公钥到新主机上 

```bash
# 1.在这之前Ubuntu系统需要先设置root用户密码
ceph_admin@cephadmin:~$ sudo passwd root

# 2.修改/etc/ssh/sshd_config配置文件中 PermitRootLogin 字段为 yes
sudo systemctl daemon-reload
sudo systemctl restart ssh
# 3.添加Ceph集群SSH公钥到新主机上 
sudo ssh-copy-id -f -i /etc/ceph/ceph.pub root@<new-host>
```

3. 在集群主机上添加新主机

`ceph orch host add *<newhost>* [*<ip>*] [*<label1> ...*]`

```bash
# Example
sudo ceph orch host add host4 10.10.0.104 --labels _admin
```

4. 查看集群中的主机 `ceph orch host ls`
4. 设置时钟同步

```bash
# 修改服务器时区
sudo timedatectl set-timezone Asia/Shanghai
# 修改配置文件
sudo vim /etc/systemd/timesyncd.conf
# 修改ntp
NTP=ntp.ntsc.ac.cn
# 重启 systemd-timesyncd 服务
sudo service systemd-timesyncd restart

https://www.cnblogs.com/chyhonor/p/8573266.html
```

### 移除主机

`ceph orch host drain <host>`

**强制移除离线主机**

`ceph orch host rm <host> --offline --force`

## [隔离环境中部署](https://docs.ceph.com/en/reef/cephadm/install/#deployment-in-an-isolated-environment)

