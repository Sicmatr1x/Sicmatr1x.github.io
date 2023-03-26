---
title: Ethereum
date: 2019-07-25 10:28:01
categories:
    - Crypto
tags: 
    - Ethereum
---

> 以太坊的目的是基于脚本、竞争币和链上元协议（on-chain meta-protocol）概念进行整合和提高，使得开发者能够创建任意的基于共识的、可扩展的、标准化的、特性完备的、易于开发的和协同的应用。以太坊通过建立终极的抽象的基础层-内置有图灵完备编程语言的区块链-使得任何人都能够创建合约和去中心化应用并在其中设立他们自由定义的所有权规则、交易方式和状态转换函数。域名币的主体框架只需要两行代码就可以实现，诸如货币和信誉系统等其它协议只需要不到二十行代码就可以实现。智能合约-包含价值而且只有满足某些条件才能打开的加密箱子-也能在我们的平台上创建，并且因为图灵完备性、价值知晓（value-awareness）、区块链知晓（blockchain-awareness）和多状态所增加的力量而比比特币脚本所能提供的智能合约强大得多。

以太坊白皮书-译文：https://ethfans.org/wikis/%E4%BB%A5%E5%A4%AA%E5%9D%8A%E7%99%BD%E7%9A%AE%E4%B9%A6

## 常用ETH客户端

- go-ethereum(Go)：官方推荐，开发使用最多 
  - 地址：https://github.com/ethereum/go-ethereum
- parity（Rust)：最轻便客户端，在历次以太坊网络攻击中表现卓越 
  - 地址：https://github.com/ethcore/parity/releases
- cpp-ethereum(C++) 
  - 地址：https://github.com/ethereum/cpp-ethereum 
- pyethapp(python) 
  - 地址：https://github.com/heikoheiko/pyethapp 
- ethereumjs-lib(javascript) 
  - 地址：https://github.com/ethereumis/ethereumis-lib 
- EthereumJ/Harmony(Java) 
  - 地址：https://github.com/ethereum/ethereumi

### 节点

#### 全节点

全节点是整个主链的一个副本，存储并维护链上的所有数据，并随时验证新区块的合法性。
区块链的健康和扩展弹性，取决于具有许多独立操作和地理上分散的全节点。每个全节点都可以帮助其他新节点获取区块数据，并提供所有交易和合约的独立验证。
运行全节点将耗费巨大的成本，包括硬件资源和带宽。
以太坊开发不需要在实时网络（主网）上运行的全节点。我们可以使用测试网络的节点来代替，也可以用本地私链，或者使用服务商提供的基于云的以太坊客户端；这些几乎都可以执行所有操作。

优点：
- 为以太坊网络的灵活性和抗审查性提供有力支持。
- 权威地验证所有交易。
- 可以直接与公共区块链上的任何合约交互。
- 可以离线查询区块链状态（帐户，合约等）。
- 可以直接把自己的合约部署到公共区块链中。

缺点：
- 需要巨大的硬件和带宽资源，而且会不断增长。
- 第一次下载往往需要几天才能完全同步。
- 必须及时维护、升级并保持在线状态以同步区块

运行全节点配置要求：

最低要求：
- 双核以上CPU
- 硬盘存储可用空间至少80GB
- 如果是SSD，需要4GB以上RAM，如果是HDD，至少8GB RAM
- 8MB/s下载带宽

推荐配置：
- 四核以上的快速CPU
- 16GB以上RAM
- 500GB以上可用空间的快速SSD
- 25+MB/s下载带宽

#### 远程客户端

不存储区块链的本地副本或验证块和交易。这些客户端一般只提供钱包的功能，可以创建和广播交易。远程客户端可用于连接到现有网络，MetaMask就是一个这样的客户端。

#### 轻节点

不保存链上的区块历史数据，只保存区块链当前的状态。轻节点可以对块和交易进行验证。

#### 本地私链

优点：
- 磁盘上几乎没有数据，也不同步别的数据，是一个完全“干净”的环境。
- 无需获取测试以太，你可以任意分配以太，也可以随时自己挖矿获得。
- 没有其他用户，也没有其他合约，没有任何外部干扰。
    
缺点：
- 没有其他用户意味与公链的行为不同。发送的交易并不存在空间或交易顺序的竞争。
- 除自己之外没有矿工意味着挖矿更容易预测，因此无法测试公链上发生的某些情况。
- 没有其他合约，意味着你必须部署要测试的所有内容，包括所有的依赖项和合约库。

### ETH客户端Geth(go-ethereum(Go))的使用

下载完毕后进入根目录：

```
geth --datadir G:\Ethereum
```

使用`--datadir`参数指定数据存放目录，因为我使用的windows版本所以这里的路径是Windows的路径

### JSON-RPC

以太坊客户端提供了API和一组远程调用（RPC）命令，这些命令被编码为JSON。这被称为JSON-RPCAPI。本质上，JSON-RPCAPI就是一个接口，允许我们编写的程序使用以太坊客户端作为网关，访问以太坊网络和链上数据。

通常，RPC接口作为一个HTTP服务，端口设定为8545。出于安全原因，默认情况下，它仅限于接受来自localhost的连接。

要访问JSON-RPCAPI，我们可以使用编程语言编写的专用库，例如JavaScript的 web3.js。

或者也可以手动构建HTTP请求并发送/接收JSON编码的请求，如：

```
$curl -X POST -H "Content-Type:application/json" --data \ {"jsonrpc":"2.0""method":"web3_clientVersion'","params":],"id":1"I http://localhost:8545
```

### 以太坊没有UTXO，但是又Account

以太坊的“状态”，就是系统中所有帐户的列表

每个账户都包括了一个余额（balance)，和以太坊特殊定义的数据（代码和内部存储）

如果发送帐户有足够的余额来支付，则交易有效；在这种情况下发送帐户先扣款，而收款帐户将记入这笔收入

如果接收帐户有相关代码，则代码会自动运行，并且它的内部存储也可能被更改，或者代码还可能向其他帐户发送额外的消息，这就会导致进一步的借贷资金关系


### 交易(Transaction)

签名的数据包，由EOA发送到另一个账户：
1. (To)消息的接收方地址
2. v,r,s：发送方EOA的ECDSA签名的三个组成部分
3. 金额(Value)
4. 数据(Data)可选：可变长度二进制数据负载(payload)
5. START GAS：交易发起人愿意支付的最大gas量
6. GAS PRICE：交易发起人愿意支付的gas单价(单位：wei)
7. nonce：发起人EOA发出的序列号，防止交易消息重播

### 消息(Message)

- 合约可以向其它合约发送“消息"
- 消息是不会被序列化的虚拟对象，只存在于以太坊执行环境（EVM）中，而交易会被矿工打包到区块中，若消息不涉及到转账就不会被序列号并打包到区块中
- 可以看作函数调用

- 消息发送方
- 消息接收方
- 金额（VALUE)
- 数据（DATA，可选）
- START GAS

### 合约(Contract)

可以读/写自己的内部存储（32字节key-value的数据库）

可向其他合约发送消息，依次触发执行

一旦合约运行结束，并且由它发送的消息触发的所有子执行（sub-execution)结束，EVM就会中止运行，直到下次交易被唤醒

### 交易中的gas

当由于交易或消息触发EVM运行时，每个指令都会在网络的每个节点上执行。这具有成本：对于每个执行的操作，都存在固定的成本，我们把这个成本用一定量的gas表示。

gas是交易发起人需要为EVM上的每项操作支付的成本名称。发起交易时，我们需要从执行代码的矿工那里用以太币购买gas。

gas与消耗的系统资源对应，这是具有自然成本的。因此在设计上gas和ether有意地解耦，消耗的gas数量代表了对资源的占用，而对应的交易费用则还跟gas对以太的单价有关。这两者是由自由市场调节的：gas的价格实际上是由矿工决定的，他们可以拒绝处理gas价格低于最低限额的交易。我们不需要专门购买gas，只需将以太币添加到帐户即可，客户端在发送交易时会自动用以太币购买gas。而以太币本身的价格通常由于市场力量而波动。

### 交易的接收者(To)

交易接收者在to字段中指定，是一个20字节的以太坊地址。地址可以是EOA或合约地址。

以太坊没有进一步的验证，任何20字节的值都被认为是有效的。如果20字节值对应于没有相应私钥的地址，或不存在的合约，则该交易仍然有效。以太坊无法知道地址是否是从公钥正确派生的。

如果将交易发送到无效地址，将销毁发送的以太，使其永远无法访问。

验证接收人地址是否有效的工作，应该在用户界面一层完成。

## 交易的value和data

交易的主要“有效负载”包含在两个字段中：value和data。交易可以同时有value和data，仅有value，仅有data，或者既没有value也没有data。所有四种组合都有效。

- 仅有value的交易就是一笔以太的付款
- 仅有data的交易一般是合约调用
- 进行合约调用的同时，我们除了传输data，还可以发送以太，从而交易中同时包含data和value没有value也
- 没有data的交易，只是在浪费gas，但它是有效的

## 向EOA或合约传递data
  
当交易包含数据有效负载时，它很可能是发送到合约地址的，但它同样可以发送给EOA
  
如果发送data给EOA，数据负载（data payload）的解释取决于钱包
  
如果发送数据负载给合约地址，EVM会解释为函数调用，从payload里解码出函数名称和参数，调用该函数并传入参数
  
发送给合约的数据有效负载是32字节的十六进制序列化编码：
- 函数选择器：函数原型的Keccak256哈希的前4个字节。这允许EVM明确地识别将要调用的函数。
- 函数参数：根据EVM定义的各种基本类型的规则进行编码。

## 以太坊虚拟机(EVM)

- 以太坊虚拟机EVM是智能合约的运行环境
- 作为区块验证协议的一部分，参与网络的每个节点都会运行EVM。他们会检查正在验证的块中列出的交易，并运行由EVM中的交易触发的代码
- EVM不仅是沙盒封装的，而且是完全隔离的，也就是说在EVM中运行的代码是无法访问网络、文件系统和其他进程的，甚至智能合约之间的访问也是受限的
- 合约以字节码的格式（EVMbytecode)存在于区块链上
- 合约通常以高级语言（solidity)编写，通过EVM编译器编译为字节码，最终通过客户端上载部署到区块链网络中

### EVM和账户

- 以太坊中有两类账户：外部账户和合约账户，它们共用EVM中同一个地址空间
- 无论帐户是否存储代码，这两类账户对EVM来说处理方式是完全一样的
- 每个账户在EVM中都有一个键值对形式的持久化存储。其中key和value的长度都是256位，称之为存储空间(storage)

### EVM和交易

- 交易可以看作是从一个帐户发送到另一个帐户的消息，它可以包含二进制数据（payload)和以太币
- 如果目标账户含有代码，此代码会在EVM中执行，并以payload作为入参，这就是合约的调用
- 如果目标账户是零账户（账户地址为0），此交易就将创建一个新合约，这个用来创建合约的交易的payload会被转换为EVM字节码并执行，执行的输出作为合约代码永久存储

### EVM和gas

- 合约被交易触发调用时，指令会在全网的每个节点上执行：这需要消耗算力成本；每一个指令的执行都有特定的消耗，gas就用来量化表示这个成本消耗
- 一经创建，每笔交易都按照一定数量的gas预付一笔费用，目的是限制执行交易所需要的工作量和为交易支付手续费
- EVM执行交易时，gas将按特定规则逐渐耗尽
- gas price是交易发送者设置的一个值，作为发送者预付手续费的单价。如果交易执行后还有剩余，gas会原路返还
- 无论执行到什么位置，一旦gas被耗尽（比如降为负值），将会触发一个out-of-gas异常。当前调用帧（call frame)所做的所有状态修改都将被回滚

### EVM数据存储

Storage
- 每个账户都有一块持久化的存储空间，称为storage，这是一个将256位字映射到256位字的key-value存储区，可以理解为合约的数据库
- 永久储存在区块链中，由于会永久保存合约状态变量，所以读写的gas开销也最大
Memory(内存）
- 每一次消息调用，合约会临时获取一块干净的内存空间·生命周期仅为整个方法执行期间，函数调用后回收，因为仅保存临时变量，故读写gas开销较小Stack(栈）
- EVM不是基于寄存器的，而是基于栈的，因此所有的计算都在一个被称为栈(stack)的区域执行
- 存放部分局部值类型变量，几乎免费使用的内存，但有数量限制

### EVM指令集

- 所有的指令都是针对“256位的字（word)“这个基本的数据类型来进行操作
- 具备常用的算术、位、逻辑和比较操作，也可以做到有条件和无条件跳转
- 合约可以访问当前区块的相关属性，比如它的块高度和时间戳

### 消息调用(Message Calls)

- 合约地址到合约地址的value=0的调用被称为消息调用
- 合约可以通过消息调用的方式来调用其它合约或者发送以太币到非合约账户
- 合约可以决定在其内部的消息调用中，对于剩余的gas，应发送和保留多少
- 如果在内部消息调用时发生了out-of-gas异常（或其他任何异常），这将由一个被压入栈顶的错误值所指明；此时只有与该内部消息调用一起发送的gas会被消耗掉

### 委托调用(Delegatecall)

- 一种特殊类型的消息调用
- 目标地址的代码将在发起调用的合约的上下文中执行，并且msg.sender和msg.value不变
- 可以由此实现“库”（library)：可复用的代码库可以放在一个合约的存储上，通过委托调用引入相应代码

### 合约的创建和自毁

- 通过一个特殊的消息调用create calls，合约可以创建其他合约（不是简单的调用零地址）
- 合约代码从区块链上移除的唯一方式是合约在合约地址上的执行自毁操作selfdestruct；合约账户上剩余的以太币会发送给指定的目标，然后其存储和代码从状态中被移除


