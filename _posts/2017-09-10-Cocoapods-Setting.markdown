---
layout: post
title: CocoaPods 私有库配置
date: 2017-09-10 15:32:24.000000000 +09:00
---

**前言**

现在的项目完全是用组件化的思路开发的，也是借此机会亲自实现了一把。
项目里的每个业务模块都是使用`cocoapods` 私有库开发，依赖下层开发或者二次封装的私有库。这里先介绍一下我的私有库使用案例。

**私有库创建**

我项目里的功能模块都是用的官方的私有库模版。跑如下命令

```
pod lib create __ProjectName__
```

然后根据一系列提示定制你的私有库配置.

编辑 `.podspec` 文件

* `s.name` 就是你私有库的名称
* `s.version` 你的私有库版本
* `s.summary` 简单的介绍
* `s.description` 描述，要比上面一项文字长一点哦
* `s.source` 代码, git 地址，以及版本(或指定分支)
* `s.ios.deployment_target` 支持iOS 版本
* `s.source_files` 私有库的源文件 `'Pods/Classes/**/*'` , `**` 表示匹配一层或多层目录，`*`匹配任何类型文件.
* `s.public_header_files` 暴露的头文件
* `s.resource_bundles` 资源文件存放路径
* `s.frameworks` 私有库所需要引用的系统库
* `s.vendored_libraries` 引用第三方静态库 , 自己引入的`.a`文件
* `s.dependency` 依赖的其他第三方`pod`库
* `s.pod_target_xcconfig` 指定`target` 添加配置

下面给出我的一个 `podspec` 文件样例

```
Pod::Spec.new do |s|
  s.name             = 'musicXML'
  s.version          = '0.1.6'
  s.summary          = 'musicxml for piano to play'
  s.description      = 'draw music sheet and play sound according to musicXML'
  s.homepage         = 'https://gitlab.oneitfarm.com/SpecPods/MusicXML'
  s.license          = { :type => 'MIT', :file => 'LICENSE' }
  s.author           = { 'yourName' => 'your@email.com' }
  s.source           = { :git => 'your git address', :tag => s.version.to_s } #指定version
  s.ios.deployment_target = '8.0'
  s.source_files = ['Pod/Classes/**/*','Pod/musicFramework/**/*.{cpp}']
  s.resource_bundles = {
    'musicXML' => ['Pod/Assets/*.xib','Pod/Assets/*.png',
                   'Pod/Assets/*.dls','Pod/Assets/*.xcassets'
                  ]
  }
  s.xcconfig = { 'HEADER_SEARCH_PATHS' => 'Pod/MusicFramework/**/*.h'}
  s.public_header_files = 'Pod/Classes/**/*.h'
#  s.vendored_frameworks = 'Pod/MusicFramwork.framework'
  s.frameworks = 'UIKit', 'MapKit','AVFoundation','AudioToolbox','CoreAudio','CoreBluetooth'
  s.vendored_libraries = 'Pod/libraries/*.a'
  s.dependency 'MBProgressHUD'
  s.dependency 'CIRouter','0.0.3'
  s.dependency 'CIProgressHUD','0.2.0'
  s.dependency 'Moya','8.0.5'
  s.dependency 'Moya/RxSwift','8.0.5'
  s.dependency 'RxSwift','3.6.1'
  s.dependency 'SwiftyJSON','3.1.4'
  s.dependency 'CIUtilS'
  s.pod_target_xcconfig = { 'SWIFT_VERSION' => '3.2' }
end
```

**添加该项目到你的仓库**

`pod repo add __ProjectName__  Project_Git_Address`

**验证私有库**

`pod lib lint ProjectName.podspec`

跑上面的命令时可能会遇到各种报错

* `warning` 导致失败，添加 `--allow-warnings` 
* `SomePod not Found` 依赖的其他私有库未找到,添加 `--sources="SomePod's Git Address,https://github.com/CocoaPods/Specs.git"`
* `--verbose` 添加这个选项会有更多的打印，主要是为了看错误


**推送**

当本地`lint`成功后，就可以将该版本推到的你仓库中

`pod repo push __PodName__  --allow-warnings --allow-warnings`

注意，`lint` 命令中添加的参数，`push` 时也要添加
