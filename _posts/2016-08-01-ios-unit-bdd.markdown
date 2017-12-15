
---
layout: post
title: iOS 单元测试 - BDD
date: 2016-08-01 15:59:28.000000000 +09:00
---

# 单元测试

#### 为什么需要单元测试
* 减少代码中的低级错误。
* 有效的降低bug的出现率。
* 增强可维护性。
* 有助于设计：写单元测试首先给了你一个如何设计 API 的清晰视角。
* 质量保证，根据我的自身经历，让一个开发者记得要测试所有的特性，在代码改变后回归测试所有的功能以及新增或移除的功能，几乎是一件不可能的事情。





#### 被测试的对象，方法大概分为三种:   

* 有明确的返回值，采用返回值验证法，验证返回值是否符合预期。   
* 没有返回值，但方法内部修改了对象的属性或者状态，采用状态验证法，是否符合预期。  
* 依赖于外部的类，方法，会调用外部的方法，采用行为验证法。   


#### 单元测试可能遇到的问题  

* 测试上下文有太多依赖，设计的耦合性太高。   
* 运行的速度缓慢，你的单元测试中可能存在外部系统，列入数据库，网络请求，文件系统等。   
* 改变一个地方，多处测试受影响，可能是测试设计的问题，也可能是代码的粒度拆分不够。   


* 怎样测试私有方法——私有方法有太多的行为 。 

#### 测试哪些东西

你在测试哪个组件切面（component aspect）？
这个特性做什么用？你测试的具体行为需求是什么？
针对行为的测试，这是一种行为驱动开发技术(BDD)，可以参考 [unit-testing-tdd-and-bdd](https://codeutopia.net/blog/2015/03/01/unit-testing-tdd-and-bdd/)。那什么是行为？
你设计的App中有一个对象，它有一个接口定义了其方法和依赖关系。这些方法和依赖，声明了你对象的约定。它们定义了如何与你应用的其他部分交互，以及它的功能是什么。它们定义了对象的行为。这同时也是你的目标，测试对象的行为。
比如点击按钮是否触发了某个行为。

#### 怎样进行单元测试 

单元测试本质上来说就是用断言来判断对象是否达到预期的行为。   
单元测试的关注点单一，单元测试需要保证你每个测试用例是针对一个单元，而不是一个有很多复杂依赖注入的综合行为。   
我们尽可能让类方法的职责单一，这样才能保证变化点都集中在被测试的单元中。   
单元测试一般比较静态，它只是验证某一动作的正确性。
大部分单元测试将针对对象的状态，来断言一个特定的交互是否发生，或者一个特定值是否返回。将依赖提取出来，这可以允许你轻松mock。
注意，你不该将对象的所有依赖都暴露在头文件中，尤其是你开始测试的时候，这样看起来很诱人，但会破坏你类结构的封装，你的接口应该只表述设计需求。

* 充分了解需要测试的类的行为特性。
* 对代码重构，针对这些行为编写单元测试。
* 使用伪造类避免对其它类的依赖：被测试的方法需要某个对象作为参数，但测试类中并不关心对象的具体实现。
* 伪造环境避免其它环境的干扰：比如在网络请求异步的环境中，测试代码不好写，可以将该步骤分开，测试请求网络行为，再模拟数据返回，测试网络数据返回后的行为。也可以采用GHUnit等第三方框架。


#### 单元测试坏的实践

* 不应该测试构造函数--构造函数中应该是实现类的一些细节，而我们是针对对象的行为做测试，所以构造函数没有值得测的东西。
* 不要测试私有方法--私有方法意味着私有，如果你觉得你有必要测试私有方法，那可能你的私有方法中做的事太多了，从而违背了单一职责原则。
* 不要stub私有方法--因为你的私有方法是可以在类中不经通知自由修改的，当stub私有方法后，私有方法修改后可能与你的期望背到而驰，但你的测试还是会通过，这是见很可怕的事情。
* 不要stub第三方库--比如你stub了[AFNetworking sendRequest]方法，不需要通过实际的网络调用是测试内容更单一，当你更换了这个网络库之后，这个测试用例就会失效，而你实际上stub的目的就是模拟网络请求成功。所以测试中应该封装一层，来代替那个库的全部功能。

### Given / When / Then 模式

将测试用例分为三个部分   
* Given 部分，通过创建对象，活着stub对象，将测试的系统设定到指定的状态，来设置测试的环境。  
* When 部分包含了我们具体需要测试的代码。   
* Then 部分是验证测试的结果，是否达到我们的期望，对象是否有改变，返回值是否合格等。   

下面是家园宝测试作业是否正常继续下载的例子：

	-(void)testResumeWithHomeWorkModel{
	  //Given
	  if (!self.manager) {
	      self.manager = [DownLoadHomeWorkManager manager];
	  }
	  HomeModel* homeModel = [[HomeModel alloc]init];
	  homeModel.pid = @"497";
	  //When
	  [self.manager resumeTaskWith:homeModel];
	  //Then    
	  XCTAssertEqual(homeModel.status, HomeworkStateDownloading);
	}

### Mock

在iOS测试中的mock框架可以采用OCMock，我们用mock来管理一个对象的所有依赖。当被测试的方法里耦合着其它对象时，但是你不想让这个对象的返回值对这个方法有影响，你可以通过mock 的方式返回一个默认值。   
另外，我们的测试代码中不能过度的使用mock，mock除去了被测试对象以外的其它对象，这样其它对象修改了之后，这个被测试的对象就不能自动失败。

## 单元测试的要求
* 测试用例应该是自解释且独立的：每个测试用例应该不依赖于其它方法的结果作为输入，没有网络请求，没有数据库操作，保证其原子性。
* 测试方法需要解释测试的目的：如 -(void)testObject 就不是一种规范的写法，每个方法的目标应该是单一的，大多数每个方法里都有一个断言。
* 断言语句需要解释测试者的用途：如XCTAssertNotNil,[xxx should]beNil],等等。
* 判断某个测试是否成功是检测方法影响的数据有没有合理的变化：由于单元测试是使用断言来判断的，单元测试中不会对显示层进行约束，所以限定了单元测试的范围，即引起数据的变化。
* 对所有暴露的属性和方法提供测试，私有方法则不必：测试私有方式可以通过子类化，设计分类，kvo等方式获取私有或者内部对象。
* 变化需要新测试的支持：当对外的接口的实现发生变化时，需要编写新的测试。
* 发现bug 并修复后，为了确保修复时成功的，需要进行单元测试。
* 测试有其他依赖时需要避免其它依赖的副作用，可以采用依赖注入的方法， [依赖注入](https://objccn.io/issue-15-3/) ，具体可以使用mock 或者真实对象注入。
* 对每一个功能类都要做单元测试。
* 单元测试需要描述和记录代码需要实现的所有需求。

## 单元测试框架

### XCTest

XCTest 是iOS自带的一个测试框架，相比于其他第三方集成度高，能满足大部分测试需求。但是并没有提供mock的功能。

### OCMock

OCMock 是一个OC的模拟对象库，他提供了关于mock 和 stub 的功能，可以和XCTest一起使用。它看起来像这样。

	- (void)testAddDownLoadHomeWorkModel{
	   if (!self.manager) {
	       self.manager = [DownLoadHomeWorkManager manager];
	   }
	   HomeModel* homeModel = [[HomeModel alloc]init];
	   //mock 出一个dataCenter
	   id dataCenter = OCMClassMock([HomeworkDataCenter class]);
	   self.manager.dataCenter = dataCenter;
	   homeModel.pid = @"497";
	   //该方法被调用时返回1
	   OCMStub([dataCenter insertHomeModel:homeModel]).andReturn(1);
	   [self.manager addDownLoadTask:homeModel];
	   XCTAssertEqual(homeModel.status, HomeworkStateDownloading);
	}

### Kiwi
[Kiwi](https://github.com/kiwi-bdd/Kiwi) 是一个行为驱动开发(BDD)的框架，它旨在解决具体问题，帮助开发人员确定应该测什么内容。你不应该关注于测试，而是应该关注于行为。
该框架相比iOS自带的XCTest，它的语法更类似于自然语言，易读性强。
Kiwi 更多使用方法点击 [这里](http://www.tuicool.com/articles/3auQbez)

    SPEC_BEGIN(First)
       describe(@"First", ^{
        context(@"create a string", ^{
            __block NSString * name = nil;
            beforeEach(^{
                name = @"aa";
            });
            it(@"name should be aa", ^{
                [[name shouldNot]beNil];
            }); 
        });
    });
    SPEC_END

## 测试实例

![Unitest.png](http://upload-images.jianshu.io/upload_images/1453111-bfee9746bad9e568.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/500)
  
首先，我们来看一下iOS的UIViewController。对代码分析时，发现大量的的逻辑都被写在 .m 文件里，我们知道，UIViewController 在 .h 里面暴露的方法很少，可是 .m 大量的逻辑单元测试又不能不做，这就相当于要对代码中的private 方法进行测试。
进一步分析发现，如果在ViewController 中 添加一个ViewModel层，将UIViewController 里的业务逻辑放入中间层，该层可以负责网络的请求，数据的处理等。一方面会使ViewController 更加简洁和实现单一原则，另一方面保证了逻辑的可能性，该中间层会对ViewController 暴露一些接口。在MVC的设计模式中，ViewController 承受了太多的任务导致测试的难度增加，将ViewController 拆分（MVVM）就会更加有利于单元测试。
                   
    describe(@"Bind ViewModel", ^{
        __block WrapJSMessageView* replyView = nil;
        __block WrapJSMessageViewModel* replyViewModel = nil;
        replyView = [[WrapJSMessageView alloc]init];
        replyViewModel = [[WrapJSMessageViewModel alloc]init];
        replyView.replyViewModel = replyViewModel;
        
        context(@"Test Text Binding  ", ^{
            it(@"Image Should NotNil", ^{
                [[replyViewModel.image shouldNot]beNil];
            });
            
            //select Image;
            {
                UIImage* image = [UIImage new];
                NSDictionary* dict = @{};
                [dict stub:@selector(objectForKey:) andReturn:image withArguments:@"UIImagePickerControllerOriginalImage"];
                [replyView imagePickerController:[UIImagePickerController new] didFinishPickingMediaWithInfo:dict];
                it(@"select Image", ^{
                    [[replyViewModel.image should]equal:image];
                });
            }
            
            // clear text
            {
                it(@"After reset Image should Be reset", ^{
                    [[replyView should]receive:@selector(resetReplyView)];
                    replyViewModel.text = nil;
                });
            }
            
            
            UIButton* sendBtn = nil;
            [UIView getViewByTitle:@"发送" rootView:replyView resultView:&sendBtn];
            
            // test Click ReplyBtn
            {
                it(@"Get Send Btn", ^{
                    [[sendBtn shouldNot]beNil];
                });
            }
            // Send Btn should Be disable
            {
                it(@"Send Btn should Disable", ^{
                    [[theValue(sendBtn.enabled)should] beFalse];
                });
            }
            
            //Send Btn should Be disable
            {
                it(@"Send Btn should Enable", ^{
                    replyViewModel.text = @"222";
                    [[theValue(sendBtn.enabled)should] beFalse];
                });
            }
            
            // ReplyBtnClicked
            {
                [[replyViewModel.replyCommand should]receive:@selector(execute:)];
                
                it(@"Send Should Invalid", ^{
                    [sendBtn sendActionsForControlEvents:UIControlEventTouchUpInside];
                });
            }
            
        });
        
    })
                       
DetailNewViewController

	describe(@"DetailNewViewController", ^{
	    __block DetailNewViewController* detail = [[DetailNewViewController alloc]init];
	    context(@"initial with valid post_id", ^{
            
            id viewModel = OCMClassMock([PostDetailViewModel class]);
            detail.viewModel = viewModel;
            detail.postid = @"100";
            it(@"initWithPostid should be invoked", ^{
                OCMVerify([viewModel initWithPostId:[OCMArg any]]);
            });
        });
        
        context(@"Inital", ^{
            
            PostDetailViewModel* viewModel = [[PostDetailViewModel alloc]initWithPostId:@"1"];
            
            // 测试获取帖子数据
            {
                
                MJRefreshGifHeader* header = nil;
                //获取header
                [UIView getRefeshHeader:detail.view resultView:&header];
                it(@"RefreshController should not be nil", ^{
                    [[header shouldNot]beNil];
                });
                
                
                RACCommand* fetchRawCommand = OCMClassMock([RACCommand class]);
                viewModel.fetchRawDataCommand = fetchRawCommand;
                
                [detail viewDidLoad];
                it(@"should Request Post Data", ^{
                    OCMVerify([header beginRefreshing]);
                });
            }
            
            
            //测试获取评论列表
            
            {
                NSDictionary* returnData = @{
                                             @"ret":@(1),
                                             @"retCode":@(1),
                                             };
                
                detail.viewModel = viewModel;
                id fetchReplyCommand = OCMClassMock([RACCommand class]);
                viewModel.fetchReplyListCommand = fetchReplyCommand;
                
                [detail showWithDict:returnData];
                it(@"Then Request ReplyList Data", ^{
                    OCMVerify([fetchReplyCommand execute:[OCMArg any]]);
                });
            }
        });
        
        context(@"Test Click Collect Btn", ^{
            
            detail.isIntersting = NO;
            detail.contentData = [NSDictionary mock];
            PostDetailViewModel* viewModel = [[PostDetailViewModel alloc]init];
            id command = OCMClassMock([RACCommand class]);
            viewModel.collectCommand = command;
            
            detail.viewModel = viewModel;
            [detail soucangBtnDidClicked];
            
            it(@"Send NetWork should Raised ", ^{
                OCMVerify([command execute:[OCMArg any]]);
            });
            
        });
        
        context(@"Test Click Uncollect Btn", ^{
            detail.isIntersting = YES;
            detail.contentData = [NSDictionary mock];
            PostDetailViewModel* viewModel = [[PostDetailViewModel alloc]init];
            id command = OCMClassMock([RACCommand class]);
            viewModel.unCollectCommand = command;
            
            detail.viewModel = viewModel;
            [detail soucangBtnDidClicked];
            
            it(@"Uncollected should Raised ", ^{
                OCMVerify([command execute:[OCMArg any]]);
            });
        });
        
        
        context(@"Test NavgationItem ", ^{
            UIView* title = detail.titleSegment;
            it(@"should Not Nil", ^{
                [[title shouldNot]beNil];
            });
            
            it(@"should Be UISegmentControl", ^{
                [[title should]beKindOfClass:[UISegmentedControl class]];
            });
            UISegmentedControl* segTitle = (UISegmentedControl*)title;
            it(@"should have Three Segment ", ^{
                [[theValue(segTitle.numberOfSegments) should]equal:@(3)];
            });
        });
        
    })
                       
DetailNewViewController+Spec.h
                       
    #import "DetailNewViewController.h"
                       
	@interface DetailNewViewController (Spec)
	   
	@property(nonatomic,assign)NSInteger  isIntersting;
	   
	@property(nonatomic , strong) NSDictionary* contentData;
	   
	@property(nonatomic , strong) UISegmentedControl* titleSegment;
	   
	-(void)showWithDict:(NSDictionary*)dict;
	   
	@end
	               
// UIView 的分类
                   
    #import "UIView+Spec.h"
    #import "MJRefresh.h"
	@class MJRefreshGifHeader;
	   
	@implementation UIView (Spec)
	   
	+(void)getRefeshHeader:(UIView*)rootView resultView:(UIView**)result{
	   for(UIView * view in rootView.subviews){
	       if ([view isKindOfClass:[MJRefreshGifHeader class]]) {
	           *result = view;
	       }else {
	           [self getRefeshHeader:view resultView:result];
	       }
	   }
	}
	   
	+(void)getViewByTitle:(NSString*)title rootView:(UIView*)rootView resultView:(UIView**)result{
       
       for(UIView * view in rootView.subviews){
           if ([view isKindOfClass:[UIButton class]]&&[[(UIButton*)view titleLabel].text isEqualToString:title]) {
               *result = view;
           }else {
               [self getViewByTitle:title rootView:view resultView:result];
           }
       }
	}
	@end
   

[BDD实例](https://objccn.io/issue-15-1/)

## 我遇到的问题
刚开始做单元测试的时候，根本无法下手，帖子模块的版本迭代频繁，业务逻辑复杂，代码行数达到2400行左右。再看代码结构相当混乱。帖子模块所有的数据，包括从网络的发起，数据的接收，界面的显示都混杂在一起，只有少量的view 单独抽了出去，即使是封装的view，数据的显示还是在帖子的控制器中做的。这真的是MVC(Massive-View-Controller)了。为了方便测试，先找几个行为特性测起来，我把网络数据的请求和接受，全部封装到帖子的ViewModel中，抽离出回复框，并给这个view配备了一个ViewModel（因为回复框中也有不少的逻辑），控制器只需要新建并添加就ok了。这样针对回复框的一些行为就可以提取出来测了。帖子页一些行为可以在ViewModel 中测试。   
在测试的过程中，由于大量的原生数据的显示逻辑都在帖子页的ViewController中，而在测试这个控制器的一些行为时，无法提供帖子的原生数据，或者说因为原生数据格式复杂而难以高效的注入，导致测试的时程序崩溃。经过思考，觉得还是应该将所有的数据交给ViewModel 管理，ViewController或View 应该仅仅和ViewModel 进行数据上的绑定。这样在测试ViewContrllor时就不会对数据有过多的依赖。在测试ViewModel时也能更集中的测试数据的有效性。

# 总结

在做单元测试的时候，更多思考一个对象的行为，它的接口应该如何，并减少对实现的关注。这样你会有更加健壮的代码，以及同样杰出的套件。单元测试的代码简单，但是写好单元测试却不是一件简单的事，对程序员的代码质量要求较高，如何有效的组织行为就考验程序员的水平了。从现在开始，让单元测试来帮你描述代码的行为。