---
title: iOS 设备分辨率
date: 2018-10-16 20:32
tags: iOS
---

iPhone 设备屏幕尺寸越来越多，下面用一张表汇总

设备|屏幕尺寸(英寸)|屏幕分辨率|说明
---|------------|--------|----
iPhone4/4s|3.5|640 x 960|Retain显示屏,326ppi
iPhone5/5s/5c/SE|4.0|640 x 1136|Retain显示屏,326.ppi
iPhone6/6s/7|4.7|750 x 1334|Retain HD显示屏,326.ppi
iPhone6P/6sP/7P|5.5|1080 x 1920|Retain HD显示屏,401ppi

* 其中 ppi (pixel per inch) 是像素密度单位, 即像素/英寸.
例如: 401ppi 表示每英寸屏幕上有401个像素.
* ppi的计算
例如: 以 iPhone 6Plus 为例
屏幕分辨率 1920 x 1080,
屏幕尺寸 5.5 英寸(手机屏幕对角物理线的长度)
屏幕所有像素数量: √(1920² + 1080²) ≈ 2202.9 (像素)
单位英寸像素数量: 2202.9 ÷ 5.5 ≈ 405.27 (像素/英寸) ≈ 401 ppi
* iOS提供了三种分辨率，分别是：
1. 设计分辨率：
逻辑上的屏幕大小，单位是点。
我们在Interface Builder设计器中的单位和程序代码中的单位都是设计分辨率中的“点”。
2. 资源分辨率：
资源图片的大小，单位是像素。
3. 是以像素为单位的屏幕大小。所有的应用都会渲染到这个屏幕上展示给用户。

    **iOS提供了一种考虑分辨率的简单方式**
    * 例如，在iPhone5和iPhone6之前，iPhone的屏幕大小为320x480点（请注意：这里的单位是“点”而不是像素）。在此之前，iPhone的屏幕分辨率是320x480像素；自从iPhone4采用了Retina屏幕，iOS设备的实际分辨率就变成了上述分辨率与缩放因子的乘积。这意味着虽然在小设备上对元素进行定位时使用的是数字320x480，但实际的像素个数可能更多。
    * 例如，iPhone 4(s)、5(s) 、6(s)和7的缩放因子为2，那么iPhone 4s的实际分辨率为(320x2)x(480x2)=640x960像素。iPhone 5的屏幕更大，为320x568点，即640x1136像素。不同设备的三种分辨率如下表所示：

**不同设备的三种分辨率区别表格**


设备|屏幕尺寸(英寸)|设计分辨率(点)|屏幕分辨率(像素)|资源分辨率(像素)|说明
---|------------|------------|-------------|-------------|---
iPhone4/4s|3.5|320 x 480|640 x 960|640 x 960|1点 = 2倍像素,326ppi
iPhone5/5s/5c/SE|4.0|320 x 568|640 x 1136|640 x 1136|1点 = 2倍像素,326ppi
iPhone6/6s/7|4.7|373 x 667|750 x 1334|750 x 1334|1点 = 2倍像素,326ppi
iPhone6P/6sP/7P|5.5|414 x 736|1080 x 1920|1242 x 2208|1点 = 3倍像素,资源缩小1.15倍,渲染到屏幕上,401ppi