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
