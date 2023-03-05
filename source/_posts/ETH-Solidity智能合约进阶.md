---
title: ETH Solidity智能合约进阶
date: 2020-09-16 16:10:47
categories:
    - Solidity
tags: 
    - Ethereum
    - SmartContract
---

### 根据例子学习Solidity

#### 创建一个简单的发行代币的合约

```sol
pragma solidity >0.4.22 <0.6.0;

contract Coin {
    address public minter;
    mapping(address => uint) public balances;

    event Sent(address from, address to, uint amount);

    constructor() public {
        minter = msg.sender;
    }

    function mint(address receiver, uint amount) public {
        require(msg.sender == minter);
        balances[receiver] += amount;
    }
    
    function send(address receiver, uint amount) public {
        require(amount <= balances[msg.sender]);
        balances[msg.sender] -= amount;
        balances[receiver] += amount;
        emit Sent(msg.sender, receiver, amount);
    }
}
```

以上代码就是一个合约创建人可以无限发币的中心化代币合约

下面这个就是一个规定了发行代币总量的简单代币模型：

```sol
pragma solidity >0.4.22 <0.6.0;

contract Token {
    
    mapping(address => uint) public balances;

    event Sent(address from, address to, uint amount);

    constructor(uint initalSupply) public {
        balances[msg.sender] = initalSupply;
    }
    
    function send(address receiver, uint amount) public returns(bool success){
        require(amount <= balances[msg.sender]);
        require(balances[receiver] + amount >= balances[receiver]);
        balances[msg.sender] -= amount;
        balances[receiver] += amount;
        return true;

        emit Sent(msg.sender, receiver, amount);
    }
}
```

将以上代币模型抽象出来增加细节就成了[ERC20](https://eips.ethereum.org/EIPS/eip-20)

这里使用[ConsenSys implementation](https://github.com/ConsenSys/Tokens/blob/fdf687c69d998266a95f15216b1955a4965a0a6d/contracts/eip20/EIP20.sol)对ERC20的实现来讲解

```sol
/*
Implements EIP20 token standard: https://github.com/ethereum/EIPs/blob/master/EIPS/eip-20.md
.*/


pragma solidity ^0.4.21;

import "./EIP20Interface.sol";


contract EIP20 is EIP20Interface {

    uint256 constant private MAX_UINT256 = 2**256 - 1;
    mapping (address => uint256) public balances;
    //  允许别人用你的地址转币的映射
    mapping (address => mapping (address => uint256)) public allowed;
    /*
    NOTE:
    The following variables are OPTIONAL vanities. One does not have to include them.
    They allow one to customise the token contract & in no way influences the core functionality.
    Some wallets/interfaces might not even bother to look at this information.
    */
    string public name;                   //fancy name: eg Simon Bucks
    uint8 public decimals;                //How many decimals to show.
    string public symbol;                 //An identifier: eg SBX

    function EIP20(
        uint256 _initialAmount,
        string _tokenName,
        uint8 _decimalUnits,
        string _tokenSymbol
    ) public {
        balances[msg.sender] = _initialAmount;               // Give the creator all initial tokens
        totalSupply = _initialAmount;                        // Update total supply
        name = _tokenName;                                   // Set the name for display purposes
        decimals = _decimalUnits;                            // Amount of decimals for display purposes
        symbol = _tokenSymbol;                               // Set the symbol for display purposes
    }

    function transfer(address _to, uint256 _value) public returns (bool success) {
        require(balances[msg.sender] >= _value);
        balances[msg.sender] -= _value;
        balances[_to] += _value;
        emit Transfer(msg.sender, _to, _value); //solhint-disable-line indent, no-unused-vars
        return true;
    }

    function transferFrom(address _from, address _to, uint256 _value) public returns (bool success) {
        uint256 allowance = allowed[_from][msg.sender];
        require(balances[_from] >= _value && allowance >= _value);
        balances[_to] += _value;
        balances[_from] -= _value;
        if (allowance < MAX_UINT256) {
            allowed[_from][msg.sender] -= _value;
        }
        emit Transfer(_from, _to, _value); //solhint-disable-line indent, no-unused-vars
        return true;
    }

    function balanceOf(address _owner) public view returns (uint256 balance) {
        return balances[_owner];
    }

    function approve(address _spender, uint256 _value) public returns (bool success) {
        allowed[msg.sender][_spender] = _value;
        emit Approval(msg.sender, _spender, _value); //solhint-disable-line indent, no-unused-vars
        return true;
    }

    //  允许别人用你的地址转币
    function allowance(address _owner, address _spender) public view returns (uint256 remaining) {
        return allowed[_owner][_spender];
    }
}
```

#### 创建一个简单的投票合约

- 电子投票的主要问题是如何将投票权分配给正确的人员以及如何防止被操纵。这个合约展示了如何进行委托投票，同时，计票又是自动和完全透明的
- 为每个（投票）表决创建一份合约，然后作为合约的创造者——即主席，将给予每个独立的地址以投票权
- 地址后面的人可以选择自己投票，或者委托给他们信任的人来投票
- 在投票时间结束时，`winningProposal()`将返回获得最多投票的提案

```sol
pragma solidity >=0.4.22 <0.7.0;

/** 
 * @title Ballot
 * @dev Implements voting process along with vote delegation
 */
contract Ballot {
   
    struct Voter {
        uint weight; // weight is accumulated by delegation
        bool voted;  // if true, that person already voted
        address delegate; // person delegated to
        uint vote;   // index of the voted proposal
    }

    struct Proposal {
        // If you can limit the length to a certain number of bytes, 
        // always use one of bytes1 to bytes32 because they are much cheaper
        bytes32 name;   // short name (up to 32 bytes)
        uint voteCount; // number of accumulated votes
    }

    address public chairperson;

    mapping(address => Voter) public voters;

    Proposal[] public proposals;

    /** 
     * @dev Create a new ballot to choose one of 'proposalNames'.
     * @param proposalNames names of proposals
     */
    constructor(bytes32[] memory proposalNames) public {
        chairperson = msg.sender;
        voters[chairperson].weight = 1;

        for (uint i = 0; i < proposalNames.length; i++) {
            // 'Proposal({...})' creates a temporary
            // Proposal object and 'proposals.push(...)'
            // appends it to the end of 'proposals'.
            proposals.push(Proposal({
                name: proposalNames[i],
                voteCount: 0
            }));
        }
    }
    
    /** 
     * @dev Give 'voter' the right to vote on this ballot. May only be called by 'chairperson'.
     * @param voter address of voter
     */
    function giveRightToVote(address voter) public {
        require(
            msg.sender == chairperson,
            "Only chairperson can give right to vote."
        );
        require(
            !voters[voter].voted,
            "The voter already voted."
        );
        require(voters[voter].weight == 0);
        voters[voter].weight = 1;
    }

    /**
     * @dev Delegate your vote to the voter 'to'.
     * @param to address to which vote is delegated
     */
    function delegate(address to) public {
        Voter storage sender = voters[msg.sender];
        require(!sender.voted, "You already voted.");
        require(to != msg.sender, "Self-delegation is disallowed.");

        while (voters[to].delegate != address(0)) {
            to = voters[to].delegate;

            // We found a loop in the delegation, not allowed.
            require(to != msg.sender, "Found loop in delegation.");
        }
        sender.voted = true;
        sender.delegate = to;
        Voter storage delegate_ = voters[to];
        if (delegate_.voted) {
            // If the delegate already voted,
            // directly add to the number of votes
            proposals[delegate_.vote].voteCount += sender.weight;
        } else {
            // If the delegate did not vote yet,
            // add to her weight.
            delegate_.weight += sender.weight;
        }
    }

    /**
     * @dev Give your vote (including votes delegated to you) to proposal 'proposals[proposal].name'.
     * @param proposal index of proposal in the proposals array
     */
    function vote(uint proposal) public {
        Voter storage sender = voters[msg.sender];
        require(sender.weight != 0, "Has no right to vote");
        require(!sender.voted, "Already voted.");
        sender.voted = true;
        sender.vote = proposal;

        // If 'proposal' is out of the range of the array,
        // this will throw automatically and revert all
        // changes.
        proposals[proposal].voteCount += sender.weight;
    }

    /** 
     * @dev Computes the winning proposal taking all previous votes into account.
     * @return winningProposal_ index of winning proposal in the proposals array
     */
    function winningProposal() public view
            returns (uint winningProposal_)
    {
        uint winningVoteCount = 0;
        for (uint p = 0; p < proposals.length; p++) {
            if (proposals[p].voteCount > winningVoteCount) {
                winningVoteCount = proposals[p].voteCount;
                winningProposal_ = p;
            }
        }
    }

    /** 
     * @dev Calls winningProposal() function to get the index of the winner contained in the proposals array and then
     * @return winnerName_ the name of the winner
     */
    function winnerName() public view
            returns (bytes32 winnerName_)
    {
        winnerName_ = proposals[winningProposal()].name;
    }
}

```

## Solidity源文件布局

- 源文件可以被版本杂注pragma所注解，表明要求的编译器版本
- 例如：pragma solidity A0.4.0；
- 源文件将既不允许低于0.4.0版本的编译器编译，也不允许高于（包含）0.5.0版本的编译器编译（第二个条件因使用A被添加）import（导入其它源文件）
- Solidity所支持的导入语句import，语法同JavaScript（从ES6起）非常类似

### import

```
import "filename";
```

从“filename”中导入所有的全局符号到当前全局作用域中 

```
import * as symbolName from "filename";
```

创建一个新的全局符号symbolName，其成员均来自“filename”中全局符号 

```
import {symbol1 as alias,symbol2}from "filename";
```

创建新的全局符号alias和symbol2，分别从"filename"引 用symbol1和symbol2

```
import "filename"as symbolName;
```

这条语句等同于import * as symbolName from "filename";

### 基本数据类型

- 布尔（bool）：可能的取值为字符常量值true或false
- 整型（int/uint)：分别表示有符号和无符号的不同位数的整型变量；支持关键字uint8到uint256（无符号，从8位到256位）以及int8到int256，以8位为步长递增
- 定长浮点型（fixed/ufixed)：表示各种大小的有符号和无符号的定长浮点型；在关键字ufixedMxN和fixedMxN中，M表示该类型占用的位数，N表示可用的小数位数
- 地址（address)：存储一个20字节的值（以太坊地址大小）
- 定长字节数组：关键字有bytes1,bytes2,bytes3,…,bytes32
- 枚举（enum)：一种用户可以定义类型的方法，与C语言类似，默认从0开始递增，一般用来模拟合约的状态
- 函数（function)：一种表示函数的类型

数组（Array)
- 数组可以在声明时指定长度（定长数组），也可以动态调整大小（变长数组、动态数组）
- 对于存储型（storage）的数组来说，元素类型可以是任意的（即元素也可以是数组类型，映射类型或者结构体）；对于内存型(memory)的数组来说，元素类型不能是映射（mapping)类型

结构（Struct)
- Solidity支持通过构造结构体的形式定义新的类型

映射（Mapping)
- 映射可以视作哈希表，在实际的初始化过程中创建每个可能的key，并将其映射到字节形式全是零的值（类型默认值）

address
- 地址类型存储一个20字节的值（以太坊地址的大小）；地址类型也有成员变量，并作为所有合约的基础

address payable(v0.5.0引入)
- 与地址类型基本相同，不过多出了transfer和send两个成员变量

两者区别和转换
- Payable地址是可以发送ether的地址，而普通address不能
- 允许从payable address到address的隐式转换，而反过来的直接转换是不可能的（唯一方法是通过uint160来进行中间转换）
- 从0.5.0版本起，合约不再是从地址类型派生而来，但如果它有payable的回退函数，那同样可以显式转换为address或者address payable类型


`<address>.balance(uint256)`
- 该地址的ether余额，以Wei为单位

`<address payable>.transfer(uint256 amount)`
- 向指定地址发送数量为amount的ether（以Wei为单位），失败时抛出异常，发 送2300gas的矿工费，不可调节 

`<address payable>.send(uint256 amount)returns(bool)`
- 向指定地址发送数量为amount的 ether（以Wei为单位），失败时返回false，发 送2300gas的矿工费用，不可调节 

`<address>.call(bytes memory)returns(bool,bytes memory)`
- 发出底层函数CALL，失败时返回false，发送所有可用gas，可调节

`<address>.delegatecall(bytes memory)returns(bool,bytes memory)`
- 发出底层函数DELEGATECALL，失败时返回false，发送所有可用gas，可调节 

`<address>.staticcall(bytes memory)returns(bool,bytes memory)`
- 发出底层函数STATICCALL，失败时返回false，发送所有可用gas，可调节

### Solidity数据位置

- 所有的复杂类型，即数组、结构和映射类型，都有一个额外属性，“数据位置”，用来说明数据是保存在内存memory中还是存储storage中
- 根据上下文不同，大多数时候数据有默认的位置，但也可以通过在类型名后增加关键字storage或memory进行修改
- 函数参数（包括返回的参数）的数据位置默认是memory，局部变量的数据位置默认是storage，状态变量的数据位置强制是storage
- 另外还存在第三种数据位置，calldata，这是一块只读的，且不会永久存储的位置，用来存储函数参数。外部函数的参数（非返回参数）的数据位置被强制指定为calldata，效果跟memory差不多


强制指定的数据位置
- 外部函数的参数（不包括返回参数）：calldata；
- 状态变量：storage

默认数据位置
- 函数参数（包括返回参数）：memory；
- 引用类型的局部变量：storage
- 值类型的局部变量：栈（stack)

特别要求
- 公开可见（publicly visible)的函数参数一定是memory类型，如果要求是storage类型则必须是private或者internal函数，这是为了防止随意的公开调用占用资源

### 未初始化的storage类型指针会指向第一个状态变量

```sol
pragma solidity ^0.4.0;
/**
这个案例涉及到solidity的一个坑
a, b, data和x都是storge的
x是一个可变长度的数组的引用，默认指向该合约空间的头部
合约空间大致如下：
a-b-data
^
|
x

而一个可变数组的指针指向的空间默认存储该可变数组的长度，所以每调一次f()就会导致变量a++
而又因为solidity的可变数组的内容是采用hash表的形式存储的，
所以b的值大概率不变，真正push进去的值不知道存到哪里去了
 */
contract C {
    uint public a;
    uint public b;
    uint[] public data;
    function f() public {
        uint[] x;
        x.push(2);
        data = x;
    }
}
```

接下来看一个更加真实的例子：

```sol
pragma solidity ^0.4.17;

/**
这里你即使猜52合约也不会返还你给的value*2的ETH，因为这个newGuess是sorage类型的指针
未初始化导致其指向此合约第一个状态变量即luckyNumber，导致会把address类型的值赋给luckyNumber覆盖掉原本的值
 */
contract Honeypot {
    uint luckyNumber = 52;
    struct Guess {
        address player;
        uint number;
    }
    Guess[] public guessHistory;
    address owner = msg.sender;
    function guess(uint _num) public payable {
        Guess newGuess;
        newGuess.player = msg.sender;
        newGuess.number = _num;
        guessHistory.push(newGuess);
        if (_num == luckyNumber) {
            msg.sender.transfer(msg.value * 2);
        }
    }
}
```

### solidity函数的可见性

函数的可见性可以指定为external，public，internal 或者private；对于状态变量，不能设置为external，默认是internal。
- external：外部函数作为合约接口的一部分，意味着我们可以从其他合约和交易中调用。一个外部函数f不能从内部调用（即f不起作用，但this.f()可以）。当收到大量数据的时候，外部函数有时候会更有效率。
- puhlic：public函数是合约接口的一部分，可以在内部或通过消息调用。
对于public状态变量，会自动生成一个getter函数。
- internal：这些函数和状态变量只能是内部访问（即从当前合约内部或从它派生的合约访问），不使用this调用。
- private：private函数和状态变量仅在当前定义它们的合约中使用，并且不能被派生合约使用。

- pure：纯函数，不允许修改或访问状态
- view：不允许修改状态
- payable：允许从消息调用中接收以太币Ether。
- constant：与view相同，一般只修饰状态变量，不允许赋值（除初始化以外）

以下情况被认为是修改状态：
- 修改状态变量。
- 产生事件。
- 创建其它合约。
- 使用`selfdestruct`。
- 通过调用发送以太币。
- 调用任何没有标记为view或者pure的函数。
- 使用低级调用。
- 使用包含特定操作码的内联汇编。

以下被认为是从状态中进行读取：
- 读取状态变量。
- 访问`this.balance`或者`<address>.balance`。
- 访问block，tx，msg中任意成员（除msg.sig和msg.data之外）。
- 调用任何未标记为pure的函数。
- 使用包含某些操作码的内联汇编。

### 函数修饰器(modifier)

使用修饰器modifier可以轻松改变函数的行为。例如，它们可以在执行函数之前自动检查某个条件。修饰器modifier是合约的可继承属性，并可能被派生合约覆盖

如果同一个函数有多个修饰器modifier，它们之间以空格隔开，修饰器modifier会依次检查执行。

### 回退函数(fallback)

- 回退函数（fallback function)是合约中的特殊函数；没有名字，不能有参数也不能有返回值
- 如果在一个到合约的调用中，没有其他函数与给定的函数标识符匹配（或没有提供调用数据），那么这个函数（fallback函数）会被执行
- 每当合约收到以太币（没有任何数据），回退函数就会执行。此外，为了接收以太币，fallback函数必须标记为payable。如果不存在这样的函数，则合约不能通过常规交易接收以太币
- 在上下文中通常只有很少的gas可以用来完成回退函数的调用，所以使fallback函数的调用尽量廉价很重要

### 事件(event)

事件是以太坊EVM提供的一种日志基础设施。事件可以用来做操作记录，存储为日志。也可以用来实现一些交互功能，比如通知UI，返回函数调用结果等

当定义的事件触发时，我们可以将事件存储到EVM的交易日志中，日志是区块链中的一种特殊数据结构；日志与合约关联，与合约的存储合并存入区块链中；只要某个区块可以访问，其相关的日志就可以访问；但在合约中，我们不能直接访问日志和事件数据

可以通过日志实现简单支付验证SPV（Simplified PaymentVerification)如果一个外部实体提供了一个带有这种证明的合约，它可以检查日志是否真实存在于区块链中

### solidity异常处理

Solidity使用“状态恢复异常”来处理异常。这样的异常将撤消对当前调用（及其所有子调用）中的状态所做的所有更改，并且向调用者返回错误。

函数assert和require可用于判断条件，并在不满足条件时抛出异常

- `assert()`一般只应用于测试内部错误，并检查常量
- `require()`应用于确保满足有效条件《如输入或合约状态变量），或验证调用外部合约的返回值
- `revert()`用于抛出异常，它可以标记一个错误并将当前调用回退

## web3.js

- Web3 JavaScript app API
- web3.js是一个JavaScriptAPI库。要使DApp在以太坊上运行，我们可以使用web3.js库提供的web3对象
- web3.js通过RPC调用与本地节点通信，它可以用于任何暴露了RPC层的以太坊节点
- web3包含eth对象-web3.eth（专门与以太坊区块链交互）和shh对象-web3.shh（用于与Whisper交互）

### web3模块加载

首先需要将web3模块安装在项目中：

```
npm install web3@0.20.1 
```

然后创建一个web3实例，设置一个“provider”

为了保证我们的MetaMask设置好的provider不被覆盖掉，在引入 web3之前我们一般要做当前环境检查（以v0.20.1为例）：

```js
if (typeof web3!==undefined) {
    web3 = new Web3(web3.currentProvider); 
} else { 
    web3 = new Web3(new Web3.providers .HttpProvider("http://localhost:8545");
}
```

### 异步回调(callback)

web3jsAPI设计的最初目的，主要是为了和本地RPC节点共同使用，所以默认情况下发送的是同步HTTP请求

如果要发送异步请求，可以在函数的最后一个参数位置上，传入一个回调函数。回调函数是可选（optioanl)的

我们一般采用的回调风格是所谓的“错误优先”，例如：

```js
web3.eth.getBlock(48,function(error,result){ 
    if (!error) {
        console.log(JSON.stringify(result));
    } else {
        console.error(error);
    }
});
```

### 回调Prmise事件(v1.0.0)

为了帮助web3集成到不同标准的所有类型项目中，1.0.0版本提供了多种方式来处理异步函数。大多数的web3对象允许将一个回调函数作为最后一个函数参数传入，同时会返回一个promise用于链式函数调用。

以太坊作为一个区块链系统，一次请求具有不同的结束阶段。为了满足这样的要求，1.0.0版本将这类函数调用的返回值包成一个“承诺事件”（promiEwent)，这是一个promise和EventEmitter的结合体。

PromiEvent的用法就像promise一样，另外还加入了.on，.once和.off方法


```js
web3.eth.sendTransaction({from:'Ox123...',data:'Ox432..'})
.once('transactionHash', function(hash){...}) 
.once('receipt', function(receipt){..})
.on('confirmation', function(confNumber,receipt){...})
.on('error', function(error){...})
.then(function(receipt)({
    //will be fired once the receipt is mined 
});
```

### 应用二进制接口(ABI)

web3.js通过以太坊智能合约的json接口（Application Binary Interface,ABl)创建一个JavaScript对象，用来在js 代码中描述 

函数(functions) 
- type:函数类型，默认`function`，也可能是`constructor`
- constant, payable, stateMutability:函数的状态可变性 
- inputs,outputs:函数输入、输出参数描述列表 

事件(events) 
- type：类型，总是`event`
- inputs：输入对象列表，包括name、type、indexed



