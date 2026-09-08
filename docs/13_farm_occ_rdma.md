# LEC 13 - 乐观并发控制与 RDMA 极速存储 (FaRM)

> **经典论文**：*No compromises: distributed transactions with RPC over RDMA* (Aleksandar Dragojević et al., Microsoft Research, SOSP 2015)  
> **核心主题**：单边 RDMA (One-sided RDMA)、非易失内存 (NVRAM)、乐观并发控制 (OCC) 与超低延迟分布式事务

---

## 1. 现代硬件革新对分布式存储架构的颠覆

传统分布式系统的核心性能瓶颈是：网络协议栈内核态开销 (TCP/IP stack) 和磁盘机械/SSD 写入延迟。
FaRM (Fast Remote Memory) 基于两大颠覆性硬件：
1. **RDMA (Remote Direct Memory Access)**：网卡直接旁路 OS 内核与 CPU，通过单边 RDMA 读写远程机器的物理内存，微秒级极低延迟。
2. **非易失内存 (NVRAM / Battery-backed RAM)**：掉电不丢数据，内存写即持久化。

---

## 2. 乐观并发控制 (Optimistic Concurrency Control - OCC)

在读多写少的高并发负载下，悲观加锁（2PL）会导致严重的锁排队争用。FaRM 采用 OCC 范式：
- **执行阶段 (Execute)**：客户端使用单边 RDMA 读直接读取远程节点内存，完全不经过对端 CPU，记录对象版本号。
- **锁定阶段 (Lock)**：在主节点对写入集对象加锁并更新版本号。
- **校验阶段 (Validate)**：客户端校验读取集的版本号是否发生变化。若未变，则证明执行期间无并发冲突；若发生修改，则立即回滚重试。
- **提交与落盘 (Commit & Apply)**：利用 NVRAM 并发追加事务日志，并行向备份节点广播。
