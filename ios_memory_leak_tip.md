---
title: iOS设计开发中避免内存泄漏的几条Tip
data: 2018-10-14 19:52:00
tags: iOS开发
---
# iOS设计开发中避免内存泄漏的几条Tip
## Swift

* 对于 `lazy` 懒加载，可以不在闭包前加 `[weak self]` 
* 对象被释放之后，DispatchWorkItem 可能还在工作。DispatchWorkItem 的cancel函数，只是用于标志isCancel变量。故应在WorkItem工作闭包中检测isCancel。那么我们在设计复杂任务的API的时候，对于循环任务，应该提供一个控制循环结束的变量或者函数。
* 局部作用域中 `DispatchQueue.main.async{block: _ }` 等需要闭包的操作不需要在闭包前加 `[weak self]`
* 实现`deinit`方法，以查看变量是否被释放。这样最直观的可以看出对象是否发生内存泄漏。接着就可以用Instrument工具来查看具体泄漏点.
* 使用Segue取得destinationViewController并作为一个成员变量的方法中，destinationViewController如果不标志为`weak`，将在下一个destinationViewController来时释放，建议改为weak持有。
* 在DisptchWorkItem中调用局部定义的函数，并且此函数中引用了self，则会导致泄漏，如下所示
``` Swift
class Foo {
    var count = 1
    var progressHud: MBProgressHUD!
    var workItem: DispatchWorkItem!
    
    func bar() {
        func doSomething() {
            DispatchQueue.main.async {
                self.count += 1
            }
        } 
        workItem = DispatchWorkItem.init(block: { [weak self] in
            doSomethine()
        })    
        // this will leak
    }
}

let foo = Foo()
foo.bar()

```

* 在设计回调时，如果需要向delegate抛出手动管理内存的对象，这些对象都应该放在一个队列中，统一管理生命周期

## Objective-C




