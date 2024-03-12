# [MGR Service](https://docs.ceph.com/en/latest/cephadm/services/mgr/)

## 常用命令

```
# 查看mgr各个模块信息
sudo ceph mgr module ls
# Example
ceph_admin@cephadmin:~$ sudo ceph mgr module ls
MODULE
balancer              on (always on)
crash                 on (always on)
devicehealth          on (always on)
orchestrator          on (always on)
pg_autoscaler         on (always on)
progress              on (always on)
rbd_support           on (always on)
status                on (always on)
telemetry             on (always on)
volumes               on (always on)
cephadm               on
dashboard             on
iostat                on
nfs                   on
restful               on
alerts                -
diskprediction_local  -
influx                -
insights              -
k8sevents             -
localpool             -
mds_autoscaler        -
mirroring             -
osd_perf_query        -
osd_support           -
prometheus            -
rgw                   -
rook                  -
selftest              -
snap_schedule         -
stats                 -
telegraf              -
test_orchestrator     -
zabbix                -
```



## [Dashboard 控制台](https://docs.ceph.com/en/latest/mgr/dashboard/)

1. 开启控制台

`sudo ceph mgr module enable dashboard`

2. 设置使用http协议访问控制台

` sudo ceph config set mgr mgr/dashboard/ssl false`

3. 设置用户名及密码

`sudo ceph dashboard ac-user-create <username> -i <file-containing-password> administrator`

4. 重启dashboard

```bash
sudo ceph mgr module disable dashboard
sudo ceph mgr module enable dashboard
```