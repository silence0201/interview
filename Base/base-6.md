 #基础面试题六
####strong / weak / unsafe_unretained 的区别
* weak只能修饰OC对象,使用weak不会使计数器加1,对象销毁时修饰的对象会指向nil
* strong等价与retain,能使计数器加1,且不能用来修饰数据类型
* unsafe_unretained等价与assign,可以用来修饰数据类型和OC对象,但是不会使计数器加1,且对象销毁时也不会将对象指向nil,容易造成野指针错误_

####如何为 Class 定义一个对外只读对内可读写的属性
在头文件中将属性定义为readonly,在.m文件中将属性重新定义为readwrite
####Objective-C 中，meta-class 指的是什么
meta-class 是 Class 对象的类,为这个Class类存储类方法,当一个类发送消息时,就去那个类对应的meta-class中查找那个消息,每个Class都有不同的meta-class,所有的meta-class都使用基类的meta-class(假如类继承NSObject,那么他所对应的meta-class也是NSObject)作为他们的类
####UIView 和 CALayer 之间的关系
* UIView显示在屏幕上归功于CALayer，通过调用drawRect方法来渲染自身的内容，调节CALayer属性可以调整UIView的外观，UIView继承自UIResponder，CALayer不可以响应用户事件
* UIView是iOS系统中界面元素的基础，所有的界面元素都继承自它。它内部是由Core Animation来实现的，它真正的绘图部分，是由一个叫CALayer(Core Animation Layer)的类来管理。UIView本身，更像是一个CALayer的管理器，访问它的根绘图和坐标有关的属性，如frame，bounds等，实际上内部都是访问它所在CALayer的相关属性
* UIView有个layer属性，可以返回它的主CALayer实例，UIView有一个layerClass方法，返回主layer所使用的类，UIView的子类，可以通过重载这个方法，来让UIView使用不同的CALayer来显示

####+￼UIView animateWithDuration:animations:completion:￼ 内部大概是如何实现的
animateWithDuration:这就等于创建一个定时器
animations:这是创建定时器需要实现的SEL
completion:是定时器结束以后的一个回调block
####什么时候会发生「隐式动画」
当改变CALayer的一个可做动画的属性，它并不能立刻在屏幕上体现出来.相反，它是从先前的值平滑过渡到新的值。这一切都是默认的行为，你不需要做额外的操作,这就是隐式动画
####frame 和 bounds 的区别是什么
* frame相对于父视图,是父视图坐标系下的位置和大小。bounds相对于自身,是自身坐标系下的位置和大小。
* frame以父控件的左上角为坐标原点，bounds以自身的左上角为坐标原点

####如何把一张大图缩小为1/4大小的缩略图
data = UIImageJPEGRepresentation(image, 0.25)
####category 和 extension 的区别
* category ：分类有名字，类扩展没i有分类名字，是一种特殊的分类
* extension ：分类只能扩展方法（属性仅仅是声明，并没真正实现），类扩展可以扩展属性、成员变量和方法
####define 和 const常量有什么区别
* define在预处理阶段进行替换，const常量在编译阶段使用
* 宏不做类型检查，仅仅进行替换，const常量有数据类型，会执行类型检查
* define不能调试，const常量可以调试
* define定义的常量在替换后运行过程中会不断地占用内存，而const定义的常量存储在数据段只有一份copy，效率更高
* define可以定义一些简单的函数，const不可以
####block和weak修饰符的区别
* __block不管是ARC还是MRC模式下都可以使用，可以修饰对象，也可以修饰基本数据类型
* __weak只能在ARC模式下使用，只能修饰对象（NSString），不能修饰基本数据类型
*  block修饰的对象可以在block中被重新赋值，weak修饰的对象不可以

####static关键字的作用
* 函数（方法）体内 static 变量的作用范围为该函数体，该变量的内存只被分配一次，因此其值在下次调用时仍维持上次的值；
* 在模块内的 static 全局变量可以被模块内所用函数访问，但不能被模块外其它函数访问；
* 在模块内的 static 函数只可被这一模块内的其它函数调用，这个函数的使用范围被限制在声明 它的模块内；
* 在类中的 static 成员变量属于整个类所拥有，对类的所有对象只有一份拷贝；
* 在类中的 static 成员函数属于整个类所拥有，这个函数不接收 this 指针，因而只能访问类的static 成员变量
####堆和栈的区别
*从管理方式来讲
1. 对于栈来讲，是由编译器自动管理，无需我们手工控制；
2. 对于堆来说，释放工作由程序员控制，容易产生内存泄露(memory leak)
* 从申请大小大小方面讲
1. 栈空间比较小
2. 堆空间比较大
* 从数据存储方面来讲
1.  栈空间中一般存储基本类型，对象的地址
2. 堆空间一般存放对象本身，block的copy等

####Objective-C使用什么机制管理对象内存
* MRC 手动引用计数
* ARC 自动引用计数,现在通常ARC
* 通过 retainCount 的机制来决定对象是否需要释放。 每次 runloop 的时候，都会检查对象的 retainCount，如果retainCount 为 0，说明该对象没有地方需要继续使用了，可以释放掉了

####ARC通过什么方式帮助开发者管理内存
通过编译器在编译的时候,插入类似内存管理的代码
####ARC是为了解决什么问题诞生的
* 首先解释ARC: automatic reference counting自动引用计数
* 了解MRC的缺点
1. 在MRC时代当我们要释放一个堆内存时，首先要确定指向这个堆空间的指针都被release了
2. 释放指针指向的堆空间，首先要确定哪些指针指向同一个堆，这些指针只能释放一次(MRC下即谁创建，谁释放，避免重复释放)
3. 模块化操作时，对象可能被多个模块创建和使用，不能确定最后由谁去释放
4. 多线程操作时，不确定哪个线程最后使用完毕
* 综上所述，MRC有诸多缺点，很容易造成内存泄露和坏内存的问题，这时苹果为尽量解决这个问题，从而诞生了ARC

####ARC下还会存在内存泄露吗
* 循环引用会导致内存泄露
* Objective-C对象与CoreFoundation对象进行桥接的时候如果管理不当也会造成内存泄露
* CoreFoundation中的对象不受ARC管理，需要开发者手动释放

####什么情况使用weak关键字，相比assign有什么不同
* 首先明白什么情况使用weak关键字？
1. 在ARC中,在有可能出现循环引用的时候,往往要通过让其中一端使用weak来解决,比如:delegate代理属性，代理属性也可使用assign
2. 自身已经对它进行一次强引用,没有必要再强引用一次,此时也会使用weak,自定义IBOutlet控件属性一般也使用weak；当然，也可以使用strong，但是建议使用weak
* weak 和 assign的不同点
1. weak策略在属性所指的对象遭到摧毁时，系统会将weak修饰的属性对象的指针指向nil，在OC给nil发消息是不会有什么问题的；如果使用assign策略在属性所指的对象遭到摧毁时，属性对象指针还指向原来的对象，由于对象已经被销毁，这时候就产生了野指针，如果这时候在给此对象发送消息，很容造成程序奔溃
2. assigin 可以用于修饰非OC对象,而weak必须用于OC对象

####@property 的本质是什么
@property其实就是在编译阶段由编译器自动帮我们生成ivar成员变量，getter方法，setter方法
ivar、getter、setter是如何生成并添加到这个类中的？
• 使用“自动合成”( autosynthesis)
• 这个过程由编译器在编译阶段执行自动合成，所以编辑器里看不到这些“合成方法”(synthesized method)的源代码
• 除了生成getter、setter方法之外，编译器还要自动向类中添加成员变量（在属性名前面加下划线，以此作为实例变量的名字）

####KVO内部实现原理
*  KVO是基于runtime机制实现的
* 当某个类的属性对象第一次被观察时，系统就会在运行期动态地创建该类的一个派生类，在这个派生类中重写基类中任何被观察属性的setter 方法。派生类在被重写的setter方法内实现真正的通知机制
* 如果原类为Person，那么生成的派生类名为NSKVONotifying_Person
* 每个类对象中都有一个isa指针指向当前类，当一个类对象的第一次被观察，那么系统会偷偷将isa指针指向动态生成的派生类，从而在给被监控属性赋值时执行的是派生类的setter方法
* 键值观察通知依赖于NSObject 的两个方法: willChangeValueForKey: 和 didChangevlueForKey:；在一个被观察属性发生改变之前， willChangeValueForKey: 一定会被调用，这就 会记录旧的值。而当改变发生后，didChangeValueForKey: 会被调用，继而 observeValueForKey:ofObject:change:context: 也会被调用。
* 补充：KVO的这套实现机制中苹果还偷偷重写了class方法，让我们误认为还是使用的当前类，从而达到隐藏生成的派生类

####KVC的keyPath中的集合运算符如何使用
* 必须用在集合对象上或普通对象的集合属性上
* 简单集合运算符有@avg， @count ， @max ， @min ，@sum
* 格式 @"@sum.age" 或 @"集合属性.@max.age"？？？

####KVC和KVO的keyPath一定是属性么
可以是成员变量










