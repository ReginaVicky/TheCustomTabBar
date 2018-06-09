# 自定义TabBar

## 基本自带TabBar来实现自定义

* 在iOS原生的tabBar中，能够实现按钮的点击事件，能够实现视图控制器的切换等，但是在实际工程中，对于tabBar的要求的功能往往是系统自己实现不了的，所以我们这里就需要用到自定义的tabBar了。
* demo：[传送门](https://github.com/ReginaVicky/TheCustomTabBar)

### 目标
![image](https://upload-images.jianshu.io/upload_images/1197643-f726ef61ac18a267.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/598)
### 分析
![image](https://upload-images.jianshu.io/upload_images/1197643-fba5bea57441c211.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

### 实现
#### 1.建立新的项目，并去掉原有的
* ViewController.h 
* ViewController.m 
* Main.storyboard 
* LaunchScreen.storyboard
* 注意：要去掉info里面的main storyboard base name；

#### 2.创建窗口、设置根控制器、显示窗口

AppDelegate.m文件
```
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    
    //创建窗口
    self.window = [[UIWindow alloc] init];
    self.window.frame = [UIScreen mainScreen].bounds;
    
    //设置根控制器
    self.window.rootViewController = [[ReginaVickyTabBarController alloc] init];
    
    //显示窗口
    [self.window makeKeyAndVisible];
    
    return YES;
}
```

#### 3.自定义标签控制器(继承自UITabBarController)和创建导航栏控制器（继承自UINavigationController）

* 创建：ReginaVickyTabBarController.h 和 ReginaVickyTabBarController.m(继承自UITabBarController)
* 创建：ReginaVickyNavigationController.h和#ReginaVickyNavigationController.m（继承自UINavigationController）
* 添加四个子控制器
    - ReginaVickyTabBarController.m文件中

```
/**
 *  添加子控制器
 */
- (void)setupChildViewControllers
{
    [self setupOneChildViewController:[[XMGNavigationController alloc] initWithRootViewController:[[XMGEssenceViewController alloc] init]] title:@"精华" image:@"tabBar_essence_icon" selectedImage:@"tabBar_essence_click_icon"];
    
    [self setupOneChildViewController:[[XMGNavigationController alloc] initWithRootViewController:[[XMGNewViewController alloc] init]] title:@"新帖" image:@"tabBar_new_icon" selectedImage:@"tabBar_new_click_icon"];
    
    [self setupOneChildViewController:[[XMGNavigationController alloc] initWithRootViewController:[[XMGFollowViewController alloc] init]] title:@"关注" image:@"tabBar_friendTrends_icon" selectedImage:@"tabBar_friendTrends_click_icon"];
    
    [self setupOneChildViewController:[[XMGNavigationController alloc] initWithRootViewController:[[XMGMeViewController alloc] init]] title:@"我" image:@"tabBar_me_icon" selectedImage:@"tabBar_me_click_icon"];
}

/**
 *  初始化一个子控制器
 *
 *  @param vc            子控制器
 *  @param title         标题
 *  @param image         图标
 *  @param selectedImage 选中的图标
 */
- (void)setupOneChildViewController:(UIViewController *)vc title:(NSString *)title image:(NSString *)image selectedImage:(NSString *)selectedImage
{
    vc.tabBarItem.title = title;
    if (image.length) { // 图片名有具体值
        vc.tabBarItem.image = [UIImage imageNamed:image];
        vc.tabBarItem.selectedImage = [UIImage imageNamed:selectedImage];
    }
    [self addChildViewController:vc];
}
```

#### 4.自定义类ReginaVickyTabBar(继承自UITabBar)

* 声明中间的按钮
* 创建ReginaVickyTabBar.h 和 ReginaVickyTabBar.m(继承自UITabBar)

ReginaVickyTabBar.m文件中


```
/** 中间的发布按钮 */
@property (nonatomic, weak) UIButton *publishButton;
@end

@implementation ReginaVickyTabBar

#pragma mark - 懒加载
/** 发布按钮 */
- (UIButton *)publishButton
{
    if (!_publishButton) {
        UIButton *publishButton = [UIButton buttonWithType:UIButtonTypeCustom];
        [publishButton setImage:[UIImage imageNamed:@"tabBar_publish_icon"] forState:UIControlStateNormal];
        [publishButton setImage:[UIImage imageNamed:@"tabBar_publish_click_icon"] forState:UIControlStateHighlighted];
        [publishButton addTarget:self action:@selector(publishClick) forControlEvents:UIControlEventTouchUpInside];
        [self addSubview:publishButton];
        _publishButton = publishButton;
    }
    return _publishButton;
}

#pragma mark - 初始化
- (instancetype)initWithFrame:(CGRect)frame
{
    if (self = [super initWithFrame:frame]) {
        self.backgroundImage = [UIImage imageNamed:@"tabbar-light"];
    }
    return self;
}

/**
 *  布局子控件
 */
- (void)layoutSubviews
{
    [super layoutSubviews];
    /**** 按钮的尺寸 ****/
    CGFloat buttonW = self.reginavicky_width / 5;
    CGFloat buttonH = self.reginavicky_height;
    CGFloat buttonY = 0;
    // 按钮索引
    int buttonIndex = 0;
    
    for (UIView *subview in self.subviews) {
        // 过滤掉非UITabBarButton
        if (subview.class != NSClassFromString(@"UITabBarButton")) continue;
        
        // 设置frame
        CGFloat buttonX = buttonIndex * buttonW;
        if (buttonIndex >= 2) { // 右边的2个UITabBarButton
            buttonX += buttonW;
        }
        subview.frame = CGRectMake(buttonX, buttonY, buttonW, buttonH);
        
        // 增加索引
        buttonIndex++;
    }
    
    /**** 设置中间的发布按钮的frame ****/
    self.publishButton.reginavicky_width = buttonW;
    self.publishButton.reginavicky_height = buttonH;
    self.publishButton.reginavicky_centerX = self.reginavicky_width * 0.5;
    self.publishButton.reginavicky_centerY = self.reginavicky_height * 0.5;
}
#pragma mark - 监听
- (void)publishClick
{
    ReginaVickyLogFunc
    
}
```

#### 5. 设置TabBarItem文字的属性和更替TabBar

ReginaVickyTabBarController.m文件中


```
/**
 *  设置所有UITabBarItem的文字属性
 */
- (void)setupItemTitleTextAttributes
{
    UITabBarItem *item = [UITabBarItem appearance];
    // 普通状态下的文字属性
    NSMutableDictionary *normalAttrs = [NSMutableDictionary dictionary];
    normalAttrs[NSFontAttributeName] = [UIFont systemFontOfSize:14];
    normalAttrs[NSForegroundColorAttributeName] = [UIColor grayColor];
    [item setTitleTextAttributes:normalAttrs forState:UIControlStateNormal];
    // 选中状态下的文字属性
    NSMutableDictionary *selectedAttrs = [NSMutableDictionary dictionary];
    selectedAttrs[NSForegroundColorAttributeName] = [UIColor darkGrayColor];
    [item setTitleTextAttributes:normalAttrs forState:UIControlStateSelected];
}
```


```
/**
 *  更换TabBar
 */
- (void)setupTabBar
{
    [self setValue:[[ReginaVickyTabBar alloc] init] forKeyPath:@"tabBar"];
}
```


## 自定义中间有凸起按钮的TabBar
### 目标
![image](https://upload-images.jianshu.io/upload_images/1197643-1e32faee4ffe343d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/596)
### 分析
* 基于自带tabbar自定义tabbar
* 主要处理中间凸起的按钮，大小，位置的设定
* 需要注意的是，不要忘记处理超出部分点击失效的问题
### 实现

#### 1.建立新的项目，并去掉原有的
* ViewController.h 
* ViewController.m 
* Main.storyboard 
* LaunchScreen.storyboard
* 注意：要去掉info里面的main storyboard base name；

#### 2.创建窗口、设置根控制器、显示窗口

AppDelegate.m文件
```
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    
    //创建窗口
    self.window = [[UIWindow alloc] init];
    self.window.frame = [UIScreen mainScreen].bounds;
    
    //设置根控制器
    self.window.rootViewController = [[ReginaVickyTabBarController alloc] init];
    
    //显示窗口
    [self.window makeKeyAndVisible];
    
    return YES;
}
```

#### 3.自定义标签控制器(继承自UITabBarController)和创建导航栏控制器（继承自UINavigationController）

* 创建：ReginaVickyTabBarController.h 和 ReginaVickyTabBarController.m(继承自UITabBarController)
* 创建：ReginaVickyNavigationController.h和#ReginaVickyNavigationController.m（继承自UINavigationController）
* 添加四个子控制器
    - ReginaVickyTabBarController.m文件中

```
/**
 *  添加子控制器
 */
- (void)setupChildViewControllers
{
    [self setupOneChildViewController:[[ReginaVickyNavigationController alloc] initWithRootViewController:[[ReginaVickyHomeViewController alloc] init]] title:@"首页" image:@"home_normal" selectedImage:@"home_highlight"];
    
    [self setupOneChildViewController:[[ReginaVickyNavigationController alloc] initWithRootViewController:[[ReginaVickyMyCityViewController alloc] init]] title:@"同城" image:@"mycity_normal" selectedImage:@"mycity_highlight"];
    
    
    [self setupOneChildViewController:[[ReginaVickyNavigationController alloc] initWithRootViewController:[[ReginaVickyMessageViewController alloc] init]] title:@"消息" image:@"message_normal" selectedImage:@"message_highlight"];
    
    [self setupOneChildViewController:[[ReginaVickyNavigationController alloc] initWithRootViewController:[[ReginaVickyMineViewController alloc] init]] title:@"我的" image:@"account_normal" selectedImage:@"account_highlight"];
}

/**
 *  初始化一个子控制器
 *
 *  @param vc            子控制器
 *  @param title         标题
 *  @param image         图标
 *  @param selectedImage 选中的图标
 */
- (void)setupOneChildViewController:(UIViewController *)vc title:(NSString *)title image:(NSString *)image selectedImage:(NSString *)selectedImage
{
    vc.tabBarItem.title = title;
    if (image.length) { // 图片名有具体值
        vc.tabBarItem.image = [UIImage imageNamed:image];
        vc.tabBarItem.selectedImage = [[UIImage imageNamed:selectedImage] imageWithRenderingMode:UIImageRenderingModeAlwaysOriginal];
        //这句话的意思是声明这张图片按照原始的样子显示出来，不要自动渲染成其他颜色
        //vc.tabBarItem.selectedImage = [UIImage imageNamed:selectedImage];
    }
    [self addChildViewController:vc];
}
```

#### 4.自定义类ReginaVickyTabBar(继承自UITabBar)

* 声明中间的按钮
* 创建ReginaVickyTabBar.h 和 ReginaVickyTabBar.m(继承自UITabBar)

ReginaVickyTabBar.m文件中

```
/** 发布按钮 */
- (UIButton *)publishButton
{
    if (!_publishButton) {
        UIButton * publishButton = [UIButton buttonWithType:UIButtonTypeCustom];
        [publishButton setImage:[UIImage imageNamed:@"post_normal"] forState:UIControlStateNormal];
        [publishButton setTitle:@"发布" forState:UIControlStateNormal];
        [publishButton setTitleColor:[UIColor blackColor] forState:UIControlStateNormal];
        
        publishButton.titleLabel.font = [UIFont systemFontOfSize:10];
        publishButton.titleEdgeInsets = UIEdgeInsetsMake(85, -60, 0, 0);
        [publishButton setImage:[UIImage imageNamed:@"post_normal"] forState:UIControlStateHighlighted];
        [publishButton addTarget:self action:@selector(publishClick) forControlEvents:UIControlEventTouchUpInside];
        [self addSubview:publishButton];
        _publishButton = publishButton;
        
    }
    
    return _publishButton;
}

#pragma mark - 初始化
- (instancetype)initWithFrame:(CGRect)frame
{
    if (self = [super initWithFrame:frame]) {
        //self.barTintColor =
        //self.backgroundImage = [UIImage imageNamed:@"tabbar-light"];
        //[[UITabBar appearance] setTintColor:[UIColor redColor]];
    }
    return self;
}

/**
 *  布局子控件
 */
- (void)layoutSubviews
{
    [super layoutSubviews];
    /**** 按钮的尺寸 ****/
    CGFloat buttonW = self.reginavicky_width / 5;
    CGFloat buttonH = self.reginavicky_height;
    CGFloat buttonY = 0;
    // 按钮索引
    int buttonIndex = 0;
    
    for (UIView *subview in self.subviews) {
        // 过滤掉非UITabBarButton
        if (subview.class != NSClassFromString(@"UITabBarButton")) continue;
        
        // 设置frame
        CGFloat buttonX = buttonIndex * buttonW;
        if (buttonIndex >= 2) { // 右边的2个UITabBarButton
            buttonX += buttonW;
        }
        subview.frame = CGRectMake(buttonX, buttonY, buttonW, buttonH);
        
        // 增加索引
        buttonIndex++;
    }
    
    /**** 设置中间的发布按钮的frame ****/
    
    UIImage *normalImage = [UIImage imageNamed:@"post_normal"];
    self.publishButton.reginavicky_width = normalImage.size.width ;
    self.publishButton.reginavicky_height = normalImage.size.height ;
    self.publishButton.reginavicky_centerX = self.reginavicky_width * 0.5;
    self.publishButton.reginavicky_centerY = 0;
    
    
}
```

#### 5. 由于中间按钮凸出去一块，所以需要处理超出区域点击无效的问题

ReginaVickyTabBar.m文件中

```
- (UIView *)hitTest:(CGPoint)point withEvent:(UIEvent *)event{
    if (self.hidden){
        return [super hitTest:point withEvent:event];
    }else {
        //转换坐标
        CGPoint tempPoint = [self.publishButton convertPoint:point fromView:self];
        //判断点击的点是否在按钮区域内
        if (CGRectContainsPoint(self.publishButton.bounds, tempPoint)){
            //返回按钮
            return _publishButton;
        }else {
            return [super hitTest:point withEvent:event];
        }
    }
```

#### 6. 设置TabBarItem文字的属性和更替TabBar

ReginaVickyTabBarController.m文件中

```
/**
 *  设置所有UITabBarItem的文字属性
 */
- (void)setupItemTitleTextAttributes
{
    UITabBarItem *item = [UITabBarItem appearance];
    // 普通状态下的文字属性
    NSMutableDictionary *normalAttrs = [NSMutableDictionary dictionary];
    normalAttrs[NSFontAttributeName] = [UIFont systemFontOfSize:14];
    normalAttrs[NSForegroundColorAttributeName] = [UIColor grayColor];
    [item setTitleTextAttributes:normalAttrs forState:UIControlStateNormal];
    // 选中状态下的文字属性
    NSMutableDictionary *selectedAttrs = [NSMutableDictionary dictionary];
    selectedAttrs[NSForegroundColorAttributeName] = [UIColor darkGrayColor];
    [item setTitleTextAttributes:normalAttrs forState:UIControlStateSelected];
}
```

```
/**
 *  更换TabBar
 */
- (void)setupTabBar
{
    [self setValue:[[ReginaVickyTabBar alloc] init] forKeyPath:@"tabBar"];
    
}
```



## 完全自定义TabBar
### 目标
![image](https://upload-images.jianshu.io/upload_images/1197643-0f6f974ee3a95d09.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/596)
### 分析
* tabbar的文字和图片是文字在上面，图片在下面，需要自己设定
* tabbar的透明度设定的是透明，修改tabbar上面的分隔线颜色
* Navigation的透明度设为透明，隐藏下面的分割线
### 实现
#### 1.建立新的项目，并去掉原有的
* ViewController.h 
* ViewController.m 
* Main.storyboard 
* LaunchScreen.storyboard
* 注意：要去掉info里面的main storyboard base name；

#### 2.创建窗口、设置根控制器、显示窗口

AppDelegate.m文件
```
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    
    //创建窗口
    self.window = [[UIWindow alloc] init];
    self.window.frame = [UIScreen mainScreen].bounds;
    
    //设置根控制器
    self.window.rootViewController = [[ReginaVickyTabBarController alloc] init];
    
    //显示窗口
    [self.window makeKeyAndVisible];
    
    return YES;
}
```

#### 3.自定义标签控制器(继承自UITabBarController)和创建导航栏控制器（继承自UINavigationController）

* 创建：ReginaVickyTabBarController.h 和 ReginaVickyTabBarController.m(继承自UITabBarController)
* 创建：ReginaVickyNavigationController.h和#ReginaVickyNavigationController.m（继承自UINavigationController）
* 在初始化子控件时，对item的文字和图片进行位置的设置

```
[vc.tabBarItem setTitlePositionAdjustment:UIOffsetMake(0,-15)];
```

```
vc.tabBarItem.imageInsets = UIEdgeInsetsMake(25, 0, -25, 0);
```


* 添加四个子控制器
    - ReginaVickyTabBarController.m文件中

```
/**
 *  添加子控制器
 */
- (void)setupChildViewControllers
{
    
    [self setupOneChildViewController:[[ReginaVickyNavigationController alloc] initWithRootViewController:[[ReginaVickyHomeViewController alloc] init]] title:@"首页" image:@"tabbartouming" selectedImage:@"tabbarxi"];
    
    [self setupOneChildViewController:[[ReginaVickyFollowViewController alloc] init] title:@"关注" image:@"tabbartouming" selectedImage:@"tabbarxi"];
    
    
    
    [self setupOneChildViewController:[[ReginaVickyNavigationController alloc] initWithRootViewController:[[ReginaVickyMessageViewController alloc] init]] title:@"消息" image:@"tabbartouming" selectedImage:@"tabbarxi"];
    
    [self setupOneChildViewController:[[ReginaVickyMeViewController alloc] init] title:@"我" image:@"tabbartouming" selectedImage:@"tabbarxi"];

}




/**
 *  初始化一个子控制器
 *
 *  @param vc            子控制器
 *  @param title         标题
 *  @param image         图标
 *  @param selectedImage 选中的图标
 */
- (void)setupOneChildViewController:(UIViewController *)vc title:(NSString *)title image:(NSString *)image selectedImage:(NSString *)selectedImage
{
    vc.tabBarItem.title = title;
    [vc.tabBarItem setTitlePositionAdjustment:UIOffsetMake(0,-15)];
    if (image.length) { // 图片名有具体值
        vc.tabBarItem.image = [UIImage imageNamed:image];
        vc.tabBarItem.selectedImage = [[UIImage imageNamed:selectedImage] imageWithRenderingMode:UIImageRenderingModeAlwaysOriginal];
        //这句话的意思是声明这张图片按照原始的样子显示出来，不要自动渲染成其他颜色
        vc.tabBarItem.imageInsets = UIEdgeInsetsMake(25, 0, -25, 0);
        //vc.tabBarItem.selectedImage = [UIImage imageNamed:selectedImage];
    }
    [self addChildViewController:vc];
}
```

#### 4. 自定义类ReginaVickyTabBar(继承自UITabBar)

* 声明中间的按钮
* 创建ReginaVickyTabBar.h 和 ReginaVickyTabBar.m(继承自UITabBar)

ReginaVickyTabBar.m文件中

```
/** 发布按钮 */
- (UIButton *)publishButton
{
    if (!_publishButton) {
        UIButton * publishButton = [UIButton buttonWithType:UIButtonTypeCustom];
        [publishButton setImage:[UIImage imageNamed:@"tabbarpulishbutton"] forState:UIControlStateNormal];
        [publishButton setImage:[UIImage imageNamed:@"tabbarpulishbutton"] forState:UIControlStateHighlighted];
        [publishButton addTarget:self action:@selector(publishClick) forControlEvents:UIControlEventTouchUpInside];
        [self addSubview:publishButton];
        _publishButton = publishButton;
        
    }
    
    return _publishButton;
}

#pragma mark - 初始化
- (instancetype)initWithFrame:(CGRect)frame
{
    
    if (self = [super initWithFrame:frame]) {
        
        
        
    }
    return self;
}

/**
 *  布局子控件
 */
- (void)layoutSubviews
{
    [super layoutSubviews];
    /**** 按钮的尺寸 ****/
    CGFloat buttonW = self.reginavicky_width / 5;
    CGFloat buttonH = self.reginavicky_height;
    CGFloat buttonY = 0;
    // 按钮索引
    int buttonIndex = 0;
    
    for (UIView *subview in self.subviews) {
        // 过滤掉非UITabBarButton
        if (subview.class != NSClassFromString(@"UITabBarButton")) continue;
        
        // 设置frame
        CGFloat buttonX = buttonIndex * buttonW;
        if (buttonIndex >= 2) { // 右边的2个UITabBarButton
            buttonX += buttonW;
        }
        subview.frame = CGRectMake(buttonX, buttonY, buttonW, buttonH);
        
        // 增加索引
        buttonIndex++;
    }
    
    /**** 设置中间的发布按钮的frame ****/
    self.publishButton.reginavicky_width = buttonW * 0.75;
    self.publishButton.reginavicky_height = buttonH * 0.75;
    self.publishButton.reginavicky_centerX = self.reginavicky_width * 0.5;
    self.publishButton.reginavicky_centerY = self.reginavicky_height * 0.5;
    
    
}

#pragma mark - 监听
- (void)publishClick
{
    ReginaVickyLogFunc
    
}
```

#### 5. 设置TabBarItem文字的属性和更替TabBar

ReginaVickyTabBarController.m文件中

```
/**
 *  设置所有UITabBarItem的文字属性
 */
- (void)setupItemTitleTextAttributes
{
    UITabBarItem *item = [UITabBarItem appearance];
    // 普通状态下的文字属性
    NSMutableDictionary *normalAttrs = [NSMutableDictionary dictionary];
    normalAttrs[NSFontAttributeName] = [UIFont systemFontOfSize:15];
    normalAttrs[NSForegroundColorAttributeName] = [UIColor grayColor];
    [item setTitleTextAttributes:normalAttrs forState:UIControlStateNormal];
    // 选中状态下的文字属性
    NSMutableDictionary *selectedAttrs = [NSMutableDictionary dictionary];
    selectedAttrs[NSFontAttributeName] = [UIFont systemFontOfSize:17];
    selectedAttrs[NSForegroundColorAttributeName] = [UIColor whiteColor];
    [item setTitleTextAttributes:selectedAttrs forState:UIControlStateSelected];
}
```

```
/**
 *  更换TabBar
 */
- (void)setupTabBar
{
    [self setValue:[[ReginaVickyTabBar alloc] init] forKeyPath:@"tabBar"];
    
}
```