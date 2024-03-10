# [CEPH-VOLUME](https://docs.ceph.com/en/latest/ceph-volume/#ceph-volume)

### ceph-volume

```bash
# 检索硬盘信息
ceph-volume inventory /dev/sdj
#Example
ceph_admin@cephadmin:~$ sudo ceph-volume inventory /dev/sdj

====== Device report /dev/sdj ======

     path                      /dev/sdj
     ceph device               False
     lsm data                  {}
     available                 False
     rejected reasons          Insufficient space (<5GB)
     device id                 ATA_ST4000NM002A-2HZ_WS24ZTHD
```

`ceph-volume lvm` 命令用于管理 Ceph OSD 使用的 LVM 卷。它支持以下操作：

- **准备**：将 LVM 卷格式化为 Ceph OSD 使用。
- **激活**：将 LVM 卷与 Ceph OSD 关联并启动 OSD。
- **列出**：列出所有与 Ceph 关联的 LVM 卷。
- **批量**：自动调整多 OSD 配置中设备的大小。
- **取消激活**：停止 OSD 并从 Ceph 中删除 LVM 卷。
- **创建**：从 LVM 卷创建新的 OSD。
- **触发**：用于激活 OSD 的 systemd 帮助程序。
- **清除**：从逻辑卷或分区中删除所有数据和文件系统。
- **迁移**：将 BlueFS 数据从另一个 LVM 卷迁移。
- **新 WAL**：在指定的逻辑卷上为 OSD 分配新的 WAL 卷。
- **新数据库**：在指定的逻辑卷上为 OSD 分配新的数据库卷。

**注意**：使用 `create` 子命令可以将 `prepare` 和 `activate` 子命令组合成一个子命令。

Ceph 文档：https://docs.ceph.com/en/latest/man/8/ceph-volume/。

以下是一些使用 ceph-volume lvm 命令的示例：

- **准备 LVM 卷**

```
ceph-volume lvm prepare /dev/sdc
```

- **激活 LVM 卷**

```
ceph-volume lvm activate /dev/sdc
```

- **列出所有与 Ceph 关联的 LVM 卷**

```
ceph-volume lvm list
```

- **自动调整多 OSD 配置中设备的大小**

```
ceph-volume lvm batch /dev/sdc /dev/sdd /dev/sde
```

- **停止 OSD 并从 Ceph 中删除 LVM 卷**

```
ceph-volume lvm deactivate /dev/sdc
```

- **从 LVM 卷创建新的 OSD**

```
ceph-volume lvm create /dev/sdc
```

- **用于激活 OSD 的 systemd 帮助程序**

```
ceph-volume lvm trigger /dev/sdc
```

- **从逻辑卷或分区中删除所有数据和文件系统**

```
ceph-volume lvm zap /dev/sdc
```

- **将 BlueFS 数据从另一个 LVM 卷迁移**

```
ceph-volume lvm migrate /dev/sdc /dev/sdd
```

- **在指定的逻辑卷上为 OSD 分配新的 WAL 卷**

```
ceph-volume lvm new-wal /dev/sdc
```

- **在指定的逻辑卷上为 OSD 分配新的数据库卷**

```
ceph-volume lvm new-db /dev/sdc
```
