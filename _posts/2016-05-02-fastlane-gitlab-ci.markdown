---
layout: post
title:  iOS 持续集成之 Gitlab-Ci + FastLane
date: 2016-05-02 15:59:28.000000000 +09:00
---


## Gitlab-Ci

### 配置Xcode

Xcode唯一需要配置的就是要将你运行的`scheme`设置成`Shared`。

* 打开Xcode项目
* 选择Product > Scheme > Manage Schemes
* 将对应的scheme勾选上Shared


![selectShare.png](http://upload-images.jianshu.io/upload_images/1453111-220648d2ebc06394.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)


### 安装 配置 GitLab Runner

	下载runner到本地
	sudo curl --output /usr/local/bin/gitlab-ci-multi-runner https://gitlab-ci-multi-runner-downloads.s3.amazonaws.com/latest/binaries/gitlab-ci-multi-runner-darwin-amd64 
	//修改目录权限
	sudo chmod +x /usr/local/bin/gitlab-ci-multi-runner  

这样就完成了Runner的安装，接下来要为工程注册一个Runner，本地注册Runner需要GitLab项目的CI地址和Token，打开到GitLab，进入对应项目选择 Setting－Runners ，
点击Runners


![clickRunner.png](http://upload-images.jianshu.io/upload_images/1453111-91f14f4c5e3f9c5c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)
  
---------------------------------------
如下图所示，我们可以看到提示。

---------------------------------------

![configRunner.png](http://upload-images.jianshu.io/upload_images/1453111-bb76738e261a7d6f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)


知道 URL 和 Token 之后就可以就可以注册Runner了

```
	gitlab-ci-multi-runner register
	WARNING: Running in user-mode.                     
	WARNING: The user-mode requires you to manually start builds processing: 
	WARNING: $ gitlab-runner run                       
	WARNING: Use sudo for system-mode:                 
	WARNING: $ sudo gitlab-runner...  
	 
	//输入之前的URL
	Please enter the gitlab-ci coordinator URL (e.g. https://gitlab.com/ci):
	https://xxxx.com/ci
	 
	//输入token
	Please enter the gitlab-ci token for this runner:
	&lt;CI runner token from Project &gt; Settings &gt; Runner&gt;
	 
	//描述，这个随意了，一般用默认的就好
	Please enter the gitlab-ci description for this runner:
	[Your-Mac\'s-Name.local]:
	 
	//runner的tag，这个是用于执行脚本时指定runner用的，所以最好起一个比较容易区分的
	Please enter the gitlab-ci tags for this runner (comma separated):
	test_machine
	Registering runner... succeeded                     runner=724a60b5
	 
	//runner的执行器，因为Xcode项目需要用xcodebuild来执行，所以选shell
	Please enter the executor: virtualbox, ssh, shell, parallels, docker, docker-ssh:
	shell
	 
	Runner registered successfully. Feel free to start it, but if it's running 
	already the config should be automatically reloaded! 
```

	
这时候再刷新GitLab项目的 Runner 界面就会看到一个 Active Runner 了
确认好了之后可以启动 Runner 了。

	cd ~
	gitlab-ci-multi-runner install
	gitlab-ci-multi-runner start

### 编写 yml 配置文件

	stages:
	  - build
	  - archive
	
	build_project:
	  stage : build
	  script :
	        - xctool -workspace XXX.xcworkspace -scheme parent clean
	        - xctool -workspace XXX.xcworkspace -scheme parent -sdk iphonesimulator9.2 -destination name="iPhone 6"  test
	archive_project:
	  stage: archive
	  script:
	        - xctool -workspace XXX.xcworkspace -scheme parent -configuration AppStoreDistribution archive -archivePath build/parent
	        - xcodebuild -exportArchive -archivePath build/parent.xcarchive -exportOptionsPlist exportOptions.plist  -exportPath build/parent
	        - fir p $PWD/build/parent/parent.ipa -T 8a7cf204f8c40f39a6ba41db8b44929d
	  only:
	        - master


>在项目根目录下配置 `.gitlab-ci.yml` 文件

>* 上面的配置文件中定义了两个`stages`。
* 用于描述两个阶段，一般有`build，test，archive，deploy`等。
* `stage` 可以在所有job中使用,上面的`build_project`和`archive_project` 就是job，每个job 分别对应执行哪个`stage`。
* script 中就是你要执行的脚本。
* 如果你的项目中没有使用` xcworkspace`，就把相应的 `－workspace XXX.xcworkspace `改为` -xcodeproject XXX.xcodeproject` 。
* 配置文件中用到了xctool 工具,和xcodebuild 相似，具体安装及使用 [xctool](https://github.com/facebook/xctool)
* `archive` 过程将项目`export `出 ipa 文件 并上传至 fir 平台。fir 工具可以上传 ipa 文价，具体使用 [fir 使用](https://github.com/FIRHQ/fir-cli/blob/master/README.md)
* `exportOptions.plist`是放在根目录下的配置打包参数的，可以参考 `xcodebuild --help `
* `only - master `是指只有提交master 分支才有执行 `build_project job`。

> fir 安装   

	gem install fir-cli   

>上传至fir  

	fir p path/to/xxx.ipa -T #API TOKEN# 

![apitoken.png](http://upload-images.jianshu.io/upload_images/1453111-2cf30eadebbb0561.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

如果一切成功的话，就会有如下 passed 标志 。

![result.png](http://upload-images.jianshu.io/upload_images/1453111-30512da4415ca5e2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)


小结，在测试，打包过程中可能会出现许多奇怪的错误，可以自己google或者询问我。

[demo 地址](https://gitlab.oneitfarm.com/tanhui/iOS_Ci_Test)

## Fastlane

### 上述是使用xctool 打包并上传fir ，测试人员可以去fir 的网站扫码下载。这里介绍另一种方式。这里安装什么的我就不赘述了，官方都有，就将个人是怎么使用的。

整个过程还是遇到很多问题的，整个过程不断google，个人从ruby环境2.0 切换到2.3 ，最后在2.2.3 上才ok。

Fastlane 简化了测试，打包，上传testFlight 功能，其实包含了上述 xctool 的所有功能。Fastlane 每个过程都用一个lane 标示。

自动测试：
```
	lane :test do
    scan({
      workspace:"ParentAndSchool.xcworkspace",
      scheme:"parent",
      device:"iPhone 6s",
      clean: true
    })
	end
```
看一下scan 中的参数，`workspace` 表示项目的命名空间，如果你用`cocoapods` 集成是有这个文件的，如果没有使用`xcworkspace` ,这里参数就改为` project`，对应的就是 `xxx.xcodeproj `文件。

打包

```
	lane :archive do
	    increment_build_number(
	        xcodeproj: "./jyb_ios_parent/parent/frameworks/runtime-src/proj.ios_mac/parent.xcodeproj"
	    )
	    gym(
	        workspace: "ParentAndSchool.xcworkspace",
	        scheme: "parent",
	        output_directory:"build",
	        output_name: "jyb.ipa"
	    )
	end
```
	
`archive` 这个`lane` 我做了两个`action`，分别是` increment_build_number` 和 `gym` 。

* `increment_build_number `这个动作是增加`build`号，我们知道，上传`appstore` 时`build`号要增加，这个需要`xcode`中一些配置，可以看[这里](https://developer.apple.com/library/content/qa/qa1827/_index.html),如果你没配置好，fastlane 跑脚本的时候不会直接挂掉，只会有提示， 到时候上传时就会失败。
* `gym` 这个`action` 用来打包，参数就不用多说了。

上传testFlight
```
	lane :testFlight do
	#for firwall
    ENV["DELIVER_ITMSTRANSPORTER_ADDITIONAL_UPLOAD_PARAMETERS"] = "-t DAV"
    pilot(
        ipa:"build/jyb.ipa",
        username: "*******@corp-ci.com",
        skip_waiting_for_build_processing: true
    )
  	end
  ```
` pilot `是可以将包自动发布到`itunes testFlight` 后台的，一开始怎么也传不上去，好像是网络之类的问题，后来才发现需要加上下面这句话。(官方文档还是要好好读的)

``` 	
 ENV["DELIVER_ITMSTRANSPORTER_ADDITIONAL_UPLOAD_PARAMETERS"] = "-t DAV"
 ```
 具体某个action 的参数及使用，可以通过命令 `fastlane action ***` ,例如`(fastlane action gym)`查看。
 
 另外在搭配 `gitlab－ci` 的过程中，发现每次`archive` 出来的包，到上传的时候总是说ipa文件找不到。因为`gitlab` 每个`job `之后都会还原仓库的状态，就是`archive` 这个`job` 结束后，由于ipa 文佳不属于git仓库，被删了。解决办法如下，修改` .gitlab-ci.yml`:

 ```
	 archive_parent_project:
	  stage : archive
	  script :
             - fastlane match appstore 
	         - fastlane archive
	  only:
	         - master
	  cache:
	     paths :
	         - build/
	         - build/jyb.ipa
	     key: "build"
	     untracked: true
	upload_testFlight:
	  cache:
	     paths :
	         - build/
	         - build/jyb.ipa
	     key: "build"
	     untracked: true
	  stage : testFlight
	  script :
	         - fastlane testFlight
	  only:
	         - master
 ```
添加缓存文件，指定每个`job`结束后不删除`build` 布目录下的文件。
`fastlane match appstore` 是用match 给app 配置证书，具体使用请看 [Match](http://www.jianshu.com/p/a3e20bfd29f1)
