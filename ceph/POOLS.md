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
ceph_admin@ceph1:/etc/ceph$ sudo ceph osd pool delete ecpool  ecpool  --yes-i-really-really-mean-it
pool 'ecpool' removed
```

## [ERASURE CODE](https://docs.ceph.com/en/reef/rados/operations/erasure-code/)

[ERASURE CODING WITH OVERWRITES](https://docs.ceph.com/en/reef/rados/operations/erasure-code/#erasure-coding-with-overwrites)

```bash
#Since Luminous, partial writes for an erasure-coded pool may be enabled with a per-pool setting. This lets #RBD and CephFS store their data in an erasure-coded pool:

ceph osd pool set ec_pool allow_ec_overwrites true
```

```yaml
# 创建配置文件
sudo ceph osd erasure-code-profile set myprofile-02     k=4     m=2     crush-failure-domain=rack
# 创建纠删码池
sudo ceph osd pool create ecpool2 erasure myprofile-02
```

## [ERASURE CODE PROFILES](https://docs.ceph.com/en/reef/rados/operations/erasure-code-profile/)