# UI面试题二
#### 控制器的生命周期
 就是问的view的生命周期，下面已经按方法执行顺序进行了排序

// 自定义控制器view，这个方法只有实现了才会执行  
- (void)loadView  
	{  
	    self.view = [[UIView alloc] init];  
	    self.view.backgroundColor = [UIColor orangeColor](#);  
	}  
	// view是懒加载，只要view加载完毕就调用这个方法  
	- (void)viewDidLoad  
	{  
	    [super viewDidLoad](#);  
	    NSLog(@"%s",__func__);  
	}  
	// view即将显示  
	- (void)viewWillAppear:(BOOL)animated  
	{  
	    [super viewWillAppear:animated](#);  
	    NSLog(@"%s",__func__);  
	}  
	// view即将开始布局子控件  
	- (void)viewWillLayoutSubviews  
	{  
	    [super viewWillLayoutSubviews](#);  
	    NSLog(@"%s",__func__);  
	}  
	// view已经完成子控件的布局  
	- (void)viewDidLayoutSubviews  
	{  
	    [super viewDidLayoutSubviews](#);  
	    NSLog(@"%s",__func__);  
	}  
	// view已经出现  
	- (void)viewDidAppear:(BOOL)animated  
	{  
	    [super viewDidAppear:animated](#);  
	    NSLog(@"%s",__func__);  
	}  
	// view即将消失  
	- (void)viewWillDisappear:(BOOL)animated  
	{  
	    [super viewWillDisappear:animated](#);  
	    NSLog(@"%s",__func__);  
	}  
	// view已经消失  
	- (void)viewDidDisappear:(BOOL)animated  
	{  
	    [super viewDidDisappear:animated](#);  
	    NSLog(@"%s",__func__);  
	}  
	// 收到内存警告  
	- (void)didReceiveMemoryWarning  
	{  
	    [super didReceiveMemoryWarning](#);  
	    NSLog(@"%s",__func__);  
	}  
	// 方法已过期，即将销毁view  
	- (void)viewWillUnload  
	{  
	}  
	// 方法已过期，已经销毁view  
	- (void)viewDidUnload  
	{  
	}  
#### 你是怎么封装一个view的
* 可以通过纯代码或者xib的方式来封装子控件
* 建立一个跟view相关的模型，然后将模型数据传给view，通过模型上的数据给view的子控件赋值

/**  *  纯代码初始化控件时一定会走这个方法  
 */  
- (instancetype)initWithFrame:(CGRect)frame  
	{  
	    if(self = [super initWithFrame:frame](#))  
	    {  
	        [self setup](#);  
	    }  
	    return self;  
	}  
	/**  *  通过xib初始化控件时一定会走这个方法  
	 */  
	- (id)initWithCoder:(NSCoder *)aDecoder  
	{  
	    if(self = [super initWithCoder:aDecoder](#))  
	    {  
	        [self setup](#);  
	    }   
	    return self;  
	}  
	- (void)setup  
	{  
	    // 初始化代码  
	}  
#### 如何进行iOS6、7的适配
通过判断版本来控制，来执行响应的代码

* 功能适配：保证同一个功能在6、7上都能用
* UI适配：保证各自的显示风格
 
// iOS版本为7.0以上（包含7.0）
define iOS7 ([[UIDevice currentDevice](#).systemVersion doubleValue]\>=7.0)
#### 如何渲染UILabel的文字
通过NSAttributedString/NSMutableAttributedString（富文本）
#### UIScrollView的contentSize能否在viewDidLoad中设置
能
* 因为UIScrollView的内容尺寸是根据其内部的内容来决定的，所以是可以在viewDidLoad中设置的
* 补充：（这仅仅是一种特殊情况）
* 前提，控制器B是控制器A的一个子控制器，且控制器B的内容只在控制器A的view的部分区域中显示
*  假设控制器B的view中有一个UIScrollView这样一个子控件
* 如果此时在控制器B的viewDidLoad中设置UIScrollView的contentSize的话会导致不准确的问题
* 因为任何控制器的view在viewDidLoad的时候的尺寸都是不准确的，如果有子控件的尺寸依赖父控件的尺寸，在这个方法中设置会导致子控件的frame不准确，所以这时应该在下面的方法中设置子控件的尺寸

-(void)viewDidLayoutSubviews;
#### 触摸事件的传递
* 触摸事件的传递是从父控件传递到子控件
* 如果父控件不能接收触摸事件，那么子控件就不可能接收到触摸事件
* 不能接受触摸事件的四种情况
1. 不接收用户交互，即：userInteractionEnabled = NO
2. 隐藏，即：hidden = YES
3. 透明，即：alpha \<= 0.01
4. 未启用，即：enabled = NO
#### 事件响应者链
1. 如果当前view是控制器的view，那么就传递给控制器
2. 如果控制器不存在，则将其传递给它的父控件
3. 在视图层次结构的最顶层视图也不能处理接收到的事件或消息，则将事件或消息传递给UIWindow对象进行处理
4. 如果UIWindow对象也不处理，则将事件或消息传递给UIApplication对象
5. 如果UIApplication也不能处理该事件或消息，则将其丢弃
#####补充：如何判断上一个响应者
* 如果当前这个view是控制器的view，那么控制器就是上一个响应者
* 如果当前这个view不是控制器的view，那么父控件就是上一个响应者
#### 如何实现类似QQ的三角形头像
* Quartz2D
* 使用coreGraphics裁剪出一个三角形
