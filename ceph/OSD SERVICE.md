# [OSD SERVICE](https://docs.ceph.com/en/latest/cephadm/services/osd/)

## 前置条件

**满足以下条件的存储设备才会被视为可用状态**

- 设备必须没有分区（使用`gdisk` 工具删除分区解决）

- 设备必须没有 LVM 状态（使用`mkfs -t xfs -f /dev/<dev-name>` 命令移除LVM解决）
- 设备必须没有被挂载
- 设备必须没有文件系统 
  -  擦除文件系统 `sudo wipefs -af /dev/<dev-name>` 
  - `ceph-volume lvm zap /dev/<dev-name>`

- 设备必须不包含一个Ceph BlueStore OSD（使用`gdisk` 工具删除解决）
- 设备必须大于5GB

---

查看硬盘信息用到的命令：

`lsblk -f` 查看硬盘的详细信息 

`fdisk -l `查看硬盘及分区信息

### :bug:问题清单

#### 保留现场

```bash
ceph_admin@ceph1:~$ sudo mkfs -t xfs -f /dev/sdb
mkfs.xfs: cannot open /dev/sdb: Device or resource busy
```

#### 问题原因

块设备已经存在逻辑卷映射配置，移除即可

#### 解决方法

```bash
# 1
ceph_admin@ceph1:~$ sudo dmsetup ls
ceph--1714d176--2f60--49d4--bbdc--53b0728b5049-osd--block--f5209bf4--1842--484e--8e01--f720edb64756     (253:8)
ceph--2da8a2e4--6406--4973--b672--e44cae807ef9-osd--block--188e4253--0463--4d44--94c1--8141bd4dc164     (253:9)
ceph--42d15efb--8393--4bef--8aeb--6e889e510480-osd--block--d6af00af--b5dd--45f8--b097--08c7ff44c77b     (253:4)
ceph--54690c85--11e5--41b3--8a55--77d98a69d52f-osd--block--8ed72846--45ae--4c29--be87--b4e1a85fc478     (253:5)
ceph--6f6a0749--92ec--4772--8e9f--2f0f86c669b7-osd--block--97158338--6bd5--480b--8baf--7c2553d680d1     (253:1)
ceph--7ee59399--c6dc--417b--b64e--b5793e7bfbc9-osd--block--f711511c--67f8--49bb--914a--dd091537c244     (253:2)
ceph--9735a0f7--c799--424b--9f5a--8f19256f0ada-osd--block--8f77cd52--97cb--4256--b04e--0c26d0d12b88     (253:6)
ceph--aafdf7c3--9699--4af2--a514--4df1ff2d181a-osd--block--cb099963--4592--4e7c--9345--8c62afdef49e     (253:0)
ceph--cd8ea852--bb21--43c5--a150--14b1b6d6ae1d-osd--block--f1da1605--4b5d--4aa6--baf6--c834da210bd5     (253:7)
ceph--ff87e5b8--122a--4f26--8562--7d864b516b72-osd--block--dd684b75--724b--4cb4--a0ca--53f3316aea52     (253:3)

# 2
sudo dmsetup remove ceph--aafdf7c3--9699--4af2--a514--4df1ff2d181a-osd--block--cb099963--4592--4e7c--9345--8c62afdef49e

# 3 重新执行
ceph_admin@ceph1:~$ sudo mkfs -t xfs -f /dev/sdb
meta-data=/dev/sdb               isize=512    agcount=16, agsize=7317504 blks
         =                       sectsz=4096  attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=1, rmapbt=0
         =                       reflink=1    bigtime=0 inobtcount=0
data     =                       bsize=4096   blocks=117080064, imaxpct=25
         =                       sunit=64     swidth=64 blks
naming   =version 2              bsize=4096   ascii-ci=0, ftype=1
log      =internal log           bsize=4096   blocks=57168, version=2
         =                       sectsz=4096  sunit=1 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0

```

## OSD 配置

打印出一份由Cephadm发现的设备清单

`ceph orch device ls [--hostname=...] [--wide] [--refresh]`

**OSD 配置文件**

`sudo ceph orch apply -i osd.yaml` 应用配置到集群中

```yaml
service_type: osd
service_id: osd_using_path
placement:
  hosts:
    - ceph1
spec:
  data_devices:
    paths:
    - /dev/sdd
    - /dev/sde
    - /dev/sdf
    - /dev/sdg
    - /dev/sdh
    - /dev/sdi
    - /dev/sdj
    - /dev/sdk
  db_devices:
    paths:
    - /dev/sdb
    - /dev/sdc
---
service_type: osd
service_id: ceph2_osd
placement:
  hosts:
    - ceph2
spec:
  data_devices:
    all: true
```

## 操作命令

1. 向集群中添加OSD
   1.  `sudo ceph orch daemon add osd hostname:/path/dev`
   2.  :star:**推荐**：​[配置文件形式](https://docs.ceph.com/en/latest/cephadm/services/osd/#advanced-osd-service-specifications) 


2. 检查现有 OSD

`sudo ceph osd tree`

---

```bash
# 查看osd 操作状态
ceph orch osd rm status
# Example:
ceph_admin@cephadmin:~$ sudo ceph orch osd rm status
No OSD remove/replace operations reported
```

```bash
# 停止OSD移除
ceph orch osd rm stop <osd_id(s)>
# Example:
ceph_admin@cephadmin:~$ ceph orch osd rm stop 4
Stopped OSD(s) removal
```

```bash
# 查看osd状态
sudo ceph osd status

# Example
ceph_admin@ceph1:/etc/ceph$ sudo ceph osd status
ID  HOST    USED  AVAIL  WR OPS  WR DATA  RD OPS  RD DATA  STATE
 0  ceph1   111G  7451G      0        0       0        0   exists,up
 1  ceph1   111G  7451G      0        0       0        0   exists,up
 2  ceph1   111G  3725G      0        0       0        0   exists,up
 3  ceph1   111G  3725G      0        0       0        0   exists,up
 4  ceph1   111G  3725G      0        0       0        0   exists,up
 5  ceph1   111G  3725G      0        0       0        0   exists,up
 6  ceph1   111G  3725G      0        0       0        0   exists,up
 7  ceph1   111G  3725G      0        0       0        0   exists,up
 8  ceph3   111G  7451G      0        0       0        0   exists,up
 9  ceph3   111G  7451G      0        0       0        0   exists,up
10  ceph3   111G  3725G      0        0       0        0   exists,up
11  ceph3   111G  3725G      0        0       0        0   exists,up
12  ceph3   111G  3725G      0        0       0        0   exists,up
13  ceph3   111G  3725G      0        0       0        0   exists,up
14  ceph3   111G  3725G      0        0       0        0   exists,up
15  ceph3   111G  3725G      0        0       0        0   exists,up
16            0      0       0        0       0        0   exists,new
17            0      0       0        0       0        0   exists,new
18            0      0       0        0       0        0   exists,new
19            0      0       0        0       0        0   exists,new
20            0      0       0        0       0        0   exists,new
21            0      0       0        0       0        0   exists,new
22            0      0       0        0       0        0   exists,new
```



### 删除现有 OSD

1. 将OSD标记为"out"，这将使Ceph开始将数据从该OSD迁移到其他OSD

   `ceph osd out osd.<osd-id>`

2. 停止该OSD的daemon

   `systemctl stop ceph-osd@<osd-id>`

3. 使用ceph-volume命令来销毁OSD的数据和日志

   `ceph-volume lvm zap /dev/<vg-name>/<lv-name> --destroy`

4. 从Ceph集群中删除该OSD

   `ceph osd purge osd.<osd-id> --yes-i-really-mean-it`

