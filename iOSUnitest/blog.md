> 目录:    
> 1 XCTest使用示例    
> 
> >    1.1 测试目标代码    
> >   1.2 创建对应测试类    
> >   1.3 编写测试方法    
> >   1.4 测试结果    
> >   1.5 XCTAssert宏方法    
> 
> 2 需要注意的小细节   
> 
> >   2.1 workspace — project — targets 讲解     
> >   2.2.Target Membership    
> >   2.3 Link Binary With Libraries    
> >   2.4 设置本地的支持文件路径   
> >   2.5 PCH   
> >   2.6 Pods引用找不到   
> >   2.7 plist设置   
>
>    3 实战技巧   
> >   3.1 如何测私有方法?   
> >   3.2 如何获取VC中的某个View   
> >   3.3 如何测试网络请求??   
> >   3.4 如何测试异步回调??
>
> 4 心法
> 
> >   4.1 Given / When / Then
> > 

# 1 XCTest使用示例
XCTest是苹果官方提供的测试类, 只需要引入`#import <XCTest/XCTest.h>` 就可以使用.
使用该UnitTest测试一些代码逻辑，使用UITest测试UI的点击交互逻辑。
## 1.1 测试目标代码
![测试目标代码](https://raw.githubusercontent.com/ChenTF/Blog/master/iOSUnitest/Resource/1_1.png)

## 1.2 创建对应测试类
![对应的测试类](https://raw.githubusercontent.com/ChenTF/Blog/master/iOSUnitest/Resource/1_2.png)

**说明：**

* 命名规则：测试的目标类名+Tests
* 任何测试类都需要继承自 XCTestCase 类
* setUp，tearDown是系统默认方法

## 1.3 编写测试方法
![测试方法](https://raw.githubusercontent.com/ChenTF/Blog/master/iOSUnitest/Resource/1_3.png)

**说明：**

* 测试方法必须以`testXXX`开头，Xcode会自动识别出所有的测试方法
* 在一个类中测试方法的调用顺序是按照方法的顺序来调用的，本例中`testCalculate1 -> testCalculate2 -> testCalculate3`。
* 需要测试某些逻辑性操作时，要**注意**测试方法的编写顺序，比如`test1:先插入10条数据 -> test2:根据id查询数据 -> test3:查询所有数据`
* 每个测试方法是分开到，调用顺序 `setUp -> test1 -> tearDown, setUp -> test2 -> tearDown`，这样保证各个测试用例直接不会互相影响

**调用方法：**

* 执行所有测试方法：command + u
* 只执行某个测试方法：点击方法前的菱形（目前是对号/错号）
* 执行某个类的所有测试方法：点击类前的菱形（目前是对号/错号）

## 1.4 测试结果
**Log打印:**

![测试结果打印](https://raw.githubusercontent.com/ChenTF/Blog/master/iOSUnitest/Resource/1_4.png)

**测试列表:**

![测试结果列表](https://raw.githubusercontent.com/ChenTF/Blog/master/iOSUnitest/Resource/1_5.png)

**说明:**

一个测试是否通过，是需要通过XCTassert类来进行验收的，XCTAssert是一系列宏方法，提供了很多的判断。
如果通过则是在方法前是√，没有打印；如果失败则方法前是×，打印。（看上图）

## 1.5 XCTAssert宏方法

```
XCTFail(...)
任何尝试都会测试失败，...是输出的提示文字。（后面都是这样）

XCTAssertNil(expression, ...)
expression为空时通过，否则测试失败。
expression接受id类型的参数。

XCTAssertNotNil(expression, ...)
expression不为空时通过，否则测试失败。
expression接受id类型的参数。

XCTAssert(expression, ...)
expression为true时通过，否则测试失败。
expression接受boolean类型的参数。

XCTAssertTrue(expression, ...)
expression为true时通过，否则测试失败。
expression接受boolean类型的参数。

XCTAssertFalse(expression, ...)
expression为false时通过，否则测试失败。
expression接受boolean类型的参数。

XCTAssertEqualObjects(expression1, expression2, ...)
expression1和expression1地址相同时通过，否则测试失败。
expression接受id类型的参数。

XCTAssertNotEqualObjects(expression1, expression2, ...)
expression1和expression1地址不相同时通过，否则测试失败。
expression接受id类型的参数。

XCTAssertEqual(expression1, expression2, ...)
expression1和expression1相等时通过，否则测试失败。
expression接受基本类型的参数（数值、结构体之类的）。

XCTAssertNotEqual(expression1, expression2, ...)
expression1和expression1不相等时通过，否则测试失败。
expression接受基本类型的参数。

XCTAssertEqualWithAccuracy(expression1, expression2, accuracy, ...)
expression1和expression2之间的任何值都大于accuracy时，测试失败。
expression1、expression2、accuracy都为基本类型。

XCTAssertNotEqualWithAccuracy(expression1, expression2, accuracy, ...)
expression1和expression2之间的任何值都小于等于accuracy时，测试失败。
expression1、expression2、accuracy都为基本类型。

XCTAssertGreaterThan(expression1, expression2, ...)
expression1 <= expression2时，测试失败。
expression为基本类型

XCTAssertGreaterThanOrEqual(expression1, expression2, ...)
expression1 < expression2时，测试失败。
expression为基本类型

XCTAssertLessThan(expression1, expression2, ...)
expression1 >= expression2时，测试失败。
expression为基本类型

XCTAssertLessThanOrEqual(expression1, expression2, ...)
expression1 > expression2时，测试失败。
expression为基本类型

XCTAssertThrows(expression, ...)
expression没抛异常，测试失败。
expression为一个表达式

XCTAssertThrowsSpecific(expression, exception_class, ...)
expression没抛指定类的异常，测试失败。
expression为一个表达式
exception_class为一个指定类

XCTAssertThrowsSpecificNamed(expression, exception_class, exception_name, ...)
expression没抛指定类、指定名字的异常，测试失败。
expression为一个表达式
exception_class为一个指定类
exception_name为一个指定名字

XCTAssertNoThrow(expression, ...)
expression抛出异常时，测试失败。
expression为一个表达式

XCTAssertNoThrowSpecific(expression, exception_class, ...)
expression抛出指定类的异常，测试失败。
expression为一个表达式

XCTAssertNoThrowSpecificNamed(expression, exception_class, exception_name, ...)
expression抛出指定类、指定名字的异常，测试失败。
expression为一个表达式
exception_class为一个指定类
exception_name为一个指定名字
```


# 2 需要注意的小细节
## 2.1 workspace — project — targets 讲解
![wpt关系图](https://raw.githubusercontent.com/ChenTF/Blog/master/iOSUnitest/Resource/2_1.png)

一个工作空间可以包含多个项目，一个项目可以包含多个目标（生成物）。    
一个项目中根据运行的targets不同，可以进行不同的编译设置，project是基础父类，targets是子类，targets的设置会覆盖project的设置。


**与单元测试的关系:**

单元测试是在一个新的target上进行的设置，这样就不会影响程序开发，编译。    
在XCode7中创建一个项目时默认是选中创建测试target的，如果没有，创建方法如下：`File -> New -> target -> UITest/UnitTest`，创建完成后会自动创建对应的文件夹。

## 2.2.Target Membership

![Target Membership设置](https://raw.githubusercontent.com/ChenTF/Blog/master/iOSUnitest/Resource/2_2.png)

Target membership是指XCode中，一个文件属于哪一个工程，在XCode左侧的工程面板中选中一个文件，在XCode右侧的属性面板中会显示其Target Membership，如下图。
当前的文件AppDelegate.m属于书谱这个Target。

**Target Membership的一些属性:**

* .h  文件没有Target Membership, 只有.m有
* 文件夹引用有Target Membership，其子文件继承该文件夹的Target Membership。但面板中不显示子文件的Target Membership。

以前遇到一个错误，就是UIImage创建的时候返回nil，仔细查看发现，图片的Target Membership选项没有勾上。这个错误比较难以发现，特此记之。

## 2.3 Link Binary With Libraries
在测试本地存储时，如果需要一些二进制文件的支持，则test targert也需要引入相应的文件（配置和正常项目需一样）。

![Link Binary设置](https://raw.githubusercontent.com/ChenTF/Blog/master/iOSUnitest/Resource/2_3.png)

![Link Binary设置](https://raw.githubusercontent.com/ChenTF/Blog/master/iOSUnitest/Resource/2_4.png)

## 2.4 设置本地的支持文件路径
提醒：每次修改完配置文件，建议先Clean（Command+Shift+K）缓存，再编译。

![Link Binary设置](https://raw.githubusercontent.com/ChenTF/Blog/master/iOSUnitest/Resource/2_5.png)

## 2.5 PCH
和main target设置成一直, 注意Precompile Prefix Header选项

![PCH设置](https://raw.githubusercontent.com/ChenTF/Blog/master/iOSUnitest/Resource/2_6.png)


## 2.6 Pods引用找不到
当项目中有pod时, 在测试文件中引用pods的文件, 提示找不到, 错误如下:

![Pods设置](https://raw.githubusercontent.com/ChenTF/Blog/master/iOSUnitest/Resource/2_7.png)
     
解决方案: 设置PROJECT的Configurations

![Pods设置](https://raw.githubusercontent.com/ChenTF/Blog/master/iOSUnitest/Resource/2_8.png)


## 2.7 plist设置
设置成与源target一致:
![Pods设置](https://raw.githubusercontent.com/ChenTF/Blog/master/iOSUnitest/Resource/2_9.png)


# 3 实战技巧
## 3.1 如何测私有方法?
在测试中添加一个类别, 类别中是公有方法, 这样就能访问公有方法了。例:

```
@interface PhotoUploadViewController (Specs)
- (void)didTapUploadButton:(UIBarButtonItem *)uploadButton;
@end
```

## 3.2 如何获取VC中的某个View?

通过View的subViews遍历可以获取, 可以封装到Category中, 很方便。

```
/**
 *  例1: 通过Btn的title来查找Btn
 */
- (UIButton *)findButtonWithTitle:(NSString *)title {
   
    UIButton *findBtn = nil;
   
    NSArray *subViews = self.subviews;
    for (NSInteger i = 0; i < subViews.count; i++) {
        UIView *subView = [subViews objectAtIndex:i];
       
        if ([subView isKindOfClass:[UIButton class]]) {
           
            UIButton *subBtn = (UIButton *)subView;
            if ([subBtn.titleLabel.text isEqualToString:title]) {
                findBtn = subBtn;
                break;
            }
           
        }
    }
    return findBtn;
}

// 例2: 根据textField的Placeholder获取textField
- (UITextField *)findTextFieldWithPlaceholder:(NSString *)placeholder {
   
    UIView *findView = [self dj_findViewWithClass:[UITextField class] ResolveBlock:^BOOL(UIView *subView) {
       
        UITextField *subTextField = (UITextField *)subView;
        return [subTextField.placeholder isEqualToString:placeholder];
    }];

    return (UITextField *)findView;
}

/**
 *  例3:封装了查找子View的公用部分
 *
 *  @param class        子类的类
 *  @param resolveBlock 附加条件(返回YES则是)
 *
 *  @return 子View, 如果没找到为nil
 */
- (UIView *)dj_findViewWithClass:(Class)class ResolveBlock:(BOOL (^)(UIView *subView))resolveBlock {
   
    UIView *findView = nil;
   
    NSArray *subViews = self.subviews;
    for (NSInteger i = 0; i < subViews.count; i++) {
        UIView *subView = [subViews objectAtIndex:i];
       
        if ([subView isKindOfClass:class] && resolveBlock(subView)) {
           
            findView = subView;
            break;
        }
    }
   
    return findView;
    }
```

## 3.3 如何测试网络请求
使用OCMock来实现

# 4 心法
在项目中添加单元测试很迷茫, 以什么准则来给项目添加测试, 来什么方法来写测试? 
最后我在这篇[objccn](http://objccn.io/issue-15-1/) 中找到了答案: "你不应该关注于测试，而是应该关注行为。"

那么如何测行为? 我的理解是:"输入+结果 之间的就是行为",  以最少的接口暴露来模拟用户的实际操作行为, 直接给出输入与预期结果, 宏观的调用某个"功能点"。

# 4.1 Given / When / Then

我们可以根据 Given-When-Then 模式来组织我们的测试用例，将测试用例拆分成三个部分。

在 given 部分里，通过创建模型对象或将被测试的系统设置到指定的状态，来设定测试环境。when 这部分包含了我们要测试的代码。在大部分情况，这里只有一个方法调用。在 then 这部分中 ，我们需要检查我们行为的结果：是否得到了我们期望的结果？对象是否有改变？这部分主要包括一些断言。

一个简单的测试用例看起来是这个样子的：
```
- (void)testThatItDoesURLEncoding
{
    // given
    NSString *searchQuery = @"$&?@";
    HTTPRequest *request = [HTTPRequest requestWithURL:@"/search?q=%@", searchQuery];

    // when
    NSString *encodedURL = request.URL;

    // then
    XCTAssertEqualObjects(encodedURL, @"/search?q=%24%26%3F%40");
}
```
这种简单的模式使我们能够更容易地书写和理解这些测试用例，因为它们都遵循了同样的模式。为了更快地浏览，我们甚至会在每个部分的代码上写上 “given”，“when”，“then” 的注释。通过这种方式，这个方法就能很快被理解。

参考:    
[iOS 单元测试](http://my.oschina.net/ChenTF/blog/677309)    
[Target membership](http://www.cnblogs.com/graphics/p/4117353.html)    
[单元测试断言汇总](http://my.oschina.net/u/1418722/blog/340194?fromerr=RUMiSWBO)     
[objccn XCTest](https://github.com/objccn/articles/blob/master/publish/issue15/issue-15-2-morisunshine.md)    


