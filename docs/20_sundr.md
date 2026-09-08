# LEC 20 - 分叉一致性与非可信存储 (SUNDR)

> **经典论文**：*Secure Untrusted Data Repository (SUNDR)* (Jinyuan Li, Maxwell Krohn, David Mazières, and Dennis Shasha; NYU, OSDI 2004)  
> **核心主题**：零信任存储模型 (Zero-Trust Storage)、分叉一致性 (Fork Consistency)、密码学签名与恶意服务器篡改检测

---

## 1. 为什么传统安全防护无法抵御恶意存储服务器？

在云存储模型中，如果存储服务器已被黑客或内鬼完全攻陷：
- 加密（Encryption）能防止窃听泄密。
- 数字签名（Digital Signatures）能防止单条记录被篡改伪造。
- **但无法防御“重放旧版本”或“选择性隐瞒更新”！** 恶意服务器可以向客户端 A 展示最新版本，同时向客户端 B 展示三个月前的旧版本，造成客户端之间的认知分裂。

---

## 2. 分叉一致性 (Fork Consistency)

SUNDR 提出了在服务器完全不可信的前提下，系统所能达到的最强一致性边界：**分叉一致性 (Fork-Linearizability)**。

> [!IMPORTANT]
> **分叉一致性定理**：
> 如果不可信服务器试图对某一个客户端隐瞒操作或伪造时间线，系统将不可逆地“分叉 (Fork)”为两条独立的历史轨迹。
> **一旦发生分叉，被欺骗的客户端之间将再也无法进行任何后续协作**，因为任何跨客户端的读写操作都会校验对方的密码学签名版本链（Version Vectors），从而以 100% 的确定性当场捕获服务器的恶意欺诈！
