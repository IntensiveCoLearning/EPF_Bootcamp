---
timezone: UTC+8
---

# cooper

**GitHub ID:** qingfengzxr

**Telegram:** @icooperhero

## Self-introduction

大家好，我叫Cooper, 是Web3爱好者，区块链开发工程师，也是游戏爱好者。

## Notes

<!-- Content_START -->
# 2026-04-21
<!-- DAILY_CHECKIN_2026-04-21_START -->
阅读reth p2p源码
<!-- DAILY_CHECKIN_2026-04-21_END -->

# 2026-04-20
<!-- DAILY_CHECKIN_2026-04-20_START -->

今天主要还是做reth p2p模块的源码阅读，然后补了一下周末两天的进度。
<!-- DAILY_CHECKIN_2026-04-20_END -->

# 2026-04-17
<!-- DAILY_CHECKIN_2026-04-17_START -->


阅读reth p2p模块代码
<!-- DAILY_CHECKIN_2026-04-17_END -->

# 2026-04-16
<!-- DAILY_CHECKIN_2026-04-16_START -->



这两天太忙了，搞忘记了！今晚恶补一下
<!-- DAILY_CHECKIN_2026-04-16_END -->

# 2026-04-13
<!-- DAILY_CHECKIN_2026-04-13_START -->




1）为什么discovery4要升级为5，有哪些缺陷，解决什么问题？

-   discovery4使用`expiration` 字段来防止数据包重放，由于它是一个绝对时间戳，节点的时钟必须准确才能正确验证它。自 2016 年协议启动以来，有收到了大量关于用户时钟错误导致连接问题的报告。
    
-   **endpoint proof 不可靠，导致加入网络和查询经常脆弱**。discv4 里现有的 endpoint verification（先前做过的 PING/PONG 证明）并不可靠，因为任一方都可能忘掉之前的状态。结果就是，A 以为 B 已经认可自己，于是直接发 `FINDNODE`，但 B 可能并不承认，于是请求失败；v4 实现可以靠重试缓解，但这会很慢，而且通常做得不好。
    
-   **v4 能传播的信息太死板。**EIP-778 解释得很清楚：当时的 discovery v4 实际上传递的只是节点公钥、IP 和两个端口，别的信息都带不出去；这限制了协议升级、能力广告和多传输/多身份方案扩展。EIP-778 之所以提出 ENR，就是为了“lift the restrictions of discovery v4 protocol”，把节点信息变成可签名、可版本化、可扩展的记录。
    
-   **v4 的身份体系和元数据扩展性差。**discv5 的设计目标之一就是支持不止一种 node ID cryptosystem，并用 ENR 承载“任意 key/value 元数据”；它在和 v4 的对比里也直接写了：v5 支持任意节点元数据、身份密码学可扩展，不再强制绑死 secp256k1。
    
-   **明文通信，隐私和抗审查都比较弱。**
    
-   **v4 没有内建“按主题找节点”的能力。**
    

升级到discovery v5主要是需要把发现层升级成一个基于 ENR、更安全、更可扩展、支持 topic discovery、并且不依赖时钟的新协议。  
  
2）Gossip和KDA负责不同的职能目标

Kademlia 是一种 **DHT（分布式哈希表）协议**。它把节点和 key 都映射到同一个 ID 空间里，用 XOR 距离定义“谁离目标更近”，然后逐步把查询路由到更接近目标的节点，典型复杂度是对数级。

Gossip 是一种 **流言式/流行病式传播协议**。节点周期性随机挑选少量邻居交换信息，消息像“传八卦”一样扩散，目标不是精确定位某个 key，而是让集群最终大范围收敛到一致视图。
<!-- DAILY_CHECKIN_2026-04-13_END -->

# 2026-04-12
<!-- DAILY_CHECKIN_2026-04-12_START -->





明天补回吧，这两天事情比较多，mark一下。  
  
主要是继续推进P2P模块和原理的学习。
<!-- DAILY_CHECKIN_2026-04-12_END -->

# 2026-04-10
<!-- DAILY_CHECKIN_2026-04-10_START -->






KDA协议详解：  
[https://github.com/apachecn/succinctly-zh-pt3/blob/master/docs/kademlia-protocol/SUMMARY.md](https://github.com/apachecn/succinctly-zh-pt3/blob/master/docs/kademlia-protocol/SUMMARY.md)

还需要弄清楚的问题是：

1）为什么discovery4要升级为5，有哪些缺陷，解决什么问题？

2）KDA 和 Gossip的差异在哪里？分别都适合哪些场景？
<!-- DAILY_CHECKIN_2026-04-10_END -->

# 2026-04-09
<!-- DAILY_CHECKIN_2026-04-09_START -->









-   了解devp2p的架构设计
<!-- DAILY_CHECKIN_2026-04-09_END -->

# 2026-04-08
<!-- DAILY_CHECKIN_2026-04-08_START -->










-   eip-778学习
    
-   discovery4协议学习
    
-   eip-1459学习
    

一些问题：

1） ENR的提出是为了解决什么问题？历史的机制有哪些不好的点？

> EIP778 \*\*Motivation：\*\*本规范旨在通过定义一个灵活的格式——节点记录，用于连接信息，从而解除发现 v4 协议的限制。节点记录可以通过未来版本的节点发现协议进行中继。它们也可以通过 DNS、ENS、devp2p 子协议等任意其他机制进行中继。

```shell
它要解决的第一个老问题，是 旧发现协议能携带的信息太少。EIP-778 直接点明了：discovery v4 原本主要只能中继节点公钥、IP 地址和两个端口，没法传更多信息。ENR 就是为了解掉这个限制，定义一个更灵活的节点记录格式。

第二个问题是 节点信息会变，但以前缺少统一、可信、可版本化的“节点名片”。ENR 里有签名和 seq（序列号）；节点每次记录变化时应增加 seq 并重新发布，其他节点就能知道哪一份是更新后的版本，而不是继续用过期的地址或端口。这个机制就是为了解决“信息更新”和“记录真伪”两个问题。

第三个问题是 需要可扩展性。ENR 的结构允许携带任意键值对，除了 id、secp256k1、ip、tcp、udp、ip6、tcp6、udp6 这些常见字段外，还可以添加新的扩展字段；而且规范强调，未识别的可选条目可以被忽略，这让 ENR 能在不破坏兼容性的前提下不断扩展。
```

2）基于 ENR 的 DNS 节点列表方案，可以让这些节点信息更大规模、可认证、可更新地分发，而不必每次都靠客户端升级。

参考资料：

-   [https://eips.ethereum.org/EIPS/eip-778](https://eips.ethereum.org/EIPS/eip-778)
    
-   [https://github.com/ethereum/devp2p/blob/master/dnsdisc.md](https://github.com/ethereum/devp2p/blob/master/dnsdisc.md)
<!-- DAILY_CHECKIN_2026-04-08_END -->

# 2026-04-07
<!-- DAILY_CHECKIN_2026-04-07_START -->














[https://github.com/paradigmxyz/reth/issues/64](https://github.com/paradigmxyz/reth/issues/64)  
阅读这个issue,了解reth当前的p2p实现的历史渊源
<!-- DAILY_CHECKIN_2026-04-07_END -->
<!-- Content_END -->
