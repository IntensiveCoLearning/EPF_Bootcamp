---
timezone: UTC+8
---

# klizz

**GitHub ID:** klizz111

**Telegram:** 

## Self-introduction

EPF 实习计划

## Notes

<!-- Content_START -->
# 2026-04-15
<!-- DAILY_CHECKIN_2026-04-15_START -->
# 共识层客户端

## 概述

-   共识客户端（以前称为 eth2 客户端）运行以太坊的权益证明（Proof-of-Stake）共识算法，使网络能够就信标链（Beacon Chain）的头部达成一致。
    
-   **功能边界**：
    
    -   不参与验证/广播交易或执行状态转换（由执行客户端完成）。
        
    -   不进行证明或提议新区块（由验证器客户端完成，这是共识客户端的可选附加组件）。
        

## 当前的客户端

| 客户端名称 | 开发语言 | 开发者 | 状态 |
| --- | --- | --- | --- |
| Lighthouse | Rust | Sigma Prime | 生产环境 |
| Lodestar | TypeScript | ChainSafe | 生产环境 |
| Nimbus | Nim | Status | 生产环境 |
| Prysm | Go | Prysmatic Labs | 生产环境 |
| Teku | Java | ConsenSys | 生产环境 |
| Grandine | Rust | Grandine Developers | 生产环境 |
| Caplin | Go | Erigon | 开发中 |
| LambdaClass | Elixir | LambdaClass | 开发中 |

## 部分介绍

1.  **Lighthouse**
    

-   **特点**：由 Sigma Prime 用 Rust 编写，强调安全性和性能。广泛采用，但在生产环境中需注意避免形成超级多数导致链分裂。
    
-   **功能**：支持跨平台编译（包括 ARM），提供 Slashing Protection（削减保护）、Doppelganger Protection（替身保护）等功能。
    

2.  **Lodestar**
    

-   **特点**：由 ChainSafe 开发的 TypeScript 客户端，适合快速原型设计和浏览器兼容性。
    
-   **功能**：支持信标节点和验证器客户端功能，提供 BLS 和 SSZ 库，支持 MEV 和 Builder 集成。
    

3.  **Prysm**
    

-   **特点**：由 Prysmatic Labs 用 Go 开发，侧重于易用性和可靠性。
    
-   **技术栈**：使用 gRPC 进行进程间通信，BoltDB 进行存储，libp2p 进行网络通信。
    
-   **功能**：包括完整的信标节点实现和验证器客户端。
    

4.  **Nimbus**
    

-   **特点**：由 Status 用 Nim 开发，优化资源效率，支持智能手机和树莓派等轻量级设备。
    
-   **功能**：特征集成验证器客户端支持、远程签名和性能分析工具。
    

5.  **Teku**
    

-   **特点**：ConsenSys 开发的 Java 客户端，提供全面的企业级功能。
    
-   **适用场景**：适合需要可扩展性和操作控制的企业级以太坊部署。
    
-   **功能**：包括 REST API、Prometheus 指标监控和外部密钥管理。
    

6.  **Grandine**
    

-   **特点**：用 Rust 编写，专注于高性能和简洁性，旨在突破以太坊共识层的边界。
    
-   **架构**：核心是并行化和高效资源利用，针对高端机器优化，最小化延迟并最大化吞吐量。
    

7.  **Caplin 与 LambdaClass**
    

-   **Caplin**：集成在 Erigon 执行客户端中，允许在没有外部共识层的情况下运行。
    
-   **LambdaClass**：用 Elixir 编写的客户端，仍在积极开发中，尚未用于生产环境。
<!-- DAILY_CHECKIN_2026-04-15_END -->

# 2026-04-14
<!-- DAILY_CHECKIN_2026-04-14_START -->

# CL 层的P2P通信

-   共识客户端采用 **libp2p** 作为核心 P2P 协议栈，配合 **libp2p-noise** 加密、**discv5** 发现协议、**SSZ** 编码及可选的 **Snappy** 压缩。该架构旨在实现节点间的高效、安全通信。
    

## 实现细节

### **libp2p 网络协议栈**

-   **传输层 (Transport)**：必须支持 TCP（IPv4/IPv6），可选支持 QUIC（基于 UDP，支持多路复用和加密）及 WebRTC。
    
-   **加密与身份 (Encryption)**：使用 **Noise** 框架（XX 模式）进行密钥交换和身份验证，替代了旧的 secio。
    
-   **多路复用 (Multiplexing)**：支持 `mplex` 和 `yamux`，允许在单个连接上并发运行多个逻辑流。
    
-   **地址格式 (Multiaddr)**：采用多层地址编码（如 `/ip4/.../tcp/.../p2p/...`），结合位置与身份信息。
    

### **节点发现与身份**

-   **Peer ID**：基于公钥生成的 Multihash，是节点在网络中的唯一标识。
    
-   **ENR (Ethereum Node Records)**：一种结构化、可签名的记录格式，用于存储节点身份和连接详情，支持灵活的键值对扩展。
    
-   **discv5**：基于 UDP 的发现协议（v5.1），与 libp2p 并行运行，支持动态交换和更新 ENR，确保节点发现的实时性。
    

### **消息传递与优化**

-   **Gossipsub (PubSub)**：用于广播消息（如区块、交易）。文档对比了全连接网格（O(n²)）的低效性，指出 Gossipsub 通过维护部分网格和反熵机制优化了带宽使用。
    
-   **Req/Resp**：用于请求/响应模式的点对点通信。
    
-   **编码与压缩**：全网统一使用 **SSZ**（Simple Serialize）进行序列化以确保 Merkle 化效率；可选使用 **Snappy** 算法压缩数据以提升传输速度。
    

| 层级 | 核心协议 | 功能描述 |
| --- | --- | --- |
| 应用层 | Gossipsub, Ping | 消息广播、心跳检测及自定义逻辑 |
| 多路复用 | Yamux (优先), Mplex | 单连接上并发处理多个数据流 |
| 安全层 | libp2p-noise | 建立加密通道，确保通信私密与认证 |
| 传输层 | TCP, QUIC | 负责机器间的物理或虚拟数据传输 |
| 发现层 | discv5, mDNS | 节点查找、ENR 交换及网络拓扑维护 |
<!-- DAILY_CHECKIN_2026-04-14_END -->

# 2026-04-13
<!-- DAILY_CHECKIN_2026-04-13_START -->


# SSZ

SSZ 是以太坊从工作量证明（PoW）转向权益证明（PoS）后，为了替代执行层（EL）原有的 **RLP（Recursive Length Prefix）** 编码而设计的方案。

## SSZ和RLP的对比

| 维度 | RLP (旧版) | SSZ (新版) |
| --- | --- | --- |
| 设计目标 | 灵活性高，通用性强 | 专为以太坊共识层设计，注重效率 |
| 哈希性能 | 可能，但修改后需全量哈希 | 高效，支持增量哈希（Merkleization） |
| 索引能力 | 差，访问内部数据复杂度高 | 较好，支持部分数据直接访问 |
| 数据类型 | 仅支持字节串和列表，需额外抽象层 | 原生支持复杂数据结构，无需额外抽象 |
| 确定性 | 是 | 是 |

## 处理规则

-   **无符号整数 (uintN)：** 转换为小端序（Little-Endian）字节数组。
    
    -   _示例：_ `uint16` 的 `1025` (0x0401) 序列化为 `01 04`。
        
-   **布尔值 (Boolean)：** `True` 序列化为 `0x01`，`False` 序列化为 `0x00`。
    
-   **向量 (Vectors)：**
    
    -   用于固定长度的同质元素集合。
        
    -   直接将每个元素的字节序列按顺序拼接，无需长度前缀。
        
-   **列表 (Lists)：**
    
    -   用于可变长度（但有最大限制）的同质元素集合。
        
    -   序列化时包含元素数据，但在嵌入容器时会有长度元数据或偏移量处理。
        
-   **位向量与位列表 (Bitvectors & Bitlists)：**
    
    -   **Bitvector：** 固定长度的布尔值序列。按小端序打包进字节，不足补零。
        
    -   **Bitlist：** 可变长度的布尔值序列。末尾添加一个“哨兵位”（Sentinel Bit，值为1）来标记结束位置，以区分实际长度和最大容量。
        
-   **容器 (Containers)：**
    
    -   用于将多个不同类型的字段组合成一个结构。
        
    -   **序列化逻辑：**
        
        1.  **固定大小字段**：直接按顺序序列化。
            
        2.  **可变大小字段**（如列表）：在数据末尾存储，并在固定部分存储**偏移量（Offset）**。
            
        3.  **拼接**：将固定部分（含偏移量）与可变部分拼接。
            

## e.g.

`IndexedAttestation` 容器

-   **结构包含：**
    
    -   `attesting_indices`: 可变大小的列表。
        
    -   `data`: 嵌套的 `AttestationData` 容器。
        
    -   `signature`: 固定大小的签名。
        
-   **序列化结果解析：**
    
    -   **前4字节**：`e4000000`，这是一个偏移量，指向可变大小的 `attesting_indices` 在整个字节流中的起始位置（第228字节）。
        
    -   **后续字节**：依次存储 `slot`、`index`、`beacon_block_root` 等固定大小字段的序列化值。
        
    -   **末尾数据**：存储 `attesting_indices` 的实际索引值。
<!-- DAILY_CHECKIN_2026-04-13_END -->

# 2026-04-10
<!-- DAILY_CHECKIN_2026-04-10_START -->



分享个密码学方案->BBS签名

# BBS 签名

\[Revisiting BBS Signatures\]([https://eprint.iacr.org/2023/275](https://eprint.iacr.org/2023/275))

\[BBS+ Applications, Standardization, and a Bit of Theory\]([https://csrc.nist.gov/csrc/media/presentations/2023/crclub-2023-10-18/images-media/20231018-crypto-club--greg-and-vasilis--slides--BBS.pdf](https://csrc.nist.gov/csrc/media/presentations/2023/crclub-2023-10-18/images-media/20231018-crypto-club--greg-and-vasilis--slides--BBS.pdf))

\[ietf-BBS草案\]([https://www.ietf.org/archive/id/draft-irtf-cfrg-bbs-signatures-05.html](https://www.ietf.org/archive/id/draft-irtf-cfrg-bbs-signatures-05.html))

\[[w3.org](http://w3.org)\]([https://www.w3.org/TR/vc-di-bbs/#bbs-2023-functions](https://www.w3.org/TR/vc-di-bbs/#bbs-2023-functions))

\[[w3.org](http://w3.org)\-\]([https://www.w3.org/community/reports/credentials/CG-FINAL-vc-di-bbs-20230405/](https://www.w3.org/community/reports/credentials/CG-FINAL-vc-di-bbs-20230405/))

## BBS 基本流程

### 1\. 系统参数生成（BBS.Setup）

假设我们有 $\\mathcal{l}$ 条消息：

$$

\\begin{align\*}

G\_1 &\\xleftarrow{\\$ } \\mathbb{G}\_1 \\\\

H\_1,\\dots, H\_\\mathcal{l} &\\xleftarrow{\\$ } \\mathbb{G}\_1 \\\\

G\_2 &\\xleftarrow{\\$ } \\mathbb{G}\_2^\* \\\\

\\mathsf{par} &\\leftarrow (\\mathbb{G}\_1, \\mathbb{G}\_2, \\mathbb{G}\_T, e, G\_1, H\_1, G\_2)

\\end{align\*}

$$

输出公共参数 $\\mathsf{par}$。这里的 $\\vec{H}$ 可以是任意点。

### 2\. 密钥生成（[BBS.KG](http://BBS.KG)）

输入 $\\mathsf{par}$，执行：

$$

x \\xleftarrow{\\$ } \\mathbb{Z}\_p, \\quad X \\leftarrow xG\_2

$$

\- 秘密密钥：$\\mathsf{sk} = x$

\- 公钥：$\\mathsf{pk} = X$

返回 $(\\mathsf{sk}, \\mathsf{vk})$。

### 3\. 签名生成（BBS.Sign）

输入 $\\mathsf{sk} = x$ 和消息向量 $\\mathbf{m} = (m\_1,\\dots,m\_l) \\in \\mathbb{Z}\_p^\\ell$，执行：

1\. 计算消息承诺：

$$

C \\leftarrow G\_1 + \\sum\_{i=1}^\\mathcal{l} m\_i \\cdot H\_i \\in \\mathbb{G}\_1

$$

2\. 随机选择标量 $e$：

$$

e \\xleftarrow{\\$ } \\mathbb{Z}\_p

$$

3\. 计算签名：

$$

A \\leftarrow \\frac{1}{x+e} C \\in \\mathbb{G}\_1

$$

输出签名 $\\sigma = (A, e)$。

**签名长度**：1 个 $\\mathbb{G}\_1$ 点 + 1 个 $\\mathbb{Z}\_p$ 标量（比 BBS+ 少一个标量 $s$）。

### 4\. 签名验证（BBS.Ver）

输入公钥 $X$、消息 $\\mathbf{m}$、签名 $(A, e)$，执行：

1\. 重新计算承诺：

$$

C \\leftarrow G\_1 + \\sum\_{i=1}^\\ell m\_i \\cdot H\_i

$$

2\. 检查配对等式：

$$

e(A,\\ X + e G\_2) \\stackrel{?}{=} e(C, G\_2)

$$

若等式成立（且 $A \\neq \\mathcal{O}$），则验证通过；否则拒绝。

### 5\. 正确性

诚实生成的签名满足 $(x+e)A = C$。

对两边同时与 $G\_2$ 配对：

$$

e((x+e)A,\\ G\_2) = e(C,\\ G\_2)

$$

由双线性得到：

$$

e\\left(A,\\ (x+e) G\_2 \\right) = e(C,\\ G\_2) \\iff e(A,\\ X + e \\cdot G\_2) = e(C,\\ G\_2)

$$

验证等式恒成立。

### 6\. 存在的问题

对于签名 $\\sigma = (A = \\frac{C}{x+e},e)$ 来说，攻击者容易伪造 $\\sigma' = (A^2 = \\frac{C^2}{x+e}, e)$ 也是一个对承诺 $C^2$ 的有效签名。

### 7\. 盲签名构造

非常简单，只需在发送承诺 $C$ 时构造一个随机值变换为 $rC$ 即可， 在接受到签名 $A=\\frac{rC}{x+e}$ 时能很容易消去。

## 零知识披露

**BBS 的 Σ-协议**

**公共输入**（Verifier 已知）：

\- BBS 参数：$G\_1, H\_i\[1.. \\ell\], G\_2, X$

\- 设 $J =$ 已知分量下标，$I =$ 隐藏分量下标（$I \\cup J = \[ \\ell \]$）

\- 部分披露的消息向量 $\\mathbf{m}\_i \\in J$

**私密输入**（Prover 知道）：

\- 完整消息 $\\mathbf{m}$

\- 有效签名 $(A, e)$，$A \\neq \\mathcal{O}$

**目标**：Prover 向 Verifier 证明存在 $(\\mathbf{m}, A, e)$ 使得 BBS.Ver 接受，同时\*\*隐藏\*\* $A, e$ 以及 $\\mathbf{m}\_{i \\in I}$。

### 1\. 完整披露, 对Verifier所有消息已知

**Prover（P1）**：

1\. 随机 $r \\xleftarrow{\\$ } \\mathbb{Z}\_p^\*$

2\. 计算随机化签名点：

$$

\\overline{A} \\leftarrow r \\cdot A

$$

3\. 计算辅助点：

$$

C \\leftarrow G\_1 + \\sum\_{i=1}^\\ell m\_i \\cdot H\_i

$$

$$

\\overline{B} \\leftarrow r \\cdot C -r \\cdot e \\cdot A

$$

4\. 随机 $\\alpha, \\beta \\xleftarrow{\\$ } \\mathbb{Z}\_p$

5\. 计算同态承诺：

$$

U \\leftarrow \\alpha \\cdot C + \\beta \\cdot \\overline{A}

$$

发送 $(\\overline{A}, \\overline{B}, U)$ 给 Verifier。

**Verifier 发送挑战**：

$$

c \\xleftarrow{\\$ } \\mathbb{Z}\_p

$$

**Prover（P2）响应**：

$$

s \\leftarrow \\alpha + r \\cdot c, \\quad t \\leftarrow \\beta - e \\cdot c

$$

发送 $(s, t)$

**FS变换**：

**Prover** 计算

$$

\\mathcal{c} = Hash(\\text{ctx} || \\bar{A} || \\bar{B} || U)

$$

发送 $(\\bar{A}, \\bar{B}, U, s, t)$

**Verifier 检查**（PoK.V）：

1\. 配对检查（确认随机化签名合法）：

$$

e(\\overline{A},\\ X) \\stackrel{?}{=} e(\\overline{B},\\ G\_2)

$$

2\. 同态检查（证明 $\\overline{B}$ 的表示）：

$$

U + c \\cdot \\overline{B} \\stackrel{?}{=} s \\cdot C + t \\cdot \\overline{A}

$$

:::details 正确性

1\. 配对检查：

$$

\\begin{aligned}

lhs &= \\\\

&= e(r \\cdot A, X) \\\\

&= e(A, rxG\_2) \\\\

rhs &= \\\\

&= e(r \\cdot C - r \\cdot e \\cdot A, G\_2) \\\\

&= e(C - \\frac{eC}{x+e}, rG\_2) \\\\

&= e(\\frac{xC}{x+e}, rG\_2) \\\\

&= e(\\frac{C}{x+e}, xrG\_2) \\\\

&= e(A, rxG\_2) \\\\

&= lhs

\\end{aligned}

$$

2\. 同态检查：

$$

\\begin{aligned}

lhs &= U + c \\cdot \\overline{B} \\\\

&= \\alpha C + \\beta \\overline{A} + c(rC - reA) \\\\

&= (\\alpha + rc)C + (\\beta - re) \\overline{A} \\\\

&= sC + t\\overline{A} \\\\

&= rhs

\\end{aligned}

$$

:::

### 2\. 部分披露（Partial Disclosure，对于Verifier隐藏部分消息）

**Prover（P1）**：

1\. 随机 $r \\xleftarrow{\\$ } \\mathbb{Z}\_p^\*$

2\. 计算 $\\overline{A} \\leftarrow r \\cdot A$

3\. 计算公开部分承诺：

$$

C\_J \\leftarrow G\_1 + \\sum\_{j \\in J} m\_j \\cdot H\_j

$$

4\. 计算隐藏部分的辅助点：

$$

\\overline{B} \\leftarrow r \\cdot C -r \\cdot e \\cdot A

$$

5\. 随机 $\\alpha, \\beta \\xleftarrow{\\$ } \\mathbb{Z}\_p$，并对每个隐藏分量 $i \\in I$ 随机 $\\delta\_i \\xleftarrow{\\$ } \\mathbb{Z}\_p$

6\. 计算同态承诺（现在包含隐藏分量）：

$$

U \\leftarrow \\alpha C\_J + \\beta \\overline{A} + \\sum\_{i \\in I} \\delta\_i \\cdot H\_i

$$

发送 $(\\overline{A}, \\overline{B}, U)$ 给 Verifier。

**Verifier 发送挑战** $c \\xleftarrow{\\$ } \\mathbb{Z}\_p$

**Prover（P2）响应**：

$$

s \\leftarrow \\alpha + r \\cdot c, \\quad t \\leftarrow \\beta - e \\cdot c

$$

对每个隐藏分量：

$$

u\_i \\leftarrow \\delta\_i + r \\cdot m\_i \\cdot c \\quad (\\forall i \\in I)

$$

发送 $(s, t, \\{u\_i\\}\_{i \\in I})$

**Verifier 检查**（PoK.V）：

1\. 配对检查（同上）：

$$

e(\\overline{A},\\ X) \\stackrel{?}{=} e(\\overline{B},\\ G\_2)

$$

2\. 同态检查（现在使用公开部分 + 隐藏响应）：

$$

U + c \\cdot \\overline{B} \\stackrel{?}{=} s \\cdot C\_J + t \\cdot \\overline{A} + \\sum\_{i \\in I} u\_i \\cdot H\_i

$$

### FS 变换后的非交互证明

\#### 证明生成（Prover）

输入：公共参数 $\\mathsf{par}$、公钥 $X$、部分披露消息 $\\mathbf{m}\_j$、私密完整消息 $\\mathbf{m}$、私密签名 $(A, e)$ 。$J$ 为公开分量下标集合，$I$ 为隐藏分量下标集合。

1\. 随机选择 $r \\xleftarrow{\\$ } \\mathbb{Z}\_p^\*$

2\. 计算随机化签名点：

$$

\\overline{A} \\leftarrow r \\cdot A

$$

3\. 计算公开部分承诺：

$$

C\_J \\leftarrow G\_1 + \\sum\_{j \\in J} m\_j \\cdot H\_j

$$

4\. 计算辅助点（隐藏签名信息）：

$$

\\overline{B} \\leftarrow r \\cdot C -r \\cdot e \\cdot A \\quad \\text{（其中 } C = G\_1 + \\sum\_{i=1}^\\ell m\_1 \\cdot H\_i\\text{）}

$$

5\. 随机选择：

$\\alpha, \\beta \\xleftarrow{\\$ } \\mathbb{Z}\_p$

对每个隐藏分量 $i \\in I$：$\\delta\_i \\xleftarrow{\\$ } \\mathbb{Z}\_p$

6\. 计算同态承诺：

$$

U \\leftarrow \\alpha C\_J + \\beta \\overline{A} + \\sum\_{i \\in I} \\delta\_i \\cdot H\_i

$$

7\. **计算Fiat-Shamir 挑战**：

计算

$$

c \\leftarrow H\\bigl( \\mathsf{ctx} \\, \\| \\mathbf{m}\_j \\,\\|\\, \\overline{A} \\,\\|\\, \\overline{B} \\,\\|\\, U \\bigr)

$$

（$\\mathsf{ctx}$ 为领域分隔字符串，例如 `"BBS_PoK_1"` 或具体应用定义的上下文，防止重放/混淆攻击，例如在Dapp1生成的c用到了Dapp2中）

8\. 计算响应：

$$

s \\leftarrow \\alpha + r \\cdot c, \\quad t \\leftarrow \\beta - e \\cdot c

$$

对每个隐藏分量 $i \\in I$：

$$

u\_i \\leftarrow \\delta\_i + r \\cdot m\_i \\cdot c

$$

**输出证明** $\\pi$:

$$

\\pi = \\bigl( \\overline{A},\\ \\overline{B},\\ U,\\ s,\\ t,\\ \\{u\_i\\}\_{i \\in I} \\bigr)

$$

（共 **2 个 $\\mathbb{G}\_1$ 点 + (k + 3) 个 $\\mathbb{Z}\_p$ 标量**，$k = |I|$）

### 验证（Verifier）

输入：$\\mathsf{par}$、$X$、$\\mathbf{m}\_j$、证明 $\\pi$。

1\. 重新计算公开部分承诺：

$$

C\_J \\leftarrow G\_1 + \\sum\_{j \\in J} m\_j \\cdot H\_j

$$

2\. **重新计算挑战**（与 Prover 完全相同）：

$$

c' \\leftarrow H\\bigl( \\mathsf{ctx} \\, \\| \\mathbf{m}\_j \\,\\|\\, \\overline{A} \\,\\|\\, \\overline{B} \\,\\|\\, U \\bigr)

$$

3\. **检查两个等式**（若任意一个失败则拒绝）：

**配对检查**（确认随机化签名合法）：

$$

e(\\overline{A},\\ X\_2) \\stackrel{?}{=} e(\\overline{B},\\ g\_2)

$$

**同态检查**（确认 $\\overline{B}$ 的表示正确）：

$$

U + c' \\cdot \\overline{B} \\stackrel{?}{=} s \\cdot C\_J + t \\cdot \\overline{A} + \\sum\_{i \\in I} u\_i \\cdot H\_i

$$

若两个检查都通过，则接受证明（证明者知道有效 BBS 签名且公开消息与 $\\mathbf{m}\_j$ 一致）。
<!-- DAILY_CHECKIN_2026-04-10_END -->

# 2026-04-09
<!-- DAILY_CHECKIN_2026-04-09_START -->




## EVN内存结构 - 栈

\### 1. 栈的结构特性

| 特性 | 详细说明 |

| :--- | :--- |

| **数据单位** | **Word (字)**。EVM 的字长为 **32 字节 (256位)**。栈中每个元素都是一个 32 字节的值。 |

| **最大深度** | **1024 项**。如果栈深度超过 1024，程序会因栈溢出（Overflow）而失败。 |

| **访问限制** | **仅限顶部 16 项**。EVM 只能直接操作栈顶附近的元素，无法直接访问底部的数据。 |

| **生命周期** | **瞬时性**。栈是临时的，它在每次合约执行开始时被创建，并在执行结束时被重置（销毁）。 |

### 2\. Opcodes

EVM 提供了三类最基本的栈操作指令：\*\*压栈/弹栈\*\*、\*\*复制\*\* 和 **交换**。

\#### A. 基础操作：PUSH 与 POP

\* **PUSH (压栈):** 将数据放入栈顶。

\* `PUSH1` 。例如 `PUSH1 06` 会将数值 `06` 放入栈顶。

\#### B. 进阶操作：DUP (复制)

为了进行多步计算，我们需要重复使用栈中的某些值。EVM 提供了 `DUP1` 到 `DUP16` 指令。

-   **原理：** 复制栈中第 N 个位置的元素，并将其放到栈顶。
    
-   **示例：**
    

\* 假设栈顶（位置 1）是数值 A。

\* 执行 `DUP1` 后，栈顶会变成 A, A（栈顶有两个 A）。

\* 执行 `DUP2` 会复制位于第 2 个位置的元素到栈顶。

\#### C. 进阶操作：SWAP (交换)

当我们需要调整计算顺序时，使用 `SWAP`。

\* **原理：** 交换栈顶元素和第 N+1 个元素的位置。

\* **示例：**

\* 假设栈从顶到底为：B, A。

\* 执行 `SWAP1` 后，栈变为：A, B。

\### 3. 栈的实际工作流程演示

**目标：** 计算 $6 + 7 = 13$ (0x0D)

**代码逻辑：**

1\. `PUSH1 06` (将 6 放入栈)

2\. `PUSH1 07` (将 7 放入栈)

3\. `ADD` (弹出栈顶两个数，计算后将结果压回)

**执行过程追踪：**

| 步骤 | 执行的指令 | 栈顶 (Top) 到 栈底 (Bottom) | 说明 |

| :--- | :--- | :--- | :--- |

| **初始** | - | _(空)_ | 栈是空的 |

| **1** | `PUSH1 06` | **06** | 将 6 压入栈，现在栈深为 1 |

| **2** | `PUSH1 07` | **07**, 06 | 将 7 压入栈，7 在 6 的上面 (栈深为 2) |

| **3** | `ADD` | **0D** (即 13) | 1. `ADD` 弹出 7 和 6 2. 计算 7+6=13 3. 将 13 压回栈顶 |
<!-- DAILY_CHECKIN_2026-04-09_END -->

# 2026-04-08
<!-- DAILY_CHECKIN_2026-04-08_START -->





## 以太坊预编译合约

-   BLS12-381 配对
    
    -   地址：0x0f
        
    -   最小GAS：37700
        
    -   结构：384bytes
        
        -   | Byte range | Name | Description |
            | --- | --- | --- |
            | [0; 63] (64 bytes) | g1_x | X coordinate of G1 point |
            | [64; 127] (64 bytes) | g1_y | Y coordinate of G1 point |
            | [128; 191] (64 bytes) | g2_x_c0 | X coordinate c0 component of G2 point |
            | [192; 255] (64 bytes) | g2_x_c1 | X coordinate c1 component of G2 point |
            | [256; 319] (64 bytes) | g2_y_c0 | Y coordinate c0 component of G2 point |
            | [320; 383] (64 bytes) | g2_y_c1 | Y coordinate c1 component of G2 point |
            
    -   调用：result:=staticall(staticcall(gas(), 0x0F, add(input, 0x20), mul(inputSize, 0x20), out, 0x20)
        
-   BN254 配对
    
    -   地址：0x08
        
    -   最小gas：45000
        
    -   size: 192bytes
        
    -   | Byte range | Name |
        | --- | --- |
        | [0; 31] (32 bytes) | g1_x |
        | [32; 63] (32 bytes) | g1_y |
        | [64; 95] (32 bytes) | g2_x_c1 |
        | [96; 127] (32 bytes) | g2_x_c0 |
        | [128; 159] (32 bytes) | g2_y_c1 |
        | [160; 191] (32 bytes) | g2_y_c0 |
        
-   call(not(0), 0x08, 0, input, 0x0180, input, 0x20)
<!-- DAILY_CHECKIN_2026-04-08_END -->

# 2026-04-07
<!-- DAILY_CHECKIN_2026-04-07_START -->






\[EVM\]([https://epf.wiki/#/wiki/EL/evm](https://epf.wiki/#/wiki/EL/evm))

EVM 是以太坊的核心执行引擎，负责解释并执行智能合约的字节码。它是一个\*\*栈式（Stack-based）\*\*、\*\*确定性\*\*的虚拟机，通过 **Gas** 机制防止滥用和无限循环。

### EVM 的核心组成与工作原理

\- **主要组件**：

\- **栈（Stack）**：256位宽，用于存放操作数。

\- **内存（Memory）**：运行时动态扩展的字节数组（按32字节字寻址）。

\- **存储（Storage）**：持久化的键值对（合约的永久状态，通过 Merkle Trie 组织）。

\- **操作码（Opcodes）**：一套指令集，支持算术、逻辑、合约调用、创建等操作。

\- **程序计数器（Program Counter）** 和 **Gas**：控制执行流程和资源消耗。

\- **执行过程**：

EVM 通过解释器循环逐条处理字节码。每次执行一个操作码时，会更新栈、内存、存储，并扣除相应 Gas。

执行分为\*\*正常终止（Halting）\*\*（如 STOP、RETURN）和\*\*异常终止\*\*（REVERT、错误等），失败时状态会回滚（原子性）。

\- **正常终止（Normal Halting）** 的关键步骤：

\- 从栈顶读取输出起始位置和长度。

\- 检查并扩展内存（计算内存扩展成本并扣除 Gas）。

\- 生成输出数据并更新机器状态，标记执行结束。

### 区块与状态转换

**σ\_{t+1} ≡ Π(σ\_t, B)**

（旧世界状态 + 当前区块 → 新世界状态）

主要流程包括：

1\. **区块头验证**：检查 gasUsed ≤ gasLimit、时间戳、baseFee（EIP-1559）、blobGas（EIP-4844）等。

2\. **初始化环境**：设置调用者、区块信息、prevRandao、excessBlobGas 等。

3\. **交易执行**：

\- 计算\*\*内在 Gas\*\*（Intrinsic Gas）：数据成本、创建成本、访问列表成本等。

\- 支持不同交易类型（0、1、2、3），其中类型2（EIP-1559）和类型3（Blob 交易）有特殊费用模型。

\- **EIP-1559**：引入动态 **baseFee**（被燃烧销毁）和 **priorityFee**（小费给验证者）。

\- **Blob 交易**（EIP-4844）：额外 blobGas 计算，用于降低 Layer 2 数据成本。

4\. **Gas 会计**：预扣费用 → 执行 → 实际消耗 + 退款。

5\. **最终验证**：状态根（State Root）、收据根（Receipts Root）、累计 Gas 等必须匹配区块头承诺。

整个区块执行是\*\*原子\*\*的：要么全部成功，要么全部回滚。

### 重要机制与特点

\- **Gas 机制**：不仅计量资源，还包括内存扩展成本、冷/热存储访问成本（EIP-2929）等。

\- **确定性**：相同输入在任何节点执行结果完全一致。

\- **预编译合约**：地址 1~9 的特殊合约，提供高效的密码学操作（如 ECDSA 验证、哈希等）。

\- **EIP 相关更新**：大量涉及 EIP-1559（费用市场）、EIP-4844（Proto-Danksharding）、EIP-4788（Beacon 根访问）等后合并时代特性。
<!-- DAILY_CHECKIN_2026-04-07_END -->

# 2026-04-06
<!-- DAILY_CHECKIN_2026-04-06_START -->







## 以太坊设计理念

-   \[REF\]([https://epf.wiki/#/wiki/protocol/design-rationale](https://epf.wiki/#/wiki/protocol/design-rationale))
    

### 协议设计哲学

以太坊从诞生起就遵循以下核心理念：

-   Simplicity（简单性）：协议设计力求简单，让普通程序员也能理解和实现整个规范，从而减少少数精英开发者对协议的控制。尽管协议不断演进，简单性仍通过\*\*模块化\*\*和清晰规范得以维持。
    
-   Universality（通用性）：以太坊“没有内置特性”（Ethereum has no features）。它只提供一个图灵完备的 EVM（以太坊虚拟机），开发者可以用它构建任何数学上可定义的智能合约或交易类型，实现真正去中心化、信任最小化的应用。
    
-   Modularity（模块化）：这是未来-proof 的关键。协议修改应尽量局部化，不影响上层应用。许多创新（如 Patricia 树、SSZ、Proto-Danksharding）被实现为独立库，甚至可用于其他协议。核心理念是 encapsulated complexity（封装的复杂性）：子系统内部复杂，但对外提供简单、高层次接口，便于灵活选择和调试。
    
-   Non-discriminant（非歧视性）：源于 FOSS 和 Cypherpunk 运动。协议不主动限制或禁止特定应用类型，所有监管机制只针对协议本身的伤害（如通过费用调节），而非针对具体应用。例如，只要支付足够费用，甚至可以运行无限循环脚本。
    
-   Agility（敏捷性）：协议细节不是一成不变的。通过 Ethereum Improvement Process（EIP） 开放提案。修改 EVM 等高层结构时非常谨慎，但如果能显著提升可扩展性或安全性，就会采用。
    

### 核心原则（Principles）

-   Managing Complexity（管理复杂性）：目标是让协议尽可能简单，同时满足区块链所需功能。复杂性难以精确定义，常需权衡不同类型：
    
-   Sandwich model：简化底层架构和用户接口，将复杂性推到“中间层”（编译器、序列化、存储接口、wire protocol 等，用户和核心共识都看不到）。
    
-   Encapsulated complexity：优先选择内部复杂但接口简单的子系统。系统性复杂（各部分相互纠缠）比封装的复杂更危险。
    
-   复杂性放置偏好顺序：\*\*Layer 2 > 客户端实现 > 协议规范\*\*。
    
-   Freedom（自由）：用户使用协议不应受限，类似“网络中立性”。反对像比特币那样通过协议变更（如限制 OP\_RETURN）打压特定用途。以太坊倾向于 Pigovian taxation（庇古税）：通过费用让产生链上膨胀的用户自行承担成本。
    
-   Generalization（泛化）：协议特性和操作码应体现最底层的概念，便于任意组合（包括未来可能有用的方式），并可通过去除多余功能来优化效率。例如，使用 `LOG` 操作码分离“函数调用”和“外部事件日志”，而非简单记录所有消息。
    
-   We have no features（我们没有特性）：作为泛化的推论，即使常见的高层用例，也不内置到协议中，而是让用户通过合约实现子协议（例如 ether-backed 子货币、侧链）。比特币式的 locktime 功能在以太坊中可通过合约模拟。
    

### 区块链层面的具体设计选择（Blockchain Level Protocol）

Accounts over UTXOs（账户模型优于 UTXO）：

-   比特币等早期区块链使用 UTXO（未花费交易输出） 模型。
    
-   以太坊选择 账户模型（每个账户有 nonce、余额、代码）。
    
-   优势：空间节省（交易更小）、完美 fungibility（可互换性）、实现更简单、支持复杂交易（如去中心化交易所）。
    
-   缺点：需 nonce 防重放攻击，导致旧账户无法修剪（prune）。解决方案：交易中加入块号，定期重置 nonce。
    
-   总体上，账户模型更灵活，符合以太坊构建复杂应用的初衷。
    

> &emsp;

Merkle Patricia Trie (MPT)：

-   以太坊状态数据的核心结构（修改版的 Merkle-Patricia Trie）。
    
-   优点：确定性、密码学可验证、支持 Merkle 证明，插入/查找/删除为 O(log n)。
    
-   问题：状态大小已达 1-2TB，不利于节点存储和无状态化（statelessness）。
    
-   未来方向：考虑替换为 Verkle trees（向量承诺 Merkle 树），提供更小的 membership proofs（witnesses），支持高效的无状态客户端。
    

> &emsp;

序列化方案：

-   RLP（Recursive Length Prefix）：以太坊早期使用的确定性序列化方案。只处理嵌套数组结构，不定义具体数据类型（如整数、布尔），保证字节级一致性。缺点：不支持 Merkleization，对固定大小类型效率低。
    
-   SSZ（Simple Serialize）：以太坊 2.0 Beacon Chain 采用。基于预知 schema，支持变长/定长类型，高效 re-hashing 和索引，支持 Merkleization，是实现无状态化的关键。
    

> &emsp;

最终性（Finality）与共识：

-   在 PoS 下，最终性指块无法被更改，除非燃烧至少 33% 的总质押 ETH。
    
-   Casper FFG：叠加在提案机制上的最终化工具，通过验证者投票和 slashing 实现检查点最终化。
    
-   LMD GHOST：分叉选择规则（fork-choice rule），基于最新消息选择最重子树。
    
-   Gasper：Casper FFG + LMD GHOST 的组合，构成以太坊 PoS 共识的理想化抽象。
    

> &emsp;

网络层：使用 DHT：

-   使用基于 Kademlia 的 discv5 DHT 进行节点发现（存储 ENR 记录，实现对数级通信开销）。
    
-   区块传播则使用 gossipsub（非结构化覆盖网络）。
    
-   混合模式：DHT（结构化）用于引导启动，提供全局视图；随后进入非结构化 gossip 网络，适合高 churn 环境。DHT 的最大优势是设计简单。
<!-- DAILY_CHECKIN_2026-04-06_END -->
<!-- Content_END -->
