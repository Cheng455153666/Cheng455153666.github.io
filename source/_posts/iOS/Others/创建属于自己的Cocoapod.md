---
title: 创建属于自己的Cocoapod
date: 2016.12.30 11:10:20
categories: iOS
tags: [iOS]
---


能够搜索到这个文章，相信大家对Cocoapods已经有一些基本的了解，关于初始化Cocoapod的问题，网上已经有很多类似的文章了，这里就不多做赘述（传送门:https://cocoapods.org）

说了点废话，那么直接进入主题:

## 首先在github上创建pod库

![NMcJUo](http://ckopenbucket.oss-cn-beijing.aliyuncs.com//1620605986156/NMcJUo.jpg)

 * 库clone到本地

![m2lxP7](http://ckopenbucket.oss-cn-beijing.aliyuncs.com//1620606011593/m2lxP7.jpg)

* 接下来就是创建需要上传的文件了

![apfc4o](http://ckopenbucket.oss-cn-beijing.aliyuncs.com//1620606022258/apfc4o.jpg)

* 将开始创建的Pod文件拖到项目下

![ojrjMN](http://ckopenbucket.oss-cn-beijing.aliyuncs.com//1620606036311/ojrjMN.jpg)

（注意勾选）

* 然后在空文件下创建自己的Pod

![9oqXxL](http://ckopenbucket.oss-cn-beijing.aliyuncs.com//1620606054360/9oqXxL.jpg)

现在库里面也有需要上传的Pod文件了

![AlXfdy](http://ckopenbucket.oss-cn-beijing.aliyuncs.com//1620606062726/AlXfdy.jpg)

* 现在准备工作就结束了

注册trunk

```
格式为：pod trunk register EMAIL [NAME]
例：pod trunk register XXX.com '张三' --verbose
注册成功后会在你的邮箱收到一份确认邮件
可以验证一下自己的trunk
pod trunk me
```

![2l5Q6J](http://ckopenbucket.oss-cn-beijing.aliyuncs.com//1620606089810/2l5Q6J.jpg)

创建podspec文件

```
pod spec create CKPodTest
在精简后，保留下来一些基本的参数：
```

![qKGZ7r](http://ckopenbucket.oss-cn-beijing.aliyuncs.com//1620606134081/qKGZ7r.jpg)

这里要特别注意的是s.source_files = "CKPodTest/*"这个是相对于*. podspec的!（之前是在上面吃过不少亏 - -。）

将代码push到远程仓库并打上tag

```
git add -A
git commit -m"CKPodTest 0.0.1"
git push origin master
打上tag
git tag '0.0.1'
git push --tag
```

验证Pod信息

```
pod lib lint
如果出现错误需要查看错误信息可以使用：
pod lib lint --verbose
```

![JNowYs](http://ckopenbucket.oss-cn-beijing.aliyuncs.com//1620606164985/JNowYs.jpg)

常见的错误是因为DESC文字太短了，和summary必须做修改，改改就好了👀

![FKyEBn](http://ckopenbucket.oss-cn-beijing.aliyuncs.com//1620606176231/FKyEBn.jpg)

验证通过了

最后一步了，开始推Pod

```
pod trunk push CKPodTest.podspec
然后就推送成功了！如果有问题，请检查*.podspec中的信息
```

![QObFFK](http://ckopenbucket.oss-cn-beijing.aliyuncs.com//1620606199960/QObFFK.jpg)

到了这一步，个人Pod的上传就已经是结束了，但是在你search自己的pod时，可能会出现以下问题，可以按下面的做法，处理一下：

![tEVwbc](http://ckopenbucket.oss-cn-beijing.aliyuncs.com//1620606208638/tEVwbc.jpg)

解决办法：

```
1. 可以先更新cocoapods
sudo gem install -n /usr/local/bin cocoapods --pre
2. 如果还是无法搜索到，可以更新你的search cache
rm ~/Library/Caches/CocoaPods/search_index.json
3. 如果依然无法搜索到，Google可以帮助你😏😏😏，反正我是找到了
```

![cdPjbV](http://ckopenbucket.oss-cn-beijing.aliyuncs.com//1620606247157/cdPjbV.jpg)

![o3soip](http://ckopenbucket.oss-cn-beijing.aliyuncs.com//1620606252689/o3soip.jpg)

## 如果是需要提交到私有仓库

```
pod repo push podName podName.podspec

项目中使用这个私有pod的话：
pod 'podName' , :git => "https://XXXXXX.git", :tag => 'your private pod version'
```

遗留下来的问题：
如何用在Pod中包含静态库文件（*.a，*.framework）
毕竟，有时候代码封装的时候，部分核心内容是不希望暴露在外的。

> 参考：
[使用CocoaPods开发并打包静态库](http://www.cnblogs.com/brycezhang/p/4117180.html)
[Avoiding dependency collisions in iOS static library managed by CocoaPods](http://blog.sigmapoint.pl/avoiding-dependency-collisions-in-ios-static-library-managed-by-cocoapods/)