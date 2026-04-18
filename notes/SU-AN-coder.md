---
timezone: UTC+8
---

# AN_SU

**GitHub ID:** SU-AN-coder

**Telegram:** 

## Self-introduction

EPF 实习计划

## Notes

<!-- Content_START -->
# 2026-04-18
<!-- DAILY_CHECKIN_2026-04-18_START -->
# CL 客户端整体架构

## 一、结论

共识层客户端 ≠ 单个程序

真实运行的 PoS 节点 = **Beacon Node + Validator Client + Execution Client**

三者通过 **Beacon API** 和 **Engine API** 协作。

* * *

# 二、BN & VC

## 1\. Beacon Node（BN）

**负责链本身，不碰私钥**

-   P2P 网络（发现、同步、广播）
    
-   维护 Beacon State
    
-   执行 Fork Choice 选链头
    
-   处理区块、状态转移
    
-   同步、存库
    
-   对外提供 Beacon API
    
-   通过 Engine API 对接执行层
    

## 2\. Validator Client（VC）

**负责验证者行为，掌管密钥**

-   管理验证者密钥 /keystore
    
-   计算每个 slot/epoch 的职责（propose/attest/aggregate 等）
    
-   对区块、证明、同步委员会消息签名
    
-   Slashing Protection（防罚没）
    
-   通过 Beacon API 与 BN 通信
    

一句话分工：

**BN 管链，VC 管签；BN 不碰私钥，VC 不存全链。**

* * *

# 三、Beacon Node 内部模块

1.  **networking**
    
    发现节点、gossip、Req/Resp，所有共识数据入口。
    
2.  **sync**
    
    追链头：拉取缺失区块、状态、检查点。
    
3.  **fork choice**
    
    维护链视图，根据区块和证明计算 head。
    
4.  **state transition**
    
    处理区块、推进 slot/epoch、更新 BeaconState。
    
5.  **database/storage**
    
    持久化区块、状态、索引。
    
6.  **API server**
    
    对外提供 Beacon API，给 VC 和工具用。
    
7.  **execution bridge**
    
    通过 Engine API 与 EL 交互，获取 / 验证执行载荷。
    

* * *

# 四、Validator Client 内部模块

1.  **duty scheduler**
    
    按 slot/epoch 安排验证者任务。
    
2.  **key management / signing**
    
    密钥存储、签名、对接远程签名器。
    
3.  **slashing protection**
    
    防止双重签名、环绕签名，避免被罚。
    
4.  **beacon api client**
    
    从 BN 获取职责、区块、证明数据，提交签名结果。
    
5.  **builder/proposer flow**
    
    对接 Builder，实现 PBS 出块逻辑。
    

* * *

# 五、Engine API 到底在哪？

**不是给 DApp 用的接口，是 CL ↔ EL 的内部桥梁。**

出块关键路径：

1.  BN 到出块 slot
    
2.  BN 通过 Engine API 向 EL 请求 execution payload
    
3.  EL 构造并返回 payload
    
4.  BN 组装 BeaconBlock
    
5.  VC 签名
    
6.  BN 广播区块
    

Engine API 是现代 proposer 流程的**核心环节**，不是外挂。

* * *

# 六、两条最关键数据流

## 1\. 区块从生成到广播

1.  BN 计算本 slot 应由本验证者出块
    
2.  BN 从 Fork Choice 取当前 head & state
    
3.  BN 向 EL 请求 payload
    
4.  BN 组装完整 Beacon Block
    
5.  VC 签名
    
6.  BN gossip 全网
    
7.  全网节点执行 state transition + 更新 fork choice
    

## 2\. 证明（Attestation）流程

1.  BN 给 VC 分配 attestation duty
    
2.  VC 在指定 slot 签名证明
    
3.  提交回 BN
    
4.  BN 广播证明
    
5.  证明进入 fork choice store，影响链头选择
    

* * *

# 七、最容易混淆的 4 个点

1.  **BN 不持有验证者私钥**
    
    密钥安全边界在 VC。
    
2.  **VC 不是小挂件**
    
    它是安全核心，负责罚没保护和签名。
    
3.  **Fork Choice ≠ State Transition**
    
    Fork Choice：选哪条链
    
    State Transition：执行这条链上的区块
    
4.  **Engine API 不是公网 RPC**
    
    只在 BN ↔ EL 之间使用，和 Beacon API、ETH JSON-RPC 完全无关。
    

* * *

-   各客户端在 fork choice store、state cache、DB 上的差异
    
-   PBS（builder/proposer separation）如何改变 VC 职责
    
-   远程签名器 + VC 架构下的延迟与安全风险
    

* * *

# 八、总结

CL 客户端 =

**Beacon Node（管链、网络、状态、选头） +**

**Validator Client（管密钥、职责、签名、防罚） +**

**Engine API 桥接执行层**

围绕 slot 时钟运转，构成完整的以太坊 PoS 节点。
<!-- DAILY_CHECKIN_2026-04-18_END -->

# 2026-04-16
<!-- DAILY_CHECKIN_2026-04-16_START -->

# 共识层网络通信

执行层与共识层各有独立 P2P 网络，传播对象、协议分层完全不同。共识层需要传播区块、attestation、aggregate、slashing、voluntary exit、sync committee update 等对象，同时满足时效性、分叉兼容、验证成本与网络鲁棒性。共识层网络是围绕 Beacon chain 对象构建的**分层网络协议**，而非简单广播系统。

今天核心按三类域理解：discovery 域、gossip 域、Req/Resp 域。

* * *

## 机制

### 1\. 共识层网络基于 libp2p 构建

共识层网络规范是基于 libp2p 的互操作标准，底层传输、连接管理、消息域遵循统一约束。

与执行层关键区别：

-   EL：独立 DevP2P 网络，传播交易与执行区块
    
-   CL：独立 libp2p 网络，传播 Beacon blocks、attestations 等
    

这也是现代以太坊节点必须同时运行 EL 与 CL 客户端的原因之一。

### 2\. discovery 域：节点发现与 peer 管理

解决核心问题：

-   如何找到有效 peer
    
-   如何识别对方 fork 版本与 fork digest
    
-   如何稳定维护 peer 集合
    

该层不直接传播业务数据，是 gossip 与 Req/Resp 的基础前提。

-   技术：discv5（UDP）
    
-   节点身份：ENR（Ethereum Node Record）
    
-   关键信息：fork digest、attnets、ip/port、secp256k1 公钥
    

### 3\. gossip 域：高时效对象的快速广播

处理需要低延迟扩散的对象：

-   beacon\_block
    
-   beacon\_aggregate\_and\_proof
    
-   beacon\_attestation\_{subnet\_id}
    
-   voluntary\_exit、proposer\_slashing、attester\_slashing
    

设计要点：

-   按 topic 隔离不同对象
    
-   先做 gossip validation，再决定是否转发
    
-   平衡传播速度与网络安全
    

### 4\. Req/Resp 域：按需拉取与同步补全

适合历史数据、精准查询、同步补数场景：

-   BeaconBlocksByRange：按 slot 范围拉取区块
    
-   BeaconBlocksByRoot：按根哈希拉取区块
    
-   Status：链视图握手对齐
    
-   Goodbye：断开原因
    
-   Ping/GetMetaData：节点存活与元数据
    

Bellatrix 关键规则：

执行 payload 无效但共识层合法的区块，不应被扣分或断连，以兼容 optimistic 同步节点。

### 5\. fork digest：跨分叉兼容的核心标识

fork digest 由 genesis\_validators\_root 与当前 epoch 计算得出，作用：

-   隔离不同分叉 / 不同链的网络消息
    
-   确保 gossip topic 与 Req/Resp 上下文正确
    
-   让网络协议可安全跨硬分叉演进
    

* * *

## 流程

### 新区块传播最小路径

1.  Proposer 生成并签名区块
    
2.  本地 beacon node 完成基础验证
    
3.  通过 beacon\_block topic 广播
    
4.  接收 peer 执行 gossip validation
    
5.  合法区块继续转发
    
6.  进入本地 fork choice 与状态转换
    
7.  缺失父块 / 历史对象时，通过 Req/Resp 拉取
    

### 新节点同步最小路径

1.  discv5 发现 peer
    
2.  建立 libp2p 连接（Noise 加密、mplex 多路复用）
    
3.  Status 握手对齐链视图（fork digest、finalized checkpoint）
    
4.  Req/Resp 拉取历史区块
    
5.  订阅 gossip 接收头部实时更新
    

* * *

## 边界与易混点

1.  gossip 转发 ≠ 完整导入验证
    
    gossip 只做轻量检查，完整验证在状态转换阶段。
    
2.  Req/Resp ≠ 对外 RPC
    
    它是 P2P 内部协议，不是 Beacon API。
    
3.  CL 网络 ≠ EL 网络
    
    对象、验证、协议栈完全不同，不可混用直觉。
    
4.  无效 execution payload ≠ 恶意节点
    
    optimistic 同步下允许传播，避免网络分裂。
    

* * *

共识层网络同时解决三大问题：

-   discovery：如何入网找节点
    
-   gossip：高时效数据快速扩散
    
-   Req/Resp：按需拉取与同步
    

叠加 fork digest、optimistic 兼容、跨分叉升级，形成一套完整、独立、可演进的协议系统。

* * *

-   各客户端 gossip validation 与 peer scoring 策略差异
    
-   轻客户端 topic 与全节点同步的兼容方式
    
-   QUIC 在主流客户端的实际落地与网络影响
    

* * *

## 小结

共识层网络不是简单的区块广播，而是三层协作的协议系统：

discovery 负责入网、gossip 负责实时广播、Req/Resp 负责同步补数。理解这套结构，才能衔接后续 weak subjectivity 与同步安全机制。
<!-- DAILY_CHECKIN_2026-04-16_END -->

# 2026-04-15
<!-- DAILY_CHECKIN_2026-04-15_START -->


# Beacon API 与验证者交互

执行层用 JSON‑RPC 对接节点，共识层则靠 **Beacon API** 作为标准服务接口。不理解 Beacon API，共识层对象就始终停留在规范抽象里。它是把 beacon node 变成可查询、可集成、可与验证者客户端协作的**接口边界**。

今天核心看清三件事：Beacon API 为谁服务、暴露哪些共识对象、如何连接 validator client 与节点。

* * *

## 机制

### 1\. Beacon API 的定位

Beacon API 是由 **beacon node 暴露的标准化 RESTful API**，目标是实现不同共识客户端之间的互操作性。

关键点：

-   属于**共识层标准接口**，不是某客户端私有接口
    
-   基于 HTTP/REST，返回 JSON
    
-   不宜直接暴露公网，部分高成本端点存在 DoS 风险
    
-   是外部工具、监控、validator client 访问共识层的主要入口
    

### 2\. Beacon node 与 validator client 的分工边界

Beacon API 之所以重要，因为现代节点已经明确拆分：

**Beacon Node（BN）**

-   维护 BeaconState
    
-   处理 P2P、区块广播、同步
    
-   运行 fork choice
    
-   暴露 Beacon API
    
-   **不持有验证者密钥**
    

**Validator Client（VC）**

-   管理私钥、签名
    
-   获取出块 / 见证职责（duty）
    
-   构造并签名区块、attestation
    
-   通过 Beacon API 提交数据
    

Beacon API 是两者之间**标准、解耦的协作通道**。

### 3\. Beacon API 暴露的核心共识对象

它不面向交易 / 合约，而是面向共识层实体：

-   BeaconState
    
-   BeaconBlock / BeaconBlockHeader
    
-   Validator 列表与状态
    
-   Attestation
    
-   Checkpoint（justified/finalized）
    
-   SyncCommittee
    
-   节点同步、peer、运行状态
    

### 4\. Beacon API 能力分层

按用途可分为四类：

1.  **基础链信息**
    
    -   创世信息、区块头、区块内容、slot/epoch 数据
        
2.  **状态与验证者查询**
    
    -   BeaconState 切片、validator 信息、最终性检查点
        
3.  **节点运维状态**
    
    -   同步状态、peer 数量、节点信息
        
4.  **Validator 工作流接口**
    
    -   获取职责、请求区块模板、提交签名区块 /attestation、聚合
        

这一层是验证者能正常出块、投票的关键。

* * *

## 流程

### Validator Client 基于 Beacon API 的最小工作流

1.  VC 向 BN 请求当前 epoch 的职责（duty）
    
2.  若为 attester：获取 attestation data → 签名 → 提交回 BN
    
3.  若为 proposer：请求区块模板 → BN 从 EL 获取 execution payload → VC 签名 → 发布区块
    
4.  所有操作都通过 Beacon API 完成
    

### 共识监控 / 浏览器的只读路径

1.  调用 `/eth/v1/beacon/headers`、`/states/head`、`/finality` 等端点
    
2.  获取 head、finalized checkpoint、validator 数量、同步状态
    
3.  展示链健康度与共识进展
    

这也是为什么链浏览器、节点仪表盘必须对接 Beacon API。

* * *

## 边界与易混点

1.  **Beacon API ≠ JSON‑RPC**
    
    前者 REST + 共识对象；后者 RPC + 执行层对象。
    
2.  **Beacon API 不只是查询**
    
    它支撑验证者核心工作流：获取 duty、出块、提交投票。
    
3.  **Beacon node 不碰密钥**
    
    密钥、签名全部在 validator client 完成。
    
4.  **公网暴露 Beacon API 危险**
    
    部分端点会触发 heavy processing，易被 DoS。
    

* * *

Beacon API 是共识层的**标准服务面 + 操作面**：

-   对外：提供链状态、区块、验证者、最终性查询
    
-   对内：作为 BN ↔ VC 的标准协作接口
    

它让共识层从客户端内部状态机，变成**可接入、可集成、可互操作**的开放系统。

* * *

-   不同客户端对 Beacon API 的支持是否存在差异？
    
-   validator-flow 哪些接口最容易因时序 / 同步问题失败？
    
-   跨硬分叉时，Beacon API 如何保持兼容与版本管理？
    

* * *

## 小结

Beacon API 不是附加文档，而是**以太坊共识层的互操作基石**。它让 beacon node 从黑盒变成可查询、可协作的共识服务。理解它，才能把 “内部状态机” 和 “对外接口” 彻底分开。
<!-- DAILY_CHECKIN_2026-04-15_END -->

# 2026-04-14
<!-- DAILY_CHECKIN_2026-04-14_START -->



# 共识层概览与规范主线

执行层负责交易执行与状态更新，但无法让全网对区块头、主链、不可逆历史达成一致。共识层的核心作用，是将这些问题从执行层剥离，用独立状态机维护。进入第二周，需先建立整体图景：共识层维护的对象、运行的流程、推进链的规则。

应用层常将以太坊 PoS 简化为 “验证者出块和投票”，但协议层更复杂。共识层需处理验证者注册、出块、见证、奖惩，维护 fork choice、justification、finalization、sync committees，保障跨分叉兼容，并通过 Engine API 与执行层协作。共识层本质是维护 Beacon state 的协议状态机，而非简单投票系统。

* * *

## 机制

### 1\. 共识层在现代以太坊中的位置

2022 年 9 月 15 日 The Merge 完成后，以太坊完整节点由三部分组成：

-   执行客户端（execution client）
    
-   共识客户端（consensus client）
    
-   验证者客户端（validator client，参与提议与证明时需要）
    

共识层负责：

-   保持节点与 Beacon chain 视图同步
    
-   运行 fork choice
    
-   处理区块与 attestation gossip
    
-   跟踪 justification 与 finalization
    
-   为验证者分配并执行 proposer/attester 职责
    
-   通过 Engine API 驱动执行层构造或验证 execution payload
    

两层维护的状态完全不同：

-   执行层：world state、交易池、收据、执行相关区块对象
    
-   共识层：Beacon state、验证者集合、投票记录、检查点、fork choice store
    

### 2\. Beacon chain 是共识层主体，而非第二条链

Beacon Chain 于 2020 年上线，最初为独立 PoS 链；The Merge 后，成为以太坊共识逻辑本身。执行层提供交易与状态转换，共识层通过 Beacon chain 规则组织区块、管理验证者、选择链头、提供最终性。

共识层关键实体：

-   BeaconState
    
-   BeaconBlock
    
-   BeaconBlockBody
    
-   Validator
    
-   Attestation
    
-   Checkpoint
    
-   SyncCommittee
    

这些实体围绕 slot、epoch 两级时间结构持续更新。

### 3\. slot、epoch 与验证者职责

共识层基础时间单位：

-   slot：单个时间窗口，每 12 秒一个，可产出一个区块
    
-   epoch：由 32 个 slot 组成，是规则结算边界
    

每个 slot：

-   选出 1 个 proposer 提出区块
    
-   多个委员会的 attester 完成证明
    

attestation 包含：

-   对 beacon block 头部的引用
    
-   对 source/target checkpoint 的 Casper FFG 投票
    
-   委员会位置相关聚合信息
    

epoch 边界处理：

-   justification/finalization 推进
    
-   奖励与惩罚结算
    
-   验证者激活 / 退出队列推进
    
-   sync 委员会轮换
    

共识层以 slot、epoch 双节拍，同时运行即时链头选择与周期性状态结算。

### 4\. Gasper：LMD-GHOST + Casper FFG

以太坊 PoS 核心为 Gasper，融合两个机制：

-   LMD-GHOST：负责头部选择（fork choice），确定当前应扩展的链头
    
-   Casper FFG：负责最终性（finality），确定不可逆的历史检查点
    

协议层明确区分三类状态：

-   head：当前最可能被继续扩展的链头
    
-   justified checkpoint：获得足够支持的检查点
    
-   finalized checkpoint：协议假设下不可逆的检查点
    

### 5\. 共识层规范：可执行规范（executable specs）

以太坊 consensus-specs 仓库采用可执行参考实现形式，按分叉阶段组织规则，主线包括：

-   Phase 0
    
-   Altair
    
-   Bellatrix
    
-   Capella
    
-   Deneb
    
-   Electra
    

规范定义内容：

-   常量与基础类型
    
-   Beacon state 对象
    
-   区块处理函数
    
-   epoch processing 函数
    
-   fork choice 逻辑
    
-   weak subjectivity、networking 等外围规范
    

共识层规范是完整软件接口体系，而非单篇白皮书描述。

* * *

## 流程

### 一个 slot 内部最小流程

1.  时钟推进至新 slot
    
2.  fork choice store 更新时间视图
    
3.  节点为 proposer 时，CL 向 EL 请求执行内容，构造 beacon block
    
4.  其他验证者对所见 head 发起 attestation
    
5.  新 block 与 attestation 进入本地处理，重新计算 fork choice
    
6.  跨 epoch 边界时，执行 epoch processing，完成奖惩与 checkpoint 推进
    

共识层围绕时间、投票、链头选择、状态推进持续运行，而非仅存储区块。

* * *

## 边界与易混点

1.  共识层不执行交易：Beacon block 含执行内容，但合约执行与状态更新由 EL 完成，CL 仅维护共识状态
    
2.  finality≠当前 head：head 可能重组，finalized checkpoint 具备更强安全性
    
3.  validator client≠beacon node：beacon node 负责共识协议，validator client 持有密钥、执行提议与证明
    
4.  共识层规范非单一文档：规则随分叉叠加演化，需区分不同版本差异
    

* * *

## 当前

共识层是以太坊独立状态机，负责链增长、历史最终性、验证者参与，并非执行层附属组件。Beacon state、slot/epoch 节拍、fork choice、FFG 最终性、validator duties 构成其核心骨架。

* * *

-   Beacon state 的对象结构与关键字段
    
-   fork choice store 内部状态与 Beacon state 的分工
    
-   不同分叉中，共识规范的对象层与流程层变化
    

* * *

共识层以 Beacon state 为核心，通过 slot/epoch 推进状态，LMD-GHOST 选择 head，Casper FFG 提供最终性，规范仓库持续演进规则。这一整体图景，是理解 SSZ、Beacon API、网络、同步、客户端实现的基础。
<!-- DAILY_CHECKIN_2026-04-14_END -->

# 2026-04-12
<!-- DAILY_CHECKIN_2026-04-12_START -->




# EOF、预编译合约与区块构建

如果只了解执行层当前的工作方式，很容易误以为它已经定型。EOF、预编译合约、区块构建从三个方向证明：执行层仍在持续演进 —— 对象格式在重整、关键原语边界在调整、区块构建分工与激励结构在升级。执行层是活跃的问题空间，而非封闭的底层规范。

* * *

## 核心机制

### 1\. EOF：EVM 对象格式为什么要重做

传统 EVM bytecode 长期运行后存在历史包袱：

-   代码与数据边界不清晰
    
-   控制流分析不规整
    
-   合法性检查滞后
    
-   工具链与客户端静态分析不友好
    

EOF（EVM Object Format）不重写 EVM，只重新组织对象封装，核心目标：

-   code 与 data 分离
    
-   部署阶段做更严格代码校验
    
-   让控制流与代码结构更易分析
    
-   为新指令、新语义预留扩展空间
    

EOF 的价值不在新增用户功能，而在于让执行环境更易验证、扩展、被工具链理解，是协议层长期的底层结构优化。

### 2\. EOF 是执行层内部治理的一部分

EOF 解决老系统的兼容性负担：历史灵活度逐渐变成升级成本，对象格式模糊区会抬高客户端实现、形式化验证、编译器优化的成本。

EOF 不是外挂包装，而是重新定义**代码对象如何进入协议**，是执行层对内部接口的长期整理。

### 3\. 预编译合约：为什么有些能力不走普通 bytecode

EVM 虽是通用执行环境，但密码学运算用普通 opcode 实现，成本与性能不达标。

执行层在固定地址部署预编译合约，特点：

-   地址固定
    
-   外观像普通合约
    
-   调用方式接近 message call
    
-   执行由客户端内建实现，不运行普通 EVM 字节码
    

经典预编译合约：

-   ecrecover
    
-   sha256
    
-   ripemd160
    
-   identity
    

后续硬分叉持续新增椭圆曲线、配对运算等密码学能力。

预编译体现执行层原则：通用执行不代表所有高价值原语都走底层抽象，少量关键能力提供协议级高性能通道，保障系统实用性。

### 4\. precompile 的边界问题

预编译不只是 “更快”，更是在划定执行层支持边界。新增一个 precompile 需明确：

-   哪类原语值得进入协议层
    
-   如何合理定价 Gas
    
-   客户端能否稳定正确实现
    

预编译并非越多越好，会降低操作成本，但增加协议面与实现负担。

* * *

## 关键流程

### 1\. block building：从 txpool 到 execution payload

现代以太坊中，CL 提议 beacon block，执行内容来自 EL 构造的 execution payload。

最小构建流程：

1.  执行节点维护本地 txpool
    
2.  按 nonce、费用、依赖、本地策略筛选交易
    
3.  顺序模拟执行，校验有效性与收益、Gas 影响
    
4.  生成区块体、收据、状态更新与对应根
    
5.  封装为 execution payload 进入共识流程
    

区块构建的核心复杂度：

-   交易选择顺序
    
-   最优价值交易组合
    
-   失败、依赖、替换交易处理
    
-   收益、合法性、时间窗口权衡
    

区块构建是执行层的高价值决策过程。

### 2\. proposer /builder/validator 的角色边界

-   builder：专注构造高价值 payload
    
-   proposer：共识流程中负责提议
    
-   validator：CL 中负责证明、提议、投票
    

构建 payload 与 CL 提议区块的角色不再天然等同，排序价值差异推动构建端专业化。

### 3\. PBS 与 MEV 的问题背景

MEV 表明交易排序非中性，排序差异影响套利、清算、夹子等价值提取，让区块构建成为激烈竞争的专业环节。

PBS（Proposer-Builder Separation）思路：

-   允许专业 builder 构造最优 payload
    
-   proposer 无需承担全部排序与搜索工作
    
-   缓解 MEV 带来的中心化与激励扭曲
    

PBS 不只是角色拆分，是围绕区块供应链重新分配职责与风险的协议 / 中间件方向。

* * *

## 边界与易混点

1.  EOF 不是 “新 EVM”：只重整对象格式与验证边界，不推翻执行模型
    
2.  预编译合约不是普通合约库：调用方式一致，但执行依赖客户端内建实现
    
3.  block building 不等于 CL 提议流程：EL 构造 payload，CL 提议并确认区块
    
4.  MEV 不是纯应用层现象：直接影响区块构建、builder 专业化、协议分工，是执行层与共识层共同问题
    

* * *

## 当前理解

执行层演进分内外两条线：

-   向内：EOF 整理对象格式
    
-   向外：precompile 划定协议原语边界，block building / PBS 重组角色与激励
    

执行层不是封闭黑盒，是持续打磨的活跃系统。

* * *

-   EOF 落地后对客户端实现与编译器输出的影响
    
-   precompile 新增标准如何避免协议面无限扩张
    
-   builder、PBS、长期协议内化方案的未来职责边界
    

* * *

## 小结

执行层并非已完成的基础层，仍在快速演进：EOF 优化对象格式，precompile 调整原语支持，block building 与 PBS 重构分工与激励。它始终是协议研究与实现的核心区域。
<!-- DAILY_CHECKIN_2026-04-12_END -->

# 2026-04-10
<!-- DAILY_CHECKIN_2026-04-10_START -->





# 交易、数据结构与 RLP

EVM 说明了状态怎样被执行改变，但还缺少关键部分：输入如何表示、执行结果如何保存、复杂状态如何承诺验证、底层对象以什么编码在网络间传输。交易、数据结构与 RLP 共同构成执行层最核心的对象链。

今天要理清完整路径：raw transaction -> 节点验证 -> txpool -> inclusion -> execution -> receipt -> trie root update，明确交易字段、区块头根、账户树、收据和 RLP 的位置。

* * *

## 机制

### 1\. 交易是状态变更请求，不是状态变更结果

交易是外部账户签名发出的**状态变更请求**，并非状态已改变，只是请求网络推进状态机。

执行层交易核心字段：

-   nonce：保证交易顺序、防止重放
    
-   gas limit：交易允许消耗的执行上限
    
-   费用字段：参与区块空间竞争
    
-   to：区分转账、调用或创建合约
    
-   value：转账金额
    
-   data：合约调用输入
    
-   签名字段：验证授权来源
    

交易是精确定义的协议对象，而非抽象操作。

### 2\. 交易类型的演进：从 legacy 到 typed transaction

早期为 legacy transaction，后续出现 typed transaction envelope，在兼容基础上支持新类型。

作用：

-   引入访问列表
    
-   支持 EIP-1559 费用机制
    
-   承载 blob 相关新型交易
    

交易是执行层面向未来升级的输入接口，typed transaction 为协议扩展提供清晰边界。

### 3\. 区块头中的根：复杂对象的压缩承诺

交易进入区块后，区块头通过三个根完成复杂对象承诺：

-   stateRoot：执行后 world state 结果
    
-   transactionsRoot：本区块交易序列
    
-   receiptsRoot：本区块交易执行收据
    

区块头是密码学承诺表，节点导入区块时需验证区块体、执行结果与根一致。

### 4\. account trie 与 storage trie 的层级关系

stateRoot 对应两层 Trie 结构：

1.  全局 account trie：地址映射到账户对象，包含 nonce、balance、storageRoot、codeHash
    
2.  合约账户 storage trie：合约存储写入先进入该 trie，根通过 storageRoot 挂接账户对象
    

层级路径：storage slot -> storage trie -> account object -> account trie -> stateRoot

局部存储写入最终会影响全局状态根。

### 5\. receipt、logs、Bloom：执行痕迹如何被保存

区块执行后生成 receipt，是执行层正式痕迹，包含：

-   交易执行成功 / 失败信息
    
-   累积 gas 使用
    
-   logs：上层应用核心依赖的可观察对象
    
-   Bloom：日志快速预筛
    

receiptsRoot 承诺执行记录，stateRoot 承诺执行后状态，二者同等重要。

### 6\. RLP：执行层历史对象的底层表示规则

RLP 是为字节串和列表提供**统一、确定、可递归**的序列化方式，非高级类型系统。

适用场景：

-   交易编码
    
-   账户对象编码
    
-   Trie 节点编码
    
-   部分网络消息对象
    

优势：规则简单、结果确定、适配嵌套结构

限制：不支持复杂模式表达，无 SSZ 的结构化与 Merkle 化属性，但仍是执行层历史对象基础语言。

* * *

## 流程

### raw transaction 到 trie root update 的最小链路

1.  用户构造并签名原始交易
    
2.  节点验证签名、格式、nonce、余额、费用约束
    
3.  交易进入 txpool，按发送者顺序与费用排序
    
4.  构建区块 / 执行 payload 时，从 txpool 选取交易
    
5.  执行客户端逐笔执行交易，调用 EVM 完成状态转换
    
6.  每笔交易生成 receipt 与 logs
    
7.  区块执行完毕，计算 stateRoot、transactionsRoot、receiptsRoot
    
8.  区块头承诺执行结果
    

该链路将交易、执行引擎、收据、Trie 承诺形成闭环。

* * *

## 边界与易混点

1.  transaction 不等于 message call：交易是外部顶层请求，message call 是合约间内部调用
    
2.  data 不只是备注字段：合约调用中决定函数选择器与参数的核心载体
    
3.  receipt 不等于最终状态：receipt 记录执行痕迹，最终状态体现在 world state 与 stateRoot
    
4.  RLP 不是唯一编码：执行层历史对象用 RLP，共识层多用 SSZ，新对象格式持续演进
    

* * *

交易、receipt、Trie、RLP 并非分散知识点，共同回答执行层对象**如何输入、执行、保存、承诺**。沿 raw transaction 到 root update 路径，所有对象位置都会清晰对齐。

* * *

## 未解

-   receipt 中 Bloom 的具体使用边界与局限
    
-   typed transaction 扩展对协议对象边界的影响
    
-   执行层状态承诺结构演进对 RLP 和 Trie 理解的改变
    

* * *

## 小结

执行层对象链完整闭环：交易是输入，EVM 是执行机，receipt 是执行痕迹，Trie 根是结果承诺，RLP 是底层表示方式。后续学习 DevP2P 和 JSON-RPC，可清晰理解对象在网络与应用间的流动。
<!-- DAILY_CHECKIN_2026-04-10_END -->

# 2026-04-09
<!-- DAILY_CHECKIN_2026-04-09_START -->






# EVM 作为状态转换引擎

应用层常说 “EVM 负责执行智能合约”，但这不足以说明它在协议中的真实位置。核心要弄清楚：以太坊为什么需要这台虚拟机，它执行什么，以及和 world state、Gas、交易、调用语义的关系。

只把 EVM 看作合约运行时，容易只关注语言与开发体验；协议层更关心它如何把一笔交易和前状态转换成全网公认的后状态。这决定 EVM 的核心不是功能丰富，而是确定性、可重复、可多客户端实现。

* * *

## 机制

### 1\. EVM 的协议位置：状态转换函数的执行器

执行层最核心的工作，是把前状态 S 在输入下转为后状态 S'，输入通常是区块中的交易序列。EVM 让这套状态转换通过确定性执行机逐步完成。

EVM 回答三个问题：

-   交易触发的代码逻辑由谁执行
    
-   执行过程中资源如何计量
    
-   执行结果如何影响账户、存储和收据
    

EVM 的核心身份是**状态转换规则的执行载体**，而非可编程平台。

### 2\. EVM 是栈机，基本计算单元是 256-bit word

EVM 是栈式机器，stack 最大深度 1024，基本元素是 256-bit word。

-   栈机模型让指令语义更容易形式化与多实现复现
    
-   256-bit 宽度适配哈希、签名、模运算等密码学对象
    

EVM 不追求通用 CPU 式效率，而是选择适合确定性复现的计算模型。

### 3\. stack /memory/storage：三种完全不同的状态层次

stack

即时运算区，多数 opcode 靠压栈、弹栈、计算完成，生命周期最短，只服务当前指令计算。

memory

一次执行上下文里的临时线性字节数组，用于组装参数、保存中间结果、构造返回值、搬运 calldata/returndata。执行结束不持久，但扩张消耗 gas。

storage

属于账户对象的持久化状态，挂在账户 storageRoot 下。SLOAD/SSTORE 操作的是链上状态本身，而非临时空间。

三者层级：

-   stack：指令级临时运算
    
-   memory：调用级临时状态
    
-   storage：账户级持久状态
    

### 4\. opcode 不是语言特性，而是协议级执行语义

Solidity、Vyper 最终编译为 EVM 字节码，由 opcode 组成。协议要求多客户端对齐的是 opcode 行为，而非高级语言语法。

opcode 主要类型：

-   算术与位运算：ADD、SUB、MUL、AND 等
    
-   栈操作：PUSH、POP、DUP、SWAP
    
-   环境读取：ADDRESS、CALLER、CALLVALUE、CHAINID
    
-   存储访问：SLOAD、SSTORE
    
-   内存与复制：MLOAD、MSTORE、CALLDATACOPY
    
-   控制流：JUMP、JUMPI、STOP
    
-   调用相关：CALL、DELEGATECALL、STATICCALL
    
-   合约创建：CREATE、CREATE2
    

EVM 是执行 opcode 规则的机器，语言只是入口。

### 5\. call frame 与 execution context

EVM 执行不是平铺指令流，每次 message call 或 contract creation 都会生成新执行上下文（call frame）。

执行上下文包含：

-   当前执行账户地址
    
-   调用者地址
    
-   调用附带 value
    
-   当前可用 gas
    
-   当前 calldata
    
-   当前 memory/stack 视图
    

合约调用另一合约是**上下文切换**，而非普通函数跳转，这是理解三种调用差异的前提。

### 6\. revert、return data 与执行失败语义

EVM 不是简单成功 / 崩溃，REVERT 提供可回滚并携带错误数据的路径。

-   RETURN：带回执行结果数据
    
-   REVERT：回滚当前 frame 状态变更，返回错误数据
    
-   returndata 可被上层调用者读取
    

应用层看到 require/revert message，协议层看到 call frame、状态回滚、returndata 处理。

* * *

## 流程

### 1\. 一次 message call 的最小执行路径

1.  交易导入执行环境，确定 sender、to、gas、value、data
    
2.  to 指向合约账户时，创建新 execution context
    
3.  calldata 放入上下文，初始化 stack/memory
    
4.  EVM 从代码程序计数器起点逐条执行 opcode
    
5.  执行中可读写存储、扩展内存、发起外部调用、写日志
    
6.  SSTORE 触发时修改账户持久化状态
    
7.  执行结束返回数据或 revert 数据，结算 gas
    
8.  结果提交到外层状态转换，影响 receipt 与状态根
    

核心是**执行上下文如何建立、消耗资源、产生状态与结果**。

### 2\. contract creation 与 message call 的区别

message call：

-   目标为已存在账户
    
-   执行现有代码
    
-   可修改当前或其他账户状态
    

contract creation：

-   无现成目标代码
    
-   先执行 creation code
    
-   返回值成为新账户 runtime code
    
-   新地址由 sender+nonce 或 CREATE2 导出
    

创建合约是独立的状态转换路径。

* * *

## 边界与易混点

1.  EVM 不等于 Solidity：协议绑定字节码与 opcode 语义，而非源语言
    
2.  memory 不等于 storage：临时空间 vs 长期状态，gas 成本与生命周期完全不同
    
3.  外部调用不等于普通函数调用：涉及上下文切换、gas 传递、状态回滚、权限边界
    
4.  revert 不等于节点错误：revert 是协议内建执行语义，非客户端实现错误
    

* * *

EVM 的核心作用是把**交易请求转为协议承认的状态变化**。栈机模型、256-bit word、三层状态、call frame、revert 语义都围绕这个目标。从状态转换函数看待 EVM，细节才会清晰。

* * *

## 未解

-   CALL、DELEGATECALL、STATICCALL 在权限与安全上的细分差异
    
-   部分历史 opcode 与 gas 定价保留至今的原因
    
-   EOF 新对象格式对代码布局与控制流的改变
    

* * *

## 小结

EVM 是执行层承载状态转换的核心机器。stack/memory/storage 定义状态层次，opcode 定义执行语义，call frame 定义上下文边界，Gas 定义资源约束。它是整个执行层大图的中心。
<!-- DAILY_CHECKIN_2026-04-09_END -->

# 2026-04-08
<!-- DAILY_CHECKIN_2026-04-08_START -->







# 执行层规范与客户端架构

进入协议层时，很容易把某个客户端的行为直接当成协议本身，或只看抽象规范却不懂真实软件落地。执行层规范和客户端架构正卡在这条边界上，今天要把 “规则写在哪里” 和 “规则由谁实现、怎样协作” 拆开。

边界不清会带来两种偏差：把实现细节误当成协议标准，或把协议标准看成悬空理论、不知道如何变成运行节点。

* * *

## 机制

### 1\. 执行层规则有多个来源，但职责不同

执行层规则并非来自单一文件或仓库，由以下几类来源构成：

-   Yellow Paper：给出 EVM 与状态转换偏形式化的描述
    
-   EIPs：描述新增或修改的协议规则，是单项变化的标准载体
    
-   execution-specs：按硬分叉整理执行层行为定义
    
-   execution-apis：整理执行层对外和跨层接口规范
    
-   客户端实现：把上述规则落实为软件系统
    

关键是区分规范层级与实现层级：

-   EIP 不是客户端补丁说明，是协议变更提案
    
-   execution-specs 不是某客户端文档，是多客户端对齐执行行为的规则归纳
    
-   execution-apis 不是应用 SDK，是执行层接口边界的标准化来源
    
-   客户端源码属于实现层，可做内部模块、数据库与性能优化，但不能偏离协议要求的可观察行为
    

### 2\. 一个执行客户端不是 “EVM + 数据库”

从系统角度，执行客户端需协调这些子系统：

-   P2P：与其他执行节点连接、传播交易与区块
    
-   chain sync：同步链头、区块体、状态相关数据
    
-   txpool：管理待打包交易
    
-   block import：接收新区块并验证导入
    
-   state transition：调用 EVM 执行状态转换
    
-   database：持久化区块、收据、状态与索引
    
-   RPC：对外暴露 JSON-RPC 方法
    
-   engine interface：与 CL 通过 Engine API 协作
    

理解架构的核心是明确模块围绕两类输入组织：

-   新交易输入：走 txpool 与交易传播路径
    
-   新区块 / 新 payload 输入：走区块导入与状态重放路径
    

两条路径都会进入状态转换逻辑，但触发点和上下文不同。

* * *

## 流程

### 1\. 一个新交易被接收的最小路径

1.  交易通过 JSON-RPC 或 P2P 被节点接收
    
2.  基础验证：签名、字段格式、nonce、余额、费用参数等
    
3.  交易进入 txpool，按账户顺序、费用条件等组织
    
4.  节点通过 P2P 广播给其他节点
    
5.  构建区块或 payload 时，从 txpool 选出候选交易
    

交易并非一进入节点就交给 EVM，txpool 是重要中间层，负责把外部请求转为候选执行输入。

### 2\. 一个新区块被接收的最小路径

1.  节点从 P2P 或跨层接口得到新区块 / 执行 payload 信息
    
2.  检查头部、父区块关系和基本结构
    
3.  调用 block import 路径，按父状态逐笔执行区块内交易
    
4.  执行中更新状态、收据、日志与相关根
    
5.  比对计算结果与区块头承诺是否一致
    
6.  导入成功后更新本地链视图与数据库索引
    

区块导入不只是存储数据，本质是重放状态转换、验证结果是否匹配头部承诺。

* * *

## 客户端架构

### 1\. txpool 的职责边界

txpool 不是简单队列，需处理：

-   同一发送者交易的 nonce 顺序
    
-   费用竞争与替换
    
-   已失效交易的清理
    
-   本地交易与远程交易的区分
    

txpool 自带执行层经济与语义约束，不只是缓存层。

### 2\. state transition 不是独立线程，是多流程核心共享环节

导入区块、构造 payload 都要进入状态转换路径，EVM 是多子系统调用的核心引擎，block import、simulation、eth\_call、estimateGas 等都会触发执行逻辑。

### 3\. database 的职责不止 “存数据”

数据库需保存区块头、体、收据、日志索引，支持状态恢复、回滚、历史查询与同步中间视图。协议定义 stateRoot、receiptsRoot 等，实现需落地到真实读写路径，不同客户端数据库设计差异大，但必须映射回同一协议语义。

### 4\. RPC 与 Engine interface 面向不同世界

-   RPC：面向钱包、前端、SDK、后端基础设施
    
-   Engine API：面向共识层客户端
    

两者都是接口，但类型完全不同，应用层熟悉的 JSON-RPC 并非节点接口全部，执行客户端还需与 CL 协作，这部分不向普通应用暴露。

* * *

## 边界与易混点

1.  规范不等于某个客户端源码：客户端实现可不同，但共识结果必须一致，协议标准不能从某实现习惯推出
    
2.  EIP 不等于实现文档：EIP 定义规则与对象，实现文档关注落地方式，二者互相参照而非替代
    
3.  “节点同步” 不只是下载区块：同步涉及区块、状态、索引、链头视图与验证路径
    
4.  eth\_call 不是区块导入：是在指定状态视图上做不提交状态的执行模拟
    

* * *

执行层规范与客户端是 “共识边界 vs 工程实现” 的关系，前者定义必须一致的行为，后者组织为真实软件。把 txpool、block import、state transition、database、RPC、Engine API 整合看待，才能真正理解执行客户端系统。

* * *

## 未解

-   不同主流执行客户端在 chain sync 和状态存储路径上的关键差异
    
-   易导致跨客户端不一致的高风险接口 / 模块
    
-   execution-specs 与具体客户端测试套件的闭环形成方式
    

* * *

## 小结

今天是拆分协议规则与客户端实现。执行层是由 Yellow Paper、EIPs、execution-specs、execution-apis 和多客户端实现共同构成的系统。EVM 处于整个流水线中心，后续学习 EVM 时，这一框架很关键
<!-- DAILY_CHECKIN_2026-04-08_END -->

# 2026-04-07
<!-- DAILY_CHECKIN_2026-04-07_START -->








# 设计取舍与协议演进

昨天解决的是 “以太坊为什么存在、今天为什么必须分层”，今天要继续往下追：如果以太坊的目标是一个共享执行环境，它为什么会选账户模型、Gas、EVM、Trie 这一整套组合？以及，为什么这些设计没有在主网上线时就固定下来，而是通过 Frontier、Byzantium、London、The Merge、Dencun 等一系列硬分叉不断修正？

这两个问题其实不能分开看。设计理念如果只讲抽象目标，会显得像事后总结；升级历史如果只列时间线，又会像版本号背诵。真正有价值的方式，是把它们串成 “问题 -> 协议答案 -> 历史修正” 的链条。这样看，以太坊不是一开始就找到了完美结构，而是在运行中不断暴露边界、重新定价资源、重塑模块职责。

## 机制

### 1\. 账户模型回答的是 “状态如何存在”

一条通用状态机首先必须回答：状态对象是什么。如果底层继续以 UTXO 为中心，虽然能很好地表达花费关系，却不适合表达持续存在的代码与存储。因此以太坊选择了账户模型。

在协议视角里，world state 可以看作地址到账户对象的映射。每个账户对象至少包含四个核心字段：

-   `nonce`：EOA 已发送交易数，或合约账户已创建合约数
    
-   `balance`：该地址持有的 wei 数量
    
-   `codeHash`：该账户代码的哈希引用
    
-   `storageRoot`：该账户存储 Trie 的根
    
    这四个字段已经能体现账户模型的方向：
    
-   `nonce` 负责顺序与防重放
    
-   `balance` 承担价值状态
    
-   `codeHash` 让代码成为账户对象的一部分
    
-   `storageRoot` 让长期状态也成为协议对象
    
    因此，以太坊的 “账户” 不是银行式账户，也不是简单地址标签，而是状态机里的基本单元。EOA 和合约账户都处在同一个 world state 中，只是是否带有代码与存储不同。
    

### 2\. Gas 回答的是 “通用执行如何不失控”

只要允许链上逻辑具备较强表达能力，就必须处理资源滥用问题。Gas 是以太坊给出的统一答案。它不只是交易费单位，而是协议对执行资源的计量模型。

在执行层里，Gas 同时覆盖几类成本：

-   计算成本：opcode 执行本身
    
-   状态访问成本：例如 `SLOAD`、`SSTORE`
    
-   内存扩展成本：memory 的增长
    
-   数据处理成本：calldata、copy、hash 等相关操作
    
    所以 Gas 的本质作用至少有三层：
    

1.  给每次状态转换设定资源上界
    
2.  让 DoS 型执行变成有成本行为
    
3.  把区块空间和执行资源纳入经济定价
    
    如果没有 Gas，EVM 再强也无法在开放网络里稳定运行；如果只有 Gas 计费而没有严格执行语义，计费又无法与共识绑定。因此 Gas 与 EVM 是一组配套设计。
    

### 3\. EVM 回答的是 “谁来执行状态转换”

一旦协议决定支持通用状态机，就需要一个受约束、可复现、可多客户端实现的执行环境。EVM 的真正位置就在这里。

它不是为了给开发者提供最舒服的编程体验，而是为了让所有节点在同样前状态和输入下，得到同样后状态。换句话说，EVM 优先服务于共识可重复执行，而不是本地运行效率。

从设计角度看，EVM 之所以能够站住，是因为它把这些要求捆在了一起：

-   栈机模型，规则简单
    
-   256-bit word，便于配合哈希和密码学对象
    
-   opcode 语义明确
    
-   与 Gas 紧密绑定
    
-   执行结果直接作用于账户状态与收据
    
    因此，EVM 不是一个孤立组件，而是账户模型和 Gas 机制被执行出来的地方。
    

### 4\. Trie 回答的是 “复杂状态如何被承诺”

如果链只关心少量平铺状态，很多存储结构都能工作；但以太坊要求的是：维护庞大的 world state，并对其结果给出单个根哈希级别的承诺。于是，Modified Merkle Patricia Trie 成为协议层的关键数据结构。

Trie 在这里承担的不是普通数据库职责，而是承诺职责。它至少满足几件事：

-   对整个状态给出可比较的根
    
-   支持局部更新后重新计算根
    
-   允许某个账户或存储槽通过 proof 被验证
    
    这也是为什么区块头里最重要的字段之一是 `stateRoot`。协议不要求所有节点每次把整份状态互相发送一遍，而是要求所有节点就同一个根达成一致。Trie 让这种事情成为可能。
    

## 流程

### 从设计选择到实际状态转换的一条线

把四个经典设计连在一起，一条最小流程大致是这样的：

1.  用户通过 EOA 发起一笔交易。
    
2.  交易引用发送者账户的 `nonce`，表明这次状态推进的顺序。
    
3.  EVM 根据交易输入、账户代码与当前状态执行对应逻辑。
    
4.  执行过程中每一步都受到 Gas 约束。
    
5.  执行结果会修改账户对象、合约存储和日志 / 收据。
    
6.  这些修改最终被重新编码进 state trie 与相关根承诺中。
    
    这条链说明，账户模型、Gas、EVM、Trie 不是并排摆放的四个概念，而是同一条状态转换链路上的不同环节。
    

## 协议的演进

### 1\. Frontier / Homestead：先把系统跑起来

Frontier 阶段最重要的问题不是 “设计完美”，而是 “协议是否可运行”。Homestead 进一步提升了稳定性，使网络从高度实验状态进入更可持续维护的阶段。这个阶段体现的是协议最初的生存问题：能不能先作为一个真实网络存在。

### 2\. Byzantium：执行语义与密码学能力增强

Byzantium 很适合作为一个转折点来理解。它一方面带来了执行语义相关改进，例如 `STATICCALL`；另一方面也引入了更重要的密码学支持，为后来的 zk 相关能力铺路。这意味着协议开始从 “可运行” 走向 “可扩展、可增强”。

从今天回看，Byzantium 代表的是一类演进主线：执行环境不能只维持原样，它必须不断吸收新的安全边界和新原语。

### 3\. London：费用市场重构

London 中最关键的变化是 EIP-1559 引入的费用市场重构。这里的重点不只是 base fee 和 priority fee 这两个名词，而是区块空间的定价方式被系统性改写。

1559 带来的核心变化包括：

-   base fee 按拥堵动态调整
    
-   priority fee 作为给 proposer 的激励
    
-   区块目标容量与弹性容量并存
    
    这类变化说明，Gas 不只是执行约束机制，它同时还是区块空间市场的一部分。因此，费用模型的升级并不是外围优化，而是执行层经济设计的重写。
    

### 4\. Paris (The Merge)：EL / CL 分层完成

The Merge 是现代以太坊的结构性分水岭。Paris 使执行层不再承担共识完整职责，而是正式进入 EL / CL 分工模式。这里的关键不是 “PoW 变成 PoS” 这么一句话，而是：

-   fork choice 不再由 EL 单独完成
    
-   block proposal 与 finality 进入 CL
    
-   EL 以 execution payload 的形式向 CL 提供执行内容
    
    因此，从 Merge 开始，任何对执行层的理解都必须放到双层架构里。
    

### 5\. Shanghai-Shapella / Dencun：系统职责继续重写

Shanghai-Shapella 使质押提款路径完整化，说明 EL 会持续接收来自更大协议系统的重要状态变化。Dencun 则进一步明确以太坊向 rollup-oriented 路线前进，执行层越来越需要为更高层扩容方案提供基础承载能力。

Dencun 的意义不只是 “又一次硬分叉”，而是协议重心的转移：执行层不再只是服务 L1 用户交易体验，也在服务整体扩容路线。

## 边界与混淆点

### 1\. 账户模型不是中心化账户系统

它描述的是协议状态对象，而不是某种账户管理服务。账户对象的四个字段直接决定了状态机如何推进。

### 2\. Gas 不只是手续费

如果只把 Gas 理解成给矿工 / 验证者的钱，就会忽略它首先是资源计量和执行上限模型。

### 3\. Trie 不是 “为了查得快” 才被选中

Trie 在协议里最重要的功能是状态承诺和 proof，而不是单纯替代传统数据库索引。

### 4\. 升级历史不是独立于设计理念的附录

硬分叉不是版本记录，而是协议在现实网络中持续校正其设计边界的过程。

## 总结

目前更清楚的一点是：以太坊最核心的设计选择其实互相锁定。只要目标是共享状态机，就会自然走到账户对象、Gas 约束、EVM 执行和 Trie 承诺这组组合上。与此同时，这组组合并不是 “主网上线时一次定稿” 的答案，而是在每个硬分叉中被不断重估、重写和扩展。

设计选择回答的是 “协议最初如何成立”，升级历史回答的是 “这个答案后来怎样被不断修正”。两者只有放在一起看，账户模型、Gas、EVM、Trie 才不会变成孤立名词；Frontier、London、The Merge、Dencun 这些节点，也才会显出真正的结构意义。

-   Gas 重定价背后的统一原则，是否能从多个硬分叉中抽象出来？
    
-   Trie 的长期局限在实现层最集中表现在哪些地方？
    
-   随着 rollup-oriented 路线强化，执行层未来还会在哪些职责上继续变化？
<!-- DAILY_CHECKIN_2026-04-07_END -->

# 2026-04-06
<!-- DAILY_CHECKIN_2026-04-06_START -->










# 为什么会有以太坊，为什么要分层

进入协议层时，有着最容易被忽略的两个的问题。一是，已经有了比特币，为什么还需要以太坊这样一条新链？二是，明明讨论对象还是 “以太坊”，为什么还同时谈到了执行层、共识层、validator client、Engine API 这些分层的概念？这两个问题连在一起。前面的决定了以太坊试图成为什么，后面的决定了它在演进之后为什么长成今天这种结构。

简单用一句 “以太坊是智能合约平台” 来解释的话，以太坊会显得就像是比特币之上的功能增强版。但协议层的真实情况并不是 “加一个功能”，而是 “换一种账本模型、状态模型和执行模型”。应用层开发者通常先接触到钱包、合约、RPC 和 SDK然后开发DAPP，因此更容易把链理解成一个远端计算平台。但协议层则要求继续向下探索：这个平台维护的到底是什么状态、这些状态怎样被节点共同执行、以及全网怎样就结果达成一致。

## 机制

### 1\. 从BTC的边界到通用状态机

比特币的成是让网络在没有中心机构的前提下，对一组 UTXO 的所有权转移达成共识。它解决的是价值转移与防双花问题，因此脚本系统的设计目标也很明确：为支出条件提供有限表达能力，而不是提供一个开放的共享执行环境。

当需求从 “支付” 扩展到 “通用链上应用” 时，约束会立刻暴露出来。以 UTXO 为中心的表示方式非常适合描述离散资产的花费关系，但并不天然适合表达这些对象：

-   一个带有内部变量的合约账户
    
-   多次调用之间持续存在的存储状态
    
-   不同应用共享的一套全局状态空间
    
-   一组外部调用驱动下的复杂状态机
    

早期的元协议、侧链或 “把更多逻辑挂在比特币上” 的尝试，都会碰同一个问题：如果底层账本并不把 “代码” 和 “长期状态” 当作协议对象，那么应用层就必须不断绕路。以太坊的改变正在这里。它没有把区块链继续限定为 “资产花费证明系统”，而是直接定义成一个可执行状态转换函数。

具体地说，以太坊把 world state 作为协议核心对象。这个 world state 可以理解为地址到账户对象的映射，而每个账户对象又至少包含：

-   `nonce`
    
-   `balance`
    
-   `codeHash`
    
-   `storageRoot`
    

它意味着合约代码和持久化存储不再是外挂系统，而是账本本身的一部分。于是，以太坊不再只是一个 “能转账的链”，而是一个 “共享执行环境”。交易不只是在移动价值，也在驱动 world state 从旧状态 `S` 变成新状态 `S'`。

### 2\. shared execution environment 的协议含义

“共享执行环境” 只停留在概念层的话，显得很抽象，放到协议层包含三层含义

-   所有节点面对的是同一份状态空间。某个合约部署到链上之后，它的代码和存储不属于某台服务器，而属于全网共同维护的状态对象。
    
-   外部输入会触发统一的状态转换。用户发送交易，不是请求某个中心化服务处理，而是请求全网节点按照协议定义执行同一套逻辑。
    
-   执行结果必须可被共识。共享执行环境并不意味着 “大家都能算点东西”，而意味着 “大家必须在同样输入下算出同样结果”。这就是后面 EVM、Gas、状态承诺结构全部存在的原因。
    

PS：以太坊相对比特币的关键变化，不是 “脚本更强”，而是协议对象从 UTXO 集扩展成了 world state，执行对象从有限支出脚本扩展成了通用状态转换。

### 3\. 为什么有 EL 和 CL的区分

在 PoW 时代，可以更自然地把 “区块被挖出” 和 “区块里的交易被执行” 看成一个连在一起的过程。但The Merge 之后，这种直觉就不再准确。现代以太坊必须拆成两层来看：

-   `Execution Layer (EL)`：负责交易执行、EVM、状态、交易池、执行数据结构、JSON-RPC。
    
-   `Consensus Layer (CL)`：负责 slot/epoch 流程、fork choice、最终性、区块提议与证明。
    

这不是文档层面的命名变化，而是职责的正式拆分。EL 不再单独决定哪条链有效；CL 也不直接执行智能合约代码。EL 关心 “这些交易执行后状态会变成什么”，CL 关心 “哪一个区块和哪一个 payload 会成为 canonical chain 的一部分”。

这种拆分带来了三个结果

-   节点软件不再只是 “一个客户端”。一个完整的 PoS 节点至少要有 execution client 和 consensus client。如果要承担验证者职责，还要有 validator client。
    
-   跨层对象变得非常重要。最典型的对象就是 `execution payload`。CL 提议的 beacon block 中会携带与 EL 相关的 payload 或其头部信息，EL 则负责构造和验证这部分执行内容。
    
-   层与层之间必须有稳定接口。这个接口就是 `Engine API`。它不是给钱包和前端调用的外部接口，而是 CL 与 EL 的内部协作边界。
    

## 流程

### 从用户交易到 beacon block 的最小跨层流程

最核心的跨层交接举例，一条简化后的链路流程如下：

1.  用户通过某个执行节点发送原始交易。
    
2.  EL 验证交易签名、nonce、余额与格式，将其放入 txpool。
    
3.  当 CL 侧进入某个提议时机，需要一个新的执行内容时，会通过 Engine API 与 EL 交互。
    
4.  EL 从当前链头状态和待选交易集合中构造 `execution payload`。
    
5.  CL 将该 payload 嵌入对应的 beacon block 提议流程。
    
6.  其他节点的 CL 负责验证该提议是否符合共识规则，其他节点的 EL 负责验证 payload 的执行合法性。
    

PS：beacon block 和 execution payload 不是同一个对象。前者属于共识层区块结构，后者属于执行层的区块内容。两者在现代以太坊里被明确拼接在一起，但功能和边界不同

## 边界与易混点

### 1\. “出块” 不等于 “执行”

应用层常把 “区块确认” 当成一个整体事件，但协议层需要把它拆开：

-   EL 负责把交易执行成 payload
    
-   CL 负责提议、选择和最终确定包含该 payload 的区块
    

“谁负责执行” 与 “谁负责让它成为 canonical head” 是两个不同问题。

### 2\. beacon block 不是 EL 区块的别名

现代以太坊的区块提议流程发生在 CL 世界里。EL 仍然维护执行所需的区块头、状态根、收据根、交易列表等结构，但这些执行内容是在更大的 beacon block 语境下被带入共识流程的。

### 3\. validator client 不等于 consensus client

两者经常一起出现，但角色不同。consensus client 实现的是 CL 协议逻辑；validator client 则负责与验证者密钥、提议 / 证明职责等相关的操作。很多节点部署中这两者会协作，但不应在概念上合并。

## 总结

目前更重要的理解不是 “以太坊支持合约”，而是 “以太坊把区块链重新定义成一个共享状态机”。一旦从 world state 和状态转换的角度看它，EL 与 CL 的分层就会显得非常自然：一层负责执行结果，一层负责共识结果。前者回答 “状态怎么变”，后者回答 “哪次变化算数”。以太坊并不是比特币脚本的扩展版，而是围绕 world state 和共享执行环境构建出来的协议；而当前的以太坊，也不再是单层系统，而是由 EL 与 CL 明确分工、通过 Engine API 协作的双层结构。

然后又有了下列问题

-   当 EL 与 CL 视图短暂不一致时，节点内部的同步和回滚路径是怎样处理的？
    
-   Engine API 的调用在实际客户端里是如何实现的？
<!-- DAILY_CHECKIN_2026-04-06_END -->
<!-- Content_END -->
