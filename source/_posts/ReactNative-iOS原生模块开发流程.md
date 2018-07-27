---
title: ReactNative iOS原生模块开发流程
date: 2018-07-27 14:51:36
tags:
---
# ReactNative iOS原生模块开发流程

最近要开发 RN 的组件，查看了官方原生组件的开发说明文档，发现侧重点是在编写原生代码这部分，缺少环境的搭建流程，所以查了一些资料，整理一个环境搭建的流程。

整个流程基本上分以下几步：

1. 创建 RN 项目（用来调试模块）；
2. 创建原生模块项目；
3. RN 项目中引入原生模块项目；
4. 开发原生模块（参照 RN 官方文档）；
5. 发布上线；

## 创建 RN 工程

首先通过 CLI 创建一个 RN 项目，在开发原生模块的过程中会用这个工程来进行调试。

```sh
react-native init ExampleModule
```

## 创建原生模块项目

通过网上的资料，我发现创建原生模块项目有2种简便方法，通过2种方式创建项目，可以省去手动配置项目的工作，极大的减少可能出错的环节。

### 通过 RN 的 CLI 创建

```sh
#react-native new-library --name <yourLibraryName>
```

命令执行后，可以在`./Libraries`下面找到创建的原生项目，**这种方式只支持 iOS 项目**。

### 通过 react-native-create-library 来创建

1. 安装`react-native-create-library`；

    ```
    npm install -g react-native-create-library
    ```

2. 进入你要存放原生模块的目录（建议是 `Libraries`），执行下面的命令；

    ```sh
    react-native-create-library <yourLibraryName>
    ```

这种方式创建的项目中包含`iOS`、`Android`、`Windows`三个文件夹，对应三个平台。如果是开发跨平台的模块，建议使用这种方式。

## RN 项目中引入原生模块项目

这里只是开发 `iOS` 模块，所以直接使用方式一创建原生模块项目。`ExampleModule` 是 RN 项目，`MyFirstRNModule` 是原生模块。现在文件结构如下图：

![](http://oe0m44pef.bkt.clouddn.com/2018-07-27-15245383675028.jpg)

用 `xcode` 打开 `ExampleModule/ios` 下的 iOS 工程，添加 `MyFirstRNModule` 到`Libraries` 文件夹下，如下图：

![](http://oe0m44pef.bkt.clouddn.com/2018-07-27-15245390726193.jpg)

![](http://oe0m44pef.bkt.clouddn.com/2018-07-27-15245391049749.jpg)

链接 `MyFristRNModule` 到 `ExampleModule` 工程中：

![](http://oe0m44pef.bkt.clouddn.com/2018-07-27-15245394525224.jpg)

![](http://oe0m44pef.bkt.clouddn.com/2018-07-27-15245394784670.jpg)

至此，RN 项目就引入了原生模块项目。

## 编写原生模块代码

这一部分按照 [RN 官方教程](http://facebook.github.io/react-native/docs/native-modules-ios.html)来编写原生模块代码。

在这里只做个简单的测试，看是否可以调用到这个模块。

1. 打开`MyFirstRNModule.m`文件，添加代码，然后使用模拟器运行；

    ![](http://oe0m44pef.bkt.clouddn.com/2018-07-27-15245402331526.jpg)

2. 修改 RN 项目中的`App.js`；

    ![](http://oe0m44pef.bkt.clouddn.com/2018-07-27-15245408646258.jpg)

3. 选中模拟器，`cmd+R` 刷新界面；

    ![](http://oe0m44pef.bkt.clouddn.com/2018-07-27-15245409912857.jpg)

4. 点击 `test` 按键，注意观察 xcode 的调试窗口，这时会打印一条 `test` 信息出来，说明模块运行正确；

    ![](http://oe0m44pef.bkt.clouddn.com/2018-07-27-15245415527519.jpg)


## 发布上线

编写好原生模块之后，就可以发布到 npm，这样其他人就可以下载使用。

1. 创建模块的 GitHub 仓库
2. 修改模块的入口文件

    目录下已经有了 `MyFirstRNModule.ios.js` 可以作为入口文件，但是发布到 npm 上的包都是以 `index.js` 作为入口文件的，因此创建 `index.js`，将原生模块导出。

    ```js
    import React, { NativeModules } from 'react-native';
    module.exports = NativeModules.MyFirstRNModule;
    ```

3. 发布到 npm

    发布之前，需要先编辑 `package.json` 来配置原生模块的信息，如下：

    ```json
    {
      "name": "my-first-rn-module",
      "version": "0.0.1",
      "main": "index.js",
      "keywords": "react-native",
      "author": "lg",
      "license": "MIT",
      "repository": {
        "type": "git",
        "url": "https://github.com/lg8294/MyFirstRNModule.git"
      },
      "homepage": "https://github.com/lg8294/MyFirstRNModule",
      "bugs": {
        "url": "https://github.com/lg8294/MyFirstRNModule/issues"
      }
    }
    ```

    注：其中的 `name` 要采用 `kebab-case` 格式，否则发布的时候会报错。
    如果编写的原生模块依赖于其他的原生模块，需要在 package.json 添加依赖关系，这里由于没有相关依赖，所以不需要添加：

    ```json
    "dependencies": {
    }
    ```

    如果没有npm的账号，先注册一个账号，这个账号会被添加到npm本地的配置中，用来发布module用。

    ```sh
    npm adduser
    Username: your name
    Password: your password
    Email: yourmail@gmail.com
    ```

    发布：
    
    ```sh
    npm publish
    ```

    有时候，有些文件没必要发布，例如 MyFirstRNModule.ios.js 和 MyFirstRNModule.android.js 文件，可以通过 .npmignore 忽略它。例如我这里 .npmignore 文件内容如下：
    
    ```
    .git
    .gitignore
    MyFirstRNModule.ios.js
    MyFirstRNModule.android.js
    ```

4. 添加 README

    README 文件是非常重要的，如果没有 README 文件，别人看到这个组件，根本就不知道它是用来做什么的。所以，很有必要添加一个 README 文件，这个文件需要告诉别人这个原生组件是干什么的、如何安装、API、使用手册等等。

## 参考文档

[React Native 原生模块库打包指南](https://chasecs.github.io/2017/08/09/react_native_packing_native_module_for_android_n_ios.html)
[ReactNative之原生模块开发并发布——iOS篇](http://www.liuchungui.com/blog/2016/05/02/reactnativezhi-yuan-sheng-mo-kuai-kai-fa-bing-fa-bu-iospian/)
[如何创建React Native iOS原生模块](http://www.devio.org/2017/01/22/React-Native-iOS%E5%8E%9F%E7%94%9F%E6%A8%A1%E5%9D%97%E5%BC%80%E5%8F%91%E5%AE%9E%E6%88%98-%E6%95%99%E7%A8%8B-%E5%BF%83%E5%BE%97/)
