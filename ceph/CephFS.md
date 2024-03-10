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

