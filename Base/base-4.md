# 基础面试四
#### 写一个标准宏MIN，这个宏输入两个参数并返回较小的一个
> define MIN(A,B) (A)\>(B)?((B):(A)

#### iPhone OS 有没有垃圾回收机制,简易阐述一下OC内存管理
没有，oc的内存管理是依赖引用计数，ARC和MRC两个管理方式
#### 简述应用程序按HOME键后进入后台的生命周期，一起从前台进入后台时的生命周期
前者：
> - (void)applicationWillResignActive:(UIApplication *)application
> - (void)applicationDidEnterBackground:(UIApplication *)application

后者：
> - (void)applicationWillEnterForeground:(UIApplication *)application*
> - (void)applicationDidBecomeActive:(UIApplication *)application*

#### 解释程序启动后回调方法的含义
告诉代理进程启动但还没进入状态保存
> - (BOOL)application:(UIApplication *)application willFinishLaunchingWithOptions:(NSDictionary *)launchOptions

告诉代理启动基本完成程序准备开始运行
> - (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
当应用程序将要入非活动状态执行，在此期间，应用程序不接收消息或事件，比如来电话了
> - (void)applicationWillResignActive:(UIApplication *)application
当应用程序入活动状态执行，这个刚好跟上面那个方法相反
> - (void)applicationDidBecomeActive:(UIApplication *)application*
当程序被推送到后台的时候调用。所以要设置后台继续运行，则在这个函数里面设置即可
> - (void)applicationDidEnterBackground:(UIApplication *)application
当程序从后台将要重新回到前台时候调用，这个刚好跟上面的那个方法相反。
> - (void)applicationWillEnterForeground:(UIApplication *)application
当程序将要退出是被调用，通常是用来保存数据和一些退出前的清理工作。这个需要要设置UIApplicationExitsOnSuspend的键值。
> - (void)applicationWillTerminate:(UIApplication *)application
当程序载入后执行
> - (void)applicationDidFinishLaunching:(UIApplication*)application*

#### ViewController的alloc,loadView,viewDidLoad,viewWillAppear,viewDidUnload,dealloc,init分别是在什么时候调用?在自定义ViewController的时候这几个函数里面应该做什么工作?

* alloc:申请内存时调用.
* loadView:加载视图时调用.
* viewDidLoad;视图已经加载后调用.
* viewWillAppear:视图将要出现时调用.
* viewDidUnload:视图已经加载但是没有加载出来时调用.
* dealloc:销毁该视图时调用.
* init;初始化该视图时调用.

#### 描述应用程序的启动顺序
1. 程序入口main函数创建UIApplication实例和UIApplication代理实例.
2.  在UIApplication代理实例中重写启动方法,设置根ViewController
3. 在第一ViewController中添加控件,实现应用程序界面.
#### 为什么很多内置类如UITableViewControl的delegate属性都是assign而不是retain
* 防止循环引用.
* 如:对象A引用了对象B,对象B引用了对象C,对象C引用了对象B,这个时候B的引用计数是2,而C的引用计数是1,当A不再使用B的时候,就释放了B的所有权,这个时候C还引用对象B,所以B不会释放,引用计数为1,因为B也引用着对象C,B不释放,那么C也就不会被释放,所以他们的引用计数都为1,并且永远不会被释放,形成了循环引用.

#### 使用UITableView的时候必须要实现的几种方法
2个数据源方法.分别是:
> - (NSInteger)tableView:(UITableView *)tableView numberOfRowsInSection:(NSInteger)section;
> - (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath;*

#### 写一个遍历构造器
> +(id)leftModelWith{
>    leftModel * model = [[[self alloc]]init](#)];
>   return model
>  }

#### UIImage初始化图片的方法
* imageNamed:系统会先检查系统缓存中是否有该名字的image,如果有的话,则直接返回,如果没有,则先加载图像到缓存,然后再返回.
* initWithContentsOfFile:系统不会检查缓存,而直接从文件系统中记载并返回.
* imageWithCGImage:scale:orientation 当scale= 1的时候图像为原始大小,orientation指定绘制图像的方向.

#### person的retainCount值
> Person * per = [Person alloc](#)init];
> self.person = per;
> 1或者2.看person是什么类型修饰的.
> alloc+1,assign+0,retain+1.
#### 下面这段代码有何问题
 @implementation Person
- (void)setAge:(int)newAge {
`      self.age = newAge;      
` }
 @end 
死循环
#### 这段代码有什么问题,如何修改
for (int i = 0; i \< someLargeNumber; i++) {
    NSString *string = @”Abc”;
    string = [string lowercaseString](#);
    string = [string stringByAppendingString:@"xyz"](#);
    NSLog(@“%@”, string);
}
 
加入自动释放池@autoreleasepool;
for (int i = 0; i \< someLargeNumber; i++) {
    @antoreleasepool {
        NSString *string = @”Abc”;
        string = [string lowercaseString](#);
        NSLog(@“%@”, string);
    }
}
#### 截取字符串"20 | http://www.baidu.com"中，"|"字符前面和后面的数据，分别输出它们。
 ["20 | http://www.baidu.com" componentSeparatedByString:@"|"](#);
#### 用obj-c 写一个冒泡排序.
NSMutableArray *ary = [@[@"1", @"2", @"3", @"4", @"6", @"5"](#) mutableCopy];

for (int i = 0; i \< ary.count - 1; i++) {
    
    for (int j = 0; j \< ary.count - i - 1; j++) {
        
        if ([ary[j](#) integerValue] \< [ary[j + 1](#) integerValue]) {
            
            [ary exchangeObjectAtIndex:j withObjectAtIndex:j + 1](#);
            
        }
        
    }
    
}

NSLog(@"%@", ary);

#### 简述对UIView，UIWindow和CALayer的理解
* UIWindow是应用的窗口,继承于UIResponder
* UIView继承于UIView,是创建窗口中的一个视图,可以响应交互事件.一个程序只有一个主window,可以有多个window
* CALayer图层,一个view可有多个图层,不可以响应事件

#### 写一个完整的代理,包括声明,实现
代理:遵守协议的对象.

@class MyView;

第一步:指定协议:(协议名:类名+Delegate)

@protocol MyViewDelegate \<NSObject\>

@required

-(void)changeViewBackgroudColor:(MyView *)view;

@optional

-(void)test;

@end

@interface MyView : UIView

第二步:指定代理

@property (nonatomic,assign)id\<MyView\> delegate;

@end

第三步:代理遵循协议.

第四步:代理实现协议里面的必须实现的方法和其他可选方法.

第五步:委托方通知代理开始执行方法.

#### 分析json.xml的区别,底层如何实现
* Json:键值对.数据小,不复杂.便于解析,有框架支持,适合轻量级传输.作为数据包个数传输的时候效率更高.

* xml:标签套内容.xml数据两较大,比较复杂.适合大数据量的传输.xml有丰富的编码工具,比如:Dom4j,JDom.解析方式有两种,一是通过文芳模型解析,另外一种遍历节点.

#### ViewController的didReceiveMemoryWarning是在什么时候被调用的
* 当应用程序的内存使用接近系统的最大内存使用时,应用会向系统发送内存警告,这时候系统会调用方法向所有ViewController发送内存警告
* 打开系统相机.
* 加载高清图片.
**默认操作:把里面没有用的对象进行释放.**
#### 面向对象的三大特征,简单介绍
* 封装:代码模块化,方便以后调用.
* 继承:子类继承父类的所有方法和属性.
* 多态:父类指针指向子类对象.

#### 用MRC实现set和get
- (void)setName:(NSString *)name{
	    
    if(_name != name){
        
        [_name retain](#);
        
        [_name release](#);
        
        _name = name;
        
    }
    
}

- (NSString *)name{
	    
    return [[_name retain](#)autorelease];
    
}

#### 简述NotificationCenter.KVC,KVO,Delegate?并说明它们之间的区别?
* NotificationCenter:消息中心.消息通知.

* KVC:利用键-值间接访问类中的某个属性.

`self setValue:@"123" forKeyPath:@"name";
`
`NSLog(@"%@",self valueForKeyPath:@"name");
`
* KVO:利用键-路径间接访问类中的某个属性,也就是观察者模式(KVO+通知中心).基于KVC和通知中心,观察的是实例变量.

* Delegate:用于多个类之间的传值.

#### 对MVC的理解,好处
* MVC:是一种架构.model:数据处理,view:视图显示,controller:逻辑控制,负责视图和模型之间的通信.

* 高类聚,低耦合,提高代码的复用性.

#### 监测键盘的弹出
通知.
[NSNotificationCenter defaultCenteraddObserver:self   selector:@selector()  name:UIKeyboardWillShowNotification  object:nil];
#### 介绍响应者链
* 当用户点击屏幕,能够产生响应的对象组成的链.
* 继承自NSResponder,响应者链能够中断.

#### 传值方式
通知,单例,代理,属性,block
#### NSString * test = [[NSData alloc](#) init],test在编译时和运行时分别是什么类型的对象
编译时是NSString,运行时是NSData
#### OC中对象的交互是如何实现的
消息机制
#### 目标-动作机制
Target - action
#### 什么是沙盒?沙盒里包含哪些文件,如何获取文件路径
* 沙盒:程序可操作的磁盘空间,系统为之开辟.
	 
* 包含了3个文件夹.
	 
1.Documents:存放一些比较重要的文件,但是放入Documents中的文件不能过大.
2.Library :是一个资源库,存储一些不太重要的数据.里面包含了两个子文件夹,Caches文件夹,用于缓存,
Preferences文件夹,系统偏好设置,用户对应用程序的设置,如密码.perferences路径无法找到,只能通过NSUserDefaults.
如:[NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES) firstObject](#);
#### 介绍一下XMPP
* 基于XML的点对点通讯协议,实现通讯功能.
* 优点:可以跨平台开发.
* 缺点:丢包,只能发文字(图片发送发的是链接).

#### 应用程序如何省电
获取请求不能过频.优化算法
#### 递归的使用

-(NSInteger)digui:(NSInteger)i{
    if (i\>0) {
        return i*[self digui:(i-1)](#);
    }else{
        return 1;
    }
}

#### NSArray 和 NSMutableArray 的区别?多线程下那个更安全
* NSArray: 不可变数组.
* NSMutableArray: 可变数组.
* 多线程下NSArray更安全.

#### isKindOfClass,isMemberOfClass作用分别是什么
* isKindOfClass是某个类的实例或者子类的实例
* isMemberOfClass是某个类的实例

#### 请分别写出SEL,id的意思
* SEL:选择器
* id:范类型
* OC中的对象就是C语言的指针
#### iPhone上,能被应用程序直接调用的系统程序是什么
* 能:相册,相机,通讯录,音乐.
* 不能:计算器,天气,日历,指南针

#### 以.mm为扩展名的文件里,可以包含哪些代码
C++,C,OC
#### 说说后台如何运行程序
* 在plist配置Application does not run in background设置NO(默认就是NO)的前提下.
* 添加required background modes,值是App registers for location updates和App plays auto or streams audio/video using AirPlay
#### sizeof和strlen的区别和联系
* sizeof:占用空间大小.
* strlen:字符串大小.

#### sprintf,strcpy,memcpy的功能?使用上要注意哪些地方?
* sprintf:将某些类型转换成字符串类型
* strcpy:拷贝字符串,会越界,'/0'
* memcpy:拷贝内存

#### 写一个代码片实现输入一个字符串"20130322152830",输出一个NSDate类型的对象,打印该对象输出2013-03-11 15:28:32
NSString * str = @"20130322152832";
NSDateFormatter * format = [[NSDateFormatter alloc](#)init];
format.dateFormat = @"yyyyMMddHHmmss";//设置格式 
NSLog(@"%@",[[format dateFromString:str](#) dateByAddingTimeInterval:8*60*60]);
#### 用变量a写出以下定义
a、一个整型数int a = 10
b、一个指向整型数的指针int *p = 10
c、一个指向指针的指针，它指向的指针是指向一个整型数int **p =10
d、一个有10个整型数的数组 int a[10](#)
e、一个有10个指针的数组，该指针是指向一个整型数的int *a[10](#)
f、一个指向有10个整型数数组的指针int *a = {1,2,3,4,5,6,7,8,9,10};
g、一个指向函数的指针，该函数有一个整型参数，并返回一个整型数
int *a(int  b){
    return b;
}
#### cocoa和 cocoa touch
* cocoa包含Foundation和AppKit框架，可用于开发Mac OS X系统的应用程序
* cocoa touch包含Foundation和UIKit框架，可用于开发iPhone OS 系统的应用程序
* Cocoa是Mac OS X的开发环境，cocoa Touch是 Iphone OS的开发环境

#### 网络从下往上分为几层
* 从下往上：物理层、数据链路层、网络层、传输层、会话层、表示层、应用层。
* IP 协议对应网络层，TCP 协议对应传输层，HTTP 协议对应于应用层。
* socket 则是对 TCP/IP协议的封装和应用。也可以说，TCP/IP协议是传输层协议，主要解决数据如何在网络中传输，而 HTTTP 是应用层协议，主要解决

#### 多线程的底层实现
线程：进程中一个特立独行的控制单元（路径）。多线程：一个进程至少有一个线程，即主线程。
①、Mach 是第一个以多线程方式处理任务的系统，因此多线程的底层实现机制就是基于 Mach 的线程。
②、开发中很少用到 Mach 级的线程，因为 Mach级的线程没有提供多线程的基本特征，线程之间是独立的。
④、开发中实现多线程的方案：
NSThread、GCD、NSOperationQueue.NSOperation
#### 线程之间怎么通信
①.performSelect:onThread:withObject:waitUntilDone:
②.NSMachPort
#### 网络图片问题中怎么解决一个相同的网络地址重复请求的问题
利用字典:图片地址为 key, 下载操作为 value
#### 用 NSOperation和 NSOperationQueue处理 A.B.C三个线程,要求执行完 A.B 后才能执行
//创建队列

NSOperationQueue * queue = [[NSOperationQueue alloc](#)init];

//创建三个操作

NSOperation * A = [NSBlockOperation blockOperationWithBlock:^{
](#)    
    NSLog{@"A"};
    
}];

NSOperation * B = [NSBlockOperation blockOperationWithBlock:^{
](#)    
    NSLog{@"B"};
    
}];

NSOperation * C = [NSBlockOperation blockOperationWithBlock:^{
](#)    
    NSLog{@"C"};
    
}];

// 添加依赖

[C addDependency:a](#);

[C addDependency:b](#);

//执行操作

[queue addOperation:a](#);

[queue addOperation:b](#);

[queue addOperation:c](#);

#### GCD内部怎么实现的
1. OS和 OSX 的核心是 XNU 内核, GCD是基于 XNU 内核实现的(是由苹果电脑发展起来的操作系统内核).
2. GCD 的 API 全部在 libdispatch 库中.
3. GCD 底层实现主要有 Dispatch Queue(管理 block)和 Dispatch Source(处理事件).

#### 怎么保证多人开发进行内存泄露检查
使用Analuze进行代码的静态分析，为避免麻烦，多人开发尽量使用ARC
#### 非自动内存管理情况下怎么做单例模式
* 创建一个单例对象的静态实例，并初始化为nil。
* 创建一个类的类工厂方法，当且仅当这个类的实例为nil时生成一个类的实例。
* 实现NScopying协议，覆盖allocWithZone：方法，确保用户在直接分配对象时，不会产生另一个对象。 
* 覆盖release、autorelease、retain、retainCount方法，确保单例的状态。

#### 对于类方法（静态方法）默认是autorelease的，所有类方法都会这样吗
系统自带的绝大数类方法返回的对象，都是经过autorelease













