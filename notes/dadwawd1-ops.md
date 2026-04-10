---
timezone: UTC+8
---

# dadwawd1

**GitHub ID:** dadwawd1-ops

**Telegram:** @dadwawd1

## Self-introduction

热爱研究以太坊协议少年，目前正打算全面强化自己，去推销自己

## Notes

<!-- Content_START -->
# 2026-04-10
<!-- DAILY_CHECKIN_2026-04-10_START -->
EL，负责在Ethereum区块链的状态转移只管交易执行和状态更新，决定一个区块是否合法添加到链上，以及状态如何从上一个区块更新到下一个区块，跟cl配合。

状态转移函数的概念是把当前区块的所有交易按顺序执行，得到最终的新状态。不是固定存放在某个地方，而是通过状态塌陷函数动态生成

区块添加完整流程:

获取父区块的 Header—>检查多余 blob gas 等字段是否正确—>确认 ommers字段必须为空—>执行区块内所有交易—>把执行结果与区块头里的承诺对比—>如果全部通过，就把区块加到链上删除老的区块—>任何一步失败，就直接报“Invalid Block”错误，整个区块被拒绝

今天读完了vops的介绍（Validity-Only Partial Statelessness）——仅有效性部分无状态，跟现有的方案相比弱状态和强状态都有很明显的缺点，vops只需要保存节点只需保存极简账户四元组，即可本地验证交易有效性（address，nonce、balance、codeFlag），无需每笔交易的 witness。他的核心创新在于每个节点仅仅维护(address, nonce, balance, codeFlag) 四元组

EOA（普通账户）：约 40.125 字节/账户

codeFlag：

-   0 = 普通 EOA（允许多笔 pending tx）
    
-   1 = EIP-7702 委托账户（每地址仅允许 1 笔 pending tx，防止状态冲突）
    

存储规模（基于 ~2.41 亿账户）：

241 百万×40.125 字节≈8.4 GiB241\\,\\text{百万} \\times 40.125\\,\\text{字节} \\approx 8.4\\,\\text{GiB}241百万×40.125字节≈8.4GiB

→ 比当前完整状态（~233 GiB）缩小约 25 倍。

VOPS 提供了一个简单有效的桥梁：在将本地存储降低 25 倍的同时，保留了功能完备、具有审查阻力的公共 mempool。
<!-- DAILY_CHECKIN_2026-04-10_END -->

# 2026-04-09
<!-- DAILY_CHECKIN_2026-04-09_START -->


占个位
<!-- DAILY_CHECKIN_2026-04-09_END -->

# 2026-04-08
<!-- DAILY_CHECKIN_2026-04-08_START -->



根据视频的思考（[https://www.youtube.com/watch?v=1trSdmKR9co](https://www.youtube.com/watch?v=1trSdmKR9co)）：  
FOCIL 让验证者委员会生成“包含列表（Inclusion List, IL）”，强制/激励构建者把列表里的交易包含进区块，从而解决 MEV、审查、交易卡住等问题，同时保持区块生产的灵活性。

当前工作原理

-   包含委员会从内存池拉交易（按优先费、在池时间等策略）。
    
-   IL 通过 P2P 广播。
    
-   构建者收到 IL 后，应把列表交易包含进 payload（可自行排序）。
    
-   验证者（attester）检查 payload 是否符合 IL，未包含可包含交易 → 可能触发 reorg。
    
-   当前不支持 blob 交易（因数据可用性未知），但未来会通过系统合约解决。
    

未来方向

-   Frame Transactions：新型交易类型，把操作拆成“帧”（验证/执行分离），支持原生账户抽象（gas 赞助、会话密钥等）。FOCIL 将限制哪些 frame 能进 IL，控制验证者/构建者负载。
    
-   Blob Streaming：通过“票务”出售 blob 空间，提前传播、平滑带宽，提升吞吐。结合 FOCIL 可支持 blob 交易包含。
    
-   加密内存池集成：进一步提升隐私（仍在研究中）。
<!-- DAILY_CHECKIN_2026-04-08_END -->

# 2026-04-07
<!-- DAILY_CHECKIN_2026-04-07_START -->




| 维度 | CPU 伪随机 | 熔岩灯（Lava Lamp） | PoC (Proof of Cat) |
| --- | --- | --- | --- |
| 能否被预测 | 依赖初始种子 种子若弱或可预测 → 整体可预测 | 物理混沌运动（液体对流、气泡） 实际难以精确建模 | 极难建模的极端物理过程 理论上几乎无法预测 |
| 成本高低 | 极低：便宜、快、易部署（软件/CPU jitter即可） | 中高：需硬件装置 + 摄像头 + 处理系统 | 高：属于“想象级”极端物理实现，部署复杂 |
| 随机性上限 | 有限 受种子质量 + 算法限制，需持续重置种子 | 足够高 成熟的物理熵源，已被实际验证 | 极高 物理真实熵，几乎无上限 |
| 攻击者篡改难度 | 容易 软件层面可被恶意代码/侧信道攻击 | 较难 物理世界干扰成本高 | 极难 建模都困难，篡改几乎不可能 |

1.CPU 伪随机 优势：部署门槛最低，几乎零成本，速度飞快。 致命弱点：必须依赖高质量初始种子。一旦种子被预测、泄露或被攻击，整个随机序列就崩了。属于“伪随机”而非“真随机”。

2.熔岩灯 成熟的物理熵思路（Cloudflare 等公司真实使用）。 通过拍摄熔岩灯内混沌的液体流动 + 气泡运动提取熵，属于“已落地”的物理随机性方案。随机性质量远超纯软件方案，但需要专用硬件，成本和维护门槛更高。

3.PoC (Proof of Cat) 代表一种“极难建模”的极端物理熵想象。 不是具体某项技术，而是把“无法精确预测的物理现象”推到极致（如极度复杂的生物/物理系统）。其核心价值在于提供一个思想实验：如果我们能找到足够“野”、足够复杂的物理过程，就能在理论上达到近乎完美的不可预测性和抗篡改能力。

真随机（True Random） 的定义： 必须直接来自不可预测的物理过程（热噪声、量子效应、熔岩灯混沌、PoC级极端物理等），完全非确定性，没有算法参与。

/dev/urandom 的本质： 内核先把各种物理熵（CPU jitter、磁盘I/O、中断、传感器等）收集到一个熵池里搅拌成种子，然后用ChaCha20 / SHA-256 等强算法持续生成后续序列。 一旦种子就绪，后续输出就是确定性算法产生的——只是因为初始种子质量极高 + 持续混入新熵，才显得“不可预测”。
<!-- DAILY_CHECKIN_2026-04-07_END -->

# 2026-04-06
<!-- DAILY_CHECKIN_2026-04-06_START -->





以太坊的设计核心理念是简单、普适、模块化。 它起源于 Vitalik 等人在 2013-2014 年的愿景：构建一个图灵完备的虚拟机（EVM），让开发者能直接在区块链上编写任意去中心化应用，而无需底层复杂性。 早期区块链（如比特币）采用 UTXO 模型，而以太坊选择账户模型（Account-based），因为它更灵活、易实现、支持原生合约，且账户是可互换的（fungible）。 数据结构上采用 Merkle-Patricia Trie（MPT）实现高效可验证的状态管理；序列化使用 RLP（后续引入 SSZ 用于 Eth2）；共识从 PoW 演进到 PoS 的 Casper FFG + LMD GHOST（Gasper 组合）；网络层采用 discv5 DHT + GossipSub。 早期（2014-2016）设计强调“未来证明”，通过模块化封装复杂性，为后续 Layer 2 扩展和 Stateless 研究奠定基础.

![fe855952acca1343d848838f5fbf39f8.png](https://raw.githubusercontent.com/IntensiveCoLearning/EPF_Bootcamp/main/assets/dadwawd1-ops/images/2026-04-06-1775487211309-fe855952acca1343d848838f5fbf39f8.png)

讲解：这张图展现了现在的以太坊框架，用户通过json-RPC发送交易到执行层的mempool通过p2p扩散到全网，共识层的randao（伪随机源）选出proposer从mempool打包生成payload，再通过engine API吧payload交给执行层执行，通过跑EVM更新状态，返回stateRoot，共识层再通过LMD-GHOST + Attestations来确认这个区块，验证者通过beacon APIs投票达成最终目的
<!-- DAILY_CHECKIN_2026-04-06_END -->
<!-- Content_END -->
