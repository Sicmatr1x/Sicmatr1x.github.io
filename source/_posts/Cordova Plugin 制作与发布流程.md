---
title: Cordova Plugin 制作与发布流程
date: 2018-12-21 17:47:04
categories:
    - Hybrid App
tags: 
    - Cordova
mp3: http://domain.com/awesome.mp3
cover: /img/article-cover/cordova_bot.png
preview: 300
---

## 参考

- [https://blog.csdn.net/Yoryky/article/details/78516291](https://blog.csdn.net/Yoryky/article/details/78516291)
- [Plugin Development Guide](https://cordova.apache.org/docs/en/latest/guide/hybrid/plugins/index.html)

## 制作 Cordova Plugin

1. <a href="#createPlugin">创建一个新的插件</a>
2. <a href="#editPlugin">在已有的插件上修改代码并重新打包</a>

### <a name="createPlugin">创建插件</a>

#### 创建一个 Cordova 测试项目 hello 用于测试插件

#### 以加法器为例, 命令行创建一个名称为 Adder, id为 joe.adder 的插件, 初始版本为1.0.0：

```bash
hello> plugman create -name Adder -plugin_id cordova-plugin-adder -plugin_version 1.0.0
```

#### 用命令行声明这个插件是为Android平台服务的：这个命令会自动生成一个Adder.java

```bash
cd Adder

hello\Adder> plugman platform add --platform_name android
```

#### 重写 Adder.java 的 `execute` 方法

```java
    @Override
    public boolean execute(String action, JSONArray args, CallbackContext callbackContext) throws JSONException {
        if (action.equals("performAdd")) {
            int result = args.getInt(0) + args.getInt(1);
            callbackContext.success("result calculated in Java: " + result);
            return true;
        }
        return false;
    }
```

#### 修改 Adder.js

注入插件 js 函数到 windows 对象:

修改 Adder.js

```js
var exec = require('cordova/exec');

module.exports.add = function(arg0, success, error) {
    exec(success, null, "Adder", "performAdd", arg0);
};
```

#### 为插件添加 package.json

```bash
hello\Adder> plugman createpackagejson "./"
```

或

```bash
plugman createpackagejson /path/to/your/plugin
```

#### 将插件添加到混合应用中

```bash
hello> cordova plugin add Adder
Installing "joe.adder" for android
Android Studio project detected
Installing "joe.adder" for ios
Saved plugin info for "joe.adder" to config.xml
```

#### 测试

```js
cordova.plugins.Adder.add([1,2],function(msg){console.log(msg);},null);
```

[参考](https://cordova.apache.org/docs/en/latest/guide/hybrid/plugins/index.html)

---

## <a name="editPlugin">在已有的插件上修改代码并重新打包</a>

1. 下载插件源码
2. 修改需要修改的代码
3. 若需要修改插件名称则需要修改`package.json`与`plugin.xml`中的插件`name`属性与`id`属性


---

## 发布 Plugin

### 发布前版本号修改

#### 根据需要修改并补全插件的 package.json 文件

**注意**: `version`: 每次发布到 NPM 都需要更改

```json
{
  "name": "cordova-plugin-adder",
  "version": "1.1.4",
  "description": "test cordova plugin", // 插件简介，用于NPM
  "cordova": {
    "id": "cordova-plugin-adder", // 插件id，同插件名
    "platforms": [
      "android",
      "ios"
    ]
  },
  "keywords": [ // NPM搜索关键字
    "ecosystem:cordova",
    "test",
    "android",
    "add",
    "integer"
  ],
  "author": "Joe",
  "license": "Apache 2.0",
  "repository": { // 仓库地址
    "type": "git",
    "url": "git+https://github.com/J0e9u0/cordova-plugin-adder"
  },
  "bugs": { // issue反馈地址
    "url": "https://github.com/J0e9u0/cordova-plugin-adder/issues"
  },
  "homepage": "https://github.com/J0e9u0/cordova-plugin-adder#readme"  // 项目地址
}
```

#### 根据需要修改 plugin.xml 文件

plugin.xml 参数说明：
- id：插件唯一标识
- version：版本号
- js-module
  - src：js中间件相对文件地址（www目录下的那个js）
  - name：模块名称
  - clobbers/merges
    - target：H5通过它调用js中间件方法（ts调用方法的前缀）
- platform
  - name：对应平台android | ios
  - source-file
      - src：类名
      - tartget-dir：插件文件复制到到原生项目位置
      - feature
        - name：js中间件通过它调用原生方法（包名）
      - uses-permission：相关原生权限

**注意**: `version`: 要与 package.json 保持一致

```xml
<?xml version='1.0' encoding='utf-8'?>
<plugin id="cordova-plugin-adder" version="1.1.8" 
  xmlns="http://apache.org/cordova/ns/plugins/1.0" 
  xmlns:android="http://schemas.android.com/apk/res/android">
  <name>Adder</name>
  <js-module name="Adder" src="www/Adder.js">
    <clobbers target="cordova.plugins.Adder" />
  </js-module>
  <platform name="android">
    <config-file parent="/*" target="res/xml/config.xml">
      <feature name="Adder">
        <param name="android-package" value="org.apache.cordova.adder.Adder" />
      </feature>
    </config-file>
    <config-file parent="/*" target="AndroidManifest.xml"></config-file>
    <source-file src="src/android/Adder.java" target-dir="src/org/apache/cordova/adder" />
  </platform>
</plugin>
```

[参考](https://cordova.apache.org/docs/en/latest/plugin_ref/spec.html)

### 发布到 NPM

#### [注册 NPM 账号](https://www.npmjs.com/)

#### 本地登录账号

```bash
npm adduser
npm login
```

#### 上传插件

进入插件根目录

```bash
npm publish
+ cordova-plugin-adder@1.1.0
```

或

```bash
npm publish /path/to/your/plugin
```

#### 测试是否发布成功

```bash
npm i cordova-plugin-adder
```

或者

```bash
cordova plugin add cordova-plugin-adder
```

### 发布到私有 NPM

#### 安装 [NRM](https://www.npmjs.com/package/nrm) NPM 源切换工具

nrm can help you easy and fast switch between different npm registries.

Install:

```bash
npm install -g nrm
```

#### 添加私有 NPM 源:

```bash
nrm add <registry> <url>
```

for example:

```bash
nrm add mbc http://xx.xx.xx.153:10011/repository/npm-cordova/
```

查看 NPM 源列表检查是否添加成功:

```bash
nrm ls
```

#### 切换到私有 NPM 源:

```bash
nrm use <registry>
```

#### 添加账号

```bash
npm adduser
npm login
```

#### 上传插件

进入插件根目录(即包含package.json文件的目录)

```bash
npm publish
```

或

```bash
npm publish /path/to/your/plugin
```