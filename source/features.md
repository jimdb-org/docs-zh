# 特性

JIMDB是一个支持 key-value and SQL 两种协议，支持智能存储分层的分布式存储系统


## 高可用, 强一致, 可扩展, 高可靠

* raft复制协议

* 自动分裂, 自动扩容, 自动均衡

* 故障自动恢复


## 多协议支持

* KV: 支持Redis协议

* SQL: 支持MySQL协议


## 智能分层存储

* 内存 + 硬盘 双存储方式

* 智能调度: 依据访问频次以及QoS(Quality of Service), 智能调整数据属性为热/温/冷; 根据数据属性分别在RAM (masstree), SSD (rocksdb) and disks (rocksdb on CFS)中进行存储调度


## 分布式事务

read committed, currently implemented

txn record, intent, version, 2PC


## 云原生

* Kubernetes编排


## 简洁易用

* 自动捕获数据变更

* 在线DDL(Data Definition Language)

* 迁移工具, 管理系统, 报警系统, 丰富的监控报表支持

