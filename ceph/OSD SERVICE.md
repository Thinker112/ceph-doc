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

## 操作命令

1. 向集群中添加OSD
   1.  `sudo ceph orch daemon add osd hostname:/path/dev`
   2.  [配置文件形式](https://docs.ceph.com/en/latest/cephadm/services/osd/#advanced-osd-service-specifications)


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



### 删除现有 OSD

1. 将OSD标记为"out"，这将使Ceph开始将数据从该OSD迁移到其他OSD

   `ceph osd out osd.<osd-id>`

2. 停止该OSD的daemon

   `systemctl stop ceph-osd@<osd-id>`

3. 使用ceph-volume命令来销毁OSD的数据和日志

   `ceph-volume lvm zap /dev/<vg-name>/<lv-name> --destroy`

4. 从Ceph集群中删除该OSD

   `ceph osd purge osd.<osd-id> --yes-i-really-mean-it`





