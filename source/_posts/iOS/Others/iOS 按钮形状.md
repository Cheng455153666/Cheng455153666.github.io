---
title: iOS 按钮形状
date: 2016.12.23 16:25:20
categories: iOS
tags: [iOS]
---


## 需要解决的问题

iOS7之后，在 设置->通用->辅助功能->按钮形状 打开后，界面会从

![LeXQg7](http://ckopenbucket.oss-cn-beijing.aliyuncs.com//2021/{mon}/10/LeXQg7.jpg)

变成

![1XT9cH](http://ckopenbucket.oss-cn-beijing.aliyuncs.com//2021/{mon}/10/1XT9cH.jpg)

这样，除了返回按钮之外，按钮会默认加上下划线，tabbar button也会出现对应的阴影(selectionIndicatorImage)

## 解决办法

* UITabbar

```Objective-c
[UITabBar appearance].selectionIndicatorImage = [UIImage new];
```

* UIButton 给UIButton添加了一个分类

```Objective-C
+ (void)load {
    //只执行一次这个方法
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        [UIButton changeFontBottomLine];
    });
}
#pragma mark - Change Font Bottom Line
 + (void)changeFontBottomLine {
    Class class = [self class];
    
    SEL originalSelector = @selector(setTitle:forState:);
    SEL swizzledSelector = @selector(CKSetTitle:forState:);
    
    Method originalMethod = class_getInstanceMethod(class, originalSelector);
    Method swizzledMethod = class_getInstanceMethod(class, swizzledSelector);
    
    BOOL didAddMethod =
    class_addMethod(class,
                    originalSelector,
                    method_getImplementation(swizzledMethod),
                    method_getTypeEncoding(swizzledMethod));
    
    if (didAddMethod) {
        class_replaceMethod(class,
                            swizzledSelector,
                            method_getImplementation(originalMethod),
                            method_getTypeEncoding(originalMethod));
        
    } else {
        method_exchangeImplementations(originalMethod, swizzledMethod);
    }
}
 - (void)CKSetTitle:(NSString *)title forState:(UIControlState)state {
    [self CKSetTitle:title forState:state];
    NSDictionary *attrbiteDic = @{
                                  NSFontAttributeName:self.titleLabel.font,
                                  NSForegroundColorAttributeName:[self titleColorForState:state],
                                  NSUnderlineStyleAttributeName:[NSNumber numberWithInteger:NSUnderlineStyleNone]
                                  };
    
    self.titleLabel.attributedText = [[NSAttributedString alloc] initWithString:title attributes:attrbiteDic];
}
```