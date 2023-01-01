# 1.什么是区块链与智能合约，为什么要学习区块链安全

为什么区块链安全很重要？自 2008 年首次发布比特币白皮书以来，区块链的使用量呈爆炸式增长。许多应用程序依靠这项技术来增加信任和隐私，否则它们将不会出现在集中式系统中。围绕区块链技术的生态系统庞大而复杂，并且有许多活动部分。存在用户可以交易各种加密货币、NFT 和代币的交易所。可以编写智能合约以编程方式将行为应用于区块链交易。存在去中心化金融（DeFi）市场，用户无需注册账户即可交换代币。所有这些部分都容易出现漏洞，并且随着区块链处于新兴技术的前沿，每天都会发现新问题。在这个 Black Hills 信息安全 (BHIS) 网络广播中，我们将使用有关最近区块链黑客攻击的案例研究来介绍编写/设计智能合约时发生的潜在问题，这些问题最终导致攻击者损失数百万美元。

![image.png](https://image.3001.net/images/20221217/1671257792_639d5ec098d515c404192.png!small)

### 区块链有哪些不同类型？

1.公共区块链

```
公共区块链上发生的所有交易都是完全透明的，这意味着任何人都可以分析交易的细微之处。例如：比特币和以太坊。
```

2.私有区块链

```
私有区块链上发生的所有交易都是私有的，并且被允许加入私有区块链网络的系统成员可以轻松访问。
```

3.联盟区块链

```
联盟区块链与私有区块链非常相似。 它们之间的主要区别在于，联盟区块链不是由单个实体管理，而是由一个团体管理。联盟区块链的参与者可以合并从国家银行到政府再到供应链的任何人。
```

4.许可的区块链网络

```
建立私有区块链的企业通常会建立一个许可的区块链网络。需要注意的是，公共区块链网络也可以被许可。这对允许谁参与网络以及参与哪些交易施加了限制。参与者需要获得邀请或许可才能加入。
```

### 什么是区块链

![image.png](https://image.3001.net/images/20221217/1671257823_639d5edfeda2362db0df0.png!small)
官方定义：

```
https://www.ibm.com/topics/what-is-blockchain
```

##### 区块链定义

区块链是一种共享的、不可变的分类帐，有助于在业务网络中记录交易和跟踪资产的过程。资产可以是有形的 （房屋、汽车、现金、土地）或无形的（知识产权、专利、版权、品牌）。几乎任何有价值的东西都可以在区块链网络上进行跟踪和交易，从而降低所有相关人员的风险和成本。

##### 为什么区块链很重要

业务依赖于信息。接收得越快，越准确越好。区块链是传递该信息的理想选择，因为它提供即时、共享和完全透明的信息，存储在一个不可变的分类账上，只有获得许可的网络成员才能访问。区块链网络可以跟踪订单、付款、账户、生产等等。并且由于成员共享单一的真相视图，您可以端到端查看交易的所有细节，让您更有信心，以及新的效率和机会。

### 区块链上的智能合约是什么

智能合约只是存储在区块链上的程序，在满足预定条件时运行。它们通常用于自动执行协议，以便所有参与者都可以立即确定结果，而无需任何中间人的参与或时间损失。他们还可以自动化工作流程，在满足条件时触发下一个操作。
官方解释：

```
https://www.ibm.com/topics/smart-contracts
```

# 2.关于一些区块链安全的攻击手段

```
1.重入攻击
2.抢先攻击
3.间溢出和下溢
4.拒绝服务
5.访问控制
5.时间戳依赖
```

关注我，之后会一一用实战演示相关的攻击方法

# 3.学习区块链安全的资料与靶场

书籍：

```
https://github.com/ethereumbook/ethereumbook
```

靶场：

```
https://ethernaut.openzeppelin.com/
https://www.damnvulnerabledefi.xyz/
```

学习资料：

```
https://cryptozombies.io/zh/course/
https://solidity-by-example.org/
```

参考资料：

```
https://www.investopedia.com/terms/b/blockchain.asp
https://www.ibm.com/topics/smart-contracts
https://www.ibm.com/topics/what-is-blockchain
https://www.getastra.com/blog/knowledge-base/blockchain-security/
https://www.you去掉字符tube.com/watch?v=WchXkMlKj9w
https://www.you去掉字符tube.com/watch?v=M6sLKkc6bV8
```

# 4.区块链安全测试工具有哪些？

SWC-registry：智能合约弱点分类和测试用例。

```
https://swcregistry.io/
```

MythX：这是一个智能合约安全分析 API，支持 Ethereum、Quorum、Vechain、Roostock、Tron 和其他与 EVM 兼容的区块链。

```
https://mythx.io/
```

Echidna：这是一个 Haskell 程序，旨在对以太坊智能合约进行模糊测试/基于属性的测试。

```
https://github.com/crytic/echidna
```

Manticore：它是用于分析智能合约和二进制文件的符号执行工具。

```
https://github.com/trailofbits/manticore
```

Oyente：用于智能合约安全的静态分析工具。

```
https://github.com/melonproject/oyente
```

Securify 2.0：Securify 2.0 是以太坊智能合约的安全扫描器。

```
https://securify.chainsecurity.com/
```

SmartCheck：静态智能合约安全分析器。

```
https://tool.smartdec.net/
```

# 总结

在不到一年的时间里，超过12亿美元从基于智能合约的项目中被盗，这一篇文章只是简单的介绍一下什么是区块链和智能合约，以及相关的资料，关注我，我会在之后的文章里演示如何发现漏洞并利用。
