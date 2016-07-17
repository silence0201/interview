# 基础面试五
#### block在ARC中和MRC中的方法有何区别？需要注意什么？
1. 对于没有引用外部变量的Block，无论在ARC还是MRC下，类型都是_NSGlobalBlock_,这种类型的block可以理解为一种全局的block,不需要考虑作用域的问题。同时，对它进行Copy和Retain操作也是无效的。
	 
2. 避免循环引用。
	 
根据isa指针，block一共有3种类型的block
 
_NSConcreteGlobalBlock 全局静态
 
_NSConcreteStackBlock 保存在栈中，出函数作用域就销毁
 
_NSConcreteMallocBlock 保存在堆中，retainCount == 0销毁_

#### 什么情况下会发生内存泄露和内存溢出
* 当程序在申请内存后，无法释放已经申请的内存空间（例如一个对象或者变量在用完后没有释放，这个对象就一直占用着内存），一次内存泄露可以忽略，但如果泄露过多的话，就会造成内存溢出。
* 当程序在申请内存时，但存入了更大的数据，出现内存溢出。

#### [NSArray arrayWithobject\<id\>](#)这个方法添加对象后，需要对这个数组进行释放操作吗
不需要，这个对象会被放到自动释放池中
#### 自动释放池如何实现
自动释放池以栈的形式实现，当你创建一个新的自动释放池时，它将被添加到栈顶，当一个对象收到发送autorelease消息时，它将添加到当前线程的处于栈顶的自动释放池中，当自动释放池被回收时，它们从栈中被删除并且会给池子里所有对象都做一次release操作
#### KVO内部实现原理
1. KVO是基于runtime机制实现的。
2. 当某个类的对象第一次被观察时，系统就会在运行期动态的创建该类的一个派生类，在这个派生类中重写基类中任何被观察属性的setter方法。
派生类在被重写setter方法中实现了真正的通知机制。（Person-\>NSKVONotification Person）

#### Foundation对象与Core Foundation对象有何区别
* Foundation对象是OC的，Core Foundation对象是C对象
* 数据类型之间的转换：
* ARC：_bridge_retained、_bridge_transfer
* 非ARC:_bridge_

#### 不用第三变量，交换AB的值
A=A+B
B=A-B
A=A-B
或者
A=A^B
B=A^B
A=A^B
#### runtime实现的机制是什么？怎么用，一般用于干嘛
运行时机制，runtime库里面包含了跟类、成员变量、方法相关的API，比如获取类里面的所有成员变量，为类动态添加成员变量、动态改变类的方法实现，为类动态添加新的方法等，需要导入\<objc/message.h\>\<objc/message.h\>
1. runtime,运行时机制，它是一套C语言库。
2. 实际上我们编写的所有OC代码，最终都是转换成为了runtime库的东西，比如类转换成了runtime库里面的结构体等数据类型，方法转换成了runtime库里面的C语言函数，平时调方法都是转成了objc_msgSend函数（所以说OC有个消息发送机制）
3. 因此，可以说runtime是OC的底层实现，是OC的幕后执行者。
4. 有了runtime库，能做什么呢？可以获取类里面的所有成员变量、为类动态的添加成员变量、动态的改变类的方法实现、为类动态添加新的方法等等。

#### 是否使用Core Text 或者 Core Image
Core Text
随意修改文本的样式
图文混排（纯C语言）
Core Image(滤镜处理)
能够调节图片的各种属性（对比度、色温、色差等）
#### NSNotification和KVO的区别和用法是什么？什么时候应该使用通知，什么时候应该使用KVO,他们的实现有何区别？如果用protocol和delegate来实现类似的功能可能吗？可能的话有何问题？不可能的话why？
* 通知比较灵活，一个通知能被多个对象接受，一个对象可以接受多个通知。
* 代理不交规范，但是代码较多（默认是一对一）
* KVO性能不好（底层会产生新的类），只能监听某个对象属性的变化，不推荐使用。

#### block内部的实现原理
Objective-C是对C语言的扩展，block的实现是基于指针和函数指针
#### 怎么解决缓存池满的问题
iOS中不存在缓存池满的情况，通常在对象需要创建时才创建，比如UITableView中一般只会创建刚开始在屏幕中的cell，之后都是从缓存池里取，不会再创建新对象
#### 控制器View的生命周期及相关函数是什么？你在开发中是如何使用的？
1. 首先判断控制器是否有视图，如果没有就调用loadView方法创建：通过storyBoard或者代码 
2. 随后调用viewDidLoad，可以进行下一步的初始化操作，只会被调用一次。
3. 在视图显示之前调用viewWillAppear,该函数可以多次调用。
4. 视图viewDidAppear
5. 在布局变化前后，调用viewWill/DidLayoutSubViews处理相关信息。

#### 有些图片加载比较慢怎么处理？你是怎么优化程序的性能的？
1. 图片下载放在异步线程。
2. 图片下载过程使用占位图片。
3. 如果图片比较大，可以使用多线程断点下载。

#### App需要加载大量数据，给服务器发送请求，但是服务器卡住了怎么办
设置请求超时，给用户提示请求超时，根据用户操作再次请求
#### SDWebImage具体如何实现
其实就是沙盒缓存机制，主要由三块组成：内存图片缓存，内存操作缓存，磁盘沙盒缓存。
1. 利用NSOperationQueue和NSOperation下载图片，还使用了GCD（解析GIF图片）。
2. 利用URL作为key，NSOperation作为value.
3. 利用URL作为key，UIImage作为value

#### AFNetWorking实现原理
基于NSURL.采用block的方法处理请求，直接返回的是json、XML数据。AFN直接操作对象是AFHTTPClient,是一个实现了NSCoding和NSCopying协议的NSObject子类。AFGTTPClient是一个封装了一系列操作方法的工具类。AFN默认没有封装同步请求，如果开发者需要使用同步请求，需要重写相关的方法（getPath:parameters:failure），对AFHTTPRequestOperation进行同步处理。
#### Extension 是什么？能列举几个常用的 Extension 么？
Extension是扩展,没有分类名字,是一种特殊的分类,类扩展可以扩展属性,成员变量和方法
常用的扩展是在.m文件中声明私有属性和方法,这里不知道还哪些,请大家补充
####  App Bundle 里面都有什么
* Info.plist:此文件包含了应用程序的配置信息.系统依赖此文件以获取应用程序的相关信息
* 可执行文件:此文件包含应用程序的入口和通过静态连接到应用程序target的代码
* 资源文件:图片,声音文件一类的
* 其他:可以嵌入定制的数据资源

#### iOS 的签名机制大概是怎样的
假设，我们有一个APP需要发布，为了防止中途篡改APP内容，保证APP的完整性，以及APP是由指定的私钥发的。首先，先将APP内容通过摘要算法，得到摘要，再用私钥对摘要进行加密得到密文，将源文本、密文、和私钥对应的公钥一并发布即可。那么如何验证呢？
验证方首先查看公钥是否是私钥方的，然后用公钥对密文进行解密得到摘要，将APP用同样的摘要算法得到摘要，两个摘要进行比对，如果相等那么一切正常。这个过程只要有一步出问题就视为无效。
#### iOS 7的多任务添加了哪两个新的 API
* 后台获取（Background Fetch):后台获取使用场景是用户打开应用之前就使app有机会执行代码来获取数据，刷新UI。这样在用户打开应用的时候，最新的内容将已然呈现在用户眼前，而省去了所有的加载过程。
* 推送唤醒（Remote Notifications):使用场景是使设备在接收到远端推送后让系统唤醒设备和我们的后台应用，并先执行一段代码来准备数据和UI，然后再提示用户有推送。这时用户如果解锁设备进入应用后将不会再有任何加载过程，新的内容将直接得到呈现

#### UIScrollView 大概是如何实现的，它是如何捕捉、响应手势的？
对UIScrollView的理解是frame就是他的contentSize,bounds就是他的可视范围,通过改变bounds从而达到让用户误以为在滚动,以下是一个简单的UIScrollView实现

#### +load 和 +initialize 的区别是什么
+(void)load;
当类对象被引入项目时, runtime 会向每一个类对象发送 load 消息
load 方法会在每一个类甚至分类被引入时仅调用一次,调用的顺序：父类优先于子类, 子类优先于分类
load 方法不会被类自动继承
+(void)initialize;
也是在第一次使用这个类的时候会调用这个方法

#### 如何让 Category 支持属性
使用runtime可以实现
#### NSOperation 相比于 GCD 有哪些优势
* 提供了在 GCD 中不那么容易复制的有用特性。
* 可以很方便的取消一个NSOperation的执行
* 可以更容易的添加任务的依赖关系
* 提供了任务的状态：isExecuteing, isFinished.














