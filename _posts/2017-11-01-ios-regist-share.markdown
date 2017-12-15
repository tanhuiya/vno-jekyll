---
layout: post
title: iOS 注册可分享文件类型
date: 2017-11-01 15:59:28.000000000 +09:00
---

最近开发的 `App` 有个需求，需要获取本地的录音文件。找了半天没有找到好的解决方案，发现微信的方法是在 录音备忘录 中分享至微信。

[苹果官方文档地址](https://developer.apple.com/library/content/documentation/FileManagement/Conceptual/DocumentInteraction_TopicsForIOS/Articles/RegisteringtheFileTypesYourAppSupports.html#//apple_ref/doc/uid/TP40010411-SW1)

## 注册可接受文件类型

因为我打开的是录音文件，这里就以 `iOS` 的录音文件为例 (`*.m4a`)

* 打开 `info.plist` 文件，添加项 `Document types`
* 展开 `Document types` 的 `item 0` 选项
* `Document Type Name` 自定义`Name`，自己可以随便填
* `CFBundleTypeIconFiles` 指定分享时显示的 `App Icon` ,这里我没有填，会用默认的
* `Handler rank` 定义优先级，`Owner` 的优先级最高，分享至 `App` 时，你的会靠前排列
* 在 `item0` 中添加 `key` `Document Content Type UTIs` 这边是指定你注册的可以接受的文件类型, 文件类型的对应关系如下图.

![WX20171114-173628.png](http://upload-images.jianshu.io/upload_images/1453111-49016659a7593641.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

这里我要接受 `m4a` 文件，所以 注册类型为 `public.audio` 。 `public.data` 代表可以接受任何类型的文件。如果想注册多个类型，可以在 `Document types` 下面继续添加 `item` 。
![WX20171115-110036.png](http://upload-images.jianshu.io/upload_images/1453111-17f060cb0a37adca.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

这时看你的工程文件 `info` 选项卡中的 `Document types`，应该有你刚才注册的类型.

![WX20171115-110007.png](http://upload-images.jianshu.io/upload_images/1453111-0020f680edaf4728.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 接受文件代码

当文件分享至 `App` 时会调用 `Appdelegate` 的如下函数. 文件地址会在参数 `url` 中传入。

```
#if __IPHONE_OS_VERSION_MAX_ALLOWED < __IPHONE_9_0
- (BOOL)application:(UIApplication *)application openURL:(NSURL *)url sourceApplication:(nullable NSString *)sourceApplication annotation:(id)annotation
{
	// dosomething
    return YES;
}
#else
- (BOOL)application:(UIApplication *)application openURL:(NSURL *)url options:(nonnull NSDictionary<NSString *,id> *)options
{
	// dosomething
    return YES;
}
#endif
```

效果图如下
![WX20171115-113750.png](http://upload-images.jianshu.io/upload_images/1453111-ef8197ff658d9138.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/320/h/480)

注意，这里分享是显示的是拷贝至，点击更多，打开微信的 `switcher` 之后，会像上图一样多一个 '微信' 选项(没有'拷贝至')，它会在当前应用弹窗，给出分享逻辑。具体实现方法还不清楚，等有时间再研究。

![WX20171115-113722.png](http://upload-images.jianshu.io/upload_images/1453111-2bf9537306a26b2a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/320/h/480)

若有知道，还请赐教。