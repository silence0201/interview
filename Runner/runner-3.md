# 原理篇三
#### lldb（gdb）常用的调试命令
* po：打印对象，会调用对象description方法。是print-object的简写
* expr：可以在调试时动态执行指定表达式，并将结果打印出来，很有用的命令
* print：也是打印命令，需要指定类型
* bt：打印调用堆栈，是thread backtrace的简写，加all可打印所有thread的堆栈
* br l：是breakpoint list的简写
#### BAD_ACCESS在什么情况下出现
* 访问一个僵尸对象，访问僵尸对象的成员变量或者向其发消息
* 死循环
#### 如何调试BAD_ACCESS错误
* 设置全局断点快速定位问题代码所在行
#### 简述下Objective-C中调用方法的过程（runtime）
* Objective-C是动态语言，每个方法在运行时会被动态转为消息发送，即：objc_msgSend(receiver, selector)，整个过程介绍如下：
1. objc在向一个对象发送消息时，runtime库会根据对象的isa指针找到该对象实际所属的类
2.  然后在该类中的方法列表以及其父类方法列表中寻找方法运行
3.   如果，在最顶层的父类（一般也就NSObject）中依然找不到相应的方法时，程序在运行时会挂掉并抛出异常unrecognized selector sent to XXX
4.    但是在这之前，objc的运行时会给出三次拯救程序崩溃的机会，这三次拯救程序奔溃的说明见问题《什么时候会报unrecognized selector的异常》中的说明
* 补充说明：Runtime 铸就了Objective-C 是动态语言的特性，使得C语言具备了面向对象的特性，在程序运行期创建，检查，修改类、对象及其对应的方法，这些操作都可以使用runtime中的对应方法实现。
#### 什么是method swizzling（俗称黑魔法）
*  简单说就是进行方法交换
* 在Objective-C中调用一个方法，其实是向一个对象发送消息，查找消息的唯一依据是selector的名字。利用Objective-C的动态特性，可以实现在运行时偷换selector对应的方法实现，达到给方法挂钩的目的
* 每个类都有一个方法列表，存放着方法的名字和方法实现的映射关系，selector的本质其实就是方法名，IMP有点类似函数指针，指向具体的Method实现，通过selector就可以找到对应的IMP
 
* 	交换方法的几种实现方式 
*	利用 method_exchangeImplementations 交换两个方法的实现
*	利用 class_replaceMethod 替换方法的实现
*	利用 method_setImplementation 来直接设置某个方法的IMP 

#### objc中向一个nil对象发送消息将会发生什么
* 在Objective-C中向nil发送消息是完全有效的——只是在运行时不会有任何作用
1. 如果一个方法返回值是一个对象，那么发送给nil的消息将返回0(nil)
2.如果方法返回值为指针类型，其指针大小为小于或者等于sizeof(void*)
3. float，double，long double 或者long long的整型标量，发送给nil的消息将返回0
4.  如果方法返回值为结构体,发送给nil的消息将返回0。结构体中各个字段的值将都是0
5. 如果方法的返回值不是上述提到的几种情况，那么发送给nil的消息的返回值将是未定义的
* 具体原因分析
* objc是动态语言，每个方法在运行时会被动态转为消息发送，即：objc_msgSend(receiver, selector)
#### 三次拯救程序崩溃的机会
* Method resolution
1.  objc运行时会调用+resolveInstanceMethod:或者 +resolveClassMethod:，让你有机会提供一个函数实现。
2.   如果你添加了函数并返回 YES，那运行时系统就会重新启动一次消息发送的过程
3.   如果 resolve 方法返回 NO ，运行时就会移到下一步，消息转发
* Fast forwarding
1.  如果目标对象实现了-forwardingTargetForSelector:，Runtime 这时就会调用这个方法，给你把这个消息转发给其他对象的机会
2. 只要这个方法返回的不是nil和self，整个消息发送的过程就会被重启，当然发送的对象会变成你返回的那个对象。
3. 否则，就会继续Normal Fowarding。
4. 这里叫Fast，只是为了区别下一步的转发机制。因为这一步不会创建任何新的对象，但Normal forwarding转发会创建一个NSInvocation对象，相对Normal forwarding转发更快点，所以这里叫Fast forwarding
* Normal forwarding
1. 这一步是Runtime最后一次给你挽救的机会。
2. 首先它会发送-methodSignatureForSelector:消息获得函数的参数和返回值类型。
3. 如果-methodSignatureForSelector:返回nil，Runtime则会发出-doesNotRecognizeSelector:消息，程序这时也就挂掉了。
4. 如果返回了一个函数签名，Runtime就会创建一个NSInvocation对象并发送-forwardInvocation:消息给目标对象
#### HTTP协议中POST方法和GET方法有那些区别?
* GET用于向服务器请求数据，POST用于提交数据
* GET请求，请求参数拼接形式暴露在地址栏，而POST请求参数则放在请求体里面，因此GET请求不适合用于验证密码等操作
* GET请求的URL有长度限制，POST请求不会有长度限制
使用block时什么情况会发生引用循环，如何解决？
在block内如何修改block外部变量？
使用系统的某些block api（如UIView的block版本写动画时），是否也考虑循环引用问题？
* 系统的某些block api中，UIView的block版本写动画时不需要考虑，但也有一些api 需要考虑