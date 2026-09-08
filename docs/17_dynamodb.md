# LEC 17 - Amazon DynamoDB 架构与事务实现

> **经典论文**：*Amazon DynamoDB: A Scalable, Predictably Performant, and Fully Managed NoSQL Database Service* (Mostafa Elhemali et al., Amazon Web Services, USENIX ATC 2022 / 2023)  
> **核心主题**：计算存储分离、Paxos 多副本复制组、自适应容量分区 (Adaptive Partitioning) 与分布式 ACID 事务实现

---

## 1. 从原始 Dynamo 到现代化 DynamoDB

2007 年经典的 Dynamo 论文强调弱一致性、去中心化环形哈希 (Consistent Hashing) 与向量时钟客户端冲突合并（Last-Write-Wins 易丢数据）。
而现代托管云服务 **DynamoDB** 全面转向了更加严密可靠的工业级架构：
1. **强一致性支持**：支持按需强一致性读 (Strongly Consistent Read) 与最终一致性读。
2. **多副本 Paxos 组**：以 Partition 为单位构建 Multi-Paxos 复制组。
3. **计算请求路由器与存储存储节点彻底解耦**。

---

## 2. 跨分区分布式事务架构

DynamoDB 支持原子跨表、跨行事务 (`TransactWriteItems`)。
- **协议核心**：基于经过高可用 Paxos 强化的两阶段提交 (2PC)。
- **协调者高可用**：事务协调逻辑由无状态的 Transaction Coordinator 节点执行，并将事务状态通过 Paxos 日志持久化，避免单点阻塞。
