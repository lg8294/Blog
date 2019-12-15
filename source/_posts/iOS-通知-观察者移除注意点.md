---
title: iOS 通知 观察者移除注意点
date: 2019-10-27 15:48:27
tags:
---
1. **从 iOS 9 开始，即使不移除观察者对象，程序也不会出现异常。**

    对于`addObserver`：要区分`ViewController`和普通`NSObject`
    
    - ViewController：在调用ViewController的`dealloc`的时候，系统会调用`[[NSNotificationCenter defaultCenter]removeObserver:self]`方法，所以如果是在viewDidLoad中使用addObserver添加监听者的话可以省掉移除。
    
    - 普通NSObject：在iOS9之后，NSObject也会像ViewController一样在`dealloc`时调用`[[NSNotificationCenter defaultCenter]removeObserver:self]`方法，在iOS9之前的不会调用，需要自己写。
    
    但是在使用类别的时候如果我们添加了通知，那么我们是没有办法在类别里面重写`dealloc`的，如果不移除通知就会出现野指针，这个时候我们就可以在iOS9以上使用`addObserver`，将通知的移除交给系统，iOS9以下使用`addObserverForName+weakSelf`，虽然通知依然存在，但是不会调用`doSomeThing`方法（不要直接在block里面写处理过程啊）。

2. **为什么 iOS 9 之前需要手动移除观察者对象？**

    观察者注册时，通知中心并不会对观察者对象做 retain 操作，而是对观察者对象进行unsafe_unretained 引用。

    **什么是unsafe_unretained？**
    
    因为 Cocoa 和 Cocoa Touch 中的一些类仍然还没有支持 weak 引用。所以，当我们想对这些类使用弱引用的时候，只能用unsafe_unretained来替代。
    
    ```objective-c
    // for attribute
    @property (unsafe_unretained) NSObject *unsafeProperty;
    
    // for variables
    NSObject *__unsafe_unretained unsafeReference;
    ```
    
    不安全引用（unsafe reference）和弱引用 (weak reference) 类似，它并不会让被引用的对象保持存活，但是和弱引用不同的是，当被引用的对象释放的时，不安全引用并不会自动被置为 nil，这就意味着它变成了野指针，而对野指针发送消息会导致程序崩溃。
    
    因此，观察者对象在释放之前必须从通知中心移除引用，否则通知中心就会给野指针所引用的对象发送消息，导致程序崩溃。既然如此，为什么通知中心不对观察者对象进行弱引用呢？我们刚才已经提到，Cocoa 和 Cocoa Touch 中的一些类还没有支持弱引用，所以采用不安全的引用只是为了兼容旧的版本。
    
    从 iOS 9 开始通知中心会对观察者进行弱引用，所以不需要在观察者对象释放之前从通知中心移除。但是，通过`-[NSNotificationCenter addObserverForName:object:queue:usingBlock]`方法注册的观察者依然需要手动的释放，因为通知中心对它们持有的是强引用。