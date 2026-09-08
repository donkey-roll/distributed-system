# LEC 16 - 大规模缓存一致性 (Facebook Memcached)

> **经典论文**：*Scaling Memcache at Facebook* (Rajesh Nishtala et al., Facebook Inc., NSDI 2013)  
> **核心主题**：十亿级用户架构、Cache-Aside 模式、Thundering Herd (惊群效应)、Lease 租约防击穿与跨 Region 复制一致性

---

## 1. 规模驱动的极端架构演进

Facebook 的工作负载以极端的**读多写少 (Read-heavy)** 为主（例如读取个人主页内容、好友关系链与消息通知）。
单体数据库在数千万 QPS 面前将瞬间崩溃，因此系统全面采用 **Memcached 分布式内存缓存层** 阻挡 99% 以上的数据库读流量。

---

## 2. 核心挑战与精妙解法

```mermaid
flowchart TD
    subgraph Challenge1["挑战 1: 缓存击穿与雪崩 (Thundering Herd)"]
        K1[热点 Key 失效] --> C1[成千上万并发请求同时未命中]
        C1 --> DB1[瞬时击垮底层 MySQL]
        DB1 --> Solution1["【解法】Memcache Leases 租约机制<br/>只向第一个未命中请求发放写入租约 Token<br/>其他请求等待或返回稍旧快照数据"]
    end

    subgraph Challenge2["挑战 2: 故障隔离与连环瘫痪"]
        M1[单台 Memcached 宕机] --> C2[请求大量穿透至数据库引发雪崩]
        C2 --> Solution2["【解法】Gutter Pool 应急备用池<br/>专设 1% 容量的轻量备用集群<br/>临时承接故障节点的失效写入与短 TTL 缓存"]
    end
```

### 3. 跨 Region 一致性保障
跨大洋不同机房之间存在数十毫秒的光纤延迟。写操作全部路由到 Master Region 并在提交后，通过 MySQL 复制流 (McSqueal) 异步发送无效化指令 (Invalidate) 清除各 Slave Region 的本地缓存，避免脏读扩散。
