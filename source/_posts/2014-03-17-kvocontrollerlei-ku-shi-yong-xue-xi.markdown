---
layout: post
title: "KVOController类库使用学习"
date: 2014-03-17 17:29:07 +0800
comments: true
categories: [Object-C, KVOController, 第三方库]

---

##KVOController类库简介
KVOController 是一个简单安全的 KVO（Key-value Observing，键-值 观察）工具，用于 iOS 和 OS X 应用开发中，开源自 facebook。

KVO 是一个在 MVC（Model-View-Controller）应用程序开发中，用于不同模块间交流的一个很有用的技术。 KVOController 是基于 Cocoa 久经考验的 Key-value Observing 实现（implementation）开发而成的。它提供了一个简单现代的 API，同时也是线程安全的。它有如下优点：

*	使用 Blocks、自定义 Actions 或者 NSKeyValueObserving 回调进行通知.
*	观测者移除时无异常
*	控制器 dealloc 时隐式的观测者移除
*	提升使用 NSKeyValueObservingInitial 的性能
*	线程安全并提供在观测者恢复时额外的保护Thread-safety with special guards against observer resurrection – rdar://15985376.
更多关于 KVO 的信息，可浏览 Apple 的文档：[Introduction to Key-Value Observing](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/KeyValueObserving/KeyValueObserving.html)

<!--more-->

##Demo功能说明
假设我要做这么一个东西，有一个界面类，上面有一个UITextField控件，该会时时显示当前的系统时间，而显示的内容将从一个模型类中去获取。简单点就是模型类每隔一秒会获取当前的系统时间，而每一次时间的更新都时时反映到UITextField上。

##不使用KVOController之前
如下代码所示：
首先新建模型类CurrentTimer，并添加date属性
{% codeblock lang:obj-c CurrentTimer.h %}
#import <Foundation/Foundation.h>

@interface CurrentTimer : NSObject

@property (nonatomic, strong)NSDate *date;

@end
{% endcodeblock %}

添加定时器，每秒获取一次当前时间并赋给date属性
{% codeblock lang:obj-c CurrentTimer.m %}
#import "CurrentTimer.h"

@implementation CurrentTimer

- (id)init
{
    self = [super init];
    
    if (self)
    {
        [NSTimer scheduledTimerWithTimeInterval:1.0 target:self selector:@selector(changeTimer:) userInfo:nil repeats:YES];
    }
    
    return self;
}

- (void)changeTimer:(NSTimer *)timer
{
    self.date = [NSDate date];
}

@end
{% endcodeblock %}

创建视图类对象，添加输出控件UITextField
{% codeblock lang:obj-c ViewController.h %}
#import <UIKit/UIKit.h>
#import "CurrentTimer.h"

@interface ViewController : UIViewController

@property (strong, nonatomic) IBOutlet UITextField *showDate;

@property (nonatomic, strong) CurrentTimer *timer;

@end
{% endcodeblock %}

添加观察者，监听模型类CurrentTimer中的date属性，添加响应方法observeValueForKeyPath，每当date属性值发生改变时都会触发该方法
{% codeblock lang:obj-c ViewController.m %}
#import "ViewController.h"

@interface ViewController ()

@end

@implementation ViewController

- (void)viewDidLoad
{
    [super viewDidLoad];
    
    self.timer = [[CurrentTimer alloc] init];
    
    [self.timer addObserver:self forKeyPath:@"date" options:NSKeyValueObservingOptionNew | NSKeyValueObservingOptionOld context:nil];
}

- (void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary *)change context:(void *)context
{
    if ([keyPath isEqualToString:@"date"])
    {
        NSLog(@"%@", change[NSKeyValueChangeNewKey]);
        NSString *dateStr = [NSString stringWithFormat:@"%@", change[NSKeyValueChangeNewKey]];
        self.showDate.text = dateStr;
    }
}

- (void)didReceiveMemoryWarning
{
    [super didReceiveMemoryWarning];
    // Dispose of any resources that can be recreated.
}

@end
{% endcodeblock %}

运行将能看到最终效果，上述Demo代码可在[这里获取](https://github.com/iBlock/KVOTest)

##使用KVOController示例
####KVOController类库的配置
我比较习惯使用Cocoapods进行配置，如果不懂Cocoapods是什么的，可参考[这里](http://blog.devtang.com/blog/2012/12/02/use-cocoapod-to-manage-ios-lib-dependency/)。

配置好后首先还是一样先新建一个CurrentTimer模型类用来时时获取当前时间，然后新建视图类，主要代码如下：
{% codeblock lang:obj-c ViewController.h %}
#import <UIKit/UIKit.h>
#import "CurrentTimer.h"

@interface ViewController : UIViewController

@property (strong, nonatomic) IBOutlet UITextField *showDate;

@property (strong, nonatomic) CurrentTimer *timer;

@end
{% endcodeblock %}

{% codeblock lang:obj-c ViewController.m %}
#import "ViewController.h"
#import <FBKVOController.h>

@interface ViewController ()
{
    FBKVOController *kvoController;
}
@end

@implementation ViewController

- (void)viewDidLoad
{
    [super viewDidLoad];
    kvoController = [FBKVOController controllerWithObserver:self];
    self.timer = [[CurrentTimer alloc] init];
    [kvoController observe:self.timer keyPath:@"date" options:NSKeyValueObservingOptionOld |NSKeyValueObservingOptionNew block:^(ViewController *view, CurrentTimer *timer, NSDictionary *change) {
        NSString *dateStr = [NSString stringWithFormat:@"%@", change[NSKeyValueChangeNewKey]];
        self.showDate.text = dateStr;
    }];
}

- (void)didReceiveMemoryWarning
{
    [super didReceiveMemoryWarning];
    // Dispose of any resources that can be recreated.
}

@end
{% endcodeblock %}

可以看到，该类库的使用也异常方便，还支持Block，整体看起来很简洁，具体代码可[看这里](https://github.com/iBlock/KVOControllerTest)。

以上就是今天对KVOController类库的学习，记录一下。

