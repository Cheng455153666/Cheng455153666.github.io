---
title: Mac开发基础学习
date: 2017.12.27 10:12
categories: Mac
tags: [Mac]
---


## 创建第一个Mac App

### Step 1 打开Xcode

选择创建新的Xcode项目

![sV9Wsc](http://ckopenbucket.oss-cn-beijing.aliyuncs.com//1620606423699/sV9Wsc.jpg)

### Step 2 选择 Mac OS App

![yugEu5](http://ckopenbucket.oss-cn-beijing.aliyuncs.com//1620606434624/yugEu5.jpg)

### Step 3 创建项目的基本信息

![7oGayr](http://ckopenbucket.oss-cn-beijing.aliyuncs.com//1620606444018/7oGayr.jpg)

 直接运行`Command + R`
 
 ![](media/16206064046656/16206064643999.jpg)

我们会看到一个空白的窗口，这就是我们创建的第一个App啦

## 常用组件

在了解这些控件之前，我们需要了解一下AppKit的坐标系。和UIKit中有所不同的是，AppKit的原点位于右下角。向上/右延伸。

![8qDUVj](http://ckopenbucket.oss-cn-beijing.aliyuncs.com//1620606527966/8qDUVj.jpg)

### NSView

先了解一下NSView中的常用的属性方法：

`frame`:返回控件相对于父控件的位置（以上图为例:frame=(10, 10, 15, 10)）

`bounds`:返回控件相对于自身的位置（以上图为例:frame=(0, 0, 15, 10)）

`needsDisplay`:在当前控件需要重绘时，重新绘制当前控件

`window`:返回当前控件所在的window对象

`draw(_:)`:绘制当前控件(这个方法一般极少被手动调用，我们一般使用needsDisplay)

NSView和UIView中最大的不同就是系统不会默认为其创建图层(layer)，可能更对的是Apple对于性能的考虑，毕竟没有就不用绘制了┑(￣Д ￣)┍，减轻了C/GPU的压力。但是这就意味着我们办法像UIView中一样，肆意把玩layer，做各种动画什么的。不过苹果提供了一个属性--wantsLayer，当我们需要layer做一些事情的时候，只需要将其修改为true即可(默认是false)。

下面就是一段最常用的修改View背景颜色的代码：

```Swift
let v = NSView.init()
v.frame = CGRect.init(x: 10, y: 10, width: 100, height: 100)
v.wantsLayer = true
v.layer?.backgroundColor = NSColor.yellow.cgColor
view.addSubview(v)
```

### NSButton

NSButton和UIButton使用区别还是很大的，NSButton有很多的系统自带的样式，通过ButtonType和BezelStyle来设置。但是需要配合着使用，有的搭配是无效的。其实没什么好讲的，看一下Demo中的组合列表就可以了解了。

以下列出一些常用属性：

```Swift
btn.title = title                                   // 按钮文字
btn.image = image                                   // 按钮图片
btn.action = selector                               // 按钮触发的方法
btn.alternateTitle = ""                             // 开启状态文字
btn.alternateImage = image                          // 开启状态图片
btn.state = .on                                     // 按钮的状态
/**
    noImage         不显示图片
    imageOnly       仅显示图片
    imageLeft       图片在文字左侧
    imageRight      图片在文字右侧
    imageBelow      图片在文字下方
    imageAbove      图片在文字上方
    imageOverlaps   图片和文字重叠
*/
btn.imagePosition = .imageBelow                     // 图文位置
btn.imageScaling = .scaleProportionallyDown         // 设置图片缩放
btn.isBordered = true                               // 按钮是否有边框
btn.isTransparent = true                            // 按钮是否透明
// 以下设置的快捷键为: Shift + Command + I (如果设置的和系统的冲突，则不会触发)
btn.keyEquivalent = "I"                             // 快捷键
btn.keyEquivalentModifierMask = [.shift, .command]  // 快捷键掩码
btn.highlight(true)                                 // 按钮是否为高亮
```

### NSImageView

在MacOS中，推荐ImageView只做展示，如果你要做用户交互，官方推荐使用NSButton。

```Swift
/**
scaleProportionallyDown     原有尺寸
scaleAxesIndependently      图片按ImageView尺寸等比拉伸
scaleProportionallyUpOrDown 图片拉伸到ImageView尺寸
scaleNone                   默认尺寸(原有)
*/
imageView.imageScaling = .scaleProportionallyDown       // 图片缩放类型
/**
图片位置(imageScaling=scaleProportionallyUpOrDown时无效)
alignCenter         居中
alignTop            居中置顶
alignTopLeft        靠左置顶
alignTopRight       靠右置顶
alignLeft           居左
alignBottom         底部
alignBottomLeft     底部靠左
alignBottomRight    底部靠右
alignRight          居右
*/
imageView.imageAlignment = .alignBottom     // 图片对齐方式
/**
none                无样式
photo               照片样式
grayBezel
groove
button
具体的样式可以修改值看看 - - 语言不好形容
*/
imageView.imageFrameStyle = .button         // 边框样式 
imageView.isEditable = true                 // 是否支持编辑(编辑，复制，剪切，拖拽等)
imageView.allowsCutCopyPaste = true         // 图片支持剪切复制
imageView.animates = true                   // 支持动图
imageView.focusRingType = .none             // 获取焦点时状态
// NSImageView编辑时(修改，拖拽等)触发的方法
imageView.target = self
imageView.action = #selector(imageViewAction(sender:))
```

`imageFrameStyle`和背景色会产生冲突，优先级高于背景色

```Swift
imageView.imageFrameStyle = .button
// 下面的代码不会生效
imageView.imageView.wantsLayer = true
imageView.layer?.backgroundColor = NSColor.yellow.cgColor
```

系统为我们提供了一些默认的图片可供使用：

```Swift
imageView.image = NSImage.init(named: .touchBarMailTemplate)    // NSImage.Name.
```

### NSTextField

在MacOS中没有类似于iOS的UILable控件，而是使用NSTextField实现。

接下来我们先模拟出一个MacOS中的`UILabel`:

```Swift
lbl.stringValue = "I`m value"                   // 设置文本
lbl.isEditable = false                          // 是否支持编辑
lbl.isBordered = false                          // 是否有边框
lbl.backgroundColor = NSColor.clear             // 背景色
lbl.textColor = NSColor.black                   // 文字颜色
lbl.maximumNumberOfLines = 0                    // 是否支持多行 0为不限制行数
```

其他的常用属性

```Swift
lbl.placeholderString = "占位文字"
// 属性文字 这个会在后面系统研究 这里仅做了解
let attr = NSMutableAttributedString.init(string: "噜噜噜")
attr.addAttributes([NSAttributedStringKey.foregroundColor:NSColor.red], range: NSRange.init(location: 0, length: 2))
lbl.attributedStringValue = attr
// 限制文本格式 如果输入的文本与定义的格式不符，焦点会始终停留在该TextField上
let formatter = NumberFormatter.init()
formatter.numberStyle = .decimal
lbl.formatter = formatter

lbl.delegate = self
```

这里介绍一下`NSTextFieldDelegate`的代理方法

```Swift
// TextField 获取到焦点并开始编辑
override func controlTextDidBeginEditing(_ obj: Notification) {}
// TextField 文本发生变化
override func controlTextDidChange(_ obj: Notification) {}
// TextField 失去焦点结束编辑
override func controlTextDidEndEditing(_ obj: Notification) {}
// 验证内容 会在TextField失去焦点的时候触发
func control(_ control: NSControl, isValidObject obj: Any?) -> Bool {
// 监听 回车，删除，ESC 等 的输入
func control(_ control: NSControl, textView: NSTextView, doCommandBy commandSelector: Selector) -> Bool {}
// 文本不符合规则时 是否允许失去焦点 true 允许 false 不允许
func control(_ control: NSControl, didFailToFormatString string: String, errorDescription error: String?) -> Bool
```

在iOS开发中，我们需要密码输入框只需要设置一个属性即可，但是在MacOS中，密码输入框是NSTextField的子类：`NSSecureTextField`

```Swift
let slbl = NSSecureTextField.init(string: "123456")
slbl.frame = CGRect.init(x: 10, y: 100, width: 120, height: 40)
view.addSubview(slbl)
```

### NSTextView

和NSTextField有很多类似的地方，我们先来看看他和NSTextField的区别

* 父类不同
    * NSTextField继承自NSControl
    * NSTextView继承自NSText
* 对特定键盘符号响应不同
    * Enter键
        * NSTextField 结束编辑
        * NSTextView 换行
    * Tab键
        * NSTextField 焦点进入下一个控件
        * NSTextView 退格
    * 对特定符号显示不同
        * `"`符号（以下的正常显示为中英文情况都可正常显示）
            * NSTextField 可正常显示
            * NSTextView 英文的`"`会转换为中文的`”`或`“`

总结来说，NSTextField提供的是简单的文本输入，而NSTextView提供更复杂的文本输入。

下面介绍一下NSTextView的常用属性

```Swift
txtV.isAutomaticQuoteSubstitutionEnabled = false                // 关闭自动转换引号
txtV.font = NSFont.systemFont(ofSize: 14)                       // 文字样式
txtV.textColor = NSColor.red                                    // 文字颜色
txtV.backgroundColor = NSColor.yellow                           // 背景色
txtV.textContainerInset = NSSize.init(width: 10, height: 10)    // 设置上下左右边距(width:左右 height:上下)
// 属性文字
let attr = NSMutableAttributedString.init(string: "噜噜噜")
attr.addAttributes([NSAttributedStringKey.foregroundColor:NSColor.red], range: NSRange.init(location: 0, length: 2))
txtV.textStorage?.setAttributedString(attr)

txtV.delegate = self                                            // 代理
```

下面介绍一下NSTextViewDelegate代理的方法

```Swift
// 监听文本改变
func textDidChange(_ notification: Notification) {}
// 监听 回车，删除，ESC 等 的输入
func textView(_ textView: NSTextView, doCommandBy commandSelector: Selector) -> Bool {}
```

### NSAlert

这个弹窗就是NSAlert

![KywkIc](http://ckopenbucket.oss-cn-beijing.aliyuncs.com//1620606895599/KywkIc.jpg)

来看看常见属性

```Swift
alertV.icon = NSImage.init(named: NSImage.Name(rawValue: "1"))          // 弹出图片
alertV.messageText = "messageText"                                      // 弹窗标题
alertV.informativeText = "informativeText"                              // 弹窗信息
/**
critical        警告
informational   描述
warning         严重
*/
alertV.alertStyle = .informational                                      // 弹窗类型
alertV.showsHelp = true                                                 // 左下角显示帮助按钮
alertV.showsSuppressionButton = true                                    // 显示默认的勾选按钮
// alertV.suppressionButton?.state  这个可以获取到勾选按钮状态
alertV.beginSheetModal(for: NSApp.mainWindow!) { (reutrnCode) in }      // 用户点击弹窗按钮
// 添加按钮 注:当按钮过多，AlertView会自动增加宽度，按钮点击回调`reutrnCode`从1000开始计
alertV.addButton(withTitle: "巴扎黑")                                    
// 我们可以自定义AlertView:    alertV.window.contentView? 这个可以获取到弹框面板，但最好不要移除自带控件，否则会报约束错误，建议只做隐藏
```

NSAlertDelegate没有实现什么方法，只有一个监听点击帮助按钮的

```Swift
func alertShowHelp(_ alert: NSAlert) -> Bool {}
```

### NSPopover

这个弹窗就是Popover的样式

![YMB9iv](http://ckopenbucket.oss-cn-beijing.aliyuncs.com//1620606941038/YMB9iv.jpg)

下面看看常见属性

```Swift
popover.appearance = NSAppearance.init(named: .vibrantLight) // 弹窗样式
popover.contentViewController = popovc
/**
behavior在样式上没什么区别，只是对不同的用户操作有不同的响应
applicationDefined popover怎么拖拽点击都不会消失
semitransient 点击除contentViewController之外的区域，popover会消失，拖动窗口不会消失
transient  点击除contentViewController之外的区域，popover会消失，拖动窗口会消失
*/
popover.behavior = .transient
/**
计算方法
sender.(minX,minY,maxX,maxY) + sender.bounds.(minX,minY,maxX,maxY)
*/
popover.show(relativeTo: view.bounds, of: view, preferredEdge: .maxX)
```

介绍一下NSPopoverDelegate

```Swift
// popover是否能被关闭
func popoverShouldClose(_ popover: NSPopover) -> Bool {}
// popover即将显示
func popoverWillShow(_ notification: Notification) {}
// popover已经显示
func popoverDidShow(_ notification: Notification) {}
// popover即将关闭
func popoverWillClose(_ notification: Notification) {}
// popover已经关闭
func popoverDidClose(_ notification: Notification) {}
// 拖拽popover能出现单独窗口 (一个又透明度的View)
func popoverShouldDetach(_ popover: NSPopover) -> Bool {}
// 拖拽popover能出现单独窗口 (拖拽后会展示返回的Window)
func detachableWindow(for popover: NSPopover) -> NSWindow? {}
```

### NSMenu

先来实现一个按钮右键菜单

```Swift
let rbtn = NSButton.init(title: "rBtn", target: self, action: #selector(rbtnAction(sender:)))
// 创建菜单
let menu = NSMenu.init(title: "menu_01")
// 创建菜单项
let menuItem_01 = NSMenuItem.init(title: "menu_01_01", action: #selector(menuItem_01Action(item:)), keyEquivalent: "")
let menuItem_02 = NSMenuItem.init(title: "menu_01_02", action: #selector(menuItem_02Action(item:)), keyEquivalent: "")
// 在菜单中添加菜单项
menu.addItem(menuItem_01)
menu.addItem(menuItem_02)
// 按钮的菜单指向我们创建的菜单
rbtn.menu = menu
```

接着是一个左键菜单

```Swift
// 在btn上弹出cusMenu，NSApp.currentEvent表示用户当前触发的事件
NSMenu.popUpContextMenu(cusMenu, with: NSApp.currentEvent!, for: btn)
```

我们常见的Dock上的右键菜单

```Swift
// 我们需要在AppDelegate中 添加
func applicationDockMenu(_ sender: NSApplication) -> NSMenu? {
    return dockMenu()   // 这里返回的就是我们自定义的Menu
}
// dockMenu
func dockMenu() -> NSMenu {
    // 一级主菜单
    let m_1 = NSMenu.init(title: "m_1")
    // 一级主菜单下的菜单项
    m_1.addItem(withTitle: "m_1_I_1", action: #selector(m_1_I_1Action), keyEquivalent: "")
    let m_1_I_2 = m_1.addItem(withTitle: "m_1_I_2", action: #selector(m_1_I_2Action), keyEquivalent: "")
    // 二级子菜单
    let m_1_I_1_m = NSMenu.init(title: "m_1_I_1_m")
    m_1_I_2.submenu = m_1_I_1_m
    // 二级子菜单下的菜单项
    let m_1_I_2_M_I_1 = NSMenuItem.init(title: "m_1_I_1_m_I_1", action: #selector(m_1_I_2_M_I_1Action), keyEquivalent: "")
    let m_1_I_2_M_I_2 = NSMenuItem.init(title: "m_1_I_1_m_I_2", action: #selector(m_1_I_2_M_I_2Action), keyEquivalent: "")
    m_1_I_1_m.addItem(m_1_I_2_M_I_1)
    m_1_I_1_m.addItem(m_1_I_2_M_I_2)
        
    return m_1
}
```

![7mh2bF](http://ckopenbucket.oss-cn-beijing.aliyuncs.com//1620607005627/7mh2bF.jpg)

我们在AppDelegate中返回Menu就是上面的红色框的位置的菜单，其他项目都是系统默认的，我暂时不知道如何隐藏，或者说能不能隐藏，不过想隐藏上面的`√Window可以通过这几个方法:

方法一

```Swift
NSApp.mainWindow?.title = ""    // 去掉window.title
```

方法二

![FvCSpi](http://ckopenbucket.oss-cn-beijing.aliyuncs.com//1620607036361/FvCSpi.jpg)

方法三

![WvsuRR](http://ckopenbucket.oss-cn-beijing.aliyuncs.com//1620607045396/WvsuRR.jpg)

顶部左侧菜单（菜单栏）

```Swift
NSApp.mainMenu?.items.first?.submenu?.addItem(withTitle: "666", action: #selector(menuItem_01Action(item:)), keyEquivalent: "")
```

![PFjSAT](http://ckopenbucket.oss-cn-beijing.aliyuncs.com//1620607059658/PFjSAT.jpg)

### NSSlider

NSSlider的样式

![EHNzLB](http://ckopenbucket.oss-cn-beijing.aliyuncs.com//1620607071244/EHNzLB.jpg)

先看看常见属性

```Swift
slider.sliderType = .circular           // 进度条样式(圆、线)
slider.isContinuous = true              // 实时监听变化(原本是鼠标停止才触发action事件
slider.numberOfTickMarks = 5            // 分割为多少份
slider.tickMarkPosition = .above        // 分割线位置
slider.allowsTickMarkValuesOnly = true  // 只停留在标尺上
slider.minValue = 0                     // 最小值
slider.maxValue = 100                   // 最大值
slider.floatValue = 25.0                // 当前的值

slider.cell = CusSliderCell.init()      // 当前Slider的视图
```

自定义NSSlider

```Swift
class CusSliderCell: NSSliderCell {
    // 1、自定义指示标识
    override func drawKnob(_ knobRect: NSRect) {
//        // 使用图片
//        let image = NSImage.init(named: NSImage.Name(rawValue: "hand"))
//        image?.draw(in: knobRect)
        // 使用绘图方法绘制
        NSColor.cyan.set()
        let knobPath = NSBezierPath.init(ovalIn: knobRect)
        knobPath.fill()
    }
    
    // 2、自定义标志器左右两边颜色
    override func drawBar(inside rect: NSRect, flipped: Bool) {
        NSColor.red.set()
        let allPath = NSBezierPath.init(roundedRect: rect, xRadius: 2, yRadius: 2)
        allPath.fill()
        
        // 获取左边的区域
        let w = CGFloat((doubleValue - minValue) / (maxValue - minValue))  * rect.width
        
        var myRect = rect
        myRect.size.width = w
        
        let leftPath = NSBezierPath.init(rect: myRect)
        NSColor.yellow.set()
        leftPath.fill()
        
    }
}
```

结合一下NSSlider和NSPopover

```Swift
// 创建popover
let popo = NSPopover.init()
let popvc = CusPopoverVC.init()
popo.contentViewController = popvc
popo.behavior = .semitransient
// 创建slider
let slider = NSSlider.init(frame: .init(x: 10, y: 10, width: 200, height: 40))
slider.isContinuous = true
slider.minValue = 0
slider.maxValue = 100
slider.target = self
slider.action = #selector(sliderAction(slider:))

// 监听slider变化
@objc func sliderAction(slider: NSSlider) -> Void {
    let strV = String.init(format: "%0.2lf", slider.floatValue)
    if let txtF = ((popo?.contentViewController as? CusPopoverVC)?.valueTxtF) {
        txtF.stringValue = strV
    }
        
    let radio = CGFloat.init(slider.floatValue) / CGFloat.init(slider.maxValue)
    // Knob(那个点)的宽高都为21
    let w = radio * (slider.bounds.width - slider.bounds.height)
    let sliderRect = CGRect.init(x: w, y: -10, width: slider.bounds.height, height: slider.bounds.height)
    popo?.show(relativeTo: sliderRect, of: slider, preferredEdge: .minY)
}
```

### NSStatusBar & NSStatusItem

![zDa0Zh](http://ckopenbucket.oss-cn-beijing.aliyuncs.com//1620607118254/zDa0Zh.jpg)

这个就是NSStatusBar，来看看怎么实现

```Swift
// 在AppDelegate中添加
var iconItem: NSStatusItem? // 我们创建的Item必须被强引用，否则不会显示

func applicationDidFinishLaunching(_ aNotification: Notification) {
    iconItem = statusIconItem()
}
// 构建一个StatusItem
func statusIconItem() -> NSStatusItem {
  // NSStatusBar.system 获取系统的Bar    设置NSStatusItem的宽度
    let statusItem = NSStatusBar.system.statusItem(withLength: NSStatusItem.squareLength)
    statusItem.highlightMode = true
  // 设置StatusItem 图片
    statusItem.image = NSImage.init(named: NSImage.Name(rawValue: "sunny_night"))
        
    let statusMenu = NSMenu.init(title: "menu")
    let statusMenu_I_01 = NSMenuItem.init(title: "statusMenu_I_01", action: #selector(statusMenu_I_01Action), keyEquivalent: "R")
    let statusMenu_I_02 = NSMenuItem.init(title: "statusMenu_I_02", action: #selector(statusMenu_I_02Action), keyEquivalent: "T")
        
    statusMenu.addItem(statusMenu_I_01)
    statusMenu.addItem(statusMenu_I_02)
        
    statusItem.menu = statusMenu
    statusItem.toolTip = "I'm toolTip"  // 鼠标悬停在NSStatusItem上会显示
    statusItem.target = self
statusItem.action = #selector(statusItemAction(sender:))    // 设置点击方法 这里传入的是NSStatusBarButton对象
        
    return statusItem
}
```

toolTip

![3B3Zly](http://ckopenbucket.oss-cn-beijing.aliyuncs.com//1620607158384/3B3Zly.jpg)

NSStatusBar & NSStatusItem + NSPopover的练习

```Swift
@objc func statusItemAction(sender: NSStatusBarButton) -> Void {
    let popo = NSPopover.init()
    popo.behavior = .semitransient
    let popVC = NSStatusBarVC.init()
    popo.contentViewController = popVC
    popo.show(relativeTo: (sender.bounds), of: sender, preferredEdge: .minY)
}
```

![bbwGWA](http://ckopenbucket.oss-cn-beijing.aliyuncs.com//1620607177108/bbwGWA.jpg)

注意，如果你使用的是黑丝的菜单栏，有是使用黑色的icon，你可能会看不到你的Icon

![tK4JAb](http://ckopenbucket.oss-cn-beijing.aliyuncs.com//1620607189174/tK4JAb.jpg)

像这样，在不点击时时不会显示的，这时候需要设置一下图片的属性

```Swift
let icon = NSImage.init(named: NSImage.Name(rawValue: "lightning"))
icon?.isTemplate = true // 设置这个属性后，系统会自动为图片取反，以适应菜单栏
statusItem.image = icon
```

![RfRQyB](http://ckopenbucket.oss-cn-beijing.aliyuncs.com//1620607212367/RfRQyB.jpg)

### NSCollectionView基本使用

在AppKit中的NSCollectionView与UIKit中的UICollectionView有着很多异同点。他们都能够使用代理和数据源方法来构建其中的每个Cell/Item。但是对于要求简单的需求，AppKit提供了不使用代理就可以实现的简单方法。

我们先来看看通过代理方法实现的NSCollectionView，下面是一个纯代码实现的NSCollectionView

```Swift
// CKCollectionView.swift

class CKCollectionView: NSView {    // 这里自定义了一个View用来当做容器
    lazy var scrollView: NSScrollView = NSScrollView.init(frame: bounds)
    lazy var collectionView: NSCollectionView = {   // 这里就是我们的NSCollectionView
      // 这里和UICollectionView中的设计一样，我们需要设定布局                                                                                               
        let flowLayout = NSCollectionViewFlowLayout.init()
        flowLayout.scrollDirection = .vertical  // 设置排列方式
        let collection = NSCollectionView.init()
        collection.collectionViewLayout = flowLayout    // 指定布局
        collection.isSelectable = true                  // item是否可以点击
        collection.register(CKCollectionViewItem.self, forItemWithIdentifier: CKCollectionViewItem.identifier)  // 注册Item 和NSCollectionView类似
        return collection
    }()
    override init(frame frameRect: NSRect) {
        super.init(frame: frameRect)
        
        setupSubviews()
    }
    
    required init?(coder decoder: NSCoder) {
        super.init(coder: decoder)
        
        setupSubviews()
    }
    
    func setupSubviews() -> Void {
      // 设置代理
        collectionView.delegate = self
        collectionView.dataSource = self
    /**
      这里需要注意！！！
        scrollView是必要的，collectionView必须属于scrollView的子控件。否则
      func collectionView(_ collectionView: NSCollectionView, itemForRepresentedObjectAt indexPath: IndexPath) -> NSCollectionViewItem
      这个方法不会执行！
      而之所以NSClipView使用再包一层是为了和IB中的层级保持一致，这里具体功效我暂时不是很清楚，目前强行加上
      */
        let clipView = NSClipView.init(frame: bounds)
        clipView.documentView = collectionView;
        scrollView.contentView = clipView;
        addSubview(scrollView)
    
        setupSubviewsConstraints()  // 添加约束
    }
    
    func setupSubviewsConstraints() -> Void {
        NSLayoutConstraint.activate([
            scrollView.topAnchor.constraint(equalTo: (scrollView.superview?.topAnchor)!),
            scrollView.leftAnchor.constraint(equalTo: (scrollView.superview?.leftAnchor)!),
            scrollView.rightAnchor.constraint(equalTo: (scrollView.superview?.rightAnchor)!),
            scrollView.bottomAnchor.constraint(equalTo: (scrollView.superview?.bottomAnchor)!),
            ])
        scrollView.needsLayout = true
      // 使用约束的话 下面这句话是必须有的 否则会影响window，导致window不能用鼠标改变大小
        scrollView.translatesAutoresizingMaskIntoConstraints = false
    }
    
}

extension CKCollectionView: NSCollectionViewDataSource, NSCollectionViewDelegate {
  // 返回Item个数
    func collectionView(_ collectionView: NSCollectionView, numberOfItemsInSection section: Int) -> Int {
        return 100
    }
    /**
    返回我们的Item。这里有一个注意点：
    NSCollectionViewItem是继承自NSViewController。而在NSViewController的初始化方法中，默认回去寻找同名的IB，和UIViewController不同的是，在没有IB的时候，我们直接使用NSViewController.init()是不会有任何问题的，因为loadView方法中会为我们创建默认的view。但是NSViewController则不会，如果我们不手动实现loadView方法自己设置view的话，view不会被创建。所以如果如果遇到控制台说nib找不到的时候，可以重点看看是不是哪个控制器的view没有赋值。
    */
  
    func collectionView(_ collectionView: NSCollectionView, itemForRepresentedObjectAt indexPath: IndexPath) -> NSCollectionViewItem {
        let item = collectionView.makeItem(withIdentifier: CKCollectionViewItem.identifier, for: indexPath)
        return item
    }
    // 点击方法
    func collectionView(_ collectionView: NSCollectionView, didSelectItemsAt indexPaths: Set<IndexPath>) {
        print("items select")
    }
}
// NSCollectionView的布局方法
extension CKCollectionView: NSCollectionViewDelegateFlowLayout {
  // 返回Item的size
    func collectionView(_ collectionView: NSCollectionView, layout collectionViewLayout: NSCollectionViewLayout, sizeForItemAt indexPath: IndexPath) -> NSSize {
        return NSSize.init(width: 80, height: 80)   
    }
}
```

## NSWindow与转场动画

NSWindow主要用于管理应用的窗口，有点类似于一个容器。相对于iOS，MacOS中Window比较复杂。

在NSViewController的viewDidLoad中我们是无法获取到window对象的，和iOS中不同的是，MacOS中，会在view初始化后再初始化window。下面列出一下window的常用属性

```Swift
// 设置窗口标题
        window!.title = "啦啦啦"
        // 隐藏title bar
//        window?.titlebarAppearsTransparent = true
//        window?.titleVisibility = .hidden
        
        // 设置标题栏图标  ???
//        window?.standardWindowButton(.documentIconButton)?.image = (NSImage.init(named: NSImage.Name(rawValue: "lightning")))
        // 设置窗口阴影
        window?.hasShadow = true
        // 设置window背景色
        window!.backgroundColor = NSColor.yellow
        // 获取左上角按钮 并做相关设置
        /**
         只能做是否隐藏之类的操作，其他修改似乎得不到想要的效果
        */
//        let closeBtn = window.standardWindowButton(.closeButton)
//        closeBtn.isHidden = true
//        closeBtn?.image = NSImage.init(named: NSImage.Name.advanced)
        // 设置窗口级别   同级别窗口则后打开的窗口显示在前
//        window.level = NSWindow.Level(rawValue: NSWindow.Level.RawValue(CGWindowLevelForKey(.maximumWindow)))
        // 窗口是否可以通过点击背景移动
        window!.isMovableByWindowBackground = true
        // 窗口全屏/退出全屏
//        window.toggleFullScreen(window)
        // 状态了是否透明
//        window!.titlebarAppearsTransparent = true
        // 设置角标 只有窗口在最小化时才会显示
//        DispatchQueue.main.asyncAfter(deadline: DispatchTime.now() + 5) {
//            window.dockTile.badgeLabel = "666"
//        }
        
//        borderless            // 没有顶部titilebar边框
//        titled                // 有顶部titilebar边框
//        closable              // 带有关闭按钮
//        miniaturizable        // 带有最小化按钮
//        resizable             // 恢复按钮
//        texturedBackground    // 带纹理背景的window
//        unifiedTitleAndToolbar// 标题栏和toolBar 下有统一的分割线
//        fullScreen            // 全屏显示
//        fullSizeContentView   // contentView会充满整个窗口
        
        /* 下面样式只适用于NSPanel及其子类 */
//        utilityWindow
//        docModalWindow
//        nonactivatingPanel
//        hudWindow //用于头部显示的panel
        
        window?.styleMask = [.titled, .hudWindow]   // 窗口样式
        
//        retained      // 兼容老系统参数,基本很少用到
//        nonretained   // 不缓存直接绘制
//        buffered      // 缓存绘制
//        window?.backingType = NSWindow.BackingStoreType.buffered  // 缓存模式
```

### NSWindowController

看到NSWindowController的名字就很好理解，这是一个管理Window的控制器，其中包括了一些Window生命周期等等的方法。我一般把对window的初始化操作放在这个类的对象中。

## 转场动画

在MacOS中也有类似于iOS的转场动画，但是因为没有UINavigationController自动管理，所以需要自己手动做一些操作。
还是直接上代码

```Swift
class MTMainViewController: NSViewController {
 // 构建需要相互转场的2个控制器，而本控制器作为管理者，有点像是UINavigationController的角色
   var vc1 = MTMainViewController01.init(nibName: NSNib.Name(rawValue: "MTMainViewController01"), bundle: nil)
   var vc2 = MTMainViewController02.init(nibName: NSNib.Name(rawValue: "MTMainViewController02"), bundle: nil)
   
   override func viewDidLoad() {
       super.viewDidLoad()
       // 这里做一些基本的设置
       vc1.view.wantsLayer = true
       vc1.view.backgroundColor = NSColor.red
       vc1.view.frame = view.bounds
       
       vc2.view.wantsLayer = true
       vc2.view.backgroundColor = NSColor.yellow
       vc2.view.frame = view.bounds
       // 一定要将这2个控制器添加到本控制器中管理
       addChildViewController(vc1)
       addChildViewController(vc2)
       // 因为转场点击按钮是在子控制器中的，所以在这里设置一下方法
       vc1.toVC2Btn.action = #selector(toNewVC)
       vc2.backToVC1Btn.action = #selector(backToOldVC)
   }
   
   override func viewDidAppear() {
       super.viewDidAppear()
     // 需要将最开始需要显示的那个控制器添加到本控制器当中
       view.addSubview(vc1.view)
   }
   
   @objc func toNewVC() -> Void {
     /**
     转场动画 
     from(从哪个控制器来) 
     to(需要被转场的控制器)
     options 转场的动画效果
     completionHandler 转场结束后的执行
     */
       transition(from: vc1, to: vc2, options: .slideRight) {
           print("to vc2")
       }
   }
   
   @objc func backToOldVC() -> Void {
       transition(from: vc2, to: vc1, options: .slideLeft) {
           print("back to vc1")
       }
   }
}
```

## 给Mac添加定时任务

Mac OS X 的 Launch Daemon / Agent

苹果的[官方文档](https://link.jianshu.com/?t=https%3A%2F%2Fdeveloper.apple.com%2Flibrary%2Fcontent%2Fdocumentation%2FMacOSX%2FConceptual%2FBPSystemStartup%2FChapters%2FIntroduction.html)_有能力最好还是读这个，译文难免有描述不准确的地方

今天折腾了一个小需求，我们先手动实现一下添加任务的操作！

### 谈谈Launch

在Mac OS X 10.4以后，苹果开始使用launchd来管理所有的Process、Application 及 Script。Launch管理的这些进程分为四种：

1. Launch Daemon：在开机时加载
2. Launch Agent：在用户登录时加载
3. XPC Service：
4. Login Items：

下面两种暂时还不会用 = =||，所以先说说前面两种的简单使用。

### Launch Daemon 和 Launch Agents

这两个东西其实是相同的，不同的只是他们的加载时机。Launchd是通过.plist来得知系统中有哪些东西需要被管理的。所以简单的来说，想要新增被管理项，本质上就是新增一个.plist放入苹果的管理文件夹下，然后使其被加载后执行。苹果根据用户的角色提供了不同的Launch存放位置：

```shell
~/Library/LaunchAgents           # 当前用户定义的任务
 /Library/LaunchAgents           # 系统管理员定义的任务
 /Library/LaunchDaemons          # 管理员定义的系统守护进程任务
 /System/Library/LaunchAgents    # 苹果定义的任务
 /System/Library/LaunchDaemons   # 苹果定义的系统守护进程任务
```

很显然，我们是最好不要使用下面两个位置的，而管理员权限比较大，这里我用到的是第一个位置。只为当前用户定任务。进入该目录，创建一个com.hello.plist。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <!-- 任务名称 这个一定不能重复，否则无法被成功创建，系统会告诉你已经有同名的任务了！ -->
    <key>Label</key>    
    <string>com.hello</string>
    <!-- 任务加载时就默认启动一次 -->
    <key>RunAtLoad</key>
    <true/>
    <!-- 任务内容 -->
    <key>ProgramArguments</key>
    <array>     
    <!-- 执行一个脚本_(:зゝ∠)_脚本都可以执行了，基本上什么羞羞的事情都可以做了，脚本内容在最下面贴出 -->
      <string>/Users/chengliqing/Desktop/temp/test.sh</string>
    </array>
  
    <!-- 
        任务执行间隔，如果计算机进入休眠，在唤醒前有多个任务被执行，则这些时间会合并成一个事件再执行。
    -->
    <key>StartInterval</key>
    <integer>60</integer>
  
    <!-- 
        日历的形式执行任务
        Minute <integer>    分钟
        Hour <integer>      小时
        Day <integer>       哪天
        Weekday <integer>   周几(0和7都表示周日)
        Month <integer>     几月
        = = 感觉挺麻烦，在下面说几个例子方便理解
    -->
    <key>StartCalendarInterval</key>
    <array>
        <dict>
            <key>Weekday</key>  <!-- 周几 -->
            <integer>1</integer>
            <key>Hour</key>     <!-- 小时 -->
            <integer>8</integer>
            <key>Minute</key>   <!-- 分钟 -->
            <string>58</string>
        </dict>
        <dict>
            <key>Weekday</key>
            <integer>2</integer>
            <key>Hour</key>
            <integer>8</integer>
            <key>Minute</key>
            <string>52</string>
        </dict>
    </array>
    <!-- 输出日志路径 -->
    <key>StandardOutPath</key>
    <string>/Users/chengliqing/Desktop/temp/stdout.log</string>
    <!-- 异常日志路径 -->
    <key>StandardErrorPath</key>
    <string>/Users/chengliqing/Desktop/temp/stderr.log</string>
</dict>
</plist>
```

### StartCalendarInterval的例子

```xml
<!-- 这个表示每个小时的0分钟会执行此任务 -->
<key>StartCalendarInterval</key>
<dict>
  <key>Minute</key>
  <integer>0</integer>
</dict>
<!-- 在每天的3:55会执行此任务 -->
<key>StartCalendarInterval</key>
<dict>
  <key>Hour</key>
  <integer>3</integer>
  <key>Minute</key>
  <integer>55</integer>
</dict>
<!-- 在每六的3:15会执行此任务 -->
<key>StartCalendarInterval</key>
<dict>
  <key>Hour</key>
  <integer>3</integer>
  <key>Minute</key>
  <integer>15</integer>
  <key>Weekday</key>
  <integer>6</integer>
</dict>
```

写完后可以用`plutil -lint xxx.plist`验证一下，随意~

### Launchctl的基本使用

我们将我们的任务描述出来了，接下来就该使用了！

```shell
# 加载任务
launchctl load ~/Library/LaunchAgents/com.hello.plist
# 强制加载任务, -w选项会将plist文件中无效的key覆盖掉
launchctl load -w ~/Library/LaunchAgents/com.hello.plist

# (-w强制)移除任务
launchctl unload ~/Library/LaunchAgents/com.hello.plist
launchctl unload -w ~/Library/LaunchAgents/com.hello.plist

# 手动执行任务
launchctl start com.hello

# 列出所有任务
launchctl list

# 查看任务列表, 使用 grep '任务部分名字' 过滤
$ launchctl list | grep 'com.hello'
```

在使用launchctl list的时候回列出所有任务能够看到任务的状态(status)，如果出现非0的状态码就表示任务出错了，可以使用：`launchctl error [errorCode]`来查看。

*test.sh*

```shell
#!/bin/sh
say lalala
```

这个脚本的意思是让Mac打印`lalala`

写完并保存后记得将其变为可执行的sh文件，使用

```shell
chmod a+x /Users/chengliqing/Desktop/temp/test.sh
```

到这里，你应该已经会基本的手动添加任务的操作了。接下来我们在代码中添加任务。

这里涉及到权限问题，cocoa application 访问了除沙盒之外的文件且想要在上架到AppStore，是需要授权的，这里先将沙盒关闭，就可以直接写文件到我们指定的路径了。注意这样是不能上架到AppStore的！

这里涉及到使用shell脚本的情况，所以先贴出执行shell脚本的代码。

```Swift
func runCommand(launchPath: String, arguments: [String]) -> String {
    let pipe = Pipe()
    let file = pipe.fileHandleForReading
    
    let task = Process()
    task.launchPath = launchPath
    task.arguments = arguments
    task.standardOutput = pipe
    task.launch()
    
    let data = file.readDataToEndOfFile()
    return String(data: data, encoding: String.Encoding.utf8)!
}
```

先说说步骤:
1. 先将*.plist复制到指定路径
2. 注册任务

然后准备好我们的plist文件，这里我们将文件写到`~/Library/LaunchAgents/`，直接使用复制的形式，先把plist拖入项目，然后复制到指定路径。

```Swift
// 文件拷贝如指定路劲
let fromPath = Bundle.main.path(forResource: "task01", ofType: "plist")
let toPath = "/Users/chengliqing/Library/LaunchAgents/com.hello.plist"
try! FileManager.default.copyItem(atPath: fromPath!, toPath: toPath)

// 执行shell  注册
let result = runCommand(launchPath: "/Users/chengliqing/Desktop/temp/loadTask.sh", arguments: ["SPHardwareDataType"])
print(result)
```

接下来贴出`loadTask.sh`的内容

```shell
# 进入到根路径(这里之所以要进入到根，是因为我们项目到时候启动的路径会发生变化，所以为确保最终查找路径的正确性所以从根路径开始查找)
cd /                                        
cd Users/chengliqing/Library/LaunchAgents/  # 进入到某用户的任务路径
launchctl load clq.hello.plist              # 注册
```

到这里，在不上架到AppStore的情况下，我们的App就可以随意创建任务了。

> 以上的方法是不能上架到AppStore的
> 以上的方法是不能上架到AppStore的
> 以上的方法是不能上架到AppStore的
