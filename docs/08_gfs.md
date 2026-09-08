# LEC 08 - Google File System (GFS) 架构深析

> **经典论文**：*The Google File System* (Sanjay Ghemawat, Howard Gobioff, and Shun-Tak Leung; Google Inc., SOSP 2003)  
> **核心主题**：单 Master 架构、Chunk 切分、租约机制 (Leases)、弱一致性模型与原子记录追加 (Atomic Record Append)

---

## 1. 业务驱动的设计假设 (Design Assumptions)

GFS 是 Google 大数据基石（GFS、MapReduce、Bigtable）的核心第一环。与传统分布式 POSIX 文件系统（如 NFS、AFS）不同，GFS 为 Google 独特的批处理负载量身定制：
1. **组件失效是常态**：集群由数万台廉价商用服务器构成，硬盘损坏、节点掉电每时每刻都在发生。
2. **大文件为主**：文件通常以数 GB 乃至数 TB 计量。
3. **写入以海量并发“追加 (Append)”为主**：极少对文件随机覆写 (Random Overwrite)，通常是一写多读、流水线追加。

---

## 2. 核心架构与数据控制解耦

```mermaid
flowchart TB
    Client[客户端 GFS Client]
    Master[GFS Master 节点<br/>- 内存管理文件目录树<br/>- 维护 Chunk 映射与 60s 租约]

    subgraph Chunkservers["分布式 Chunkserver 集群"]
        CS_Primary["Primary Chunkserver<br/>(持有写租约 Lease)"]
        CS_Secondary1["Secondary Chunkserver 1"]
        CS_Secondary2["Secondary Chunkserver 2"]
    end

    Client -->|"① 请求 Chunk 句柄与副本位置"| Master
    Master -->|"② 返回 Primary 与 Secondary 列表"| Client

    Client == "③ 流水线传输数据包 (Data Pipelining)" ==> CS_Secondary1
    CS_Secondary1 == "数据直传" ==> CS_Primary
    CS_Primary == "数据直传" ==> CS_Secondary2

    Client -->|"④ 发送写入指令 (Write Command)"| CS_Primary
    CS_Primary -->|"⑤ 确定写入偏移量，分发写指令"| CS_Secondary1
    CS_Primary -->|"⑤ 分发写指令"| CS_Secondary2

    CS_Secondary1 -->>|"⑥ 确认写入"| CS_Primary
    CS_Secondary2 -->>|"⑥ 确认写入"| CS_Primary

    CS_Primary -->>|"⑦ 响应写入成功"| Client
```

### 单 Master 为什么不成为性能瓶颈？
GFS 严格执行**控制流与数据流彻底分离**：
- Master 仅参与元数据查询（返回 Chunk 所在的 Chunkserver 列表），绝不中转任何实际文件字节数据。
- 客户端一次查询可以缓存数个 64MB Chunk 的位置信息，后续数百兆的数据传输完全在 Client 与 Chunkserver 之间点对点直接进行。
