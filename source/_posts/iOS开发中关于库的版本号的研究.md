---
title: iOS开发中关于库的版本号的研究
date: 2017-09-06 15:39:24
tags: iOS
---

# iOS开发中关于库的版本号的研究
在Xcode中创建了库工程之后，在库头文件中会有以下默认俩个字段：

```OC
//! Project version number for lib.
FOUNDATION_EXPORT double libVersionNumber;

//! Project version string for lib.
FOUNDATION_EXPORT const unsigned char libVersionString[];
```

那么，这俩个字段具体怎么使用呢？
从注释中可以看出，这俩个字段是代表项目的版本号。注意这里说的是项目是你建立的 Xcode 项目，而不是 Xcode 项目中的 Target。我们都知道每个Target都可以很方便的在 General 中修改 Version 和 Build，但是对以上俩个字段是没有影响的。因此你要指定以上俩个字段的值的正确方式是修改 Project->Build Settings->Current Project Version 这个键对应的值。

# 自定义版本号
## 添加自定义字段表示库的版本号
如果你觉得通过过上面的方法修改不方便，那么我们可以添加一个自定义的字段来表示库的版本，例如

```OC
FOUNDATION_EXPORT double mylibVersionNumber;

FOUNDATION_EXPORT NSString *const mylglibVersionString;
```

然后在库的实现文件中指定版本号

```OC
double mylglibVersionNumber = 1.1;

NSString *const mylglibVersionString = @"1.1.1";
```

这个方法中建议使用字符串类型，数值类型由于小数点只能有一个，所以格式就会有限制。

## 通过自定义方法获取版本号
在我们开发库的过程中总是会有一个主功能类，那么我们就可以给这个类添加一个方法返回一个字符串形式的版本号，这个版本号表示这个功能的发布版本。通常由于一个库就是对一个功能的实现的打包，那么我们就可以用这个版本号作为这个库的版本号了。
例如我的一个库主要实现 `Log` 功能，那么我的就可以这样来实现：


```OC
// Log.h
#import <Foundation/Foundation.h>

@interface LGLog : NSObject

- (NSString *)log;

+ (NSString *)version;

@end

// Log.m
#import "LGLog.h"

NSString *const kLogVersionString = @"1.1.1";

@implementation Log

- (NSString *)log {
    NSLog(@"log test");
    return @"log test";
}

+ (NSString *)version {
    return kLogVersionString;
}
@end
```

这样我们就可以在使用的时候知道当前使用的库的版本号了。

## 使用 Target 的版本号来作为库的版本号
如果想要使用 Target 的版本号来作为库的版本号，那么只有采用自定义方法，然后在自定义方法中读取Info.plist信息。***这个方法只对动态库有效***。由于静态库在工程中导入后，最终打包出来后静态库和工程代码是合并到一个包里面的，因此读取到的 Info.plist 其实是 IPA 包的信息。

```OC
// Log.h
#import <Foundation/Foundation.h>

@interface LGLog : NSObject

- (NSString *)log;

+ (NSString *)version;

@end

// Log.m
#import "LGLog.h"

@implementation Log

- (NSString *)log {
    NSLog(@"log test");
    return @"log test";
}

+ (NSString *)version {
    NSDictionary *infoDic = [[NSBundle bundleForClass:self] infoDictionary];
    NSString *version = [infoDic valueForKey:(__bridge NSString*)kCFBundleVersionKey];
    return version;
}
@end
```


