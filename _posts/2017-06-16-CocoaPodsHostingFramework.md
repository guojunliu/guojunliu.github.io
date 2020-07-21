---
layout: post
title: <font color=#0099ff>CocoaPods托管Framework</font>
tags: [SDK, Framework]
Author: Steve.liu
---


在发布sdk中，需要在CocoaPods上托管Framework

### 安装CocoaPods

首先安装CocoaPods，没安装的自行安装

### 处理源码
第一步在github上创建源码库（我们的源码是Framework），并把源码push到github上

github地址：

```
https://github.com/guojunliu/AvidlyAdsSDK.git
```

### podspec文件
第二步,在目录下创建`.podspec`文件，此处创建的是`AvidlyAdsSDK.podspec`，

```
$ pod spec create AvidlyAdsSDK
```

用编辑器打开.podspec文件 (我自己用Sublime Text)

文件内容为

```
Pod::Spec.new do |s|
  s.name             = 'AvidlyAdsSDK'
  s.version          = '2.0.20'
  s.summary          = 'Avidly Ad SDK'
  s.description      = <<-DESC
Avidly Ad SDK.
                       DESC

  s.homepage         = 'http://ads-sdk-doc.haloapps.com/docs/show/2'
  s.license          = { :type => 'MIT', :file => 'LICENSE' }
  s.author           = { "steve" => "steve.liu@holaverse.com" }
  s.source           = { :git => 'https://github.com/guojunliu/AvidlyAdsSDK.git', :tag => s.version }

  s.ios.deployment_target = '8.0'

  s.source_files = 'Framework/Appnext/include/*', 'Framework/Chance/include/*', 'Framework/Domob/include/*'
  
  s.resources = "Framework/Chance/resource/*", "Framework/Domob/resource/*", "Framework/Vungle/resource/*", "Framework/AvidlyAdsSDK/resource/*",

  s.public_header_files = 'Framework/Appnext/include/*', 'Framework/Chance/include/*', 'Framework/Domob/include/*'

  s.library = 'sqlite3', 'z'

  s.frameworks = 'QuartzCore', 'MediaPlayer', 'CoreMedia', 'CoreGraphics', 'CFNetwork', 'WebKit', 'WatchConnectivity', 'SystemConfiguration', 'StoreKit', 'Social', 'MessageUI','JavaScriptCore','EventKit','CoreTelephony','AVFoundation','AudioToolbox','AdSupport'

  s.vendored_libraries = "Framework/Appnext/libAppnextLib.a", "Framework/Appnext/libAppnextSDKCore.a", "Framework/Chance/libChanceAd_Video.a", "Framework/Domob/libIndependentVideoSDK.a"

  s.vendored_frameworks = 'Framework/AdColony/AdColony.framework', 'Framework/Mobvista/MVSDK.framework', 'Framework/Mobvista/MVSDKReward.framework', 'Framework/Unity/UnityAds.framework', 'Framework/Vungle/VungleSDK.framework', 'Framework/AvidlyAdsSDK/AvidlyAdsSDK.framework', 'Framework/FBAudienceNetwork/FBAudienceNetwork.framework', 'Framework/GoogleMobileAds/GoogleMobileAds.framework', 'Framework/HolaStatisticalSDK/HolaStatisticalSDK.framework', 'Framework/OneWay/OneWaySDK.framework'
end
```

字段解释

|字段| 解释
-----|------
|name| 名称|
|version| 版本号|
|summary| 摘要|
|description| 描述|
|homepage| 主页|
|license| 许可证，这个必须要有，且按照上述格式，否则会出错|
|author| 作者|
|source| 资源，源码git库|
|ios.deployment_target| ios 编译版本|
|source_files| 源文件|
|resources| 资源包，比如图片，bundle等|
|public_header_files| 共用头文件|
|library| 依赖的系统tbd库，要去掉`lib`前缀,例如`libsqlite3.tbd`要写成`sqlite3`|
|frameworks| 依赖的系统Framework|
|vendored_libraries| 非系统库，如依赖的`.a`第三方静态库。需要按照目录写|
|vendored_frameworks| 非系统库，如依赖的`.framework`第三方静态库，我们此次托管的是framework，所以也写在这里。需要按照目录写|

<br>

编写完之后需要校验下`podspec`文件格式是的正确，Xcode是否能正常编译。使用下面代码

```
$ pod lib lint
```

出现下面代码表示验证通过

```
-> AvidlyAdsSDK (2.0.20)

AvidlyAdsSDK passed validation.
```

接下来连同`podspec文件`一起push到github，由于Cocoapods在自己配置的s.source git中是以版本号检索的，即git的tag，所以我们push之后，别忘记给当次commit打上tag标签，这里打的是`2.0.20`,要和`podspec文件`中一致，才能被检索到


### 推送到官方库
最后是使用`pod trunk`命令，把podspec文件推送到CocoaPod官方库

pod trunk 需要注册 具体做法这里不再赘述 请移步CocoaPod官网

pod trunk 设置完毕后执行命令

```
$ pod trunk push AvidlyAdsSDK.podspec
```

因为我们依赖的SDK比较多，源码比较大，所以整个过程比较耗时。

推送完成之后查询一下

```
pod search AvidlyAdsSDK
```
可以看到已经能查询到我们推送到CocoaPod官方的库了

```
-> AvidlyAdsSDK (2.0.20)
   Avidly Ad SDK
   pod 'AvidlyAdsSDK', '~> 2.0.20'
   - Homepage: http://ads-sdk-doc.haloapps.com/docs/show/2
   - Source:   https://github.com/guojunliu/AvidlyAdsSDK.git
   - Versions: 2.0.20, 2.0.19 [AvidlyAdsSDK repo]
```

### search不到？

上一步中，在推送到官方库之后，有可能在search的时候，search不到，出现下面的提示

```
[!] Unable to find a pod with name, author, summary, or description matching `AvidlyAdsSDK`
```

虽然我们push到官方可了，但是依然search不到

要解决这个问题，首先我们要了解下cocoapods的机制，在安装cocoapods的时候，使用过下面的命令

```
pod setup
```
这行命令其中最重要的作用就是将cocoapods官方repo，clone到我们本地。

我们可以看下我们本地的repo

```
pod repo list
```

```
master
- Type: git (master)
- URL:  https://github.com/CocoaPods/Specs.git
- Path: /Users/steve/.cocoapods/repos/master

1 repo
```

可以看到我们本地目前只有一个repo，就是官方的master repo

这个master repo中包含了所有push到cocoapod的`podspec文件`，可以去本机cocoapods目录下看一下，有成千上万的podspec文件。

解决上述问题的最简单办法就是重新调用下`pod setup`，重新clone下master repo

#### 但是

由于master repo实在太大（大概700M+），我们不希望CP浪费时间在clone repo上，为了提高效率，所以我们要使用我们自己的私有repo

### 私有库

先来说一个概念，什么是repo？它是所有的Pods的一个索引，就是一个容器，所有公开的Pods都在这个里面，它实际是一个Git仓库remote端在GitHub上，但是当你使用了Cocoapods后它会被clone到本地的~/.cocoapods/repos目录下。可以进入到这个目录看到master文件夹就是这个官方的repo了。这个master目录的结构是这个样子的

```
master/xxx库/0.0.1版本/xxx.podspec
            /0.0.2版本/xxx.podspec
master/zzz库/2.0.1版本/zzz.podspec
```

其中`master`是repo名称，下一级目录`xxx库`是你托管的库名称，再下一级`0.0.1版本`是你库的版本，再下一级`xxx.podspec`就是你自己的podspec

所以我们模拟这个目录创建一个就好了

首先创建一个git库，这里是

```
https://github.com/guojunliu/AvidlyAdsSDK-SpecsRepo.git
```

接下来在创建目录及文件

```
AvidlyAdsSDK/2.0.20/AvidlyAdsSDK.podspec
```

如果需要维护多个版本的线上包，那就

```
AvidlyAdsSDK/2.0.19/AvidlyAdsSDK.podspec
            /2.0.20/AvidlyAdsSDK.podspec
```

接下来push到github上

repo创建完了，接下来我们将repo clone到我们本地，先看下我们本地的repo

```
pod repo list
```

```
master
- Type: git (master)
- URL:  https://github.com/CocoaPods/Specs.git
- Path: /Users/steve/.cocoapods/repos/master

1 repo
```

只有master

接下来colen我们自己的repo

```
pod repo add Avidly https://github.com/guojunliu/AvidlyAdsSDK-SpecsRepo.git
```

其中`Avidly`是我们repo clone到本地的名字，自己命名

完成之后再来看下本地的repo

```
Avidly
- Type: git (master)
- URL:  https://github.com/guojunliu/AvidlyAdsSDK-SpecsRepo.git
- Path: /Users/steve/.cocoapods/repos/Avidly

master
- Type: git (master)
- URL:  https://github.com/CocoaPods/Specs.git
- Path: /Users/steve/.cocoapods/repos/master

2 repos
```

已经存在了，可以去~/.cocoapods/repos目录下瞅瞅

接下来再search就可以找到我们托管的Framework了

这样我们就不用让CP再等待setup了

### 测试

Podfile如下

```
platform :ios, '8.0'
use_frameworks!

target ‘testAvidlyPod’ do
  source 'https://github.com/guojunliu/AvidlyAdsSDK-SpecsRepo.git'
  pod 'AvidlyAdsSDK', '~> 2.0.20'
end
```

其中source就是我们自己的私有库

### 可能会遇到的问题

1、swift验证不过

2、xcode编译不过导致验证不过

3、clone出错

4、xcworkspace编译不过