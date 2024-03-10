# Ceph集群安装

## 前置条件

1. 操作系统：Ubuntu-Server22.04, `Ceph Version 18.2.1 reef (stable)`

2. 更新国内源，提高软件包下载速度

```bash
sudo bash -c "cat << EOF > /etc/apt/sources.list && apt update
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-updates main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-updates main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-backports main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-backports main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-security main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-security main restricted universe multiverse
EOF"
```

3. Ceph集群运行需要容器环境，安装Docker

```bash
sudo apt install apt-transport-https ca-certificates curl software-properties-common

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

#x86_64/amd64架构的系统
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

#ARM64架构的系统
echo "deb [arch=arm64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update

sudo apt install docker-ce docker-ce-cli containerd.io

sudo systemctl enable docker

sudo systemctl start docker

sudo docker -v
```

### NTP时间同步服务器 (集群各节点间需要保证时钟同步)

1. 修改服务器时区

`sudo timedatectl set-timezone Asia/Shanghai`

2. 下载ntp

`sudo apt install nt`



## 初始化Ceph集群

1. `apt install -y cephadm`
2. `sudo apt install ceph-volume`
3. `sudo cephadm bootstrap --mon-ip <mon-ip>` 初始化集群
4. 开启Ceph CLI

```Bash
cephadm add-repo --release reef

cephadm install ceph-common

#确认ceph命令可用
ceph -v

#确认ceph命令能够连接到集群
ceph status
```

## 添加主机

> [!NOTE]
>
> 添加主机前需要满足<a href="#前置条件">前置条件</a>

1. 在新主机中执行以下命令

```bash
apt install -y cephadm 

cephadm add-repo --release reef

cephadm install ceph-common

#确认ceph命令可用
ceph -v 
```

2. 添加Ceph集群SSH公钥到新主机上 

```bash
# 在这之前Ubuntu系统需要先设置root用户密码
# 修改/etc/ssh/sshd_config配置文件中 PermitRootLogin 字段为 yes

sudo ssh-copy-id -f -i /etc/ceph/ceph.pub root@<new-host>
```

3. 在集群主机上添加新主机

`ceph orch host add *<newhost>* [<ip>] [<label1> ...]`

4. 查看集群中的主机 `ceph orch host ls`

### 移除主机

`ceph orch host drain <host>`
