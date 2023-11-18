---
title: Node.js Unit Test Framework
date: 2019-10-28 17:13:04
categories:
    - Back-end
tags: 
    - Node.js
    - Unit test
cover: /img/article-cover/nodejs.png
---

基于Node.js的 Unit Test Framework，用到了如下几个工具：
- [mocha](https://mochajs.org): Mocha is a feature-rich JavaScript test framework.
- [sinon.js](https://sinonjs.org/): Standalone test spies, stubs and mocks for JavaScript.
- [chai.js](https://www.chaijs.com/): Chai is a BDD / TDD assertion library.
- [supertest](https://github.com/visionmedia/supertest): Super-agent driven library for testing node.js HTTP servers using a fluent API.
- [istanbul.js](https://istanbul.js.org): JavaScript test coverage made simple.

## 安装与配置

### 安装

```bash
npm install mocha -S
npm install sinon -S
npm install chai -S
npm install supertest -S
npm install istanbul -S
```

### 配置

1. 在项目的根目录下创建test文件夹，如果项目是分层结构的话可以在test下创建子目录
2. 添加mocha启动命令，参数为需要扫描的含有unit test文件的文件夹

```bash
mocha test/controller test/routes test/common test/service
```

为方便起见可以将命令添加到package.json

```json
  "scripts": {
    "unit-test": "mocha test/controller test/routes test/common test/service"
  }
```

## 编写 Unit Test

为方便查看unit test运行结果，可以采用每一个js文件对应一个Unit Test文件，每一个function对应一个组的结构来编写

### Given-When-Then的case命名法

Definition

The Given-When-Then formula is a template intended to guide the writing of acceptance tests for a User Story:

- Given: Set of preconditions
- When: When an event occurs, or some action is carried out
- Then: What outcome is achieved, or a particular set of observable consequences should obtain

Example：测试对象是人
- Given：人的上下文预设：这是一个诗人
- When：对人的操作：把这个人放在太平盛世中
- Then：对人产生的结果：这个人将获得富足的生活
- 则test case命名为：Given_一个人职业是诗人_When_活在太平盛世_Then_这个人生活富足

### 一个 test case的结构

一个test case分为3部分：
1. mock函数部分
2. Given步骤: 准备测试数据部分
3. When步骤：调用测试函数
4. Then步骤：断言返回结果

### mocha中Unit Test的结构

```js
describe('userController', function() {
  describe('#findUserByPaging()', function() {
    // 准备测试数据
    ...
    // mock or stub function if need
    let sandbox;
    beforeEach(function() {
      sandbox = sinon.createSandbox();
    });
    afterEach(function() {
      sandbox.restore()
    });
    // test case
    it('should return limit user list when given offset limit', (done) => {
      userController.findUserByPaging(searchCriteria).then((result) => {
        // Then
        assert.equal(result.success, true) // assert断言
        done() // 表示一个Test Case结束
      }).catch((err) => {
        done(err) // 表示一个Test Case结束
      })
    })
  })
})
```

若断言不通过或者运行中抛出一个异常则会显示测试失败

### Test Case Example

本例中用到的方法：

- mocha:
  - `describe()`
  - `done()`
- sinon.js
  - `createSandbox()`
  - `stub`
- chai.js
  - `assert.equal()`
  - `assert.deepEqual()`


```js
  describe('#findUserByPaging()', function() {
    let sandbox;
    // Given
    const searchResult = {
      "totalCount": 1,
      "data": [
        {
          "userId": 1,
          "userDomain": "TESTUSER",
          "userEmail": "test.user@outlook.com",
          "userTel": "1234567890",
          "userTeam": "MBC",
          "roleId": 1,
          "roleName": "Super Admin"
        }
      ]
    }
    const searchCriteria = {
      userDomain: 'TESTUSER',
      offset: 0,
      limit: 10
    }
    const expectData = {
      "data": [
        {
          "role": [
            {
              "roleId": 1,
              "roleName": "Super Admin"
            }
          ],
          "userDomain": "TESTUSER",
          "userEmail": "test.user@outlook.com",
          "userId": 1,
          "userTeam": "MBC",
          "userTel": "1234567890"
        }
      ],
      "limit": 10,
      "offset": 0,
      "totalCount": 1
    }
    // mock function
    beforeEach(function() {
      sandbox = sinon.createSandbox();
      sandbox.stub(UserRepo, "findUserByPaging").withArgs(sinon.match.any).returns(searchResult)
    });
    afterEach(function() {
      sandbox.restore()
    });
    it('should return limit user list when given offset limit', (done) => {
      // When
      userController.findUserByPaging(searchCriteria).then((result) => {
        // Then
        assert.equal(result.success, true)
        assert.deepEqual(result.data, expectData)
        done()
      }).catch((err) => {
        done(err)
      })
    })
  })
```

对应上述代码：
- Given部分给出了提供的数据，`searchCriteria`和`searchResult`对象
- When部分调用了`findUserByPaging`方法
- Then对返回结果进行了断言，判断了是否与预期结果`expectData`符合
- 本例中对`findUserByPaging`方法所调用到的`UserRepo`对象的`findUserByPaging`方法进行了打桩并使其返回了我们对其预期的返回数据`searchResult`

####  mock or stub

如上述代码所示，使用到了sinon.js的`createSandbox`和`stub`方法

- [sandbox](https://sinonjs.org/releases/v7.5.0/sandbox/)
- [stub](https://sinonjs.org/releases/v7.5.0/stubs/)

> Default sandbox: Since sinon@5.0.0, the sinon object is a default sandbox. Unless you have a very advanced setup or need a special configuration, you probably want to just use that one.

与直接使用`sinon.stub()`比起来使用`sandbox`要更安全，而且在部分情况下前者会出现stub不掉的情况，而后者则不会

```js
"test should call all subscribers, even if there are exceptions" : function(){
    var message = 'an example message';
    var stub = sinon.stub().throws();
    var spy1 = sinon.spy();
    var spy2 = sinon.spy();

    PubSub.subscribe(message, stub);
    PubSub.subscribe(message, spy1);
    PubSub.subscribe(message, spy2);

    PubSub.publishSync(message, undefined);

    assert(spy1.called);
    assert(spy2.called);
    assert(stub.calledBefore(spy1));
}
```

```js
// Creates a new sandbox object with spies, stubs, and mocks.
var sandbox = sinon.createSandbox();
beforeEach(function() {
    sandbox = sinon.createSandbox();
    sandbox.stub(UserRepo, "findUserByPaging").withArgs(sinon.match.any).returns(searchResult)
});
afterEach(function() {
    sandbox.restore()
});
```

#### 使用 supertest来编写模拟HTTP请求的 Unit Test

```js
  describe('/nj_dom_notification/users/page', function() {
    let sandbox;
    const data = [
      {
        "userId": 1,
        "userDomain": "TESTUSER",
        "userEmail": "test.user@outlook.com",
        "userTel": "1234567890",
        "userTeam": "MBC",
        "role": [
          {
            "roleId": 1,
            "roleName": "Super Admin"
          }
        ]
      }
    ]
    beforeEach(function() {
      sandbox = sinon.createSandbox();
      sandbox.stub(userController, "findUserByPaging").callsFake(() => {
        return responseHelper.responsePageSuccess(data)
      })
    });
    afterEach(function() {
      sandbox.restore();
    });

    it('should get user list json when call findUserByPaging', (done) => {
      request(app)
        .get('/nj_dom_notification/users/page?userDomain=TESTUSER&limit=10&offset=0')
        .set('Accept', 'application/json')
        .expect('Content-Type', 'application/json; charset=utf-8')
        .expect(200, done)
        .expect(function(res) {
          assert.equal(res.body.success, true);
        })
    })
  })
```

#### 使用 Istanbul.js来统计 Unit Test的代码覆盖率

```json
{
 "scripts": {
   "unit-test-cov": "istanbul cover node_modules/mocha/bin/_mocha -- test/controller test/routes test/common test/service"
 }
}
```

只需要在`istanbul cover`命令后面加上mocha的命令即可

若有不想统计的代码可以使用`/* istanbul ignore next */`注释来忽略掉，但是istanbul不支持文件忽略，只支持函数、选择条件级别的忽略。

