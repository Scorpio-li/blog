---
title: npm包发布及维护
comment: true
date: 2023-04-18 16:44:48
tags:
description:
categories:
keywords:
---
# npm包发布及维护

## 前言
npm（node package manager）作为 Node.js 的包管理工具，让世界各地的 JavaScript 开发者方便复用、分享代码以及造轮子；本文将介绍如何发布一个 npm 包，以及使用工具来自动化管理发布 npm 包；本文总览如下：

1. 准备 npm 账号；

2. 初始化 & 配置 package.json 文件；

3. 确定发布的包版本；

4. 发布 npm 包；

5. 使用 cli 工具 release-it 来自动管理版本、发布包等；

6. 学习 npm init release-it 原理。

## 准备

本地需要安装 Node.js 及 npm CLI，npm 将随 Node.js 一起安装，建议使用 Node 版本管理工具来安装 Node，例如 nvm 、n。


### 注册 npm 账号

第一步先到 npm 注册个账号 www.npmjs.com/signup，跟着注册步骤操作就行。

## 创建 & 配置 package.json

要发布一个 npm 包，需要创建一个 package.json 文件，用来描述包信息及依赖包。使用 npm init -y 命令创建一个默认的 package.json 文件

要发布一个npm 包，name 和 version 字段是必填的；
包名（name 字段）命名规则：

- 包名长度不能超过 214 个字符（命名空间也算在里面）；

- 包名所有字符必须小写；

- 包名可以由连字符 - 组成；

- 包名不能包含空格，不能以 . 或者 _ 开头，不能包含 ~)('!* 中的任意一个字符；

- 包名不能包含任何非 url 安全字符（因为包名将作为 url 的一部分）；

- 包名不能与 Node.js / io.js 的核心模块、保留字或黑名单相同，例如 http

## 确定版本号

### 版本号格式

格式：MAJOR.MINOR.PATCH ，值非负整数，且禁止在数字前面补 0

- MAJOR：主版本号

- MINOR：次版本号

- PATCH:：修订号

### 版本号递增逻辑

- 当有破坏性不兼容的 API 变更时，升级主版本号

- 当新增一些功能特性时，升级次版本号

- 当做一些 bug 修复时，升级修订号

可以使用 npm version 命令来修改版本号

```sh
npm version [<newversion> | major | minor | patch | premajor | preminor | prepatch | prerelease | from-git]
```
eg：

```sh
# 0.0.0 => 0.0.1
npm version patch
# 0.0.0 => 0.1.0
npm version minor
# 0.0.0 => 1.0.0
npm version major
# 0.0.0 => 0.0.1-0
# === 先行版本 ===
npm version prepatch
# 0.0.0 => 0.0.1-alpha.0
npm version prepatch --preid=alpha
# 0.0.1-alpha.0 => 0.0.1-alpha.1
npm version prerelease
```

命令行选项：

- --preid 指定先行版本的标识符，例如 1.2.3-rc.4 中的 rc

- -m 或 --message 可以指定 commit 信息，例如 npm version patch -m "Upgrade to %s for reasons” ;

- -no-git-tag-version 取消创建 git tag 和 commit 信息

其他说明：

- 执行 npm version 命令时，会修改 package.json 、package-lock.json 的 version 字段为对应版本；若当前使用 git 来管理文件，还会创建一条 commit 信息和创建一个 tag，可通过指定 -no-git-tag-version 取消生成 commit 和 tag。

- 执行 npm version 前，git 工作区和暂存区确保没有文件，否则会执行失败。

## 发布 npm 包

### npm 登录 & 确定源地址

因为我们要发布到官方源上面，所以要确保源地址为官方地址 http://registry.npmjs.org 或 https://registry.npmjs.com，可以通过 npm config get registry 命令查看当前 registry 源。

- 登录: npm login

如需修改 registry 可通过以下四种方式：

1. 在全局配置 .npmrc ，可通过命令 npm config set registry= http://registry.npmjs.org/ ；

2. 在当前项目配置，在当前项目中添加配置文件 .npmrc ；
```sh
// .npmrc
registry = http://registry.npmjs.org/
```

3. 发布包时指定 --registry 选项，npm publish —registry=http://registry.npmjs.org/ ；

4. 在当前项目的 package.json 中通过 publishConfig 字段指定。

```json
// package.json
{
  ”publicConfig“: {
    "registry": "http://registry.npmjs.org/"
  } 
}
```

### 发布版本

通过命令 npm publish 发布包

```
npm publish --access public
```

本文示例使用的命名空间为 derick ，在发布时需要指定声明为公有包，因为发布带有命名空间的包 npm 会默认为是要发布私有包，发布私有包需要另外付费的哦！
这里我们通过命令行选项 --access public 声明为公有包，也可通过 package.json 中的 publicConfig 字段配置声明。

```
"scripts": {
  // `npm install` 和 `npm publish` 前执行
  "prepare": "npm run build",

  // 发布前打包命令
  "build": "webpack --config webpack.prod.config.js",

  // 升级预发版本号并发布预发包
  "publish:alpha": "npm version prerelease --preid=alpha && npm publish --tag alpha",

  // 升级主版本并发布正式包
  "publish:mmajor": "npm version mmajor && npm publish",

  // 升级次版本并发布正式包
  "publish:minor": "npm version minor && npm publish",

  // 升级修订号并发布正式包
  "publish:patch": "npm version patch && npm publish"
},
```

## 撤销发布

如果发现已发布的包有 bug，理论上你可以使用 npm unpublish 命令撤销已发布的包，但是通常不建议这么做。通常的做法是先改变 latest 指向进行回退，然后从 master 检出新的补丁分支修复 bug 后发布一个修复版本（升级修订号）。

```sh
# 将指定版本标记为稳定的最新版本
npm dist-tag add <pkg>@<version> latest
```

如果包有重大 bug 且被很多人使用，则需要使用 npm deprecate 对所有尝试安装该软件包的人发出弃用警告，此命令可以设置特定版本或某个版本范围内的包需要发出警告信息。

```sh
# npm deprecate <pkg>[@<version range>] <message>
npm deprecate my-thing@"< 0.2.3" "critical bug fixed in v0.2.3"
npm deprecate my-thing@1.x "1.x is no longer supported"

# 取消警告将 message 选项设置为 ""
npm deprecate my-thing@"< 0.2.3" ""
```

## 使用 cli 工具 release-it

release-it 工具可以用来升级包版本、生成 changgelog、Git commit \ tag \ push、发布包等。

在项目中运行命令 npm init release-it

选择完之后将会自动安装 release-it 包、在 package.json 中加入脚本命令 "release": "release-it”、及生成一个 .release-it.json 初始配置文件。

在 .release-it 添加以下配置（查看更多配置见 release-it - Configuration），并安装生成 changelog 的插件npm i @release-it/conventional-changelog -D

```json
{
  "github": {
    "release": true
  },
  "git": {
    "commitMessage": "Release: v${version}"
  },
  "plugins": {
    "@release-it/conventional-changelog": {
      "preset": "angular",
      "infile": "CHANGELOG.md"
    }
  }
}
```

因我们发布的包带有命令空间，所以还需要在 package.json 中添加如下配置，声明一下为公有包

```json
{
    "publishConfig": {
        "access": "public"
    }
}
```

然后运行命令 npm run release ，跟着步骤操作选择，之后就会自动生成 changelog、修改包版本、git commit 信息、打 tag、创建 github release、发布包；如下图，至此就完成了 npm 包的自动发布。