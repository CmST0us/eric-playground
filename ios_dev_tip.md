---
title: iOS设计开发Tip
date: 2018-10-14 19:50:00
tags: iOS
---
# iOS设计开发Tip

## 视图跳转
1. 使用StoryBoard处理TableView的Cell跳转时，使用ViewController -> ViewController 的 Segue，并使用`performSegueWithIdentifier(_ , sender)`控制跳转

2. 

## 生命周期

1. 使用UIScrollView缩放UIImageView时，缩放会导致在每次运行循环调用`layoutSubviews()`
2. 在对象初始化init方法内修改成员变量不会出发didSet方法
```
class Foo {
    var count: Int {
        didSet {
            print("modify")
        }
    }
    
    init(withCount count: Int) {
        self.count = 1
    }
}

Foo(withCount: 1)
```
不会输出modify

## Swift Objective-C Runtime相关
1. @objc标示用于支持runtime, dynamic 用于支持KVO, 添加dynamic标示还需要加@objc标示

2. 纯Swift类没有动态性，但在方法、属性前添加dynamic修饰可以获得动态性。

3. 继承自NSObject的Swift类，其继承自父类的方法具有动态性，其他自定义方法、属性需要加dynamic修饰才可以获得动态性。

4. 若方法的参数、属性类型为Swift特有、无法映射到Objective-C的类型(如Character、Tuple)，则此方法、属性无法添加dynamic修饰（会编译错误）


## Core Animation

### 变化矩阵
变化原点：anchorPoint

坐标转换方程:

<center>![坐标转换矩阵][1]</center>
*官方给的图为什么没有转置符？*


变换矩阵:

<center>![变换矩阵][2]</center>

设备坐标：

<center>![设备坐标][3]</center>

```Swift
import UIKit
import Foundation

extension CATransform3D {
    func printMatrix() {
        let s = """
\(String(describing: m11)) \(String(describing: m12)) \(String(describing: m13)) \(String(describing: m14))
\(String(describing: m21)) \(String(describing: m22)) \(String(describing: m23)) \(String(describing: m24))
\(String(describing: m31)) \(String(describing: m32)) \(String(describing: m33)) \(String(describing: m34))
\(String(describing: m41)) \(String(describing: m42)) \(String(describing: m43)) \(String(describing: m44))
"""
        print(s)
    }
}

print("平移矩阵")
let transform = CATransform3DMakeTranslation(1, 1, 1)
transform.printMatrix()

print("缩放矩阵")
let scaleTransform = CATransform3DMakeScale(0.5, 0.5, 0.5)
scaleTransform.printMatrix()

```
`可以用这段代码验证变化矩阵`


* Translate 平移矩阵

$$
tx, ty, tz
$$


表示$x, y, z$三个坐标上增加量，即

$$
x = x + tx\\y = y + ty\\z = z + tz
$$

* Scale 缩放矩阵

$sx, sy, sz$分别表示$x, y, z$三个坐标上缩放量，即

$$
x = x * sx\\y = y * sy\\z = z * sz
$$


* 旋转矩阵

* * *m34*参数用来做透视

* * 任意轴旋转矩阵
![任意轴旋转矩阵][4]
其中
$$
(u,v,w)=(a_2, b_2, c_2) - (a_1, b_1, c_1)
$$
公式中的$a,b,c$即$a_1, b_1, c_1$
在程序中，通常另向量起点为原点.故可以将旋转轴平移到以原点为起点，此时得到新的$(a_2',b_2',c_2')$，上面的公式就可以看作
$$
(u,v,w)=(a_2',b_2',c_2') 
$$



  [1]: https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/CoreAnimation_guide/Art/transform_basic_math_2x.png
  [2]: https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/CoreAnimation_guide/Art/transform_manipulations_2x.png
  [3]: http://static.zybuluo.com/CmST0us/1gynp0pn2x0nypo9qk1i7daf/20150116141525812.jpeg
  [4]: http://static.zybuluo.com/CmST0us/2vksg8a025xb55yspzprbuuc/2012080921342570.gif