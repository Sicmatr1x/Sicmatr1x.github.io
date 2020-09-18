---
title: ETH Solidity智能合约入门
date: 2020-09-05 10:45:38
categories:
    - Solidity
tags: 
    - 踩坑
    - ETH
    - 区块链
    - 智能合约
---

Solidity 是一门面向合约的、为实现智能合约而创建的高级编程语言。这门语言受到了 C++，Python 和 Javascript 语言的影响，设计的目的是能在以太坊虚拟机（EVM）上运行。

Solidity 是静态类型语言，支持继承、库和复杂的用户定义类型等特性。

目前尝试 Solidity 编程的最好的方式是使用 [Remix](https://remix.ethereum.org/) （需要时间加载，请耐心等待）。Remix 是一个基于 Web 浏览器的 IDE，它可以让你编写 Solidity 智能合约，然后部署并运行该智能合约。

Solidity Doc: https://solidity-cn.readthedocs.io/zh/develop/

### Solidity源码和智能合约

Solidity源代码要成为可以运行在以太坊上的智能合约需要经历如下的步骤：
1. 用Solidity编写的智能合约源代码需要先使用编译器编译为字节码（Bytecode)，编译过程中会同时产生智能合约的二进制接口规范（Application Binary Interface，简称为ABl）；
2. 通过交易（Transaction)的方式将字节码部署到以太坊网络，每次成功部署都会产生一个新的智能合约账户；
3. 使用Javascript编写的DApp通常通过web3.js+ABI去调用智能合约中的函数来实现数据的读取和修改。

<img src="./slidity编译流程.png">

## 安装Solidity编译器

### Remix

[Remix](https://remix.ethereum.org/) 可在线使用，而无需安装任何东西。如果你想离线使用，可按 https://github.com/ethereum/browser-solidity/tree/gh-pages 的页面说明下载 zip 文件来使用。 该页面有进一步详细说明如何安装 Solidity 命令行编译器到你计算机上。如果你刚好要处理大型合约，或者需要更多的编译选项，那么你应该选择使用命令行编译器 solc。

### solcjs

solc是Solidity源码库的构建目标之一，它是Solidity的命令行编译器

使用npm可以便捷地安装Solidity编译器solcjs

```
npm install -g solc
```

### 根据例子学习Solidity

#### 创建水龙头合约

**打开remix在线开发环境：**

选择new file创建Fucet.sol文件

<img src="./20200905105632.png">

输入代码

**编译合约：**

选择solidity compiler tab，点击compile Faucet.sol按钮编译代码为字节码文件

<img src="./20200905105759.png">

编译完成后solidity compiler tab上面的旋转箭头会变成一个成功的绿色小勾

**发布合约：**

需要浏览器安装MetaMask插件并登录ETH钱包
 
<img src="./20200905110105.png">

这里选择Ropsten测试环境

选择Deploy & Run Transactions tab，Environment选择Injected Web3，当你选择成功之后metamask会自动提示是否授权网页访问metamask钱包，选择同意你的钱包地址就会出现在Account下面的选项卡里，如选图所示：

<img src="./20200905110222.png">

点击deploy，metamask会弹出是否支付一定的gas来创建合约，点确定，等一下，等待时间取决于你的网络状况

如果deploy成功会这个tab里出现如上图所示的Deployed Contracts一栏且内容为部署成功之后的合约的地址

本例中为：`0x3280EB0E28d1715C6fBfFb18952fB4D8A548B93d`

使用Etherscan查询deploy合约的transaction：https://ropsten.etherscan.io/tx/0x1b0a4f6223a539870cc216c98d70742fc550879648ae2235bbc77cf222f24ba7

<img src="./20200905110829.png">

使用Etherscan查询deploy合约的地址：https://ropsten.etherscan.io/address/0x3280eb0e28d1715c6fbffb18952fb4d8a548b93d

<img src="./20200905110903.png">

这个案例合约的作用是任何人可以提供一个amount参数然后该合约会给你发对应的币，但是当前合约里面没有ETH。所以需要添加一个接受ETH币的函数。重新deploy不是在旧的合约上面修改而是部署一个新的合约，因为合约一旦部署到区块链上就无法修改了。

`pragma solidity ^0.4.17;`引入solidity的版本，`^`表示高于0.4.17但小于0.5这个大版本，`>`表示大于当前版本

```solidity
pragma solidity ^0.4.17;

contract Fucet {
    function withdraw(uint amount) public {
        require(amount <= 100000000000000000);
        msg.sender.transfer(amount); // wei
    }
    
    
    function () public payable {
        
    }
}
```

重新deploy，使用Etherscan查询deploy合约的地址：https://ropsten.etherscan.io/address/0x7124eA18277f9a7764848470733F1aD98aFF9154

<img src="./20200905150141.png">

使用metamask发送1个ETH到新的合约地址：https://ropsten.etherscan.io/tx/0xff2c7d0568ab8b994ef1e0feeb779b2be19367186ba2a0e23c1014915dd5fe38

<img src="./20200905150231.png">

可以看到发送成功之后新的合约上的交易就从1个变成2个了，第一个transaction是合约创建的transaction，第二个是我们转了1个ETH到这个合约的transaction

<img src="./20200905150317.png">

好了终于可以测试水龙头的代码了：我们输入`100000000000000000`到amount参数框里，尝试从水龙头获取0.1个ETH

<img src="./20200905150523.png">

交易成功，看下transaction详情：

<img src="./20200905151148.png">

再看下水龙头合约的transaction记录：

<img src="./20200905150857.png">

注意：这里第三个transaction的value怎么是0，我们不是从合约得到了0.1个ETH吗，合约不是把ETH直接从一个账户转到另外一个账户而是走内部交易

<img src="./20200905151804.png">

那么我们的测试钱包是否也有一个对应的内部交易呢？是的

<img src="./20200905152006.png">

到这里就宣布水龙头合约大功告成啦！！！

#### 简单数据存储合约

```sol
pragma solidity >0.4.22;

contract SimpleStorage {
    uint myData;
    function setData(uint newData) public {
        myData = newData;
    }
    // view表明该function只读取数据不修改数据
    function getData() public view returns(uint) {
        return myData;
    }
}

```

- Deploy success at: 0x3280eb0e28d1715c6fbffb18952fb4d8a548b93d
- Etherscan: https://rinkeby.etherscan.io/address/0x3280eb0e28d1715c6fbffb18952fb4d8a548b93d

与合约交互测试一下效果：

<img src="./20200916113754.png">

这里设置值需要消耗gas但是获取值getData不需要消耗gas

<img src="./202009161137.png">


