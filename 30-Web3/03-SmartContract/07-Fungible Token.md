---
title: ERC-20
category: web3
tags:
  - solidity
  - blockchain
publishedAt: 2024-02-04
description: ERC-20学习，基于OpenZeppelin的ERC-20合约发布一个Token
---

# ERC-20

全称：Ethereum Request for Comment 20
意义：ERC-20是一个同质化代币标准，定义了一套规则和接口，可以在以太坊及其兼容的区块链上开发同质化代币智能合约
- ERC-20 是一个标准，遵循此标准的代币，能更好更兼容的被去中心化应用接受。也能方便用户使用；
- 对于开发者来说，使用标准化的代币合约节省精力，同时也减少漏洞的可能性。
- ERC-20标准能更推动以太坊的代币经济发展，但是对于高级、复杂的金融衍生品支持不足。

ERC-20规定的标准函数：
- `totalSupply`（总供应量）
- `balanceOf`（账户余额查询）
- `transfer`（代币转移）
- `transferFrom`（从一个账户转移到另一个账户）
- `approve`（批准他人使用自己的代币）
- `allowance`（查询他人可用的代币额度）


# ERC-20实现

[OpenZeppelin](https://www.openzeppelin.com/)就是这样一个安全的智能合约标准库提供方，提供了各类经过安全审计的合约标准；

因此可以在ERC20合约的基础上再进行个性化需求创造即可；

> ERC-20合约标准：[openzeppelin-contracts/contracts/token/ERC20/ERC20.sol](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/ERC20.sol)


基于ERC-20，实现一个FundMeToken

```js
// SPDX-License-Identifier: MIT
// Compatible with OpenZeppelin Contracts ^5.0.0
pragma solidity ^0.8.22;

import {ERC20} from "@openzeppelin/contracts/token/ERC20/ERC20.sol";
import {FundMe} from "./FundMe.sol";

// 1. FundMe的参与者，可以mint Token；基于Fund金额，获取相应数量的token，这里简化为1:1
// 2. FundMe的参与者，可以transfer Token；
// 3. Token使用完成，需要burn
contract FundMeToken is ERC20 {

    FundMe fundMe;

    constructor(address fundMeAddr) ERC20("FundTokenERC20", "FT") {
        fundMe = FundMe(fundMeAddr);
    }

    function mint(uint256 amountToMint) public {
        require(fundMe.getFundSuccess(), "The FundMe is not compeleted yet.");
        uint256 avaliableMint = fundMe.funderToAmountMapping(msg.sender) - balanceOf(msg.sender);
        require(avaliableMint >= amountToMint, "You cannot mint this many tokens");
        _mint(msg.sender, amountToMint);
    }

    // _mint, _update, _transfer 使用父合约的实现

    // 领取token，销毁对应的token
    function claim(uint256 amountToClaim) public {
        require(balanceOf(msg.sender) >= amountToClaim, "You dont have enough ERC20 tokens");
        _burn(msg.sender, amountToClaim);
    }
}
```