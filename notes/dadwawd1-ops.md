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
# 2026-04-26
<!-- DAILY_CHECKIN_2026-04-26_START -->
EIP-8141 的 Frame Transaction 机制将传统以太坊交易从“单一签名与单一调用”扩展为由多个 frame 组成的结构化交易，使验证、付款、执行、部署和后处理能够在协议层被清晰拆分。基于该机制，本文建议重点开展两个源码研究方向：一是 Mempool Admission Simulator，用于模拟节点在 public mempool 中接收 EIP-8141 交易时的判定流程，包括解析 frame transaction、识别 validation prefix、模拟验证前缀、检查 banned opcode、限制第三方 mutable storage 读取、管理 canonical 与 non-canonical paymaster reservation，并最终将交易分类为 `public`、`local-only`、`private-builder-only` 或 `invalid`；二是 Frame Wallet SDK，用于帮助钱包和 dApp 构造 frame transaction，支持 self-verify、paymaster sponsor、atomic batch、session key、P256/passkey 等能力，并在交易提交前调用 mempool simulator 判断其公共传播性和风险。两个项目可以形成联动：SDK 负责生成交易，Simulator 负责验证其 public mempool eligibility，从而同时服务于协议研究、钱包开发、测试向量设计和 EIP 文档 PR。整体上，该方向最有价值的研究点在于区分 `consensus-valid` 与 `public-mempool-propagatable`，尤其是 ERC-20 paymaster、复杂验证逻辑和 VERIFY.data 未进入签名哈希所带来的安全与传播边界问题。
<!-- DAILY_CHECKIN_2026-04-26_END -->

# 2026-04-25
<!-- DAILY_CHECKIN_2026-04-25_START -->

重新回顾EIP-8141，发现里面有很多源代码可以为我提供出设计思路
<!-- DAILY_CHECKIN_2026-04-25_END -->

# 2026-04-22
<!-- DAILY_CHECKIN_2026-04-22_START -->


打卡，回归正常
<!-- DAILY_CHECKIN_2026-04-22_END -->

# 2026-04-21
<!-- DAILY_CHECKIN_2026-04-21_START -->



打卡
<!-- DAILY_CHECKIN_2026-04-21_END -->

# 2026-04-20
<!-- DAILY_CHECKIN_2026-04-20_START -->




打卡
<!-- DAILY_CHECKIN_2026-04-20_END -->

# 2026-04-19
<!-- DAILY_CHECKIN_2026-04-19_START -->





打卡
<!-- DAILY_CHECKIN_2026-04-19_END -->

# 2026-04-18
<!-- DAILY_CHECKIN_2026-04-18_START -->






# ACDC 177 重点提要

## 1\. DevNet / Glamsterdam

-   `ePBS DevNet1` 已确认不再恢复。
    
-   `ePBS DevNet2` 不再单独推进。
    
-   下一步直接目标改为 `Glamsterdam DevNet0`。
    
-   `DevNet1` 的新增内容先不定，作为 interop 期间的 `stretch goals` 继续讨论。
    

## 2\. PR 5094

-   `5094` 会先合并。
    
-   主要原因是它解决了 `checkpoint sync` 的现实问题，并阻塞后续 DevNet。
    
-   当前方案虽然带来 execution requests 的重复数据，但会议决定先接受，避免继续增加实现复杂度。
    

## 3\. Execution Requests 重复问题

-   当前 execution requests 同时存在于 envelope 和 block 中，存在重复传播。
    
-   有人提出未来可优化为：
    
    1.  block 中只放 commitment
        
    2.  节点本地利用前一个 slot 的 payload 做展开
        
-   这项优化本次不进入发版路径，留待后续研究。
    

## 4\. 其他工程项

-   `5061` 需要基于 `5094` 更新，并尽快合并，以配合版本发布。
    
-   BAL 方向在 CL 侧预计主要只是：
    
    1.  payload 结构变化
        
    2.  engine API 调整
        
-   `8045` 的扩展方案本次未定，留到 interop 继续讨论。
    

## 5\. Hegota 提案

### EIP-7716

-   主题：相关性停机惩罚（Correlated Downtime Penalties）。
    
-   目标：更早惩罚集中化/相关性停机，提升去中心化激励。
    
-   当前结论：继续保留，建议补充 PoC 和更多参数研究。
    

### EIP-8205

-   主题：解决 validator deposit front-run 漏洞。
    
-   方案：引入可选的 preregistration 机制，先确认 withdrawal credentials，再存款。
    
-   当前结论：继续保留，后续再评估是否纳入 Hegota。
    

## 6\. Repricing 更新

-   Amsterdam repricing breakout 已处理一批开放问题。
    
-   下一版 state growth EIP 规范会更稳定。
    
-   state gas 收费与退款逻辑已调整为更贴近真实状态增长成本。
    

## 7\. 一句话总结

-   本次会议的核心方向是：先稳住 `5094` 和 `Glamsterdam DevNet0`，其他优化和新增提案先继续研究，不把当前发版路径变复杂。
<!-- DAILY_CHECKIN_2026-04-18_END -->

# 2026-04-16
<!-- DAILY_CHECKIN_2026-04-16_START -->









先打卡
<!-- DAILY_CHECKIN_2026-04-16_END -->

# 2026-04-14
<!-- DAILY_CHECKIN_2026-04-14_START -->










打卡
<!-- DAILY_CHECKIN_2026-04-14_END -->

# 2026-04-13
<!-- DAILY_CHECKIN_2026-04-13_START -->











### 主要亮点（逐节拆解）

1.  **性能与存储数据（核心卖点）**
    
    -   主网实测：**1.7 Gigagas/s** 处理速度，同时磁盘占用大幅降低至约 **240 GB**（快照下载仅 ~170 GB）。
        
    -   这些数据已在 Paradigm 自己的 Ethereum 主网节点 + Tempo 主网（他们的高性能测试链）上实时验证。
        
    -   第三方对比：ethPandaOps Lab Dashboard 显示 Reth 在块验证速度上的“胜率”领先（附历史图表）。
        
    -   基准测试硬件：AMD EPYC 4585PX CPU + 128 GB RAM + NVMe RAID SSD（高配服务器级别）。
        
    -   **实际操作**：官网提供快照下载 reth download -y && reth node，10 分钟即可跑起 Minimal 节点。
        
2.  **新架构核心：Pipelined Execution + Tiered Storage（管道执行 + 分层存储）**
    
    -   **并行状态计算优化**：引入 **Sparse Trie Cache**（跨区块持久化的内存状态 Trie 缓存），以前每块都要重建 Trie，现在状态根（state root）计算只需 **1-2ms**。代码位置：crates/engine/tree/src/tree/payload\_processor/sparse\_[trie.rs](http://trie.rs)。  
        **硬件状态计算优化** ：引入 **Sparse Trie Cache** （跨块持久化的内存状态 Trie 缓存），之前每块都要重建 Trie，现在状态根（state root）计算只需 **1-2ms** 。代码位置： crates/engine/tree/src/tree/payload\_processor/sparse\_[trie.rs](http://trie.rs) 。
        
    -   **减少数据冗余**：**Partial Proofs**（部分证明）技术，只拉取 Trie 中未缓存的唯一路径，证明请求数量减半，I/O 大幅下降。代码位置：crates/trie/trie/src/proof\_v2/[mod.rs](http://mod.rs)。
        
    -   **分层存储（Hot/Cold）**：
        
        -   只在 MDBX 存 hashed state，丢弃 plain state 表。  
            只在 MDBX 存散列状态，丢弃普通状态表。
            
        -   历史 account/storage changesets 迁移到 append-only 的 Static Files + RocksDB 索引。  
            历史帐户/存储变更集迁移到仅附加的静态文件 + RocksDB 索引。
            
        -   收益：块持久化时间从之前的 8.4s（Gigagas 块）降到 **400ms**（标准块仅 40ms），约 20 倍加速。
            
        -   额外福利：全节点 ↔ 归档节点可无缝切换，只需挂载对应 Static Files。
            
3.  **Minimal Mode + 快照系统**
    
    -   **Minimal 模式**：只保留验证所需的最小 hashed state + Trie 数据，主网节点磁盘 <300 GB。
        
    -   通过 pruning 配置自由选择额外数据（headers、tx、receipts、changesets 等）。
        
    -   快照工具全新模块化，支持并行下载、断点续传、自动生成配置，极大降低新手门槛。
        
4.  **后续路线**
    
    -   继续性能优化：AOT/JIT 执行（revmc 项目重启）、Glamsterdam 硬分叉带来的 Block Access Lists（BAL）实现**完整并行执行**。
        
    -   社区/招聘号召：加入 Telegram、GitHub、发邮件 [georgios@paradigm.xyz](mailto:georgios@paradigm.xyz)（招 Rust 工程师）。
        

**一句话总结文章**：Reth 2.0 把“节点即库”（node-as-a-library）的理念推向极致，重点解决状态根计算和存储 I/O 瓶颈，让 Reth 成为目前最快、最省的以太坊执行客户端之一。

### Reth 2.0 可能遇到的问题（基于发布笔记 + 架构分析）

Reth 2.0 是**5 天前**（2026.4.8）刚发布的 major version，目前社区反馈以正面为主（X 上多为发布庆祝和中文周会讨论），GitHub Issues 尚未出现大规模 bug 报告（因为太新）。但作为重大架构重构，以下是**潜在风险**（已结合官方 Breaking Changes 分析）：

1.  **迁移/升级风险（最需要注意）**
    
    -   Storage V2 成为**新节点默认**，V1 与 V2 **完全不兼容**（数据库格式不同）。
        
    -   现有 Reth 1.x 用户若想升级，可能需要**重新同步**（或手动迁移 DB）。官方暂未提供自动一键迁移工具，建议先备份再测试。
        
    -   SDK 用户受影响最大：大量 deprecated crate 被移除（reth-primitives → reth-ethereum-primitives、SerdeBincodeCompat 换 RLP 等），代码需重构。
        
2.  **新 Trie 机制的稳定性风险**
    
    -   Sparse Trie Cache + Partial Proofs + Proof V2 是核心创新，但属于全新代码路径。
        
    -   潜在问题：大块（Gigagas 级）、历史状态回放、边缘 case 下可能出现状态不一致或 panic。
        
    -   虽有 trie-debug 特性辅助调试，但生产环境大规模采用后才可能暴露。
        
3.  **资源消耗变化**
    
    -   内存占用可能上升（Sparse Trie Cache 是 in-memory 的跨块缓存）。
        
    -   Minimal Mode 省磁盘，但如果你后期需要历史数据，必须重新开启 pruning（操作门槛提高）。
        
4.  **Engine Backpressure 新特性  发动机背压新特性**
    
    -   引入 --engine.persistence-backpressure-threshold（默认 16）防止内存块缓冲爆炸。
        
    -   对 MEV 构建者、自定义 payload builder 可能造成延迟，需要调参适应。
        
5.  **其他潜在问题**
    
    -   **硬件依赖**：1.7 Ggas/s 是高配服务器数据，普通 SSD/CPU 可能达不到宣传峰值。
        
    -   **L2/自定义链兼容**：Reth 1.0 强调 L2 扩展性，2.0 重点性能，需验证是否影响特定 OP Stack 或自定义链。
        
    -   **快照信任**：依赖 [snapshots.reth.rs](http://snapshots.reth.rs)，若快照被污染或不完整会有风险（虽有 PGP 签名）。
        
    -   **早期采用风险**：任何 major release 初期都可能有小 bug，建议先在测试网跑 1-2 周再上主网。
        

总体来说，Paradigm 把 Reth 2.0 定位为“production-ready”，并已在自家节点跑起来。但**建议**：

-   先用 reth node --storage.v2（或新 CLI）测试；
    
-   关注 GitHub Issues 和 ethPandaOps Dashboard 最新数据；
    
-   社区反馈（如 Telegram）目前积极，但多观察一周。
<!-- DAILY_CHECKIN_2026-04-13_END -->

# 2026-04-12
<!-- DAILY_CHECKIN_2026-04-12_START -->












完成lifi黑客松
<!-- DAILY_CHECKIN_2026-04-12_END -->

# 2026-04-11
<!-- DAILY_CHECKIN_2026-04-11_START -->













**VOPS 和 EIP-8141：以太坊对抗状态膨胀的最真实现实主义路径**

说实话，以太坊现在真的卡在一个很现实的十字路口。2026年4月，链上交互和 DApp 生态已经炸了锅，“状态膨胀”不再是技术名词，而是每天跑节点、刷浏览器时都能实实在在摸到的麻烦。全节点几百 GB 的状态树，把普通用户、独立开发者甚至很多小节点运营商直接挡在门外。公共内存池慢慢被几家大基础设施悄悄垄断，去中心化就这样被硬件门槛一点点啃掉。

VOPS（Validity-Only Partial Statelessness，唯有效性部分无状态）是我觉得最务实、最接地气的应对办法。它没想着一步跳到“完全无状态”的那种浪漫理想，而是用最简单的工程思路，给以太坊争取一点喘息空间——节点不用重放整块历史的交易，只要大幅砍掉验证需要的数据，就能自己验证区块状态转换到底合不合法。这样，去中心化网络才不会被外面的硬件门槛和中心化力量彻底卡死。

**一、VOPS 的核心机制**

老的全节点活得像苦力，得把完整状态树、存储树、智能合约代码全塞硬盘里。VOPS 直接从底层把节点的存储和验证方式改掉了：对外部拥有账户（EOA），节点只用存四个字段——Address、Nonce、Balance、codeFlag。存储瘦到极致，普通家用 SSD 就够了。

它不用节点重放历史交易，而是直接靠 Verkle 树的多重证明，或者 zkVM 生成的 SNARK 证明，来确认状态转换合法。同时还能兼容内存池 Gossip 和 FOCIL 抗审查机制，再通过 AA-VOPS 往账户抽象那边延伸——智能合约账户额外缓存几个头部存储槽。根据现在测试的数据，整个方案存储成本能压到 10GB 左右（运气好还能更低），Raspberry Pi 那种家用小硬件又能重新当网络主力了。

这就是 VOPS 最实在的地方：它不是在妥协去中心化，而是用密码学把去中心化真正瘦身，让普通人也能跑得起来。

**二、VOPS 与 EIP-8141 的互相作用和影响**

VOPS 是实打实的节点运行方式和工程方案，EIP-8141（Frame Transaction，目前还在 Draft + CFI）则是给它发“合法身份证”的共识层和执行层标准。两者不是简单搭伙，而是互相推、互相拉、互相拖后腿的真实共生——EIP-8141 给 VOPS 划了条活下去的底线，VOPS 则把 EIP-8141 从纸面拉到真实压力测试里。

EIP-8141 对 VOPS 的作用很直接：它把“部分无状态”怎么玩写进了规则——节点不用存完整状态树，只吃区块头里的 State Diffs + Verkle Proofs 就能验证；Builder 必须把状态差异打包进区块头；还规定了 VERIFY frame 只能读有限范围（EOA 只读 nonce/balance/codeHash 加前面 N 个 storage slot）。这等于给 VOPS 发了主网通行证，10GB 存储底线有了协议背书，AA-VOPS 的头部缓存也变得可行。要是没有 EIP-8141，VOPS 节点就算跑起来，在 P2P 网络里也会被当成不靠谱的验证者直接踢掉。

但现实没那么美好：EIP-8141 同时把 VOPS 的自由度卡死了。它要求 VERIFY frame 必须走 APPROVE opcode，FOCIL 兼容性也有硬边界。VOPS 节点想真正参与抗审查，就得死守这些线。一旦社区把读取限制再收紧，VOPS 的存储优势就会被咬掉一块。

反过来，VOPS 对 EIP-8141 的推动更残酷：家用 10GB 轻节点一旦大规模上线，就是对 EIP-8141 最狠的压力测试。它会直接测出 VERIFY frame 在高 TPS 下的证明延迟、内存池 DoS 风险、FOCIL 列表到底能不能真生成。CPerezz 在 [ethresear.ch](http://ethresear.ch) 里已经说得很清楚：VOPS 才是让“部分无状态节点”真正守住公共内存池和 FOCIL 的关键。

真实的反哺总是带着痛：2026 年 3 月的测试已经摆在那里，如果 AA 采用率高，AA-VOPS 存储可能从 10GB 直接跳到 20-60GB，证明膨胀也会逼协议赶紧改 Gas 计量和 default code。

两者一起带来的深层影响：

1.  **路径彻底锁死**：以太坊 stateless 的路被钉死在“Verkle + 有限证明”上。The Verge 迁移只要卡壳，两边都会一起完蛋。
    
2.  **权力真正下放**：监控内存池和生成 FOCIL 列表的能力，从少数大节点手里交到普通家用节点手上，PBS 里的 Builder 终于要被广泛盯着了。
    
3.  **生态被迫重做**：钱包、DApp、前端得同时适配 State Diffs、frame 交易和轻量证明，逼出一个全新的“证明优先”开发方式。
    

风险也是绑在一起的：Verkle 共识分裂、EIP-4762 Gas 重定价引发的合约迁移潮、钱包适配跟不上——任何一边延期，另一边就得跟着遭殃，这就是真实的互相绑架。

**三、客观存在的挑战**

我不会在这儿粉饰太平。VOPS + EIP-8141 落地绝对不是轻松事儿，整个网络得经历一次真刀真枪的阵痛。而且现在（2026 年 4 月）两者都还只是“半成品”——VOPS 是 CPerezz 的研究提案，EIP-8141 刚到 CFI，离 Hegota 分叉至少还有 1-2 个周期。

最扎手的几块硬骨头：

-   **Verkle Tries 生死绑定**：没有高效多重证明，VOPS 直接跑不起来。The Verge 迁移工程量巨大，共识分裂风险极高。
    
-   **Gas 经济学的全面重构**：EIP-4762 要把 SLOAD/SSTORE 这些操作码的 Gas 按“证明验证”而不是磁盘读取重新算。这会直接砸到现有合约逻辑，带来真实的重入、Gas 耗尽风险。
    
-   **前端和生态的适配压力**：DApp、钱包得从单纯靠 RPC 改成处理 frame + proof，开发者跟用户体验都会集体遭罪。
    
-   **AA 采用率的不确定性**：AA 一旦爆发，存储成本和证明复杂度都会明显上升，VOPS 那句“10GB 极致”优势可能就被稀释掉。
    

这些不是纸上谈兵，是现在 [ethresear.ch](http://ethresear.ch) 和测试网里天天被反复讨论的真实问题。

**结语**

VOPS 和 EIP-8141 的组合，是以太坊在去中心化和状态膨胀这场拉锯战里，最现实主义的一条中间路。它放弃了完全无状态那种不切实际的幻想，用 10-60GB 的低存储成本，换回了家用节点对内存池的维护权和对 Builder 的抗审查监督权。这不是什么一劳永逸的救世主，而是以太坊底层协议演进里最重要、也最疼的一步。它会实实在在影响未来好几年 Web3 基础设施怎么开发、安全审计标准怎么定，甚至整个行业的硬件门槛。

2026 年 4 月的我，对这个东西保持谨慎乐观：它有戏，但必须靠真节点跑起来、真数据反馈、真社区迭代才能走通。只有普通人拿一台老电脑就能重新守住以太坊的去中心化，我们才真正配得上 Web3 这个名字。

这就是我对 VOPS 和 EIP-8141 最完整、最真实的看法。它不完美，但够实在、够及时，也值得我们认真推。因为以太坊的去中心化，从来不是靠一个 EIP 喊口号就能搞定，而是靠无数节点运营商、开发者、研究员在真实硬件和真实网络里反复试错、一步步守住的底线。希望社区继续往前冲，也希望更多人把节点跑起来——只有真正跑起来，我们才知道这条路到底行不行。
<!-- DAILY_CHECKIN_2026-04-11_END -->

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
