---
title: 正确使用NS_DESIGNATED_INITIALIZER
date: 2019-11-02 23:29:46
tags:
---
# 正确使用NS_DESIGNATED_INITIALIZER

### 为什么要用`NS_DESIGNATED_INITIALIZER`

Objective-C 中主要通过`NS_DESIGNATED_INITIALIZER`宏来实现指定构造器的。这里之所以要用这个宏，往往是想告诉调用者要用这个方法去初始化（构造）类对象。

如果子类指定了新的初始化器，那么在这个初始化器内部必须调用父类的Designated Initializer。并且需要重写父类的Designated Initializer，将其指向子类新的初始化器，否则会出现警告`Method override for the designated initializer of the superclass *** not found`，具体取决于你的继承的父类类型：

- UIViewController

  Method override for the designated initializer of the superclass '- initWithNibName:bundle:' not found
  Method override for the designated initializer of the superclass '-initWithCoder:' not found

- UIView

  Method override for the designated initializer of the superclass '-initWithFrame:' not found
  Method override for the designated initializer of the superclass '-initWithCoder:' not found

- NSObject

  Method override for the designated initializer of the superclass '-init' not found

正确写法如下：

```objective-c
// .h
- (instancetype)initWithName:(NSString *)name NS_DESIGNATED_INITIALIZER;
  
// .m
- (instancetype)init
{
    return [self initWithName:@""];
}
 
- (instancetype)initWithName:(NSString *)name
{
    self = [super init];
    if (self) {
        // do something
    }
    return self;
}
```

### 更好的做法

如果定义`NS_DESIGNATED_INITIALIZER`，大多是不想让调用者调用父类的初始化函数，只希望通过该类指定的初始化进行初始化，这时候就可以用`NS_UNAVAILABLE`宏。

如下：

```objective-c
// .h
- (instancetype)init NS_UNAVAILABLE;
- (instancetype)initWithName:(NSString *)name NS_DESIGNATED_INITIALIZER;
```

如果调用者使用`init`初始化，编译器就会给出一个编译错误。使用`NS_UNAVAILABLE`后，就不需要在.m中重写父类初始化函数了。如果要允许调用者使用`init`就需要在.m中重写父类的初始化函数，如上提到的，否则就会报警告。

### 避免使用`new`

如果使用`new`来创建对象的话，即使`init`被声明为`NS_UNAVAILABLE`，也不会收到编译器的警告和错误提示了。

```objective-c
// .h
+ (instancetype)new NS_UNAVAILABLE;
- (instancetype)init NS_UNAVAILABLE;
- (instancetype)initWithName:(NSString *)name NS_DESIGNATED_INITIALIZER;
```

