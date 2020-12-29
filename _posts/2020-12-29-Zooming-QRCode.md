---
layout: post
title: "Zooming QRCode"
subtitle: "放大缩小识别二维码"
date: 22020-12-29 20:59:10
author: "Axag"
header-img: "img/post-bg-rwd.jpg"
tags: [iOS百宝箱]
---
# 放大缩小识别二维码
## 起因
扫描识别二维码其实是一个很常规化的功能，在今天公司年会上有一个通过大屏幕上的App扫码玩小游戏的环节，因为会场比较大，我发现扫码并没有带摄像头放大缩小的功能，根本无法识别远处的二维码即使用人眼看，二维码还是挺大的，当场就十分的抓狂，还好立刻让会场人员打印了N份的二维码每一桌发一张挽救下,年会结束肯定要改进下这个需求.

## 需求
这样看需求就十分明确，就是需要调用摄像头放大缩小的功能来识别远处的二维码

## 过程
这其实是一个挺常规的功能(微信上有的就是常规功能哈哈哈)，第一反应肯定是google或者baidu一份比较成熟的实现方案,接下来看一份搜索结果出来的代码
```
-(void) setVideoScale:(CGFloat) scale {
    [self.device lockForConfiguration:nil];
    
    AVCaptureConnection *videoConnection = [self connectionWithMediaType:AVMediaTypeVideo fromConnections:[[self stillImageOutput] connections]];
    CGFloat maxScaleAndCropFactor = ([[self.stillImageOutput connectionWithMediaType:AVMediaTypeVideo] videoMaxScaleAndCropFactor])/16;
    maxScaleAndCropFactor = 16;
    if (scale > maxScaleAndCropFactor)
        scale = maxScaleAndCropFactor;    
    CGFloat zoom = scale / videoConnection.videoScaleAndCropFactor;
    
    videoConnection.videoScaleAndCropFactor = scale;

    [self.device unlockForConfiguration];
    
    CGAffineTransform transform = self.preview.transform;
    [CATransaction begin];
    [CATransaction setAnimationDuration:.025];
    
    self.preview.transform = CGAffineTransformScale(transform, zoom, zoom);
    
    [CATransaction commit];
}

```
基本找遍了全网络基本都是这么一份代码，核心思路获取摄像头最大放大倍数，然后在允许范围内放大缩小即可，可是实际在真机跑下来发现**maxScaleAndCropFactor**会一直都为1,然后通过进行放大缩小只是后面的**CGAffineTransformScale**有效果，把当前的视图进行放大缩小了，摄像头根本没有任何变动. 搜遍了全网都是这种垃圾代码，一点用都没有,还浪费了很多时间，通过Git上找遍了很多开源的没找到有通过手势或者放大缩小的功能，身为一个代码的搬运工一筹莫展了.

## 思路
静下心来，我们分析下这边和核心步骤
1. 通过屏幕的放大缩小功能 来获取当前手势的放大或者缩小 这需要手势
2. 获取摄像头的最大放大倍数进行安全性限制
3. 获取到当前的设想倍数然后在合理区间进行 倍数增加或者缩小
4. 用一个较远或者较小的二维码进行测试是否能顺利识别
步骤大概如此我们上代码

## 具体实现
1. 首先在view上添加 放大缩小的手势
```
if (@available(iOS 11.0, *)) {
            UIPinchGestureRecognizer *pinchGesture = [[UIPinchGestureRecognizer alloc] initWithTarget:self action:@selector(pinchDetected:)];
            pinchGesture.delegate = self;
            [self.preview addGestureRecognizer:pinchGesture];
        }
```
2. 触发函数 这边有一个关键是45  这是因为在实际测试中发现放大缩小太敏感，稍微一点点放大和缩小都很明显 因此把这个倍数缩小
```
- (void)pinchDetected:(UIPinchGestureRecognizer*)recogniser {
    [self setVideoScale:recogniser.velocity/45];
}
```
3. 这边就是设置安全边界 然后进行放大或者缩小，这边我们要注意maxAvailableVideoZoomFactor 这个是iOS11的 属性 因此在添加手势的时候做个版本判断, 低于iOS11的实现我们就不考虑了,目前微信的支持已经从11开始了.
```
-(void) setVideoScale:(CGFloat) scale {
    [self.device lockForConfiguration:nil];
    
    CGFloat currentZoom = self.device.videoZoomFactor;
    CGFloat maxZoom = self.device.maxAvailableVideoZoomFactor;
    
    CGFloat currentResult = currentZoom + scale;
    
    if (currentResult > maxZoom) {
        currentResult = maxZoom;
    } else if (currentResult < 1) {
        currentResult = 1;
    }
    [self.device setVideoZoomFactor:currentResult];
    
    [self.device unlockForConfiguration];
}
```
4.测试下 

## 最后
基本上实现了我们原先的需求，但是这边还有个问题就是maxZoom 我用6s 7plus 8plus 11 运行后的值都是16 我这边有所疑惑. 
目前这个疑惑还没有答案，有知道的小伙伴也可以告知下，如果有更好的实现方式也希望能互相交流互相进步.