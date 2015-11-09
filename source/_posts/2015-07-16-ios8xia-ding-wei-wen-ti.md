---
layout: post
title: "iOS8 以上定位问题<一>"
date: 2015-07-16 16:50:38 +0800
comments: true
categories: 
---

Xcode6 下，对于系统版本 iOS8 以上版本定位需要自己写授权，不然没有权限定位，即 `CLLocationManagerDelegate` 代理方法不会执行。 

<!--more-->

解决方法：

步骤一：需要在 `info.plist` 文件中加入下面两个缺省字段，值设置为 `YES`。

```objc
NSLocationWhenInUseUsageDescription // 允许在前台获取 GPS 的描述
NSLocationAlwaysUsageDescription // 允许在前、后台获取 GPS 的描述 
```

步骤二：判断设备系统版本，如果系统版本大于8.0，设置定位权限。代码如下：

```objc
#import "ViewController.h"
#import <CoreLocation/CoreLocation.h>
@interface ViewController ()<CLLocationManagerDelegate>
@property (nonatomic, strong) CLLocationManager *locationManager;
@end
 
@implementation ViewController
// 懒加载初始化 self.locationManager
- (CLLocationManager *)locationManager{
    if(!_locationManager){
        _locationManager = [[CLLocationManager alloc] init];
        _locationManager.delegate = self;
    }
    return _locationManager;
}
 
- (void)viewDidLoad {
    [super viewDidLoad];
    
    // 调用请求
    if ([[[UIDevice currentDevice] systemVersion] doubleValue] > 8.0)
    {
        // 设置定位权限仅 iOS8 有意义
        [self.locationManager requestWhenInUseAuthorization]; // 前台定位
         
        // [locationManager requestAlwaysAuthorization]; // 前后台同时定位
    }
    [self.locationManager startUpdatingLocation];
}
 
// 代理方法
- (void)locationManager:(CLLocationManager *)manager didUpdateLocations:(NSArray *)locations{
    NSLog(@"%zd", locations.count);
}
@end
```

至此，iOS8 下定位问题就可以解决了！


