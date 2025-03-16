---
title: 跨链协议
category: web3
tags:
  - solidity
  - blockchain
publishedAt: 2023-03-14
description: 学习什么是跨链、跨链协议、跨链的方式等
---

# 跨链

Token跨链   -->  可编程Token跨链   -->  任意信息跨链

简化的跨链操作：
1. 锁定source链资产（信息）；
2. <font color="#de7802">准确监听到锁定信息</font>，铸造Destination链资产（信息）；

如何保证跨链信息的准确：
- A链的信息如何准确无误的传递给B链信息；
- 谁来保证信息的完整、准确；
- 谁来执行？
- 谁来验证？

类似于Oracle预言机的DON网络，需要一个基础设施来提供统一的跨链协议来保证跨链的稳定准确执行；
- A链对于B链来说，相当于链下数据；反之亦然；
- 因此跨链技术就是基于Oracle预言机来实现，传递信息的过程需要查询某个链、向某个链写入数据。
- 重要的是保证信息在传递过程中的完整、不可篡改、去中心化；

# Chainlink跨链协议CCIP

Chainlink CCIP is a blockchain interoperability protocol that enables developers to build secure applications that can transfer tokens, messages (data), or both tokens and messages across chains.

> Chainlink CCIP 是一种区块链互操作性协议，使开发人员能够构建可以跨链传输代币、消息（数据）或代币和消息的安全应用程序

文档：[https://docs.chain.link/ccip](https://docs.chain.link/ccip)

- Committing DON
- RMN（Risk Manage Network   风险管理网络）
- Executing DON

# 跨链执行流程

类似于<font color="#de7802">2阶段提交</font>，实际上是一个<font color="#de7802">分布式事务</font>

TODO

# 跨链的方式

## 1. Burn & Mint

![](/images/web3-crosschain-burnmint.png)

Token在源链Burn，然后在目标链上被原生铸造。

适用场景：
- Token在多条链上都有原生部署；
- CCIP可以调用`burn`和`mint`函数；（要求较高）

## 2. Lock & Mint

![](/images/web3-crosschain-lockmint.png)

Token在源链被锁在池子中，对应的Token的包装、衍生、合成的资产在目标链上被铸造；

Token被跨链回源链时：包装Token在目标链上被Burn，在源链上解锁释放出来；

适用场景：
- 不需要在多条链上部署原生Token；只需要在源链上发行Token；
- 适用于带有编码约束的Token；



# 实现一个跨链NFT

1. NFT
2. 创建`WrapperNFT`合约，实现对Wrapper后的Token的铸造；
3. 创建`LockAndRelease`合约，实现对源链的NFT的锁定；

查看：[http://docs.chian.link/ccip](http://docs.chian.link/ccip)


