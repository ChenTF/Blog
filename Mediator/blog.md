# iOS跳转中心设计与实践


# 目录:
---
> 目录:  
> [1.面临的问题与背景](#1)
> 
> > [1.1 如果没有跳转中心时, 工程内组件件是如何调用的?](#1.1)  
> > [1.2 加入跳转中心后](#1.2)  
> > [1.3 跳转中心愿景](#1.3)  
> 
> [2 业内方案对比](#2)  
> 
> > [2.1 URL -> Block](#2.1)  
> > [2.2 Target-Action](#2.2)  
> > [2.2.1 直接使用RunTime的场景](#2.2.1)  
> > [2.2.2 优化后](#2.2.2)  
> > [2.3 LJRoute](#2.3)  
> 
> [3.Target-Action在当前业务中应用效果](#3)  
> > [3.1 本地跳转](#3.1)  
> > [3.2 远程跳转](#3.2)  
>
> [4.未来的愿景](#4)  
> > [4.1 横向切分与纵向切分关系](#4.1)  
> > [4.2 跳转中心与组件化](#4.2)  
>
> [5.相关参考](#5)  


<span id="#1"> </span>
# 1.面临的问题与背景
---
<span id="#1.1"> </span>
## 1.1 如果没有跳转中心时, 工程内组件件是如何调用的?
在项目中, 总是会有各个模块之间的相互调用与耦合. 一般情况是这样的:
![没跳转中心时的场景](https://raw.githubusercontent.com/ChenTF/Blog/master/Mediator/Resource/1_1.jpg)

<span id="#1.2"> </span>
## 1.2 加入跳转中心后
各个模块之间通过Mediator／Route来实现相互调用。
![加入跳转中心后的场景](https://raw.githubusercontent.com/ChenTF/Blog/master/Mediator/Resource/1_2.jpg)

**疑问:**  
好像并没有什么区别？只是将耦合放到了Mediator层, 

<span id="#1.3"></span>
#1.3 跳转中心愿景
*一个模块只与Mediator耦合，不与外部模块耦合*

**3个问题：**

* 只与Mediator耦合，如何知道别的模块提供了什么接口？  
* 如何实现跨组件调用？  
* 如何破除Mediator与模块间的耦合？  

# 2、业内方案对比<span id="#2"></span>
---
## 2.1 URL -> Block<span id="#2.1"></span>
**代码推演:**  
*核心思想:* 将URL与Block一一绑定. 在使用的时候只使用URL, 就可以调用到对应的Block.

核心模块Code:

```
// 跳转中心模块
// 注册
- (void)registerMediatorWithMoudle:(NSString *)moudle eventBlock:(EventBlock)block {
    [self.eventDict setObject:block forKey:moudle];
}

// 打开
- (id)openUrl:(NSString *)urlStr {
    NSLog(@"URLMediator openUrl : %@", urlStr);
    
    NSRange spaceRange = [urlStr rangeOfString:@"?"];
    if (spaceRange.location == NSNotFound) {
        NSLog(@"URLMediator 不支持该url");
        return nil;
    }
    
    NSString *moudle = [urlStr substringToIndex:spaceRange.location];
    EventBlock block = self.eventDict[moudle];
    
    if (block) {
        NSDictionary *param = [urlStr parameterFromUrlStr];
        return block(param);
    } else {
        NSLog(@"URLMediator openUrl Error.");
    }
    
    return nil;
}
// 组件方接入
@implementation AMediatorRegister

+ (void)load {
    [[URLMediator sharedInstance] registerMediatorWithMoudle:@"suyun://A" eventBlock:^id(NSDictionary *param) {
        NSLog(@"A组件被调用了");
        
        AViewController *aVC = [[AViewController alloc] init];
        
        NSString *labelStr = param[@"labelStr"];
        if (labelStr) {
            [aVC setLabelStr:labelStr];
        }
        
        NSString *jump = param[@"jump"];
        if ([jump integerValue] == 1) {
            [kRootNavigation pushViewController:aVC animated:YES];
        }
        
        return aVC;
    }];
}
// 调用
[[URLMediator sharedInstance] openUrl: @"suyun://A?jump=1&labelStr=fdafs"];
```

**类图:**

![URL->Block类图](https://raw.githubusercontent.com/ChenTF/Blog/master/Mediator/Resource/2_1.jpg)

**存在的问题:**

* Block内存常驻, 不能释放
* 本地调用需要使用转URL的方式, 容易出错
* 只能传可字符串的数据

**解决方案:**

* 问题1: 无法解决
* 问题2: 通过文档规范
* 问题3: 无法解决

**远程跳转与本地跳转说明:**

*远程跳转:* URL方式调整, 例: suyun://homeVC?jump=1&title=aaa  
*本地跳转:* 类似于方法调用, 可以包含特殊参数  
URL方式就不能传非常规数据类型, 比如UIImage, NSData, 自定义类. 只能传可被URL化的数据, 所以URL->Block的方式也就限制了只能传常规参数.  

<img src="https://raw.githubusercontent.com/ChenTF/Blog/master/Mediator/Resource/2_2.jpg"  alt="远程调用与本地调用关系图"  height=180px width=300px />


## 2.2 Target-Action<span id="#2.2"></span>

### 2.2.1 直接使用RunTime的场景

```
    Class class = NSClassFromString(@"AViewController");
    SEL sel = NSSelectorFromString(@"setLabelStr:");
    
    UIViewController *vc = [[class alloc] init];
    [vc performSelector:sel withObject:@"aaa"];
    
    [self.navigationController pushViewController:vc animated:YES];
```

**存在的缺点:**

* 类名, 方法名, 参数不明确, 容易出错;
* 使用方极其不方便, 需要写一大堆代码;

**优化点:**

* RunTime调用的部分是通用的, 可以抽离到一个类中
* 在这个类中添加安全处理, 如class为nil, sel不存在
* 使用Category类对Mediator进行扩充, Mediator+MouduleName
* 使用Target_MouduleName来提取组件接口

### 2.2.2 优化后

**核心模块代码:**

```
// 跳转中心 Mediator
- (id)performTarget:(NSString *)targetName action:(NSString *)actionName params:(NSDictionary *)params shouldCacheTarget:(BOOL)shouldCacheTarget
{
    
    NSString *targetClassString = [NSString stringWithFormat:@"Target_%@", targetName];
    NSString *actionString = [NSString stringWithFormat:@"Action_%@:", actionName];
    Class targetClass;
    
    NSObject *target = self.cachedTarget[targetClassString];
    if (target == nil) {
        targetClass = NSClassFromString(targetClassString);
        target = [[targetClass alloc] init];
    }
    
    SEL action = NSSelectorFromString(actionString);
    
    if (target == nil) {
        // 这里是处理无响应请求的地方之一，这个demo做得比较简单，如果没有可以响应的target，就直接return了。实际开发过程中是可以事先给一个固定的target专门用于在这个时候顶上，然后处理这种请求的
        return nil;
    }
    
    if (shouldCacheTarget) {
        self.cachedTarget[targetClassString] = target;
    }

    if ([target respondsToSelector:action]) {
        return [self safePerformAction:action target:target params:params];
    }
}
// 组件A的接口 Mediator+A 
NSString * const kCTMediatorTargetA = @"A";

NSString * const kCTMediatorAction_AVC = @"AVC";

@implementation CTMediator (A)

- (void)CTMediator_AWithOpen:(BOOL)open
                    LabelStr:(NSString *)labelStr {
    NSMutableDictionary *paramsToSend = [[NSMutableDictionary alloc] init];
    
    if (open) {
        paramsToSend[@"open"] = @"1";
    }
    if (labelStr) {
        paramsToSend[@"labelStr"] = labelStr;
    }
    
    [self performTarget:kCTMediatorTargetA action:kCTMediatorAction_AVC params:paramsToSend shouldCacheTarget:NO];
}
// 组件A接口的实现 Target_A
@implementation Target_A

- (id)Action_AVC:(NSDictionary *)params {
    AViewController *aVC = [[AViewController alloc] init];
    
    NSString *labelStr = params[@"labelStr"];
    if (labelStr) {
        [aVC setLabelStr:labelStr];
    }
    
    NSString *open = params[@"open"];
    if ([open integerValue] == 1) {
        [kRootNavigation pushViewController:aVC animated:YES];
    }
    
    return aVC;
}
// 使用方
// 1.本地调用
[[CTMediator sharedInstance] CTMediator_AWithOpen:YES LabelStr:@"fsfda”];
// 2.远程调用
[[CTMediator sharedInstance] dispatchCenterWithUrlStr:@“suyun://xxx"];
```

**类图:**

![CTMediator类图](https://raw.githubusercontent.com/ChenTF/Blog/master/Mediator/Resource/2_3.jpg)

**存在的缺点:**

* Moudle中的调用在Target_A中需要再写一遍.

*如何实现远程调用?*  
本组件的重点是先实现本地调用, 远程调用是将对应的URL进行解析后, 进行的转换, 分发实现的.

## 2.3 LJRoute<span id="#2.3"></span>
**核心思想:**  
   使用宏方法来重写系统方法, 在宏方法内注册组件, 将对应的类名, 方法名保存到跳转中心中.  在使用时通过RunTime+对应的名称来实现实例化与调用.

**内部调用**  
<img src="https://raw.githubusercontent.com/ChenTF/Blog/master/Mediator/Resource/2_4.jpg" lat="LJRoute内部调用方法" width="600" height="350">


**存在的问题:**

* 接入成功过高: 每次写代码需要使用宏方法, 而不是原方法. 修改了原有编码习惯.
* 业务侵入太深: 跳转中心与组件完全绑定了, 后续如果有修改, 将是灾难

**优化点:**

* 采用无侵入的方式来实现

# 3.Target-Action在当前业务中应用效果<span id="#3"></span>
## 3.1 本地跳转<span id="#3.1"></span>
```
// CTmediator+Login.h
@interface CTMediator (Login)

- (void)CTMediator_jumpToLoginVC;

@end

// CTmediator+Charge.h
@interface CTMediator (Charge)

/**
 生成收费页
 
 @param orderID 订单id
 @return <#return value description#>
 */
- (UIViewController *)CTMediator_beforeHandSettleVCWithOrderID:(NSString *)orderID;
```

##3.2 远程跳转<span id="#3.2"></span>
```
// CTmediator+SYDispatch.h
@interface CTMediator (SYDispatch)

/**
 跳转中心(会直接进行跳转)

 @param urlStr urlStr
 @return YES(有处理)/NO(无处理)
 */
- (BOOL)dispatchCenterWithUrlStr:(NSString *)urlStr;

@end
```

# 4.未来的愿景<span id="#4"></span>
## 4.1 横向切分与纵向切分关系<span id="#4.1"></span>

横向拆分业务、功能模块:
![横向拆分](https://raw.githubusercontent.com/ChenTF/Blog/master/Mediator/Resource/4_1.jpg)

纵向拆分技术、架构模块:
![纵向拆分](https://raw.githubusercontent.com/ChenTF/Blog/master/Mediator/Resource/4_2.jpg)


# 5 相关参考:<span id="#5"></span>
[CTMediator](https://casatwy.com/iOS-Modulization.html)  
[蘑菇街 App 的组件化之路](http://limboy.me/tech/2016/03/10/mgj-components.html)  
[iOS组件化方案探索](http://blog.cnbang.net/tech/3080/)  
[模块化与组件化](http://tutuge.me/2016/03/29/modular-and-component-summary/)
