# [POOLS](https://docs.ceph.com/en/reef/rados/operations/pools/)

## 删除池

```bash
ceph osd pool delete {pool-name} [{pool-name} --yes-i-really-really-mean-it]

# Example
ceph_admin@ceph1:/etc/ceph$ sudo ceph osd pool delete ecpool ecpool --yes-i-really-really-mean-it
Error EPERM: pool deletion is disabled; you must first set the mon_allow_pool_delete config option to true before you can destroy a pool
# 解决：
ceph_admin@ceph1:/etc/ceph$ sudo ceph tell mon.* injectargs '--mon-allow-pool-delete=true'
mon.ceph1: {}
mon.ceph1: mon_allow_pool_delete = 'true'
mon.ceph4: {}
mon.ceph4: mon_allow_pool_delete = 'true'
mon.ceph3: {}
mon.ceph3: mon_allow_pool_delete = 'true'
# 重试
ceph_admin@ceph1:/etc/ceph$ sudo ceph osd pool delete ecpool  ecpool  --yes-i-really-really-mean-it
pool 'ecpool' removed
```

## 常用命令

```bash
# 显示池的利用率统计信息
sudo rados df
# 获取特定池或所有池的I/O信息
sudo ceph osd pool stats [{pool-name}]
# 查看集群中的pool
sudo ceph osd lspools
# 查看 pool 的空间使用情况
sudo ceph df
ceph_admin@ceph1:~$ sudo ceph df
--- RAW STORAGE ---
CLASS     SIZE   AVAIL    USED  RAW USED  %RAW USED
hdd    112 TiB  93 TiB  19 TiB    19 TiB      16.87
TOTAL  112 TiB  93 TiB  19 TiB    19 TiB      16.87

--- POOLS ---
POOL                ID   PGS   STORED  OBJECTS     USED  %USED  MAX AVAIL
.mgr                 1     1   18 MiB        6   55 MiB      0     27 TiB
cephfs.cephfs.meta   2    16  261 MiB       96  782 MiB      0     27 TiB
cephfs.cephfs.data   3  1024  1.2 TiB  315.26k  3.6 TiB   4.26     27 TiB
erasurepool          9    32      0 B        0      0 B      0     53 TiB
erasurepool2        10    32  9.1 TiB    4.93M   14 TiB  14.50     53 TiB
metadata            11    16  1.9 GiB    5.15k  5.8 GiB      0     27 TiB

```

## [ERASURE CODE](https://docs.ceph.com/en/reef/rados/operations/erasure-code/)

[ERASURE CODING WITH OVERWRITES](https://docs.ceph.com/en/reef/rados/operations/erasure-code/#erasure-coding-with-overwrites)

```bash
#Since Luminous, partial writes for an erasure-coded pool may be enabled with a per-pool setting. This lets #RBD and CephFS store their data in an erasure-coded pool:

ceph osd pool set ec_pool allow_ec_overwrites true
```

[CRUSH CONSTRAINTS CANNOT BE SATISFIED](https://docs.ceph.com/en/reef/rados/troubleshooting/troubleshooting-pg/#crush-constraints-cannot-be-satisfied)

```yaml
# 创建配置文件
sudo ceph osd erasure-code-profile set myprofile-02    k=2   m=1  crush-failure-domain=osd
# 创建纠删码池
sudo ceph osd pool create ecpool3 erasure myprofile-02
# 关联应用
ceph osd pool application enable {pool-name} {application-name}

# Example
ceph_admin@ceph1:~$ sudo ceph osd pool application enable cephfs.cephfs.ecpool3 cephfs
enabled application 'cephfs' on pool 'cephfs.cephfs.ecpool3'
```

## [ERASURE CODE PROFILES](https://docs.ceph.com/en/reef/rados/operations/erasure-code-profile/)







 



