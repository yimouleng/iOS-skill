前言
----------

iOS开发过程中，各种小问题，小技巧，代码片段， 在这里收集，写下，方便查阅，有时间会将其分类，并添加导航和标签。

PS:查找东西，现在只能用com+f,搜索关键字查找。。。


返回输入键盘
```objective-c
<UITextFieldDelegate>

- (BOOL)textFieldShouldReturn:(UITextField *)textField {
    [textField resignFirstResponder];
    return YES;
}
```
CGRect
```objective-c
CGRectFromString(<#NSString *string#>)//有字符串恢复出矩形
CGRectInset(<#CGRect rect#>, <#CGFloat dx#>, <#CGFloat dy#>)//创建较小或者较大的矩形
CGRectIntersectsRect(<#CGRect rect1#>, <#CGRect rect2#>)//判断两巨星是否交叉，是否重叠
CGRectZero//高度和宽度为零的，位于（0，0）的矩形常量
```
隐藏状态栏
```objective-c
[UIApplication sharedApplication] setStatusBarHidden:<#(BOOL)#> withAnimation:<#(UIStatusBarAnimation)#>//隐藏状态栏
```
自动适应父视图大小
```objective-c
self.view.autoresizesSubviews = YES;
    self.view.autoresizingMask = UIViewAutoresizingFlexibleWidth | UIViewAutoresizingFlexibleHeight;
```
UITableView的一些方法
```objective-c
这里我自己做了个测试，缩进级别设置为行号，row越大，缩进越多
<UITableViewDelegate>

- (NSInteger)tableView:(UITableView *)tableView indentationLevelForRowAtIndexPath:(NSIndexPath *)indexPath {
    NSInteger row = indexPath.row;
    return row;
}
```
把plist文件中的数据赋给数组

```objective-c
NSString *path = [[NSBundle mainBundle] pathForResource:@"States" ofType:@"plist"];
NSArray *array = [NSArray arrayWithContentsOfFile:path];
```
获取触摸的点

```objective-c
- (CGPoint)locationInView:(UIView *)view;
- (CGPoint)previousLocationInView:(UIView *)view;
```
获取触摸的属性

```objective-c
@property(nonatomic,readonly) NSTimeInterval      timestamp;
@property(nonatomic,readonly) UITouchPhase        phase;
@property(nonatomic,readonly) NSUInteger          tapCount;
```
从plist中获取数据赋给字典

```objective-c
NSString *plistPath = [[NSBundle mainBundle] pathForResource:@"book" ofType:@"plist"];
NSDictionary *dictionary = [NSDictionary dictionaryWithContentsOfFile:plistPath];
```
NSUserDefaults注意事项

```objective-c
设置完了以后如果存储的东西比较重要的话，一定要同步一下
[[NSUserDefaults standardUserDefaults] synchronize];
```

获取Documents目录

```objective-c
NSString *documentsDirectory = NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES)[0];
```
获取tmp目录

```objective-c
NSString *tmpPath = NSTemporaryDirectory();
```
利用Safari打开一个链接

```objective-c
NSURL *url = [NSURL URLWithString:@"http://baidu.com"];
[[UIApplication sharedApplication] openURL:url];
```
利用UIWebView显示pdf文件，网页等等

```objective-c
<UIWebViewDelegate>

UIWebView *webView = [[UIWebView alloc]initWithFrame:self.view.bounds];
webView.delegate = self;
webView.scalesPageToFit = YES;
webView.autoresizingMask = UIViewAutoresizingFlexibleWidth | UIViewAutoresizingFlexibleHeight;
[webView setAllowsInlineMediaPlayback:YES];
[self.view addSubview:webView];
    
NSString *pdfPath = [[NSBundle mainBundle] pathForResource:@"book" ofType:@"pdf"];
NSURL *url = [NSURL fileURLWithPath:pdfPath];
NSURLRequest *request = [NSURLRequest requestWithURL:url cachePolicy:(NSURLRequestUseProtocolCachePolicy) timeoutInterval:5];
[webView loadRequest:request];
```
UIWebView和html的简单交互

```objective-c
myWebView = [[UIWebView alloc]initWithFrame:self.view.bounds];
[myWebView loadRequest:[NSURLRequest requestWithURL:[NSURL URLWithString:@"http://www.baidu.com"]]];
NSError *error;
NSString *errorString = [NSString stringWithFormat:@"<html><center><font size=+5 color='red'>AnError Occurred;<br>%@</font></center></html>",error];
[myWebView loadHTMLString:errorString baseURL:nil];

//页面跳转了以后，停止载入
-(void)viewWillDisappear:(BOOL)animated {
    if (myWebView.isLoading) {
        [myWebView stopLoading];
    }
    myWebView.delegate = nil;
    [UIApplication sharedApplication].networkActivityIndicatorVisible = NO;
}
```
汉字转码

```objective-c
NSString *oriString = @"\u67aa\u738b";
NSString *escapedString = [oriString stringByReplacingPercentEscapesUsingEncoding:NSUTF8StringEncoding];
```
处理键盘通知
```objective-c
先注册通知，然后实现具体当键盘弹出来要做什么，键盘收起来要做什么
- (void)registerForKeyboardNotifications {
    keyboardShown = NO;//标记当前键盘是没有显示的
    [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(keyboardWasShown:) name:UIKeyboardWillShowNotification object:nil];
    [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(keyboardWasHidden:) name:UIKeyboardDidHideNotification object:nil];
}
//键盘显示要做什么
- (void)keyboardWasShown:(NSNotification *)notification {
    if (keyboardShown) {
        return;
    }
    NSDictionary *info = [notification userInfo];
    
    NSValue *aValue = [info objectForKey:UIKeyboardFrameBeginUserInfoKey];
    CGSize keyboardSize = [aValue CGRectValue].size;
    CGRect viewFrame = scrollView.frame;
    viewFrame.size.height = keyboardSize.height;
    
    CGRect textFieldRect = activeField.frame;
    [scrollView scrollRectToVisible:textFieldRect animated:YES];
    keyboardShown = YES;
}

- (void)keyboardWasHidden:(NSNotification *)notification {
    NSDictionary *info = [notification userInfo];
    NSValue *aValue = [info objectForKey:UIKeyboardFrameEndUserInfoKey];
    CGSize keyboardSize = [aValue CGRectValue].size;
    
    CGRect viewFrame = scrollView.frame;
    viewFrame.size.height += keyboardSize.height;
    scrollView.frame = viewFrame;
    
    keyboardShown = NO;
}
```
点击键盘的next按钮，在不同的textField之间换行

```objective-c
- (BOOL)textFieldShouldReturn:(UITextField *)textField {
    
    if ([textField returnKeyType] != UIReturnKeyDone) {
        NSInteger nextTag = [textField tag] + 1;
        UIView *nextTextField = [self.tableView viewWithTag:nextTag];
        [nextTextField becomeFirstResponder];
    }else {
        [textField resignFirstResponder];
    }
    
    return YES;
}
```
设置日期格式

```objective-c
dateFormatter = [[NSDateFormatter alloc]init];
dateFormatter.locale = [NSLocale currentLocale];
dateFormatter.calendar = [NSCalendar autoupdatingCurrentCalendar];
dateFormatter.timeZone = [NSTimeZone defaultTimeZone];
dateFormatter.dateStyle = NSDateFormatterShortStyle;
NSLog(@"%@",[dateFormatter stringFromDate:[NSDate date]]);
```
加载大量图片的时候，可以使用

```objective-c
NSString *imagePath = [[NSBundle mainBundle] pathForResource:@"icon" ofType:@"png"];
UIImage *myImage = [UIImage imageWithContentsOfFile:imagePath];
```
有时候在iPhone游戏中，既要播放背景音乐，同时又要播放比如枪的开火音效。

```objective-c
NSString *musicFilePath = [[NSBundle mainBundle] pathForResource:@"xx" ofType:@"wav"];
NSURL *musicURL = [NSURL fileURLWithPath:musicFilePath];
AVAudioPlayer *musicPlayer = [[AVAudioPlayer alloc]initWithContentsOfURL:musicURL error:nil];
[musicPlayer prepareToPlay];
musicPlayer.volume = 1;
musicPlayer.numberOfLoops = -1;//-1表示一直循环
```
从通讯录中读取电话号码，去掉数字之间的-
```objective-c
NSString *originalString = @"(123)123123abc";
NSMutableString *strippedString = [NSMutableString stringWithCapacity:originalString.length];
NSScanner *scanner = [NSScanner scannerWithString:originalString];
NSCharacterSet *numbers = [NSCharacterSet characterSetWithCharactersInString:@"0123456789"];

    while ([scanner isAtEnd] == NO) {
        NSString *buffer;
        if ([scanner scanCharactersFromSet:numbers intoString:&buffer]) {
            [strippedString appendString:buffer];
        }else {
            scanner.scanLocation = [scanner scanLocation] + 1;
        }
    }
    NSLog(@"%@",strippedString);
```

字符串是否为空
```objective-c

- (BOOL) isBlankString:(NSString *)string {
    if (string == nil || string == NULL) {
        return YES;
    }
    if ([string isKindOfClass:[NSNull class]]) {
        return YES;
    }
    if ([[string stringByTrimmingCharactersInSet:[NSCharacterSet whitespaceCharacterSet]] length]==0) {
        return YES;
    }
    return NO;
} 
```

正则判断：字符串只包含字母和数字
```objective-c

NSString *myString = @"Letter1234";
NSString *regex = @"[a-z][A-Z][0-9]";
NSPredicate *predicate = [NSPredicate predicateWithFormat:@"SELF MATCHES %@",regex];

    if ([predicate evaluateWithObject:myString]) {
        //implement
    }
```
设置`UITableView`的滚动条颜色
```objective-c
self.tableView.indicatorStyle = UIScrollViewIndicatorStyleWhite;
```

网络编程
开发web等网络应用程序的时候，需要确认网络环境，连接情况等信息。如果没有处理它们，是不会通过apple的审查的。
系统自带的网络检查是原生的，AFNetworking也为我们添加了相关检测机制，所以这个直接在介绍AFNetworking的时候详解吧。

使用`NSURLConnection`下载数据
```objective-c
1. 创建对象
NSMutableURLRequest *request = [NSMutableURLRequest requestWithURL:[NSURL URLWithString:@"http://www.baidu.com"]];
[NSURLConnection connectionWithRequest:request delegate:self];

2. NSURLConnection delegate 委托方法
- (void)connection:(NSURLConnection *)connection didReceiveResponse:(NSURLResponse *)response {

}

- (void)connection:(NSURLConnection *)connection didReceiveData:(NSData *)data {
    
}

- (void)connection:(NSURLConnection *)connection didFailWithError:(NSError *)error {
    
}

- (void)connectionDidFinishLoading:(NSURLConnection *)connection {
    
}

3. 实现委托方法
- (void)connection:(NSURLConnection *)connection didReceiveResponse:(NSURLResponse *)response {
    self.receiveData.length = 0;//先清空数据
}

- (void)connection:(NSURLConnection *)connection didReceiveData:(NSData *)data {
    [self.receiveData appendData:data];
}

- (void)connection:(NSURLConnection *)connection didFailWithError:(NSError *)error {
    //错误处理
}

- (void)connectionDidFinishLoading:(NSURLConnection *)connection {
    [UIApplication sharedApplication].networkActivityIndicatorVisible = NO;
    NSString *returnString = [[NSString alloc]initWithData:self.receiveData encoding:NSUTF8StringEncoding];
    firstTimeDownloaded = YES;
}
```
iOS的动画以及自定义图形，开个专栏总结。

隐藏状态栏
```objective-c
[UIApplication sharedApplication].statusBarHidden = YES;
```
.m文件与.mm文件的区别
```objective-c
.m文件是objective-c文件
.mm文件相当于c++或者c文件
```
Safari其实没有把内存的缓存写到存储卡上

读取一般性文件
```objective-c
- (void)readFromTXT {
    NSString *tmp;
    NSArray *lines;//将文件转化为一行一行的
    lines = [[NSString stringWithContentsOfFile:@"testFileReadLines.txt"] componentsSeparatedByString:@"\n"];
    
    NSEnumerator *nse = [lines objectEnumerator];
    
    //读取<>里的内容
    while (tmp == [nse nextObject]) {
        NSString *stringBetweenBrackets = nil;
        NSScanner *scanner = [NSScanner scannerWithString:tmp];
        [scanner scanUpToString:@"<" intoString:nil];
        [scanner scanString:@"<" intoString:nil];
        [scanner scanUpToString:@">" intoString:&stringBetweenBrackets];
        NSLog(@"%@",[stringBetweenBrackets description]);
    }
}
```
隐藏`UINavigationBar`
```objective-c
 [self.navigationController setNavigationBarHidden:YES animated:YES];
```

调用电话，短信，邮件
```objective-c
[[UIApplication sharedApplication] openURL:[NSURL URLWithString:@"mailto:apple@mac.com?Subject=hello"]];
sms://调用短信
tel://调用电话
itms://打开MobileStore.app
```
获取版本信息
```objective-c
UIDevice *myDevice = [UIDevice currentDevice];
NSString *systemVersion = myDevice.systemVersion;
```
`UIWebView`的使用
```objective-c
<UIWebViewDelegate>

webView.delegate = self;

(BOOL)webView:(UIWebView *)webView shouldStartLoadWithRequest:(NSURLRequest *)request navigationType:(UIWebViewNavigationType)navigationType {
    NSURL *url = request.URL;
    NSString *urlStirng = url.absoluteString;
    NSLog(@"%@",urlStirng);
    return YES;
}
```
`UIButton`的`title`和`image`不能同时显示
`UINavigationItem`也是

不要再语言包里面设置空格

`NSNotificationCenter`带参数发送
```objective-c
MPMoviePlayerController *theMovie = [[MPMoviePlayerController alloc]initWithContentURL:[NSURL fileURLWithPath:moviePath]];
[[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(myMovieFinishedCallback:) name:MPMoviePlayerPlaybackDidFinishNotification object:theMovie];
[theMovie play];

- (void)myMovieFinishedCallback:(NSNotification *)aNotification {
    MPMoviePlayerController *theMovie = [aNotification object];
    [[NSNotificationCenter defaultCenter] removeObserver:self name:MPMoviePlayerPlaybackDidFinishNotification object:theMovie];
}
```
延迟一段时间执行某个函数
```objective-c
[self performSelector:@selector(dismissModal) withObject:self afterDelay:1.0];
```
用`NSDateFormatter`调整时间格式代码
```objective-c
NSDateFormatter *dateFormatter = [[NSDateFormatter alloc]init];
dateFormatter.dateFormat = @"yyyy-MM-dd HH:mm:ss";
NSString *currentDateStr = [dateFormatter stringFromDate:[NSDate date]];
```
`UIView`设置成圆角的方法
```objective-c
mainView.layer.cornerRadius = 6;
mainView.layer.masksToBounds = YES;
```
Objective-C 内存管理

1. 一个对象可以有一个或多个拥有者
2. 当它一个拥有者都没有的时候，它就会被回收
3. 如果想保留一个对象不被回收，你就必须成为它的拥有者

关键字

1. alloc
为对象分配内存，计数设为1，并返回此对象。
- copy
复制一个对象，此对象计数为1，返回此对象。你将成为此克隆对象的拥有者。
- retain
对象计数+1，并成为次对象的拥有者。
- release
对象计数-1，并丢掉此对象。
- autorelease
在未来的某一个时刻，对象计数-1。并在未来的某个时间放弃此对象。

原则

1. 一个代码块内要确保copy，alloc 和 retain 的使用数量与 release 和 autorelease 的数量相等。
- 在使用以 alloc 或 new 开头或包含 copy 的方法，或 retain 一个对象时，你将会编程它的拥有者。
- 实现 dealloc 方法，这是系统当 retain -> 0 的时候，自动调用的。手动调用会引起 retain count 计数错误（多一次的 release）。

iPhone 更改键盘右下角按键的 type
```objective-c
SearchBar *mySearchBar = [[UISearchBar alloc]init];
mySearchBar.frame = CGRectMake(0, 0, self.view.bounds.size.width, 44);
mySearchBar.placeholder = @"placeholderString";
mySearchBar.delegate = self;
[self.view addSubview:mySearchBar];
UITextField *searchField = [[mySearchBar subviews] lastObject];
searchField.returnKeyType = UIReturnKeyDone;
```
tableView左滑出多个按钮

```
- (NSArray *)tableView:(UITableView *)tableView editActionsForRowAtIndexPath:(NSIndexPath *)indexPath
{
    //删除按钮
    UITableViewRowAction *deleteRowAction = [UITableViewRowAction rowActionWithStyle:UITableViewRowActionStyleDefault title:@"删除" handler:^(UITableViewRowAction * _Nonnull action, NSIndexPath * _Nonnull indexPath){
        [_dataArray removeObjectAtIndex:indexPath.row];
        [tableView deleteRowsAtIndexPaths:@[indexPath]withRowAnimation:UITableViewRowAnimationAutomatic];
    }];
     
 
    //置顶按钮
    UITableViewRowAction *toTopRowAction = [UITableViewRowAction rowActionWithStyle:UITableViewRowActionStyleDefault title:@"置顶" handler:^(UITableViewRowAction * _Nonnull action, NSIndexPath * _Nonnull indexPath){
         
        NSLog(@"置顶");
         
    }];
    toTopRowAction.backgroundColor = [UIColor orangeColor];
     
    //其他按钮
    UITableViewRowAction *otherRowAction = [UITableViewRowAction rowActionWithStyle:UITableViewRowActionStyleDefault title:@"其他" handler:^(UITableViewRowAction * _Nonnull action, NSIndexPath * _Nonnull indexPath){
        NSLog(@"其他");
    }];
 
    otherRowAction.backgroundColor = [UIColor lightGrayColor];
 
    //返回按钮数组
    return @[deleteRowAction, toTopRowAction, otherRowAction];
}
```

给图片增加模糊效果

```
//加模糊效果，image是图片，blur是模糊度
+ (UIImage *)blurryImage:(UIImage *)image withBlurLevel:(CGFloat)blur {
    //模糊度,
    if ((blur < 0.1f) || (blur > 2.0f)) {
        blur = 0.1f;
    }
     
    //boxSize必须大于0
    int boxSize = (int)(blur * 100);
    boxSize -= (boxSize % 2) + 1;
    NSLog(@"boxSize:%i",boxSize);
    //图像处理
    CGImageRef img = image.CGImage;
     
    //图像缓存,输入缓存，输出缓存
    vImage_Buffer inBuffer, outBuffer;
    vImage_Error error;
    //像素缓存
    void *pixelBuffer;
     
    //数据源提供者，Defines an opaque type that supplies Quartz with data.
    CGDataProviderRef inProvider = CGImageGetDataProvider(img);
    // provider’s data.
    CFDataRef inBitmapData = CGDataProviderCopyData(inProvider);
     
    //宽，高，字节/行，data
    inBuffer.width = CGImageGetWidth(img);
    inBuffer.height = CGImageGetHeight(img);
    inBuffer.rowBytes = CGImageGetBytesPerRow(img);
    inBuffer.data = (void*)CFDataGetBytePtr(inBitmapData);
     
    //像数缓存，字节行*图片高
    pixelBuffer = malloc(CGImageGetBytesPerRow(img) * CGImageGetHeight(img));
     
    outBuffer.data = pixelBuffer;
    outBuffer.width = CGImageGetWidth(img);
    outBuffer.height = CGImageGetHeight(img);
    outBuffer.rowBytes = CGImageGetBytesPerRow(img);
     
     
    // 第三个中间的缓存区,抗锯齿的效果
    void *pixelBuffer2 = malloc(CGImageGetBytesPerRow(img) * CGImageGetHeight(img));
    vImage_Buffer outBuffer2;
    outBuffer2.data = pixelBuffer2;
    outBuffer2.width = CGImageGetWidth(img);
    outBuffer2.height = CGImageGetHeight(img);
    outBuffer2.rowBytes = CGImageGetBytesPerRow(img);
     
    //Convolves a region of interest within an ARGB8888 source image by an implicit M x N kernel that has the effect of a box filter.
    error = vImageBoxConvolve_ARGB8888(&inBuffer;, &outBuffer2;, NULL, 0, 0, boxSize, boxSize, NULL, kvImageEdgeExtend);
    error = vImageBoxConvolve_ARGB8888(&outBuffer2;, &inBuffer;, NULL, 0, 0, boxSize, boxSize, NULL, kvImageEdgeExtend);
    error = vImageBoxConvolve_ARGB8888(&inBuffer;, &outBuffer;, NULL, 0, 0, boxSize, boxSize, NULL, kvImageEdgeExtend);
     
     
    if (error) {
        NSLog(@"error from convolution %ld", error);
    }
     
    //    NSLog(@"字节组成部分：%zu",CGImageGetBitsPerComponent(img));
    //颜色空间DeviceRGB
    CGColorSpaceRef colorSpace = CGColorSpaceCreateDeviceRGB();
    //用图片创建上下文,CGImageGetBitsPerComponent(img),7,8
    CGContextRef ctx = CGBitmapContextCreate(
                                             outBuffer.data,
                                             outBuffer.width,
                                             outBuffer.height,
                                             8,
                                             outBuffer.rowBytes,
                                             colorSpace,
                                             CGImageGetBitmapInfo(image.CGImage));
     
    //根据上下文，处理过的图片，重新组件
    CGImageRef imageRef = CGBitmapContextCreateImage (ctx);
    UIImage *returnImage = [UIImage imageWithCGImage:imageRef];
     
    //clean up
    CGContextRelease(ctx);
    CGColorSpaceRelease(colorSpace);
     
    free(pixelBuffer);
    free(pixelBuffer2);
    CFRelease(inBitmapData);
    CGImageRelease(imageRef);
     
    return returnImage;
}
```

网易新闻头部滚动切换

```
- (void)titleButtonDidClick:(UIButton *)titleButton
{
    self.mainScrollView.contentOffset  = CGPointMake(kScreenWidth * titleButton.tag, 0);
    [UIView animateWithDuration:0.5 animations:^{
        if (self.titleScrollView.contentOffset.x >= 0) {
    self.titleScrollView.contentOffset = CGPointMake(titleButton.x - 150, 0);
        }
        // 先改变contentOffset，若偏移则修正
        if (self.titleScrollView.contentOffset.x <= 0) {
    self.titleScrollView.contentOffset = CGPointZero;
        }
        if (self.titleScrollView.contentOffset.x >= 2 * 900 - kScreenWidth) {
    self.titleScrollView.contentOffset = CGPointMake(2 * 900 - kScreenWidth, 0);
        }
    }];
}
```

七牛上传图片的简单封装

```
#import "QiNiuTool.h"
#import "JKRAccountTool.h"
#import "JKRAccount.h"
#import "QiniuSDK.h"
#import "QiNiuUpLoadResult.h"
#import <UIKit/UIKit.h>
 
@implementation QiNiuTool
 
+ (void)uploadImages:(NSArray *)images complete:(void (^)(NSDictionary *))complete
{
    QNUploadManager *qnm = [[QNUploadManager alloc] init];
    NSDateFormatter *mattter = [[NSDateFormatter alloc]init];
    [mattter setDateFormat:@"yyyyMMddHHmmss"];
    NSString *Nowdate = [mattter stringFromDate:[NSDate date]];
     
    QiNiuUpLoadResult *result = [[QiNiuUpLoadResult alloc] init];
     
    __block int count = 0;
     
    for (int i = 0; i < images.count; i++) {
         
        UIImage *image = images[i];
         
        NSData *uploadData = UIImageJPEGRepresentation(image, 1);
         
        //这里对大图片做了压缩，不需要的话下面直接传uploadData就好
        NSData *cutdownData = nil;
        if (uploadData.length < 9999) {
            cutdownData = UIImageJPEGRepresentation(image, 1.0);
        } else if (uploadData.length < 99999) {
            cutdownData = UIImageJPEGRepresentation(image, 0.6);
        } else {
            cutdownData = UIImageJPEGRepresentation(image, 0.3);
        }
         
        //[JKRAccountTool getAccount].token:token
        [qnm putData:cutdownData key:[NSString stringWithFormat:@"%@%d", Nowdate, i+1] token:[JKRAccountTool getAccount].token complete:^(QNResponseInfo *info, NSString *key, NSDictionary *resp) {
             
            count++;
             
            NSString *resultKey = [NSString stringWithFormat:@"%d",i];
             
            result.keys[resultKey] = [resp objectForKey:@"key"];
             
            if (count == images.count) {
                complete(result.keys);
            }
             
        } option:nil];
         
    }
     
}
 
@end

```

导航条渐隐渐现

```
[_tableView addObserver: self forKeyPath: @"contentOffset" options: NSKeyValueObservingOptionNew context:nil];
 
- (void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary<NSString *,id> *)change context:(void *)context{
    CGFloat offset=_tableView.contentOffset.y;
    NSLog(@"offset:%f",offset);//越来越大  想要实现效果：一开始全显示，后来颜色变浅
    CGFloat delta=1-offset/100.f;
    delta=MIN(1,delta);
    NSLog(@"%f",delta);
    self.navigationController.navigationBar.alpha=MIN(1,delta);
 
}

```
图片压缩

```
用法：UIImage *yourImage= [self imageWithImageSimple:image scaledToSize:CGSizeMake(210.0, 210.0)];
//压缩图片
- (UIImage*)imageWithImageSimple:(UIImage*)image scaledToSize:(CGSize)newSize
{
// Create a graphics image context
UIGraphicsBeginImageContext(newSize);
// Tell the old image to draw in this newcontext, with the desired
// new size
[image drawInRect:CGRectMake(0,0,newSize.width,newSize.height)];
// Get the new image from the context
UIImage* newImage = UIGraphicsGetImageFromCurrentImageContext();
// End the context
UIGraphicsEndImageContext();
// Return the new image.
return newImage;
}
```

图片裁剪，类似新浪微博小图显示效果

```

CGSize asize = CGSizeMake(300, 300);
    UIImage *newimage;
    UIImage *image = [UIImage imageNamed:@""];
    if (nil == image) {
        newimage = nil;
    }
    else{
        CGSize oldsize = image.size;
        CGRect rect;
        if (asize.width/asize.height > oldsize.width/oldsize.height) {
            rect.size.width = asize.width;
            rect.size.height = asize.width*oldsize.height/oldsize.width;
            rect.origin.x = 0;
            rect.origin.y = (asize.height - rect.size.height)/2;
        }
        else{
            rect.size.width = asize.height*oldsize.width/oldsize.height;
            rect.size.height = asize.height;
            rect.origin.x = (asize.width - rect.size.width)/2;
            rect.origin.y = 0;
        }
        UIGraphicsBeginImageContext(asize);
        CGContextRef context = UIGraphicsGetCurrentContext();
        CGContextClipToRect(context, CGRectMake(0, 0, asize.width, asize.height));
        CGContextSetFillColorWithColor(context, [[UIColor clearColor] CGColor]);
        UIRectFill(CGRectMake(0, 0, asize.width, asize.height));//clear background
        [image drawInRect:rect];
        newimage = UIGraphicsGetImageFromCurrentImageContext();
        UIGraphicsEndImageContext();
    }
     
    UIImageView * imgView = [[UIImageView alloc]initWithFrame:CGRectMake(10, 10, 100, 100)];
    imgView.image = newimage;
    [self.view addSubview:imgView];
```
把时间戳转换为时间

```
+ (NSDate *)dateWithTimeIntervalInMilliSecondSince1970:(double)timeIntervalInMilliSecond {
    NSDate *ret = nil;
    double timeInterval = timeIntervalInMilliSecond;
    // judge if the argument is in secconds(for former data structure).
    if(timeIntervalInMilliSecond > 140000000000) {
        timeInterval = timeIntervalInMilliSecond / 1000;
    }
    ret = [NSDate dateWithTimeIntervalSince1970:timeInterval];
     
    return ret;
}
```

自定义cell中获取不到cell实际大小的办法
```
-(void)drawRect:(CGRect)rect {
    // 重写此方法，并在此方法中获取
    CGFloat width = self.frame.size.width;
}
```

长按图标抖动

```
-(void)longPress:(UILongPressGestureRecognizer*)longPress
{
    if (longPress.state==UIGestureRecognizerStateBegan) {
        CAKeyframeAnimation* anim=[CAKeyframeAnimation animation];
        anim.keyPath=@"transform.rotation";
        anim.values=@[@(angelToRandian(-7)),@(angelToRandian(7)),@(angelToRandian(-7))];
        anim.repeatCount=MAXFLOAT;
        anim.duration=0.2;
        [self.imageView.layer addAnimation:anim forKey:nil];
        self.btn.hidden=NO;
    }
}
```

阿拉伯数字转化为汉语数字

```
+(NSString *)translation:(NSString *)arebic
 
{   NSString *str = arebic;
    NSArray *arabic_numerals = @[@"1",@"2",@"3",@"4",@"5",@"6",@"7",@"8",@"9",@"0"];
    NSArray *chinese_numerals = @[@"一",@"二",@"三",@"四",@"五",@"六",@"七",@"八",@"九",@"零"];
    NSArray *digits = @[@"个",@"十",@"百",@"千",@"万",@"十",@"百",@"千",@"亿",@"十",@"百",@"千",@"兆"];
    NSDictionary *dictionary = [NSDictionary dictionaryWithObjects:chinese_numerals forKeys:arabic_numerals];
     
    NSMutableArray *sums = [NSMutableArray array];
    for (int i = 0; i < str.length; i ++) {
        NSString *substr = [str substringWithRange:NSMakeRange(i, 1)];
        NSString *a = [dictionary objectForKey:substr];
        NSString *b = digits[str.length -i-1];
        NSString *sum = [a stringByAppendingString:b];
        if ([a isEqualToString:chinese_numerals[9]])
        {
            if([b isEqualToString:digits[4]] || [b isEqualToString:digits[8]])
            {
                sum = b;
                if ([[sums lastObject] isEqualToString:chinese_numerals[9]])
                {
                    [sums removeLastObject];
                }
            }else
            {
                sum = chinese_numerals[9];
            }
             
            if ([[sums lastObject] isEqualToString:sum])
            {
                continue;
            }
        }
         
        [sums addObject:sum];
    }
     
    NSString *sumStr = [sums  componentsJoinedByString:@""];
    NSString *chinese = [sumStr substringToIndex:sumStr.length-1];
    NSLog(@"%@",str);
    NSLog(@"%@",chinese);
    return chinese;
}
```

两种方法删除NSUserDefaults所有记录

```
//方法一
NSString *appDomain = [[NSBundle mainBundle] bundleIdentifier];
[[NSUserDefaults standardUserDefaults] removePersistentDomainForName:appDomain];
 
//方法二
- (void)resetDefaults {
    NSUserDefaults * defs = [NSUserDefaults standardUserDefaults];
    NSDictionary * dict = [defs dictionaryRepresentation];
    for (id key in dict) {
        [defs removeObjectForKey:key];
    }
    [defs synchronize];
}
```
截屏 全图

```
- (UIImage *)imageFromView: (UIView *) theView
{
     
    UIGraphicsBeginImageContext(theView.frame.size);
    CGContextRef context = UIGraphicsGetCurrentContext();
    [theView.layer renderInContext:context];
    UIImage *theImage = UIGraphicsGetImageFromCurrentImageContext();
    UIGraphicsEndImageContext();
     
    return theImage;
}
```

判断是否用户开启了定位服务

```
if ([CLLocationManager locationServicesEnabled] &&  
                ([CLLocationManager authorizationStatus] == kCLAuthorizationStatusAuthorized  
                || [CLLocationManager authorizationStatus] == kCLAuthorizationStatusNotDetermined)) {  
                //定位功能可用，开始定位  
                _locationManger = [[CLLocationManager alloc] init];  
                locationManger.delegate = self;  
                [locationManger startUpdatingLocation];  
            }  
            else if ([CLLocationManager authorizationStatus] == kCLAuthorizationStatusDenied){  
        NSlog("定位功能不可用，提示用户或忽略");   
            }
            
            
         
```

图片旋转

```

UIImage * image = [UIImage imageNamed:@"iphone.png"];
NSData * tempData;
if (UIImagePNGRepresentation(image)) {
    tempData = UIImagePNGRepresentation(image);
    NSLog(@"%@",tempData);
}
else{
    tempData = UIImageJPEGRepresentation(image, 1);
}
CIImage * iImg = [CIImage imageWithData:tempData];
UIImage * tImg = [UIImage imageWithCIImage:iImg scale:1 orientation:UIImageOrientationRight];
NSLog(@"%@",UIImagePNGRepresentation(tImg));
UIImageView * imageView = [[UIImageView alloc]initWithFrame:CGRectMake(10, 200, 200, 80)];
imageView.image = tImg;
[self.view addSubview:imageView];
```

去除UIImageView锯齿

```
	
imageView.layer.shouldRasterize = YES;

```

判断两个日期之间的间隔

```
NSDateFormatter * dateFormatter = [[NSDateFormatter alloc] init];
[dateFormatter setDateFormat:@"yyyyMMddHHmmss"];
NSDate* toDate     = [dateFormatter dateFromString:@"19700608142033"];
NSDate*  startDate    = [ [ NSDate alloc] init ];
NSCalendar* chineseClendar = [ [ NSCalendar alloc ] initWithCalendarIdentifier:NSGregorianCalendar ];
NSUInteger unitFlags = NSHourCalendarUnit | NSMinuteCalendarUnit | NSSecondCalendarUnit | NSDayCalendarUnit | NSMonthCalendarUnit | NSYearCalendarUnit;
 
NSDateComponents *cps = [ chineseClendar components:unitFlags fromDate:startDate  toDate:toDate  options:0];
 
NSInteger diffYear = [cps year];
NSInteger diffMon = [cps month];
NSInteger diffDay = [cps day];
NSInteger diffHour = [cps hour];
NSInteger diffMin = [cps minute];
NSInteger diffSec = [cps second];
 
NSLog(@" From Now to %@, diff: Years: %d  Months: %d, Days; %d, Hours: %d, Mins:%d, sec:%d", [toDate description], diffYear, diffMon, diffDay, diffHour, diffMin,diffSec );
```

类似微信发送视频的流程

```
//获取视频的本地url或者path，对视频进行获取第一帧图片，然后初始化消息的Model，设置封面

NSString *mediaType = [editingInfo objectForKey: UIImagePickerControllerMediaType];
                NSString *videoPath;
                NSURL *videoUrl;
                if (CFStringCompare ((__bridge CFStringRef) mediaType, kUTTypeMovie, 0) == kCFCompareEqualTo) {
                    videoUrl = (NSURL*)[editingInfo objectForKey:UIImagePickerControllerMediaURL];
                    videoPath = [videoUrl path];
                     
                    AVURLAsset *asset = [[AVURLAsset alloc] initWithURL:videoUrl options:nil];
                    NSParameterAssert(asset);
                    AVAssetImageGenerator *assetImageGenerator = [[AVAssetImageGenerator alloc] initWithAsset:asset];
                    assetImageGenerator.appliesPreferredTrackTransform = YES;
                    assetImageGenerator.apertureMode = AVAssetImageGeneratorApertureModeEncodedPixels;
                     
                    CGImageRef thumbnailImageRef = NULL;
                    CFTimeInterval thumbnailImageTime = 0;
                    NSError *thumbnailImageGenerationError = nil;
                    thumbnailImageRef = [assetImageGenerator copyCGImageAtTime:CMTimeMake(thumbnailImageTime, 60) actualTime:NULL error:&thumbnailImageGenerationError;];
                     
                    if (!thumbnailImageRef)
                        NSLog(@"thumbnailImageGenerationError %@", thumbnailImageGenerationError);
                     
                    UIImage *thumbnailImage = thumbnailImageRef ? [[UIImage alloc] initWithCGImage:thumbnailImageRef] : nil;
                     
                    XHMessage *videoMessage = [[XHMessage alloc] initWithVideoConverPhoto:thumbnailImage videoPath:videoPath videoUrl:nil sender:@"Jack" timestamp:[NSDate date]];
                    [weakSelf addMessage:videoMessage];
                }

```

刷新某行cell的方法

```
//有时候只需要刷新某行的cell的数据，完全没必要调用[tableView reloadData]刷新整个列表的数据，调用以下方法即可。

NSIndexPath *indexPath_1=[NSIndexPath indexPathForRow:1 inSection:0];
      NSArray *indexArray=[NSArray  arrayWithObject:indexPath_1];
      [myTableView  reloadRowsAtIndexPaths:indexArray withRowAnimation:UITableViewRowAnimationAutomatic];

```
由身份证号码返回性别

```
-(NSString *)sexStrFromIdentityCard:(NSString *)numberStr{
    NSString *result = nil;
     
    BOOL isAllNumber = YES;
     
    if([numberStr length]<17)
        return result;
     
    //**截取第17为性别识别符
    NSString *fontNumer = [numberStr substringWithRange:NSMakeRange(16, 1)];
     
    //**检测是否是数字;
    const char *str = [fontNumer UTF8String];
    const char *p = str;
    while (*p!='\0') {
        if(!(*p>='0'&&*p<='9'))
            isAllNumber = NO;
        p++;
    }
     
    if(!isAllNumber)
        return result;
     
    int sexNumber = [fontNumer integerValue];
    if(sexNumber%2==1)
        result = @"男";
    ///result = @"M";
    else if (sexNumber%2==0)
        result = @"女";
    //result = @"F";
     
    return result;
     
     
}
```
数组随机重新排列

```
+ (NSArray *)getRandomWithPosition:(NSInteger)position positionContent:(id)positionContent array:(NSArray *)baseArray {
    NSMutableArray *resultArray = [NSMutableArray arrayWithCapacity:baseArray.count];
    NSMutableArray *tempBaseArray = [NSMutableArray arrayWithArray:baseArray];
     
    while ([tempBaseArray count]) {
        NSInteger range = [tempBaseArray count];
        id string = [tempBaseArray objectAtIndex:arc4random()%range];
        [resultArray addObject:string];
        [tempBaseArray removeObject:string];
    }
     
    NSUInteger index = [resultArray indexOfObject:positionContent];
    [resultArray exchangeObjectAtIndex:index withObjectAtIndex:position - 1];
     
    return resultArray;
}
```
利用陀螺仪实现更真实的微信摇一摇动画

```
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
{    
    //想摇你的手机嘛？就写在这，然后，然后，没有然后了
    application.applicationSupportsShakeToEdit=YES;
}
 
-(void)motionEnded:(UIEventSubtype)motion withEvent:(UIEvent *)event {
    if(motion==UIEventSubtypeMotionShake) {
         // 真实一点的摇动动画
        [self addAnimations];
         // 播放声音
        AudioServicesPlaySystemSound (soundID); 
    }
}
 
- (void)addAnimations {
    CABasicAnimation *translation = [CABasicAnimation animationWithKeyPath:@"transform"];
    translation.timingFunction = [CAMediaTimingFunction functionWithName:kCAMediaTimingFunctionEaseInEaseOut];
    translation.toValue=[NSValue valueWithCATransform3D:CATransform3DMakeRotation(-M_PI_4, 0, 0, 100)];
    translation.duration = 0.2;
    translation.repeatCount = 2;
    translation.autoreverses = YES;
     
    [shake.layer addAnimation:translation forKey:@"translation"];
}
```
GIF图片解析

```
+ (NSMutableArray *)praseGIFDataToImageArray:(NSData *)data;
 {
    NSMutableArray *frames = [[NSMutableArray alloc] init];
    CGImageSourceRef src = CGImageSourceCreateWithData((CFDataRef)data, NULL);
    CGFloat animationTime = 0.f;
    if (src) {
        size_t l = CGImageSourceGetCount(src);
        frames = [NSMutableArray arrayWithCapacity:l];
        for (size_t i = 0; i < l; i++) {
            CGImageRef img = CGImageSourceCreateImageAtIndex(src, i, NULL);
            NSDictionary *properties = (NSDictionary *)CGImageSourceCopyPropertiesAtIndex(src, i, NULL);
            NSDictionary *frameProperties = [properties objectForKey:(NSString *)kCGImagePropertyGIFDictionary];
            NSNumber *delayTime = [frameProperties objectForKey:(NSString *)kCGImagePropertyGIFUnclampedDelayTime];
            animationTime += [delayTime floatValue];
            if (img) {
                [frames addObject:[UIImage imageWithCGImage:img]];
                CGImageRelease(img);
            }
        }
        CFRelease(src);
    }
     return frames;
}
```

在后台播放音乐

```
//1. 在Info.plist中，添加"Required background modes"键，其值设置是“App plays audio" 
//2. 在播放器播放音乐的代码所在处，添加如下两段代码（当然，前提是已经添加了AVFoundation框架）

//添加后台播放代码：
AVAudioSession *session = [AVAudioSession sharedInstance];    
[session setActive:YES error:nil];    
[session setCategory:AVAudioSessionCategoryPlayback error:nil];   
 
//以及设置app支持接受远程控制事件代码。设置app支持接受远程控制事件，
//其实就是在dock中可以显示应用程序图标，同时点击该图片时，打开app。
//或者锁屏时，双击home键，屏幕上方出现应用程序播放控制按钮。
[[UIApplication sharedApplication] beginReceivingRemoteControlEvents]; 
 
 
//用下列代码播放音乐，测试后台播放
// 创建播放器  
AVAudioPlayer *player = [[AVAudioPlayer alloc] initWithContentsOfURL:url error:nil];  
[url release];  
[player prepareToPlay];  
[player setVolume:1];  
player.numberOfLoops = -1; //设置音乐播放次数  -1为一直循环  
[player play]; //播放
```

