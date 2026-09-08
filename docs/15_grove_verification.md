# LEC 15 - 分布式系统形式化验证 (Grove)

> **经典论文**：*Verified distributed systems made practical* (Upamanyu Sharma et al., MIT CSAIL, SOSP 2023)  
> **核心主题**：形式化证明、Iris 分离逻辑 (Separation Logic)、并发无 Bug 证明与真实 Go 分布式系统自动验证

---

## 1. 为什么测试无法保证分布式系统的绝对正确？

Edsger Dijkstra 曾有名言：“测试只能证明程序存在 Bug，而不能证明程序没有 Bug”。
在分布式系统中，并发网络时序、偶发丢包和机器宕机重叠产生的天文数字级状态空间，使得随机压测和混沌工程（Chaos Engineering）永远无法覆盖所有边缘罕见 Case。

---

## 2. Grove 的突破：直接形式化验证生产级 Go 代码

历史上形式化验证系统（如 IronFleet、CertiKOS）通常要求使用专用数学证明语言（如 Coq、Dafny），执行性能极差且与现代云原生工程完全脱节。

**Grove 的核心创新**：
1. **面向真实 Go 代码**：直接翻译和验证标准的 Go 并发程序。
2. **基于 Iris 分离逻辑**：将跨节点的崩溃一致性、并发锁协议以及网络包不变量统一纳入数学模型。
3. **可证明的高性能**：验证构建的 KV 存储与两阶段提交系统，在真实硬件上具备与非验证系统相当的吞吐量与极低延迟。
