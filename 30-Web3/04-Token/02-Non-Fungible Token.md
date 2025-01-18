
# NFT

`NFT`是一种基于区块链技术的数字资产，它代表了对特定数字或物理资产的所有权或使用权。与比特币等同质化代币不同，NFT 具有独一无二、不可分割、不可替代的特性，每个 NFT 都有其独特的标识和价值

`NFT`的创建、转让、所有权等规则都需要通过`智能合约`在去中心化的情况下完成；

## Metadata

opensea规定的metadata标准：https://docs.opensea.io/docs/metadata-standards

# 去中心化存储

真实的NFT实质数据不需要存储在链上，有去中心化的存储基础设施可以完成：

[IPFS](https://ipfs.tech/)：一个点对点的内容分发网络；
[Filebase](https://filebase.com/)：一个基于IPFS网络提供更方便的内容操作服务，可以便捷的上传内容到IPFS网络中；


# ERC-721

OpenZeppelin提供了安全的合约标准；并提供定制化features的合约生成的Wizard，更安全便捷；

ERC-721合约标准：[https://wizard.openzeppelin.com/#erc721](https://wizard.openzeppelin.com/#erc721)

使用[https://wizard.openzeppelin.com/#](https://wizard.openzeppelin.com/#)快速创建合约：

![](/images/web3-token-nft-openzeppelin-wizard.png)


# 实现NFT

## 1. 制作内容并存储在IPFS

登录[Filebase](https://filebase.com/)，并创建自己的bucket，上传自己的内容，这里以图片为例；获取IPFS的cid：QmPjXuwwGuDczjV1RGHVu4gUpbiKYJrGmF6j2RyiQyPKvo

![](/images/web3-nft-filebase.png)

## 2. 创建项目并完成NFT合约

在OpenZeppelin的ERC-721合约Wizard中构建一个MyToken合约，合约中的`_safeMint`函数则是铸造NFT，传入铸造者和NFT内容的uri：

```js
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.22;

import {ERC721} from "@openzeppelin/contracts/token/ERC721/ERC721.sol";
import {ERC721Burnable} from "@openzeppelin/contracts/token/ERC721/extensions/ERC721Burnable.sol";
import {ERC721Enumerable} from "@openzeppelin/contracts/token/ERC721/extensions/ERC721Enumerable.sol";
import {ERC721URIStorage} from "@openzeppelin/contracts/token/ERC721/extensions/ERC721URIStorage.sol";
import {Ownable} from "@openzeppelin/contracts/access/Ownable.sol";

contract MyToken is
    ERC721,
    ERC721Enumerable,
    ERC721URIStorage,
    ERC721Burnable,
    Ownable
{
    uint256 private _nextTokenId;

    constructor(
        address initialOwner
    ) ERC721("MyToken", "MTK") Ownable(initialOwner) {}

    function safeMint(address to, string memory uri) public onlyOwner { 
		uint256 tokenId = _nextTokenId++; 
		_safeMint(to, tokenId); 
		_setTokenURI(tokenId, uri); 
	}

    // The following functions are overrides required by Solidity.

    function _update(
        address to,
        uint256 tokenId,
        address auth
    ) internal override(ERC721, ERC721Enumerable) returns (address) {
        return super._update(to, tokenId, auth);
    }

    function _increaseBalance(
        address account,
        uint128 value
    ) internal override(ERC721, ERC721Enumerable) {
        super._increaseBalance(account, value);
    }

    function tokenURI(
        uint256 tokenId
    ) public view override(ERC721, ERC721URIStorage) returns (string memory) {
        return super.tokenURI(tokenId);
    }

    function supportsInterface(
        bytes4 interfaceId
    )
        public
        view
        override(ERC721, ERC721Enumerable, ERC721URIStorage)
        returns (bool)
    {
        return super.supportsInterface(interfaceId);
    }
}
```


在`Remix`中编译并部署合约，执行`_safeMint`函数，将自己的地址、IPFS中的`ipfs://cid`传入，即可铸造NFT：

合约地址：0x7EA9820e6E5C5f07E5450D1A132007a8A8D2B11B

![](/images/web3-nft-myToken.png)

执行`totalSupply`可以看到结果为1
## 3. 在OpenSea测试网查看铸造完成的NFT

使用铸造者登录OpenSea测试网：[https://testnets.opensea.io/](https://testnets.opensea.io/)

在`profile`中即可查看到铸造的NFT


