---
title: "Etcd Backup" # Title of the blog post.
date: 2020-09-07T17:20:21+08:00 # Date of post creation.
description: "Article description." # Description used for search engine.
featured: true # Sets if post is a featured post, making appear on the home page side bar.
draft: false # Sets whether to render this page. Draft of true will not be rendered.
toc: false # Controls if a table of contents should be generated for first-level links automatically.
# menu: main
#featureImage: "/images/etcd.svg" # Sets featured image on blog post.
thumbnail: "/images/etcd.svg" # Sets thumbnail image appearing inside card on homepage.
shareImage: "/images/path/share.png" # Designate a separate image for social media sharing.
codeMaxLines: 10 # Override global value for how many lines within a code block before auto-collapsing.
codeLineNumbers: false # Override global value for showing of line numbers within code block.
figurePositionShow: true # Override global value for showing the figure label.
categories:
  - Technology
tags:
  - etcd
  - k8s
---

### install etcd-backup
etcd-backup 用于etcd 集群的补充备份，采用etcd snapshot ，进行全量备份
- 支持本地存储、分部署存储
- 支持配置化，备份周期、备份数量等可配
- 支持自动清除过期的备份数据
- 存储类型可扩展

#### create client secret
```bash
oc create secret generic etcd-backup-client-tls --from-file=etcd-client-ca.crt --from-file=etcd-client.crt --from-file=etcd-client.key --namespace=litsky
```

#### Set up RBAC

Set up basic [RBAC rules][rbac-rules] for etcd operator:

```bash
sh example/rbac/create_role.sh
```
```
oc adm policy add-scc-to-user privileged  system:serviceaccount:sky:etcd-operator
```
#### create etcd backup 
```bash
oc create -f ./example/etcd-backup-operator/statefulset/statefulset.yaml
```
添加nodeSelector，指定调度节点
```
spec:
  nodeSelector:
    kubernetes.io/hostname: infra01
    kubernetes.io/hostname: infra02
    kubernetes.io/hostname: infra03
```

#### create backup crd
默认备份数据存储在宿主机器上,支持分布式云存储（例如:storageType=S3）
```bash
oc create -f ./example/etcd-backup-operator/periodic_backup_cr.yaml
```
crd 配置相关项
```
spec:
  etcdEndpoints: ["192.168.55.124:2379"]   ##etcd 节点
  clientTLSSecret: "tumbler-etcd-client-tls"   ## etcd client secret name
  storageType: LOCAL   ## 存储类型，local、s3
  backupPolicy:
    # 0 > enable periodic backup
    backupIntervalInSecond: 125   # 备份周期，单位秒
    maxBackups: 4       # 保留最大备份数
    timeoutInSecond: 600  # 单次备份超时时间，单位秒
  local:
    path: /data/ #存储位置
```

### restore etcd data

#### 集群恢复
停掉所有节点，删除etcd_data下的数据，将备份数据拷贝到每个节点，执行数据恢复(以192.168.55.65:2379 为例)
```$xslt
rm -rf /$ETCD_HOME/etcd_data
etcdctl snapshot restore ./_v18026710_2019-04-01-07\:47\:35 --data-dir=/$ETCD_HOME/etcd_data --skip-hash-check=true --name=master1 --endpoints="https://192.168.55.65:2379" --initial-advertise-peer-urls="https://192.168.55.65:2380"  --initial-cluster="master1=https://192.168.55.65:2380,master2=https://192.168.55.124:2380,master3=https://192.168.55.55:2380"

#验证etcd数据恢复情况（如下查看k8s节点信息）
etcdctl get /kubernetes.io/minions --prefix --keys-only
```
执行完成后，分别在etcd所在的主机执行
```bash
systemctl start etcd
```
#### 单节点故障恢复(与集群扩容节点通用)
```
# 删除故障节点
[13:23:33 root@master1 opt]$ ETCDCTL_API=3 etcdctl --cert="/etc/origin/master/master.etcd-client.crt" --key="/etc/origin/master/master.etcd-client.key" --cacert="/etc/origin/master/master.etcd-ca.crt" --endpoints=192.168.55.124:2379,192.168.55.55:2379,192.168.55.65:2379 member list 
1063b12b4fd4e7b5, started, master1, https://192.168.55.65:2380, https://192.168.55.65:2379
43e7c14849bf13ab, started, master3, https://192.168.55.55:2380, https://192.168.55.55:2379
8eb116cd5f377a0e, started, master2, https://192.168.55.124:2380, https://192.168.55.124:2379
[13:24:24 root@master1 opt]$ ETCDCTL_API=3 etcdctl --cert="/etc/origin/master/master.etcd-client.crt" --key="/etc/origin/master/master.etcd-client.key" --cacert="/etc/origin/master/master.etcd-ca.crt" --endpoints=192.168.55.124:2379,192.168.55.55:2379,192.168.55.65:2379 member remove 1063b12b4fd4e7b5
Member 1063b12b4fd4e7b5 removed from cluster 5a26d8a168e75ef5
# 重新添加故障节点到集群中
[13:24:27 root@master1 opt]$ ETCDCTL_API=3 etcdctl --cert="/etc/origin/master/master.etcd-client.crt" --key="/etc/origin/master/master.etcd-client.key" --cacert="/etc/origin/master/master.etcd-ca.crt" --endpoints=192.168.55.124:2379,192.168.55.55:2379,192.168.55.65:2379 member add  master1 --peer-urls=https://192.168.55.65:2380
Member 60a74fd86702bb93 added to cluster 5a26d8a168e75ef5

ETCD_NAME="master1"
ETCD_INITIAL_CLUSTER="master3=https://192.168.55.55:2380,master1=https://192.168.55.65:2380,master2=https://192.168.55.124:2380"
ETCD_INITIAL_CLUSTER_STATE="existing"
# 删除故障节点数据
rm -rf $ETCD_DATA/*
```
  将故障节点配置修改为 ETCD_INITIAL_CLUSTER_STATE=existing
```bash
#启动etcd
systemctl start etcd
```
  验证
```cassandraql
[13:32:06 root@master1 opt]$ ETCDCTL_API=3 etcdctl --cert="/etc/origin/master/master.etcd-client.crt" --key="/etc/origin/master/master.etcd-client.key" --cacert="/etc/origin/master/master.etcd-ca.crt" --endpoints=192.168.55.124:2379,192.168.55.55:2379,192.168.55.65:2379 member list --write-out=table
+------------------+---------+---------+-----------------------------+-----------------------------+
|        ID        | STATUS  |  NAME   |         PEER ADDRS          |        CLIENT ADDRS         |
+------------------+---------+---------+-----------------------------+-----------------------------+
| 43e7c14849bf13ab | started | master3 |  https://192.168.55.55:2380 |  https://192.168.55.55:2379 |
| 60a74fd86702bb93 | started | master1 |  https://192.168.55.65:2380 |  https://192.168.55.65:2379 |
| 8eb116cd5f377a0e | started | master2 | https://192.168.55.124:2380 | https://192.168.55.124:2379 |
+------------------+---------+---------+-----------------------------+-----------------------------+

[13:32:36 root@master1 opt]$ ETCDCTL_API=3 etcdctl --cert="/etc/origin/master/master.etcd-client.crt" --key="/etc/origin/master/master.etcd-client.key" --cacert="/etc/origin/master/master.etcd-ca.crt" --endpoints=192.168.55.124:2379,192.168.55.55:2379,192.168.55.65:2379 endpoint status --write-out=table
+---------------------+------------------+---------+---------+-----------+-----------+------------+
|      ENDPOINT       |        ID        | VERSION | DB SIZE | IS LEADER | RAFT TERM | RAFT INDEX |
+---------------------+------------------+---------+---------+-----------+-----------+------------+
| 192.168.55.124:2379 | 8eb116cd5f377a0e |  3.2.22 |  334 MB |      true |    121572 |   82894906 |
|  192.168.55.55:2379 | 43e7c14849bf13ab |  3.2.22 |  334 MB |     false |    121572 |   82894906 |
|  192.168.55.65:2379 | 60a74fd86702bb93 |  3.2.22 |  334 MB |     false |    121572 |   82894906 |
+---------------------+------------------+---------+---------+-----------+-----------+------------+

```

### Cleanup etcd-backup
```bash
oc delete -f ./example/etcd-backup-operator/periodic_backup_cr.yaml
oc delete -f ./example/etcd-backup-operator/statefulset/statefulset.yaml
oc delete -f ./example/etcd-backup-operator/statefulset/secret.yaml
oc delete crd etcdbackups.etcd.database.coreos.com
```
