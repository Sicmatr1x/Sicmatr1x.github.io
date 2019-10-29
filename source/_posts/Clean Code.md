---
title: Clean Code
date: 2019-10-29 14:13:04
categories:
    - 后端
tags: 
    - Java
---

## 整洁代码

### 单一权责原则(Single Responsibility ,SRP)

一个类只负责一功能领域中的相应职，或者可以定义为：就一个类而言,应该只有一个引起它变化的原因.

修改前:

```java
public class CustomerDataChart{
    public Connection getConnection(){

    }
    public List<Customers> findCustomers(){

    }
    public void createChart(){

    }
    public void displayChart(){

    }
}
```

修改后:

```java
public class DBUtil{
    public Connection getConnection(){

    }
}

public class CustomerDAO{
    private DBUtil util;
    public List<Customers> findCustomers(){

    }
}

public class CustomerDataChart{
    private CustomerDAO dao;
    public void createChart(){

    }
    public void displayChart(){

    }
}
```

---

### 开放闭合原则(Open Closed Principle,OCP)

一个软件实体应当对扩展开放，对修改关闭。即软件实体应尽量在不修改原有代码
的情况下进行扩展。

修改前:

```java
public class PieChart{
    public void display(){
        System.out.println("PieChart:" + type);
    }
}

public class BarChart{
    public void display(){
        System.out.println("BarChart:" + type);
    }
}

public class ChartDisplay {
    public void display(String type){
        switch(type){
            case "PieChart":
                PieChart pie = new PieChart();
                pie.display();
                break;
            case "BarChart":
                BarChart bar = new BarChart();
                bar.display();
                break;
        }
    }
}
```

修改后:

```java
public abstract class AbstractChart{
    public void display();
}

public class PieChart extends AbstractChart{
    @Override
    public void display(){
        System.out.println("PieChart");
    }
}

public class BarChart extends AbstractChart{
    @Override
    public void display(){
        System.out.println("BarChart");
    }
}

public class ChartDisplay{
    private AbstractChart chart;
    public setChart(AbstractChart chart){
        this.chart = chart;
    }

    public void display(){
        chart.display();
    }
}
```

---

### LSP里氏代换原则(Liskov Substitution Principle)

所有引用基类（父类）的地方必须能透明地使用其子类的对象。

修改前:

```java
public class CommonCustomer {
    private String name;
    private String email;
    public getName(){
        return this.name;
    }
    public setName(String name){
        this.name = name;
    }
    public getEmail(){
        return this.email;
    }
    public setEmail(String email){
        this.email = email;
    }
}

public class VIPCustomer {
    private String name;
    private String email;
    public getName(){
        return this.name;
    }
    public setName(String name){
        this.name = name;
    }
    public getEmail(){
        return this.email;
    }
    public setEmail(String email){
        this.email = email;
    }
}

public class EmailSender {
    public send(CommonCustomer customer){
        //send
    }
    public send(VIPCustomer customer){
        //send
    }
}
```

修改后:

```java
public class Customer {
    private String name;
    private String email;
    public getName(){
        return this.name;
    }
    public setName(String name){
        this.name = name;
    }
    public getEmail(){
        return this.email;
    }
    public setEmail(String email){
        this.email = email;
    }
}

public class CommonCustomer extends Customer{

}

public class VIPCustomer extends Customer{

}

public class EmailSender {
    public send(Customer customer){
        //send
    }
}
```

---

### 接口隔离原则(Interface Segregation Principle, ISP)

使用多个专门的接口，而不使用单一的总接口，即客户端不应该依赖那些它不需要
的接口。

修改前:

```java
public interface CustomerDataDisplay {
    public void dataRead();
    public void transformToXML();
    public void createChart();
    public void displayChart();
    public void createReport();
    public void displayReport();
}

public class ConcreteClass implements CustomerDataDisplay{

}
```

修改后:

```java
public interface DataHandler {
    public void dataRead();
}

public interface XMLTransformer {
    public void transformToXML();
}

public interface ChartHandler {
    public void createChart();
    public void displayChart();
}

public interface ReportHandler {
    public void createReport();
    public void displayReport();
}

public class ConcreteClass implements DataHandler, ChartHandler{

}
```

---

### 依赖倒置原则(Dependency Inversion Principle,DIP)

抽象不应该依赖于细节，细节应当依赖于抽象。换言之，要针对接口编程，而不是
针对实现编程。

修改前:

```java
public interface TXTDataConvertor {
    public void addCustomer();
}

public interface ExcelDataConvertor {
    public void addCustomer();
}

public class CustomerDAO implements TXTDataConvertor, ExcelDataConvertor{
    public void addCustomer(){

    }
}
```

修改后:

```java
public interface TXTDataConvertor {
    public void readFile();
}

public interface ExcelDataConvertor {
    public void readFile();
}

public abstract class DataConvertor implements TXTDataConvertor, ExcelDataConvertor{
    public void readFile(){

    }
}

public class CustomerDAO extends DataConvertor{
    public void addCustomer(){

    }
}
```

```xml
...
<className>TX TDataConvertor</className>
...
```

---

## 有意义的命名

### 名副其实

如果名称需要注释来补充，那就不算是名副其实。

### 避免误导 & 做有意义的区分

避免误导：避免留下掩藏代码本意的错误线索。应当避免使用与本意相悖的词。

e.g:

- 以数字系列命名
- 错误的拼写
- 命名一组账号：accountGroup、bunchOfAccounts和accounts 好于 accountList(List对程序员有特殊意义)
- 废话都是冗余。Variable一词永远不应当出现在变量名中。Table一词永远不应当出现在表名中。

### 使用读得出来的名称 & 使用可搜索的名称

使用读的出来的名称：

- genymdhms(生成日期，年、月、日、时、分、秒)，他们一般读作“gen why emm dee aich emm ess”或“gen-yah-mudda-hims”

使用可搜索的名称：

- WORK_DAYS_PER_WEEK 要比数字 5 好找
- 长名称胜于短名称，搜得到的名称胜于用自造编码代写就的名称
- 若变量或常量可能在代码中多处使用，则应赋其以便于搜索的名称

### 避免使用编码

- 如果编译器不做类型检查或动态类型语言，程序员需要匈牙利语标记法来帮助自己记住类型
- 不推荐接口采用IShapeFactory，可以考虑接口ShapeFactory,实现采用ShapeFactoryImp或者丑陋的CShapeFactory，比对接口名称编码好

### 常见误区

- 避免思维映射:不应当让读者在脑中把你的名称翻译为他们熟知的名称。这种问题经常出现在选择是使用问题领域术语还是解决方案领域术语时。
- 类名或对象名应该是名词或名词短语
- 方法名应当是动词或者动词短语
- 重构构造器，使用描述了参数的静态工厂方法名
  ```java
    Complex fulcrumPoint = Complex.FromRealNumber(23.0); 
  ```
- 给每个抽象概念选一个词，并且一以贯之
- 别用双关语
- 使用解决方案领域名称
- 采用从所涉问题领域而来的名称
- 你需要用有良好命名的类、函数或名称空间来放置名称，给读者提供语境,如果没这么做，给名称添加前就是最后一招了

---

## 函数

Function should do one thing. They should do it well. They should do it only.

### 函数的原则

1. 短小:
   - if语句、else语句、while语句等，其中的代码块只能有一行
   - 函数的缩进层级不该多于一层或两层
2. 函数应该做一件事。做好这件事。只做这一件事。
3. 每个函数一个抽象层级:要让每个函数后面都跟着位于下一抽象层级的函数，这样一来，在查看函数列表时，就能偱抽象层级向下阅读了。我把这叫做向下规则。
4. swith语句。

```java
public Money calculatePay(Empoyee e) throws InvalidEmployeeType{
    switch (e.type) {
        case COMMISSIONED:
            return calculateCommissionedPay(e);
        case HOURLY:
            return calculateHourlyPay(e);
        case SELARIED:
            return calculateSalariedPay(e);
        default:
          throw new InvalidEmployeeType(e.type);
    }
}
```

    - 太长，当出现新的雇员类型时，还会变得更长。
    - 明显做了不止一件事。
    - 违反了单以权责原则，有好几个修改的理由。
    - 违反了开放闭合原则，每当添加新类型时，就必须修改之。
    - 可能会到处出现类似的结构函数

可以改成:

```java
public abstract class Employee {
    public abstract boolean isPayday();
    public abstract Money calculatePay();
    public abstract void deliverPay();
}

public interface EmployeeFactory {
    public Employee makeEmployee(EmployeeRecord r) throws InvalidEmployeeType;
}

public class EmployeeFactoryImpl implements EmployeeFactory {
    public Employee makeEmployee(EmployeeRecord r) throws InvalidEmployeeType{
        switch (e.type) {
            case COMMISSIONED:
              return CommissionedEmployee(r);
            case HOURLY:
              return HourlyPayEmployee(r);
            case SELARIED:
              return SalariedPayEmployee(r);
            default:
              throw new InvalidEmployeeType(e.type);
        }
    }
}
```

将swith语句埋到抽象工厂下，该工厂使用 switch 语句为 Employee 的派生物创建适当的实体，而不同的函数，如 calculatePay、isPayday 和 deliverPay等，则藉由 Employee 接口多态地接受派遣。对于 switch 语句，我的规矩是如果只出现一次，用于创建多态对象，而且隐藏在某个继承关系中。

5. 使用描述性的名称。
6. 函数参数个数 <= 3。
7. 无副作用
8. 分割指令与询问。函数要么做什么事，要么回答什么事，但二者不可得兼。函数应该修改某对象的状态或是返回该对象的有关信息。两样都干常会导致混乱。
9. 使用异常替代返回错误码。从指令式函数返回错误码轻微违反了指令与询问分隔的规则。它鼓励了在if 语句判断中把指令当作表达式使用。这不会引起动词/形容词混淆，但却导致更深层次的嵌套结构。当返回误码时，就是在要求调用者立刻处理错误。另一方面，如果使用异常替代返回错误码，错误处理代码就能从主路径代码中分离出来。
10. 别重复自己。
11. 结构化编程。每个函数、函数中的每个代码块都应该有一个入口、一个出口。每个函数中之该有一个return语句。

### 函数修改的策略

大师级程序员把系统当作故事来讲，而不是当作程序来写。他们使用选定编程语言提供的工具构建一种更为丰富且更具表达力的语言，用来讲那个故事。那种领域特定语言的一个部分，就是描述在系统中发生的各种行为的函数层级。在一种狡猾的递归操作中，这些行为使用它们定义的与领域紧密相关的语言讲述自己那个小故事。

---

## 注释

注释不能美化糟糕的代码

好注释:

- 唯一真正好的注释是你想办法不去写的注释
- 法律信息:公司代码规范要求编写与法律有关的注释，只写引用即可
- 阐释:注释把某些晦涩难明的参数或返回值的意义翻译为某种可读形式
- 警示:警告其他程序员会出现某种后果的注释。
- TODO注释:有理由在源代码中放置要做的工作列表。定期查看，删除不再需要的。
- 放大:注释可以用来放大某种看来不合理之物的重要性。
- 公共API中的Javadoc,Javadoc也可能误导、不适用或者提供错误信息

坏注释:

- 喃喃自语
- 多余的注释
- 误导性注释
- 循规式注释
- 日志式注释:应由源代码控制系统来管理
- 废话注释
- 能用函数或变量时就别用注释
- 位置标记
- 括弧后面的注释:while,if等的多层嵌套，用短小的封装的函数代替
- 归属与签名:源代码控制系统是这类信息最好的归属地
- 注释掉的代码
- HTML注释
- 非本地信息:别在本地注释的上下文环境中给出系统级的信息
- 信息过多:有趣的历史话题或者无关的细节描述
- 不明显的联系
- 函数头
- 非公共代码中的JavaDoc

---

## 格式

### 格式的目的

- 垂直格式
- 横向格式
  - 尽力保持代码行短小。无需拖动滚动条到右侧的原则。
  - 水平对齐。
  - 缩进。
- 团队风格
  - 项目开始时制定一套编码风格，将规则编写进IDE。

---

## 对象和数据结构

### 数据抽象

类并不简单地用取值器和赋值器将其变量推向外间，而是曝露抽象接口，以便用户无需了解数据的实现就能操作数据本体。过程式代码难以添加新数据结构，因为必须修改所有函数。面向对象代码难以添加新函数，因为必须修改所有类。

### 得墨忒耳律

模块不应了解所操作对象的内部情形。得墨忒耳律认为，类C的方法f只应该调用以下对象的方法：

- C
- 由f创建的对象
- 作为参数传递给f的对象
- 由C持有的实体变量持有的对象

### 数据传送对象

DTO（Data Transfer Objects）经常用作与数据库通信、或解析套接字传递的消息之类场景中。

---

## 错误处理

1. 使用异常而非返回码
2. 先写Try-Catch-Finally语句

### 异常的使用

- 使用不可控异常：可控异常的代价就是违反开放/闭合原则。//TODO:?
- 给出异常发生的环境说明,都应当提供足够的环境说明，以便判断错误的来源和处所。
- 定义常规流程：采用特例模式。创建一个类或配置一个对象，用来处理特例.
- 别返回null值：在方法中返回null值，不如抛出异常，或是返回特例对象

---

## 边界

### 整洁的边界

应该避免我们的代码过多地了解第三方代码中的特定信息。在这种场景下，适配器模式是非常好的设计，它不仅能将不兼容的接口改写成兼容的接口，还能够对通过对第三方工具重新封装来避免边界的变化对系统的影响。

---

## 单元测试

### TDD三定律

1. 在编写不能通过的单元测试前，不可编写生产代码。
2. 只可编写刚好无法通过的单元测试，不能编译也算不通过。
3. 只可编写刚好足以通过当前失败测试的生产代码。

### 整洁的测试

- 测试与生产代码一起编写，这样可以保证：测试将覆盖所有生产代码。
- 整洁测试的三个要素：可读性、可读性、可读性。要明确，简洁，有足够的表达力。
- 测试呈现构造-操作-检验（BUILD-OPERATE-CHECK）模式。
- 生产环境和测试环境可以用双重标准。
- 每个测试一个断言。每个测试一个概念。
- Given-when-then：Given在某种场景下 When发生了事件 Then导致了什么结果。

### F.I.R.S.T

整洁的测试环境遵循以下5条规则：

1. 快速(Fast)
   - 测试应该快速，因为需要不断的运行测试得到反馈，我们需要的快速反馈，错误的快速定位。所以你的测试就不能依赖太多的外部资源，数据库，硬件环境等等，对于这些外部资源应该采用伪对象模式来隔离。
2. 独立(Independent)
   - 测试应该是相互独立的，独立于测试用例之间，独立于特定的环境，独立于测试的运行顺利。
   - 数据的独立方式:
     1. 每个测试环境的独立
     2. 数据的隔离
3. 可重复(Repeatable)
    - 测试应该可以在任何环境中重复通过，可运行，因为测试独立于环境外部资源。
4. 自足验证(Self-Validation)
   - 测试应该有通过失败的标示，从每一个测试上能得到一处代码逻辑的通过失败。
5. 及时(Timely)
   - 测试应该是及时编写的。TDD要求测试必须在实现代码之前，提前以使用者的角度定义使用接口方式。

---

## 类

### 类的组织

类应该从一组变量列表开始。如果有公共静态常量，应该先出现。然后是私有静态变量，以及私有实体变量。很少会有公共变量。公共函数应更在变量列表之后，把由某个公共函数调用的私有工具函数紧随在该公共函数后面。符合自定向下的原则。

内聚：类应该只有少量实体变量。

### 为了修改而组织

- 开放-闭合原则（OCP）：类应当对扩展开放，对修改封闭。
- 隔离修改：具体类包含实现细节，而抽象类只呈现概念。可以借助接口和抽象类来隔离细节修改带来的风险。
- 依赖倒置原则（Dependency Inversion Principle,DIP）:类应该依赖于抽象而不是依赖于具体细节。

---

## 系统

### 将系统的构造与使用分开

软件系统应将启始过程和启始过程之后的运行时逻辑分离开，在启始过程中构建应用对象，也会存在相互缠结的依赖关系。依赖注入（Dependency Injection,DI）机制,可以实现分离构造与使用。控制反转（Inversion of Control,IoC）将第二权责从对象中拿出来，转移到另一个专注于此的对象中，从而遵循了单一权责原则。

### 扩容

我们应该只去实现今天的用户故事，然后重构，明天再扩展系统、实现新的用户故事。这就是迭代和增量敏捷的精髓所在。测试驱动开发、重构以及他们打造出的整洁代码，在代码层面保证了这个过程的实现。

### 横切关注点

横切关注点（Cross-Cutting Concerns）：在AOP中，被称为方面（aspect）的模块构造指明了系统中哪些点的行为会以某种一致的方式被修改，从而支持某种特定的场景。

Java代理：适用于简单的情况，例如在单独的对象或类中包装方法调用。

纯Java AOP框架：如Spring AOP和Jboss AOP等，在概念上更简单、更易于测试驱动。

### 测试驱动系统架构

用POJO编写应用程序的领域逻辑，在代码层面与架构关注面分离开，就有可能真正地用测试来驱动架构。

没必要先做大设计（Big Design Up Front，DBUF）。实际上，它阻碍改进，架构上的方案选择影响到后续的设计思路

### 系统需要领域特定语言

领域特定语言（Domain-Specific Language,DSL）：在有效使用时能提升代码惯用法和设计模式之上的抽象层次。允许所有抽象层级和应用程序中的所有领域，从高级策略到底层细节，使用POJO来表达。

---

## 迭代

关于简单设计的四条规则：

1. 运行所有测试；
2. 不可重复；
3. 表达了程序员的意图；
4. 尽可能减少类和方法的数量；
5. 以上规则按其重要程度排列。

---

## 并发编程

并发是一种解耦策略，帮助把做什么（目的）和何时（时机）做分解开。

解耦目的与时机能明显地改进应用程序的吞吐量和结构。

### 挑战与并发防御原则

单一权责原则

限制数据作用域（采用`synchronized`保护临界区）

建议：谨记数据封装，严格限制对可能被共享的数据的访问。

推论：使用数据副本：假使使用对象复本能避免代码同步执行，则因避免了锁定而省下的价值有可能补偿得上额外的创建陈本和垃圾收集开销。

推论：线程应尽可能地独立

建议：尝试将数据分解到可被独立线程（可能在不同处理器上）操作的独立子集。

### 了解执行模型

### 警惕同步方法之间的依赖

### 保持同步区域微小

---

## Code Smell List

### 注释

1. 不恰当的注释
    - 让不恰当的注释保存到源代码控制系统。
2. 废弃的注释
    - 过时 、无关或不正确的注释就是废弃的注释不应该保留必马上删除。
3. 冗余的注释
    - 注释应该谈及代码自身没提到的东西，否则就是冗余的。
4. 糟糕的注释
    - 值得编写的注释必须正确写出最好的注释，如果不是就不写 。
5. 注释掉的代码
    - 注释掉的代码必须删除。

### 环境

1. 需要多步才能实现的构建:构建系统应该是单步的小操作 。
2. 需要多步才能实现的测试:只需要单个指令就可以运行所有单元测试。

### 函数

1. 过多的参数:函数参数应该越少越好，坚决避免有3个参数 的函数
2. 出参数:出参数违反直接，抵制出参数
3. 标识参数:布尔值参数令人迷惑，应该消灭掉
4. 死函数:永不被调用函数应该删除掉

### 一般性问题

1. 一个源文件存在多个语言
  - 尽量减少源文件语言的数量和范围。
2. 明显的行为未被实现
  - 遵循 “最少惊异原则 ”，函数或者类应该实现其他程序员有理由期待的行为，不要让其他程序员看代码才清楚函数的作用。
3. 不正确的边界行为
  - 代码应该有正确的行为，追索每种边界条件并进行全面 测试。
4. 忽视安全
  - 关注可能引起问题的代码，注重安全与稳定。
5. 重复
  - 消除重复代码，使用设计模式。
6. 在错误的抽象层级上的代码
  - 抽象类和派类概念模型必须完整分离，例如 ：与实现细节有关的代码不应该在基类中出现。
7. 基类依赖于派类
  - 基类应该对派类一无所知。
8. 信息过多
  - 类中的方法，变量越少越好，隐藏所有实现，公开接口越少越好。
9. 死代码
  - 找到并删除所有不被调用的代码。
10. 垂直分隔
  - 变量和函数的定义应该靠近被调用代码。
11. 前后不一致
  - 函数参数变量应该从一而终，保持一致，让代码便于阅读和修改。
12. 混淆视听
  - 没用的变量，不被调用的函数，没有信息量的注释应该清理掉。
13. 人为耦合
  - 不互相依赖的东西不该耦合。
14. 特性依恋
  - 类的方法应该只对自身的方法和变量感兴趣，不应该垂青其他类的方法和变量。
15. 选择算参数
  - 避免布尔类型参数，使用多态代替。
16. 晦涩的意图
  - 代码要尽可能具有表达力，明白的意图比高效和性能重要。
17. 位置错误的权责
  - “最少惊异原则 ”，把代码放在读者想到的地方，而不是对自己方便的地方。
18. 不恰当的静态方法
  - 如果要使用静态方法，必须确保 没机会打算让它有多态行为。
19. 使用解释性变量
  - 把计算过程打散成一系列命名良好的中间值使程序更加可读性。
20. 函数名称应该表达其行为
21. 理解算法
22. 把逻辑依赖改为物理依赖
  - 依赖应该是明显而不应该是假设的依赖。
23. 用多态替代If/Else或Switch/Case
24. 遵循标准约定
25. 用命名常量替代魔术数
26. 准确
  - 代码中的含糊和不准确要么是意见不同的结果，要么源于懒散，都必须消除。
27. 结构甚于约定
28. 封装条件
  - 把条件封装成方法。
29. 避免否定性条件
  - 使用肯定性条件。
30. 函数只该做一件事
31. 掩蔽时序耦合
  - 创建顺序队列暴露时序耦合，每个函数都产一下函数所需参数，就可保障正确的时序。
32. 别随意
  - 代码不能随意，需要谨慎考虑。
33. 封装边界条件
  - 例如 ：+1或-1操作必须封装起来。
34. 函数应该只在一个抽象层级上
  - 封装不在一个抽象层级上的代码，保持每个函数只在一个抽象层级上。
35. 在较高层级放置可配置数据
  - 把配置数据和常量放到基类里。
36. 避免传递浏览
  - “得墨忒耳律 ”，编写害羞代码，让直接协作者提供所需的服务，而不要逛遍整个系统。

### 名称

1. 采用描述性名称
  - 名称对应可读性有 90%的作用，必须认真命名。
2. 名称应与抽象层级相符
  - 不要取沟通实现的名称：取反映类或函数抽象层级的名称。
3. 尽可能使用标准命名法
4. 无歧义的名称
5. 为较大作用范围选用较长名称
6. 避免编码
  - 不应该在名称中包含类型或范围的信息，例如：m_，f等前缀。
7. 名称应该说明副作用
  - 名称应该说明类 、变量或函数的所有信息，不应该 隐藏副作用。

### 测试

1. 测试不足
  - 保证足够的测试。
2. 使用覆盖率工具
  - 覆盖率工具可以更好地找到测试不足的模块 、类、函数。
3. 别略过小测试
4. 被忽略的测试就是对不确定事物的疑问
  - 用 `@Ignore` 表达我们对需求的疑问。
5. 测试边界条件
  - 边界判读错误很常见，必须测试边界条件。
6. 全面测试相近的缺陷
  - 缺陷趋向于扎堆，如果在函数中发现一个缺陷，那么就全面测试这个函数。
7. 测试失败的模式有启发性
  - 你可以通过测试失败找到问题所在。
8. 测试覆盖率的模式有启发性
  - 通过测试覆盖率检查，往往可以找 到测试失败的线索。
9. 测试应该快速
  - 慢测试会导致时间紧时会跳过，导致可能出现问题。
