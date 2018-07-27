---
title: 使用Jenkins构建iOS应用并上传蒲公英
date: 2018-07-27 14:36:45
tags:
---

# 使用 Jenkins 构建 iOS 应用并上传蒲公英

## 安装

在 Mac 上推荐使用 `brew` 来安装 `jenkins`:


```sh
$ brew install jenkins
```

可以选择 `lts` 版本：

```sh
$ brew install jenkins-lts
```

## 第一次启动

通过 `brew` 安装成功后，在系统环境变量中会有 `jenkins`（或 `jenkins-lts`），执行命令就可以启动了：

```sh
$ jenkins
```

执行命令之后，不要关闭命令行窗口，否则这个服务就被关闭了。在命令行窗口中会有一条重要的信息，一条字符串信息，下面的步骤会用到。

通过浏览器访问 `http://localhost:8080`，会出现如下界面：
![](http://oe0m44pef.bkt.clouddn.com/2018-07-27-15326539274571.png)

这个界面要求输入管理员密码，这个密码就是之前命令行中的那个字符串，或者安装界面红色信息提示的，打开对应的文件，拷贝里面的字符串，填入这里的输入框中，点击 `Continue`。

然后会显示自定义 Jenkins 的界面，其实就是安装 Plugins，Jenkins 的所有功能基本都是通过 Plugin 来实现的，所以这里现在第一项，`Install suggested plugins`：

![](http://oe0m44pef.bkt.clouddn.com/2018-07-27-15326546010952.jpg)
![](http://oe0m44pef.bkt.clouddn.com/2018-07-27-15326546219760.jpg)
![](http://oe0m44pef.bkt.clouddn.com/2018-07-27-15326546264089.jpg)
![](http://oe0m44pef.bkt.clouddn.com/2018-07-27-15326546499338.jpg)
![](http://oe0m44pef.bkt.clouddn.com/2018-07-27-15326546826845.jpg)

到这里，输入用户名、密码、邮箱，创建第一个 Admin 用户。

## 配置 iOS 构建项目

构建 iOS 项目的方式有多种，下面是最长用的2种：

1. 通过 Xcode Plugin;
2. 通过 shell 脚本执行 fastlane；

这里只说一下第一种方式。

### 安装 `Xcode integration` 插件

首先选择”系统管理“->”插件管理“->”可选插件“中搜索 `Xcode integration` 安装。
安装完成之后，我们需要先去”系统管理“->”系统设置“里面配置一下 `Xcode Builder`：

![QQ20180727-093711@2x](http://oe0m44pef.bkt.clouddn.com/2018-07-27-QQ20180727-093711@2x.png)

在 `Apple Development Teams` 中配置你的开发者账号信息。你可以登录 [Apple 开发者中心](https://developer.apple.com/account)，然后选择 `Membership`，获取 `Team Name` 和 `Development Team ID`。

在 `Keychains` 中配置你的钥匙串，通常这里配置成“登录”钥匙串：
* `Keychain path` 通常是 `$(HOME)/Library/Keychains/login.keychains`
* `Keychain password` 通常是你的登录密码
* `Keychain Name` 你可已自定义

**注意：这里配置的开发者账号的 development.cer 和 distribution.cer 要安装到安装 Jenkins 这台机器的 keychains 中，病区这个 keychinas 就是刚刚配置的”登录“钥匙串，这个要对应起来，否则在构建的时候会因为在钥匙串中找不到证书导致构建失败**

### 安装 `Upload to pgyer` 插件

在”系统管理“->”插件管理“->”可选插件“中搜索 `Upload to pgyer` 安装；

### 创建 iOS 构建项目

点击”新建任务“，输入一个任务名称，选择”构建一个自由风格的软件项目"：
![](http://oe0m44pef.bkt.clouddn.com/2018-07-27-15326566521137.jpg)
![](http://oe0m44pef.bkt.clouddn.com/2018-07-27-15326566641292.jpg)

点击 `OK`，进入项目的配置界面：
![](http://oe0m44pef.bkt.clouddn.com/2018-07-27-15326608381287.jpg)

所有的构建项目基本上都是由这几项构成的：

- General（通用信息）
- 源码管理
- 构建触发器
- 构建环境
- 构建
- 构建后操作

在 `General` 中可以配置项目的基本信息，通常会配置”丢弃旧的构建“，如下图：
![](http://oe0m44pef.bkt.clouddn.com/2018-07-27-15326613438610.jpg)


在”源码管理“中可以配置源码地址，通常使用 Git 来管理。

![](http://oe0m44pef.bkt.clouddn.com/2018-07-27-15326615267864.jpg)

- `Repository URL`：源码仓库地址
- `Credentials`：登录仓库的用户名和密码或者 SSH private key
- `Branches to build`：用来构建的分支

在”构建触发器“中可以配置如何触发这个构建过程。
![](http://oe0m44pef.bkt.clouddn.com/2018-07-27-15326617635924.jpg)

通常选择”轮询 SCM“，如上图，代表每5分钟去检查一次仓库是否有更新，如果有更新，触发构建过程。

在”构建环境“中可以配置构建工程的环境。
![](http://oe0m44pef.bkt.clouddn.com/2018-07-27-15326620001847.jpg)

在”构建“中配置具体的构建过程。
点击”添加构建步骤“->”Xcode“，添加 `Xcode` 构建过程。
![](http://oe0m44pef.bkt.clouddn.com/2018-07-27-15326621055206.jpg)
![](http://oe0m44pef.bkt.clouddn.com/2018-07-27-15326621669132.jpg)

在 `General build settings` 中配置具体的构建参数：

![](http://oe0m44pef.bkt.clouddn.com/2018-07-27-15326624614150.jpg)
![](http://oe0m44pef.bkt.clouddn.com/2018-07-27-15326626076461.jpg)


- `Development Team`：选择配置好的 Apple 开发者账号，如果留空，代表使用 iOS 工程中的配置
- `Configuration`：Release 或者 Debug
- `Scode Schema File`：用来打包 IPA 的 Schema

在 `Code signing & OS X keychain options` 中配签名方式：

![](http://oe0m44pef.bkt.clouddn.com/2018-07-27-15326626704555.jpg)

我这里配置的是自动签名，选择对应的 `keychain`，这个 `keychain` 是在之前”系统管理“中添加的；

`Advanced Xcode build options` 和 `Versioning` 没有特殊需求，可以不管；


在”构建后操作“中配置上传 IPA 到蒲公英的过程。

![](http://oe0m44pef.bkt.clouddn.com/2018-07-27-15326632311292.jpg)

点击”增加构建后操作步骤“->”Upload to pgyer with apiV2“：

![](http://oe0m44pef.bkt.clouddn.com/2018-07-27-15326632178098.jpg)
![](http://oe0m44pef.bkt.clouddn.com/2018-07-27-15326633020725.jpg)

输入 `pyger api_key` 和 `file wildcard`。

**注意：`file wildcard`（文件通配符）一定要能够匹配到构建出来的 IPA ，否则上传会失败。**

最有一定要点击”保存“，完成项目的配置。

## 构建

在主面板上可以看到我们刚刚创建的项目：

![](http://oe0m44pef.bkt.clouddn.com/2018-07-27-15326636650942.jpg)

点击最右侧的”构建按钮“，开始构建，这样可以立即检验我们之前的配置是否可用；

![](http://oe0m44pef.bkt.clouddn.com/2018-07-27-15326637535445.jpg)
![](http://oe0m44pef.bkt.clouddn.com/2018-07-27-15326638019874.jpg)


如果构建失败了，可以去查看Console Output可以查看log日志。

![](http://oe0m44pef.bkt.clouddn.com/2018-07-27-15326691235578.png)


## 配置 Jenkins 作为后台进程

执行命令：

```sh
$ brew services start jenkins
```

这样 Jenkins 就作为后台进程运行了，但是这样会有个问题，就是局域网内其他机器无法通过 ip 地址来访问，这时因为 jenkins 进程默认监听 ListenAddress 是 127.0.0.1。知道了原因，那么就可以通过修改 ListenAddress 来修复这个问题；

打开 Jenkins 安装的文件夹 `/usr/local/Cellar/jenkins/`，打开对应版本下面的 `homebrew.mxcl.jenkins.plist` 文件，修改 `--httpListenAddress=0.0.0.0`，如下图：

![](http://oe0m44pef.bkt.clouddn.com/2018-07-27-15326698393440.jpg)

然后执行命令：

```sh
$ brew services restart jenkins
```

这样，局域网内的其他机器就可以通过 ip 地址来访问 Jenkins 服务了。

以上就是我关于Jenkins持续集成的一次实践经验。