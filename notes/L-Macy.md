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
# 2026-04-18
<!-- DAILY_CHECKIN_2026-04-18_START -->
web3斜杠青年的多种搞钱路子

-   就业市场趋势
    
    -   极致内卷与流动性增大
        
        -   内卷加剧：大量裁员将导致各岗位极致内卷，经验丰富的高薪人士也会与年轻人竞争。
            
        -   职场流动性：多数人工作年限缩短，可能频繁换工作，需具备安身立命的能力和多种搞钱方式。
            
    -   行业变化与个体应对
        
        -   行业形态：未来大公司与协作伙伴生态协作，个人公司将更常见。
            
        -   个体能力要求：职场青年需具备斜杠能力和自我驱动力，构建核心竞争优势和多重收入保障。
            
-   多种搞钱路径
    
    -   做 KOL 或 KOC
        
        -   社交媒体优势：推特、Youtube 等平台对华语内容算法推荐友好，人人可做号，易获机会。
            
        -   个人品牌价值：做号可构建个人品牌，吸引优质粉丝，带来工作机会和合作可能。
            
        -   广告变现：天赋异禀者做号成功后，广告会主动找上门，实现变现。
            
    -   多跳槽提升收入
        
        -   跳槽策略：前期可将第一家公司作为跳板，骑驴找马，多换几家公司，明确自身市场价值。
            
        -   简历优化：跳槽时可优化简历，突出匹配经验，新兴科技行业对工作时长要求相对宽松。
            
        -   职位与薪资提升：每次跳槽争取小组长位置，大胆要求涨薪，提升收入。
            
    -   卡位重要赛道
        
        -   赛道研究：关注投资号分析的赛道和项目，研究行业机会，提前卡位。
            
        -   案例分析：如从市场部转做增长，提前布局平台业务；DEX 平台比 CEX 机会更多。
            
        -   传统金融机会：有传统金融资源和经验者，可专注相关岗位，积累经验后跳槽或创业。
            
    -   利用行业波动机会
        
        -   金融行业机会：金融行业有波峰波谷，抓住 FT、铭文、MEME 等机会可赚钱，要吃鱼头不吃鱼尾。
            
        -   猎头与就业辅导：人才流动加剧，猎头和就业辅导需求大，可从中获利。
            
        -   返佣业务：借助离岸金融，通过有一定粉丝量的号做返佣业务可赚钱。
            
    -   其他搞钱方式
        
        -   创业案例：18 岁学生通过整合微博 KOL 资源与电商老板合作，第一年赚 100 万。
            
        -   00 后赚钱姿态：00 后目的性强，放下羞怯感，积极赚钱。
            
-   WEB3 增长逻辑
    
-   增长底层逻辑
    
    -   流量杠杆：流量是杠杆，交易所开代理可放大品牌和流量，获取更多用户。
        
    -   IP 号与社群：做真人 IP 号是流量入口，撬线下社群可获取资源。 \[图片\]
        
    -   业务场景匹配
        
        -   明确自身优势：思考自身优势和目标用户，如交易所要明确合约深度、现货上币速度等优势。
            
        -   差异化服务：传统业务可提供差异化服务，如设计、视频宣发、社群运营等。
            
        -   平台币策略：DEX 可将增长方案与平台币绑定，吸引用户。 \[图片\]
            
-   行业发展前景
    
    -   WEB3 行业现状与趋势
        
        -   流动性下滑：币圈流动性萎缩，传统金融标的增长超越币圈币种，散户交易欲望弱。
            
        -   行业转型：行业从野蛮增长转向规范化、系统化，正规军将打败野路子。
            
    -   行业机会分析
        
        -   前沿领域：链上 DEX、pancake、AI 相关研究等前沿领域机会多。
            
        -   与传统金融结合：传统金融资源将进入，如支付、数据资产 ABS 等领域有机会。
            
        -   基础设施价值：行业基础设施价值存在，比特币稳定，稳定币增多，AI 更倾向用数字资产作 TOKEN。
<!-- DAILY_CHECKIN_2026-04-18_END -->

# 2026-04-17
<!-- DAILY_CHECKIN_2026-04-17_START -->

SSZ 编码与树摘要（Merkleization）今天主要学了以太坊共识层的 SSZ (Simple Serialize)，它是专门为 Beacon Chain 设计的序列化格式，比执行层的 RLP 更适合共识需求。核心就是两件事：

-   序列化：把对象拆成固定长度（fixed）和可变长度（variable）两部分。固定部分直接写数据，可变部分（比如 List、ByteList）先放一个偏移量（offset），实际内容统一放到最后。这样结构清晰，也方便快速访问。
    
-   树摘要（Merkleization）：SSZ 不光是序列化，还天生支持转成默克尔树。每个 SSZ 对象都能算出 hash\_tree\_root，用 Generalized Indices 可以快速定位树里的任意节点。这点超级重要，因为它能轻松生成多重证明（multiproofs），轻客户端验证数据时只需要一小部分证明就够了，效率很高。
    

简单说，SSZ 的设计目标就是确定性 + 高效生成默克尔证明。Beacon 节点接口（Beacon Node API）Beacon Node 是共识层核心，Validator Client 通过标准 API 和它交互。主要流程：

-   Validator 客户端定期去轮询 /eth/v1/validator/duties 接口，查自己哪个 Slot 要负责 propose 区块或做 attestation。
    
-   需要出块时，先向 Beacon Node 请求区块模板（produceBlock），签名后再提交回 Beacon Node，让它广播出去。
    

整个交互比较清晰，Validator 主要负责签名，广播和状态维护还是靠 Beacon Node。共识层网络通信（Networking）共识层网络基于 libp2p，主要分成两个部分：

1.  GossipSub（传闻广播）  
    用于全网快速扩散信息，比如新出的 Beacon Block、Attestations（见证投票，这部分流量最大）、Sync Committee 数据等。  
    所有消息都是 SSZ 编码 + Snappy 压缩后传播，转发前还会做快速验证，防止垃圾信息。
    
2.  Req/Resp（请求/响应）  
    点对点模式，主要用来拉取特定数据，比如新节点同步历史区块、请求状态或证明等。
    

额外还有：

-   发现层：用 discv5 协议（UDP）找其他 Beacon 节点。
    
-   传输层：TCP + Noise 协议加密。
    
-   所有消息传输前基本都会 Snappy 压缩，省带宽。
    

整体感觉共识层网络设计得很实用，Gossip 负责广播，Req/Resp 负责精确拉取，配合 SSZ 的树结构，效率和安全性都兼顾了。
<!-- DAILY_CHECKIN_2026-04-17_END -->

# 2026-04-16
<!-- DAILY_CHECKIN_2026-04-16_START -->


今日学习

1\. 以太坊权益证明（PoS）概述

-   核心系统：信标链（Beacon Chain） 是以太坊共识层（Consensus Layer）的核心链。
    
-   作用：负责存储和管理所有验证者（Validator） 的登记信息、状态、余额和参与记录。
    
-   与执行层关系：信标链通过“合并（The Merge）”后与原执行层（Execution Layer）紧密协同，信标链产生区块头，执行层负责交易执行和状态更新。
    
-   当前状态（2026年）：信标链已运行多年，质押总量超 36M ETH，参与率（Participation Rate）通常保持在 97%~99.5% 以上。
    

2\. 验证者（Validator）生命周期成为验证者（入金/激活流程）

1.  存款机制：
    
    -   向执行层存款合约（Deposit Contract） 发送一笔单向 ETH 存款交易（最低 32 ETH）。
        
    -   存款凭证（Deposit Data）包含验证者公钥、提款凭证（Withdrawal Credentials）等信息。
        
2.  处理流程：
    
    -   执行层存款事件被信标链监听并处理。
        
    -   进入激活队列（Activation Queue） —— 限制新验证者加入速度，防止瞬间大量验证者涌入影响网络稳定性。
        
    -   满足条件：余额 ≥ 32 ETH + 通过验证 + 排队完成 → 激活（Active）。
        
3.  历史变化：
    
    -   初始阶段（Beacon Chain 早期）：只能单向存款，无法提取。
        
    -   Capella 升级（上海升级 Shapella，2023年4月） 后：支持提取（Withdrawal），分为：
        
        -   部分提取（Partial Withdrawal）：自动提取超过 32 ETH 的奖励（需 0x01 提款凭证）。
            
        -   完全提取（Full Withdrawal）：退出后提取全部本金。
            

退出机制（Exit）

-   主动退出（Voluntary Exit）：
    
    -   验证者自行发起退出请求。
        
    -   进入退出队列（Exit Queue），排队处理（防止大规模同时退出影响安全性）。
        
    -   退出后进入“退出中”状态，最终可提取 ETH。
        
-   强制退出（Slashing / Forced Exit）：
    
    -   因违规行为（如双重签名、长时间离线）被惩罚（Slashing）。
        
    -   罚没部分或全部质押 ETH，并强制退出。
        

注意：Pectra（Prague/Electra）升级后，验证者最大数量、存款机制等有进一步优化（如链上直接供应验证者存款等）。3. 信标链的主要负载：证明（Attestations）

-   证明（Attestation） 是信标链上最主要的负载来源。
    
-   核心作用：
    
    -   权益证明投票：验证者对信标链当前链头（Head）、最近的合理化区块（Justified）和最终确定区块（Finalized）进行投票。
        
    -   可用性投票：在分片（Sharding，后续 Danksharding 等升级中）中，对分片区块数据的可用性进行证明。
        
    -   支持最终性（Finality）：通过 Casper FFG（Friendly Finality Gadget）机制，实现经济最终性（经济上不可逆）。
        
-   证明过程：
    
    -   每个 Epoch（约 6.4 分钟，32 个 Slot）中，验证者委员会（Committee）被随机分配任务。
        
    -   验证者必须在指定 Slot 内提交 Attestation。
        
    -   参与率（Participation Rate） 是衡量网络健康的重要指标（当前通常 >98%）。
        
-   激励与惩罚：
    
    -   正确及时提交证明 → 获得奖励（基于有效余额）。
        
    -   未提交或延迟 → 轻微惩罚（Inactivity Leak）。
        
    -   恶意行为（如互相矛盾的证明） → Slashing（严重罚没）。
        

4\. 信标链关键概念补充

-   Epoch 与 Slot：
    
    -   Slot：时间单位（12 秒一个）。
        
    -   Epoch：32 个 Slot 为一个 Epoch，用于批量处理证明和最终性检查。
        
-   验证者职责（Validator Duties）：
    
    -   提案者（Proposer）：在被选中的 Slot 提出信标区块。
        
    -   证明者（Attester）：提交 Attestation（主要职责）。
        
    -   同步委员会（Sync Committee）：轻客户端同步支持。
        
-   提款凭证（Withdrawal Credentials）：
    
    -   0x00：早期 BLS 凭证（不可自动提取）。
        
    -   0x01：执行层地址凭证（支持自动部分提取，推荐）。
        

5\. 安全与面试视角（结合之前 devp2p/密码学背景）

-   PoS 安全核心：经济激励 + 惩罚机制（Slashing）防止 51% 攻击、长程攻击（Long Range Attack）等。
    
-   常见风险：
    
    -   大规模验证者离线 → Inactivity Leak 机制自动惩罚。
        
    -   相关性攻击（Correlated Failure） → 通过随机委员会分配缓解。
        
    -   质押集中 → 去中心化质押池（如 Rocket Pool、Lido）需关注。
        
-   与执行层交互：存款/提款通过 Engine API 在信标链与执行层间桥接。
    
-   CTF/渗透角度：可关注协议层攻击，如伪造 Attestation、fuzzing 存款处理逻辑、Slashing 条件绕过等。
<!-- DAILY_CHECKIN_2026-04-16_END -->

# 2026-04-15
<!-- DAILY_CHECKIN_2026-04-15_START -->



1\. devp2p 概述（Ethereum 执行层 P2P 网络协议栈）devp2p 是以太坊节点间点对点通信的核心协议栈，采用经典分层设计：

-   传输层（Transport Layer）：
    
    -   建立安全连接（TCP-based）。
        
    -   加密通信（使用 ECDH 密钥交换等）。
        
    -   多协议复用（multiplexing，支持不同 capability 子协议）。
        
-   发现层（Discovery Layer）：
    
    -   用于节点发现，找到网络中的其他对等节点。
        
    -   主要机制：
        
        -   discv4：基于 Kademlia 的 UDP 发现协议（较旧）。
            
        -   discv5：改进版，支持主题（topics）、更高效的发现。
            
        -   DNS Discovery：通过 DNS 列表快速引导节点发现。
            
-   应用层（Application Layer）：
    
    -   包括 RLPx（核心传输协议）和各种子协议（如 Ethereum Wire Protocol eth/xx）。
        

RLPx 的核心职责（注意：你笔记中提到“PLPx”，应为 RLPx 的笔误）：

-   加密通信 + MAC 校验（确保消息完整性和认证）。
    
-   连接管理（握手、会话建立、断开）。
    
-   多协议复用（通过 capability 协商，支持多个子协议在同一连接上运行）。
    
-   RLP 序列化（用于消息的结构化编码）。
    

RLPx 建立在 TCP 之上，握手后通过帧（frame）传输消息，帧头包含协议 ID 和大小信息。2. RLP 序列化（Recursive Length Prefix，递归长度前缀）RLP 是以太坊中用于序列化任意嵌套二进制数据（字节数组和列表）的标准格式。目标：空间高效、确定性、无需额外类型信息（结构由前缀决定）。编码规则（Encoding Rules）RLP 只处理两种基本类型：字符串（字节数组，bytes） 和 列表（list）。

-   单字节编码：
    
    -   如果输入是一个字节，且值在 0x00 ~ 0x7F（0~127）之间，直接保持原字节不变。
        
-   短字符串编码（长度 1~55 字节）：
    
    -   前缀 = 0x80 + 字符串长度。
        
    -   后面直接跟字符串本身。
        
    -   示例：空字符串（长度 0）编码为 0x80。
        
-   长字符串编码（长度 > 55 字节）：
    
    -   先将字符串长度编码为大端（big-endian）字节数组（无前导零）。
        
    -   前缀 = 0xb7 + 长度数组的字节长度。
        
    -   后面跟长度数组 + 字符串本身。
        
-   短列表编码（所有项编码后的总 payload 长度 1~55 字节）：
    
    -   前缀 = 0xc0 + 总 payload 长度。
        
    -   后面跟各项的 RLP 编码（递归）。
        
-   长列表编码（总 payload 长度 > 55 字节）：
    
    -   将总 payload 长度编码为大端字节数组。
        
    -   前缀 = 0xf7 + 长度数组的字节长度。
        
    -   后面跟长度数组 + 各项的 RLP 编码。
        
-   特殊值：
    
    -   空字符串（或整数 0）：0x80。
        
    -   空列表：0xc0。
        

注意：

-   正整数必须用 big-endian 表示，无前导零（0 等同于空字节数组）。
    
-   RLP 是递归的：列表中的项可以是字符串或嵌套列表。
    

解码规则（Decoding Rules）

1.  读取第一个字节（prefix）。
    
2.  根据 prefix 判断类型和长度：
    
    -   0x00~0x7F：单字节字符串。
        
    -   0x80~0xb7：短字符串（长度 = prefix - 0x80）。
        
    -   0xb8~0xbf：长字符串（后续字节表示长度）。
        
    -   0xc0~0xf7：短列表（长度 = prefix - 0xc0）。
        
    -   0xf8~0xff：长列表（后续字节表示长度）。
        
3.  递归解码 payload（对于列表，逐项解码）。
    

掌握建议（针对面试）：

-   手写几个例子：空值、短字符串（如 "dog"）、长字符串、嵌套列表（如以太坊交易或区块头）。
    
-   理解为什么 RLP 高效：无类型标签，只用前缀区分结构。
    
-   常见考点：RLP 编码交易、区块头、状态树节点；与 SSZ 的对比（SSZ 更结构化，用于共识层）。
    

3\. EOF（Ethereum Object Format，EVM 对象格式）EOF 是 EVM 字节码的可扩展、版本化容器格式（主要由 EIP-3540 等引入），在合约部署时进行一次性静态验证。设计目标与优势

-   更完善的验证：部署前进行全面静态分析（而非运行时）。
    
-   提升执行效率：清晰的代码段定义，便于优化和静态跳转。
    
-   增强安全性：强制严格的代码结构，减少无效指令、栈溢出、截断 PUSH 数据等攻击面。
    
-   面向未来：版本化设计，便于引入新 opcode、类型系统、子程序等，而不破坏向后兼容。
    

与传统 EVM 字节码的区别

| 特征 | 传统 EVM 字节码 | EOF 格式 |
| --- | --- | --- |
| 容器结构 | 非结构化纯字节序列 | 明确的头部 + 多个章节（header + body） |
| 代码验证 | 有限（主要运行时检查） | 全面、部署时一次性验证 |
| 跳跃机制 | 动态 JUMP / JUMPI（易出错） | 支持静态相对跳转（更安全、可验证） |
| 类型系统 | 无 | 函数签名、输入/输出类型、栈高度等 |
| 数据处理 | 代码与数据混合（难以区分） | 单独的 数据段（Data Section） |
| 扩展性 | 困难（新特性易引入不兼容） | 版本化容器，便于未来升级 |

EOF 基本结构（简要）

-   Header：魔法字节（0xEF00）、版本号、节类型（kind）、大小信息等。
    
-   Body：
    
    -   类型节（Type Section）：描述代码段的输入/输出、最大栈高度等。
        
    -   代码节（Code Section）：一个或多个子代码段（支持函数式调用，如 CALLF/RETF）。
        
    -   数据节（Data Section）：不可变数据，与代码分离。
        
    -   可能有容器节（嵌套 EOF）。
        

掌握建议（针对面试）：

-   理解 EOF 如何解决传统 EVM 的痛点（无结构导致的验证困难、动态跳转风险）。
    
-   知道相关 EIP：EIP-3540（容器）、EIP-3670（代码验证）、EIP-4750（多代码段）、EIP-5450（栈验证）等。
    
-   面试高阶问题：EOF 如何支持未来 EVM 改进？与传统 bytecode 部署的区别（以 0xEF 开头视为 EOF）？
    

第一周学习反思与掌握建议

-   作为了解：你已经基本掌握了高层概念和核心规则，恭喜完成第一周！底层协议（如 devp2p + RLP）是理解以太坊节点同步、交易传播的基础；EOF 则是 EVM 现代化的关键。
    
-   作为面试/深入掌握：底层知识“高抽象”部分难懂很正常。真正掌握的标准：
    
    1.  能手写/模拟：RLP 编码/解码几个复杂嵌套结构；画出 devp2p 连接流程图。
        
    2.  能解释为什么：为什么 RLP 用前缀而不是类型标签？EOF 为什么在部署时验证而不是运行时？
        
    3.  能关联实际：RLP 在交易/区块中的应用；EOF 对 Solidity 编译器、gas 优化、安全审计的影响。
        
    4.  实践：用代码实现简单 RLP 编码器（Python/Go），阅读 geth/devp2p 源码片段，部署一个简单 EOF 合约（如果测试网支持）。
        

掌握路径：

-   多画图、写伪代码。
    
-   边学边实现小工具（例如 RLP 编解码器）。
    
-   第二周开始前，复习时尝试用自己的话重新讲解每个部分（Feynman 技巧）。
    
-   资源推荐：[Ethereum.org](http://Ethereum.org) RLP 文档、devp2p GitHub spec、EIP-3540 等。
<!-- DAILY_CHECKIN_2026-04-15_END -->

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
