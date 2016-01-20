前言
----------

iOS开发过程中，总有那么一些个小问题让人纠结，它们不会让程序崩溃，但是会让人崩溃。除此之外，还将分享一些细节现在我通过自己的总结以及从其他地方的引用，来总结一下一些常见小问题。

本篇长期更新，多积累，多奉献，同时感谢其中一些文章的作者的整理，感谢！

iOS高级开发实战讲解
----------
这是我在网上搜索到的[iOS高级开发实战讲解][1]，由于原文不是很方便浏览，所以我在这里整理一部分出来，方便查阅，同时谢谢原作者。

这里我不是每一个都收录进来，这里只是放出一部分，有些用的太多，我就没整理了，大家如果想看可以去看原文。

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


