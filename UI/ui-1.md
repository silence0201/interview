# UI面试题一
#### Size Classes
对屏幕进行分类
#### UIView和CALayer是什么关系
* UIView显示在屏幕上归功于CALayer，通过调用drawRect方法来渲染自身的内容，调节CALayer属性可以调整UIView的外观，UIView继承自UIResponder，比起CALayer可以响应用户事件，Xcode6之后可以方便的通过视图调试功能查看图层之间的关系
* UIView是iOS系统中界面元素的基础，所有的界面元素都继承自它。它内部是由Core Animation来实现的，它真正的绘图部分，是由一个叫CALayer(Core Animation Layer)的类来管理。UIView本身，更像是一个CALayer的管理器，访问它的跟绘图和坐标有关的属性，如frame，bounds等，实际上内部都是访问它所在CALayer的相关属性
* UIView有个layer属性，可以返回它的主CALayer实例，UIView有一个layerClass方法，返回主layer所使用的类，UIView的子类，可以通过重载这个方法，来让UIView使用不同的CALayer来显示

#### IBOutlet连出来的视图属性为什么可以被设置成weak
因为父控件的subViews数组已经对它有一个强引用
#### IB中User Defined Runtime Attributes如何使用？
* User Defined Runtime Attributes是一个不被看重但功能非常强大的的特性，它能够通过KVC的方式配置一些你在interface builder中不能配置的属性
* 当你希望在IB中作尽可能多得事情，这个特性能够帮助你编写更加轻量级的viewcontroller
#### pushViewController和presentViewController有什么区别
* 两者都是在多个试图控制器间跳转的函数
* presentViewController提供的是一个模态视图控制器(modal)
* pushViewController提供一个栈控制器数组，push/pop
#### 请简述UITableView的复用机制
* 每次创建cell的时候通过dequeueReusableCellWithIdentifier:方法创建cell，它先到缓存池中找指定标识的cell，如果没有就直接返回nil
* 如果没有找到指定标识的cell，那么会通过initWithStyle:reuseIdentifier:创建一个cell
* 当cell离开界面就会被放到缓存池中，以供下次复用
####如何高性能的给 UIImageView 加个圆角?
不好的解决方案  
使用下面的方式会强制Core Animation提前渲染屏幕的离屏绘制, 而离屏绘制就会给性能带来负面影响，会有卡顿的现象出现  

self.view.layer.cornerRadius = 5;
self.view.layer.masksToBounds = YES;
正确的解决方案：使用绘图技术

- (UIImage *)circleImage  
	{  
	    // NO代表透明  
	    UIGraphicsBeginImageContextWithOptions(self.size, NO, 0.0);  
	    // 获得上下文  
	    CGContextRef ctx = UIGraphicsGetCurrentContext();  
	    // 添加一个圆  
	    CGRect rect = CGRectMake(0, 0, self.size.width, self.size.height);  
	    CGContextAddEllipseInRect(ctx, rect);  
	    // 裁剪  
	    CGContextClip(ctx);  
	    // 将图片画上去  
	    [self drawInRect:rect](#);  
	    UIImage *image = UIGraphicsGetImageFromCurrentImageContext();  
	    // 关闭上下文  
	    UIGraphicsEndImageContext();  
	    return image;  
	}  
* 还有一种方案：使用了贝塞尔曲线"切割"个这个图片, 给UIImageView 添加了的圆角，其实也是通过绘图技术来实现的

	UIImageView *imageView = [[UIImageView alloc](#)   initWithFrame:CGRectMake(0, 0, 100, 100)];  
	imageView.center = CGPointMake(200, 300);  
	UIImage *anotherImage = [UIImage imageNamed:@"image"](#);  
	UIGraphicsBeginImageContextWithOptions(imageView.bounds.size, NO, 1.0);  
	[[UIBezierPath bezierPathWithRoundedRect:imageView.bounds  
	](#)                            cornerRadius:50] addClip];  
	[anotherImage drawInRect:imageView.bounds](#);  
	imageView.image = UIGraphicsGetImageFromCurrentImageContext();  
	UIGraphicsEndImageContext();  
[self.view addSubview:imageView](#);  

#### 使用drawRect有什么影响
* drawRect方法依赖Core Graphics框架来进行自定义的绘制
* 缺点：它处理touch事件时每次按钮被点击后，都会用setNeddsDisplay进行强制重绘；而且不止一次，每次单点事件触发两次执行。这样的话从性能的角度来说，对CPU和内存来说都是欠佳的。特别是如果在我们的界面上有多个这样的UIButton实例，那就会很糟糕了
* 这个方法的调用机制也是非常特别. 当你调用 setNeedsDisplay 方法时, UIKit 将会把当前图层标记为dirty,但还是会显示原来的内容,直到下一次的视图渲染周期,才会将标记为 dirty 的图层重新建立Core Graphics上下文,然后将内存中的数据恢复出来, 再使用 CGContextRef 进行绘制
#### 描述下SDWebImage里面给UIImageView加载图片的逻辑
* SDWebImage 中为 UIImageView 提供了一个分类UIImageView+WebCache.h, 这个分类中有一个最常用的接口sd_setImageWithURL:placeholderImage:，会在真实图片出现前会先显示占位图片，当真实图片被加载出来后在替换占位图片
* 加载图片的过程大致如下：
1. 首先会在 SDWebImageCache 中寻找图片是否有对应的缓存, 它会以url 作为数据的索引先在内存中寻找是否有对应的缓存
2. 如果缓存未找到就会利用通过MD5处理过的key来继续在磁盘中查询对应的数据, 如果找到了, 就会把磁盘中的数据加载到内存中，并将图片显示出来
3. 如果在内存和磁盘缓存中都没有找到，就会向远程服务器发送请求，开始下载图片
4. 下载后的图片会加入缓存中，并写入磁盘中
5. 整个获取图片的过程都是在子线程中执行，获取到图片后回到主线程将图片显示出来


