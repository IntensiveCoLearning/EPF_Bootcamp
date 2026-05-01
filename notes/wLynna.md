---
timezone: UTC+8
---

# wLynna

**GitHub ID:** wLynna

**Telegram:** 

## Self-introduction

EPF 实习计划

## Notes

<!-- Content_START -->
# 2026-05-01
<!-- DAILY_CHECKIN_2026-05-01_START -->
# ZK 02｜核心原理：证明、验证者与秘密

> 核心句：  
> **ZK is not about hiding everything. It is about proving the right thing without revealing unnecessary information.**  
> ZK 不是把一切都藏起来，而是在不暴露不必要信息的情况下，证明关键事实成立。

* * *

# 1\. ZK 里最重要的三个角色

ZK 里先记住三个词：

**Prover**：证明者  
**Verifier**：验证者  
**Proof**：证明

比如你想证明：“我知道某个密码。”

你就是：

**Prover / 证明者**

对方是：

**Verifier / 验证者**

你交给对方的东西是：

**Proof / 证明**

注意，不是把密码交给对方，而是交给对方一个“可以验证的证明”。

**The prover creates a proof. The verifier checks the proof.**  
证明者生成证明，验证者检查证明。

* * *

# 2\. 最简单的例子：我知道密码，但不告诉你密码

普通互联网里，如果你要证明自己知道密码，通常要输入密码。

系统会检查：

你输入的密码是否正确。

但在 ZK 的世界里，理想情况是：

你不直接展示密码。  
你生成一个证明。  
别人验证这个证明。  
验证通过后，对方相信：你确实知道密码。

重点是：

验证者知道你“知道密码”，但不知道密码是什么。

**I can prove that I know the secret without revealing the secret itself.**  
我可以证明我知道这个秘密，但不暴露秘密本身。

这就是“零知识”的直觉。

* * *

# 3\. 什么叫“零知识”？

“零知识”不是说验证者什么都不知道。

验证者至少知道一件事：

**这个命题是真的。**

比如：

你确实知道密码。  
你确实超过 18 岁。  
这批交易确实计算正确。  
这个用户确实有资格投票。  
这个钱包确实满足某个条件。

但验证者不知道多余的信息。

比如：

不知道你的密码。  
不知道你的生日。  
不知道你的完整身份。  
不知道你的全部资产。  
不知道你的完整交易路径。  
不知道你的全部链上历史。

所以更准确地说：

**Zero-knowledge means the verifier learns the truth of a statement, but learns nothing extra.**  
零知识的意思是：验证者知道某个命题为真，但没有学到额外信息。

* * *

# 4\. ZK 不是“隐藏一切”，而是“只暴露必要结论”

这是很多人容易误解的地方。

ZK 不是单纯“隐身术”。  
ZK 不是把一切都加密起来。  
ZK 也不是说所有信息都不能看。

ZK 的核心是：

**最小披露。**

也就是：

只证明当前场景需要证明的事情，其他信息不暴露。

比如年龄验证：

不需要告诉别人你的生日、身份证号、住址。  
只需要证明：

“我超过 18 岁。”

比如 DAO 投票资格：

不需要暴露你的完整钱包历史。  
只需要证明：

“我拥有投票资格。”

比如合规：

不一定要把所有 KYC 信息放到链上。  
只需要证明：

“我已通过某种合规检查。”

**ZK enables selective disclosure.**  
ZK 允许选择性披露。

这个概念以后会和 identity / ENS / DID / governance 强相关。

* * *

# 5\. ZK 里的两个核心概念：Statement 和 Witness

现在进入一点点系统层。

ZK 证明里经常会出现两个词：

**Statement**：要证明的公开命题  
**Witness**：证明这个命题所需要的秘密信息

听起来抽象，但其实很好理解。

比如：

你要证明：

“我知道某个密码。”

这里的 Statement 是：

**我知道正确密码。**

Witness 是：

**那个真正的密码。**

再比如：

你要证明：

“我超过 18 岁。”

Statement 是：

**我超过 18 岁。**

Witness 是：

**你的真实生日 / 身份资料。**

再比如：

你要证明：

“这批 L2 交易计算正确。”

Statement 是：

**新的链上状态是正确的。**

Witness 是：

**完整交易数据、执行过程、中间计算信息等。**

**The statement is what you prove. The witness is the hidden information that makes the proof possible.**  
Statement 是你要证明的命题，Witness 是让这个证明成立的隐藏信息。

* * *

# 6\. 用一个“门”的故事理解 ZK

经典理解方式是“洞穴故事”，但我们换成更简单的“门”。

假设有一扇智能门。  
只有知道密码的人，才能从门里走出来。

你想向我证明：

“我知道门的密码。”

但你不想告诉我密码。

于是你可以这样做：

我站在门外。  
你进去。  
门关上。  
我看不到你输入了什么。  
过一会儿，你从门里面出来。  
我就知道：你确实知道打开门的方法。

我没有看到密码。  
但我相信你知道密码。

这个过程就是 ZK 的直觉：

证明者通过某种动作证明自己知道秘密。  
验证者只确认结果，不获得秘密。

当然，真实 ZK 不是靠“门”，而是靠数学证明。  
但逻辑类似。

**The verifier checks the result, not the secret.**  
验证者检查结果，而不是查看秘密本身。

* * *

# 7\. ZK 为什么能用于区块链？

区块链最重要的问题是：

**如何让不信任彼此的人，共同相信一个结果？**

以前的方式是：

所有节点都重新执行。  
所有数据都公开。  
大家一起验证。

这很安全，但成本高，隐私差。

ZK 提供了另一种路径：

某个系统先在链下完成计算。  
然后生成一个证明。  
链上只验证这个证明。  
验证通过后，链就相信结果正确。

这就是 ZK Rollup 的基本逻辑。

比如：

L2 处理了 10,000 笔交易。  
它不需要让 Ethereum L1 重新执行这 10,000 笔交易。  
它只需要提交一个 proof。  
Ethereum 验证 proof。  
如果 proof 有效，就接受新的状态。

**Ethereum does not need to re-execute every transaction; it only needs to verify the proof.**  
Ethereum 不需要重新执行每一笔交易，只需要验证证明。

这就是为什么 ZK 对扩容很重要。

* * *

# 8\. ZK 的三大性质

一个好的 ZK proof 通常需要满足三个性质：

## 1）Completeness｜完整性

如果命题是真的，诚实的证明者应该能让验证者相信。

比如你真的知道密码，那你应该能成功证明。

**If the statement is true, the proof should pass.**  
如果命题是真的，证明应该能通过。

## 2）Soundness｜可靠性

如果命题是假的，作弊者很难骗过验证者。

比如你不知道密码，却想假装知道，理论上应该几乎不可能成功。

**If the statement is false, a dishonest prover should not be able to convince the verifier.**  
如果命题是假的，不诚实的证明者不应该能骗过验证者。

## 3）Zero-Knowledge｜零知识性

验证者除了知道“命题是真的”以外，不应该获得额外信息。

比如验证者知道你确实知道密码，但不知道密码本身。

**The verifier learns that the statement is true, and nothing more.**  
验证者知道命题为真，但除此之外不获得更多信息。

这三个性质是 ZK 的基础骨架。

* * *

# 9\. ZK 和普通加密有什么不同？

这点很重要。

普通加密更像是：

**我把信息锁起来，不让你看。**

ZK 更像是：

**我不把信息给你，但我证明它满足某个条件。**

比如：

加密：我把生日加密，你看不到。  
ZK：我不告诉你生日，但证明我超过 18 岁。

加密：我把交易金额隐藏。  
ZK：我证明交易合法，但不暴露具体金额。

加密：保护数据内容。  
ZK：证明数据关系。

**Encryption hides data. ZK proves statements about data.**  
加密隐藏数据，ZK 证明关于数据的命题。

这句话非常关键。

* * *

# 10\. ZK 也不是“绝对隐私”

ZK 很强，但不要神化。

ZK 只能隐藏 proof 设计中不需要公开的信息。

如果系统设计不好，仍然可能泄露信息。  
如果链上地址本身暴露过，仍然可能被关联。  
如果应用层 metadata 暴露太多，仍然可能被追踪。  
如果用户行为模式很明显，也可能被分析出来。

所以 ZK 是隐私工具，但不是万能隐私魔法。

**ZK is a powerful privacy tool, but privacy also depends on system design.**  
ZK 是强大的隐私工具，但隐私也取决于系统设计。

这点对你理解 ENS / identity 很重要。

* * *

# 11\. 把它放回你的主线：身份与信任

ZK 对你的主线最重要的意义是：

它改变了身份系统的表达方式。

以前身份系统经常是：

证明我是谁 = 暴露我的资料。

ZK 之后可以变成：

证明我满足某个条件 ≠ 暴露我的完整身份。

比如：

我可以证明我是某个社区成员，但不暴露主钱包。  
我可以证明我是某个 ENS name 的控制者，但不暴露所有相关地址。  
我可以证明我有投票资格，但不暴露我的完整链上历史。  
我可以证明我是人类用户，但不暴露真实身份。  
AI agent 也可以证明自己拥有某种权限，但不暴露所有内部信息。

**ZK turns identity from full exposure into conditional proof.**  
ZK 把身份从“完整暴露”变成“条件证明”。
<!-- DAILY_CHECKIN_2026-05-01_END -->

# 2026-04-30
<!-- DAILY_CHECKIN_2026-04-30_START -->

# ZK 学习笔记 01

## 从“大地图”开始：ZK 为什么突然重要？

**ZK 的理论非常早，Web3 里的爆发比较晚。**

更准确地说：

**ZK is old in theory, but young in real-world blockchain adoption.**  
ZK 在理论上很早，但在区块链里的大规模应用还很年轻。

ZK 的核心思想最早可以追溯到 1980s 的密码学研究。后来在 Crypto / Web3 里，比较早被大众看到的是 **Zcash**，它在 2016 年使用 zk-SNARK 来实现更强的交易隐私。([Investopedia](https://www.investopedia.com/terms/z/zksnark.asp?utm_source=chatgpt.com))

但真正让 ZK 成为 Web3 热点的，是最近几年：

1.  Ethereum 需要扩容
    
2.  L2 需要更强的安全验证
    
3.  链上透明性带来了隐私问题
    
4.  AI、身份、合规、数据证明等新场景需要“可验证但不泄露”
    

所以你说“从 3 年前开始看到”是对的，因为大概从 2021–2024 以后，ZK 从密码学圈、隐私币圈，逐渐进入 Ethereum 主流路线。

* * *

# 1\. ZK 到底是什么？

ZK 是 **Zero-Knowledge Proof**，中文通常叫：

**零知识证明**

它的核心意思是：

**I can prove something is true without revealing the underlying information.**  
我可以证明一件事是真的，但不暴露背后的具体信息。

比如：你想证明自己超过 18 岁，但不想公开身份证号、生日、地址。

普通方式是：“把身份证给我看。”

ZK 方式是：  
“我只给你一个数学证明，证明我确实超过 18 岁，但你看不到我的生日和身份证信息。”

再比如：  
你想证明你知道某个密码，但不想把密码告诉别人。

普通方式是：“你把密码输入系统。”

ZK 思路是：  
“我证明我知道这个密码，但不把密码本身交出来。”

* * *

# 2\. ZK 的一句话理解

**ZK separates verification from disclosure.**  
ZK 把“验证”与“公开信息”分开了。

这句话很重要。以前的互联网和区块链世界，经常是：想验证，就要看见数据。

比如：

要验证你是谁，就要看你的身份资料。  
要验证你有钱，就要看你的账户余额。  
要验证交易正确，就要重新计算很多东西。  
要验证链上行为，就要把所有数据公开。

但 ZK 提供了一种新方式：  
**You do not need to see everything to verify that something is correct.**  
你不需要看到全部信息，也可以验证某件事是正确的。

这就是 ZK 最迷人的地方。

* * *

# 3\. ZK 为什么在 Ethereum 里这么重要？

因为 Ethereum 面临两个长期问题：

### 第一，扩容问题

Ethereum L1 很安全、很去中心化，但处理能力有限。所以出现了 L2。

L2 的基本思路是：把很多交易放到链下处理，然后把结果提交回 Ethereum。

但问题来了：Ethereum 怎么知道 L2 没有作弊？

ZK Rollup 的回答是：不用 Ethereum 重新计算所有交易，只要提交一个数学证明。

Ethereum 官方文档解释，ZK-rollups 会把大量交易放到链下执行，然后向主网提交较小的数据摘要和一个密码学证明，用来证明状态变化是正确的。([ethereum.org](http://ethereum.org))

**ZK-rollups allow Ethereum to verify many transactions without re-executing all of them.**  
ZK Rollup 让 Ethereum 不必重新执行所有交易，也能验证它们是正确的。

这就像：  
学生做了 100 道题。  
老师不需要重做 100 道题。  
学生交上来一个“数学证明”，老师只验证这个证明，就知道结果没错。

* * *

### 第二，隐私问题

区块链默认是透明的。这对信任很好，但对隐私不好。

比如：  
钱包地址、交易路径、资产变化、DAO 投票行为、身份关系，很多都可能被追踪。

ZK 可以让用户证明某件事，而不暴露全部细节。  
比如：  
我属于某个社区，但不暴露我是谁。  
我有某种资格，但不暴露完整身份。  
我投过票，但不暴露我投给谁。  
我有足够资产，但不暴露具体余额。  
我满足合规要求，但不公开所有个人信息。

这也是为什么 ZK 会跟 **identity / privacy / compliance / governance** 这些主题越来越相关。

* * *

# 4\. ZK 为什么现在成了热点？

我觉得可以用四个阶段理解：

## 阶段一：学术密码学阶段

这个阶段，ZK 主要是密码学研究者在研究。核心问题是：

**Can we prove knowledge without revealing knowledge?**  
我们能不能证明自己知道某件事，但不暴露这件事本身？

这听起来很哲学，也很数学。

* * *

## 阶段二：隐私币阶段

Zcash 是代表。  
它用 zk-SNARK 来隐藏交易里的发送方、接收方和金额等信息。([Investopedia](https://www.investopedia.com/terms/z/zksnark.asp?utm_source=chatgpt.com))

这个阶段，ZK 的主要标签是：  
**privacy**  
隐私

* * *

## 阶段三：Ethereum 扩容阶段

这是 ZK 进入主流 Web3 的关键。ZK 不再只是“隐藏信息”，而是变成：

**prove computation**  
证明计算是正确的

也就是说：不只是证明“我知道一个秘密”，而是证明：

这批交易计算正确。  
这个程序执行正确。  
这个状态变化正确。  
这个 L2 没有作弊。

Ethereum 文档也把 ZK-rollups 和 validiums 归为使用 validity proofs 的扩容方案，它们通过链下执行交易并在 Ethereum 上验证证明，来提升处理能力。([ethereum.org](http://ethereum.org))

这个阶段，ZK 的标签变成：  
**scaling + verification**  
扩容 + 验证

* * *

## 阶段四：通用证明基础设施阶段

这是现在正在发生的阶段。

ZK 不只是 Rollup 技术，而是逐渐变成一种更底层的基础设施。

a16z crypto 在 2025 年的报告里也提到，ZK 和 succinct proof systems 正从几十年的学术研究，演进为关键基础设施，并被用于 rollups、合规工具，甚至主流 Web 服务里的身份系统。([a16z crypto](https://a16zcrypto.com/posts/article/state-of-crypto-report-2025/?utm_source=chatgpt.com))

这个阶段，ZK 的标签变成：

**verifiable computing infrastructure**  
可验证计算基础设施

这句话很重要。它意味着未来很多系统可能都需要 ZK：

不是因为它“很酷”，  
而是因为未来的互联网需要一种能力：

**让别人相信结果，而不是暴露全部过程。**

* * *

# 5\. ZK 和 ETH 的关系：越来越深

你前面学 ETH 的时候，我们说过：

Ethereum 不只是一个币，也不是一个普通数据库。 它更像一个全球共享的可信执行与结算层。  
但如果所有事情都直接在 L1 上执行，成本很高。

所以 Ethereum 未来很可能走向：

-   L1 做安全和结算。
    
-   L2 / zkVM / offchain systems 做大量计算。
    
-   ZK 负责证明这些计算是正确的。
    

Ethereum Foundation 在 2025 年发布的 L1 zkEVM 相关文章中提到，很多 zkVM 已经可以证明 Ethereum blocks，并且性能突破正在频繁出现；EF 的 zkEVM 网站也明确写到，其目标是通过 zkVM 和 ZK proofs 扩展 Ethereum 主网。([Ethereum Foundation Blog](https://blog.ethereum.org/2025/07/10/realtime-proving?utm_source=chatgpt.com))

**Ethereum may become a verification layer for a much larger computation world.**  
Ethereum 可能会成为一个更大计算世界的验证层。

这句话对理解 Ethereum 未来很关键。

以前我们想的是：Ethereum 执行一切。

后来变成：Ethereum 结算 L2。

未来可能是：Ethereum 验证各种计算证明。

* * *

# 6\. ZK 的几个核心用途

## 1）扩容 Scaling

这是目前 Web3 里最主流的 ZK 用途。

代表：

zkSync  
Starknet  
Scroll  
Polygon zkEVM  
Linea  
Taiko  
Succinct / SP1  
RISC Zero  
Brevis  
Aligned  
等等

它们不完全一样，有些是 ZK Rollup，有些是 zkVM，有些是 proving infrastructure。

核心逻辑是：

**Do computation elsewhere, prove correctness onchain.**  
在链下完成计算，在链上证明正确性。

* * *

## 2）隐私 Privacy

比如：

private transfer  
private voting  
private identity  
private membership  
private DeFi  
private reputation

这跟 ENS 也有潜在关系。

比如未来可能会出现：

我可以证明我是某个 ENS name 的控制者，但不暴露主钱包。  
我可以证明我属于某个社区，但不暴露所有链上历史。  
我可以证明我的身份资质，但不暴露完整身份。

这和你之前关注的 **ENS × AI identity / Opaque / privacy identity safety** 是同一条线。

* * *

## 3）身份 Identity

ZK 身份的典型例子：

证明我成年，但不公开生日。  
证明我是某国居民，但不公开住址。  
证明我是某组织成员，但不公开真实身份。  
证明我有某种链上声誉，但不公开完整钱包历史。

**ZK identity allows selective disclosure.**  
ZK 身份允许“选择性披露”。

不是完全匿名，也不是完全公开，而是：我只证明当前场景需要知道的那一部分。

这个方向很适合连接 ENS、DID、VC、DAO governance。

* * *

## 4）合规 Compliance

这点很现实。

传统金融、支付、稳定币、RWA 都需要合规。但 Web3 用户不希望把全部隐私交出去。

ZK 可以提供一种中间路线：

-   证明你不是受制裁地址。
    
-   证明你通过了 KYC。
    
-   证明你符合某个地区规则。
    
-   但不把所有个人信息暴露在公开链上。
    

所以未来 ZK 可能会出现在：

稳定币  
交易所  
支付  
RWA  
跨境金融  
机构 DeFi

这里你可以跟你昨天分析的“支付赛道”连接起来。  
支付赛道成熟、合规、机构化，ZK 可能成为底层工具，但不一定适合你直接转去做支付公司。

* * *

## 5）AI × Web3

这是更未来的方向。  
AI 时代会有一个问题：AI 输出很多内容、决策、计算结果，但我们怎么知道它可信？

ZK 可能用于：

证明某个模型确实运行过。  
证明某个数据满足条件，但不暴露数据。  
证明某个 AI agent 有权限执行某事。  
证明某个推荐或判断遵循某个规则。  
证明某个计算结果没有被篡改。

这也是为什么 ZK 会跟 AI identity、agent identity、verifiable AI 连接起来。

* * *

# 7\. ZK 当前现状：热，但还没完全大众化

我会这样判断：

ZK 现在已经不是纯概念热点，而是基础设施热点。  
但它还没有完全进入普通用户体验层。

也就是说：  
工程师、研究员、协议团队、基金、L2 项目都很重视。  
普通用户可能还不知道自己什么时候用了 ZK。  
很多 ZK 产品仍然难用、成本高、开发门槛高。

a16z 在 2024 年报告中提到，ZK proofs 的验证成本在下降，而 ZK rollups 上的 ETH 价值在增加，也就是说 ZK 正变得更便宜、更常用。([a16z crypto](https://a16zcrypto.com/posts/article/state-of-crypto-report-2024/?utm_source=chatgpt.com))

同时，Succinct 在 2024 年融资 4300 万美元，目标就是让开发者更容易使用 ZK，因为当前 ZK 的一个核心问题是“很难用”。([Axios](https://www.axios.com/2024/03/21/succinct-raises-43-million-cryptography-zero-knowledge-proof-zk?utm_source=chatgpt.com))

所以现状可以总结为：

**ZK is powerful, but still hard to use.**  
ZK 很强大，但仍然很难用。

* * *

# 8\. 未来展望：ZK 会走向哪里？

我觉得有五个方向值得你长期观察。

## 方向一：ZK Rollup 继续成熟

这是最确定的方向。

未来几年，ZK Rollup 会继续围绕：

更低成本  
更快证明  
更好开发体验  
更强去中心化  
更安全的 prover / sequencer / bridge

竞争。

* * *

## 方向二：zkVM 成为关键基础设施

zkVM 可以简单理解为：一个可以生成证明的虚拟机。

以前写 ZK 应用，需要懂很多密码学和电路设计。

zkVM 的目标是：

让开发者用更熟悉的语言写程序，然后系统自动生成证明。

这非常重要，因为它把 ZK 从“密码学专家工具”推向“普通开发者工具”。

**zkVMs may make ZK programmable and developer-friendly.**  
zkVM 可能让 ZK 变得可编程，也更容易被开发者使用。

* * *

## 方向三：Ethereum L1 自身 ZK 化

这是很大的方向。

也就是说，未来 Ethereum L1 的执行过程本身，可能也会越来越依赖 ZK 证明。

Ethereum Foundation 的 L1 zkEVM 方向，正是在探索用 zkVM / ZK proofs 扩展 Ethereum 主网。([zkEVM](https://zkevm.ethereum.foundation/?utm_source=chatgpt.com))

如果这个方向成熟，Ethereum 的角色会进一步变化：

从“所有节点重复执行所有计算”，  
走向“节点更高效地验证证明”。

* * *

## 方向四：ZK 身份与隐私应用

这是和 ENS、DID、DAO、AI identity 关系最密切的方向。

未来可能出现更多：

privacy-preserving identity  
匿名但可信的 DAO 投票  
隐藏钱包历史的 reputation  
隐私保护的社交图谱  
可证明的人类身份  
AI agent 权限证明

这条线很适合你观察，因为它不只是技术，而是直接关系到社区、治理、身份和信任。

* * *

## 方向五：ZK 进入 Web2 / 传统行业

比如：

金融合规  
医疗数据  
学历证明  
年龄证明  
信用证明  
企业数据审计  
AI 模型验证

未来 ZK 不一定只属于 Crypto。

它可能成为更广义的“可信互联网”基础设施。
<!-- DAILY_CHECKIN_2026-04-30_END -->

# 2026-04-29
<!-- DAILY_CHECKIN_2026-04-29_START -->


明日复明日，明日何其多
<!-- DAILY_CHECKIN_2026-04-29_END -->

# 2026-04-28
<!-- DAILY_CHECKIN_2026-04-28_START -->



唉 再请假
<!-- DAILY_CHECKIN_2026-04-28_END -->

# 2026-04-26
<!-- DAILY_CHECKIN_2026-04-26_START -->




# 🧠 Two different models: Ethereum vs Bitcoin

Ethereum and Bitcoin use two different ways to record state.  
以太坊和比特币用两种不同的方式记录状态。

* * *

Ethereum uses the account model.  
以太坊使用“账户模型”。

* * *

Bitcoin uses the UTXO model.  
比特币使用“UTXO 模型”。

* * *

👉 Key idea:

Account vs UTXO  
账户 vs 未花费输出

* * *

* * *

# 🧩 What is UTXO (simple idea)

* * *

UTXO means “Unspent Transaction Output”.  
UTXO 的意思是“未花费的交易输出”。

* * *

In Bitcoin, you don’t have a balance.  
在比特币中，你其实没有“余额”。

* * *

You have a set of UTXOs.  
你拥有的是一组 UTXO。

* * *

Each UTXO is like a coin.  
每一个 UTXO 就像一枚硬币。

* * *

👉 Example:

You may have:

-   0.3 BTC  
    
-   0.5 BTC  
    

你可能拥有：

-   一个 0.3 BTC 的 UTXO  
    
-   一个 0.5 BTC 的 UTXO  
    

* * *

👉 Total = 0.8 BTC  
👉 总和才是你的“余额”

* * *

* * *

# 🧠 How a BTC transaction works

* * *

To send BTC, you must consume UTXOs.  
发送 BTC 时，你必须“花掉”已有的 UTXO。

* * *

You cannot partially spend one directly.  
你不能直接“部分使用”一个 UTXO。

* * *

👉 So:

You take inputs → create outputs  
你用输入 → 生成新的输出

* * *

* * *

### Example:

You have one UTXO of 1 BTC.  
你有一个 1 BTC 的 UTXO。

* * *

You want to send 0.3 BTC.  
你想发送 0.3 BTC。

* * *

Transaction will:

-   use 1 BTC as input  
    
-   create 0.3 BTC to receiver  
    
-   create 0.7 BTC back to you (change)  
    

交易会：

-   用掉 1 BTC 输入  
    
-   生成 0.3 BTC 给对方  
    
-   生成 0.7 BTC 找零给你  
    

* * *

👉 Key idea:

BTC = consume and recreate  
BTC = 消耗旧的，生成新的

* * *

* * *

# 🧠 What is Ethereum account model

* * *

Ethereum uses accounts with balances.  
以太坊使用账户 + 余额的模型。

* * *

Each account has:

-   balance  
    
-   nonce  
    
-   storage  
    

每个账户有：

-   余额  
    
-   nonce  
    
-   存储  
    

* * *

👉 Example:

You have 1 ETH in your account.  
你账户里有 1 ETH。

* * *

You send 0.3 ETH.  
你发送 0.3 ETH。

* * *

Result:

Your balance becomes 0.7 ETH.  
你的余额变成 0.7 ETH。

* * *

👉 No need to recreate outputs  
👉 不需要“拆分再生成”

* * *

* * *

# ⚖️ Core difference（核心区别）

* * *

Bitcoin:

State = set of UTXOs  
状态 = 一组 UTXO

* * *

Ethereum:

State = account balances + storage  
状态 = 账户余额 + 存储

* * *

* * *

👉 One sentence:

BTC tracks coins  
ETH tracks accounts

BTC 跟踪“币”  
ETH 跟踪“账户”

* * *

* * *

# 🧠 Deeper intuition（很重要）

* * *

UTXO model is stateless per transaction.  
UTXO 模型在每笔交易层面是“无状态”的。

* * *

Each transaction only cares about inputs and outputs.  
每笔交易只关心输入和输出。

* * *

👉 Like cash:

You pay with coins.  
像现金一样，用硬币支付。

* * *

* * *

Ethereum is stateful.  
以太坊是“有状态”的。

* * *

Each transaction changes global state.  
每一笔交易都会改变全局状态。

* * *

👉 Like bank account:

You update your balance.  
像银行账户，直接更新余额。

* * *

* * *

# 🔗 Why Ethereum chose account model

* * *

Ethereum needs to support smart contracts.  
以太坊需要支持智能合约。

* * *

Smart contracts require persistent state.  
智能合约需要持续存在的状态。

* * *

👉 Example:

ENS needs to store:

-   ownership  
    
-   records  
    
-   mappings  
    

ENS 需要存储：

-   所有权  
    
-   解析记录  
    
-   映射关系  
    

* * *

👉 UTXO is not good for this  
👉 UTXO 不适合这种复杂状态

* * *

* * *

# 🔗 Connect back to what you learned

* * *

Ethereum = global state machine  
以太坊 = 全球状态机

* * *

Account model fits this perfectly.  
账户模型完美匹配这个结构。

* * *

👉 Because:

Transactions modify state directly  
交易直接修改状态

* * *

* * *

# ❤️ Final mental model（最重要）

* * *

Bitcoin:

Transactions move coins  
交易移动“币”

* * *

Ethereum:

Transactions change state  
交易改变“状态”
<!-- DAILY_CHECKIN_2026-04-26_END -->

# 2026-04-24
<!-- DAILY_CHECKIN_2026-04-24_START -->





又没学
<!-- DAILY_CHECKIN_2026-04-24_END -->

# 2026-04-23
<!-- DAILY_CHECKIN_2026-04-23_START -->






疲惫，休息，明天继续
<!-- DAILY_CHECKIN_2026-04-23_END -->

# 2026-04-22
<!-- DAILY_CHECKIN_2026-04-22_START -->







## Layer 1 、Layer2

# 🧠 What is Layer 1?

Layer 1 is the base blockchain itself.  
Layer 1 是区块链本身的基础层。

* * *

For Ethereum, Layer 1 is Ethereum mainnet.  
对以太坊来说，Layer 1 就是以太坊主网。

* * *

It includes:

-   Execution Layer（执行层）  
    
-   Consensus Layer（共识层）  
    

它包含：

-   执行层  
    
-   共识层  
    

* * *

Layer 1 is where the final truth lives.  
Layer 1 是“最终真相”所在的地方。

* * *

👉 Key idea:

Layer 1 = source of truth  
Layer 1 = 真相来源

* * *

# 🧠 What is Layer 2?

Layer 2 is built on top of Layer 1 to improve scalability.  
Layer 2 是构建在 Layer 1 之上的扩展层，用来提升性能。

* * *

It processes transactions off the main chain.  
它把交易放到主链之外处理。

* * *

But it still relies on Layer 1 for security.  
但它仍然依赖 Layer 1 来保证安全。

* * *

👉 Key idea:

Layer 2 = faster, cheaper, but anchored to L1  
Layer 2 = 更快、更便宜，但依附于 L1

* * *

# 🧭 Why do we need Layer 2?

* * *

Layer 1 is secure but expensive and slow.  
Layer 1 很安全，但成本高、速度慢。

* * *

Because every node must process every transaction.  
因为每个节点都要处理每一笔交易。

* * *

👉 That limits scalability.

👉 这限制了扩展性。

* * *

Layer 2 solves this by moving computation away.  
Layer 2 通过把计算移到链外来解决这个问题。

* * *

👉 So:

L1 = security  
L2 = scalability

L1 = 安全  
L2 = 扩展性

* * *

# 🧩 How Layer 2 works (simple version)

* * *

Users send transactions to Layer 2.  
用户把交易发送到 Layer 2。

* * *

Layer 2 executes many transactions off-chain.  
Layer 2 在链外执行大量交易。

* * *

Then it sends a summary back to Layer 1.  
然后把结果“打包摘要”发回 Layer 1。

* * *

Layer 1 verifies or accepts this result.  
Layer 1 验证或确认这个结果。

* * *

👉 Key idea:

L2 computes, L1 verifies  
L2 计算，L1 验证

* * *

# 🔍 Two common types (轻微了解就够)

* * *

Optimistic Rollups assume transactions are correct unless challenged.  
Optimistic Rollup 默认交易是正确的，除非有人提出挑战。

* * *

ZK Rollups prove correctness with math proofs.  
ZK Rollup 用数学证明来保证正确性。

* * *

👉 不需要深入，只要记：

Different L2s = different ways to prove correctness  
不同 L2 = 不同的“证明正确性”的方式

* * *

# 🧭 Relationship between L1 and L2

* * *

Layer 2 cannot exist without Layer 1.  
Layer 2 离不开 Layer 1。

* * *

Layer 1 provides security and finality.  
Layer 1 提供安全和最终确认。

* * *

Layer 2 provides speed and low cost.  
Layer 2 提供速度和低成本。

* * *

👉 One sentence:

L2 scales Ethereum, L1 secures it  
L2 扩展以太坊，L1 保护它

* * *

# 🧠 Connect back to what you learned

* * *

Execution Layer (EL) runs transactions.  
执行层负责执行交易。

* * *

Consensus Layer (CL) finalizes them.  
共识层负责最终确认。

* * *

👉 Now add L2:

L2 does execution faster  
L1 still provides final consensus

L2 更快执行  
L1 仍然提供最终共识

* * *

👉 所以：

L2 is like an extension of EL  
L2 有点像“外包的执行层”

* * *

But final truth still comes from L1  
但最终真相仍然来自 L1
<!-- DAILY_CHECKIN_2026-04-22_END -->

# 2026-04-20
<!-- DAILY_CHECKIN_2026-04-20_START -->








HK Web3 Festival 嘉年华感受熊市，太熊了，好难受

# 🧠 One Transaction = EL + CL Cooperation

A full transaction lifecycle is a cooperation between EL and CL.  
一笔交易的完整生命周期，是执行层（EL）和共识层（CL）的协作。

* * *

# 🧭 Full Lifecycle（完整生命周期）

* * *

## 1️⃣ User creates a transaction（链外开始）

A user creates and signs a transaction.  
用户创建并签名一笔交易。

* * *

This is just an intent, nothing has happened on-chain yet.  
这只是一个“意图”，链上还没有发生任何事情。

* * *

👉 关键理解：

Nothing is real yet.  
👉 此时还没有“真实发生”。

* * *

## 2️⃣ Transaction enters the network（进入系统）

The transaction is broadcast to the network.  
交易被广播到网络中。

* * *

Nodes check basic validity (signature, nonce, balance).  
节点检查基本合法性（签名、nonce、余额）。

* * *

👉 这一步：

Still not final, still not executed.  
👉 还没执行，也没有被确认。

* * *

## 3️⃣ CL selects a proposer（CL第一次介入）

Consensus Layer selects a validator to propose a block.  
共识层选择一个验证者来出块。

* * *

👉 关键点：

CL decides who has the right to act.  
👉 共识层决定“谁有权出手”。

* * *

## 4️⃣ Block is built（EL内容被组织）

The proposer selects transactions from the mempool.  
出块者从 mempool 选择交易。

* * *

They build a block with these transactions.  
把这些交易打包成一个区块。

* * *

👉 关键点：

Content = EL  
Authority = CL

内容属于 EL  
权力来自 CL

* * *

## 5️⃣ EL executes transactions（核心执行）

Execution Layer runs all transactions inside the block.  
执行层执行区块中的所有交易。

* * *

EVM processes them step by step.  
EVM 一步一步执行。

* * *

State is updated accordingly.  
状态随之改变。

* * *

👉 关键点：

Now things actually happen.  
👉 到这里，“事情真正发生了”。

* * *

## 6️⃣ CL verifies the block（共识验证）

Other validators check the block.  
其他验证者检查这个区块。

* * *

They verify both:

-   correctness (EL result)  
    
-   proposer behavior  
    

他们会验证：

-   执行结果是否正确（EL）  
    
-   出块者是否正常  
    

* * *

👉 关键点：

CL checks EL’s work.  
👉 共识层在“审核执行层”。

* * *

## 7️⃣ Block is finalized（最终确认）

The block gets finalized through consensus.  
区块通过共识机制被最终确认。

* * *

After this, it cannot be reversed.  
之后就不可逆了。

* * *

👉 关键点：

Now it becomes truth.  
👉 到这里，它才成为“真相”。

* * *

# 🎯 最核心的理解（这一句非常重要）

* * *

Execution makes it happen.  
执行让事情发生。

* * *

Consensus makes it permanent.  
共识让结果变成永久。
<!-- DAILY_CHECKIN_2026-04-20_END -->

# 2026-04-19
<!-- DAILY_CHECKIN_2026-04-19_START -->









今天周日 想休息

下周好悬啊，一周HK
<!-- DAILY_CHECKIN_2026-04-19_END -->

# 2026-04-17
<!-- DAILY_CHECKIN_2026-04-17_START -->










04/17 继续降维学习

# 🧠 Consensus Layer（CL）— Human Version

* * *

## 1️⃣ What is Consensus Layer?

Consensus Layer is the part of Ethereum that decides which blocks are accepted.  
共识层是以太坊中负责决定“哪些区块被接受”的那一层。

* * *

It answers one core question:  
“Which version of the blockchain is the true one?”  
它解决一个核心问题：  
👉 哪一条链才是真正的链？

* * *

Without CL, everyone could have different versions.  
如果没有共识层，每个人看到的链可能都不一样。

* * *

👉 So:

CL creates shared truth.  
共识层创造“共同认可的真相”。

* * *

## 2️⃣ What does CL actually do?

CL coordinates validators to agree on blocks.  
共识层协调验证者对区块达成一致。

* * *

It decides:

-   who proposes a block
    
-   who checks the block
    
-   when the block is finalized  
    

它决定：

-   谁来出块
    
-   谁来验证
    
-   什么时候最终确认  
    

* * *

👉 Simple idea:

CL = coordination + agreement  
CL = 协调 + 共识

* * *

## 3️⃣ Who are validators?

Validators are participants who secure the network.  
验证者是维护网络安全的参与者。

* * *

They stake ETH to join the system.  
他们需要质押 ETH 才能参与。

* * *

If they behave honestly, they earn rewards.  
如果行为诚实，会获得奖励。

* * *

If they cheat, they get punished.  
如果作弊，会被惩罚（slash）。

* * *

👉 Key idea:

Validators = decision makers  
验证者 = 决策参与者

* * *

## 4️⃣ How a block is created (CL view)

* * *

CL selects a validator randomly.  
共识层随机选择一个验证者。

* * *

This validator becomes the proposer.  
这个验证者成为出块者。

* * *

The proposer builds a block with transactions.  
出块者把交易打包成区块。

* * *

Other validators check the block.  
其他验证者检查这个区块。

* * *

If enough validators agree, the block is accepted.  
如果足够多的验证者同意，这个区块就被接受。

* * *

👉 Simple flow:

Select → Propose → Validate → Accept  
选择 → 出块 → 验证 → 接受

* * *

## 5️⃣ What is finality?

Finality means the block cannot be changed anymore.  
最终性（finality）意味着区块不可再被更改。

* * *

Before finality, a block might be replaced.  
在最终确认前，区块有可能被替换。

* * *

After finality, it becomes permanent.  
一旦最终确认，就永久不可逆。

* * *

👉 Simple idea:

Finality = irreversible truth  
最终性 = 不可改变的真相

* * *

## 6️⃣ Why staking is important

Validators must lock ETH as stake.  
验证者必须锁定 ETH 作为质押。

* * *

This gives them skin in the game.  
这让他们“有代价、有责任”。

* * *

If they try to cheat, they lose money.  
如果他们作弊，会损失资金。

* * *

👉 So:

Security comes from economic incentives.  
安全性来自经济激励机制。

* * *

## 7️⃣ Why CL exists separately

* * *

Execution is complex (EVM, contracts).  
执行很复杂（EVM、合约等）。

* * *

Consensus is also complex (agreement, timing).  
共识也复杂（协调、时间机制）。

* * *

Separating them makes both simpler.  
把它们分开，让两边都更简单。

* * *

👉 This is a major upgrade in Ethereum design.  
👉 这是以太坊设计中的一个重要升级。

* * *

## 8️⃣ What CL does NOT do

\==CL does not execute smart contracts.==  
共识层不执行智能合约。

* * *

CL does not compute balances.  
共识层不计算余额。

* * *

👉 That is EL’s job.  
👉 那是执行层的工作。

* * *

## 🧭 Final mental model（最重要）

* * *

CL selects who can propose blocks.  
CL 选择谁来出块。

* * *

EL executes the transactions inside the block.  
EL 执行区块中的交易。

* * *

CL confirms and finalizes the block.  
CL 确认并最终确定这个区块。

* * *

👉 One sentence:

CL decides truth, EL produces results.  
CL 决定“真相”，EL 产生“结果”。
<!-- DAILY_CHECKIN_2026-04-17_END -->

# 2026-04-16
<!-- DAILY_CHECKIN_2026-04-16_START -->











04/16 继续降维学习

EL vs CL — The Real Separation

* * *

## 1️⃣ Why do we even have two layers?

Ethereum used to be one system, but now it is split into two layers.  
以太坊以前是一个整体系统，现在被拆成了两层。

This split makes the system more modular and scalable.  
这种拆分让系统更模块化，也更容易扩展。

* * *

Execution Layer handles “what happens”.  
执行层负责“发生什么”。

Consensus Layer handles “who decides”.  
共识层负责“谁说了算”。

* * *

👉 This separation is very important.

It allows Ethereum to evolve faster and safer.  
这个分层非常重要，让以太坊可以更快、更安全地演进。

* * *

## 2️⃣ What does EL do again?

Execution Layer executes transactions and updates the state.  
执行层执行交易，并更新状态。

* * *

It does not care who created the block.  
它不关心是谁创建了区块。

* * *

It only cares:  
“Are these transactions valid, and what is the new state?”  
它只关心：这些交易是否合法，以及新的状态是什么。

* * *

👉 EL is like a calculator.

You give it input, it gives you the result.  
执行层就像计算器，你给输入，它给结果。

* * *

## 3️⃣ What does CL do?

Consensus Layer decides who can propose and validate blocks.  
共识层决定谁可以出块、谁来验证区块。

* * *

It runs mechanisms like staking and validator selection.  
它运行质押、验证者选择等机制。

* * *

It ensures everyone agrees on the same chain.  
它确保所有人对同一条链达成一致。

* * *

👉 CL is about agreement.

Not computation, but coordination.  
共识层是关于“达成一致”，不是计算，而是协调。

* * *

## 4️⃣ How EL and CL work together

* * *

CL selects a validator to propose a block.  
共识层选择一个验证者来出块。

* * *

The validator builds a block with transactions.  
验证者把交易打包成一个区块。

* * *

EL executes those transactions and computes the new state.  
执行层执行这些交易，并计算出新的状态。

* * *

CL then verifies and finalizes the block.  
共识层再来验证并最终确认这个区块。

* * *

👉 Simple flow:

CL → choose proposer  
EL → execute transactions  
CL → finalize block

共识层选人 → 执行层计算 → 共识层确认

* * *

## 5️⃣ Why this separation matters

* * *

It reduces complexity in each layer.  
这种拆分降低了每一层的复杂度。

* * *

Developers can improve EL without touching CL.  
开发者可以优化执行层，而不用改共识层。

* * *

And vice versa.  
反之亦然。

* * *

It also allows different implementations.  
也允许不同的客户端实现。

* * *

👉 This is why Ethereum is resilient.

因为模块化，让系统更有韧性。

* * *

## 6️⃣ A simple analogy (很好用)

* * *

Think of a company.

把以太坊想象成一家公司。

* * *

EL is the “operations team”.  
执行层是“执行团队”。

They do the actual work.  
他们负责干活。

* * *

CL is the “governance team”.  
共识层是“决策/治理团队”。

They decide who is in charge and what is accepted.  
他们决定谁有权、什么被认可。

* * *

👉 Without EL → nothing gets done  
没有执行层 → 什么都不会发生

👉 Without CL → chaos, no agreement  
没有共识层 → 一片混乱，没有共识

* * *

## 7️⃣ One key sentence (非常重要)

* * *

Ethereum works because execution and consensus are separated but coordinated.  
以太坊之所以能运转，是因为执行和共识分离，但又协同。

* * *

## 8️⃣ Why this matters for YOU（这才是关键）

* * *

ENS runs on Ethereum, so it depends on both layers.  
ENS 运行在以太坊上，因此依赖这两层。

* * *

When you register a name, it is a transaction → handled by EL.  
当你注册 ENS 名字，这是一个交易 → 执行层处理。

* * *

But whether that transaction is accepted → decided by CL.  
但这个交易是否被确认 → 由共识层决定。

* * *

👉 So:

EL = execution of ENS logic  
CL = security and finality of ENS

EL = ENS 的执行  
CL = ENS 的安全与确认
<!-- DAILY_CHECKIN_2026-04-16_END -->

# 2026-04-15
<!-- DAILY_CHECKIN_2026-04-15_START -->












04/15 EL 协议规范，对我来说太难了，远远超纲，所以 降维学习

🧠 Execution Layer（EL）— Human Version // 降维学习

* * *

# 🧠 Execution Layer（EL）

* * *

## 1️⃣ What is Execution Layer?

Execution Layer is the part of Ethereum that runs transactions and updates the state.  
执行层是以太坊中负责运行交易、并更新整个系统状态的部分。

It is like the “engine” of Ethereum that actually does the work.  
它就像以太坊的“引擎”，真正干活的地方。

When someone sends a transaction, EL is the place where it gets executed.  
当有人发送交易时，就是在执行层被真正执行。

So if you ask “what really happened on-chain”, the answer is inside EL.  
所以如果你问“链上到底发生了什么”，答案就在执行层。

* * *

## 2️⃣ What is “state” in Ethereum?

State means all the current data of the blockchain.  
状态就是区块链当前所有的数据。

It includes account balances, smart contract code, and stored data.  
包括账户余额、智能合约代码、以及存储的数据。

You can imagine state as a global database shared by all nodes.  
你可以把状态想象成一个所有节点共享的“全局数据库”。

Every transaction will try to change this database.  
每一笔交易，都会尝试去改变这个数据库。

* * *

👉 One key idea:

A transaction is just a state change.  
一笔交易，本质上就是一次状态改变。

* * *

## 3️⃣ What does EL actually do?

EL takes transactions and applies them to the current state.  
执行层接收交易，并把它们作用到当前状态上。

It processes them one by one, in a specific order.  
它按顺序一笔一笔执行交易。

After execution, the state becomes a new state.  
执行完成后，状态就变成了新的状态。

This process is called “state transition”.  
这个过程叫做“状态转换”。

* * *

👉 Very simple flow:

Old state → Transactions → Execution → New state  
旧状态 → 交易 → 执行 → 新状态

* * *

## 4️⃣ How a transaction works (step by step)

* * *

A user creates a transaction from a wallet.  
用户从钱包发起一笔交易。

This transaction may transfer ETH or call a smart contract.  
这笔交易可能是转账，也可能是调用智能合约。

* * *

The network first checks if the transaction is valid.  
网络首先会检查这笔交易是否有效。

It checks signature, nonce, and gas.  
会检查签名、nonce（防止重复）、以及 gas 是否合理。

* * *

Then the transaction is included into a block.  
接着这笔交易会被打包进一个区块。

* * *

Inside the block, the EVM executes the transaction.  
在区块中，EVM 会执行这笔交易。

* * *

Execution may change balances or contract storage.  
执行过程中可能改变账户余额或合约存储。

* * *

Finally, the state is updated, and gas is paid.  
最终状态被更新，同时用户支付 gas 费用。

* * *

👉 One sentence summary:

A transaction goes from “intent” to “state change”.  
一笔交易，从“意图”变成“状态改变”。

* * *

## 5️⃣ What is EVM?

EVM is the virtual machine of Ethereum.  
EVM 是以太坊的虚拟机。

It is like a global computer that runs smart contracts.  
它就像一个全球共享的计算机，专门运行智能合约。

* * *

Every node runs the same EVM computation.  
每个节点都会运行相同的 EVM 计算。

This ensures that everyone gets the same result.  
这保证了所有人得到相同的结果。

* * *

EVM does not trust anyone, it just follows rules.  
EVM 不相信任何人，它只按照规则执行。

* * *

👉 Key idea:

EVM = deterministic execution  
EVM = 确定性执行（所有人算出来都一样）

* * *

## 6️⃣ Why do we need Gas?

Gas is used to measure computation.  
Gas 用来衡量计算量。

Every operation in EVM costs gas.  
EVM 中的每一个操作都要消耗 gas。

* * *

More complex operations cost more gas.  
越复杂的操作，消耗越多 gas。

* * *

Gas also prevents spam.  
Gas 还能防止垃圾攻击。

If computation were free, people could overload the network.  
如果计算免费，网络会被无限滥用。

* * *

Users must pay for what they use.  
用户必须为自己使用的资源付费。

* * *

👉 Simple understanding:

Gas is the price of using Ethereum.  
Gas 就是使用以太坊的价格。

* * *

## 7️⃣ ==What is a block in EL?

A block is a list of transactions grouped together.  
一个区块就是一组被打包在一起的交易。

* * *

EL executes all transactions inside the block.  
执行层会执行区块里的所有交易。

* * *

Transactions are executed in order.  
交易是按顺序执行的。

Order matters, because earlier transactions can change the state.  
顺序很重要，因为前面的交易会影响后面的状态。

* * *

After all transactions are executed, we get a new state.  
所有交易执行完后，我们得到新的状态。

* * *

👉 Key idea:

Block = a batch of state transitions  
区块 = 一批状态转换

* * *

## 8️⃣ What EL does NOT do

Execution Layer does not decide who creates blocks.  
执行层不决定谁来出块。

* * *

Execution Layer does not handle consensus.  
执行层不负责共识。

* * *

That is the job of the Consensus Layer.  
这些是共识层（CL）的职责。

* * *

👉 Important separation:

EL decides “what happens”  
CL decides “who decides”

EL 决定“发生什么”  
CL 决定“谁说了算”

* * *

## 🧭 Final mental model（最重要的整体理解）

* * *

User sends a transaction, and EL executes it using EVM, then updates the state.  
用户发送交易，执行层用 EVM 执行它，然后更新状态。

* * *

Blocks are just containers of transactions.  
区块只是交易的容器。

* * *

Ethereum is basically a state machine.  
以太坊本质上是一个状态机。

* * *

Each block moves the system from one state to another.  
每一个区块，都让系统从一个状态变到另一个状态。

* * *

👉 最核心一句

Ethereum is a global state machine, and EL is the part that runs the machine.  
以太坊是一个全球状态机，而执行层就是让这个机器运转的部分。

* * *
<!-- DAILY_CHECKIN_2026-04-15_END -->

# 2026-04-14
<!-- DAILY_CHECKIN_2026-04-14_START -->













# [Execution Layer Specification](https://epf.wiki/#/wiki/EL/el-specs?id=execution-layer-specification)

看的很晕，纠结要不要放弃？
<!-- DAILY_CHECKIN_2026-04-14_END -->

# 2026-04-12
<!-- DAILY_CHECKIN_2026-04-12_START -->














04/12

Protocol History and Evolution

## 一、整体理解：协议不是一次设计，而是不断演化

**EN:** Ethereum protocol is not designed once and fixed, but evolves through continuous upgrades.  
**中:** 以太坊协议并不是一次性设计完成的系统，而是在实践中不断迭代升级的过程。

这一部分的核心不是记住每个升级细节，而是理解：  
👉 Ethereum 是一个“live system”（活系统），通过 **EIPs + hard fork** 持续演进。  
👉 每一阶段都在解决“上一阶段暴露的问题”，逐步走向更稳定、更复杂的系统。

* * *

## 二、Frontier：起点是“可运行”，不是“完美”

**EN:** Frontier was a beta launch focused on enabling developers to experiment and build.  
**中:** Frontier 是一个偏实验性质的 beta 版本，重点是“让系统跑起来”。

Frontier（2015）标志着 Ethereum 主网上线，它的设计思路非常务实：  
不是追求完美，而是先让开发者可以部署合约、开始实验。

关键特点包括：

-   Gas limit 很低（5000），用于控制系统风险
    
-   允许开发者开始构建 dApp 和工具
    
-   引入 “canary contracts”（金丝雀机制）用于升级协调  
    

👉 **理解重点：**  
Frontier 本质是一个“可启动系统（bootstrapping phase）”，  
目标是：让网络活起来，而不是优化体验或性能。

* * *

## 三、Homestead：从“能用”到“更可靠”

**EN:** Homestead marked Ethereum’s transition from experimental to more stable and mature.  
**中:** Homestead 标志着以太坊从实验阶段进入更稳定、可用的阶段。

Homestead（2016）开始处理 Frontier 中暴露的现实问题，重点在于：  
👉 提升系统可靠性、安全性、以及规则的严谨性

几个关键改进 ：

1\. Gas 机制调整，从“鼓励尝试” → “开始约束行为”

2\. 交易签名安全（malleability 修复），提升交易不可篡改性（integrity）

3\. Out-of-gas 处理更严格，系统从“宽松容错” → “明确失败机制”

4\. Difficulty 调整算法优化，开始关注系统运行的稳定性（performance consistency）

5\. 新 opcode：DELEGATECALL（EIP-7），支持更灵活的合约调用结构（如代理模式）。

6\. 网络协议前向兼容（EIP-8）

* * *

## 四、The Merge：架构级跃迁（不是小修小补）

**EN:** The Merge replaced Proof-of-Work with Proof-of-Stake.  
**中:** The Merge 将共识机制从 PoW 切换为 PoS。

The Merge（2022）不是“优化”，而是一次**架构级重构**：Ethereum 从“单层系统” → “模块化分层系统”

### 核心变化

-   从 PoW → PoS（不再需要挖矿）
    
-   引入 Beacon Chain（独立的共识层）
    
-   执行层（Execution）与共识层（Consensus）分离，协议可以独立演进不同层，为未来扩展（如 rollups、Danksharding）打基础  
    

* * *

## 五、总结：从历史看到设计逻辑

### 1️⃣ Frontier

👉 目标：让系统存在（make it exist）

### 2️⃣ Homestead

👉 目标：让系统可靠（make it reliable）

### 3️⃣ The Merge

👉 目标：让系统可扩展（make it scalable & modular）
<!-- DAILY_CHECKIN_2026-04-12_END -->

# 2026-04-11
<!-- DAILY_CHECKIN_2026-04-11_START -->















04/11

还得再休一天，睡觉更重要

明天一定
<!-- DAILY_CHECKIN_2026-04-11_END -->

# 2026-04-10
<!-- DAILY_CHECKIN_2026-04-10_START -->
















04/10

今天折腾网络，休息下

周末补
<!-- DAILY_CHECKIN_2026-04-10_END -->

# 2026-04-08
<!-- DAILY_CHECKIN_2026-04-08_START -->

















04/08

### 🧭 Ethereum Protocol Design Philosophy

* * *

## 一、核心设计理念（Core Design Philosophy）

Ethereum 的设计不是围绕某一个具体应用，而是围绕一组长期有效的原则展开。这些原则决定了以太坊为何成为一个开放、可扩展的通用平台。

Ethereum is not designed for a specific application, but around a set of long-term principles that shape it into an open and extensible platform.

* * *

### 1\. 简单性（Simplicity）

以太坊在设计之初就强调简单性，希望普通开发者也能够理解协议，从而减少对少数精英开发者的依赖。

Ethereum aims to keep the protocol as simple as possible so that average developers can understand it, reducing reliance on a small group of experts.

👉 核心理解：  
简单意味着更容易被理解，从而更有利于去中心化。

Simplicity enables accessibility, which supports decentralization.

* * *

### 2\. 通用性（Universality）

以太坊并不内置具体功能，而是提供一个图灵完备的虚拟机（EVM），允许开发者构建任意类型的应用。

Ethereum does not define specific features but provides a Turing-complete virtual machine (EVM) to support arbitrary applications.

👉 核心理解：  
以太坊不是一个应用，而是一个平台。

Ethereum is not an application, but a platform.

* * *

### 3\. 模块化（Modularity）

以太坊采用模块化设计，使不同部分可以独立演进和升级，而不影响整体系统运行。

Ethereum is designed in a modular way so that components can evolve independently without breaking the system.

👉 核心理解：  
模块化使系统更具长期适应能力。

Modularity enables long-term adaptability.

* * *

### 4\. 非歧视性（Non-discriminant）

以太坊不限制用户的使用方式，不对应用类型进行判断或干预，而是通过 gas 机制来调节资源使用。

Ethereum does not restrict use cases and does not judge applications. Resource usage is regulated through gas fees.

👉 核心理解：  
协议保持中立，只约束资源消耗。

The protocol remains neutral and only regulates resource usage.

* * *

### 5\. 灵活性（Agility）

以太坊的设计允许持续改进，通过 EIP（Ethereum Improvement Proposal）机制不断演化。

Ethereum is designed to evolve over time through mechanisms like Ethereum Improvement Proposals (EIPs).

👉 核心理解：  
协议不是静态的，而是不断发展的。

The protocol is not fixed but continuously evolving.

* * *

## 二、设计原则（Design Principles）

除了核心理念之外，以太坊还遵循一系列具体设计原则，以应对系统复杂性和长期发展问题。

* * *

### 1\. 管理复杂性（Managing Complexity）

以太坊的目标之一是尽量降低复杂性，同时保证系统功能完整。

Ethereum aims to minimize complexity while maintaining full functionality.

* * *

（1）夹心模型（Sandwich Model）

系统的底层（核心协议）和用户接口应尽量简单，而复杂性被放置在中间层。

The lowest layer and user-facing interface should be simple, while complexity is pushed into middle layers.

👉 核心理解：  
用户看到简单，复杂被隐藏。

Keep the edges simple, push complexity inward.

* * *

（2）封装复杂性（Encapsulated Complexity）

系统内部可以复杂，但对外必须提供清晰简单的接口。

Internal complexity is acceptable if it is properly encapsulated behind simple interfaces.

👉 核心理解：  
复杂可以存在，但必须被良好封装。

Complexity should be hidden behind abstraction.

* * *

### 2\. 自由（Freedom）

用户可以自由使用以太坊，协议不应限制用途或偏好某类应用。

Users should be free to use Ethereum without restrictions or preferences.

👉 核心理解：  
协议不做价值判断。

The protocol should not enforce value judgments.

* * *

### 3\. 泛化（Generalization）

以太坊更倾向于提供底层原语，而不是直接实现具体功能。

Ethereum focuses on providing low-level primitives instead of high-level features.

👉 核心理解：  
提供“积木”，而不是“成品”。

Provide building blocks, not finished products.

* * *

### 4\. 无功能原则（We Have No Features）

以太坊尽量避免将高层功能直接写入协议，而是交由智能合约实现。

Ethereum avoids embedding high-level features into the protocol, leaving them to smart contracts.

👉 核心理解：  
功能属于应用层，而不是协议层。

Features belong to the application layer, not the protocol.

* * *

## 三、区块链层设计（Blockchain-Level Design）

在具体实现上，以太坊在数据模型、结构和网络方面也体现了这些哲学。

* * *

### 1\. 账户模型（Account Model）

以太坊采用账户模型，而不是比特币的 UTXO 模型，使系统更简单和灵活。

Ethereum uses an account-based model instead of Bitcoin’s UTXO model, making it simpler and more flexible.

* * *

### 2\. 状态结构（Merkle Patricia Trie）

以太坊使用 MPT 来存储状态，使数据具有可验证性。

Ethereum uses a Merkle Patricia Trie to store state, ensuring cryptographic verifiability.

👉 核心理解：  
每一个状态变化都可以被验证。

State is verifiable through cryptographic proofs.

* * *

### 3\. Verkle Tree（未来优化）

Verkle tree 被提出作为替代方案，以提高效率并支持无状态客户端。

Verkle trees are proposed to improve efficiency and support statelessness.

* * *

### 4\. 数据序列化（RLP & SSZ）

RLP 用于早期数据编码，而 SSZ 在以太坊 2.0 中提供更高效的结构化序列化方式。

RLP is used for encoding, while SSZ improves efficiency and structure in Ethereum 2.0.

* * *

### 5\. 最终性（Finality）

通过 Casper FFG 和 LMD-GHOST，共识机制可以确保区块最终不可回滚。

Finality ensures that blocks cannot be reverted once confirmed, using Casper FFG and LMD-GHOST.

* * *

### 6\. 网络机制（Networking）

以太坊网络结合 DHT（节点发现）和 Gossip（数据传播）来实现高效通信。

Ethereum uses DHT for peer discovery and gossip protocols for data propagation.

👉 核心理解：  
DHT 用来“找节点”，Gossip 用来“传数据”。

* * *

## 四、总结（Summary）

以太坊的设计可以归纳为以下核心特征：

Ethereum’s design can be summarized as:

> **Simple, General, Modular, and Permissionless**

简单、通用、模块化、无许可。

* * *
<!-- DAILY_CHECKIN_2026-04-08_END -->

# 2026-04-07
<!-- DAILY_CHECKIN_2026-04-07_START -->


















04/07

## 🧩 核心结构（Core Structure）

以太坊被拆分为两层：  
**执行层（Execution Layer, EL）**  
**共识层（Consensus Layer, CL）**

👉 分离：  
**执行（what happens） vs 共识（what is agreed）**

* * *

## 🧮 执行层（Execution Layer, EL）

所有交易真正发生的地方  
👉 执行逻辑 + 数据变化  
👉 Global Computer

~ **EVM（Ethereum Virtual Machine）**  
执行智能合约的“虚拟计算机”

~ **State（状态）**  
存储所有链上数据（余额、合约）

~ **Transactions（交易池 / mempool）**  
等待被打包的交易

~ **p2p network**  
传播交易与区块数据

* * *

## 🤝 共识层（Consensus Layer, CL）

决定哪条链是“正确的”  
👉 验证 + 选择 + 最终确认  
👉 Global Agreement

~ **Validators（验证者）**  
负责出块和投票（质押ETH）

~ **PoS（Proof of Stake）**  
基于质押的共识机制

~ **Fork Choice（LMD-GHOST）**  
决定当前主链是哪条

~ **Finality（Casper FFG）**  
确定区块不可回滚

~ **RANDAO**  
生成随机数（用于选择验证者）

~ **p2p network**  
传播区块与投票

* * *

## 🔌 层间连接（Layer Communication）

**Engine API（核心桥梁）**

~ 共识层 → 执行层  
“执行这个区块”

~ 执行层 → 共识层  
“执行结果是这样”

👉 Execution ↔ Consensus

* * *

## 🌐 对外接口（Interfaces）

~ **JSON-RPC API**  
用户 / dApp → 执行层  
（发送交易、调用合约）

~ **Beacon API**  
验证者 → 共识层  
（参与共识）

* * *

## 🧠 三层极简模型（Mental Model）

| 层 | 作用 | 关键词 |
| --- | --- | --- |
| 🧮 Execution | 做事情 | 执行 |
| 🤝 Consensus | 定规则 | 共识 |
| 🔌 Interface | 连接外部 | 入口 |

* * *

## ⚡ 一句话总结

**Ethereum = Execution + Consensus + Communication**

以太坊 = 执行 + 共识 + 连接
<!-- DAILY_CHECKIN_2026-04-07_END -->

# 2026-04-06
<!-- DAILY_CHECKIN_2026-04-06_START -->



















2026/04/06

今天学习两部分

首先搞明白 如何学习，在哪里打卡

其次快速学习 以太坊 史前历史

## 《Prehistory of Ethereum》

**Ethereum is not an isolated invention — it is the natural outcome of decades of open-source culture, cryptography, and cypherpunk ideology.  
以太坊不是一个孤立的发明，而是几十年开源文化、密码学发展与赛博朋克思想的自然结果.**

### 五条核心演进路径

1.  Internet（互联网）
    
2.  Unix & Open Source（开源文化）
    
3.  Cryptography（密码学）
    
4.  Cypherpunk Movement（赛博朋克）
    
5.  Digital Currency Experiments（数字货币尝试）
    

👉 最终汇聚 → Ethereum

### Ethereum = 技术 × 文化 × 思想 × 实验 的交汇点

1️⃣技术层 Technology ：Internet Unix Cryptography，提供“能力”  
2️⃣ 文化层（Culture）： Open Source、Hacker Culture；提供“协作方式”  
3️⃣ 思想层（Ideology）：Cypherpunk，Freedom / Privacy；提供“价值观”  
4️⃣ 实验层（Experiments）：DigiCash，RSA / PGP；提供“试错路径”

### Key Insights：

Ethereum is an evolution, not an invention.  
以太坊是演化出来的，不是凭空创造的。

Cryptography made trust optional.  
密码学让“信任”变成可选项。

Open systems beat closed systems in the long run.  
开放系统最终胜过封闭系统。
<!-- DAILY_CHECKIN_2026-04-06_END -->
<!-- Content_END -->
