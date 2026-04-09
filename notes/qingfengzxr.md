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
