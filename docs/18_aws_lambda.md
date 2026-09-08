# LEC 18 - Serverless 与按需容器加载 (AWS Lambda)

> **经典论文**：*On-demand Container Loading in AWS Lambda* (Marc Brooker, Mike Danilov et al., Amazon Web Services, USENIX ATC 2023)  
> **核心主题**：无服务器计算 (Serverless)、极低延迟冷启动 (Cold-start)、分块内容寻址存储与树形按需加载 (Lazy Loading)

---

## 1. 无服务器架构下的冷启动物理挑战

AWS Lambda 每天需要处理数十亿次函数调用。当突发流量到达未预热实例的函数时，必须在几十毫秒内下载函数容器镜像（通常在数百 MB 至数 GB 之间）并完成启动。
- **痛点**：下载整个镜像消耗海量数据中心内部带宽，且冷启动耗时可达数秒。
- **核心洞察**：绝大多数容器在启动与处理前几次请求期间，**仅访问整个镜像中不到 6% 的数据块**！

---

## 2. 破局方案：分块内容寻址与分布式分发树

AWS Lambda 团队重构了镜像加载流水线：
1. **分块内容寻址 (Chunk-based Content Addressable Storage)**：镜像被拆分为固定大小的加密分块（由 SHA-256 唯一寻址）。
2. **全局去重**：不同用户、不同镜像中的相同操作系统基础层和公共依赖库被全局去重，极大压缩存储占用。
3. **分层分发树 (Tree-based Tiered Caching)**：计算节点通过本地快表和多级缓存树，仅在文件系统发生真实的按需 Page Fault 时以微秒级延迟拉取对应的数据块，将冷启动延迟从数秒暴降至几十毫秒。
