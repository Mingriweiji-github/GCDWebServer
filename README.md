###2017.8.15日更新
##GCDWebServer 
###摘要

####关键词：iOS服务器框架，基于GCD
GCDWebServer是一个现代和轻量级的基于 HTTP 1.1的服务器，它的设计旨在嵌入OS X和iOS应用程序中。它的实现在一开始就考虑了以下目标：

+ 一个优雅轻巧的使用架构带有四个核心的类：请求类，连接类，请求类和响应类（详情请参阅“了解GCDWebServer的架构”下）。
+  一个精心设计的可以轻松集成和定制完整的方便查看的头文件
+  完全使用基于事件驱动的[Grand Central Dispatch](https://en.wikipedia.org/wiki/Grand_Central_Dispatch)以求最佳性能和并发能力
+  不依赖任何第三方的源码
+  新的友好的BSD许可证下可使用

####其他的内置功能
+ 实现-对于进入的HTTP请求采用完全异步的处理手段
+ 最小化内存消耗-尽可能把大量的HTTP请求和响应体放在磁盘流中
+ 使用 "application/x-www-form-urlencoded" 或者 "multipart/form-data" 编码 解析网站表单提交
+  对HTTP体的请求和响应采用JSON解析和序列化
+  对HTTP体的请求和响应 分块传输解码
+  对HTTP体的请求和响应 使用gzip对HTTP进行压缩
+  对请求的本地文件的范围也支持
+  密码保护有基本的[摘要接入认证-维基百科](https://translate.google.com.hk/translate?hl=zh-CN&sl=en&u=https://en.wikipedia.org/wiki/Digest_access_authentication&prev=search)
+  自动处理iOS apps前台，后台和假死状态的转换过渡
+  全面支持IPv4和IPv6
+  NAT端口映射（仅限IPv4）

#####扩展
+  GCDWebUploader ： GCDWebServer的子类，使用Web浏览器实现 上传和下载文件的接口
+  GCDWebDAVServer ： GCDWebServer的子类，实现1级WebDAV服务器（与OS X的Finder部分2级支持）

#####暂不支持（非一个嵌入式HTTP服务器必须的）
+ 长连接
+ HTTPS

####使用条件
+ OS X 10.7以后（X86_64）
+ iOS 5.0之后（armv7,armv7s或者arm64）
+ ARC的内存管理(MRC使用GCDWebServer 3.1和更早版本)

####开始使用
下载或查看[GCDWebServer](https://github.com/swisspol/GCDWebServer)的最新版本，直接添加整个“ GCDWebServer ”子文件夹到Xcode项目。如果您打算使用像GCDWebDAVServer或GCDWebUploader扩展之一，还要添加这些子文件夹。
+ 1.如果是直接手动添加[GCDWebServer](https://github.com/swisspol/GCDWebServer)，那么第一步需要添加动态库libz(路径： Target > Build Phases > Link Binary With Libraries) 第二步添加 $(SDKROOT)/usr/include/libxml2)到header search paths(路径：arget > Build Settings > HEADER_SEARCH_PATHS)；

+ 2.安装使用[CocoaPods](https://cocoapods.org/)可以简单的添加（通过添加下面到Podfile中即可）：

	pod "GCDWebServer", "~> 3.0"	
	
+ 如果你还想使用GCDWebUploader ，使用该行：

	pod "GCDWebServer/WebUploader", "~> 3.0"
	
+ GCDWebDAVServer使用下面：

	pod "GCDWebServer/WebDAV", "~> 3.0"
	
最后终端输入命令：$ pod install

您还可以使用由[Carthage](https://github.com/Carthage/Carthage)加入这一行到你的Cartfile （ 3.2.5第一次正式发布与Carthage支持） ：

	github "swisspol/GCDWebServer" ~> 3.2.5
	
然后终端输入命令 $ carthage update 添加生成的frameworks到你的xcode项目中即可（详情查看[Carthage instructions](https://github.com/Carthage/Carthage#adding-frameworks-to-an-application)  ）

###帮助与支持
有关使用GCDWebServer的帮助下，最好把你的问题上在StackOverflow贴上GCDWebserver标签。

请务必先虽然阅读整本自述！
###Hello World
下面代码介绍了如何实现在8080端口上运行，而且无论任何请求服务器都会返回一个“ Hello World”的HTML页面。

由于GCDWebServer使用GCD块来处理请求，没有使用子类或者代理，因此代码整体看起来很干净。


#####重要提示：如果不使用的CocoaPods ，务必在项目中将libz进行系统共享库添加到Xcode的target中。

OS X版本（命令行工具） ：


	 #import "GCDWebServer.h"
	 #import "GCDWebServerDataResponse.h"
	
	int main(int argc, const char* argv[]) {
	  @autoreleasepool {
	
	    // Create server
	    GCDWebServer* webServer = [[GCDWebServer alloc] init];
	
	    // Add a handler to respond to GET requests on any URL
	    [webServer addDefaultHandlerForMethod:@"GET"
	                             requestClass:[GCDWebServerRequest class]
	                             processBlock:^GCDWebServerResponse *(GCDWebServerRequest* request) {
	
	      return [GCDWebServerDataResponse responseWithHTML:@"<html><body><p>Hello World</p></body></html>"];
	
	    }];
	
	    // Use convenience method that runs server on port 8080
	    // until SIGINT (Ctrl-C in Terminal) or SIGTERM is received
	    [webServer runWithPort:8080 bonjourName:nil];
	    NSLog(@"Visit %@ in your web browser", webServer.serverURL);
	
	  }
	  return 0;
	}

iOS版本:

	 #import "GCDWebServer.h"
	 #import "GCDWebServerDataResponse.h"
	
	@interface AppDelegate : NSObject <UIApplicationDelegate> {
	  GCDWebServer* _webServer;
	}
	@end
	
	@implementation AppDelegate
	
	- (BOOL)application:(UIApplication*)application didFinishLaunchingWithOptions:(NSDictionary*)launchOptions {
	
	  // Create server
	  _webServer = [[GCDWebServer alloc] init];
	
	  // Add a handler to respond to GET requests on any URL
	  [_webServer addDefaultHandlerForMethod:@"GET"
	                            requestClass:[GCDWebServerRequest class]
	                            processBlock:^GCDWebServerResponse *(GCDWebServerRequest* request) {
	
	    return [GCDWebServerDataResponse responseWithHTML:@"<html><body><p>Hello World</p></body></html>"];
	
	  }];
	
	  // Start server on port 8080
	  [_webServer startWithPort:8080 bonjourName:nil];
	  NSLog(@"Visit %@ in your web browser", _webServer.serverURL);
	
	  return YES;
	}
	
	@end
	
OS X Swift（命令行工具） ：

#####webServer.swift
	import Foundation
	import GCDWebServers
	
	func initWebServer() {
	
	    let webServer = GCDWebServer()
	
	    webServer.addDefaultHandlerForMethod("GET", requestClass: GCDWebServerRequest.self, processBlock: {request in
	    return GCDWebServerDataResponse(HTML:"<html><body><p>Hello World</p></body></html>")
	
	    })
	
	    webServer.runWithPort(8080, bonjourName: "GCD Web Server")
	
	    print("Visit \(webServer.serverURL) in your web browser")
	}


#####WebServer-Bridging-Header.h

	 #import <GCDWebServers/GCDWebServer.h>
	 #import <GCDWebServers/GCDWebServerDataResponse.h>

	
####iOS Apps中基于Web 的上传

GCDWebUploader是GCDWebServer的子类，提供了一个现成供 HTML5文件上传下载。GCDWebUploader让用户可以在浏览器里使用一个干净的UI来上传，下载，删除文件，以及在iOS应用的沙盒中的目录创建目录。


从你的浏览器中简单地实例化GCDWebUploader和运行然后访问 http://{YOUR-IOS-DEVICE-IP-ADDRESS}/ 

	 #import "GCDWebUploader.h"
	
	@interface AppDelegate : NSObject <UIApplicationDelegate> {
	  GCDWebUploader* _webUploader;
	}
	@end
	
	@implementation AppDelegate
	
	- (BOOL)application:(UIApplication*)application didFinishLaunchingWithOptions:(NSDictionary*)launchOptions {
	  NSString* documentsPath = [NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES) firstObject];
	  _webUploader = [[GCDWebUploader alloc] initWithUploadDirectory:documentsPath];
	  [_webUploader start];
	  NSLog(@"Visit %@ in your web browser", _webUploader.serverURL);
	  return YES;
	}
	
	@end
	


#####在iOS应用WebDAV服务器

GCDWebDAVServer 是GCDWebServer的子类提供了一个兼容的[WebDAV](https://en.wikipedia.org/wiki/WebDAV)服务器。 使用任何的WebDAV服务器像[Transmit](https://panic.com/transmit/)(Mac),[ForkLift](http://binarynights.com/forklift/) (Mac) 或者 [CyberDuck](https://cyberduck.io/)(Mac / Windows)一样可以在ISO沙盒目录中让用户上传，下载，删除或者创建目录文件。

简单地实例化和运行GCDWebDAVServer，然后使用WebDAV连接到 http://{自己的IOS设备的IP地址}/ :

	 #import "GCDWebDAVServer.h"
	
	@interface AppDelegate : NSObject <UIApplicationDelegate> {
	  GCDWebDAVServer* _davServer;
	}
	@end
	
	@implementation AppDelegate
	
	- (BOOL)application:(UIApplication*)application didFinishLaunchingWithOptions:(NSDictionary*)launchOptions {
	  NSString* documentsPath = [NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES) firstObject];
	  _davServer = [[GCDWebDAVServer alloc] initWithUploadDirectory:documentsPath];
	  [_davServer start];
	  NSLog(@"Visit %@ in your WebDAV client", _davServer.serverURL);
	  return YES;
	}
	
	@end





####Serving a Static Website 服务于静态网站
GCDWebServer有一个固定的handler用来递归式的服务一个目录（一个容许你控制并设置HTTP header的 "Cache-control"，Cache-control用于控制HTTP缓存）

####OS X 版（command line tool）:

 #import "GCDWebServer.h"

int main(int argc, const char* argv[]) {
  @autoreleasepool {

    GCDWebServer* webServer = [[GCDWebServer alloc] init];
    [webServer addGETHandlerForBasePath:@"/" directoryPath:NSHomeDirectory() indexFilename:nil cacheAge:3600 allowRangeRequests:YES];
    [webServer runWithPort:8080];
      }
       return 0;
     }
---     
### 使用 GCDWebServer
你可以通过创建一个实例 GCDWebServer类开始。注意：你可以在同一个应用程序上运行多个web服务器只要这些web服务器是挂在不同的端口。
然后你可以添加一个或者多个“处理”给服务器：每一个处理器都有机会去处理传入的Web请求并且提供响应。处理程序我们叫做LIFO队列，所以最新加入的“处理器”会覆写之前所有添加的“处理程序”。
 
###理解 GCDWebServer的架构

GCDWebServer的体系结构包括只有4个核心类：
GCDWebServer 负责管理监听新的HTTP连接和服务器使用的一系列的处理程序列表 的接口socket.

GCDWebServerConnection 是由GCDWebServer来实例化处理每一个新的HTTP连接。每一个GCDWebServerConnection实例一直保持活跃状态一直到连接被关闭！ 你可不能直接使用这个类，但是由于它是暴露的所以你可以继承至它来覆写一些hooks.

GCDWebServerRequest 由GCDWebServerConnection在接收到HTTP 表头后实例化 创建。 用它来包装请求和处理HTTP主体（如果有主体的话）。GCDWebServer 包含了几个GCDWebServerRequest的子类来处理常见情况下的情况：如存储 body 到内存或者传输到磁盘的一个文件中。

GCDWebServerResponse 由请求处理器创建 和 包装该响应HTTP headers和一些可选择的body. GCDWebServer 也是通过由几个GCDWebServerResponse的子类来处理常见的情况的如内存中的HTML文本或者从磁盘来传输一个文件时，

###GCDWebServer 实现

GCDWebServer的实现依赖于“处理程序” 来处理传入的Web请求并且做出响应。 “处理程序”通过GCD块来实现的使得GCDWebServer方便你使用。然而，由于在GCD中“处理程序”是在任意的线程中执行，所以要注意的是线程安全和同一程序重复执行问题。

处理程序用到2个GCD块：
GCDWebServerMatchBlock 会被 在添加到GCDWebServer中的每一个"处理程序"所调用只要一个web请求已经开始后（如HTTP 表头已经收到）。它可以传递Web请求的基本信息（HTTP method, URL, headers...）而且必须要决定是否会处理这个请求。  如果返回yes，那么必须要返回一个新的这有这些信息的GCDWebServerRequest实例（看上面的GCDWebServerRequest），否则它只是返回nil.

之后，Web请求已经被完全接收并传递在上一步创建的GCDWebServerRequest实例GCDWebServerProcessBlock或GCDWebServerAsyncProcessBlock被调用。它必须返回同步（如果使用GCDWebServerProcessBlock）或异步（如果使用GCDWebServerAsyncProcessBlock）一个GCDWebServerResponse实例（见上文）或nil ，nil返回一个500的HTTP状态代码返回给客户端。一般我们推荐针对错误 返回一个GCDWebServerErrorResponse实例 以便可以把更多有用信息返回给客户端。
######请注意：GCDWebServer的大多数方法来添加handlers只需要GCDWebServerProcessBlock或GCDWebServerAsyncProcessBlock ,因为它们已经提供了一种内置位于GCDWebServerMatchBlock中--例如 通过正则Regex匹配URL路径。

---



######异步响应HTTP

GCDWebServer 3.0新增特性是异步地处理HTTP请求，如增加了 请求服务器后可以异步生成GCDWebServerResponse 的handlers，
这一过程通过使用GCDWebServerAsyncProcessBlock代替GCDWebServerProcessBlock的处理程序来实现。下面是一个例子：


######（同步版）handler在响应HTTP后会产生blocks回调：

		[webServer addDefaultHandlerForMethod:@"GET"
		                         requestClass:[GCDWebServerRequest class]
		                         processBlock:^GCDWebServerResponse *(GCDWebServerRequest* request) {
		
		  GCDWebServerDataResponse* response = [GCDWebServerDataResponse responseWithHTML:@"<html><body><p>Hello World</p></body></html>"];
		  return response;
		
		}];

######（异步版本） handler会立即返回结果并会在响应HTTP后回调GCDWebServer

	[webServer addDefaultHandlerForMethod:@"GET"
	                         requestClass:[GCDWebServerRequest class]
	                    asyncProcessBlock:^(GCDWebServerRequest* request, GCDWebServerCompletionBlock completionBlock) {
	
	  // Do some async operation like network access or file I/O (simulated here using dispatch_after())
	  dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(3 * NSEC_PER_SEC)), dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
	    GCDWebServerDataResponse* response = [GCDWebServerDataResponse responseWithHTML:@"<html><body><p>Hello World</p></body></html>"];
	    completionBlock(response);
	  });
	
	}];

######（高级异步版本）handler会立即返回一个HTTP本身异步内容流的HTTP响应：

	[webServer addDefaultHandlerForMethod:@"GET"
	                         requestClass:[GCDWebServerRequest class]
	                         processBlock:^GCDWebServerResponse *(GCDWebServerRequest* request) {
	
	  NSMutableArray* contents = [NSMutableArray arrayWithObjects:@"<html><body><p>\n", @"Hello World!\n", @"</p></body></html>\n", nil];  // Fake data source we are reading from
	  GCDWebServerStreamedResponse* response = [GCDWebServerStreamedResponse responseWithContentType:@"text/html" asyncStreamBlock:^(GCDWebServerBodyReaderCompletionBlock completionBlock) {
	
	    // Simulate a delay reading from the fake data source
	    dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(1 * NSEC_PER_SEC)), dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
	      NSString* string = contents.firstObject;
	      if (string) {
	        [contents removeObjectAtIndex:0];
	        completionBlock([string dataUsingEncoding:NSUTF8StringEncoding], nil);  // Generate the 2nd part of the stream data
	      } else {
	        completionBlock([NSData data], nil);  // Must pass an empty NSData to signal the end of the stream
	      }
	    });
	
	  }];
	  return response;
	
	}];
######注意：甚至你可以结合异步的和高级异步这两个版本去异步返回一个异步的HTTP响应！
***********************

####GCDWebServer &  iOS APPs的后台运行模式
 当iOS APP在进行网络请求时，我们必须注意当我们把应用程序压入后台后会发生什么情况。 一般情况下，应用程序进入后台我们应该停止网络请求当应用程序再次进入前台时我们再重新开始网络请求。
这使得情况可能会较为复杂，考虑到当服务器需要停止服务时他们仍然在保持着不断连接的状态。
	幸运的事是GCDWebServer会代替我们自动完成以上的工作：
	
+ GCDWebServer 会开启后台模式，当第一次HTTP连接开始至最后一个HTTP请求结束，这样就阻止了应用系统一旦进入后台就会被iOS系统杀死，而且也会把HTTP连接一起杀死。

+ 当我们的应用程序在后台时，只要新的HTTP连接一直保持创建，后台任务就会持续进行，iOS系统就不会kill我们的应用程序(除了app很快的意外的内存升高问题).
+ 如果应用程序一直保持后台运行当一个HTTP请求都没有打开，GCDWebServer会立即结束自己  而且  一旦调用了 -stop方法就会停止接收新的HTTP请求(这一模式在 GCDWebServerOption_AutomaticallySuspendInBackground 选项中无效).
+如果app进入了前台模式而且GCDWebServer是在暂停期间， 一旦调用了 -start 方法，GCDWebServer会自动的结束自己暂停状态开始接收新的HTTP连接。

+ HTTP连接经常是分批次（或者突然性）发起的，例如当加载一个带有多个资源文件的网页时就会这样。 当很靠后的HTTP 连接已经被关闭时就很难去精确地检测到了：它可能同一批的2个连续的HTTP连接部分很短的时间的延迟，而不是覆盖。如果GCDWebServer在它们之间的时间缝隙里停止在客户端了就不太好了（估计这样的话效率就低了），GCDWebServerOption_ConnectedStateCoalescingInterval这个enum值通过优雅的强制GCDWebServer 在每一个HTTP连接结束后等待一定时间的间隔，以防一个新的HTTP请求会在这个很短的时间间隔里被再次创建。

####GCDWebServer日志
无论出于调试还是情报的目的， GCDWebServer无论何时都会记录messages。此外，在“调试”模式与“释发布”模式建设GCDWebServer时，它会记录更多的信息，但也进行了一些内部的重复性内容检查。要启用此行为，编译时GCDWebServer定义预处理器时设置常量DEBUG = 1 。在Xcode目标设置，这可以通过增加DEBUG = 1 开启 “调试”模式和 设置GCC_PREPROCESSOR_DEFINITIONS完成。最后，您还可以通过运行时调用类方法 +[GCDWebServer setLogLevel:]来完成。

+ 默认情况下， GCDWebServer记录的所有消息都发送到其内置的日志记录功能，它只是输出到stderr （假设终端设备连接了） 。为了更好地依靠日志的记录来完善APP，你可能想使用另一种记录工具。

+ GCDWebServer 对XLFacility自动支持（GCDWebServer的同一个作者而且开源）和CocoaLumberjack 。如果其中一方是在同一个Xcode项目， GCDWebServer会自动用它代替内置的日志记录功能（见GCDWebServerPrivate.h的实施细节）。

它也可以使用自定义日志记录功能 - 见GCDWebServer.h获取更多信息。

#####高级示例1 ：实现HTTP重定向

下面是重定向的例子处理程序“/ ”到“/index.htm“，使用的GCDWebServerResponse的简便方法（它会自动设置HTTP状态代码和”位置“标头） ：

	[self addHandlerForMethod:@"GET"
	                     path:@"/"
	             requestClass:[GCDWebServerRequest class]
	             processBlock:^GCDWebServerResponse *(GCDWebServerRequest* request) {
	
	  return [GCDWebServerResponse responseWithRedirect:[NSURL URLWithString:@"index.html" relativeToURL:request.URL]
	                                          permanent:NO];
	
	}];
	
	
#####高级示例2 ：实现形式
要实现HTTP表单，你需要两个处理程序：

+ 一个获取处理程序不期望在HTTP请求任何机构，因此使用GCDWebServerRequest类。处理程序生成包含一个简单的HTML表单的响应。
+ 另一个POST处理器期望表单值是在HTTP请求和百分比编码的主体。幸运的是， GCDWebServer提供请求类GCDWebServerURLEncodedFormRequest能够自动解析这样的主题。处理程序简单地显示对应的用户提交的表单中的值。

		[webServer addHandlerForMethod:@"GET"
		                          path:@"/"
		                  requestClass:[GCDWebServerRequest class]
		                  processBlock:^GCDWebServerResponse *(GCDWebServerRequest* request) {
		
		  NSString* html = @" \
		    <html><body> \
		      <form name=\"input\" action=\"/\" method=\"post\" enctype=\"application/x-www-form-urlencoded\"> \
		      Value: <input type=\"text\" name=\"value\"> \
		      <input type=\"submit\" value=\"Submit\"> \
		      </form> \
		    </body></html> \
		  ";
		  return [GCDWebServerDataResponse responseWithHTML:html];
		
		}];
		
		[webServer addHandlerForMethod:@"POST"
		                          path:@"/"
		                  requestClass:[GCDWebServerURLEncodedFormRequest class]
		                  processBlock:^GCDWebServerResponse *(GCDWebServerRequest* request) {
		
		  NSString* value = [[(GCDWebServerURLEncodedFormRequest*)request arguments] objectForKey:@"value"];
		  NSString* html = [NSString stringWithFormat:@"<html><body><p>%@</p></body></html>", value];
		  return [GCDWebServerDataResponse responseWithHTML:html];
			}];

#####高级示例3 ：服务动态网站

GCDWebServer提供的扩展类 - GCDWebServerDataResponse，可以返回从模板和一组变量中（使用格式％变量％）生成的HTML内容。这是一个非常基本的模板系统，并且打算以它为出发点通过继承GCDWebServerResponse建造更先进的系统。

假设你的应用程序有一个网站目录在包含了HTML模板文件和相应的CSS，脚本和图像。这是很容易把它变成一个动态的网站：

	// Get the path to the website directory
	NSString* websitePath = [[NSBundle mainBundle] pathForResource:@"Website" ofType:nil];
	
	// Add a default handler to serve static files (i.e. anything other than HTML files)
	[self addGETHandlerForBasePath:@"/" directoryPath:websitePath indexFilename:nil cacheAge:3600 allowRangeRequests:YES];
	
	// Add an override handler for all requests to "*.html" URLs to do the special HTML templatization
	[self addHandlerForMethod:@"GET"
	                pathRegex:@"/.*\.html"
	             requestClass:[GCDWebServerRequest class]
	             processBlock:^GCDWebServerResponse *(GCDWebServerRequest* request) {
	
	    NSDictionary* variables = [NSDictionary dictionaryWithObjectsAndKeys:@"value", @"variable", nil];
	    return [GCDWebServerDataResponse responseWithHTMLTemplate:[websitePath stringByAppendingPathComponent:request.path]
	                                                    variables:variables];
	
	}];
	
	// Add an override handler to redirect "/" URL to "/index.html"
	[self addHandlerForMethod:@"GET"
	                     path:@"/"
	             requestClass:[GCDWebServerRequest class]
	             processBlock:^GCDWebServerResponse *(GCDWebServerRequest* request) {
	
	    return [GCDWebServerResponse responseWithRedirect:[NSURL URLWithString:@"index.html" relativeToURL:request.URL]
	                                            permanent:NO];
	
	];
#####最后一个例子：iOS 文件的下载和上传
GCDWebServer最初被写为iPad上的ComicFlow漫画阅读器应用程序。它允许用户使用其Web浏览器通过WiFi连接到他们的iPad ，然后上传，下载和整理漫画文件的应用程序内。

ComicFlow是完全开源的，你可以看到它是如何在WebServer.h和WebServer.m文件使用GCDWebServer 。
