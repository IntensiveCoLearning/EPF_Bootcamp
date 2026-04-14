---
timezone: UTC+8
---

# Macy

**GitHub ID:** L-Macy

**Telegram:** 

## Self-introduction

EPF 实习计划

## Notes

<!-- Content_START -->
# 2026-04-14
<!-- DAILY_CHECKIN_2026-04-14_START -->
Ethereum 共识层（Gasper）概述Ethereum 共识由 LMD GHOST（分叉选择）+ Casper FFG（最终性）组成，合称 Gasper。其核心目标是在不可靠的网络和硬件环境下，让全球数万节点维护一份完全一致的账本。PoW/PoS 仅是 Sybil 抵抗机制，并非共识本身。2022 年 The Merge 后，Ethereum 从 PoW 切换到 PoS，由 Beacon 链接管共识，使用质押 ETH 的验证者（最低 32 ETH）通过 proposer 和 committees 产生区块与 attestations。时间结构以 12 秒一个 slot、32 个 slot 为一个 epoch 为基础。每个验证者每 epoch 进行一次 attestation，同时包含 LMD GHOST（选链头）和 FFG（检查点最终性）投票。通过 checkpoints 的 justification 与 finality 机制，实现交易约 14-16 分钟最终确认。Deneb 升级引入 EIP-4844（proto-danksharding），支持 blobs 用于 L2 数据可用性，大幅降低成本。系统通过奖励（attester/proposer）、惩罚及 slashing 激励诚实行为，严重违规或长期不活跃会触发 inactivity leak 机制。总体而言，Ethereum 共识层通过经济激励、密码学随机性和两阶段最终性，在去中心化与安全性之间取得平衡。
<!-- DAILY_CHECKIN_2026-04-14_END -->

# 2026-04-13
<!-- DAILY_CHECKIN_2026-04-13_START -->

以太坊共识层（Consensus Layer）核心笔记1. 核心目标

-   在不可靠的基础设施（消费级硬件、不可靠网络、可能存在恶意节点）上，构建可靠的分布式系统
    
-   让全球数万个独立节点维护完全一致的账本（状态）
    
-   确保所有诚实节点最终对单一交易历史达成共识（包括交易顺序和结果）
    

2\. 关键概念

-   拜占庭容错 (BFT)：系统能容忍部分节点故障或恶意行为（拜占庭故障）
    
-   共识协议：解决拜占庭将军问题，让不可靠节点达成一致
    
-   PoW 与 PoS：不是共识协议，而是抗女巫攻击机制 + 分叉选择规则
    
    -   PoW：以计算量赋予链权重
        
    -   PoS：以质押价值赋予链权重
        

3\. 以太坊共识协议（Gasper）

-   由两种协议组合而成：
    
    -   LMD GHOST：分叉选择规则（选最重链）
        
    -   Casper FFG：最终性工具（提供经济最终性）
        
-   两者结合称为 Gasper
    

4\. 从 PoW 到 PoS 的转变（The Merge，巴黎升级）

-   2022 年通过 终端总难度 (TTD) 触发，而非区块高度
    
-   信标链（Beacon Chain）接管区块生产
    
-   矿工 → 验证者（Validators）
    
-   主要优势：大幅降低能耗 + 提升可扩展性潜力
    

5\. 信标链（Beacon Chain）核心职责

-   管理 PoS 共识机制
    
-   协调验证者（Validators）
    
-   处理区块提议（Proposing）与见证（Attesting）
    

6\. 验证者（Validators）

-   最低质押：32 ETH
    
-   最多有效质押：2048 ETH
    
-   主要职责：
    
    -   提议区块（随机选中）
        
    -   对区块进行见证（Attest）→ 投票
        
    -   参与共识投票
        
-   激励机制：质押 ETH 作为抵押品，作恶会被罚没（Slashing）
    

以太坊共识层通过 Gasper (LMD GHOST + Casper FFG) + PoS，让数万个不可靠节点在信标链的协调下，安全、高效地就单一链状态达成拜占庭容错共识。
<!-- DAILY_CHECKIN_2026-04-13_END -->

# 2026-04-12
<!-- DAILY_CHECKIN_2026-04-12_START -->


vibecoding的一天

![{D1D1B0EE-020D-4EFF-BB17-3BCBAC890E23}.png](https://raw.githubusercontent.com/IntensiveCoLearning/EPF_Bootcamp/main/assets/L-Macy/images/2026-04-12-1776009460576-_D1D1B0EE-020D-4EFF-BB17-3BCBAC890E23_.png)
<!-- DAILY_CHECKIN_2026-04-12_END -->

# 2026-04-11
<!-- DAILY_CHECKIN_2026-04-11_START -->



Execution Layer (EL) 基础部分EL Specs（执行层核心规范）

-   主要参考：Ethereum Execution Layer Specifications (EELS) —— Python 参考实现，注重可读性，用于原型 EIP 和状态测试。包含各分叉快照和 diff。
    
-   Yellow Paper：经典 EVM 正式规范（虽较旧，仍是基础）。
    
-   核心内容：区块处理、交易验证、状态转换函数（State Transition Function）、Gas 计算等。
    
-   EELS 优势：模块清晰（EVM 模块 + 区块链执行模块），易于追踪协议演进。
    
    [openzeppelin.com](http://openzeppelin.com)
    

Client architecture（客户端架构概述）（EL 客户端模块总览）

-   EL Client 职责：
    
    -   执行交易与智能合约（EVM）。
        
    -   维护世界状态（State Trie）。
        
    -   处理 JSON-RPC 请求。
        
    -   P2P 传播交易/区块。
        
-   典型客户端：Geth（Go）、Nethermind（.NET）、Besu（Java）、Erigon、Reth。
    
-   与 CL 交互：通过 Engine API 接收执行 payload，执行后返回结果。
    
-   模块总览：EVM 解释器、状态数据库、交易池（mempool）、区块构建逻辑。
<!-- DAILY_CHECKIN_2026-04-11_END -->

# 2026-04-10
<!-- DAILY_CHECKIN_2026-04-10_START -->




今天主要围绕 Ethereum 交易的核心数据结构、编码规则、提交机制和执行链路进行学习。结合官方文档（[ethereum.org](http://ethereum.org)）和社区笔记（如 learn\_blockchain GitHub 学习记录、Medium 技术文章），我把零散知识点系统化成“从用户侧构造 → 网络传播 → 执行层处理”的完整流程。同时补充了实际编码示例和客户端/链上边界划分，解决了我之前对 RLP 抽象、EVM 复杂度的困惑。1. Transaction 的核心字段交易本质是一个签名指令，由 EOA（外部拥有账户）发出，用于状态变更（转账、合约调用或部署）。核心字段（以传统 Type 0 交易为例，后续有 Type 1/2/3/4 扩展）：

-   nonce：发送者账户的交易序号（防止重放攻击，必须严格递增）。
    
-   gas（gasLimit）：本次交易允许消耗的最大 Gas 量（简单 ETH 转账固定 21,000 Gas）。
    
-   to：接收地址（转账/调用合约时填写；部署合约时留空）。
    
-   value：转账金额（单位 wei，1 ETH = 10¹⁸ wei）。
    
-   data（或 input）：合约调用数据（前 4 字节是函数选择器，后续是 ABI 编码参数）或合约字节码。
    

其他重要字段（EIP-1559 后常见）：

-   maxFeePerGas / maxPriorityFeePerGas：动态 Gas 费用（base fee + tip）。
    
-   v, r, s：ECDSA 签名组件（证明所有权 + 防重放）。
    

交易对象示例（未签名）：

json

```json
{
  "nonce": "0x0",
  "gasLimit": "0x5208",  // 21000
  "to": "0x...", 
  "value": "0x...", 
  "data": "0x...",
  "maxFeePerGas": "0x...",
  "maxPriorityFeePerGas": "0x..."
}
```

注意：Type 0（Legacy）用 gasPrice；Type 2（EIP-1559）用动态费用；Type 4（Pectra）支持 Account Abstraction。

[ethereum.org](http://ethereum.org)

2\. RLP Serialization（交易编码规则）RLP（Recursive Length Prefix）是 Ethereum 执行层统一的序列化格式，用于交易、区块、状态树等数据的紧凑、二进制传输。核心目标：结构化 + 确定性 + 最小字节，不关心数据语义（整数、字符串全当字节数组处理）。RLP 编码核心规则（官方总结）

-   单个字节（0x00–0x7f）：直接输出。
    
-   字符串/字节数组（0–55 字节）：0x80 + 长度 + 数据。
    
-   长字符串（>55 字节）：0xb7 + 长度字节数 + 长度（big-endian）+ 数据。
    
-   正整数：转最短 big-endian 字节（0 视为空数组 0x80），再按字符串规则编码（禁止前导零）。
    
-   列表：先把所有元素 RLP 编码后拼接，再加列表前缀（0xc0 + 长度 或长列表前缀）。
    

交易 RLP 编码格式

-   Legacy (Type 0)：RLP(\[nonce, gasPrice, gasLimit, to, value, data, v, r, s\])
    
-   Typed 交易：TransactionType (1 字节) || RLP(具体 payload)
    

简单示例（"dog" 字符串）：

-   0x83 64 6f 67（0x83 = 0x80 + 3）
    

列表示例（\["cat", "dog"\]）：

-   0xc8 83 63 61 74 83 64 6f 67
    

交易实际编码（SDK 会自动完成）：

python

```python
# 伪代码（官方文档风格）
tx_fields = [nonce, gas_price, gas_limit, to, value, data, v, r, s]
encoded = rlp_encode(tx_fields)  # 最终 raw transaction（hex 格式）
```

RLP 让交易在 P2P 网络中高效传播，同时保证所有节点解析结果完全一致。

[ethereum.org](http://ethereum.org)

我的补充体验：之前觉得抽象，现在通过规则手算小例子后，理解了“长度前缀 + 递归”的设计精髓。实际开发中用 ethers.js / [web3.py](http://web3.py) 自动处理，无需手动写，但调试 raw tx 时会用到。3. JSON-RPC 接口在交易发送中的作用JSON-RPC 是客户端与节点交互的标准接口（轻量、无状态）。交易发送全流程依赖它：

-   eth\_signTransaction：节点帮你签名（返回 RLP 编码后的 signed tx）。
    
-   eth\_sendRawTransaction：最常用，传入已签名的 raw tx（RLP hex），节点验证后放入 mempool 并广播。
    
-   eth\_sendTransaction：节点直接签名 + 发送（适用于 unlocked 账户）。
    
-   查询类：eth\_getTransactionByHash、eth\_getTransactionReceipt（确认执行结果、gasUsed、logs）。
    

作用总结：JSON-RPC 是“客户端 → 执行层节点”的桥梁，负责提交与查询，不负责实际执行。

[ethereum.org](http://ethereum.org)

4\. Execution Layer（执行层）在交易处理中的职责Post-Merge 后，Ethereum 分成 Execution Layer (EL) 和 Consensus Layer (CL)：

-   EL 职责（Geth、Nethermind 等客户端）：
    
    -   接收交易 → 验证签名、nonce、Gas → 放入 mempool（pending/queue）。
        
    -   打包进区块后：EVM 执行（opcode 逐条运行、Gas 扣除、状态变更）。
        
    -   更新世界状态（State Trie）、生成 Receipt。
        
    -   通过 Engine API 与 CL 交互（CL 只管共识，EL 管执行）。
        
-   EVM 是 EL 的核心虚拟机，负责解释合约字节码、计算 Gas、维护状态一致性。
    

边界清晰：EL 只管计算与状态，不决定区块最终确认（那是 CL 的 RANDAO + 证明）。

[github.com](http://github.com)

5\. 整体流程（从构造到上链）

1.  用户侧 / 客户端：构造 tx 对象 → 本地签名（私钥生成 v,r,s）→ RLP 编码成 raw tx。
    
2.  JSON-RPC 发送：eth\_sendRawTransaction → 节点接收。
    
3.  Mempool：节点验证 → 放入交易池（P2P gossip 传播给其他节点）。
    
4.  打包：Proposer 节点从 mempool 选 tx → 打包进区块。
    
5.  EVM 执行（EL）：按 nonce 顺序执行 → Gas 消耗 → 状态变更 → 生成 Receipt。
    
6.  上链确认：区块被 CL 最终确认 → 交易不可逆。
    

客户端完成：构造、签名、RLP、RPC 发送。  
链上（EL）完成：验证、mempool、EVM 执行、状态更新。

[ethereum.org](http://ethereum.org)

6\. 遇到的问题与补充

-   RLP 抽象 → 现在通过规则 + 伪代码已能手算简单案例，实际用 SDK 即可；调试时推荐 ethers.js utils.serializeTransaction 查看 raw hex。
    
-   EVM 执行复杂 → 重点关注 Gas 计量（每个 opcode 有固定 Gas 价）和状态变更；初期不用深挖所有 opcode，先理解“交易 → Message → EVM 执行栈”即可。
    
-   客户端 vs 执行层边界 → 客户端只负责“提交正确格式的签名 tx”，执行层负责“安全执行并更新状态”。这个划分让 AA（Account Abstraction）成为可能（未来用户操作可被抽象成 UserOp，而非传统 tx）。
    

7\. 思考与收获

-   交易不再是孤立的“数据结构”，而是一条完整的执行链路：客户端构造 → 网络传播 → EL 执行 → 状态终态。
    
-   职责划分极致清晰：客户端 = 构造与签名，执行层 = 验证与计算，这为后续学习 AA、P2P 传播、Engine API 打下坚实基础。
    
-   最大收获：从“背知识点”转向“画流程图”，理解了为什么 RLP、JSON-RPC、mempool 各司其职。
<!-- DAILY_CHECKIN_2026-04-10_END -->

# 2026-04-09
<!-- DAILY_CHECKIN_2026-04-09_START -->





Evolution部分系统梳理了以太坊协议从诞生到现在的完整升级脉络与历史演进，Execution Layer（EL）作为交易执行与状态维护的核心引擎，在每一次升级中都承担着关键角色。从2015年Frontier版本正式上线开始，以太坊以PoW共识机制运行，早期通过Homestead升级完善基础稳定性；2016-2019年间先后经历Byzantium、Constantinople、Istanbul等硬分叉，重点优化EVM执行效率、Gas费用计算和智能合约安全性，同时引入难度炸弹机制为后续PoS转型做准备；2021年London升级引入EIP-1559动态费用机制，彻底改变了Gas定价逻辑，让基础费用可销毁、优先费直接给矿工，显著提升了用户体验与网络经济模型；2022年9月The Merge（Paris升级）实现历史性PoW向PoS切换，Execution Layer与Consensus Layer正式分离，EL继续负责EVM执行、交易打包和世界状态维护，而CL接管信标链验证与最终性，Merge后EL通过Engine API与CL无缝交互，彻底解决能耗问题并大幅提升安全性；2023年上海+Capella升级（Shapella）开启质押ETH提款功能，让用户终于能从信标链提取资金；2024年Dencun升级（Deneb+Cancun）引入EIP-4844 Blob数据，极大降低了Layer2的Rollup上链成本，为大规模扩展铺平道路；后续Pectra（Prague+Electra）路线图将继续推进单槽最终性、账户抽象、EVM对象格式优化等，Execution Layer在整个演进过程中始终保持稳定运行，确保协议在去中心化、安全性和可扩展性之间动态平衡，为Layer2繁荣和全栈开发者生态奠定了坚实基础。
<!-- DAILY_CHECKIN_2026-04-09_END -->

# 2026-04-08
<!-- DAILY_CHECKIN_2026-04-08_START -->







EL Specs（执行层核心规范）

-   主要参考：Ethereum Execution Layer Specifications (EELS) —— GitHub 上的 Python 可执行规范仓库（[https://github.com/ethereum/execution-specs）。](https://github.com/ethereum/execution-specs）。)
    
-   EELS 是 Yellow Paper（黄皮书）的现代继任者，更适合开发者阅读和原型开发。
    
-   黄皮书仍是正式数学规范，但 EELS 提供每个分叉的完整快照、清晰代码实现和 diff，便于理解状态转移规则、EVM 执行逻辑等。
    
-   EELS 不包含 JSON-RPC 和 P2P 网络，只专注执行层核心（状态转移、区块/交易处理）。
    

Client Architecture（客户端架构概述）

-   执行层客户端（EL Client，也称 Execution Engine）负责：
    
    -   接收并验证交易
        
    -   在 EVM 中执行智能合约
        
    -   维护世界状态（World State）和数据库
        
    -   构建区块 payload 并通过 Engine API 与共识层（CL）通信
        
-   流行 EL 客户端：Geth (Go)、Erigon、Besu、Nethermind、EthereumJS 等。
    
-   节点实际运行两个客户端：EL + CL（The Merge 后分离）。
    
-   主要模块包括：EVM 执行引擎、交易池（mempool）、状态 Trie、区块链数据库、JSON-RPC 接口。
<!-- DAILY_CHECKIN_2026-04-08_END -->

# 2026-04-07
<!-- DAILY_CHECKIN_2026-04-07_START -->








密码学是研究信息加密与安全传输的学科，主要通过将明文转换为密文来保护数据不被未授权访问。常见方法包括对称加密和非对称加密，其中对称加密使用同一密钥进行加解密，效率较高，而非对称加密利用公钥和私钥提高安全性。此外，哈希函数和数字签名也在数据完整性和身份认证中发挥重要作用，是现代信息安全的重要基础。
<!-- DAILY_CHECKIN_2026-04-07_END -->
<!-- Content_END -->
