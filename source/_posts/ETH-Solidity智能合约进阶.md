---
title: ETH Solidity智能合约进阶
date: 2020-09-16 16:10:47
categories:
    - Solidity
tags: 
    - 踩坑
    - ETH
    - 区块链
    - 智能合约
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




