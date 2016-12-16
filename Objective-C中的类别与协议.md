---
title: Objective-C中的类别与协议
date: 2016-10-12 9:32:45
tags: IOS
categories: IOS
---

最近在学习*objective-c*,主要是想学习ios开发。这篇文章的主要目的是记录，记录关于类别，类拓展，协议的使用，博文中有小部分内容摘自*《Objective-C基础教程》*。<!--more-->
## 类别(category)
在面向对象程序设计中，如果你想为一个类添加一个行为，你可以给类添加一个函数来实现，也可以使用子类的方式。但是如果你想为库中的某个类添加一个功能，你无法直接使用在类中创建函数的方式，用子类实现也是不合理的。那么，使用类别就是一种方式。利用*Objective-C*的动态运行时机制。为现有的类添加新的方法。
### 例子
现在，我想要实现一个功能，用户输入一堆字符串，我要得到字符串的长度，并将其存入`NSArray`或者`NSDictionary`中，因为无法将不是对象的整型数据存入上面两个集合中，我们只能先将其转化为`NSNumber`，然后再存入集合中。如下:

```Objective-C
NSNumber *number;
number = [NSNumber numberWithUnsignedInt: [string length]];
// ......
```
这种方式很烦，使用类别我们可以添加多个方法，如下，我们先创建头文件.
### 创建头文件
```Objective-C
#import <Foundation/Foundation.h>

@interface NSString (NumberConvenience)
// 添加属性
@property(nonatomic,strong) NSString *string;

- (NSNumber *) lengthAsNumber;
@end

```
*note:我们只能添加动态属性，无法添加实例变量，只能在运行时创建，而且，必须以`@property`声明。*

### 实现接口
```Objective-C
#import "NSString+NumberConvenience.h"

@implementation NSString (NumberConvenience)

- (NSNumber *)lengthAsNumber
{
    NSUInteger length = [self length];
    return ([NSNumber numberWithUnsignedInt:length]);
}
@end

```
获取字符串的长度，并转化为`NSNumber`.

### 调用

引入头文件,将获取到的`NSNumber`存入`NSMutableDictionary`,并且打印出来:

```Objective-C
#import <Foundation/Foundation.h>
#import "NSString+NumberConvenience.h"

int main(int argc, const char * argv[]) {
    @autoreleasepool {
        NSMutableDictionary *dict = [NSMutableDictionary dictionary];
        [dict setObject:[@"hello" lengthAsNumber] forKey:@"hello"];
        [dict setObject:[@"one" lengthAsNumber] forKey:@"one"];
        [dict setObject:[@"two" lengthAsNumber] forKey:@"two"];
        [dict setObject:[@"three" lengthAsNumber] forKey:@"three"];
        NSLog(@"dictionary is %@",dict);
    }
    return 0;
}

```

### 结果
![](http://7xk0q3.com1.z0.glb.clouddn.com/ios%E7%B1%BB%E5%88%AB.png)
### 缺陷
虽然类别很强大，但是也有几个不足之处，如下：

- 无法添加实例变量，类别没有空间容纳实例变量。
- 名称冲突，类别的方法与原始api有冲突，那么类别拥有高优先级。一般情况，用加前缀的方式避免这个问题。

## 类拓展
类拓展是没有名字的，并且可以拥有实例变量，可以改写权限，并且可以创建数量不限。

### 声明接口
首先，我们声明一个借口，如下:

```Objective-C
#import <Foundation/Foundation.h>

@interface Things : NSObject

@property(assign) NSInteger thing1;
@property(readonly,assign) NSInteger thing2;
- (void) resetAllValue;
@end
```

### 创建类拓展及其实现类

```Objective-C
#import "NSObject+Things.h"

@interface Things()

{
    NSInteger thing4;
}
@property(readwrite,assign) NSInteger thing2;
@property(assign) NSInteger thing3;

@end

@implementation Things

- (void)resetAllValue
{
    self.thing1 = 200;
    self.thing2 = 300;
    self.thing3 = 400;
    thing4 = 5;
}

- (NSString *) description
{
    NSString *desc = [NSString stringWithFormat: @"%ld %ld %ld",
                      _thing1, _thing2, _thing3];
    
    return (desc);
    
}

@end
```

### 实现
```Objective-C
Things *things = [[Things alloc] init];
        things.thing1 = 1;
        NSLog(@"%@", things);
        [things resetAllValue];
        NSLog(@"%@", things);

```

### 结果
![](http://7xk0q3.com1.z0.glb.clouddn.com/%E7%B1%BB%E6%8B%93%E5%B1%95.png)
##  协议

### 非正式协议中的委托
在*IOS*中，经常会使用一种名为*委托*的技术。委托是一种对象，由另一个类请求执行某些操作。比如当程序启动时，`AppKit`的`NSAplication`会询问其委托对象是否打开这个应用。通常情况下，我们需要编写委托对象并将其提供给其他对象。通过实现特定的方法，控制其对象的行为。

- 声明接口与协议

```Objective-C
#import <Foundation/Foundation.h>


@interface ITunesFinder : NSObject<NSNetServiceBrowserDelegate>

@end
```

- 实现接口,编写其响应方法，只需要实现你想要实现的方法。

```Objective-C
#import "NSObject+ITunesFinder.h"

@implementation ITunesFinder

// 搜索到设备
- (void)netServiceBrowser:(NSNetServiceBrowser *)browser didFindService:(NSNetService *)service moreComing:(BOOL)moreComing
{
    [service resolveWithTimeout:10];
    NSLog(@"found one! Name is %@",[service name]);
}

// 设备消失
- (void)netServiceBrowser:(NSNetServiceBrowser *)browser didRemoveService:(NSNetService *)service moreComing:(BOOL)moreComing
{
    [service resolveWithTimeout:10];
    NSLog(@"loat one! name is %@",[service name]);
}
@end

```

- 测试

```Objective-C
ITunesFinder *finder = [[ITunesFinder alloc] init];
        NSNetServiceBrowser *browser = [[NSNetServiceBrowser alloc] init];
        [browser setDelegate:finder];
        [browser searchForServicesOfType:@"_daap._tcp" inDomain:@"local."];
        NSLog (@"begun browsing");
        [[NSRunLoop currentRunLoop] run];

```

### 正式协议(protocol)
正式协议根`Java`的接口一样，实现接口必须要实现所有方法。声明协议使用`@protocol...@end`,采用协议使用`@interface Car : NSObject <NSCopying>`。一个头文件可以使用多个协议，就像实现多个接口一样`@interface Car : NSObject <NSCopying,NSCoding>`。

####  例子

