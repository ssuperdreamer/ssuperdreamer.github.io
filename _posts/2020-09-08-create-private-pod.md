---
layout: post
title: "Create private Pod"
subtitle: "Cocoapod 私有库搭建"
date: 2020-09-04 16:59:10
author: "Axag"
header-img: "img/post-ios-solution.jpg"
tags: [iOS百宝箱]
---
# 使用Cocoapods创建私有podspec

## 目的以及文章要解决什么问题
私有库是将一个项目中不同部分切割成不同的模块，然后通过工具进行分别管理，这样可以有效的划分人员和代码的职责，方便管理，同时做到代码的模块化，这篇文章主要就是针对如何用cocoapod做私有库搭建，同时不同模块将如何通过cocoapod进行管理和使用进行一个较为详细的梳理

## 总体步骤如下
1. 创建一个私有的`Spec Repo` 仓库
2. 使用命令创建一个私有模块
3. 调整并测试配置好的`podspec`文件是否可用
4. 向私有的`Spec Repo`提交 `podspec`
5. 在具体工程项目中增加刚刚制作好的`Pod`并使用
6. 更新维护`podspec` 
 
## 1.创建一个私有的`Spec Repo` 仓库
首先在自己的代码管理的服务器上创建一个私有仓库，个人习惯用 xxxxSpesc来命名私有仓库这边我就创建了一个TTYXSpecs.git

ps:下面命令行中具体的私有仓库地址必须根据自己的私有仓库地址进行替换哦

```pod repo add TTYXSpecs http://xxx.xx.xx.xxx:3000/yanghongxiang/TTYXSpecs.git ```

这样就将Spec Repo 仓库加入到本地来了
此时如果成功的话进入到~/.cocoapods/repos目录下就可以看到TTYXSpecs这个目录了。至此第一步创建私有Spec Repo完成。

## 2.使用命令创建一个私有模块
私有模块也必须在自己的代码管理服务器上创建一个Git仓库，这边我创建了一个TTYXUtils.git 地址 

`http://xxx.xx.xx.xxx:3000/yanghongxiang/TTYXUtils.git`

然后命令行输入代码

``` pod lib create TTYXUtils ```

这边会有一些问题需要选择

1. What platform do you want to use?? [ iOS / macOS ]
2. What language do you want to use?? [ Swift / ObjC ]
3. Would you like to include a demo application with your library? [ Yes / No ]
4. Which testing frameworks will you use? [ Specta / Kiwi / None ]
5. Would you like to do view based testing? [ Yes / No ]
6. What is your class prefix?

为了更干净这边4和5我选择了 None 和 No
然后当执行成功后命令行窗口会显示

 > Ace! you're ready to go! We will start you off by opening your project in Xcodeopen 'TTYXUtils/Example/TTYXUtils.xcworkspace' 
 
 我们会得到这样一个工程文件


## 3.调整并测试配置好的`podspec`文件是否可用

我们需要调整下TTYXUtils.podspec这个文件(根据自己需求调整，具体参数的含义可以问度娘)
然后运行命令行

```pod lib lint``` 

这是用来验证podspec文件是否正确配置的，当显示 xxx passed validation. 就说明是正确的，然后我们将本地项目和我们刚才创建的git仓库连接起来

``` 
$ git add .
$ git commit -s -m "Initial Commit of Library"
$ git remote add origin http://xxx.xx.xx.xxx:3000/yanghongxiang/TTYXUtils.git        #添加远端仓库
$ git push origin master     #提交到远端仓库
```

因为`podspec`文件中获取Git版本控制的项目还需要`tag`号，所以我们要打上一个tag

```
git tag -m "first release" 0.1.0
git push --tags 
```

这样我们就完成一个私有组件的工程构造 并且发布到代码仓库

## 4. 向私有的`Spec Repo`提交 `podspec`

这个很简单，进入到你私有组件到工程目录下 xxxx.podspec 这一层级

```pod repo push TTYXSpecs TTYXUtils.podspec```

<!-- pod repo push TTYXSpecs TTYXUtils.podspec --allow-warnings --verbose --use-libraries -->

这时候还会做一个验证，如果验证通过则顺利加到私有仓库了，验证不通过则还需要根据错误信息调整
完成之后这个组件库就添加到我们的私有Spec Repo中了，可以进入到~/.cocoapods/repos/TTYXSpecs目录下查看

```
├── LICENSE
├── TTYXSpecs
│   └── 0.1.5
│       └── TTYXSpecs.podspec
└── README.md
```

再去看我们的Spec Repo远端仓库，也有了一次提交，这个podspec也已经被Push上去了。

## 5. 在具体工程项目中增加刚刚制作好的`Pod`并使用

在具体工程项目的Podfile中 添加
```
source 'http://xxx.xx.xx.xxx:3000/yanghongxiang/TTYXSpecs.git'
pod 'TTYXUtils'
```
然后执行`pod install`就可以运行了

## 6. 更新维护`podspec` 
后续如果需要更新代码，则修改完代码后，需要将podspec中的 版本往上提
然后push到远端仓库，打上tag.
然后执行第四步骤即可