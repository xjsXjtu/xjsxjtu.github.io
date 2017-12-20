---
layout: post
title: "Learning Objective-C in 30min - from CPP’s perspective"
description: "从一个C++程序员的角度出发，了解Objective-C"
date: 2017-12-20
tags: [objective-c]
comments: true
share: false
---

* auto-gen TOC:
{:toc}


本文基本是一篇读书笔记([苹果官网: ProgrammingWithObjectiveC](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/ProgrammingWithObjectiveC/Introduction/Introduction.html#//apple_ref/doc/uid/TP40011210-CH1-SW1
))，主要从一个C++程序员的角度出发，学习Objective-C。


## Defining classes

- 内容：声明、定义类
- @interface关键词，类似于CPP里的class关键词，与Java的interface关键词完全不同。
- 所有OC类都继承自NSObject(提供init/alloc/new等方法)，类似于Java所有类都继承java.lang.Object
- @property用于定义成员变量，并且自动生成getter、setter方法。
  - 直到XCode8以后才支持静态成员变量
- 成员函数: -表示对象方法，+表示类方法。
- OC所有的对象变量，都用指针表示，不像CPP还有引用等
- OC没有namespace，所有类都不能冲突，所以一般用类前缀区分，如题库用的TTLLiveEngine

```
// XYZPerson.h
@interface XYZPerson : NSObject

- (void)saySomething:(NSString *)greeting;

@property NSString *firstName;
@property NSString *lastName;
@end

// XYZPerson.m
@implemetaion XYZPerson

- (void)saySomething:(NSString *)greeting {
NSLog(@"%@", greeting);
}

@end
```

## Working with Objects

- 内容：对象的定义、初始化；调用对象方法；
- id
  - id是OC提供的特殊关键词，类似于CPP的void*
  - ==id可以动态绑定任何对象，通过id对象，可以调用任何方法==，如果该对象没有该方法，会抛出一个runtime exception
- OC表示发送消息/调用方法：
  - 最常用的方括号：id obj = [[TTLMyClass alloc] init]
  - 也可以像C++一样用小括号：NSLog(@"Hello")

## Encapsulating Data

- 内容：成员变量的访问；OC内存管理
- @property会自动生成getter和setter。（@synthesize 没有仔细研究）
- OC可以不用@property定义成员变量，也可以在直接定义
  - 但是，@property是best practice!

```
@interface SomeClass : NSObject {
NSString *_myNonPropertyInstanceVariable;
}
@end

@implementation SomeClass {
NSString *_anotherCustomInstanceVariable;
}
@end
```

- @property默认为atomic
- ==Object Ownership==(非常重要！！)
  - OC里的指针，默认都是strong reference，相当于CPP的shared_ptr
  - ARC(Automatic Referce counting)为OC的内存管理机制；为了避免strong reference cycle导致的内存泄漏，需要使用weak reference，类似于CPP的weak_ptr

```
@property (weak) id delegate;
// or
NSObject * __weak weakVariable;
```

  - 访问weak reference时，为了保证使用过程中不被dealloc，需要先用strong reference保留一份

```
NSObject *cachedObject = self.someWeakProperty;           // 1
if (cachedObject) {                                       // 2
[someObject doSomethingImportantWith:cachedObject];   // 3
}                                                         // 4
cachedObject = nil;                                       // 5
```

  - weak指针所指向的对象销毁以后，weak指针自动置为nil(意味着再次给它发消息，会导致抛异常)
  - OC有些类不能用weak修饰，只能用unsafe(NSObject * __unsafe_unretained unsafeReference;),这时如果对象被销毁，那么unsafeReference就会是野指针，发消息会导致崩溃，而不是抛异常
- @property可以用(copy)修饰符，此时等号操作符表示深拷贝；否则为浅拷贝。


## Customizing exsiting class

- 内容：Class Category；Class Extension
- Class Category用于给已有的类增加成员方法，包括已有的framework类
  - 该方法可以访问原始类以及原始类子类的property
  - 原始类对象以及原始类的子类对象都可以访问该方法。
  - Category名称必须是独一无二的
    - Category也应该有自己的前缀，因为一个类可能有多个Category，没有前缀容易命名冲突。
    - 如果命名冲突，则对象接收消息为“未定义行为”

```
// YZPerson+XYZPersonNameDisplayAdditions.h
#import "XYZPerson.h"

@interface XYZPerson (XYZPersonNameDisplayAdditions)
- (NSString *)lastNameFirstNameString;
@end

// YZPerson+XYZPersonNameDisplayAdditions.m
#import "XYZPerson+XYZPersonNameDisplayAdditions.h"

@implementation XYZPerson (XYZPersonNameDisplayAdditions)
- (NSString *)lastNameFirstNameString {
return [NSString stringWithFormat:@"%@, %@", self.lastName, self.firstName];
}
@end
```

- Class Extension,也叫匿名Category，只能在compiler-time提供对已有类的扩展。
  - 篇幅不多，也没有仔细看，估计用的不多（猜测类似于Java里的extend关键字，以及于CPP里的实现继承？）


## working with protocol

- 内容：接口@protocol，
- @protocol:类似于CPP的纯虚类，Java的interface，比较好理解

```
// MyClass实现MyProtocol
@interface MyClass : NSObject <MyProtocol>
...
@end
```

## values and collections
- 内容：OC内置的数据结构，类似于CPP里的string、容器等
- scalar数据
  - C里的int、float等都可用
  - OC里定义了相应的类型，NSInteger、NSNumber等，这些类型在不同硬件平台会有不同的存储类型
    - It’s best practice to use these platform-specific types if you might be passing values across API boundaries (both internal and exported APIs), such as arguments or return values in method or function calls between your application code and a framework.
- NSArray(NSMutableArray)，NSSet(NSMutableSet),NSDictionary(NSMutableDictionary)
  - 非常类似于Pyhton等语言提供的容器类型，里面可以装不同类型的对象
  - 初始化、遍历的方法，用时再查。

## working with blocks
- 内容：Block与多线程
- block访问外部变量:
  - 默认：只能取值，不能改变值
  - __block修饰的外部变量，可以在block内改变值
- 其他block性质与CPP函数指针一模一样
- 多线程:"Operation Queue" 与 "GCD（Grand Central Dispatch)"
  - Operation Queue封装了GCD，另外提供了更多的功能，详细参考[stackoverflow](https://stackoverflow.com/questions/10373331/nsoperation-vs-grand-central-dispatch)
  - WebRTC好像用的都是GCD，实现sync/async dispatch

## dealing with Error
- 内容：错误处理，用时再查


## Conventions
- 内容：coding convention，用时再查

## Reference
- [苹果官网: ProgrammingWithObjectiveC](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/ProgrammingWithObjectiveC/Introduction/Introduction.html#//apple_ref/doc/uid/TP40011210-CH1-SW1
)

----
xjs.xjtu@gmail.com

2017-12-20
