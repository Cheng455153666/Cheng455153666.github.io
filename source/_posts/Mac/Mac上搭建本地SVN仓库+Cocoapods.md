# ---
title: Mac上搭建本地SVN仓库+Cocoapods
date: 2018.02.02 12:54:25
categories: Mac
tags: [Mac]
---

Mac上本身已经自带svn，可以通过一下命令查看一下:

```shell
svnserve --version
```

看到一些输出之后，接下来正式开始创建本地的SVN:

## 创建代码仓库

```shell
sudo mkdir -p ~/Documents/MySVNServer
```

## 初始化代码仓库

```shell
sudo svnadmin create ~/Documents/MySVNServer
```

可以在`~/Documents/MySVNServer`下看到我们创建的SVN服务：

![z3cO8s](http://ckopenbucket.oss-cn-beijing.aliyuncs.com//1620608469461/z3cO8s.jpg)

## 配置SVN权限

接下来我们对`conf`下的文件做一些修改，删除前面的注释，不要留空格~

### svnserve.conf 配置用户权限

![bzuxpr](http://ckopenbucket.oss-cn-beijing.aliyuncs.com//1620608489632/bzuxpr.jpg)

anon-access = read代表匿名访问的时候是只读的，若改为anon-access = none代表禁止匿名访问，需要帐号密码才能访问。

### passwd 配置账号信息

在`[users]`下添加用户

![8IGbL7](http://ckopenbucket.oss-cn-beijing.aliyuncs.com//1620608508949/8IGbL7.jpg)

用户名 = 密码

### authz 配置权限

![sz95Tw](http://ckopenbucket.oss-cn-beijing.aliyuncs.com//1620608526431/sz95Tw.jpg)

配置名为`iosdev`的用户组，组下用户为`clq`，如果多个用户用,分割

在最下面添加`[/]`,表示授权目录路径访问权限，`@iosdev = rw`表示给`iosdev`组读写权限，`r`读，`w`写，`rw`读写。

如果只允许用户访问项目下Demo文件目录，则:

![RVIfK4](http://ckopenbucket.oss-cn-beijing.aliyuncs.com//1620608567625/RVIfK4.jpg)

@xxxx 表示授权给xxxx组

不使用@则表示授权给某用户

以上就是配置，接下来启动我们配置的svn服务。

```shell
svnserve -d -r ~/Documents/MySVNServer
```

使用上面的命令，会直接启动配置好的`MySVNServer SVN`服务器。默认使用`80`端口。但是我们很多时候并不想占用80端口。可以使用：

```shell
svnserve -d -r ~/Documents/MySVNServer --listen-port 8080
```

现在我们的SVN服务器就会在8080端口上启动了。

如果要关闭SVN，可以通过`Activity Monitor`搜索`svn`来结束进程。

### 使用Cornerstone连接SVN

![noXuSP](http://ckopenbucket.oss-cn-beijing.aliyuncs.com//1620608640213/noXuSP.jpg)

配置好信息之后，我们就可以正式开发了。接下来创建好我们的项目，尝试提一次提交。

![ySb641](http://ckopenbucket.oss-cn-beijing.aliyuncs.com//1620608650960/ySb641.jpg)

接下来你可以会看到授权失败的情况

![zBb2RT](http://ckopenbucket.oss-cn-beijing.aliyuncs.com//1620608657693/zBb2RT.jpg)

```shell
sudo chmod -R a+w ~/Documents/MySVNServer/
```

然后输入密码后，就可以正常提交了！

## Cocoapods+svn搭建私有仓库

SVN的准备工作都结束了，就可以开始在SVN上部署Cocoapods了。

### 创建SVN目录

这点很重要，Cocoapods在和SVN结合的时候，对项目结构有很严格的要求。

![qBmOCe](http://ckopenbucket.oss-cn-beijing.aliyuncs.com//1620608928956/qBmOCe.jpg)

### 创建项目

使用Xcode创建名为ModuleA的项目，项目下创建`Classes`文件夹，我们开发的模块中代码就存放在这里。

![riGj7K](http://ckopenbucket.oss-cn-beijing.aliyuncs.com//1620608946368/riGj7K.jpg)

同时使用pod spec create 文件名创建私有库配置文件`ModuleA.podspec`, 此文件要与模块名相同。

然后将项目下的文件，拷贝到`trunk`目录下

![6DG3tw](http://ckopenbucket.oss-cn-beijing.aliyuncs.com//1620608967308/6DG3tw.jpg)

接下来看看`ModuleA.podspec`的内容：

```yaml
Pod::Spec.new do |s|

# 模块名
s.name         = "ModuleA"
# 版本号。需要注意的是：当仓库代码 push 到远程仓库的时候，需要打上 tag。tag 和 版本号必须一致！！！
s.version      = "0.0.1"
# 简短描述
s.summary      = "This is ModuleA for myProj"
# 模块主页 可能不存在，但是一定要写
s.homepage     = "http://XXXX/ModuleA"
# license 类型
s.license      = { :type => "MIT", :file => "FILE_LICENSE" }

# 创建者信息
s.author             = { "姓名" => "邮箱地址" }
# 平台信息，后面的数字指的是最低的系统要求。
s.platform     = :ios, "9.0"

#  推荐使用下面这几个，针对指定平台最系统配置要求
# s.ios.deployment_target = "9.0"
# s.osx.deployment_target = "11.0"
# s.watchos.deployment_target = "2.0"
# s.tvos.deployment_target = "9.0"

# 远程仓库路径
s.source       = { :git => "http://EXAMPLE/ModuleA.git", :tag => "#{s.version}" }

# 需要暴露给别人的代码文件
s.source_files  = "Classes", "Classes/**/*.{h,m}"

# 需要暴露给别人的资源文件
# s.resources = "Resources/*.png"

# 需要添加的系统 framework
# s.frameworks = "AFramework", "BFramework"

# 需要添加的系统 .tbd 库
# s.libraries = "iconv", "xml2"

# 是否是 ARC 环境
s.requires_arc = true
# xcconfig 路径配置
# s.xcconfig = { "HEADER_SEARCH_PATHS" => "$(SDKROOT)/usr/include/libxml2" }
# 需要依赖的三方库
# s.dependency "JSONKit", "~> 1.4"
# s.dependency "AFNetWorking"

end
```

这里我们只做最基础配置

```yaml
Pod::Spec.new do |s|

  s.name         = "ModuleA"
  s.version      = "0.0.1"
  s.summary      = "ModuleA for MyProject"
  s.description  = <<-DESC
                ModuleA for MyProjectModuleA for MyProjectModuleA for MyProjectModuleA for MyProjectModuleA for MyProject
                   DESC
  s.homepage     = "http://www.baidu.com"
  s.license      = { :type => 'MIT', :file => 'LICENSE' }
  s.author             = { "程立卿" => "xxxxx@xx.com" }
  s.source       = { :svn => 'svn://10.0.3.112:8080/app/Common/ios/ModuleA', :tag => "#{s.version}" }

  s.source_files  = "ModuleA/Classes/**/*"

end
```

我们可以验证一下配置信息是否合格，使用`pod lib lint`

![rBYyrb](http://ckopenbucket.oss-cn-beijing.aliyuncs.com//1620609017556/rBYyrb.jpg)

看到这个，表示已经验证通过了。接下来需要将我们的模块打包推到tags目录下。

这里我使用的是`Cornerstone`。

![MejSHn](http://ckopenbucket.oss-cn-beijing.aliyuncs.com//1620609032119/MejSHn.jpg)

更新一下本地的代码，会发现我们的tags目录下会创建以下的文件

![F8ve6A](http://ckopenbucket.oss-cn-beijing.aliyuncs.com//1620609040588/F8ve6A.jpg)

这里就是打包好的我们对应的ModuleA模块了！

到这里，我们就可以使用我们的私有仓库了！

创建项目过程就不多说了，我们之间看`pod init`后的`Podfile`文件

```
target 'RootProj' do
  pod 'ModuleA',:svn =>'svn://10.0.3.112:8080/ModuleA',:tag =>'0.0.1'
end
```

填入我们的模块名和对应的svn地址还有版本即可。

这时候我们高高兴兴的`pod update`，可能会出现无法导入的情况。这时候八成是需要权限的问题，使用

```shell
svn checkout https://10.0.3.112:8080/ModuleA
```

然后根据提示，输入你的svn账号密码，然后再重新update即可。