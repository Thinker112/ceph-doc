# [CephFS](https://docs.ceph.com/en/latest/cephfs/#getting-started-with-cephfs)

## 常用命令

```bash
# 查看FS状态
sudo ceph fs status

ceph_admin@cephadmin:~$ sudo ceph fs status
cephfs - 0 clients
======
RANK   STATE              MDS            ACTIVITY   DNS    INOS   DIRS   CAPS
 0    creating  cephfs.cephadmin.qtkfal              10     13     12      0
       POOL           TYPE     USED  AVAIL
cephfs.cephfs.meta  metadata     0      0
cephfs.cephfs.data    data       0      0
      STANDBY MDS
cephfs.cephadmin.gmewbj
MDS version: ceph version 17.2.7 (b12291d110049b2f35e32e0de30d70e9a4c060d2) quincy (stable)
```



## 创建CephFS

 ```bash
 # Create a CephFS volume named (for example) "cephfs":
 ceph fs volume create cephfs
 ```

## 客户端挂载

1. 为客户端主机生成一个最小的conf文件并将其放置在标准位置

```bash
# on client host
mkdir -p -m 755 /etc/ceph
ssh {user}@{mon-host} "sudo ceph config generate-minimal-conf" | sudo tee /etc/ceph/ceph.conf
```

2. 确保ceph.conf具有适当的权限

```bash
chmod 644 /etc/ceph/ceph.conf
```

3. 创建一个CephX用户并获取其密钥,   ==注意修改`cephfs` 文件系统名称==

```bash
# foo是ceph 管理员用户名
ssh {user}@{mon-host} "sudo ceph fs authorize cephfs client.foo / rw" | sudo tee /etc/ceph/ceph.client.foo.keyring
```

4. 确保keyring具有适当的权限

```bash
chmod 600 /etc/ceph/ceph.client.foo.keyring
```

5. 使用`mount`命令使用内核驱动程序挂载CephFS

[官方文档](https://docs.ceph.com/en/reef/cephfs/mount-using-kernel-driver/#mounting-cephfs)

```bash
mkdir /mnt/mycephfs
mount -t ceph <name>@<fsid>.<fs_name> =/ /mnt/mycephfs
```

