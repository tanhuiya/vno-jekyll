---
layout: post
title: iOS 单元测试 - TDD
date: 2016-08-01 15:59:28.000000000 +09:00
---

# TDD 

### 测试驱动开发（Test Driven Development,缩写TDD），它的基本思想就是在开发功能代码之前，先编写测试代码。也就是说在明确要开发某个功能后，首先思考如何对这个功能进行测试，并完成相应的测试用例，然后编写相关代码满足这些测试，然后循环添加这些功能，直至开发结束。

## TDD 的优点

* 开发完成即完工。传统的编码方式很难知道什么时候编码结束了，TDD模式下开发人员可以明确自己的编码工作已经结束了。
* 代码大部分保持在高质量状态。
* 减少文档和代码之间的差别。

## 开发过程:

* 明确当前要完成的功能。可以记录成一个 TODO 列表。
* 快速完成针对此功能的测试用例编写。
* 测试代码编译不通过。
* 编写对应的功能代码。
* 测试通过。
* 对代码进行重构，并保证测试通过。
* 循环完成所有功能的开发。

## 第1次迭代 用户场景

![Example.png](http://upload-images.jianshu.io/upload_images/1453111-dc61888a76f72d5b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/300 )

用户场景 

* 作为用户，我要看到文章列表。
* 作为用户，我要切换分类，看到不同的文章。
* 作为用户，当我点击更多时，我要看到更多的分类。
* 作为用户，我可以刷新最新的数据。
* 作为用户，我可以获取更多的数据。
* 作为用户，我要看到上方的轮播图。
* 作为用户，我点击没一个文章，要进入文章详情。
* 作为用户，我点击轮播图，要进入文章详情。

初步构建模块结构图


![TDD-case.png](http://upload-images.jianshu.io/upload_images/1453111-28b3520ae7a965aa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000)

## TDD - Model
> 我们看到列表的每个条目都会显示 title和 content。
> 那么我们就可以先编写测试用例。
>
	- (void)testInitial {
	    ArticalModel * artical = [ArticalModel new];
	}
	
> 编译，显然不会通过，因为我们没有 ArticalModel 这个类，所以在我们的项目的代码里创建一个 ArticalModel 这个类。再编译，通过。此时不管时我们的测试的代码，还是项目的代码，都没有可以重构的。
> 
> 接下来，我们看到列表里有title ,content, 日期和图片，对应到我们ArticalModel 里应该也有。接下来编写我们的测试用例。
> 
 	- (void)testInitial_with_infomation {
	    ArticalModel * artical = [ArticalModel new];
	    artical.title = @"";
	    artical.name = @"";
	    artical.date = @"";
	    artical.imageUrl = @"";
	    XCTAssertNotNil(artical,"artical shouldNot be nil");
	}
> 编译，失败。因为我们的Artical 里没有title，name等属性。好，那我们为 Artical 添加这些属性。
> 
	@interface ArticalModel : NSObject
	@property(nonatomic , strong) NSString* title;
	@property(nonatomic , strong) NSString* name;
	@property(nonatomic , strong) NSString* date;
	@property(nonatomic , strong) NSString* imageUrl;
	@end
	
> ok,编译通过，那看看我们的测试。测试通过了。
> 我们的最简易的模型先到这里。接下来，管理模型的是ArticalManager ，新建 ArticalManagerTest 测试类，同样测试初始化代码。
> 
	- (void)testInitialManager{
	    ArticalManager * manager = [ArticalManager new];
	    XCTAssertNotNil(manager ,@"manager should not be nil");
	}
> 编译出错，同样，新建ArticalManager 类并导入。
> articalManager 里是放 ArticalModel 的，所以接下来测试获取所有 AricalModel。
> 
	- (void)testGetArticalsNotNil{
	    ArticalManager * manager = [ArticalManager new];
	    id articals = manager.articals;
	    XCTAssertNotNil(articals,@"articals should not be nil");
	}
同样，编译报错，因为我们的ArticalManager 里没有articals属性，我们给ArticalManager 加上artical 属性。
>
	@interface ArticalManager : NSObject
	@property(nonatomic , strong) NSArray* articals;
	@end
	
>编译成功，但是测试没有通过，显然，因为我们的articals 没有初始化，默认为空。我们默认情况下应该返回一个空数组，再为ArticalManager 里实现 articals 的懒加载方法。
>
	@implementation ArticalManager
	-(NSArray*)articals{
	    if (!_articals) {
	        _articals = @[];
	    }
	    return _articals;
	}
	@end
	
> 再次编译。测试通过，这时看看测试代码发现有地方可以重构，两个测试方法里都创建了ArticalManager 新的实例
> 
	- (void)testInitialManager{
	    ArticalManager * manager = [ArticalManager new];
	    XCTAssertNotNil(manager ,@"manager should not be nil");
	}
	- (void)testGetArticalsNotNil{
	    ArticalManager * manager = [ArticalManager new];
	    id articals = manager.articals;
	    XCTAssertNotNil(articals,@"articals should not be nil");
	}
> 我们可以把 manager 抽出来，变为测试类的一个属性，初始化方法放在 setUp 里，代码就变成这样
> 
	- (void)setUp {
	    [super setUp];
	    self.manager = [ArticalManager new];
	}
	- (void)testInitialManager{
	    XCTAssertNotNil(self.manager ,@"manager should not be nil");
	}
	- (void)testGetArticalsNotNil{
	    id articals = self.manager.articals;
	    XCTAssertNotNil(articals,@"articals should not be nil");
	}
> 接下来，我们的ArticalManager 应该可以添加一个 文章的模型。
> 
	- (void)testAddAnArticalModel{
	    ArticalModel* artical = [ArticalModel new];
	    [self.manager addArticalModel:artical];
	    XCTAssertEqual(1,self.manager.articals.count,@"should have  one artical");
	}
> 编译报错。那我们为 ArticalManager 添加 addArticalModel 的接口,并完成它的实现。
> 
	－(void)addArticalModel:(ArticalModel*)artical{
	    [self.articals addObject : artical];
	}
> 编译报错，self.articals 没有addObject方法，因为articals 是 NSArray 类型的，我们把它定义成NSMutableArray再看看，修改如下
> 
	@property(nonatomic , strong) NSMutableArray* articals;
	-(NSMutableArray*)articals{
	    if (!_articals) {
	        _articals =[ @[] mutableCopy ];
	    }
	    return _articals;
	}
> 编译，测试通过，可以正常添加ArticalModel。再取出来测试一下是不是我们添加的 ArticalModel 
> 
	- (void)testAddAnArticalModel{
	    ArticalModel* artical = [ArticalModel new];
	    [self.manager addArticalModel:artical];
	    XCTAssertEqual(1,self.manager.articals.count,@"should have  one artical");
>	    
	    artical.title = @"titleTest";
	    XCTAssertEqual(@"titleTest",[self.manager.articals[0] title],@"artical title should be equal");
	}
> 果然测试通过，那要是我们随便添加一个模型试试看
> 
	- (void)testAddIlegalModel{
	    NSObject* illegalModel = [NSObject new];
	    [self.manager addArticalModel: illegalModel];
	    XCTAssertEqual(1,self.manager.articals.count,@"should have  one artical");
	}
> 通过了，但这不应该是一个成功的测试，或者说上一步断言编写不正确，那我们修改测试用例，
> 
	- (void)testAddIlegalModel{
	    NSObject* illegalModel = [NSObject new];
	    [self.manager addArticalModel: illegalModel];
	    XCTAssertEqual(0,self.manager.articals.count,@"should have no articals");
	}
> 测试不通过，因为非法的模型已经被加到 articals 数组里去了，然而，不是什么对象我们都可以给它添加进我们的ArticalManager 的，我们的ArticalManager 要为 TableView 服务，所以要严格控制 ArticalManager 数组内的元素。  
>  
> 如何控制呢，可以在添加前判断一下被添加的对象是不是 Artical 类。但是这种代码可扩展性低，将来如果tableView 需要显示其它类型（比如 公告）的cell，而数据格式不完全一样，公告就不该被加进来。
> 
> 第二种方法可以以继承的方式实现，比如我们把ArticalModel 作为基类，将来扩展的 BroadCastModel 可以继承 ArticalModel ,因此在添加进 Articals 数组的时候我们直接使用基类的指针判读类型。
> 
> 第三种方法使用interface 的方式，对应于 OC 的protocal ，协议（接口）相比于类，耦合度更低。假设我还有一种模型 MessageModel 需要显示，其数据模型和 BroadCastModel 相似 ，即需要被加进去 articals 数组，用第二种方法实现让 MessageModel 继承 BroadCastModel ，同样可以达到需求 ， 但是如果日后业务需求增加，需要更多的显示种类，一昧的使用继承的方式将导致整棵模型树层级越来越深。假设有需求，需要改变你的层级树中某一个Model 的属性或者私有方法，那如果继承了它的子类用到了该属性或方法，就要相应的去修改。当我需要测试 MessageModel 时，需要纵向依赖BroadCastModel 和 ArticalModel ，一旦业务抽离，在另一个模块或者项目中使用 MessageModel 时，需要将这里的继承树连根拔起。所以，如果你的继承关系层级达到了三层或者更多，就应该停下来思考一下设计是否合理。 采用 interface 或 protocal 的方式，是横向依赖，不管是ArticalModel 或者 MessageModel ，我要做tableView 的数据源，就要实现 TableViewModelCellProtocal 的方法, 这样就能减少耦合度。其实这解决的是如何在 tableView 中显示不同的数据格式的数据。
> 
> 因为我们这边只是测试的实例，我们就采用最简单的第一种方法，在 addArticalModel 中添加如下代码 
> 
	if (![artical isKindOfClass:[ArticalModel class]]) {
        return;
    }
> 接下来，考虑到我们的数据是从网络回来，肯定不是一条记录一条记录的添加，那我们就编写批量数据的测试
>
	- (void)testAddArticalArrToManager{
	    NSArray * articalArr = nil;
	    [self.manager.articals addArticalArrs : articalArr];
	    XCTAssertEqual(0,[self.manager.articals count],@"articals should be eqmpty");
	}
> 编译失败，我们为 ArticalManager 添加该接口
> 
	-(void)addArticalArrs: (NSArray <NSObject * >* )aricalArr{   
	}
> 编译测试均通过，接下来我们看一下添加一个真实的模型数组，
> 
	-(void)testAddRealArticalArrToManager{
	    ArticalModel * articalModel = [ArticalModel new];
	    articalModel.name = @"artical_one";
	    NSArray * articalArr = @[articalModel];
	    [self.manager addArticalArrs : articalArr];
	    XCTAssertEqual(1,[self.manager.articals count],@"articals should be eqmpty");
	}
> 测试不通过，原因是addArticalArrs 方法里没有实现，我们将添加它的实现如下，
>
	[self.articals addObjectsFromArray: articalArr];
	
>  测试全都pass ，我们再看看我们添加的是不是正确的模型，添加如下代码
> 
	XCTAssertEqual(articalModel.name , [self.manager.articals[0] name],@"they should be same name");
> 果然是同一个，测试通过，再试试添加非法的Array,我们设置的断言是不应该被加入数组。
> 
	-(void)testAddIllegalArrToManager{
	    NSArray * articalArr = @[@"1",@"haha",@(2)];
	    [self.manager addArticalArrs : articalArr];
	    XCTAssertEqual(0,self.manager.articals.count,@"aricals should not be added");
	}
> 测试失败，结果是0 != 3 ，我们修改代码来满足我们的测试，我们希望它只要有一个元素不合格，就都不能插入。
>
	-(void)addArticalArrs: (NSArray <ArticalModel *>* )articalArr{
	    __block BOOL illegal = false;
	    [articalArr enumerateObjectsUsingBlock:^(NSObject* obj, NSUInteger idx, BOOL *stop){
	        if (![obj isKindOfClass:[ArticalModel class]]) {
	            illegal = true;
	            *stop = YES;
	        }
	    }];
	    if (!illegal) {
	       [self.articals addObjectsFromArray: articalArr];
	    }
	}
> 好了这次通过了，看看有没有什么可以重构的，我们可以把判断元素合格提取出来，将来如果需要更换判断方式，比如采用协议来验证，就可以很方便的修改
> 
	#pragma mark privateMethod
	-(BOOL)isItemVaild:(NSObject*)obj{
	    return  [artical isKindOfClass:[ArticalModel class]];
	}
> 提取出私有方法，isItemValid ,现在ArticalManager 代码如下
> 
	-(void)addArticalModel:(ArticalModel*)artical{
	    if([self isItemVaild: artical]){
	        [self.articals addObject:artical];
	    };
	}
	-(void)addArticalArrs: (NSArray <ArticalModel *>* )articalArr{
	    __block BOOL illegal = false;
	    [articalArr enumerateObjectsUsingBlock:^(NSObject* obj, NSUInteger idx, BOOL *stop){
	        if(![self isItemVaild: obj]){
	            illegal = YES,*stop = YES;
	        }
	    }];
	    if (!illegal) {
	       [self.articals addObjectsFromArray: articalArr];
	    }
	}
	#pragma mark privateMethod
	-(BOOL)isItemVaild:(NSObject*)obj{
	    return  [obj isKindOfClass:[ArticalModel class]];
	}
> 接下来还有清空 artical 数组的测试己，编写，过程是一样的，假设该过程我们已经完成，现在回看我们的 ArticalManager 还有没有什么可以修改的。
> articals 作为 MSMutableArray 暴露在头文件，这是相当危险的，也就是说可以让外部的类随意修改元素个数或articals 实例。所以我们将头文件的 articals 改为外界只读的。在ArticalManger.h 文件里声明articals 为NSArray，并且只读，.m 文件里使用 _articals 变量，该变量在对象创建时实例化。
> 
	.h 
	@property(nonatomic , strong , readonly) NSArray* articals;
	.m
	@interface ArticalManager(){
	    NSMutableArray*  _articals;
	}
	@end
>	
	@implementation ArticalManager
>	
	-(instancetype)init{
	    _articals = @[].mutableCopy;
	    return self;
	}
	-(void)addArticalModel:(ArticalModel*)artical{
	    if([self isItemVaild: artical]){
	        [_articals addObject:artical];
	    };
	}
	...


大概 Model 层就是这样子的，测试的思路大概也就是这样，然而这些测试还只是冰山一角。完整写下来，测试代码大概是功能代码的2～3 倍。