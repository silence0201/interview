# 原理篇二
#### runloop和线程有什么关系？
* 每条线程都有唯一的一个RunLoop对象与之对应的
* 主线程的RunLoop是自动创建并启动
* 子线程的RunLoop需要手动创建
* 子线程的RunLoop创建步骤如下：
1. 在子线程中调用[NSRunLoop currentRunLoop](#)创建RunLoop对象（懒加载，只创建一次）
2. 获得RunLoop对象后要调用run方法来启动一个运行循环
 
####runloop的mode作用是什么？
* 用来控制一些特殊操作只能在指定模式下运行，一般可以通过指定操作的运行mode来控制执行时机，以提高用户体验
* 系统默认注册了5个Mode
1. kCFRunLoopDefaultMode：App的默认Mode，通常主线程是在这个Mode下运行，对应OC中的：NSDefaultRunLoopMode
2. UITrackingRunLoopMode：界面跟踪 Mode，用于 ScrollView 追踪触摸滑动，保证界面滑动时不受其他Mode影响
3. kCFRunLoopCommonModes:这是一个标记Mode，不是一种真正的Mode，事件可以运行在所有标有common modes标记的模式中，对应OC中的NSRunLoopCommonModes，带有common modes标记的模式有：UITrackingRunLoopMode和kCFRunLoopDefaultMode
4. UIInitializationRunLoopMode：在启动 App时进入的第一个 Mode，启动完成后就不再使用
5. GSEventReceiveRunLoopMode：接受系统事件的内部Mode，通常用不到
#### 以+scheduledTimerWithTimeInterval...的方式触发的timer，在滑动页面上的列表时，timer会暂定回调，为什么？如何解决？
* 这里强调一点：在主线程中以+scheduledTimerWithTimeInterval...的方式触发的timer默认是运行在NSDefaultRunLoopMode模式下的，当滑动页面上的列表时，进入了UITrackingRunLoopMode模式，这时候timer就会停止
* 可以修改timer的运行模式为NSRunLoopCommonModes，这样定时器就可以一直运行了
* 以下是我的笔记补充：
1. 在子线程中通过scheduledTimerWithTimeInterval:...方法来构建NSTimer
2. 方法内部已经创建NSTimer对象，并加入到RunLoop中，运行模式为NSDefaultRunLoopMode
3. 由于Mode有timer对象，所以RunLoop就开始监听定时器事件了，从而开始进入运行循环
4. 这个方法仅仅是创建RunLoop对象，并不会主动启动RunLoop，需要再调用run方法来启动
* 如果在主线程中通过scheduledTimerWithTimeInterval:...方法来构建NSTimer，就不需要主动启动RunLoop对象，因为主线程的RunLoop对象在程序运行起来就已经被启动了
#### 猜想runloop内部是如何实现的？
* 从字面意思看：运行循环、跑圈；
* 本质：内部就是do-while循环，在这个循环内部不断地处理各种事件(任务)，比如：Source、Timer、Observer；
* 每条线程都有唯一一个RunLoop对象与之对应，主线程的RunLoop默认已经启动，子线程的RunLoop需要手动启动；
* 每次RunLoop启动时，只能指定其中一个 Mode，这个Mode被称作 CurrentMode，如果需要切换Mode，只能退出Loop，再重新指定一个Mode进入，这样做主要是为了隔离不同Mode中的Source、Timer、Observer，让其互不影响；

#### 不手动指定autoreleasepool的前提下，一个autorealese对象在什么时刻释放？
* 分两种情况：手动干预释放时机、系统自动去释放
1. 手动干预释放时机：指定autoreleasepool就是所谓的：当前作用域大括号结束时就立即释放
2. 系统自动去释放：不手动指定autoreleasepool，Autorelease对象会在当前的 runloop 迭代结释放
#### 苹果为什么要废弃dispatch_get_current_queue
• 容易误用造成死锁
如何用GCD同步若干个异步调用？（如根据若干个url异步加载多张图片，然后在都下载完成后合成一张整图）
• 必须是并发队列才起作用
• 需求分析
○ 首先，分别异步执行2个耗时的操作
○ 其次，等2个异步操作都执行完毕后，再回到主线程执行一些操作
• 使用队列组实现上面的需求
 
// 创建队列组
dispatch_group_t group =  dispatch_group_create();
// 获取全局并发队列
dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
// 往队列组中添加耗时操作
dispatch_group_async(group, queue, ^{
    // 执行耗时的异步操作1
});
// 往队列组中添加耗时操作
dispatch_group_async(group, queue, ^{
    // 执行耗时的异步操作2
});
// 当并发队列组中的任务执行完毕后才会执行这里的代码
dispatch_group_notify(group, queue, ^{
    // 如果这里还有基于上面两个任务的结果继续执行一些代码，建议还是放到子线程中，等代码执行完毕后在回到主线程
    // 回到主线程
    dispatch_async(group, dispatch_get_main_queue(), ^{
        // 执行相关代码...
    });
});
#### dispatch_barrier_async的作用是什么
• 函数定义
 
dispatch_barrier_async(dispatch_queue_t queue, dispatch_block_t block);
• 必须是并发队列，要是串行队列，这个函数就没啥意义了
• 注意：这个函数的第一个参数queue不能是全局的并发队列
• 作用：在它前面的任务执行结束后它才执行，在它后面的任务等它执行完成后才会执
• 示例代码
 
-(void)barrier
{
    dispatch_queue_t queue = dispatch_queue_create("12342234", DISPATCH_QUEUE_CONCURRENT);
    dispatch_async(queue, ^{
        NSLog(@"----1-----%@", [NSThread currentThread](#));
    });
    dispatch_async(queue, ^{
        NSLog(@"----2-----%@", [NSThread currentThread](#));
    });
    // 在它前面的任务执行结束后它才执行，在它后面的任务等它执行完成后才会执行
    dispatch_barrier_async(queue, ^{
        NSLog(@"----barrier-----%@", [NSThread currentThread](#));
    });
    dispatch_async(queue, ^{
        NSLog(@"----3-----%@", [NSThread currentThread](#));
    });
    dispatch_async(queue, ^{
        NSLog(@"----4-----%@", [NSThread currentThread](#));
    });
}
#### 以下代码运行结果如何？
- (void)viewDidLoad
{
    [super viewDidLoad](#);
    NSLog(@"1");
    dispatch_sync(dispatch_get_main_queue(), ^{
        NSLog(@"2");
    });
    NSLog(@"3");
}
• 答案：主线程死锁

