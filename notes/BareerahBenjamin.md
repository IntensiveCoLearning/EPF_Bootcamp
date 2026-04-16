---
timezone: UTC+8
---

# BareerahBenjamin

**GitHub ID:** BareerahBenjamin

**Telegram:** 

## Self-introduction

EPF 实习计划

## Notes

<!-- Content_START -->
# 2026-04-16
<!-- DAILY_CHECKIN_2026-04-16_START -->
# 共识客户端

## 概览

\- 共识客户端（Consensus client，原“eth2 client”）运行以太坊 PoS 共识算法，维护 Beacon Chain 的头部和状态。

\- 它不执行交易、不维护执行层状态，这些由执行客户端负责；它也不直接提议 / 证明区块，这些由可选的 validator client 完成。

## 1\. 主流共识客户端一览

这些客户端用不同语言实现，特性和性能侧重点各异，都支持主网和活跃测试网，客户端多样性是选择时的重要考量。

| Client | Language | Developer | Status |

| --- | --- | --- | --- |

| Lighthouse | Rust | Sigma Prime | Production |

| Lodestar | TypeScript | ChainSafe | Production |

| Nimbus | Nim | Status | Production |

| Prysm | Go | Prysmatic Labs | Production |

| Teku | Java | ConsenSys | Production |

| Grandine | Rust | Grandine Devs | Production |

| Caplin | Go | Erigon | Development |

| LambdaClass | Elixir | LambdaClass | Development |

## 2\. 客户端分布与多样性

\- 目前绝大部分节点运营者使用 Prysm 或 Lighthouse 作为共识客户端。

\- 为了 Beacon 链整体健康，推荐刻意选择少数派客户端，降低单一实现的系统性风险（详见 [clientdiversity.org](http://clientdiversity.org) 的说明）。

## 3\. 各客户端简要笔记

### Lighthouse（Rust, Sigma Prime）

\- 定位：高安全性、高性能，生产环境非常常见，但“过于主流”也会带来链分裂风险。

\- 特点：提供多平台二进制（含 ARM）、支持交叉编译，有“便携版”构建；发布二进制由 GPG key `15E66D9...674B0` 签名。

### Lodestar（TypeScript, ChainSafe）

\- 定位：TypeScript 共识客户端，适合快速原型和与浏览器生态集成，提供 beacon node + validator client + 一套协议库（BLS、SSZ 等）。

\- 许可：Apache 2.0 与 LGPL 双重授权，可根据项目选择宽松或弱 copyleft 模型。

### Prysm（Go, Prysmatic Labs）

\- 定位：Go 实现的共识客户端，强调易用性与可靠性，包含完整 beacon node 与 validator client。

\- 技术栈：libp2p 网络、BoltDB 存储、gRPC 进程间通信。

### Nimbus（Nim, Status）

\- 定位：极度资源友好，适合手机、树莓派等轻量设备，也适合在强服务器上保留更多资源给其他任务。

\- 特性：集成 validator client、支持远程签名、带性能分析与监控工具。

### Teku（Java, ConsenSys）

\- 定位：Java 实现，偏企业级场景，强调可运维性与可观测性。

\- 提供完整 beacon node、validator client、REST 管理 API、Prometheus 指标与外部密钥管理集成。

### Grandine（Rust, Grandine Developers）

\- 定位：高性能、轻量级 Rust 共识客户端，强调并行化和高端硬件上的吞吐与低延迟。

\- 架构上与 Lighthouse 同为 Rust 生态，复用部分库，但目标是提供更“精简、高速”的替代方案。

Caplin（Go, Erigon）

\- 定位：Erigon 内嵌的共识客户端，让 Erigon 可以在无外部 CL 的情况下独立运行。

\- 本质是 Erigon 的一个“额外特性”，直接在单一进程内整合 EL + CL。

### LambdaClass（Elixir）

\- 定位：4 期开始的 Elixir 共识客户端，从实验逐渐发展为完整实现，仍在积极开发中，但尚未用于生产网络。
<!-- DAILY_CHECKIN_2026-04-16_END -->

# 2026-04-15
<!-- DAILY_CHECKIN_2026-04-15_START -->

# 共识层网络

## 总览

共识层客户端的网络栈基于 **libp2p**：用它做 P2P 传输和多路复用；用 **libp2p-noise** 做加密与身份认证；用 **discv5** 做节点发现；用 **SSZ** 做消息编码；可选用 **Snappy** 做压缩。

\---

## 1\. libp2p 协议栈

libp2p 是最底层的 P2P 网络框架，支持多种传输（TCP、QUIC、WebRTC、WebSockets 等），Ethereum 共识层是其一个重要用户。

### 1.1 libp2p 协议分层

文中给出一个“libp2p Protocol Stack”表，可以按自下而上记：

| 层级 | 协议示例 | 作用 |

| --- | --- | --- |

| Discovery Layer | mdns, kademlia, rendezvous, identify | 发现节点、获取对方地址和能力 |

| NAT/Relay Layer | relay, dcutr, autonat, pnet | 穿透 NAT、防火墙，或在私网中通信 |

| Transport Layer | tcp, websockets, quic, webrtc, webtransport | 真实数据传输通道 |

| Security Layer | noise, tls, secio(已废弃) | 加密与认证 |

| Multiplexing Layer | yamux, mplex | 单连接上多路复用多条逻辑流 |

| Application Layer | pubsub/gossipsub, ping, 自定义协议 | 具体应用逻辑（区块 gossip、请求响应等） |

要点：

\- 最小连接至少要有：transport + security + multiplexing + application protocol。

\- relay/dcutr 在 NAT 阻碍直连时使用，用中继和打洞实现连接。

### 1.2 Protocol IDs 与 Identify

\- 每个应用协议用一个字符串 ID 标识`/app/protocol/version`，如 `/ipfs/ping/1.0.0`。

\- `Identify` 协议 `/ipfs/id/1.0.0`) 让节点在连接后互相告知：

\- protocolVersion、agentVersion（客户端 UA）

\- publicKey

\- listenAddrs（对方监听的 multiaddrs）

\- observedAddr（自己在对方眼中的地址，用于 NAT 诊断）

\- 支持的应用协议列表

\- signedPeerRecord（对 listenAddrs 的签名记录）

### 1.3 Multiaddress 与 Peer IDs

\- **Multiaddr**：统一封装多层地址，如：位置 + 传输 + peer id，便于将不同协议叠加成一个路径，例如 `/ip4/1.2.3.4/tcp/9000/p2p/<peer-id>`。

\- **Peer Id**：libp2p 中节点的唯一标识，是基于公钥的 multihash，编码为 protobuf；支持多种公钥类型（RSA、Ed25519、secp256k1、ECDSA）。

\- 文本表示常见有两种：

\- base58btc（以 `QM` 或 `1` 开头）。

\- multibase 编码的 CID（libp2p 正逐步迁移到这种形式）。

\---

## 2\. 建立一个 libp2p 连接的流程

libp2p 官方“connections spec”给出了一套通用流程，页面用一个 flowchart 概括：

1\. **Discovery：发现 peers**

\- `mdns`：局域网广播查询，零配置发现同网段节点。

\- `rendezvous`：大家在某个 rendezvous 节点注册，其他节点来这里查询 peer 列表。

\- `kademlia` DHT：全局发现，通过查 DHT 获取 peer 的最新 multiaddrs。

\- `identify`：连接建立后互相告诉地址与能力。

结果：我们得到一组 `(peer-id, multiaddr)`。

2\. **Transport：建立物理连接**

\- `TCP`：基础方案，但可能被 NAT / 防火墙阻断。

\- `WebSockets`：HTTP 封装的 TCP，更容易穿透部分网络限制。

\- `QUIC`：基于 UDP，连接建立快，自带多路复用和加密（TLS 1.3）。

\- `WebRTC` / `WebTransport`：更适合浏览器等场景。

如果 direct 连接失败（被 NAT 阻挡）：

\- `relay`：通过第三方中继 peer 转发，类似 TURN。

\- `dcutr`：通过已有 relay 连接尝试打洞，升级为点对点直连。

3\. **Encryption：加密与认证**

\- `noise`：目前默认首选，使用 Noise XX handshake 做双向认证和密钥交换。

\- `tls`：基于 TLS 的安全信道。

\- `secio`：旧方案，已弃用。

4\. **Multiplexing：多路复用**

\- `yamux`：简单且性能好，许多实现默认。

\- `mplex`：较早的轻量方案。

在一个 TCP/QUIC 连接上开多条 stream，分别跑不同协议或独立会话。

5\. **Application：应用协议**

\- `ping`：测延迟、保活。

\- `pubsub` / `gossipsub` / `episub`：发布订阅式广播（用于区块 / attestation gossip）。

\- 自定义协议：客户端可自定义 CL 相关的 Req/Resp、状态同步等。

\---

## 3\. GossipSub 优化的直觉

页面用两个思路对比解释为什么需要 GossipSub：

\- **方案 1：全连接网状**

\- 每个节点和所有其他节点直接相连（O(n²) 边），广播一个消息会收到 n−1 份重复拷贝，区块数据时浪费巨大带宽。

\- **方案 2：Pub/Sub + 稀疏 mesh（GossipSub）**

\- 所有节点订阅同一 topic（如 `/eth2/beacon_block/ssz_snappy`），只在 topic 内构造一个“小群体”网状（每个节点只连少数 peers）。

\- 消息在这个 mesh 内传递，避免重复广播给所有人。

\- 仍通过 gossip 机制确保最终全网传播。

效果：在保证鲁棒性的同时，大幅减少重复消息和带宽浪费，更适合区块链这种高频广播场景。

\---

## 4\. CL 网络使用的其他组件

### 4.1 libp2p-noise（加密）

\- 使用 Noise framework（而非单一固定协议），共识客户端采用 `XX` pattern：

\- initiator 和 responder 都在初期阶段发送自己的公钥，完成双向认证。

\- 协议提供机密性、完整性、身份认证和前向保密。

### 4.2 ENR（Ethereum Node Records）

\- ENR（EIP‑778）是 Ethereum 推荐的节点地址格式，结构化记录节点的身份与连接信息：

\- **Signature**：使用某种 identity scheme（如 secp256k1）签名，保证记录来自对应节点。

\- **Sequence Number (seq)**：64 位递增整数，用来比较“哪个记录更新”。

\- **Key/Value Pairs**：如 `ip`, `tcp`, `udp`, 公钥、额外元数据等。

\- 文本形式：RLP 编码再 base64，前缀 `enr:`，例如：

`enr:-Jq4QOXd31zNJBTB...`。

\- 在 Ethereum 中，ENR 是首选的网络地址表示方式，各客户端会用 discv5 交换和更新 ENR。

### 4.3 discv5（发现协议）

\- Discovery v5 是专门用于“peer 发现”的 UDP 协议，与 libp2p 传输栈并行运行。

\- 职责：

\- 维护 Kademlia DHT。

\- 使用 ENR 作为记录格式，提供最新节点信息。

\- 定期刷新 / 更新 ENR，保证节点地址变化（IP、端口）时仍能被找到。4

### 4.4 SSZ（编码）

\- SSZ（Simple Serialize）取代执行层的 RLP，在共识层用于所有结构化数据（除发现协议外）。

\- 特性：

\- 非自描述（需预先 schema）。

\- 兼容高效 Merkle 化（每个对象可直接 Merkleize），便于轻客户端和 DAS 场景使用。

### 4.5 Snappy（压缩）

\- Snappy 是 Google 2011 年创建的压缩算法，侧重速度而非最佳压缩比。

\- 共识层在 gossip 大型 SSZ 对象时常配合 Snappy 减少带宽占用，尤其是块和状态更新。

\---

## 5\. PeerDAS 与相关 R&D

\- EIP‑7594：Peer Data Availability Sampling（PeerDAS），为 beacon nodes 提供 DAS 协议，使它们无需下载全部 blob 数据就能抽样验证“数据已被发布”。

\---

# 共识层规范

\- 以太坊最初以 PoW 启动，但从一开始就计划在网络稳定后切换到 PoS。Gasper 论文提出了将 Casper（finality）与 GHOST（fork-choice）结合的共识机制设计。

\- 在 Gasper 设计基础上，社区用 Python 写出了共识层规范 **Pyspec**，它是一份“可执行规范”，既是客户端实现的参考，也是生成测试向量的权威来源。

\---

## Pyspec 的角色

\- 作为 **参考实现**：共识客户端（Prysm、Lighthouse、Teku、Nimbus、Lodestar 等）在实现时都对照 Pyspec，以确保行为与规范一致。

\- 作为 **测试向量生成器**：通过运行 Pyspec，可以生成各类状态转移、边界条件和攻击场景的测试向量，供不同客户端共享，用来检验兼容性。

\- 作为 **研究与教学工具**：研究者可以在 Pyspec 基础上做实验性修改（例如新共识规则、奖励曲线），快速验证设计效果。

\---

# 共识链**规范与 API 接口**

## 1\. 概览

\- Beacon Chain 是以太坊 PoS 的“系统链”，负责维护验证者注册表、处理质押/退出、追踪最终性与同步委员会等共识状态。

\- Beacon API 是共识客户端（beacon node）提供的 REST 接口，供外部读取共识信息，也供 validator 客户端使用（获取职责、提交签名等）。

\---

## 2\. BeaconState：共识层的“全局状态”

`BeaconState` 是一个 SSZ Container，表示某个 slot 的完整共识状态快照，主要字段可以分为几类：

### \- 元信息与历史

\- `genesis_timegenesis_validators_rootslot`：创世时间、初始验证者根、当前 slot。

\- `fork`：当前在用的 fork 版本（phase0/Altair/Bellatrix/Capella/Electra 等）。

\- `latest_block_headerblock_rootsstate_rootshistorical_roots`：用于追踪最近和深层历史的块根与状态根，并支持历史证明。

### \- Eth1 相关

\- `eth1_dataeth1_data_voteseth1_deposit_index`：跟踪存款合约相关数据和投票，用于从执行层接入新的质押。

### \- 验证者注册表与余额

\- `validators: List[Validator]`：所有验证者条目（状态、有效余额、公钥、提款地址等）。

\- `balances: List[Gwei]`：每个验证者的 Gwei 余额。

### \- 随机性与惩罚相关

\- `randao_mixes`：历史 RANDAO 值，用于生成随机数、分配 committees 等。

\- `slashings`：每个 epoch 的 slashed 有效余额总和，用于计算 slashing 惩罚系数。

### \- 出勤与最终性

\- `previous_epoch_participationcurrent_epoch_participation`：每个验证者在前一 / 当前 epoch 的参与标志（head/target/source 是否正确投票）。

\- `justification_bits`：最近若干 epoch 是否被 justified 的 bit 位。

\- `previous_justified_checkpointcurrent_justified_checkpointfinalized_checkpoint`：最终性相关 checkpoint 状态。

### \- Inactivity 与 Sync

\- `inactivity_scores`：每个验证者的 inactivity 分数，用于 inactivity leak。

\- `current_sync_committeenext_sync_committee`：当前与下一轮同步委员会，用于轻客户端与 sync committee 签名。

### \- 执行层桥接与提款（Capella 及之后）

\- `latest_execution_payload_header`：最新执行载荷头，桥接到执行层。

\- `next_withdrawal_indexnext_withdrawal_validator_index`：自动提款进度。

### \- 深层历史与 Electra 新增字段

\- `historical_summaries`：自 Capella 起用于压缩历史的摘要列表。

\- `deposit_requests_start_indexdeposit_balance_to_consumeexit_balance_to_consumeearliest_exit_epochconsolidation_balance_to_consumeearliest_consolidation_epoch`：Electra 之后有关存款、退出、合并（consolidation）的排队与余额消费控制。

\- `pending_depositspending_partial_withdrawalspending_consolidations`：挂起的存款、部分提款和合并请求队列。

\---

## 3\. BeaconBlockBody：一个 Beacon 区块里有什么

`BeaconBlockBody` 是 Beacon block 的“payload”，Electra/Deneb 之后的字段包括：

### \- 基础字段

\- `randao_reveal`：proposer 提供的 BLS 签名，用于 RANDAO 随机性混合。

\- `eth1_data`：对 Eth1 / 执行链数据的投票。

\- `graffiti`：32 字节任意数据，常用作自定义标记。

### \- 操作集合（operations）

\- `proposer_slashings: List[ProposerSlashing]`：提议者被 slash 的证明。

\- `attester_slashings: List[AttesterSlashing]`：attester 被 slash 的证明，Electra 提高上限。

\- `attestations: List[Attestation]`：attester 对 head/target/source 的投票。

\- `deposits: List[Deposit]`：新的质押存款。

\- `voluntary_exits: List[SignedVoluntaryExit]`：自愿退出请求。

\- `sync_aggregate: SyncAggregate`：同步委员会聚合签名，用于轻客户端安全。

### \- 执行层与 blobs

\- `execution_payload: ExecutionPayload`：内嵌执行层 payload（交易、收据根等）。

\- `bls_to_execution_changes: List[SignedBLSToExecutionChange]`：从 BLS 密钥到执行层提款地址的绑定变更。

\- `blob_kzg_commitments: List[KZGCommitment]`：与 EIP‑4844 blobs 对应的 KZG 承诺列表。

### \- Execution requests（Electra 之后）

\- `execution_requests: ExecutionRequests`：从共识层向执行层发出的各种请求打包（如提款、存款、合并请求等），取代 Capella 时版本中散落的单项指令。

\---

## 4\. Beacon API 的定位

基于上下文可总结出 Beacon API 的主要用途：

### \- 共识数据查询：

\- 获取 BeaconState / BeaconBlock（按根或 slot）。

\- 查询 validators 状态、余额、slashing、finality 等。

\- 查询 sync committee、fork choice、头部信息等。

### \- Validator 客户端接口：

\- 获取 proposer 与 attester 职责（duties）。

\- 获取待签名 block / attestation 模板。

\- 提交已签名的 blocks / attestations / slashings / exits。
<!-- DAILY_CHECKIN_2026-04-15_END -->

# 2026-04-14
<!-- DAILY_CHECKIN_2026-04-14_START -->


# 共识层概述

## 总览：目标与挑战

-   共识层的目标：在“**不可靠基础设施**”（消费者级硬件、不稳定网络、恶意节点）上，构建“**可靠的分布式系统**”，让全球成千上万节点维护一份完全一致的账本。
    
-   Ethereum 今天的共识由两套协议组合而成：LMD GHOST（fork choice）+ Casper FFG（finality），合称 Gasper。
    

* * *

## 1\. PoW / PoS、区块与区块链

-   PoW 和 PoS 本身不是共识协议，而是 **Sybil 抵抗机制**：通过算力或质押成本限制“刷号”，为共识协议提供权重（PoW=累计工作量，PoS=累计质押）。
    
-   区块是由领导者（proposer）打包的一批交易（或投票）：
    
    -   执行链：payload 是用户交易。
        
    -   Merge 前 Beacon 链：payload 主要是验证者的 attestations。
        
    -   Merge 后 Beacon 链：既包含 execution payload，又包含共识层信息；Deneb/4844 之后还包含 blob commitments。
        
-   除创世区块外，所有块都指向父块形成链；共识协议的任务是让所有节点最终就同一条“规范链”达成一致。
    

* * *

## 2\. 向 PoS 的迁移：The Merge

-   2022‑09‑15，巴黎硬分叉（The Merge）发生，Ethereum 共识从 PoW 切换到 PoS，由 Beacon 链接管共识。
    
-   激活条件是 **终端总难度 TTD** 而非区块高度：
    
    -   TTD 达到预设阈值的最后一个 PoW 区块成为 terminal block，确保过渡不被恶意分叉干扰。
        
    -   这套机制同样用于 testnet / devnet 等网络的 Merge 激活。
        
-   Merge 之后：
    
    -   Beacon 链成为新的“引擎”，负责 slot/epoch、验证者、attestations 和 finality。
        
    -   能耗大幅下降，安全性与去中心化通过 PoS 经济激励与惩罚机制来维护。
        

* * *

## 3\. Beacon 链与验证者

## 3.1 骨干角色：Beacon Chain

-   Beacon 链是 PoS 共识的骨干：
    
    -   维护验证者注册表及其状态。
        
    -   安排 proposer、委员会，收集与聚合 attestations。
        
    -   负责 epochs、checkpoints 和 finality。
        

## 3.2 验证者与激活条件

-   验证者：参与提议区块和投票（attest）的主体，需要质押 ETH 作为抵押。
    
-   条件与要点：
    
    -   最低 32 ETH 才能激活一个 validator，单个 validator 最大有效余额 2048 ETH。
        
    -   质押 ETH 是违约保证金，恶意或严重失职会被 slash。
        
    -   validator 客户端可以管理多个验证者，依赖 beacon 节点读取 Beacon 链状态。
        

## 3.3 随机性与委员会

-   验证者分配依赖 **RANDAO + VDF** 提供的加密随机性，防止操纵 proposer/committee 分配。
    
-   机制：
    
    -   每个 slot 分配一个 proposer 和若干 committees（每个至少 128 验证者，理想情况下 4096+ 验证者网络）。
        
    -   committee 内的 attestations 使用 BLS 聚合签名压缩，降低链上负载。
        

* * *

## 4\. 时间结构：Slot / Epoch / Attestations / Finality

## 4.1 Slots 与 Epochs

-   slot：12 秒，每个 slot 最多一个 Beacon block（可为空）。
    
-   epoch：32 个 slot ≈ 6.4 分钟，slot 0 起始，slot 0 属于 epoch 0。
    
-   每个 slot：
    
    -   一名 proposer 有机会提块。
        
    -   对应的 committees 对该 slot 的链头进行 attestation。
        

## 4.2 Attestations：LMD GHOST + FFG vote

-   每个 validator 每个 epoch 提交 1 个 attestation，包含：
    
    -   LMD GHOST vote：对“当前 Beacon 链 head”的投票。
        
    -   FFG vote：对“source checkpoint → target checkpoint”的投票。
        
-   LMD GHOST：通过“每个验证者最新投票”选出加权最重子树，决定链头。
    
-   Casper FFG：通过对 checkpoints 的投票实现 justification & finality。
    

## 4.3 Checkpoints 与 Finality

-   每个 epoch 选一个 checkpoint：
    
    -   如果 epoch 第一 slot 有块，则该块为 checkpoint。
        
    -   否则取最近的前一个块作为该 epoch 的 checkpoint。
        
-   定义：
    
    -   justified：该 checkpoint 收到 ≥ 2/3 有效 stake 的 FFG vote。
        
    -   finalized：一个 checkpoint 被 justified 后，再有一个后续 epoch 的 checkpoint 也被 justified，前者即被 finalized，连带其之前的所有块也变为最终不可逆。
        
-   典型时间尺度：
    
    -   交易在 epoch 中部被打包：大约半个 epoch 到下一个 checkpoint，再加一个 epoch 完成 finality，约 2.5 个 epoch ≈ 14–16 分钟。
        

* * *

## 5\. Blob 与数据可用性（EIP‑4844）

-   Deneb/Cancun 升级引入 **EIP‑4844（proto‑danksharding）**，为 Ethereum 提供“临时数据可用性层”：
    
    -   新增 **blobs**，每个 block 可携带 3–6 个 blob sidecars。
        
    -   L2 可将批量交易数据放入 blobs 中，降低 L1 gas 成本，提高吞吐。
        
-   采用 **KZG commitments** 来承诺和验证 blobs 内容，需要一次 KZG trusted setup（KZG Ceremony）。
    
-   存储影响：
    
    -   粗略估算：`≈ 52–104 GB` 用于 4096 epochs（默认保留周期）内 blobs 的存储；到期后客户端可删旧 blobs。
        

* * *

## 6\. 激励与惩罚：Rewards / Penalties / Slashings

## 6.1 奖励：Attester / Proposer / Whistleblower

-   Attester rewards：
    
    -   对正确 head 和 checkpoints 进行及时 attestation 会获得奖励，越早上链奖励越高。
        
-   Proposer rewards：
    
    -   提出的 block 被纳入最终链并贡献 finality，可获得显著奖励（约整体收益的 1/8）。
        
-   Whistleblower rewards：
    
    -   举报 slashable offense 的证明会获得奖励，目前奖励发给 proposer。
        

## 6.2 惩罚与 Inactivity Leak

-   Attester penalties：
    
    -   不按时 attestation 或投给错误头 / 未被 final 的 checkpoint，会受到罚分，防止长期懒惰或偏离共识。
        
-   Slashings（严重违规）：
    
    -   Double proposal、LMD GHOST double vote、FFG surround vote、FFG double vote 都可被 slashing。
        
    -   至少罚没 1/32 余额并强制退出，罚没规模与同时间被 slash 的人数相关，可高达全部质押。
        
-   Inactivity leak：
    
    -   长时间无法 final（>4 epochs）时启动 inactivity leak：
        
        -   Attester rewards 变为 0。
            
        -   对未能参与的验证者持续施加递增罚分，直至剩余活跃验证者重新形成 ≥ 2/3 多数并恢复 finality。
            

## 6.3 Validator 生命周期

-   激活：余额 ≥ 32 ETH 后排队进入 active set。
    
-   自愿退出：服务 ≥ 2048 epochs 后可申请退出，退出后 4 epochs 冷却期才能取回余额。
    
-   强制退出：余额跌至 16 ETH 或被 slash。
    
-   提款延迟：
    
    -   正常退出：约 27 小时。
        
    -   被 slash：约 36 天（8192 epochs）。
        

* * *

# 共识层架构

从共识协议视角讲“客户端架构”，核心是：Ethereum PoS 把两套协议（LMD GHOST + Casper FFG）合在一起形成 Gasper，并通过 CL 客户端 + EL 客户端协同来达成共识。

重点包括：fork-choice 机制、reorg、safety & liveness 取舍、CL/EL 之间的控制流，以及 Beacon 链状态转移函数。

* * *

## 1\. Fork-choice 机制与 Reorg

## Block tree 与 fork choice rules

-   网络延迟、恶意节点等会让不同节点看到不同的 block tree，需要一个 fork-choice rule 来选“头部区块”，从而选出一条线性规范链。
    
-   典型规则：
    
    -   PoW（比特币、旧以太坊）：heaviest chain rule（累计工作量最大的链）。以太坊 PoW 实际从未实现白皮书中的 GHOST 变体，而是使用简单的 heaviest chain。
        
    -   Casper FFG（PoS）：选择包含“最高高度已 justified checkpoint”的链，且永不回滚 finalized block。
        
    -   LMD GHOST（PoS）：Greediest Heaviest Observed SubTree，基于验证者最新 votes 对各子树加权计分，同时也遵守 Casper FFG 的“不可回滚 finalized block”的约束。
        

## Reorgs 与 Reversion

-   当新块到来改变 fork-choice 结果时，节点可能需要 reorg：
    
    -   回滚旧 head 分支上的若干区块，从回滚点恢复状态。
        
    -   改为沿新 head 分支重放区块。
        
-   一两块的短 reorg 很常见，通常源自网络延迟；长 reorg 应当极少出现，否则可能意味着攻击或实现 bug。
    

* * *

## 2\. Safety vs Liveness：以太坊的取舍

-   **Safety**：不发生坏事，比如不会 finalize 两条互相冲突的历史，所有诚实节点保持账本一致。
    
-   **Liveness**：好事最终会发生，链不会停摆，总能持续产生新块、推进状态。
    
-   CAP 定理 + 不可靠网络 ⇒ 无法在所有情形下同时保证完美 safety 和完美 liveness。
    
-   以太坊 PoS 的策略：
    
    -   正常网络条件下尽量兼顾 safety 与 liveness。
        
    -   在严重网络分区时偏向保持 liveness：各分区内链仍会出块，但无法 finality；若分区持续，很可能最终在不同侧 finalize 冲突历史，出现“不可合并的两条链”。
        

* * *

## 3\. Gasper：LMD GHOST + Casper FFG

-   LMD GHOST 专注 liveness：
    
    -   根据验证者“最新消息”（latest message driven）对 block tree 加权打分，持续选择最重子树作为 head，从而保持链向前推进。
        
-   Casper FFG 专注 safety：
    
    -   通过对 checkpoint 的投票来 justify / finalize 区块，防止长距离回滚，为链尾提供“硬边界”。
        
-   Gasper = LMD GHOST（驱动 head）+ Casper FFG（周期性 finality）：
    
    -   即使由于网络问题一段时间无法 final，LMD GHOST 仍会让链继续前进；
        
    -   一旦条件恢复，Casper FFG 会重新开始 finalizing，从而恢复安全边界。
        

* * *

## 4\. CL / EL 客户端架构与控制流

## 4.1 组件

-   **Beacon Node（共识客户端）**：
    
    -   运行 PoS 共识，处理 beacon blocks、attestations，与其它 beacon nodes 通过独立的 CL p2p 网络通信。
        
    -   本地与执行客户端通过 Engine API 连接，可选地还连接本地 validator 客户端。
        
-   **Validator Client**：
    
    -   管理 32 ETH 质押、生成提议块和 attestations，只与本地 beacon node 对话，由后者负责广播到网络。
        
-   每个以太坊节点都包含 EL + CL：
    
    -   执行客户端（Geth、Nethermind、Besu、Erigon 等）。
        
    -   共识客户端（Prysm、Teku、Lighthouse、Nimbus、Lodestar 等）。
        
    -   两者通过本地 RPC（Engine API）互联，并共享同一个 ENR（但有 eth1 / eth2 不同密钥）。
        

## 4.2 控制流（非出块节点）

当本节点不是出块者时：

1.  共识客户端通过 block gossip 收到新的 beacon block。
    
2.  本地预验证 block（签名、结构等）。
    
3.  提取其中 execution payload，发给执行客户端。
    
4.  执行客户端执行其中交易，检查状态转移是否合法。
    
5.  执行客户端将验证结果（成功或错误）返回给共识客户端。
    
6.  若有效，共识客户端把 block 加入本地链，并对其做 attestation，然后把 attestation 广播出去。
    

## 4.3 控制流（本节点是出块者）

当本节点被选为 proposer 时：

1.  共识客户端收到“你是下一个 block producer”的通知。
    
2.  通过 Engine API 调用执行客户端的“create block / build payload”接口。
    
3.  执行客户端从 mempool 选择交易。
    
4.  执行客户端执行交易、构造 execution payload 和 block hash。
    
5.  共识客户端把 payload + block hash 封装进 beacon block。
    
6.  共识客户端将 beacon block 广播到 CL 网络。
    
7.  其它节点验证该 block，并对其出 attestations。
    
8.  当足够验证者 attestation 后，该 block 成为链头，随后通过 Casper FFG 被 justify / finalize。
    

* * *

## 5\. Beacon 链状态转移函数

-   每个节点维护一个 beacon state，按顺序应用区块来更新状态：
    
    -   把 state transition 看成纯函数 `S' = f(S, B)`，相同输入必然给出相同输出，保证所有诚实节点状态一致。
        

## Slot-driven 结构

Beacon 链是 **slot 驱动** 而非 block 驱动：即使某个 slot 没有 block，也要执行 per-slot 逻辑。

状态转移拆成三类函数：

1.  **Per-slot transition**：`S' = f_s(S)`
    
    -   每个 slot 必执行一次，更新 slot 号、计时器、RANDAO 混合等。
        
2.  **Per-block transition**：`S' = f_b(S, B)`
    
    -   只在该 slot 有 block 时执行，处理 block header、operations（attestations、slashings、deposits 等）。
        
3.  **Per-epoch transition**：`S' = f_e(S)`
    
    -   每到 epoch 开始 slot，会处理奖励 / 惩罚、crosslink / checkpoint 更新等。
        

## 代码片段要点

伪代码的核心逻辑：

```python
def state_transition(state, signed_block, validate_result=True):
    block = signed_block.message
    process_slots(state, block.slot)        # 处理空 slot + epoch 逻辑
    if validate_result:
        assert verify_block_signature(state, signed_block)
    process_block(state, block)             # 应用 block 层规则
    if validate_result:
        assert block.state_root == hash_tree_root(state)
```

其中 `process_slots` 会循环调用 `process_slot`，并在 epoch 边界调用 `process_epoch`。
<!-- DAILY_CHECKIN_2026-04-14_END -->

# 2026-04-13
<!-- DAILY_CHECKIN_2026-04-13_START -->



# RLP  
网址：[epf](https://epf.wiki/#/wiki/EL/RLP)

## 1\. RLP 是什么

-   全称：Recursive Length Prefix，递归长度前缀编码。
    
-   作用：把“嵌套的字节数组”（bytes 和 list of bytes）编码成一个字节序列，用于以太坊执行层（交易、区块、状态数据等）。
    
-   特点：
    
    -   极简格式，只关心结构（字符串 / 列表、长度），不关心具体类型（整数、布尔等）。
        
    -   上层协议自己约定“这些 bytes 表示什么类型”。
        
    -   目标是跨客户端保持字节级一致性。
        

## 2\. RLP 使用场景与对比

-   执行层（EL）：使用 RLP 序列化交易、区块头等数据结构。
    
-   共识层（CL）：使用 SSZ（Simple Serialize），而不是 RLP。
    
-   为什么不用 JSON / XML / Protobuf 等：
    
    -   JSON/XML 太重，有语法和类型；
        
    -   Protobuf/Cap’n Proto 需要事先 schema；
        
    -   RLP 只做“通用字节数组 + 嵌套结构”编码，简单稳定，适合共识。
        

## 3\. 编码规则总览

RLP 对两类对象编码：字符串（byte 序列）和列表（字符串或列表的有序集合）。

1.  单字节（0x00–0x7F）：
    
    -   规则：值本身就是编码（不加前缀）。
        
2.  短字符串（长度 0–55 字节，且不是单字节 0x00–0x7F）：
    
    -   编码：一个前缀字节（0x80 + 长度） + 原始字节序列。
        
3.  长字符串（长度 ≥ 56 字节）：
    
    -   先把“长度”转为 big-endian 字节序列 L；
        
    -   前缀：0xB7 + L 的长度；
        
    -   编码：前缀 + L + 原始字节序列。
        
4.  短列表（总 payload 长度 0–55 字节）：
    
    -   先对列表中每个元素做 RLP 编码，拼接得到 payload；
        
    -   前缀：0xC0 + payload 长度；
        
    -   编码：前缀 + payload。
        
5.  长列表（总 payload 长度 ≥ 56 字节）：
    
    -   先得到 payload（同上）；
        
    -   把 payload 长度转为 big-endian 字节序列 L；
        
    -   前缀：0xF7 + L 的长度；
        
    -   编码：前缀 + L + payload。
        
6.  空值与空列表：
    
    -   空字符串：编码为 0x80。
        
    -   空列表：编码为 0xC0。
        

## 4\. 前缀区间速记

-   0x00–0x7F：单字节自身。
    
-   0x80–0xB7：短字符串（前缀=0x80+len）。
    
-   0xB8–0xBF：长字符串（前缀=0xB7+len(len)，后跟长度，再跟数据）。
    
-   0xC0–0xF7：短列表（前缀=0xC0+len(payload)）。
    
-   0xF8–0xFF：长列表（前缀=0xF7+len(len)，后跟长度，再跟 payload）。
    

## 5\. 解码思路（自顶向下）

给定一个字节序列，从第一个字节（前缀）开始分类：

-   0x00–0x7F：这是一个单字节字符串，长度 1，内容就是该字节。
    
-   0x80–0xB7：短字符串，长度 = 前缀 - 0x80，读出后面对应长度的字节。
    
-   0xB8–0xBF：长字符串，先读长度字段（长度 = 前缀 - 0xB7），再读该长度字段表示的真实长度，然后再读数据。
    
-   0xC0–0xF7：短列表，payload 长度 = 前缀 - 0xC0；对 payload 区间递归解码出多项元素。
    
-   0xF8–0xFF：长列表，和长字符串类似，先读长度字段，再读 payload，再对 payload 递归解码。
    

## 6\. 示例（概念级）

文中用字符串 "dog"、列表 \["cat", "dog"\] 等例子演示：

-   先把字符串转为 ASCII bytes，再按“短字符串”规则加前缀。
    
-   列表则是把每个元素单独 RLP 编码后拼接，再整体按“列表”加前缀。
    

## 7\. 实现与工具

页面列出多种语言的 RLP 实现和资料：

-   Go：Geth 中的 RLP 包。
    
-   JavaScript / TypeScript：以太坊相关 JS 库中的 RLP 工具。
    
-   Python、Rust、C#（Nethermind）等多种实现。
    
-   参考资料：
    
    -   以太坊黄皮书中 RLP 章节。
        
    -   官方文档与若干教程、博客文章，对编码规则和示例做进一步说明。
        

# EOF

网址：[epf](https://epf.wiki/#/wiki/EL/RLP)

## 1\. RLP 是什么

-   全称：Recursive Length Prefix，递归长度前缀编码。
    
-   作用：把“嵌套的字节数组”（bytes 和 list of bytes）编码成一个字节序列，用于以太坊执行层（交易、区块、状态数据等）。
    
-   特点：
    
    -   极简格式，只关心结构（字符串 / 列表、长度），不关心具体类型（整数、布尔等）。
        
    -   上层协议自己约定“这些 bytes 表示什么类型”。
        
    -   目标是跨客户端保持字节级一致性。
        

## 2\. RLP 使用场景与对比

-   执行层（EL）：使用 RLP 序列化交易、区块头等数据结构。
    
-   共识层（CL）：使用 SSZ（Simple Serialize），而不是 RLP。
    
-   为什么不用 JSON / XML / Protobuf 等：
    
    -   JSON/XML 太重，有语法和类型；
        
    -   Protobuf/Cap’n Proto 需要事先 schema；
        
    -   RLP 只做“通用字节数组 + 嵌套结构”编码，简单稳定，适合共识。
        

## 3\. 编码规则总览

RLP 对两类对象编码：字符串（byte 序列）和列表（字符串或列表的有序集合）。

1.  单字节（0x00–0x7F）：
    
    -   规则：值本身就是编码（不加前缀）。
        
2.  短字符串（长度 0–55 字节，且不是单字节 0x00–0x7F）：
    
    -   编码：一个前缀字节（0x80 + 长度） + 原始字节序列。
        
3.  长字符串（长度 ≥ 56 字节）：
    
    -   先把“长度”转为 big-endian 字节序列 L；
        
    -   前缀：0xB7 + L 的长度；
        
    -   编码：前缀 + L + 原始字节序列。
        
4.  短列表（总 payload 长度 0–55 字节）：
    
    -   先对列表中每个元素做 RLP 编码，拼接得到 payload；
        
    -   前缀：0xC0 + payload 长度；
        
    -   编码：前缀 + payload。
        
5.  长列表（总 payload 长度 ≥ 56 字节）：
    
    -   先得到 payload（同上）；
        
    -   把 payload 长度转为 big-endian 字节序列 L；
        
    -   前缀：0xF7 + L 的长度；
        
    -   编码：前缀 + L + payload。
        
6.  空值与空列表：
    
    -   空字符串：编码为 0x80。
        
    -   空列表：编码为 0xC0。
        

## 4\. 前缀区间速记

-   0x00–0x7F：单字节自身。
    
-   0x80–0xB7：短字符串（前缀=0x80+len）。
    
-   0xB8–0xBF：长字符串（前缀=0xB7+len(len)，后跟长度，再跟数据）。
    
-   0xC0–0xF7：短列表（前缀=0xC0+len(payload)）。
    
-   0xF8–0xFF：长列表（前缀=0xF7+len(len)，后跟长度，再跟 payload）。
    

## 5\. 解码思路（自顶向下）

给定一个字节序列，从第一个字节（前缀）开始分类：

-   0x00–0x7F：这是一个单字节字符串，长度 1，内容就是该字节。
    
-   0x80–0xB7：短字符串，长度 = 前缀 - 0x80，读出后面对应长度的字节。
    
-   0xB8–0xBF：长字符串，先读长度字段（长度 = 前缀 - 0xB7），再读该长度字段表示的真实长度，然后再读数据。
    
-   0xC0–0xF7：短列表，payload 长度 = 前缀 - 0xC0；对 payload 区间递归解码出多项元素。
    
-   0xF8–0xFF：长列表，和长字符串类似，先读长度字段，再读 payload，再对 payload 递归解码。
    

## 6\. 示例（概念级）

文中用字符串 "dog"、列表 \["cat", "dog"\] 等例子演示：

-   先把字符串转为 ASCII bytes，再按“短字符串”规则加前缀。
    
-   列表则是把每个元素单独 RLP 编码后拼接，再整体按“列表”加前缀。
    

## 7\. 实现与工具

页面列出多种语言的 RLP 实现和资料：

-   Go：Geth 中的 RLP 包。
    
-   JavaScript / TypeScript：以太坊相关 JS 库中的 RLP 工具。
    
-   Python、Rust、C#（Nethermind）等多种实现。
    
-   参考资料：
    
    -   以太坊黄皮书中 RLP 章节。
        
    -   官方文档与若干教程、博客文章，对编码规则和示例做进一步说明。
<!-- DAILY_CHECKIN_2026-04-13_END -->

# 2026-04-12
<!-- DAILY_CHECKIN_2026-04-12_START -->




# JSON-RPC

## 1\. 概念与作用

\- JSON-RPC 是一种用 JSON 编码的远程过程调用协议，基于 OpenRPC 规范。 \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC))

\- 在以太坊里，它是 Execution API 的一部分，用来和执行层客户端交互，也是钱包、DApp 与节点交互的主要方式。 \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC))

\- 共识层（CL）与执行层（EL）之间的 Engine API 也是通过 JSON-RPC 接口通信的。 \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC))

## 2\. 基本请求结构

\- 标准请求格式`{ "id": 1, "jsonrpc": "2.0", "method": "...", "params": [...] }`。 \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC))

\- 字段含义：

\- **id**：请求的唯一标识符。

\- **jsonrpc**：协议版本（一般为 "2.0"）。

\- **method**：调用的方法名。

\- **params**：方法参数，可以为空数组，有些参数有默认值（如 block tag 为 "latest"）。 \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC))

## 3\. 命名空间与常见命名空间

\- 方法由 “命名空间 + 下划线 + 方法名” 组成，例如 `eth_getBalance`。 \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC))

\- 客户端必须实现规范要求的基础方法，同时可以扩展客户端自有命名空间；不同客户端（如 Geth、Reth）命名空间略有差异，需要参考各自文档。 \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC))

**常见命名空间**（是否敏感）： \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC))

| Namespace | 作用简记 | 是否敏感 |

| --- | --- | --- |

| eth | 与以太坊核心交互（余额、交易、区块） | 可能敏感（涉账号数据） \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC)) |

| web3 | 客户端工具与信息 | 否 \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC)) |

| net | 网络信息（网络 ID、peer 等） | 否 \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC)) |

| txpool | 交易池状态检查 | 否 \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC)) |

| debug | 深度调试、状态检查（Geth trace 等） | 一般视为敏感，常被限制 \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC)) |

| trace | 状态追踪（Parity 风格 trace） | 否 \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC)) |

| admin | 节点管理配置 | 是（可控制节点） \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC)) |

| rpc | RPC 服务与模块信息 | 否 \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC)) |

“敏感”通常指可以配置节点或访问本地账户数据的接口。 \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC))

### 常用方法示例

**eth 命名空间**（对钱包和 DApp 最重要）

\- `eth_blockNumber`：返回最新区块号。 \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC))

\- `eth_call`：在本地执行一笔消息调用，不上链，常用来读合约状态。 \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC))

\- `eth_chainId`：当前链的 chain id。 \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC))

\- `eth_estimateGas`：模拟执行以估算需要的 gas。 \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC))

\- `eth_gasPrice`：返回当前 gas price（wei）。 \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC))

\- `eth_getBalance(address, block)`：查询地址在指定区块的余额。 \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC))

\- `eth_getBlockByHash / eth_getBlockByNumber`：按 hash / 高度获取区块信息，可选是否返回完整交易对象。 \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC))

\- `eth_getBlockTransactionCountByHash / ...ByNumber`：获取区块中交易数量。 \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC))

\- `eth_getCode(address, block)`：获取某地址上的合约代码。 \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC))

\- `eth_getLogs(filter)`：按过滤器（地址、topics、区块范围）获取日志。 \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC))

\- `eth_getStorageAt(address, position, block)`：获取合约某个存储槽位的值。 \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC))

**debug 命名空间**（调试、研究用）

\- 通常用于区块浏览器、研究或问题排查，对性能消耗大，公共 RPC 多半限制。 \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC))

\- 典型方法：

\- `debug_getBadBlocks`：返回近期的坏块列表。 \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC))

\- `debug_getRawBlock(block_number)`：返回 RLP 编码的区块。 \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC))

\- `debug_getRawHeader(block_number)`：返回 RLP 编码的区块头。 \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC))

\- `debug_getRawReceipts(block_number)`：返回 EIP‑2718 编码的收据数组。 \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC))

\- `debug_getRawTransactions(tx_hash)`：返回 EIP‑2718 编码的交易数组。 \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC))

## 4\. Engine API（CL 与 EL 之间）

\- Engine API 不对普通用户开放，单独跑在受 JWT 认证保护的端口上，只允许共识客户端访问。 \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC))

\- 核心目的：在共识和执行客户端之间交换共识状态、分叉选择、区块验证等信息。 \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC))

\- 通信方式：HTTP 上的 JSON-RPC，使用 JWT 做身份认证（不加密流量）。 \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC))

\- 主要方法示例（有多版本的以支持升级）： \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC))

\- `engine_exchangeTransitionConfigurationV1`：交换 CL/EL 的配置。 \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC))

\- `engine_forkchoiceUpdatedV1*`：更新 forkchoice 状态并可触发 payload 构建。 \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC))

\- `engine_getPayloadBodiesByHashV1*`：按区块哈希数组获取 payload bodies。 \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC))

\- `engine_getPayloadV1*`：从执行层获取构建好的 execution payload。 \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC))

\- `debug_newPayloadV1*`：返回 payload 验证细节，用于调试。 \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC))

## 5\. 编码约定（十六进制）

\- 数值（Quantities）统一用带 "0x" 前缀的十六进制，不允许前导零。 \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC))

\- 例：十进制 65 表示为 `"0x41"`，0 表示为 `"0x0"`。 \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC))

\- 非法示例`"0x"`（没有数字）`"ff"`（缺少 0x 前缀）。 \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC))

\- 未格式化数据（哈希、地址、字节数组）同样使用 0x 十六进制，但不允许多余前导零。 \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC))

## 6\. 传输层：协议无关（Transport-agnostic）

\- JSON-RPC 与传输协议解耦，可通过 HTTP、WebSocket (WSS)、IPC 等方式发送。 \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC))

\- **HTTP**：单次请求‑响应，响应后连接关闭，适合简单查询。 \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC))

\- **WSS**：双向长连接，允许订阅/事件推送，适合实时事件（logs、newHeads 等）。 \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC))

\- **IPC**：本机进程间通信，比 HTTP/WSS 更快，但不能远程访问，常用于本地 JS 控制台。 \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC))

## 7\. 常用调用方式与工具

### 直接用 curl

\- 示例：获取最新区块号`eth_blockNumber`，无参数，默认使用 "latest"）。 \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC))

\- 请求是 HTTP POST，JSON-RPC 数据放在请求体里`Content-Type` 为 `application/json`。 \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC))

### 使用 axios（JS/TS）

\- 典型步骤：

\- 指定节点 URL 和查询的地址。

\- POST 到节点`jsonrpc: '2.0'method: 'eth_getBalance'params: [address, 'latest']id: 1`，并带 `Content-Type: application/json` 头。 \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC))

\- 本质上仍是 HTTP+JSON-RPC，只是通过库封装成更友好的接口。 \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC))

### 使用 web3 库（推荐日常开发）

\- 一般使用 web3py、web3.js 或 ethers.js，这些库封装了底层 JSON-RPC 调用。 \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC))

示例：web3py。 \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC))

\- 使用 HTTPProvider 连接`Web3.HTTPProvider('http://localhost:8545')`[。](http://localhost:8545'\)`。)

\- 获取余额`w3.eth.get_balance('0xaddress')`。

示例：ethers.js。 \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC))

\- 创建 provider`new ethers.providers.JsonRpcProvider('http://localhost:8545')`[。](http://localhost:8545'\)`。)

\- 获取区块号`await provider.getBlockNumber()`。 \[epf\]([https://epf.wiki/#/wiki/EL/JSON-RPC](https://epf.wiki/#/wiki/EL/JSON-RPC))

# DevP2P

## 1\. 整体概念与作用

\- DevP2P 是执行层（EL）使用的点对点网络协议栈，用来让节点发现彼此并在链上安全地交换数据。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

\- EL 网络里有两部分：发现层（Discovery，基于 UDP）负责找节点，传输层（Transport，RLPx，基于 TCP）负责建立加密连接并传消息。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

\- 执行客户端负责 gossip 交易，共识客户端负责 gossip 区块，各自有独立的 P2P 网络（EL：devp2p，CL：libp2p）。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

## 2\. 传输层基础：TCP / UDP

\- TCP：面向连接、可靠、有顺序保证，适合重要数据传输（RLPx）。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

\- UDP：无连接、更快、不保证可靠和顺序，适合节点发现（Discv）。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

## 3\. Discovery：节点发现（Discv 协议族）

### 整体流程

\- 节点通过硬编码 bootnodes 启动，使用 Kademlia DHT 风格的算法，在路由表中存节点信息（k-buckets，k=16）。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

\- 新节点对 bootnode 发 PING，收到 PONG 后完成“验证 + 绑定”，再用 FINDNODE / NEIGHBOURS 获取更多邻居并重复流程。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

\- 路由表按“最近见到”排序，12 小时不响应的节点会被踢出。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

### Wire 子协议中的核心包结构

\- PING：带版本、from/to 地址、过期时间、ENR 序号等信息。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

\- PONG：带 to、ping-hash、过期时间、ENR 序号等。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

\- FindNode`[target, expiration, ...]`，target 是 64 字节 secp256k1 公钥。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

\- Neighbours：返回最多 16 个邻居节点 `[ip, udp-port, tcp-port, node-id, ...]`。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

\- ENRRequest / ENRResponse：请求 / 返回 Ethereum Node Record。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

\- 所有数据包都封装在 1280 字节左右的 UDP 数据报中，前面是 `hash || signature || packet-type` 头部。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

### Discovery 协议版本：Discv4 vs Discv5

\- 大多数执行客户端已迁移到 Discv5，少数仍在从 Discv4 过渡（如 Besu、Erigon 等）。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

**Discv4 要点**

\- 节点身份：每个节点有 secp256k1 公私钥，公钥作为 Node ID，距离用 Node ID 哈希的 XOR 定义。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

\- ENR：使用 “v4” 身份方案的 Ethereum Node Record 存储连接信息和身份，用 ENRRequest/ENRResponse 拉最新记录。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

\- Kademlia 路由表：256 个 k-buckets，每个桶最多 16 个节点，按距离分段、按最近使用排序，旧节点优先被替换。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

\- Endpoint 验证：只有通过 Ping/Pong 验证的节点才被视为可响应，防止放大攻击。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

\- 递归查找：从若干个最接近目标的节点开始迭代查询，直到找不到更近的 k 个节点为止（典型参数 α=3、k=16）。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

\- 消息类型：Ping、Pong、FindNode、Neighbors、ENRRequest、ENRResponse，全部走 UDP。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

**Discv5 要点**

\- 只存签名的 ENR，避免任意 key–value，增强真实性和可扩展性。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

\- 加密线路：使用 AES‑GCM 加密、ECDH 交换会话密钥，并通过 WHOAREYOU + Handshake 防止伪造。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

\- 路由与查找：同样基于 Kademlia，支持自适应路由和自我修复，使用并行查询增强抗攻击性。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

\- 服务发现：节点可以做 topic advertisement，按“主题半径”搜索特定服务提供者，支持自适应 radius 估计，提高查找效率。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

\- 额外消息类型：Nodes、WhoAreYou、Handshake、TalkReq/TalkResp 等，用于加密握手和自定义协议。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

**Discv4 vs Discv5 对比表**

\- ENR：v4 基本 ENR；v5 可拓展 ENR + metadata。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

\- 安全性：v4 明文；v5 AES‑GCM 加密，有握手和会话密钥。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

\- 服务发现：v4 能力有限；v5 有 topic‑based 查找。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

\- 可扩展性：v4 比较静态；v5 支持多身份方案，更可扩展。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

\- 时间依赖：v4 依赖时钟；v5 消除了时钟依赖。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

\- 可扩展和大网络：v5 对大规模网络更友好。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

### ENR：Ethereum Node Records

\- ENR 由 EIP‑778 定义，是节点连接信息的标准格式：IP、端口、公钥、ID 等。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

\- 主要字段（都可选，只有 `id` 必须）`id`（身份方案，如 "v4"）`secp256k1iptcpudpip6tcp6udp6`。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

\- 结构`[signature, seq, k1, v1, k2, v2, ...]seq` 为 64 位序号`signature` 对 `content = [seq, k, v...]` 的签名。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

\- 编码：RLP 序列化后最大 300 字节，再 base64 文本化并加 `enr:` 前缀（例如示例 ENR 包含 127.0.0.1:30303 和对应 node ID）。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

\- 兼容其他格式：

\- multiaddr`/ip4/127.0.0.1/tcp/30303/<node-id>`。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

\- enode`enode://<node-id>@ip:tcp?discport=udp`，更人类可读。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

## 4\. RLPx：传输与加密（TCP）

### 职责与流程概览

\- RLPx 是基于 TCP 的传输协议，用于在节点间建立加密连接并进行消息收发。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

\- 核心阶段：加密握手建立连接、会话初始化与多路复用、消息分帧与完整性校验。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

### 安全连接建立

\- 节点通过 secp256k1 椭圆曲线生成临时（ephemeral）密钥对，发起认证消息（包含临时公钥和 nonce）。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

\- 对端解密并验证后发回 ack，并发送首个加密 frame（包含 Hello 消息：端口、节点 ID、客户端 ID、协议列表）。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

\- 认证完成后，双方进入正常的加密通信阶段。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

### 会话建立与加密算法

\- 使用 ECIES（基于 secp256k1）进行密钥交换和会话建立：

\- 曲线：secp256k1；KDF：NIST SP 800‑56 concat KDF；MAC：HMAC‑SHA‑256；加密：AES‑128‑CTR。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

\- 加密流程：

1\. 发起方生成随机临时密钥对。

2\. 用 ECDH 计算 shared secret。

3\. 从 shared secret 派生加密密钥 kE 和 MAC 密钥 kM。

4\. 用 AES‑128‑CTR 加密消息。

5\. 用 kM 计算 MAC，保证完整性。

6\. 发送密文和 MAC。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

\- 解密流程反向进行：提取临时公钥、ECDH 计算 shared secret、派生密钥、校验 MAC、AES‑128‑CTR 解密。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

### 节点身份与会话秘密

\- 每个节点有持久 secp256k1 密钥对，公钥为 Node ID，私钥长期保存但不频繁更换。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

\- 重要的密钥材料包括：static‑shared‑secret（长钥对+对方公钥）、ephemeral‑key（临时 ECDH）、shared‑secret、aes‑secret、mac‑secret 等。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

\- static‑shared‑secret 跨会话不变，容易受长期密钥泄露影响；ephemeral‑key 每次会话新生成，提供前向保密（forward secrecy）。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

### 消息分帧与完整性

\- 每个 frame 包含：header‑ciphertext、header‑mac、frame‑ciphertext、frame‑mac，用 AES 加密和 MAC 保护。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

\- 使用两个 keccak256 MAC 状态（入站/出站），随每个 frame 更新，防篡改。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

\- RLPx 支持在一个连接上多路复用多个“能力”（capabilities）/子协议。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

## 5\. P2P 能力与子协议

### p2p 基础能力与核心消息

\- “p2p” capability 必须支持，用于初始协商。核心消息包括：Hello、Disconnect、Ping、Pong 等。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

\- Hello：宣布支持的子协议列表和版本。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

\- Disconnect：优雅断开，可带原因码（例如协议违规、重复连接、身份不符等）。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

### 常见子协议

\- `eth`：主链数据交换，负责区块传播、交易转发等。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

\- `snap`：状态同步协议，让节点能按片加载状态 trie，加快同步。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

\- `les`：轻客户端协议，轻节点从全节点请求所需数据，无需存全状态。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))

\- `portal`：Portal Network，用于轻量客户端的分布式状态/区块/交易检索网络。 \[epf\]([https://epf.wiki/#/wiki/EL/devp2p](https://epf.wiki/#/wiki/EL/devp2p))
<!-- DAILY_CHECKIN_2026-04-12_END -->

# 2026-04-11
<!-- DAILY_CHECKIN_2026-04-11_START -->





# 以太坊虚拟机 (EVM)

## 1\. 核心概念

\* **定义**: EVM（Ethereum Virtual Machine）是以太坊世界计算机的核心，负责执行智能合约的计算，并将结果永久存储在区块链上。

\* **状态机模型**: 以太坊本质上是一个\*\*基于交易的状态机\*\*。

\* **输入 (Input)**: 交易 (Transactions)

\* **状态转换函数**: EVM 本身。它根据输入和当前状态，计算出新的状态。

\* **输出 (Output)**: 新的世界状态 (World State)

\* **世界状态**: 20字节地址到账户状态（包含存储、代码、余额等）的映射。账户分为\*\*外部账户\*\*（私钥控制，无代码）和\*\*合约账户\*\*（由EVM代码控制）。

## 2\. 虚拟机范式与字节码

\* **为什么需要虚拟机?** 跨平台兼容。开发者只需为 EVM 编写代码，EVM 负责在不同操作系统的节点上统一执行。

\* **字节码 (Bytecode)**: 智能合约编译后的形式。由一系列 8 位（1字节）的指令组成。

\* **操作码 (Opcode)**: 指令本身（如 `ADD`, `MSTORE`）。

\* **操作数 (Operand)**: 操作码的输入参数。

\* **字长 (Word Size)**: EVM 的基本处理单元是 **32 字节 (256 bits)**。

## 3\. EVM 的四大核心数据位置

EVM 在执行期间通过四个主要位置存储和读取数据：

### ① 栈 (Stack)

\* **特点**: 后进先出 (LIFO)，用于存放计算的中间值和操作码的参数。

\* **限制**: 最大深度 1024 项，每项 32 字节。只能访问栈顶的 16 项（由 `DUP1-16` 和 `SWAP1-16` 决定）。栈耗尽会导致交易失败。

### ② 内存 (Memory)

\* **特点**: 临时存储 (非持久化)，用于存储整个程序生命周期内需要的大块数据。

\* **寻址**: 线性字节数组。初始值全为 0。

\* **计费机制**: 内存按“页”（32字节）动态扩展，扩展会消耗额外的 Gas。

\* **常用操作码**: `MSTORE` (存入一个字), `MSTORE8` (存入一字节), `MLOAD` (读取)。

### ③ 存储 (Storage)

\* **特点**: 永久存储，保存在区块链的世界状态中。与特定账户绑定，其他账户无法直接访问。

\* **结构**: 巨大的键值对数据库（基于字寻址的字数组，2^256 个槽位，每个槽位 32 字节）。

\* **成本**: 读写存储极度消耗 Gas（因为需要全网节点同步）。

\* **常用操作码**: `SSTORE` (写入), `SLOAD` (读取)。将存储值从非 0 置为 0 会获得 Gas 退款机制。

### ④ 调用数据 (Calldata)

\* **特点**: 只读数据。包含触发智能合约执行的原始交易数据或函数参数。

\* **常用操作码**: `CALLDATALOAD` (加载到栈), `CALLDATACOPY` (复制到内存)。

## 4\. 关键运行机制

\* **程序计数器 (Program Counter)**: 追踪字节码数组中下一条需要执行的指令位置。通过 `JUMP` 和 `JUMPDEST` 可以实现动态控制流（如循环、条件分支）。

\* **Gas 机制**:

\* **作用**: EVM 的计算资源货币，用于防止无限循环和 DoS 攻击。

\* **原理**: 每执行一个操作码都会消耗特定的 Gas。如果交易耗尽 Gas，EVM 会强制中止执行并回滚状态，但已消耗的 Gas 不会退还。

## 5\. 总结

高级语言（如 Solidity）编写的智能合约被编译为 EVM 字节码。EVM 作为状态转换引擎，通过操作码消耗 Gas，并在栈、内存、调用数据和存储之间处理数据，最终更新以太坊的世界状态。

# 以太坊区块构建与交易执行流程

## 1\. 核心架构背景

\* **节点组成**：以太坊节点由 **共识层 (CL)** 和 **执行层 (EL)** 组成。

\* **协作关系**：两者在每个时隙 (Slot) 协同工作以产出区块。CL 负责共识决策，EL 负责区块构建和交易执行。

\* **区块来源**：验证者不仅可以使用自身 EL 构建的区块，还可以通过 **PBS (提议者-构建者分离)** 广播外部构建者生成的区块。

## 2\. 有效负载构建流程 (Payload Building Routine)

区块构建的触发逻辑如下：

1\. **触发**：当共识层 (CL) 选定提议者后，通过 Engine API 的 `engine_forkchoiceUpdated` 端点指示执行层 (EL) 开始构建。

2\. **交易来源**：节点通过 **Gossip 协议** 在 P2P 网络中广播交易，验证通过后存入 **内存池 (Mempool)**。

3\. **构建环境**：CL 提供上下文信息（时间戳、块号、基础费用等），EL 据此初始化构建环境。

## 3\. 区块构建的简化逻辑 (以 Geth 为例)

区块构建是一个循环挑选交易的过程，核心逻辑如下：

```
// 简化模拟 Geth 的区块构建过程
func build(env environment, pool txpool.Pool, state state.StateDB) {
    gasUsed := 0
    // 循环直到 gas 达到限制 (30M) 或交易池为空
    for gasUsed < 30_000_000 && !pool.Empty() {
        transaction := pool.Pop() // 获取当前价值最高的交易
        
        // 在 EVM 中尝试运行交易
        res, gas, err := vm.Run(env, transaction, state)
        
        if err != nil {
            continue // 交易无效则跳过，尝试下一笔
        }
        
        // 累加 gas 并记录交易
        gasUsed += gas
        transactions.append(transaction)
    }
    // 最终处理：计算状态根、收据根、提款根等，封包区块
    return core.Finalize(env, transactions, state)
}
```

## 4\. 区块构建在 Geth 源码中的具体路径

![image.png](https://raw.githubusercontent.com/IntensiveCoLearning/EPF_Bootcamp/main/assets/BareerahBenjamin/images/2026-04-11-1775890779970-image.png)

## 5\. 关键点

-   **Gas 限制**：单块 Gas 限制目前约为 30,000,000，决定了区块的容量上限。
    
-   **鲁棒性**：如果单笔交易执行失败（如 Gas 不足或合约逻辑报错），EL 会跳过该交易并继续尝试下一笔，以确保区块尽可能被填满。
    
-   **最终化 (Finalization)**：区块构建的最后一步是计算复杂的 Merkle 根（交易根、收据根等），这些根会被包含在区块头中。
    

# 执行层数据结构

以太坊执行客户端需要存储当前状态和历史数据。这些数据主要通过类似字典树（Trie）的结构——**默克尔帕特里夏树 (Merkle Patricia Trie, MPT)** 来组织和存储。

## 一、 底层基础数据结构

### 1\. 默克尔树 (Merkle Tree)

\* **核心作用**：数据完整性与防篡改验证。

\* **结构特点**：

_基于哈希的_**二叉树**（叶子节点数必须为偶数，奇数则复制最后一个）。

\* **叶子节点**存储实际数据，**非叶子节点**存储其子节点的哈希值。

_具有_**雪崩效应**：底层数据的微小改变会导致根哈希（Root Hash）完全改变。

\* **在以太坊中的应用**：区块头中只存储默克尔根（Merkle Root），以此指代整个底层数据集。

### 2\. 帕特里夏树 (Patricia Trie / Radix Tree)

\* **核心作用**：高效的数据存储与检索。

\* **结构特点**：

\* n 叉树结构。

\* **前缀压缩**：将具有相同前缀的路径合并为一条边（Edge），而不是每个字符占用一个节点。

_与普通字典树不同，帕特里夏树的_**值 (Value) 仅存储在叶子节点**，极大地提高了空间效率。

### 3\. 默克尔帕特里夏树 (MPT)

以太坊结合了上述两者的优点：**Merkle（密码学验证） + Patricia（高效键值存储）**。

\* **寻址方式**：使用半字节（Nibble，4位/1个十六进制字符）作为键值的路径导航单位。

\* **节点类型（3种）**：

1\. **分支节点 (Branch Node)**：长度为 17 的数组（16 个十六进制分支 + 1 个自身值），用于路径分叉。

2\. **扩展节点 (Extension Node)**：当路径没有分叉时，用于压缩共同的前缀路径，优化空间。

3\. **叶子节点 (Leaf Node)**：包含剩余的键路径（前缀）和最终的实际值（Value）。

## 二、 以太坊的“四大核心树”

以太坊在运行过程中维护着四棵修改版的 MPT 树。每个区块的区块头都会包含这前三棵树的**根哈希**。

### 1\. 交易树 (Transaction Trie)

\* **作用域**：区块级别（每个区块一棵树）。

\* **生命周期**：区块确认后**永久不变**。

\* **存储内容**：当前区块内的所有交易。

\* **键值对**`RLP(交易索引 Index) -> RLP(交易详情 Transaction)`。

### 2\. 收据树 (Receipt Trie)

\* **作用域**：区块级别（每个区块一棵树）。

\* **生命周期**：区块确认后**永久不变**。

\* **存储内容**：交易执行的结果（消耗的 Gas、日志/事件、状态码等）。

\* **核心用途**：允许轻客户端通过 Merkle 证明独立验证交易结果；全节点通过快照同步 (Snap Sync) 快速下载收据而无需重新执行历史交易。

\* **键值对**：交易在区块中的索引作为 Key。

### 3\. 世界状态树 (World State Trie)

\* **作用域**：全局唯一。

\* **生命周期**：动态演进，随每个区块的交易执行而**不断更新**。

\* **存储内容**：所有以太坊账户（包含 EOA 外部账户和智能合约账户）的当前最新状态。

\* **键值对**：

\* **Key**`keccak256(账户地址)`。哈希化可以防止恶意构造长路径攻击。

\* **Value**：RLP 编码的账户对象 `[nonce, balance, storageRoot, codeHash]`。

\* **核心意义**：这棵树的根哈希（State Root）是以太坊当前全局状态的密码学承诺。

### 4\. 存储树 (Storage Trie)

\* **作用域**：合约级别（每个智能合约账户一棵树，普通 EOA 账户此树为空）。

\* **生命周期**：动态演进（通过 `SSTORE` 写入`SLOAD` 读取）。

\* **存储内容**：特定智能合约内部的所有持久化变量和数据。

\* **键值对**`keccak256(存储槽索引 Slot Index) -> RLP(存储值)`。

\* **关联方式**：存储树的根哈希`storageRoot`）存储在世界状态树的对应账户叶子节点中。

## 三、 维克尔树 (Verkle Trees)

Verkle Tree `Vector Commitment` + `Merkle Tree`) 是以太坊路线图（The Verge阶段）中提议取代现有 MPT 的新一代数据结构。

### 1\. 为什么需要 Verkle Tree？

\* **MPT 的痛点**：由于是基于哈希的 16 叉树，树的深度很深。要证明一个叶子节点的值，需要提供整条路径上所有的兄弟哈希，导致**见证数据（Witness）极大**（约 4MB）。这阻碍了“无状态客户端”的实现。

### 2\. 结构与优势

\* **更宽更扁**：节点拥有 256 个子节点，大幅降低树的深度。

\* **向量承诺 (Vector Commitments)**：使用特殊的向量承诺取代普通的哈希计算。

\* **极小的证明体积**：不管树有多大，见证数据可以通过聚合承诺被压缩到极小。见证数据量从 4MB 锐减至 **150 KB** 左右。

### 3\. 核心目标

\* 能够将见证数据直接打包在区块中广播。

_实现真正的_**无状态客户端 (Stateless Clients)**：节点无需在本地硬盘存储数百 GB 的庞大以太坊全局状态，仅凭区块内携带的少量证明数据，即可验证状态转换的正确性，极大降低全节点的硬件门槛。
<!-- DAILY_CHECKIN_2026-04-11_END -->

# 2026-04-10
<!-- DAILY_CHECKIN_2026-04-10_START -->







今天主要把[https://ethcoreplay.cc.cd/](https://ethcoreplay.cc.cd/)上的测试都做了一下，感觉不是很难
<!-- DAILY_CHECKIN_2026-04-10_END -->

# 2026-04-09
<!-- DAILY_CHECKIN_2026-04-09_START -->








# 执行层客户端架构

## 一、 执行层架构概述

-   以太坊执行层客户端的核心职责不仅限于基础的交易执行，还包括区块链数据的验证与本地副本存储、通过 Gossip 协议与其他客户端进行网络通信、维护交易池，以及响应共识层（Consensus Layer, CL）的指令以驱动其功能 。
    
-   这种多方面的高效协作确保了以太坊网络的稳健性和数据完整性 。
    
-   架构的核心是由顶部的执行引擎（Execution Engine）驱动执行层，而执行引擎自身受控于以太坊共识层 。
    

## 二、 核心执行与状态组件

**以太坊虚拟机 (EVM)**：EVM 是专为以太坊程序设计的虚拟化执行引擎 。它旨在消除底层硬件架构（如 x86、ARM、RISC-V）指令集差异带来的不可预测性，确保在所有以太坊客户端上实现一致的计算结果，从而促成网络共识 。以太坊采用“三明治复杂性模型”作为设计理念，即外层保持极简（如高级语言 Solidity 与 EVM 字节码），而将所有复杂逻辑集中在中间层（即将 Solidity 编译为字节码的编译器） 。

**全局状态 (State)**：以太坊是一个按状态机运行的通用计算系统，其状态转换取决于输入的交易 。这与比特币基于未花费交易输出（UTXO）的模型存在显著差异，以太坊维护的是一个“全局状态” 。状态包含了全面的数据集合及数据结构（如 Merkle-Patricia 树），用于存储账户地址、余额、智能合约代码与网络当前状态等 。

**状态转换 (State Transition)**：EVM 内处理合法交易会触发状态转换，进而修改以太坊网络的全局状态 。在执行层中，区块内任何交易或标头的验证失败都会“污染”整个区块，导致整个区块被判定为无效 。

## 三、 通信与接口规范

执行层客户端采用不同的接口协议应对网络中不同的交互主体：

**DevP2P**：这是执行层客户端之间的对等（P2P）通信接口 。外部交易最初存放在内存池（Mempool）中，随后客户端通过 DevP2P 在网络中广播 。每个接收节点在进行二次广播前均会验证交易的合法性，以防止垃圾信息和无效状态转换 。

**JSON-RPC API**：作为面向外部用户（如钱包、DApp）的标准化通信层，用于查询以太坊状态或派发由钱包签名的交易 。

**Engine API**：专门用于共识层与执行层之间进行内部通信的受身份验证接口 。它基于 HTTP 提供 JSON-RPC 接口，并采用 JWT 令牌进行身份验证（证明发送方为合法的共识层客户端），不对外部公开 。这实现了清晰的职责分离：共识层处理分叉选择和共识，执行层负责验证并执行交易 。主要端点包括：

**Exchange of Capabilities**：在节点启动时协商支持的 API 版本（如 V1, V2, V3），确保双方协议兼容 。

**New Payload**：负责载荷验证及状态插入。当接收到新信标区块时，共识层调用此方法，执行层进而验证父哈希、执行交易并更新状态，返回状态反馈（如 VALID 或 INVALID） 。

**Fork Choice Updated**：管理状态同步并触发区块构建，更新规范链头部（Canonical Head） 。

## 四、 节点同步机制

为准确处理以太坊交易，执行层客户端需要就网络的全局状态达成共识，该过程由共识层的 LMD-GHOST 分叉选择规则触发 。

**全同步 (Full Sync)**：客户端请求自创世以来的每个区块头及区块体，在 EVM 中按顺序重放每一笔交易，逐步重建状态树 。此模式提供了最高的安全性保障，但消耗极大的 CPU、磁盘和网络资源，通常耗时数天 。注：在 EIP-4444 实施后，网络将不再支持从创世区块进行全同步，而是基于弱主观性检查点（Checkpoint）进行同步 。

**快照同步 (Snap Sync)**：以大幅降低时间成本为目标（从数天缩短至数小时），客户端选择一个近期敲定的区块作为枢纽块（Pivot block），直接获取并下载状态树叶子节点（账户和存储）的连续数据块以及关联合约字节码 。

**自愈阶段 (Healing Phase)**：由于在下载快照数据库期间区块链仍在推进，获取的数据可能出现过时或不一致。客户端通过扫描数据库并获取缺失的 Trie 节点或存储槽来进行数据自愈，确保能重建完整且合法的 MPT 树 。

## 五、 交易池与底层数据库

**1.交易池架构**：

**Legacy Pools (传统池)**：由执行客户端管理，利用双重堆结构（优先考虑未来区块的有效小费或 Gas 费上限）按价格对交易进行组织和淘汰 。

**Blob Pools**：同样采用优先级堆进行交易驱逐，但其内部操作机制包含对数函数等不同实现方式 。

**2.数据库后端引擎**：执行客户端需要将庞大的历史区块（古代数据库）及当前状态（近期状态树）持久化至磁盘 。

**LevelDB**：早期常用的嵌入式键值数据库，采用日志结构合并树（LSM-tree）。其设计虽有利于高吞吐量的顺序写入，但在以太坊极度频繁的状态更新场景下会导致严重的写入放大及不可预测的延迟 。

**Pebble**：部分执行客户端（如 Geth）已将 Pebble 作为默认数据库后端替代 LevelDB 。它同样基于 LSM-tree，但专为以太坊典型的高频写入和延迟敏感型工作负载进行了深度优化，可显著加强对数据压实（Compaction）行为的控制权 。此外，**MDBX** 也是部分客户端探索的高性能数据库方案之一 。

## 六、目前的客户端介绍

| 客户端 | 开发语言 | 核心开发者 | 核心杀手锏 (Noteworthy Features) |

| ------ | ------ | ------ | ------ |

| Geth | Go | 以太坊基金会 | 老牌霸主。生态最稳定、支持最广；拥有强大的自定义 EVM 追踪器 (Tracers) 和完善的监控面板。 |

| Nethermind | C# | Nethermind | 性能先锋。针对 .NET 基础设施深度优化，适合企业级集成，插件系统非常灵活。 |

| Besu | Java | Consensys / Hyperledger | 企业级首选。支持私有链、并行交易执行，是唯一在 Hyperledger 框架下的主流执行层客户端。 |

| Erigon | Go | Ledgerwatch | 存储优化专家。重新设计了 MPT 数据库，存档节点体积缩小 5 倍（3TB 以内），且能在 3 天内完成同步。 |

| Reth | Rust | Paradigm | 新晋性能怪兽。参考 Erigon 设计并用 Rust 重写，模块化程度极高，兼具极速同步与极小的磁盘占用。 |

如果是**普通开发者/节点运营商**，选 **Geth** 最稳；如果**磁盘空间有限**或需要分析全量历史数据，选 **Erigon** 或 **Reth**；如果是**企业用户**且需要私有链功能，选 **Besu**。
<!-- DAILY_CHECKIN_2026-04-09_END -->

# 2026-04-08
<!-- DAILY_CHECKIN_2026-04-08_START -->









**以太坊执行层**

**一、执行层规范概述**

以太坊执行层最初在黄皮书中被完整定义，当时它涵盖了协议的全部体系。目前最新的规范为以太坊执行层规范（Ethereum Execution Layer Specification，简称 EELS）的 Python 实现版。黄皮书 Paris 版本（705168a，2024-03-04）已过时，未包含合并后的协议更新内容。Python 执行层规范是当前核心实现参考，所有 EIP 细节可查阅规范仓库的 README 文件。本笔记聚焦执行层的核心架构、设计逻辑，以及与状态转换、区块头验证、Gas 计费相关的关键规范。

**二、状态转换函数**

执行层的核心职责是排他性地执行状态转换函数（State Transition Function，简称 STF）。该函数解决两个根本问题：当前区块能否被合法追加到区块链末端，以及区块执行后系统状态会发生怎样的变化。黄皮书中定义的区块级状态转换函数简化公式为：

\\\[

\\sigma\_{t+1} \\equiv \\Pi(\\sigma\_t, B) \\tag{2}

\\\]

其中 \\(\\sigma\_{t+1}\\) 表示应用当前区块后的新状态，\\(\\Pi\\) 为区块级状态转换函数，\\(\\sigma\_t\\) 为父状态（旧状态），\\(B\\) 为当前待处理的区块。需要注意，公式中的 \\(\\sigma\\) 与 Python 规范中的 State 类概念不同，系统状态是通过状态折叠函数动态推导得出的，体现了区块链的数学模型与实际实现细节的分离。

Python 规范中状态转换函数的标准化执行流程依次为：获取父区块头、基于父区块头计算并校验超额 Blob Gas、完成当前区块头的全量合法性校验、验证叔块字段为空、执行区块内所有交易并输出 Gas 已使用量、特里树根、日志布隆过滤器及最终系统状态，随后核验执行结果与区块头参数完全匹配。若全部校验通过则将区块上链，否则抛出无效区块错误。最后清理超过最新 255 个区块的历史数据。

**三、区块头验证**

区块头验证是区块链同步与上链的核心环节，在黄皮书与 Python 规范中均有严格定义。其目标是基于协议规则校验区块的完整性，包括哈希、Gas 使用、时间戳等，确保节点可独立验证当前与历史数据。要验证当前区块头 \\(H\\) 的合法性，必须同时满足以下所有条件（依赖父区块 \\(P(H)\\))：

\\\[

\\begin{align\*}

V(H) &\\equiv H\_{\\text{gasUsed}} \\leq H\_{\\text{gasLimit}} \\tag{57a} \\\\

&\\land H\_{\\text{gasLimit}} < P(H)\_{H\_{\\text{gasLimit}}‘} + \\left\\lfloor \\frac{P(H)\_{H\_{\\text{gasLimit}}’}}{1024} \\right\\rfloor \\tag{57b} \\\\

&\\land H\_{\\text{gasLimit}} > P(H)\_{H\_{\\text{gasLimit}}‘} - \\left\\lfloor \\frac{P(H)\_{H\_{\\text{gasLimit}}’}}{1024} \\right\\rfloor \\tag{57c} \\\\

&\\land H\_{\\text{gasLimit}} > 5000 \\tag{57d} \\\\

&\\land H\_{\\text{timeStamp}} > P(H)\_{H\_{\\text{timeStamp}}'} \\tag{57e} \\\\

&\\land H\_{\\text{numberOfAncestors}} = P(H)\_{H\_{\\text{numberOfAncestors}}'} + 1 \\tag{57f} \\\\

&\\land \\text{length}(H\_{\\text{extraData}}) \\leq 32\\ \\text{bytes} \\tag{57g} \\\\

&\\land H\_{\\text{baseFeePerGas}} = F(H) \\tag{57h} \\\\

&\\land H\_{\\text{parentHash}} = \\text{KEC}(\\text{RLP}(P(H)\_H)) \\tag{57i} \\\\

&\\land H\_{\\text{ommersHash}} = \\text{KEC}(\\text{RLP}(())) \\tag{57j} \\\\

&\\land H\_{\\text{difficulty}} = 0 \\tag{57k} \\\\

&\\land H\_{\\text{nonce}} = 0x0000000000000000 \\tag{57l} \\\\

&\\land H\_{\\text{withdrawalHash}} \\neq \\text{nil} \\tag{57n} \\\\

&\\land H\_{\\text{blobGasUsed}} \\neq \\text{nil} \\tag{57o} \\\\

&\\land H\_{\\text{blobGasUsed}} \\leq \\text{MaxBlobGasPerBlock} = 786432 \\tag{57p} \\\\

&\\land H\_{\\text{blobGasUsed}} \\bmod \\text{GasPerBlob} = 2^{17} = 0 \\tag{57q} \\\\

&\\land H\_{\\text{excessBlobGas}} = \\text{CalcExcessBlobGas}(P(H)\_H) \\tag{57r}

\\end{align\*}

\\\]

超额 Blob Gas 计算公式为：

\\\[

\\text{CalcExcessBlobGas}(P(H)\_H) \\equiv

\\begin{cases}

0, & \\text{if } P(H)\_{\\text{blobGasUsed}} < \\text{TargetBlobGasPerBlock} \\\\

P(H)\_{\\text{blobGasUsed}} - \\text{TargetBlobGasPerBlock}, & \\text{其他情况}

\\end{cases} \\tag{57s}

\\\]

其中 \\(P(H)\_{\\text{blobGasUsed}} \\equiv P(H)\_{H\_{\\text{excessBlobGas}}} + P(H)\_{H\_{\\text{blobGasUsed}}}\\) 且 \\(\\text{TargetBlobGasPerBlock}=393216\\)。

这些规则的核心释义包括：Gas 使用不得超过上限、Gas 上限波动幅度不超过 1/1024 并设置最低 5000 的阈值、时间戳必须严格递增、区块高度依次递增、附加数据长度不超过 32 字节、基础费严格遵循 EIP-1559 计算。合并后还增加了叔块哈希为空、难度值为 0、nonce 全零等兼容性校验，以及提款哈希非空和 Blob Gas 的各项上限、对齐、超额计算规则。这些校验共同落地了以太坊的经济模型，通过 EIP-1559 降低 Gas 波动、提升交易可预测性、稳定发行奖励并销毁基础费，同时 EIP-4844 的 Blob 机制进一步扩展了 Layer 2 的低成本数据上链能力。

**四、Gas 计费**

Gas 计费是执行层经济模型的核心，分为固有 Gas 计算、有效 Gas 价格与优先级费、前置有效性校验三大部分。

**4.1 固有 Gas 计算**

固有 Gas 是一笔交易执行前必须支付的最低费用，涵盖 EVM 处理的基础资源与数据传输成本。适配上海升级的计算公式为：

![image.png](https://raw.githubusercontent.com/IntensiveCoLearning/EPF_Bootcamp/main/assets/BareerahBenjamin/images/2026-04-08-1775654135007-image.png)

其中 g{0} 为总固有 Gas 成本，G{transaction} 固定为 21000 Gas，初始化代码按 32 字节对齐计费，输入数据零字节 4 Gas、非零字节 16 Gas，合约创建额外 32000 Gas，访问列表按地址与存储槽分别加收对应 Gas 成本。交易执行前会先从 Gas 上限中扣除固有 Gas，再初始化 EVM 上下文。

**4.2 有效 Gas 价格与优先级费**

适配 Blob 交易（类型 3）的有效 Gas 价格公式为：

![image.png](https://raw.githubusercontent.com/IntensiveCoLearning/EPF_Bootcamp/main/assets/BareerahBenjamin/images/2026-04-08-1775654037752-image.png)

优先级费公式为：

![image.png](https://raw.githubusercontent.com/IntensiveCoLearning/EPF_Bootcamp/main/assets/BareerahBenjamin/images/2026-04-08-1775653679005-image.png)

有效 Gas 费用为 effectiveGasPrice \* T{gasLimit}，总 Blob Gas 为 G{gasPerBlob}=2^17 \* length(T{blobVersionedHashes})，Blob Gas 价格在未超出目标时固定为 1，超出时呈指数上涨，Blob Gas 费用全部销毁。最高 Gas 费用和前置总成本 v{0} ≡ upfrontCost ≡ (effectiveGasFee + blobGasFee) 共同构成交易执行前的全额扣除项。

**4.3 交易执行前置有效性校验**

交易必须通过以下全部校验才能进入执行阶段：

![image.png](https://raw.githubusercontent.com/IntensiveCoLearning/EPF_Bootcamp/main/assets/BareerahBenjamin/images/2026-04-08-1775653621927-image.png)

其中 m 根据交易类型取 Gas Price 或 Max Fee Per Gas，空账户定义为代码哈希为空、nonce 为 0 且余额为 0。这些规则确保发送者账户存在且为外部账户、nonce 顺序正确、余额充足、Gas 价格不低于基础费且符合 EIP-1559 约束，最终保障交易在执行前已满足所有协议安全与经济要求。
<!-- DAILY_CHECKIN_2026-04-08_END -->

# 2026-04-07
<!-- DAILY_CHECKIN_2026-04-07_START -->










# 协议

## 架构

![Image_20260407_020432.jpg](https://raw.githubusercontent.com/IntensiveCoLearning/EPF_Bootcamp/main/assets/BareerahBenjamin/images/2026-04-06-1775498699238-Image_20260407_020432.jpg)

## 设计原理：

1.  协议设计理念：简洁性、通用性（EVM）、模块化（面向未来）、非歧视性（开源、密码朋克）、敏捷性（协议修改）
    
2.  原则：管理复杂性（三明治模型复杂性（中间层）、封装式复杂性（复杂子系统→简单接口））、自由、泛化、无预设功能。
    
3.  区块链级协议：  
    **账户模型与 UTXO 模型对比**  
    早期区块链如比特币及其衍生品采用 UTXO 模型，将用户余额存储为未花费交易输出（UTXO）。每个 UTXO 代表一笔已授权、可供接收方花费的特定金额加密货币，用户余额就是其私钥能签名花费的所有 UTXO 总和。以太坊则选择账户模型，余额直接记录在账户中。这种模型更灵活，能支持复杂交易场景，尤其适合去中心化交易所等应用。UTXO 虽能提供更高隐私性，但会增加系统复杂度，而账户模型的可互换性（fungibility）更强，任何代币都能相互替换，不会因历史交易而被“污染”。
    
    **账户模型的优势**  
    账户模型显著节省存储空间。例如，一个账户若有 5 个 UTXO，UTXO 模型需约 300 字节，而账户模型只需约 30 字节（包含地址、余额和 nonce）。实际中账户需存储在 Patricia 树里，节省幅度略小，但仍很可观。同时，交易体积更小（以太坊约 100 字节，对比比特币 200-250 字节），因为每笔交易只需一次引用、一次签名和一个输出。账户模型实现简单、易于理解，也更强大，能轻松实现 UTXO 模型无法支持的功能，如不绑定特定 UTXO 的卖单。唯一缺点是为防重放攻击需引入 nonce，旧账户难以完全裁剪；解决方案是让交易携带区块号，定期重置 nonce。
    
    **以太坊状态数据结构：Merkle Patricia Trie（MPT）**  
    以太坊使用改进的 Merkle-Patricia Trie 存储状态，它结合 Patricia 算法和 Merkle 树特性，实现高效检索。MPT 具有确定性和密码学可验证性：状态根哈希唯一对应当前所有状态，任何修改都会改变根哈希，支持 O(log n) 的插入、查找和删除效率。通过 Merkle 证明，可轻松验证两个状态是否一致。目前 MPT 广泛用于需要网络传输成员证明的场景，但以太坊状态已达 1-2TB，节点存储压力巨大，因此社区正在研究更高效的替代方案。
    
    **Verkle 树：下一代状态结构**  
    Verkle 树（向量承诺 Merkle 树）是正在积极研究的替代方案，旨在解决 MPT 的状态膨胀问题。它通过更高的分支因子 k，实现 O(kn) 构造时间和 O(log\_k n) 的成员证明大小，大幅减少证明体积（称为 witness）。与传统 Merkle 树需提供所有兄弟节点哈希不同，Verkle 树只需提供路径上的父节点即可完成证明，这对实现“无状态客户端”（statelessness）至关重要，能让节点无需完整存储全状态即可验证交易。目前 Verkle 树仍在研发阶段，未来有望取代 MPT，进一步提升网络可扩展性。
    
    **序列化方案：RLP 与 SSZ**  
    以太坊早期使用 Recursive Length Prefix（RLP）序列化数据。RLP 设计简单且确定性强，不依赖具体数据类型，只以嵌套数组形式存储结构，保证字节级一致性，不支持键值映射（需手动转为有序数组）。但 RLP 对固定长度数据（如整数、布尔值）效率不高，且不支持 Merkleization，导致轻客户端证明和无状态性难以实现。Ethereum 2.0 Beacon 链引入 Simple Serialize（SSZ）作为改进方案。SSZ 依赖预先定义的 schema，支持可变和固定长度数据，具有高效重哈希和快速索引能力，复杂度远低于 RLP 的 O(n)，完美支持 Merkleization，是实现无状态性和轻客户端证明的关键。
    
    **共识最终性机制：从 Casper FFG 到 Gasper**  
    以太坊 PoS 共识中，最终性（finality）指区块一旦最终确定，除非销毁至少 33% 质押 ETH，否则不可被篡改或移除。Casper Friendly Finality Gadget（FFG）是实现这一目标的叠加协议，它在提案机制之上通过两轮投票最终确定检查点，所有最终检查点成为规范链历史。LMD-GHOST（Latest Message Driven Greediest Heaviest Observed Sub-Tree）则是分叉选择规则，验证者通过认证区块表达支持，选择最重子树作为规范链。两者结合形成 Gasper 协议，成为 Ethereum 2.0 完整的权益证明共识抽象，既保证最终性，又能高效处理分叉。
    
    **P2P 网络中的 DHT 应用**  
    以太坊 P2P 网络同时使用结构化和非结构化技术。底层采用 discv5（基于 Kademlia 的 DHT）存储 ENR 记录，用于节点发现和路由表构建。新节点通过引导节点在 DHT 中查找自身 node\_id，逐步发现其他对等节点，并定期枚举随机 node\_id 扩展网络视图。区块传播则依赖上层 GossipSub 覆盖网络（非结构化）。DHT 虽增加一步，但提供全局视图和简单设计，适合高动态网络的引导启动，比单纯朋友间传播（f2f/PEX）更具鲁棒性。这种混合模式在 BitTorrent 等协议中也被广泛采用，确保节点高效加入并传播最新区块。
    

## 进化路径

**Frontier 阶段（边境）**  
Frontier 是以太坊协议的首个正式发布版本，于 2015 年 7 月 30 日凌晨 3:26:13（UTC）上线，对应创世区块时间戳。它本质上是一次 beta 测试发布，目的是让开发者学习、实验并开始构建去中心化应用和工具。初始 gas 上限被硬编码为 5000，以确保矿工和用户能快速安装客户端并启动网络。随后通过 Frontier Thawing 分叉将 gas 上限提升至约 314 万（精确值为 3,141,592）。为了实现安全启动，Frontier 引入了金丝雀合约机制，这些合约会发出 0 或 1 的二进制信号。如果客户端检测到多个金丝雀合约信号变为 1，就会自动停止挖矿并提示用户升级客户端，从而避免长时间网络中断，确保矿工不会阻碍协议升级。

**Homestead 阶段（家园）**  
Homestead 是以太坊协议的第二个主要版本，于 2016 年 3 月 14 日正式发布，标志着以太坊从 beta 测试阶段过渡到更加成熟稳定的生产环境。这一阶段通过多项 EIP 进行了重要改进。EIP-2 修复了多项问题，包括将通过交易创建合约的 gas 成本从 21,000 提高到 53,000，以减少不必要的交易式合约创建；同时确保合约创建要么完全成功、要么彻底失败，防止出现空合约；还优化了难度调整算法，使区块时间更加稳定。EIP-2 还引入了高 s 值签名无效规则，解决交易可塑性问题，提升交易哈希的可靠性和完整性。EIP-7 在 opcode 0xf4 位置新增 DELEGATECALL 操作码，其功能类似 CALLCODE，但会将 sender 和 value 从父作用域传递到子作用域，方便合约将其他地址作为可变代码源并“透传”调用，且不会额外增加 gas 津贴，使 gas 管理更可预测。EIP-8 则增强了 devp2p 协议的向前兼容性，要求客户端在处理 hello、ping 数据包时忽略版本号和额外元素，静默丢弃未知包类型，并支持新的加密握手编码，从而确保未来网络升级时所有客户端都能平滑过渡，符合 Postel 定律的宽松接受原则。

**The Merge（合并）**  
2022 年 9 月 15 日，以太坊通过激活 EIP-3675 完成了 The Merge 升级，将共识机制从工作量证明（PoW）正式切换为权益证明（PoS）。在此之前，PoW 与执行逻辑处于同一层级，The Merge 后 PoW 被彻底弃用，取而代之的是更加复杂且节能的 PoS 共识。该共识被独立部署为单独一层，即信标链（Beacon Chain），拥有独立的 P2P 网络和逻辑。信标链自 2020 年 12 月 1 日起已独立运行并达成共识，经过长时间无故障稳定表现后，正式成为以太坊的主共识提供者。“The Merge”这一名称正是源于执行层与共识层的成功合并，标志着以太坊从能源密集型挖矿转向更高效、更环保的权益证明时代，同时为后续协议演进奠定了模块化基础。

### （拓展）

**Shapella 升级（Shanghai/Capella）**  
Shapella 是 The Merge 之后的首个重大协议升级，于 2023 年 4 月 12 日正式激活。它同时对执行层（Shanghai）和共识层（Capella）进行协调升级，主要实现了质押 ETH 的提款功能。通过 EIP-4895 等关键改进，验证者终于能够将信标链上锁定的 ETH 及累积奖励提取到执行层，彻底完成了 PoS 转型的最后一块拼图。这一升级极大提升了 staking 的灵活性和用户参与度，同时还引入了多项优化，如 EIP-3651（warm COINBASE）、EIP-3855（PUSH0 操作码）和 EIP-3860（合约创建 gas 优化），让开发者体验更流畅，也为后续扩展奠定了基础。

**Dencun 升级（Cancun/Deneb）**  
Dencun 升级于 2024 年 3 月 13 日上线，是 Ethereum 扩展路线图中的重要里程碑。它通过执行层（Cancun）和共识层（Deneb）的协同升级，首次引入了 Proto-Danksharding 机制（核心 EIP-4844），为 Layer 2 提供了廉价的临时数据可用性（blobs）。这一创新大幅降低了 Rollup 的交易成本，让 Layer 2 费用显著下降，同时保留了主链的安全性。Dencun 还包含其他改进，如 EIP-4788（信标根存储）和多项 gas 优化，进一步推动了 Ethereum 向高吞吐量、可扩展网络的演进，为大规模采用铺平道路。

**Pectra 升级（Prague/Electra）**  
Pectra 升级于 2025 年 5 月 7 日激活，是 The Merge 以来 EIP 数量最多的一次硬分叉（共 11 个 EIP）。它同时升级了执行层（Prague）和共识层（Electra），重点引入了账户抽象能力（EIP-7702），允许外部拥有账户（EOA）临时执行智能合约代码，实现交易批量处理、gas 赞助和社会恢复等高级功能。这一升级还优化了验证者体验：EIP-7251 将最大有效验证者余额提升至 2048 ETH，EIP-6110 显著缩短了验证者入网时间，并进一步扩大了 blob 吞吐量。Pectra 极大提升了用户体验和网络效率，被视为 Merge 后最具影响力的协议改进之一。

**Fusaka 升级**  
Fusaka 升级于 2025 年 12 月 3 日完成，标志着 Ethereum 开始进入半年一次的固定升级节奏。它主要聚焦进一步扩展和数据可用性优化，核心引入 PeerDAS（EIP-7594）机制，让验证者通过随机采样而非完整下载即可验证 blob 数据，从而将 blob 容量理论上提升 8 倍。同时伴随多个 Blob Parameter Only（BPO）小升级，逐步提高每区块 blob 数量。这一升级延续了 Dencun 和 Pectra 的扩展成果，进一步降低 Layer 2 成本，并为后续并行执行等高级特性打下基础。

**后续发展展望**  
截至 2026 年 4 月，Glamsterdam 升级（计划于 2026 年上半年）已在开发中，重点方向包括并行执行、更高的 gas 限制、内置 PBS（Proposer-Builder Separation）以及持续的 blob 扩展和隐私增强。其后的 Hegotá 升级也已规划，Ethereum 正稳步迈向“完全扩展、极致韧性”的长期目标。这些升级共同体现了协议在安全性、可扩展性和用户友好性上的持续演进。
<!-- DAILY_CHECKIN_2026-04-07_END -->

# 2026-04-06
<!-- DAILY_CHECKIN_2026-04-06_START -->












历史发展：

ARPANET（互联网开放无国界精神）→ Unix哲学（模块化、简洁、可组合的软件设计）→ 公钥密码学（去中心化系统的安全底层）→ 自由开源运动（全球化分布式协作范式、代码自由）→ 密码朋克运动（代码即规则、个人主权、去中心化理念）→ 早期数字货币理论与实践 → 比特币（去中心化账本与数字货币的落地验证）→ 以太坊（通用去中心化世界计算机，完整继承并落地了半个世纪的核心理念）

关键人物及其贡献：

| 领域 | 核心人物 | 关键贡献 |

|---------------|-----------------------------------|-----------------------------------------------|

| 软件设计根基 | 肯·汤普森、丹尼斯·里奇 | 发明Unix系统与C语言，奠定Unix哲学 |

| 开源运动 | 理查德·斯托曼 | 发起GNU项目，创造GPL协议，定义自由软件核心理念 |

| 开源运动 | 林纳斯·托瓦兹 | 开发Linux内核，奠定分布式开源协作范式 |

| 现代密码学 | 拉尔夫·默克尔、迪菲、赫尔曼 | 开创非对称加密与无需信任的密钥交换体系 |

| 现代密码学 | 李维斯特、萨莫尔、阿德曼 | 发明RSA公钥加密系统，落地现代密码学商用 |

| 密码朋克 | 蒂莫西·梅、埃里克·休斯 | 发起密码朋克运动，提出加密无政府主义核心理念 |

| 密码朋克 | 菲尔·齐默尔曼 | 开发PGP加密程序，推动加密技术向公众普及 |

| 数字货币先驱 | 大卫·乔姆、戴伟、尼克·萨博 | 早期数字货币与数字稀缺性的理论与实践探索 |

| 比特币 | 中本聪 | 发明比特币与区块链，实现去中心化点对点电子现金 |

| 以太坊 | 维塔利克·布特林、加文·伍德 | 设计并落地以太坊，打造通用智能合约平台 |
<!-- DAILY_CHECKIN_2026-04-06_END -->
<!-- Content_END -->
