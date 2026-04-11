---
timezone: UTC+8
---

# 海边电视机

**GitHub ID:** JohnBOMBS

**Telegram:** 

## Self-introduction

EPF 实习计划

## Notes

<!-- Content_START -->
# 2026-04-12
<!-- DAILY_CHECKIN_2026-04-12_START -->
疑问  
执行层为什么要查看共识层？，跑在 EVM 里的智能合约，根本不知道高层发生了什么有什么关系吗？系统每 12 秒，自动把刚才说的那个\*\*“超级防伪二维码”\*\*贴到这个地址上。什么意思？

![image.png](https://raw.githubusercontent.com/IntensiveCoLearning/EPF_Bootcamp/main/assets/JohnBOMBS/images/2026-04-11-1775924474762-image.png)

这张图片简单来说EIP-4788每 12 秒送来的那个信标块根，就可以查看所有节点，赚取了多少，就可以反推质押账号利息是多少  
有状态的系统合约 (Stateful System Contract)：有存储功能的合约

执行层和共识层是分别的两条链

## [Gas Accounting](https://epf.wiki/#/wiki/EL/el-specs?id=gas-accounting)

[Intrinsic Gas Calculation](https://epf.wiki/#/wiki/EL/el-specs?id=intrinsic-gas-calculation)

![image.png](https://raw.githubusercontent.com/IntensiveCoLearning/EPF_Bootcamp/main/assets/JohnBOMBS/images/2026-04-11-1775928839564-image.png)

\*\*$G\_{initCodeWordCost}$：\*\*部署新合约时，代码每占用一个“字”的内存，要收的标价。  
$\\sum\_{i \\in \\{T\_{inputData}\\}}$：把你交易里携带的数据（拆成一个个字节），挨个算一遍钱，然后总和加起来。  
$G\_{txdatazero}$：交易数据里，内容为 `00` 的字节单价（目前物价局定价：4 Gas）。  
$G\_{txdatanonzero}$：交易数据里，内容不是零的实心字节单价（目前物价局定价：16 Gas）。  
**$\\vee$**：数学逻辑符号里的“或 (OR)”。  
$G\_{txcreate}$：这笔交易要发给谁  
$if \\ T\_{to} = \\emptyset$ 意思是“如果这笔交易没有接收方”。  
\*\*$T\_{accessList}$：\*\*交易自带的“预约访问清单”（也就是你提前告诉系统你要动哪些数据）。  
\*\*$G\_{accesslistaddress}$：\*\*清单里每预约一个“地址”，系统收取的准备费。  
\*\*$G\_{accessliststorage}$：\*\*清单里每预约一个具体的“存储硬盘位”，系统收取的准备费。  
总结：\_g\_0​≡字节收费+base fee+ 交易的地址收费+ 交易的存储收费  
  
[Effective Gas Price & Priority Fee](https://epf.wiki/#/wiki/EL/el-specs?id=effective-gas-price-amp-priority-fee)  

![image.png](https://raw.githubusercontent.com/IntensiveCoLearning/EPF_Bootcamp/main/assets/JohnBOMBS/images/2026-04-11-1775930005857-image.png)

**effectiveGasPrice**：是从你钱包里实打实扣掉的钱（起步价被系统直接烧毁销毁了）。  
**priorityFee**：是真正发给接单干活的节点（矿工）的钱。  
  
Gas Price 是 _gas的单价，like_ 10 Gwei / Gas
<!-- DAILY_CHECKIN_2026-04-12_END -->

# 2026-04-11
<!-- DAILY_CHECKIN_2026-04-11_START -->



# 公式10

$$L\_s(\\sigma) \\equiv \\{p(a) : \\sigma\[a\] \\neq \\emptyset\\}$$  
**$\\sigma$ (World State，世界状态)**：

当前全网所有以太坊地址状态的总和。在底层，它就是一个巨大的键值对映射表（Key 是钱包地址，Value 是账户的四大字段：`nonce`, `balance`, `storageRoot`, `codeHash`）。

-   **$a$ (Address)**：
    
    具体的以太坊地址（比如 `0x123...`）。
    
-   **$\\sigma\[a\]$ (Account State)**：
    
    具体某个地址在底层的状态数据。
    
-   **$\\emptyset$ (Empty Account，空账户)**：
    
    **这是关键！** 在以太坊的定义里，什么是真正的“空”？
    
    条件极其苛刻：`nonce` **为 0 且** `balance` **为 0 且** `codeHash` **为空。** 只要你是一个部署过的智能合约，哪怕里面没钱了，你也**绝对不是空集**。
    
-   **$L\_s$ (State Collapse Function)**：
    
    状态坍缩函数。  
    在以太坊里，判断一个账号是不是垃圾空壳（$\\emptyset$），标准极度苛刻。必须**同时满足三个条件**：
    
    1.  **没有钱**（余额 Balance = 0）。
        
    2.  **没有交易动作**（Nonce = 0，也就是没往外发过交易）。
        
    3.  **没有代码**（不是智能合约，Code = 空）。
        

简单来说，公式10判断\*\*$\\sigma\[a\]$ 中的\*\*nonce、balance、codeHash、storageRoot这些是否都等于0，然后就进行压缩  
公式39  
**$\\sigma$ (Local World State)**：你当前节点 LevelDB 数据库里存着的、此刻的本地全局状态（由无数个账户键值对组成）。

-   **$L\_s(…)$ (State Pruning / Collapse)**：就是咱们上一步聊的，把本地状态树中满足 `nonce==0 && balance==0 && codeHash==empty` 的物理节点剔除。
    
-   **$TRIE(…)$ (MPT Root Calculation)**：将修剪后的本地状态，通过 Modified Merkle Patricia Trie 算法进行层层哈希，最终算出你本地的 `StateRoot` 哈希值。
    
-   **$P$ (Parent Block)**：当前即将处理的新区块的**父区块**（也就是链上的前一个区块）。
    
-   **$B\_H$ (Block Header)**：区块头数据结构。
    
-   **$\_{H\_{stateRoot}}$**：区块头结构中的 `stateRoot` 字段（一个 32 字节的哈希值）。
    

\=就是说明我本地的存储跟全网的存储都一样  
区块只存交易的动作  
状态根 (StateRoot)像是你的 MySQL 数据库当前所有表数据的全量 Hash。  
状态根 (StateRoot) 是个账号数据处理成一个哈希，然后跟另一个账号数据结合处理成一个哈希，“两两合并、层层向上”的结构  
公式 35b  
$$H\_{stateRoot} \\equiv TRIE(L\_s(\\Pi(\\sigma, B)))$$  
**准备材料 ($\\sigma, B$)**：节点拿出现有的本地底层数据库/旧状态 ($\\sigma$)，和刚刚收到的新区块 ($B$，里面装满了新交易)。

-   **执行交易并算账 ($\\Pi$)**：大写的 $\\Pi$ (Pi) 是状态转换函数（其实也就是 EVM 虚拟机）。节点用 EVM 把新区块 $B$ 里的几百笔交易全部跑一遍，该扣钱扣钱，该加钱加钱，得到一个**更新后的状态大网**。
    
-   **状态坍缩/大扫除 ($L\_s$)**：对这个更新后的状态大网，执行咱们刚刚聊过的公式 10，把那些变为空壳的垃圾账户物理剔除。
    
-   **生成新哈希 ($TRIE$)**：把清理干净的全新状态数据，全部扔进 MPT 树里，层层向上哈希。
    
-   **最终结果 ($H\_{stateRoot}$)**：树的最顶端算出来的这个新哈希值，就是**新的状态根**。
    

总结，之前的公式 7、10、39都是在处理数据，公式35b才是交易执行本身，公式 39 (身份核对)，公式 7 (打包规则)，公式 10 (打扫卫生)，公式 35b 中的 $\\Pi$ (核心引擎)

[区块头验证](https://epf.wiki/#/wiki/EL/el-specs?id=block-header-validation)  
57a 一直到 57t都是下一个区块需要满足的条件  
其中比较主要的是  
**(57e) 时间必须向前流逝**：`H_timeStamp > P(H)_timeStamp`。新区块的时间戳必须比上一个区块大。  
(**57f) 区块号必须严格 +1**：`H_number = P(H)_number + 1`。如果上个区块是 1000，新区块必须是 1001，绝不能跳号。  
**(57a) gas不能超标**：`H_gasUsed <= H_gasLimit`。这个区块里所有交易消耗的算力，绝不能超过硬性上限。  
**(57b, 57c) gas上限不能瞎改**：它规定了当前区块的 `gasLimit` 相对于上一个区块的 `gasLimit`，波动的幅度不能超过 $1/1024$。这是为了防止有人恶意瞬间拉高全网算力上限  
**(57k) 难度必须为 0**：`H_difficulty = 0`

**(57l) 随机数必须为 0**：`H_nonce = 0x000000...`  
57p 到 57t专门用来校验 Layer2 提交上来的“大包数据 (Blobs)”有没有超载。

![image.png](https://raw.githubusercontent.com/IntensiveCoLearning/EPF_Bootcamp/main/assets/JohnBOMBS/images/2026-04-10-1775842555178-image.png)

为了让费用“可预测”，并且消灭网络里“虚假的拥堵”。

EIP-1559 整个提案的终极使命，是为了给以太坊装上“确定性的计价器”和“有弹性的减震器”，让这台世界计算机变得真正好用！  
  
总手续费 = 基础费 (Base Fee) + 优先费/小费 (Priority Fee)  
1\. 基础费 (Base Fee)：100% 销毁  
2\. 优先费 (Priority Fee)：100% 发给验证者（矿工）

  
[区块间天然气价格动态](https://epf.wiki/#/wiki/EL/el-specs?id=dynamics-of-gas-price-block-to-block)  
L1每12s打包一个区块，然后根据上一个区块的大小来定下一个区块的gas费用  
  
设置$\\xi$，让base Fee可预估，并防止大量的冗余数据会把全球节点的内存和硬盘撑爆，导致低配置的节点大面积掉线。  
  
$\\rho$决定何时调整gas fee,相互作用ρ_ρ_和ξ_ξ_弹性乘数（ρ_ρ_不仅会移动拐点，还会调节由基本费用最大变化分母变化引起的调整的敏感性（ξ_ξ_这种互动凸显了以太坊为确保网络效率和稳定性而在不同需求下维持的微妙平衡。  
  
总结EIP-1559 是以太坊主网的gas“调控器”  
  
  
L1区块外挂Blob  
  

![image.png](https://raw.githubusercontent.com/IntensiveCoLearning/EPF_Bootcamp/main/assets/JohnBOMBS/images/2026-04-11-1775902556613-image.png)

X轴对应的时间，蓝线Y轴对应Blob消耗的gas fee,绿线Y轴对应累计个数（Excess Blob 数量），红线Y轴对应钱（Wei / Gwei）  
整个图表其实是核心开发者做的一场长达 **2 小时 40 分钟的“世界末日压力测试”**！  
L2打包的区块越大交给L1的gas fee就越大
<!-- DAILY_CHECKIN_2026-04-11_END -->

# 2026-04-10
<!-- DAILY_CHECKIN_2026-04-10_START -->






以太坊是个“单线程”的机器  
因为**状态依赖（State Dependency）**。  
_σt_ + 1​≡Π ( \_σt\_​，_B_ ）就是极其单纯地只管**一笔交易**的进出。而以太坊的强大之处就在于，靠着这个极其确定、极其死板的“单步函数”，硬生生通过循环，推演出了整个庞大区块、乃至整条区块链十几年来的沧海桑田！  
针对**单笔交易**，公式的作用是\*\*“走一步”**； 针对**整个区块\*\*，它是把这个公式\*\*“连续循环走 200 步  
像是以下的操作显示

![image.png](https://raw.githubusercontent.com/IntensiveCoLearning/EPF_Bootcamp/main/assets/JohnBOMBS/images/2026-04-09-1775754790070-image.png)![image.png](https://raw.githubusercontent.com/IntensiveCoLearning/EPF_Bootcamp/main/assets/JohnBOMBS/images/2026-04-09-1775754813821-image.png)![image.png](https://raw.githubusercontent.com/IntensiveCoLearning/EPF_Bootcamp/main/assets/JohnBOMBS/images/2026-04-09-1775754828352-image.png)![image.png](https://raw.githubusercontent.com/IntensiveCoLearning/EPF_Bootcamp/main/assets/JohnBOMBS/images/2026-04-09-1775754852241-image.png)![image.png](https://raw.githubusercontent.com/IntensiveCoLearning/EPF_Bootcamp/main/assets/JohnBOMBS/images/2026-04-09-1775754865404-image.png)

接下来认识黄皮书的一些公式  
$$L\_I((k, v)) \\equiv (KEC(k), RLP(v))$$ 它是以太坊存数据时的“第一道安检门”。  
**$k$ (Key)**：比如你想存一个变量，它在智能合约存储槽里的位置索引（或者是钱包的地址）。  
**$v$ (Value)**：你想存的具体数据内容，比如余额 `100`，或者一段字符串 `"Hello"`。

**业务现状：** 此时的 $k$ 和 $v$ 还是人类能看懂的数据形式，以太坊底层的数据库（MPT 树）是拒绝直接接收它们的。  
**$KEC$ (Keccak-256)**：这是以太坊特有的哈希算法。**$RLP$ (Recursive Length Prefix)**：“特供版 JSON”  
**$L\_I$** 其实就是一个**数据处理流水线函数（Map Function）**。终极归宿：丢进 $TRIE$ 树  
_公式7_  
**$\\sigma$ (Sigma)**：代表整个“国家档案馆”（全世界所有人的大账本）。

-   **$a$ (Address)**：代表张三的“身份证号”（账号 ID / 钱包地址）。
    
-   **$\\sigma\[a\]$**：代表档案馆里，**专属张三的那个“大档案袋”**。
    
-   **$\\sigma\[a\]\_s$**：右下角这个小写的 $s$ (Storage)，代表张三档案袋里面装的\*\*“所有私密文件和单据”\*\*。  
    **右边的 $\\sigma\[a\]\_s$**：指的是处理完之后，在这个微型货架顶端生成的那把\*\*“专属张三档案袋的密码小锁（Storage Root 存储根哈希）”\*\*。  
    公式7简单来说就是把某账户的数据处理成可接入树的数据的形式  
    **Key 必须是定长的**（所以用 Keccak 算成统一长度的哈希）。
    
-   **Value 必须是极致压缩的**（所以用 RLP 抽干所有水分）
<!-- DAILY_CHECKIN_2026-04-10_END -->

# 2026-04-09
<!-- DAILY_CHECKIN_2026-04-09_START -->









今天有点忙，先欠着
<!-- DAILY_CHECKIN_2026-04-09_END -->

# 2026-04-08
<!-- DAILY_CHECKIN_2026-04-08_START -->










# Merkle树& Verkle树

Merkel树像是二分法，如果要查阅某条数据的正确性，需要这条数据上所有的节点，如果数据量大的情况下，这个Merkel就会比较臃肿，占用的空间会非常大  
Merkel树，如果需要验证子节点，需要父节点的上一层全部数据，以此往上推，致全部验证，证明体积随人数和深度迅速增加  
而 Verkle树，如果需要验证子节点，只需要验证连接子节点的父节点数据，以此往上验证，证明体积极其精简，只跟深度有关，跟宽度无关  
[**递归长度前缀（RLP）**](https://epf.wiki/#/wiki/protocol/design-rationale?id=recursive-length-prefix-rlp)  
RLP 不试图定义任何特定的数据类型，例如布尔值、浮点数、双精度浮点数甚至整数——相反，它的存在仅仅是为了以嵌套数组的形式存储结构。  
RLP传输数据，是以\[前缀标签\] + dog（假设要传输的数据），  
RLP 的规则是：如果是短字符串，前缀就是 `0x80` 加上字符串的长度（3）。

-   所以 `0x80 + 3 = 0x83`。
    
-   最终 RLP 编码出来的十六进制就是：`83 64 6f 67` (`64 6f 67` 就是 dog 的 ASCII 码)。  
    当ETF读到83这个字节，就会知道这个是一个短字符串并且后面的 **3** 个字节全是它的内容。  
    RLP 只认识两种极其简陋的数据结构：字符串 和列表
    

[**简单序列化（SSZ）**](https://epf.wiki/#/wiki/protocol/design-rationale?id=simple-serialize-ssz)  
重新在ETF定义固定区域布尔值 `boolean`（永远占 1 字节）、无符号整数 `uint64`（永远占 8 字节）、固定的哈希值 `bytes32`（永远占 32 字节），可变区域字符串 `String`（可能是 “A”，也可能是一篇 1 万字的文章）、动态数组 `List`（比如里面装了不知多少笔交易）使用SSZ能快速找到某笔交易  
最终性指的是保证区块无法被篡改或从区块链中移除，除非销毁至少33%的质押ETH。在以前的 PoW（工作量证明，也就是挖矿时代）区块链里，规则叫 **“最长链原则”（Longest Chain Rule）**。  
**Slot（槽位）**：每 **12 秒** 一个 Slot。  
**Epoch（纪元）**：每 **32 个 Slot** 组成一个 Epoch。  
**_Casper_** [FFG 负责处理Epoch](https://arxiv.org/abs/1710.09437v4) 它不关心中间那 31 个小区块，它只看每个 Epoch 的第一个区块。  
**_LMD GHOST负责处理Slot,LMD GHOST注重每个区块质押的ETF数量_**  
**_今天就到这吧 已经把协议那些看完了_**
<!-- DAILY_CHECKIN_2026-04-08_END -->

# 2026-04-07
<!-- DAILY_CHECKIN_2026-04-07_START -->















# 协议设计理念

了解了协议的简洁性，通用性，模块化，非歧视性，敏捷性，  
简洁性，复杂的架构会来更多的漏洞，需要投入更多的精力去维护，投入的产出比较低。  
通用性，EVM在全球的电脑上都使用的同一套规则，某一台EVM进行了任务，其他的EVM也会同时记录，以太坊本身没有**_特性，怎么理解呢_**，EVM只提供底层最基础的能力，以太坊底层**什么特性都没有，只有图灵完备的计算能力（EVM），所以我们在EVM可以开发各式各样的东西，**不预设未来，才能承载所有的未来  
模块化，Layer 2协议是在Layer 1上进行的，所以修改协议并不会造成很大的影响，Layer 1是地基，Layer 2就是在Layer 2上建立的各种商业产地，商业产地的变化并不会影响到Layer 1，Layer 2 模块化了 执行层，共识层，结算层，数据可用性层，极大的加快了链上的速度，外挂上去的智能合约（二分法）  
非歧视性，不需要KYC认证，不怕某个人直接把你合约给收走，你的就是你自己的，任何人都无可奈何，对于任何人的权利都是一样的  
封装式复杂性较低的方案也意味着系统性复杂性较低的方案  
**三明治模型复杂性 协议底层 顶层应用层 中间层逻辑层**  
**BTC 进行交易时 原先的纸币会分裂，一份是你支出的 ，一份是分裂后留下的 原先的纸币已经被销毁，所以**UTXO提供了更高的隐私性  
ETF的账号优势，余额整合，节省交易UTXO的空间，但不能追踪UTXO的历史记录
<!-- DAILY_CHECKIN_2026-04-07_END -->

# 2026-04-06
<!-- DAILY_CHECKIN_2026-04-06_START -->
















今天，我阅读了一篇关于以太坊史前时期的文章，并了解到所有的编程语言都是从 Unix 迭代而来的，Unix 是更早期的编程语言，对后来的编程语言有深远的启发。  
Merkel 谜题是为那些秘密窃听的黑客创建的，黑客需要花费大量时间来破译这些信息，而 Diffi-Herman 密钥交换算法是一个单向数学问题，它使用“离散对数”来防止黑客破解，它也是主流的底层安全通信方式。RSA 加密系统。正推容易，反推难。  
自由，如同自由，了解了我们现在经常在用的软件，在之前都是需要通过付费才能使用，GNU/Linux 证明了软件应该赋予用户权力，而不是限制用户。  
读了这篇文章[《无需身份识别的安全：让“老大哥”式监控过时的交易系统》，知道了加密货币的由来](https://dl.acm.org/doi/pdf/10.1145/4372.4373)  
比特币的产生，比特币网络只能进行交易，不能再其网络上建立应用
<!-- DAILY_CHECKIN_2026-04-06_END -->
<!-- Content_END -->
