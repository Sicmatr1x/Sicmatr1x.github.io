---
title: JavaScript 设计模式
date: 2019-12-06 17:11:36
categories:
    - Design Patterns
tags: 
    - JavaScript
cover: /img/article-cover/jigsaw_puzzle_hires.jpg
---


## 设计原则

1. 单一职责原则（SRP）：一个对象或方法只做一件事情。如果一个方法承担了过多的职责，那么在需求的变迁过程中，需要改写这个方法的可能性就越大。应该把对象或方法划分成较小的粒度

2. 最少知识原则（LKP）：一个软件实体应当 尽可能少地与其他实体发生相互作用。应当尽量减少对象之间的交互。如果两个对象之间不必彼此直接通信，那么这两个对象就不要发生直接的 相互联系，可以转交给第三方进行处理

3. 开放-封闭原则（OCP）：软件实体（类、模块、函数）等应该是可以 扩展的，但是不可修改。当需要改变一个程序的功能或者给这个程序增加新功能的时候，可以使用增加代码的方式，尽量避免改动程序的源代码，防止影响原系统的稳定

## 设计模式

### 单例模式

保证一个类仅有一个实例，并提供一个访问它的全局访问点

```js
function SetManager(name) {
    this.manager = name;
}

SetManager.prototype.getName = function() {
    console.log(this.manager);
};

var SingletonSetManager = (function() {
    var manager = null;

    return function(name) {
        if (!manager) {
            manager = new SetManager(name);
        }

        return manager;
    }
})();

SingletonSetManager('a').getName(); // a
SingletonSetManager('b').getName(); // a
SingletonSetManager('c').getName(); // a
```

### 策略模式

定义一系列的算法，把它们一个个封装起来，并且使它们可以相互替换。

将算法的使用和算法的实现分离开来。

一个基于策略模式的程序至少由两部分组成：

第一个部分是一组策略类，策略类封装了具体的算法，并负责具体的计算过程。

第二个部分是环境类Context，Context接受客户的请求，随后把请求委托给某一个策略类。要做到这点，说明Context 中要维持对某个策略对象的引用

```js
// 加权映射关系
var levelMap = {
    S: 10,
    A: 8,
    B: 6,
    C: 4
};

// 组策略
var scoreLevel = {
    basicScore: 80,

    S: function() {
        return this.basicScore + levelMap['S']; 
    },

    A: function() {
        return this.basicScore + levelMap['A']; 
    },

    B: function() {
        return this.basicScore + levelMap['B']; 
    },

    C: function() {
        return this.basicScore + levelMap['C']; 
    }
}

// 调用
function getScore(level) {
    return scoreLevel[level] ? scoreLevel[level]() : 0;
}

console.log(
    getScore('S'),
    getScore('A'),
    getScore('B'),
    getScore('C'),
    getScore('D')
); // 90 88 86 84 0
```

### 代理模式

为一个对象提供一个代用品或占位符，以便控制对它的访问

当客户不方便直接访问一个 对象或者不满足需要的时候，提供一个替身对象 来控制对这个对象的访问，客户实际上访问的是 替身对象。

替身对象对请求做出一些处理之后， 再把请求转交给本体对象

代理和本体的接口具有一致性，本体定义了关键功能，而代理是提供或拒绝对它的访问，或者在访问本体之前做一 些额外的事情

代理模式主要有三种：保护代理、虚拟代理、缓存代理

```js
// 主体，发送消息
function sendMsg(msg) {
    console.log(msg);
}

// 代理，对消息进行过滤
function proxySendMsg(msg) {
    // 无消息则直接返回
    if (typeof msg === 'undefined') {
        console.log('deny');
        return;
    }
    
    // 有消息则进行过滤
    msg = ('' + msg).replace(/泥\s*煤/g, '');

    sendMsg(msg);
}


sendMsg('泥煤呀泥 煤呀'); // 泥煤呀泥 煤呀
proxySendMsg('泥煤呀泥 煤'); // 呀
proxySendMsg(); // deny
```

### 迭代器模式

迭代器模式是指提供一种方法顺序访问一个聚合对象中的各个元素，而又不需要暴露该对象的内部表示。

在使用迭代器模式之后，即使不关心对象的内部构造，也可以按顺序访问其中的每个元素

```js
function each(obj, cb) {
    var value;

    if (Array.isArray(obj)) {
        for (var i = 0; i < obj.length; ++i) {
            value = cb.call(obj[i], i, obj[i]);

            if (value === false) {
                break;
            }
        }
    } else {
        for (var i in obj) {
            value = cb.call(obj[i], i, obj[i]);

            if (value === false) {
                break;
            }
        }
    }
}

each([1, 2, 3], function(index, value) {
    console.log(index, value);
});

each({a: 1, b: 2}, function(index, value) {
    console.log(index, value);
});

// 0 1
// 1 2
// 2 3

// a 1
// b 2
```

### 发布-订阅模式

也称作观察者模式，定义了对象间的一种一对多的依赖关系，当一个对象的状态发 生改变时，所有依赖于它的对象都将得到通知

取代对象之间硬编码的通知机制，一个对象不用再显式地调用另外一个对象的某个接口。

与传统的发布-订阅模式实现方式（将订阅者自身当成引用传入发布者）不同，在JS中通常使用注册回调函数的形式来订阅

```js
// 订阅
document.body.addEventListener('click', function() {
    console.log('click1');
}, false);

document.body.addEventListener('click', function() {
    console.log('click2');
}, false);

// 发布
document.body.click(); // click1  click2
```

### 命令模式

用一种松耦合的方式来设计程序，使得请求发送者和请求接收者能够消除彼此之间的耦合关系

命令（command）指的是一个执行某些特定事情的指令

命令中带有execute执行、undo撤销、redo重做等相关命令方法，建议显示地指示这些方法名

```js
// 自增
function IncrementCommand() {
    // 当前值
    this.val = 0;
    // 命令栈
    this.stack = [];
    // 栈指针位置
    this.stackPosition = -1;
};

IncrementCommand.prototype = {
    constructor: IncrementCommand,

    // 执行
    execute: function() {
        this._clearRedo();
        
        // 定义执行的处理
        var command = function() {
            this.val += 2;
        }.bind(this);
        
        // 执行并缓存起来
        command();
        
        this.stack.push(command);

        this.stackPosition++;

        this.getValue();
    },
    
    canUndo: function() {
        return this.stackPosition >= 0;
    },
    
    canRedo: function() {
        return this.stackPosition < this.stack.length - 1;
    },

    // 撤销
    undo: function() {
        if (!this.canUndo()) {
            return;
        }
        
        this.stackPosition--;

        // 命令的撤销，与执行的处理相反
        var command = function() {
            this.val -= 2;
        }.bind(this);
        
        // 撤销后不需要缓存
        command();

        this.getValue();
    },
    
    // 重做
    redo: function() {
        if (!this.canRedo()) {
            return;
        }
        
        // 执行栈顶的命令
        this.stack[++this.stackPosition]();

        this.getValue();
    },
    
    // 在执行时，已经撤销的部分不能再重做
    _clearRedo: function() {
        this.stack = this.stack.slice(0, this.stackPosition + 1);
    },
    
    // 获取当前值
    getValue: function() {
        console.log(this.val);
    }
};
```

### 组合模式

是用小的子对象来构建更大的 对象，而这些小的子对象本身也许是由更小 的“孙对象”构成的。

可以用树形结构来表示这种“部分- 整体”的层次结构。

调用组合对象 的execute方法，程序会递归调用组合对象 下面的叶对象的execute方法

但要注意的是，组合模式不是父子关系，它是一种HAS-A（聚合）的关系，将请求委托给它所包含的所有叶对象。基于这种委托，就需要保证组合对象和叶对象拥有相同的接口

此外，也要保证用一致的方式对待 列表中的每个叶对象，即叶对象属于同一类，不需要过多特殊的额外操作

### 模板方法模式

模板方法模式由两部分结构组成，第一部分是抽象父类，第二部分是具体的实现子类。

在抽象父类中封装子类的算法框架，它的 init方法可作为一个算法的模板，指导子类以何种顺序去执行哪些方法。

由父类分离出公共部分，要求子类重写某些父类的（易变化的）抽象方法

模板方法模式一般的实现方式为继承

以运动作为例子，运动有比较通用的一些处理，这部分可以抽离开来，在父类中实现。具体某项运动的特殊性则有自类来重写实现。

最终子类直接调用父类的模板函数来执行