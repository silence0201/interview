# 原理篇一
#### runtime怎么添加属性、方法等
• ivar表示成员变量
• class_addIvar
• class_addMethod
• class_addProperty
• class_addProtocol
• class_replaceProperty
#### 是否可以把比较耗时的操作放在NSNotificationCenter中
• 首先必须明确通知在哪个线程中发出，那么处理接受到通知的方法也在这个线程中调用
• 如果在异步线程发的通知，那么可以执行比较耗时的操作；
• 如果在主线程发的通知，那么就不可以执行比较耗时的操作
#### runtime 如何实现 weak 属性
• 首先要搞清楚weak属性的特点
 
weak策略表明该属性定义了一种“非拥有关系” (nonowning relationship)。
为这种属性设置新值时，设置方法既不保留新值，也不释放旧值。此特质同assign类似;
然而在属性所指的对象遭到摧毁时，属性值也会清空(nil out)
• 那么runtime如何实现weak变量的自动置nil？
 
runtime对注册的类，会进行布局，会将 weak 对象放入一个 hash 表中。
用 weak 指向的对象内存地址作为 key，当此对象的引用计数为0的时候会调用对象的 dealloc 方法，
假设 weak 指向的对象内存地址是a，那么就会以a为key，在这个 weak hash表中搜索，找到所有以a为key的 weak 对象，从而设置为 nil。
weak属性需要在dealloc中置nil么
• 在ARC环境无论是强指针还是弱指针都无需在 dealloc 设置为 nil ， ARC 会自动帮我们处理
• 即便是编译器不帮我们做这些，weak也不需要在dealloc中置nil
• 在属性所指的对象遭到摧毁时，属性值也会清空
 
// 模拟下weak的setter方法，大致如下
- (void)setObject:(NSObject *)object
{
    objc_setAssociatedObject(self, "object", object, OBJC_ASSOCIATION_ASSIGN);
    [object cyl_runAtDealloc:^{
](#)        _object = nil;
    }];
}

#### 一个Objective-C对象如何进行内存布局？（考虑有父类的情况）
• 所有父类的成员变量和自己的成员变量都会存放在该对象所对应的存储空间中
• 父类的方法和自己的方法都会缓存在类对象的方法缓存中，类方法是缓存在元类对象中
• 每一个对象内部都有一个isa指针,指向他的类对象,类对象中存放着本对象的如下信息
○ 对象方法列表
○ 成员变量的列表
○ 属性列表
#### 一个objc对象的isa的指针指向什么？有什么作用？
• 每一个对象内部都有一个isa指针，这个指针是指向它的真实类型
• 根据这个指针就能知道将来调用哪个类的方法
#### 下面的代码输出什么
 
@implementation Son : Father
- (id)init
{
    self = [super init](#);
    if (self) {
        NSLog(@"%@", NSStringFromClass([self class](#)));
        NSLog(@"%@", NSStringFromClass([super class](#)));
    }
    return self;
}
@end
• 答案：都输出 Son
• 这个题目主要是考察关于objc中对 self 和 super 的理解：
○ self 是类的隐藏参数，指向当前调用方法的这个类的实例。而 super 本质是一个编译器标示符，和 self 是指向的同一个消息接受者
○ 当使用 self 调用方法时，会从当前类的方法列表中开始找，如果没有，就从父类中再找；
○ 而当使用 super时，则从父类的方法列表中开始找。然后调用父类的这个方法
○ 调用[self class](#) 时，会转化成 objc_msgSend函数
 
id objc_msgSend(id self, SEL op, ...)
○ 调用 [super class](#)时，会转化成 objc_msgSendSuper函数
 
id objc_msgSendSuper(struct objc_super *super, SEL op, ...)
○ 第一个参数是 objc_super 这样一个结构体，其定义如下
 
struct objc_super {
    __unsafe_unretained id receiver;
    __unsafe_unretained Class super_class;
};
○ 第一个成员是 receiver, 类似于上面的 objc_msgSend函数第一个参数self
○ 第二个成员是记录当前类的父类是什么，告诉程序从父类中开始找方法，找到方法后，最后内部是使用 objc_msgSend(objc_super-\>receiver, @selector(class))去调用， 此时已经和[self class](#)调用相同了，故上述输出结果仍然返回 Son
○ objc Runtime开源代码对- (Class)class方法的实现
 
-(Class)class {
    return object_getClass(self);
}
#### runtime如何通过selector找到对应的IMP地址？（分别考虑类方法和实例方法）
• 每一个类对象中都一个对象方法列表（对象方法缓存）
• 类方法列表是存放在类对象中isa指针指向的元类对象中（类方法缓存）
• 方法列表中每个方法结构体中记录着方法的名称,方法实现,以及参数类型，其实selector本质就是方法名称,通过这个方法名称就可以在方法列表中找到对应的方法实现.
• 当我们发送一个消息给一个NSObject对象时，这条消息会在对象的类对象方法列表里查找
• 当我们发送一个消息给一个类时，这条消息会在类的Meta Class对象的方法列表里查找
objc中的类方法和实例方法有什么本质区别和联系
• 类方法：
○ 类方法是属于类对象的
○ 类方法只能通过类对象调用
○ 类方法中的self是类对象
○ 类方法可以调用其他的类方法
○ 类方法中不能访问成员变量
○ 类方法中不能直接调用对象方法
○ 类方法是存储在元类对象的方法缓存中
• 实例方法：
○ 实例方法是属于实例对象的
○ 实例方法只能通过实例对象调用
○ 实例方法中的self是实例对象
○ 实例方法中可以访问成员变量
○ 实例方法中直接调用实例方法
○ 实例方法中可以调用类方法(通过类名)
○ 实例方法是存放在类对象的方法缓存中
#### 使用runtime Associate方法关联的对象，需要在主对象dealloc的时候释放么
• 无论在MRC下还是ARC下均不需要
• 被关联的对象在生命周期内要比对象本身释放的晚很多，它们会在被 NSObject -dealloc 调用的object_dispose()方法中释放
• 补充：对象的内存销毁时间表，分四个步骤
 
1.调用 -release ：引用计数变为零
* 对象正在被销毁，生命周期即将结束.
* 不能再有新的 __weak 弱引用，否则将指向 nil.
* 调用 [self dealloc](#)
2. 父类调用 -dealloc
* 继承关系中最直接继承的父类再调用 -dealloc
* 如果是 MRC 代码 则会手动释放实例变量们（iVars）
* 继承关系中每一层的父类 都再调用 -dealloc
3. NSObject 调 -dealloc
* 只做一件事：调用 Objective-C runtime 中的 object_dispose() 方法
4. 调用 object_dispose()
* 为 C++ 的实例变量们（iVars）调用 destructors
* 为 ARC 状态下的 实例变量们（iVars） 调用 -release
* 解除所有使用 runtime Associate方法关联的对象
* 解除所有 __weak 引用
* 调用 free()
_objc_msgForward函数是做什么的？直接调用它将会发生什么？
• _objc_msgForward是IMP类型，用于消息转发的：当向一个对象发送一条消息，但它并没有实现的时候，_objc_msgForward会尝试做消息转发
• 直接调用_objc_msgForward是非常危险的事，这是把双刃刀，如果用不好会直接导致程序Crash，但是如果用得好，能做很多非常酷的事
• JSPatch就是直接调用_objc_msgForward来实现其核心功能的
• 详细解说参见这里的第一个问题解答
能否向编译后得到的类中增加实例变量？能否向运行时创建的类中添加实例变量？为什么？
• 不能向编译后得到的类中增加实例变量；
• 能向运行时创建的类中添加实例变量；
• 分析如下：
○ 因为编译后的类已经注册在runtime中，类结构体中的objc_ivar_list 实例变量的链表和instance_size实例变量的内存大小已经确定，同时runtime 会调用class_setIvarLayout 或 class_setWeakIvarLayout来处理strong weak引用，所以不能向存在的类中添加实例变量
○ 运行时创建的类是可以添加实例变量，调用 class_addIvar函数，但是得在调用objc_allocateClassPair之后，objc_registerClassPair之前，原因同上。

