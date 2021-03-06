# 有关Share Extension 【iOS扩展开发攻略】
###最近接到的share Extension需求，要求做原生的share Extension，就去百度很多资料，坑坑洼洼的完成了任务上线，现在回过来总结一下踩过的坑.
------------------------------------------------------------------------------------

* 1、首先给大家介绍一下iOS扩展.

   扩展（ Extension ）是 iOS 8 中引入的一个非常重要的新特性。扩展让 app 之间的数据交互成为可能。用户可以在 app 中使用其他应用提供的功能，而无需离开当前的应用。在 iOS 8 系统之前，每一个 app 在物理上都是彼此独立的， app 之间不能互访彼此的私有数据。而在引入扩展之后，其他 app 可以与扩展进行数据交换。基于安全和性能的考虑，每一个扩展运行在一个单独的进程中，它拥有自己的 bundle ， bundle 后缀名是.appex 。扩展 bundle 必须包含在一个普通应用的 bundle 的内部。

   iOS 8 系统有 6 个支持扩展的系统区域，分别是 Today 、 Share 、 Action 、 Photo Editing 、 Storage Provider 、 Custom keyboard 。支持扩展的系统区域也被称为扩展点。
   接下来我们主要讲一下share Extension具体实现。
* 2、代码实现

   ####1、 share Extension实现是需要依赖一个工程，如果你没有工程需要重新创建一个(如果你不会创建工程，你可以关闭浏览器了)；创建完工程后我们需要创建一个share Extension的工程，创建方法如图：
 
<!--<img src="http://github.com/wang22290/share-Extension/raw/master/Snip20180510_9.png" border="1" width="300" height="100" alt="d">-->
![image scr](http://github.com/wang22290/share-Extension/raw/master/Snip20180510_9.png)
  接着继续点击
  ![image](http://github.com/wang22290/share-Extension/raw/master/Snip20180510_10.png)
  ![image](http://github.com/wang22290/share-Extension/raw/master/Snip20180510_12.png)
  
  ![image](http://github.com/wang22290/share-Extension/raw/master/Snip20180510_13.png)

  完成后会如最后一张图片，现在我们就可以尝试一下share的功能。
  ![image](http://github.com/wang22290/share-Extension/raw/master/Snip20180510_15.png)
  share Extension项目运行必须有一个容器，我们先用默认的浏览器尝试一下，
  ![image](http://github.com/wang22290/share-Extension/raw/master/Snip20180510_16.png)
  进入手机页面你会发现share Extension按钮是灰色状态不能点击，我先需要打开一个网页，例如：（www.baidu.com），现在我们点击方向按钮，会在share Extension栏找到我们的APP
  
![image](http://github.com/wang22290/share-Extension/raw/master/Snip20180510_17.png)
  
 如果没有发现APP，点击旁边更多按钮，进入下级页面，把APP权限按钮打开
 ![image](http://github.com/wang22290/share-Extension/raw/master/Snip20180510_18.png)
<!--<img src="http://github.com/wang22290/share-Extension/raw/master/Snip20180510_18.png" border="0" width="300" height="370">-->
####2、接下来我们准备处理share Extension数据
![image](http://github.com/wang22290/share-Extension/raw/master/Snip20180512_22.png)
图片为苹果原生为我们提供的share Extension页面，进入程序shareViewController页面
![image](http://github.com/wang22290/share-Extension/raw/master/Snip20180512_23.png)
#####2.1依次来解析一下这三个方法

	/*
	isContentValid来判断我们获取到得数据是否是我们想要的。
	*/
	- (BOOL)isContentValid {
	// Do validation of contentText and/or NSExtensionContext attachments here
	return YES;
	} 
	/**
	*  点击取消按钮
	*/
	- (void)didSelectCancel
	{
	[super didSelectCancel];
	}
	
	/**
	 *  点击提交按钮
	 */
	- (void)didSelectPost
	{
	    // This is called after the user selects Post. Do the upload of contentText and/or NSExtensionContext attachments.
	
	    // Inform the host that we're done, so it un-blocks its UI. Note: Alternatively you could call super's -didSelectPost, which will similarly complete the extension context.
	    [self.extensionContext completeRequestReturningItems:@[] completionHandler:nil];
	    }

在这两个方法里面可以进行一些自定义的操作。一般情况下，当用户点击提交按钮的时候，扩展要做的事情就是要把数据取出来，并且放入一个与Containing App（** 容器程序，尽管苹果开放了Extension，但是在iOS中extension并不能单独存在，要想提交到AppStore，必须将Extension包含在一个App中提交，并且App的实现部分不能为空,这个包含Extension的App就叫Containing app。Extension会随着Containing App的安装而安装，同时随着ContainingApp的卸载而卸载。**）共享的数据介质中（包括NSUserDefault、Sqlite、CoreData），要跟容器程序进行数据交互需要借助AppGroups服务，下面的章节会对这块进行详细说明。下面先来看看怎么获取扩展中的数据。

#####2.2在ShareExtension中，UIViewController包含一个extensionContext这样的上下文对象：

	@interface UIViewController(NSExtensionAdditions) <NSExtensionRequestHandling>
	
	// Returns the extension context. Also acts as a convenience method for a view controller to check if it participating in an extension request.
	@property (nullable, nonatomic,readonly,strong) NSExtensionContext *extensionContext NS_AVAILABLE_IOS(8_0);
	@end

通过操作它就可以获取到share Extension的数据，返回宿主应用的界面等操作。我们可以先看一下extensionContext的定义。

	NS_CLASS_AVAILABLE(10_10, 8_0)
	@interface NSExtensionContext : NSObject
	
	// The list of input NSExtensionItems associated with the context. If the context has no input items, this array will be empty.
	@property(readonly, copy, NS_NONATOMIC_IOSONLY) NSArray *inputItems;
	
	// Signals the host to complete the app extension request with the supplied result items. The completion handler optionally contains any work which the extension may need to perform after the request has been completed, as a background-priority task. The `expired` parameter will be YES if the system decides to prematurely terminate a previous non-expiration invocation of the completionHandler. Note: calling this method will eventually dismiss the associated view controller.
	- (void)completeRequestReturningItems:(nullable NSArray *)items completionHandler:(void(^ __nullable)(BOOL expired))completionHandler;
	
	// Signals the host to cancel the app extension request, with the supplied error, which should be non-nil. The userInfo of the NSError will contain a key NSExtensionItemsAndErrorsKey which will have as its value a dictionary of NSExtensionItems and associated NSError instances.
	- (void)cancelRequestWithError:(NSError *)error;
	
	// Asks the host to open an URL on the extension's behalf
	- (void)openURL:(NSURL *)URL completionHandler:(void (^ __nullable)(BOOL success))completionHandler;
	
	@end
	
	// Key in userInfo. Value is a dictionary of NSExtensionItems and associated NSError instances.
	FOUNDATION_EXTERN NSString * __null_unspecified const NSExtensionItemsAndErrorsKey NS_AVAILABLE(10_10, 8_0);
	
	// The host process will enter the foreground
	FOUNDATION_EXTERN NSString * __null_unspecified const NSExtensionHostWillEnterForegroundNotification NS_AVAILABLE_IOS(8_2);
	
	// The host process did enter the background
	FOUNDATION_EXTERN NSString * __null_unspecified const NSExtensionHostDidEnterBackgroundNotification NS_AVAILABLE_IOS(8_2);
	
#####2.3NSExtensionContext的结构比较简单，包含一个属性和三个方法。其说明如下：

方法 | 说明
--------- | -------------
inputItems | 该数组存储着容器应用传入给NSExtensionContext的NSExtensionItem数组。其中每个NSExtensionItem标识了一种类型的数据。要获取数据就要从这个属性入手。
completeRequestReturningItems:<br />completionHandler: | 通知宿主程序的扩展已完成请求。调用此方法后，扩展UI会关闭并返回容器程序中。其中的items就是返回宿主程序的数据项。
cancelRequestWithError: | 通知宿主程序的扩展已取消请求。调用此方法后，扩展UI会关闭并返回容器程序中。其中error为错误的描述信息。
NSExtensionItemsAndErrorsKey | NSExtensionItem的userInfo属性中对应的错误信息键名。

#####2.4类的下面还定义了一些通知，这些通知都是跟宿主程序的行为相关，在设计扩展的时候可以根据这些通知来进行对应的操作。其说明如下：


通知名称 | 说明
--------- | -------------
NSExtensionHostWillEnterForegroundNotification |宿主程序将要返回前台通知NSExtensionHostDidEnterBackgroundNotification |宿主程序进入后台通知
NSExtensionHostWillResignActiveNotification |宿主程序将要被挂起通知
NSExtensionHostDidBecomeActiveNotification |宿主程序被激活通知
#####2.5从inputItems中获取数据
inputItems是包含NSExtensionItem类型对象的数组。那么，要处理里面的数据还得先来了解一下NSExtensionItem的结构：
	
	@interface NSExtensionItem : NSObject<NSCopying, NSSecureCoding>
	
	// (optional) title for the item
	@property(nullable, copy, NS_NONATOMIC_IOSONLY) NSAttributedString *attributedTitle;
	
	// (optional) content text
	@property(nullable, copy, NS_NONATOMIC_IOSONLY) NSAttributedString *attributedContentText;
	
	// (optional) Contains images, videos, URLs, etc. This is not meant to be an array of alternate data formats/types, but instead a collection to include in a social media post for example. These items are always typed NSItemProvider.
	@property(nullable, copy, NS_NONATOMIC_IOSONLY) NSArray *attachments;
	
	// (optional) dictionary of key-value data. The key/value pairs accepted by the service are expected to be specified in the extension's Info.plist. The values of NSExtensionItem's properties will be reflected into the dictionary.
	@property(nullable, copy, NS_NONATOMIC_IOSONLY) NSDictionary *userInfo;
	
	@end
	
	// Keys corresponding to properties exposed on the NSExtensionItem interface
	FOUNDATION_EXTERN NSString * __null_unspecified const NSExtensionItemAttributedTitleKey NS_AVAILABLE(10_10, 8_0);
	FOUNDATION_EXTERN NSString * __null_unspecified const NSExtensionItemAttributedContentTextKey NS_AVAILABLE(10_10, 8_0);
	FOUNDATION_EXTERN NSString * __null_unspecified const NSExtensionItemAttachmentsKey NS_AVAILABLE(10_10, 8_0);

NSExtensionItem包含四个属性

属性 | 说明
--------- | -------------
attributedTitle |	标题
attributedContentText |	内容。
attachments |	附件数组，包含图片、视频、链接等资源，封装在NSItemProvider类型中。
userInfo |	一个key－value结构的数据。NSExtensionItem中的属性都会在这个属性中一一映射。

对应userInfo结构中的NSExtensionItem属性的键名如下：

名称 | 说明
--------- | -------------
NSExtensionItemAttributedTitleKey |	标题的键名
NSExtensionItemAttributedContentTextKey |	内容的键名。
NSExtensionItemAttachmentsKey |	附件的键名

从上面的定义可以看出除了文本内容，其他类型的内容都是作为附件存储的，而附件又是封装在一个叫NSItemProvider的类型中，其定义如下：

	typedef void (^NSItemProviderCompletionHandler)(__nullable id <NSSecureCoding> item, NSError * __null_unspecified error);
	typedef void (^NSItemProviderLoadHandler)(__null_unspecified NSItemProviderCompletionHandler completionHandler, __null_unspecified Class expectedValueClass, NSDictionary * __null_unspecified options);
	
	// An NSItemProvider is a high level abstraction for file-like data objects supporting multiple representations and preview images.
	NS_CLASS_AVAILABLE(10_10, 8_0)
	@interface NSItemProvider : NSObject <NSCopying>
	
	// Initialize an NSItemProvider with a single handler for the given item.
	- (instancetype)initWithItem:(nullable id <NSSecureCoding>)item typeIdentifier:(nullable NSString *)typeIdentifier NS_DESIGNATED_INITIALIZER;
	
	// Initialize an NSItemProvider with load handlers for the given file URL, and the file content.
	- (nullable instancetype)initWithContentsOfURL:(null_unspecified NSURL *)fileURL;
	
	// Sets a load handler block for a specific type identifier. Handlers are invoked on demand through loadItemForTypeIdentifier:options:completionHandler:. To complete loading, the implementation has to call the given completionHandler. Both expectedValueClass and options parameters are derived from the completionHandler block.
	- (void)registerItemForTypeIdentifier:(NSString *)typeIdentifier loadHandler:(NSItemProviderLoadHandler)loadHandler;
	
	// Returns the list of registered type identifiers
	@property(copy, readonly, NS_NONATOMIC_IOSONLY) NSArray *registeredTypeIdentifiers;
	
	// Returns YES if the item provider has at least one item that conforms to the supplied type identifier.
	- (BOOL)hasItemConformingToTypeIdentifier:(NSString *)typeIdentifier;
	
	// Loads the best matching item for a type identifier. The client's expected value class is automatically derived from the blocks item parameter. Returns an error if the returned item class does not match the expected value class. Item providers will perform simple type coercions (eg. NSURL to NSData, NSURL to NSFileWrapper, NSData to UIImage).
	- (void)loadItemForTypeIdentifier:(NSString *)typeIdentifier options:(nullable NSDictionary *)options completionHandler:(nullable NSItemProviderCompletionHandler)completionHandler;
	
	@end
	
	// Common keys for the item provider options dictionary.
	FOUNDATION_EXTERN NSString * __null_unspecified const NSItemProviderPreferredImageSizeKey NS_AVAILABLE(10_10, 8_0); // NSValue of CGSize or NSSize, specifies image size in pixels.
	
	@interface NSItemProvider(NSPreviewSupport)
	
	// Sets a custom preview image handler block for this item provider. The returned item should preferably be NSData or a file NSURL.
	@property(nullable, copy, NS_NONATOMIC_IOSONLY) NSItemProviderLoadHandler previewImageHandler NS_AVAILABLE(10_10, 8_0);
	
	// Loads the preview image for this item by either calling the supplied preview block or falling back to a QuickLook-based handler. This method, like loadItemForTypeIdentifier:options:completionHandler:, supports implicit type coercion for the item parameter of the completion block. Allowed value classes are: NSData, NSURL, UIImage/NSImage.
	- (void)loadPreviewImageWithOptions:(null_unspecified NSDictionary *)options completionHandler:(null_unspecified NSItemProviderCompletionHandler)completionHandler NS_AVAILABLE(10_10, 8_0);
	
	@end
	
	// Keys used in property list items received from or sent to JavaScript code
	
	// If JavaScript code passes an object to its completionFunction, it will be placed into an item of type kUTTypePropertyList, containing an NSDictionary, under this key.
	FOUNDATION_EXTERN NSString * __null_unspecified const NSExtensionJavaScriptPreprocessingResultsKey NS_AVAILABLE(10_10, 8_0);
	
	// Arguments to be passed to a JavaScript finalize method should be placed in an item of type kUTTypePropertyList, containing an NSDictionary, under this key.
	FOUNDATION_EXTERN NSString * __null_unspecified const NSExtensionJavaScriptFinalizeArgumentKey NS_AVAILABLE_IOS(8_0);
	
	// Errors
	
	// Constant used by NSError to distinguish errors belonging to the NSItemProvider domain
	FOUNDATION_EXTERN NSString * __null_unspecified const NSItemProviderErrorDomain NS_AVAILABLE(10_10, 8_0);
	
	// NSItemProvider-related error codes
	typedef NS_ENUM(NSInteger, NSItemProviderErrorCode) {
	    NSItemProviderUnknownError                                      = -1,
	    NSItemProviderItemUnavailableError                              = -1000,
	    NSItemProviderUnexpectedValueClassError                         = -1100,
	    NSItemProviderUnavailableCoercionError NS_AVAILABLE(10_11, 9_0) = -1200
	} NS_ENUM_AVAILABLE(10_10, 8_0);

NSItemProvider结构说明

名称 | 说明
--------- | -------------
initWithItem:typeIdentifier: | 初始化方法，item为附件的数据，typeIdentifier是附件对应的类型标识,对应UTI的描述。
initWithContentsOfURL | 根据制定的文件路径来初始化。
registerItemForTypeIdentifier:loadHandler:|为一种资源类型自定义加载过程。这个方法主要针对自定义资源使用，例如自己定义的类或者文件格式等。当调用loadItemForTypeIdentifier:options:completionHandler:方法时就会触发定义的加载过程。
hasItemConformingToTypeIdentifier: | 用于判断是否有typeIdentifier(UTI)所指定的资源存在。存在则返回YES，否则返回NO。<br />该方法结合loadItemForTypeIdentifier:options:completionHandler:使用。
loadItemForTypeIdentifier:options:completionHandler: | 加载typeIdentifier指定的资源。加载是一个异步过程，加载完成后会触发completionHandler。
loadPreviewImageWithOptions:completionHandler: | 加载资源的预览图片。

由此可见，其结构如下图所示：
![image](http://github.com/wang22290/share-Extension/raw/master/1804600.png)

为了要取到宿主程序提供的数组，那么只要关注loadItemTypeIdentifier:options:completionHandler方法的使用即可。有了上面的了解，那么接下来就是对inputItems进行数据分析并提取了，这里以一个链接的share Extension为例，改写视图控制器中的didSelectPost方法。看下面的代码：

	- (void)didSelectPost
	{
	    __block BOOL hasExistsUrl = NO;
	    [self.extensionContext.inputItems enumerateObjectsUsingBlock:^(NSExtensionItem * _Nonnull extItem, NSUInteger idx, BOOL * _Nonnull stop) {
	        [item.attachments enumerateObjectsUsingBlock:^(NSItemProvider * _Nonnull itemProvider, NSUInteger idx, BOOL * _Nonnull stop) {
	         //获取图片
				if ([itemProvider hasItemConformingToTypeIdentifier:@"public.url"])
				            {
				                [itemProvider loadItemForTypeIdentifier:@"public.url"
				                                                options:nil
				                                      completionHandler:^(id<NSSecureCoding>  _Nullable item, NSError * _Null_unspecified error) {
				
				                                          if ([(NSObject *)item isKindOfClass:[NSURL class]])
				                                          {
				                                              NSLog(@"share Extension的URL = %@", item);
				                                          }
				
				                                      }];
				
				                hasExistsUrl = YES;
				                *stop = YES;
				            }
				
				        }];
              //获取链接
	            if ([itemProvider hasItemConformingToTypeIdentifier:@"public.url"])
	            {
	                [itemProvider loadItemForTypeIdentifier:@"public.url"
	                                                options:nil
	                                      completionHandler:^(id<NSSecureCoding>  _Nullable item, NSError * _Null_unspecified error) {
	
	                                          if ([(NSObject *)item isKindOfClass:[NSURL class]])
	                                          {
	                                              NSLog(@"share Extension的URL = %@", item);
	                                          }
	
	                                      }];
	
	                hasExistsUrl = YES;
	                *stop = YES;
	            }
	
	        }];
	
	        if (hasExistsUrl)
	        {
	            *stop = YES;
	        }
	
	    }];
	
	    // This is called after the user selects Post. Do the upload of contentText and/or NSExtensionContext attachments.
	    // Inform the host that we're done, so it un-blocks its UI. Note: Alternatively you could call super's -didSelectPost, which will similarly complete the extension context.
	//    [self.extensionContext completeRequestReturningItems:@[] completionHandler:nil];
	}

上面的例子中遍历了extensionContext的inputItems数组中所有NSExtensionItem对象，然后从这些对象中遍历attachments数组中的所有NSItemProvider对象。匹配第一个包含public.url标识的附件（具体要匹配什么资源，数量是多少皆有自己的业务所决定）。** 注意：在上面代码中注释了[self.extensionContext completeRequestReturningItems:@[] completionHandler:nil];这行代码，主要是使到视图控制器不被关闭，等到实现相应的处理后再进行调用该方法，对share Extension视图进行关闭。** 在下面的章节会说明这一点。
####2.5 将share Extension数据传递给容器程序
上面章节已经讲述了如何取得宿主应用所share Extension的内容。那么，接下来就是将这些内容传递给容器程序进行相应的操作（如：在一款社交应用中，可能会为取得的share Extension内容发布一条用户动态）。在默认情况下，iOS的应用是存在一个沙盒里面的，不允许应用与应用直接进行数据的交互。为此，苹果提供了一项叫App Groups的服务，该服务允许开发者可以在自己的应用之间通过NSUserDefaults、NSFileManager或者CoreData来进行相互的数据传输。下面介绍如何激活App Groups服务：

首先要有一个独立的AppID（带通配符＊号的AppID是不允许激活App Groups的）
==*Xcode中直接打开group，设置group后，开发者网站会同步*==

![image](http://github.com/wang22290/share-Extension/raw/master/Snip20180512_24.png)
点击添加按钮，会出现添加框
![image](http://github.com/wang22290/share-Extension/raw/master/Snip20180512_25.png)
gronp.后面填写你项目的bundle identifer 即可；同样在share 项目中添加group信息，（系统应该已经默认为你添加上，默认选择就好）
![image](http://github.com/wang22290/share-Extension/raw/master/Snip20180512_26.png)

至此，应用和扩展的App Groups服务都已经启动，现在就要进行share Extension内容的传输操作。下面分别介绍一下NSUserDefaults、NSFileManager以及CoreData三种方式是如何实现App Groups下的数据操作：

* NSUserDefaults：要想设置或访问Group的数据，不能在使用standardUserDefaults方法来获取一个NSUserDefaults对象了。应该使用initWithSuiteName:方法来初始化一个NSUserDefaults对象，其中的SuiteName就是创建的Group的名字，然后利用这个对象来实现，跨应用的数据读写，代码如下：
	
		
		//初始化一个供App Groups使用的NSUserDefaults对象
		NSUserDefaults *userDefaults = [[NSUserDefaults alloc] initWithSuiteName:@"group.Taiyi.shareP"];
		
		//写入数据
		[userDefaults setValue:@"value" forKey:@"key"];
		
		//读取数据
		NSLog(@"%@", [userDefaults valueForKey:@"key"]);
	
* NSFileManager：通过调用 containerURLForSecurityApplicationGroupIdentifier:方法可以获得AppGroup的共享目录，然后在此目录的基础上实现任意的文件操作。代码如下：
   //获取分组的共享目录
   
		NSURL *groupURL = [[NSFileManager defaultManager] containerURLForSecurityApplicationGroupIdentifier:@"group.cn.vimfung.ShareExtensionDemo"];
		NSURL *fileURL = [groupURL URLByAppendingPathComponent:@"demo.txt"];
		
		//写入文件
		[@"abc" writeToURL:fileURL atomically:YES encoding:NSUTF8StringEncoding error:nil];
		
		//读取文件
		NSString *str = [NSString stringWithContentsOfURL:fileURL encoding:NSUTF8StringEncoding error:nil];
		NSLog(@"str = %@", str);
	
* CoreData：其实CoreData是基于NSFileManager取得共享目录后来实现数据共享的。即在初始化CoreData时，先使用NSFileManager取得共享目录，然后再指定共享目录为存储数据文件的目录（如存储的sqlite文件）。代码如下：
		 
		   //获取分组的共享项目
		NSURL *containerURL = [[NSFileManager defaultManager] containerURLForSecurityApplicationGroupIdentifier:@"group.cn.vimfung.ShareExtensionDemo"];
		NSURL *storeURL = [containerURL URLByAppendingPathComponent:@"DataModel.sqlite"];
		
		//初始化持久化存储调度器
		NSURL *modelURL = [[NSBundle mainBundle] URLForResource:@"DataModel" withExtension:@"momd"];
		
		NSManagedObjectModel *model = [[NSManagedObjectModel alloc] initWithContentsOfURL:modelURL];
		NSPersistentStoreCoordinator *coordinator = [[NSPersistentStoreCoordinator alloc] initWithManagedObjectModel:model];
		
		[coordinator addPersistentStoreWithType:NSSQLiteStoreType
		                          configuration:nil
		                                    URL:storeURL
		                                options:nil
		                                  error:nil];
		
		//创建受控对象上下文
		NSManagedObjectContext *context = [[NSManagedObjectContext alloc] initWithConcurrencyType:NSMainQueueConcurrencyType];
		
		[context performBlockAndWait:^{
		    [context setPersistentStoreCoordinator:coordinator];
	}];

为了方便演示，这里会使用NSUserDefault来直接把取到的url地址保存起来。代码如下所示：
_(默认情况下，如果用户点击Post按钮后，share Extension界面就会消失，用户可以继续对宿主程序进行操作。这些都要靠NSExtensionContextd的completeRequestReturningItems:completionHandler:方法来实现。现在，由于在didSelectPost方法中加入了share Extension内容的处理，由于获取附件是一个异步过程，那么，就需要做好界面上的提示。否则，share Extension界面消失后由于没有操作提示，会使用户误以为界面进行卡死的状态，其实是share Extension内容还没有处理完成。接下来就是优化UI上的提示操作，
)_

	
	- (void)didSelectPost {
	    
	    _viewCon = [ViewController new];
	    NSString *aa = [_viewCon setNumber];
	    NSLog(@"%@",aa);
	    //加载动画初始化
	    UIActivityIndicatorView *activityIndicatorView = [[UIActivityIndicatorView alloc] initWithActivityIndicatorStyle:UIActivityIndicatorViewStyleWhiteLarge];
	    activityIndicatorView.frame = CGRectMake((self.view.frame.size.width - activityIndicatorView.frame.size.width) / 2,
	                                             (self.view.frame.size.height - activityIndicatorView.frame.size.height) / 2,
	                                             activityIndicatorView.frame.size.width,
	                                             activityIndicatorView.frame.size.height);
	    activityIndicatorView.autoresizingMask = UIViewAutoresizingFlexibleTopMargin | UIViewAutoresizingFlexibleLeftMargin | UIViewAutoresizingFlexibleRightMargin | UIViewAutoresizingFlexibleBottomMargin;
	    [self.view addSubview:activityIndicatorView];
	    
	    //激活加载动画
	    [activityIndicatorView startAnimating];
	
	    // Inform the host that we're done, so it un-blocks its UI. Note: Alternatively you could call super's -didSelectPost, which will similarly complete the extension context.
	    [self.extensionContext completeRequestReturningItems:@[] completionHandler:nil];
	    
	    __block BOOL hasExistsUrl = NO;
	    [self.extensionContext.inputItems enumerateObjectsUsingBlock:^(NSExtensionItem * _Nonnull extItem, NSUInteger idx, BOOL * _Nonnull stop) {
        NSLog(@"%@-----------%@",extItem.attributedTitle,extItem.attributedContentText);
        
        NSUserDefaults *userDefaults = [[NSUserDefaults alloc] initWithSuiteName:@"group.taiyi.shareP"];
        NSAttributedString *strings = [extItem.attributedContentText attributedSubstringFromRange:NSMakeRange(0, extItem.attributedContentText.length)];
        NSArray *array = [strings.string componentsSeparatedByString:@"\n"];
        NSString *firstString = array[0];
        NSLog(@"%@",firstString);
        [userDefaults setValue:firstString forKey:@"share-content"];
       
        [extItem.attachments enumerateObjectsUsingBlock:^(NSItemProvider * _Nonnull itemProvider, NSUInteger idx, BOOL * _Nonnull stop) {

            NSLog(@"%d",[itemProvider hasItemConformingToTypeIdentifier:@"public.url"]);
            
            if ([itemProvider hasItemConformingToTypeIdentifier:@"public.text"])
            {
                //加载typeIdentifier指定的资源
                [itemProvider loadItemForTypeIdentifier:@"public.text"
                                                options:nil
                                      completionHandler:^(id<NSSecureCoding>  _Nullable item, NSError * _Null_unspecified error) {
                                          
                                          if ([(NSObject *)item isKindOfClass:[NSURL class]])
                                              
                                          {
                                              NSLog(@"share Extension的URL = %@", item);
                                              
                                              [userDefaults setValue:((NSURL *)item).absoluteString forKey:@"share-text-url"];
                                              
                                              //用于标记是新的share Extension
                                              [userDefaults setBool:YES forKey:@"has-new-share"];
                                              
                                              [activityIndicatorView stopAnimating];
                                              [self.extensionContext completeRequestReturningItems:@[extItem] completionHandler:nil];
                                              
                                          }
                                          
                                      }];
                
                hasExistsUrl = YES;
                *stop = YES;
            }
            
            if ([itemProvider hasItemConformingToTypeIdentifier:@"public.image"])
            {
                //加载typeIdentifier指定的资源
                [itemProvider loadItemForTypeIdentifier:@"public.image"
                                                options:nil
                                      completionHandler:^(id<NSSecureCoding>  _Nullable item, NSError * _Null_unspecified error) {
                                          
                                          if ([(NSObject *)item isKindOfClass:[NSURL class]])
                                              
                                          {
                                              NSLog(@"share Extension的URL = %@", item);
                                              
                                              [userDefaults setValue:((NSURL *)item).absoluteString forKey:@"share-image-url"];
                                              
                                              //用于标记是新的share Extension
                                              [userDefaults setBool:YES forKey:@"has-new-share"];
                                              
                                              [activityIndicatorView stopAnimating];
                                              [self.extensionContext completeRequestReturningItems:@[extItem] completionHandler:nil];
                                              
                                          }
                                          
                                      }];
                
                hasExistsUrl = YES;
                *stop = YES;
            }

            
            
            if ([itemProvider hasItemConformingToTypeIdentifier:@"public.url"])
            {
                //加载typeIdentifier指定的资源
                [itemProvider loadItemForTypeIdentifier:@"public.url"
                                                options:nil
                                      completionHandler:^(id<NSSecureCoding>  _Nullable item, NSError * _Null_unspecified error) {
                                          
                                          if ([(NSObject *)item isKindOfClass:[NSURL class]])

                                          {
                                              NSLog(@"share Extension的URL = %@", item);
                                              
                                              [userDefaults setValue:((NSURL *)item).absoluteString forKey:@"share-url"];
                                             
                                              //用于标记是新的share Extension
                                              [userDefaults setBool:YES forKey:@"has-new-share"];
                                              
                                              [activityIndicatorView stopAnimating];
                                              [self.extensionContext completeRequestReturningItems:@[extItem] completionHandler:nil];
                                              
                                          }
                                          
                                      }];
                
                hasExistsUrl = YES;
                *stop = YES;
            }
            
        }];
        
        if (hasExistsUrl)
        {
            *stop = YES;
        }
        
    }];
    
    if (!hasExistsUrl)
    {
        //直接退出
        [self.extensionContext completeRequestReturningItems:@[] completionHandler:nil];
    }
    }

####2.6 容器程序获取share Extension数据
插件的工作基本上已经全部开发完成了，接下来就是容器程序获取数据并进行操作。下面是容器程序的处理代码：

	- (void)applicationDidBecomeActive:(UIApplication *)application
	{
	    //获取共享的UserDefaults
	    NSUserDefaults *userDefaults = [[NSUserDefaults alloc] initWithSuiteName:@"group.cn.vimfung.ShareExtensionDemo"];
	    if ([userDefaults boolForKey:@"has-new-share"])
	    {
	        NSLog(@"新的share Extension : %@", [userDefaults valueForKey:@"share-url"]);
	
	        //重置share Extension标识
	        [userDefaults setBool:NO forKey:@"has-new-share"];
	    }
	}
	
为了方便演示，这里直接在AppDelegate中的applicationDidBecomeActive:方法中检测是否有新的share Extension，如果有则通过Log打印链接出来。

####2.7 在share share Extension中，应该会有很多同学因为数据传输，页面调用问题发愁，接下来重点介绍一下我使用的方法==直接唤起APP，完成 share Extension==
 * 我们需要给APP配置一个url Schemes；
 ![image](http://github.com/wang22290/share-Extension/raw/master/Snip20180512_28.png)
 
* 然后在shareViewController里面调用

        __block BOOL hasExistsUrl = NO;
	    [self.extensionContext.inputItems enumerateObjectsUsingBlock:^(NSExtensionItem * _Nonnull extItem, NSUInteger idx, BOOL * _Nonnull stop) {
	        NSLog(@"%@-----------%@",extItem.attributedTitle,extItem.attributedContentText);
	        NSAttributedString *strings = [extItem.attributedContentText attributedSubstringFromRange:NSMakeRange(0, extItem.attributedContentText.length)];
        
        NSArray *array = [strings.string componentsSeparatedByString:@"\n"];
        self.titleString = [NSString stringWithFormat:@"%@",array[0]];
        
        [extItem.attachments enumerateObjectsUsingBlock:^(NSItemProvider * _Nonnull itemProvider, NSUInteger idx, BOOL * _Nonnull stop) {
            //用于判断是否有typeIdentifier(UTI)所指定的资源存在。
            if ([itemProvider hasItemConformingToTypeIdentifier:@"public.url"])
            {
                //加载typeIdentifier指定的资源
                [itemProvider loadItemForTypeIdentifier:@"public.url"
                                                options:nil
                                      completionHandler:^(id<NSSecureCoding>  _Nullable item, NSError * _Null_unspecified error) {
                                          
                                          if ([(NSObject *)item isKindOfClass:[NSURL class]])
                                          {
                                              NSLog(@"分享的URL = %@", item);
                                              self.urlString = [NSString stringWithFormat:@"%@",item];


                                              NSString *urlStr = [NSString stringWithFormat:@"shareP://?articleTitle=%@&articleUrl=%@",[self encode:self.titleString], [self encode:self.urlString]];

                                              if ([[UIApplication sharedApplication] canOpenURL:[NSURL URLWithString:urlStr]]) {
                                                  //可以调起APP
                                                  [[UIApplication sharedApplication] openURL:[NSURL URLWithString:urlStr]];
                                                  NSLog(@"调起成功");
                                                  
                                                  //直接退出
                                                  [self.extensionContext completeRequestReturningItems:@[] completionHandler:nil];
                                              }
                                              
                                          }
                                          
                                      }];
                
                hasExistsUrl = YES;
                *stop = YES;
            }
            
        }];
        
        if (hasExistsUrl)
        {
            *stop = YES;
        }
	         }];
	    
	    if (!hasExistsUrl)
	    {
	        //直接退出
	        [self.extensionContext completeRequestReturningItems:@[] completionHandler:nil];
	    }
	    [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(userHasebeenLocation) name:@"dismissController" object:nil];
 
* 在项目appDelegate中接收信息
				  
		//授权登录操作
		-(BOOL)application:(UIApplication *)application openURL:(NSURL *)url sourceApplication:(NSString *)sourceApplication annotation:(id)annotation {
		    NSString *urlStr = url.absoluteString;
		    NSString *sechemes = url.scheme;
		    if([[self getAppSchemeString] isEqualToString:sechemes]){
        
        if ([urlStr containsString:@"articleTitle"] && [urlStr containsString:@"articleUrl"]) {
            NSRange range1 = [urlStr rangeOfString:@"="];
            NSRange range2 = [urlStr rangeOfString:@"&"];
            NSString *articleTitle = [urlStr substringWithRange:NSMakeRange(range1.location + range1.length, range2.location - range1.location - range1.length)];
            
            NSString *stateStr = [urlStr substringFromIndex:range2.location+1];
            NSRange range3 = [stateStr rangeOfString:@"="];
            NSString *articleUrl = [stateStr substringFromIndex:range3.location+1];
            NSLog(@"%@====%@",articleTitle,articleUrl);
            
            

                //跳转代码就是跳转到你自己设计的控制器，我这里就不写了
        }
        
        }
	    return YES;
		}
到此，share Extension 调用APP完成，剩下的全部都可以在APP内部操作了，是不是很方便，


####2.7配置info文件
group设置完成后，我们需要配置修改info文件中的NSExtensionActivationRule字段
![image](http://github.com/wang22290/share-Extension/raw/master/Snip20180512_27.png)
我们只需要关注以下几个字段的设置：

名称 | 说明
--------- | -------------
Bundle display name |	扩展的显示名称，默认跟你的项目名称相同，可以通过修改此字段来控制扩展的显示名称。
NSExtension |	扩展描述字段，用于描述扩展的属性、设置等。作为一个扩展项目必须要包含此字段。
NSExtensionAttributes |	扩展属性集合字段。用于描述扩展的属性。
NSExtensionActivationRule |		激活扩展的规则。默认为字符串“TRUEPREDICATE”，表示在share Extension菜单中一直显示该扩展。可以将类型改为Dictionary类型，然后添加以下字段：<br />NSExtensionActivationSupportsAttachmentsWithMaxCount<br />NSExtensionActivationSupportsAttachmentsWithMinCount<br />NSExtensionActivationSupportsImageWithMaxCount<br />NSExtensionActivationSupportsMovieWithMaxCount<br />NSExtensionActivationSupportsWebPageWithMaxCount<br />NSExtensionActivationSupportsWebURLWithMaxCount
NSExtensionMainStoryboard |	设置主界面的Storyboard，如果不想使用storyboard，也可以使用NSExtensionPrincipalClass指定自定义UIViewController子类名
NSExtensionPointIdentifier |	扩展标识，在share Extension扩展中为：com.apple.share-services
NSExtensionPrincipalClass |	自定义UI的类名
NSExtensionActivationSupportsAttachmentsWithMaxCount |	附件最多限制，为数值类型。附件包括File、Image和Movie三大类，单一、混选总量不超过指定数量
NSExtensionActivationSupportsAttachmentsWithMinCount |	附件最少限制，为数值类型。当设置NSExtensionActivationSupportsAttachmentsWithMaxCount时生效，默认至少选择1个附件，share Extension菜单中才显示扩展插件图标。
NSExtensionActivationSupportsFileWithMaxCount|	文件最多限制，为数值类型。文件泛指除Image/Movie之外的附件，例如【邮件】附件、【语音备忘录】等。<br /><br />单一、混选均不超过指定数量。
NSExtensionActivationSupportsImageWithMaxCount | 图片最多限制，为数值类型。单一、混选均不超过指定数量
NSExtensionActivationSupportsMovieWithMaxCount | 视频最多限制，为数值类型。单一、混选均不超过指定数量。
NSExtensionActivationSupportsText | 是否支持文本类型，布尔类型，默认不支持。如【备忘录】的share Extension
NSExtensionActivationSupportsWebURLWithMaxCount| Web链接最多限制，为数值类型。默认不支持share Extension超链接，需要自己设置一个数值。
NSExtensionActivationSupportsWebPageWithMaxCount | 	Web页面最多限制，为数值类型。默认不支持Web页面share Extension，需要自己设置一个数值。

对于不同的应用里面有可能出现只允许接受某种类型的内容，那么Share Extension就不能一直出现在share Extension菜单中，因为不同的应用提供的share Extension内容不一样，这就需要通过设置NSExtensionActivationRule字段来决定Share Extension是否显示。例如，只想接受其他应用share Extension链接到自己的应用，那么可以通过下面的步骤来设置：

将NSExtensionActivationRule字段类型由String改为Dictionary。
展开NSExtensionActivationRule字段，创建其子项NSExtensionActivationSupportsWebURLWithMaxCount，并设置一个限制数量。

==一定要把全部的规则配置，否则对应的share Extension中不会显示APP==



#####3 提审AppStore的注意事项
* 扩展中的处理不能太长时间阻塞主线程（建议放入线程中处处理），否则可能导致苹果拒绝你的应用。
* 扩展不能单独提审，必须要跟容器程序一起提交AppStore进行审核。
* 提审的扩展和容器程序的Build Version要保持一致，否则在上传审核包的时候会提示警告，导致程序无法正常提审。
* 如果你的APP要送审APPStore必须全部配置NSExtensionActivationRule，字段类型必须对应，否则会提交失败，
* 如果你的APP是用企业账号分发，==强烈建议不要使用group方式传递数据==，企业账号分发会关闭group，导致不能数据传输，

####4. 进阶研究
  * 4.1 对默认分享界面进行扩展
  在某些情况下，在分享界面中会加入一下其它信息的显示，或者其它的选项供用户操作。如：内容要分享给什么好友、分享内容的可见权限等等。那么，默认的分享界面（ SLComposeServiceViewController）提供了相关的方法来对其进行扩展。这些方法定义如下
			
		  #if TARGET_OS_IPHONE
		/*
		 Configuration Item Support (account pickers, privacy selection, location, etc.)
		 */
			
		// Subclasses should implement this, and return an array of SLComposeSheetConfigurationItem instances, if if needs to display configuration items in the sheet. Defaults to nil.
		- (NSArray *)configurationItems;
		
		// Forces a reload of the configuration items table.
		// This is typically only necessary for subclasses that determine their configuration items in a deferred manner (for example, in -presentationAnimationDidFinish).
		// You do not need to call this after changing a configuration item property; the base class detects and reacts to that automatically.
		- (void)reloadConfigurationItems;
		
		// Presents a configuration view controller. Typically called from a configuration item's tapHandler. Only one configuration view controller is allowed at a time.
		// The pushed view controller should set preferredContentSize appropriately. SLComposeServiceViewController observes changes to that property and animates sheet size changes as necessary.
		- (void)pushConfigurationViewController:(UIViewController *)viewController;
		
		// Dismisses the current configuration view controller.
		- (void)popConfigurationViewController;
		#endif
其属性说明如下：	
属性 | 说明
--------- | -------------
title | 配置项标题
value | 当前的配置值
valuePending | YES时，显示值位置显示加载动画，NO时，显示配置的值。
tapHandler | 点击配置项的事件处理

下面将通过使用这些方法来扩展UI，使插件增加两个配置项：一个是是否公开分享的配置项，该选项标识一个开关值。另外一个是公开权限设置项，在是否公开分享的开关为开时显示。可以选择分享给所有人还是好友。代码如下所示：
	
	- (NSArray *)configurationItems {
	    // To add configuration options via table cells at the bottom of the sheet, return an array of SLComposeSheetConfigurationItem here.
	
	    //定义两个配置项，分别记录用户选择是否公开以及公开的权限，然后根据配置的值
	    static BOOL isPublic = NO;
	    static NSInteger act = 0;
	
	    NSMutableArray *items = [NSMutableArray array];
	
	    //创建是否公开配置项
	    SLComposeSheetConfigurationItem *item = [[SLComposeSheetConfigurationItem alloc] init];
	    item.title = @"是否公开";
	    item.value = isPublic ? @"是" : @"否";
	
	    __weak ShareViewController *theController = self;
	    __weak SLComposeSheetConfigurationItem *theItem = item;
	    item.tapHandler = ^{
	
	        isPublic = !isPublic;
	        theItem.value = isPublic ? @"是" : @"否";
	
	
	        [theController reloadConfigurationItems];
	    };
	
	    [items addObject:item];
	
	    if (isPublic)
	    {
	        //如果公开标识为YES，则创建公开权限配置项
	        SLComposeSheetConfigurationItem *actItem = [[SLComposeSheetConfigurationItem alloc] init];
	
	        actItem.title = @"公开权限";
	
	        switch (act)
	        {
	            case 0:
	                actItem.value = @"所有人";
	                break;
	            case 1:
	                actItem.value = @"好友";
	                break;
	            default:
	                break;
	        }
	
	        actItem.tapHandler = ^{
	
	            //设置分享权限时弹出选择界面
	            ShareActViewController *actVC = [[ShareActViewController alloc] init];
	            [theController pushConfigurationViewController:actVC];
	
	            [actVC onSelected:^(NSIndexPath *indexPath) {
	
	                //当选择完成时退出选择界面并刷新配置项。
	                act = indexPath.row;
	                [theController popConfigurationViewController];
	                [theController reloadConfigurationItems];
	
	            }];
	
	        };
	
	        [items addObject:actItem];
	    }
	
	    return items;
		}

ShareActViewController 的实现
	
		@interface ShareActViewController () <UITableViewDelegate, UITableViewDataSource>
	
	@property (nonatomic, strong) void (^selectedHandler) ();
	
	@end
	
	@implementation ShareActViewController
	
	- (void)viewDidLoad
	{
	    [super viewDidLoad];
	
	    UITableView *tableView = [[UITableView alloc] initWithFrame:self.view.bounds];
	    tableView.backgroundColor = [UIColor clearColor];
	    tableView.autoresizingMask = UIViewAutoresizingFlexibleWidth | UIViewAutoresizingFlexibleHeight;
	    tableView.dataSource = self;
	    tableView.delegate = self;
	    [tableView registerClass:[UITableViewCell class] forCellReuseIdentifier:@"Cell"];
	    [self.view addSubview:tableView];
	}
	
	- (void)onSelected:(void(^)(NSIndexPath *indexPath))handler
	{
	    self.selectedHandler = handler;
	}
	
	- (NSInteger)tableView:(UITableView *)tableView numberOfRowsInSection:(NSInteger)section
	{
	    return 2;
	}
	
	- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath
	{
	    UITableViewCell *cell = [tableView dequeueReusableCellWithIdentifier:@"Cell"];
	    cell.backgroundColor = [UIColor clearColor];
	
	    switch (indexPath.row)
	    {
	        case 0:
	            cell.textLabel.text = @"所有人";
	            break;
	        case 1:
	            cell.textLabel.text = @"好友";
	            break;
	        default:
	            break;
	    }
	
	    return cell;
	}
	
	- (void)tableView:(UITableView *)tableView didSelectRowAtIndexPath:(NSIndexPath *)indexPath
	{
	    if (self.selectedHandler)
	    {
	        self.selectedHandler (indexPath);
	    }
	}
在分享插件界面中重写了configurationItems方法，然后定义了两个配置项属性，分别是是否公开标识isPublic和公开权限act。然后创建是否公开的SLComposeSheetConfigurationItem配置项和根据isPublic的值来判断是否创建公开权限配置项。其中是否公开配置点击时会变更isPublic的值，从而达到显示或隐藏公开权限配置。而公开权限配置的点击则弹出一个选择的TableView，用于选择给定的值然后返回到分享界面。

####5. 替换Share Extension中的默认分享界面
1、如果通过扩展SLComposeServiceViewController还不能满足需求的情况下，这时候就需要自己设计一个分享视图控制器来替换默认的SLComposeServiceViewController。

首先，创建一个自定义视图控制器，如：CustomShareViewController。

2、然后打开扩展的Info.plist文件，删除NSExtensionMainStoryboard属性并增加一项NSExtensionPrincipalClass属性并指向CustomShareViewController（注：这里没有使用Storyboard所以要删除该属性），如图：

![image](http://github.com/wang22290/share-Extension/raw/master/Snip20180514_7.png)
3、接下来根据实际的需要来设计分享视图的展示与交互形式。

4、然后调用CustomShareViewController的extensionContext属性来控制扩展的提交与取消等操作（注：由于扩展中导入了关于ExtensionContext的UIViewController类目，因此，每个ViewController都带有extensionContext属性）。

为了演示的简单性，下面的代码会通过extensionContext获取到url后，给到自定义分享视图的Label中显示，同时也提供一个提交和取消按钮，用于用户对分享内容的操作。代码如下：
	
	- (void)viewDidLoad
	{
	    [super viewDidLoad];
	    // Do any additional setup after loading the view.
	
	    //定义一个容器视图来存放分享内容和两个操作按钮
	    UIView *container = [[UIView alloc] initWithFrame:CGRectMake((self.view.frame.size.width - 300) / 2, (self.view.frame.size.height - 175) / 2, 300, 175)];
	    container.layer.cornerRadius = 7;
	    container.layer.borderColor = [UIColor lightGrayColor].CGColor;
	    container.layer.borderWidth = 1;
	    container.layer.masksToBounds = YES;
	    container.backgroundColor = [UIColor whiteColor];
	    container.autoresizingMask = UIViewAutoresizingFlexibleTopMargin | UIViewAutoresizingFlexibleLeftMargin | UIViewAutoresizingFlexibleRightMargin | UIViewAutoresizingFlexibleBottomMargin;
	    [self.view addSubview:container];
	
	    //定义Post和Cancel按钮
	    UIButton *cancelBtn = [UIButton buttonWithType:UIButtonTypeSystem];
	    [cancelBtn setTitle:@"Cancel" forState:UIControlStateNormal];
	    cancelBtn.frame = CGRectMake(8, 8, 65, 40);
	    [cancelBtn addTarget:self action:@selector(cancelBtnClickHandler:) forControlEvents:UIControlEventTouchUpInside];
	    [container addSubview:cancelBtn];
	
	    UIButton *postBtn = [UIButton buttonWithType:UIButtonTypeSystem];
	    [postBtn setTitle:@"Post" forState:UIControlStateNormal];
	    postBtn.frame = CGRectMake(container.frame.size.width - 8 - 65, 8, 65, 40);
	    [postBtn addTarget:self action:@selector(postBtnClickHandler:) forControlEvents:UIControlEventTouchUpInside];
	    [container addSubview:postBtn];
	
	    //定义一个分享链接标签
	    UILabel *label = [[UILabel alloc] initWithFrame:CGRectMake(8,
	                                                               cancelBtn.frame.origin.y + cancelBtn.frame.size.height + 8,
	                                                               container.frame.size.width - 16,
	                                                               container.frame.size.height - 16 - cancelBtn.frame.origin.y - cancelBtn.frame.size.height)];
	    label.numberOfLines = 0;
	    label.textAlignment = NSTextAlignmentCenter;
	    [container addSubview:label];
	
	    //获取分享链接
	    __block BOOL hasGetUrl = NO;
	    [self.extensionContext.inputItems enumerateObjectsUsingBlock:^(NSExtensionItem *  _Nonnull obj, NSUInteger idx, BOOL * _Nonnull stop) {
	
	        [obj.attachments enumerateObjectsUsingBlock:^(NSItemProvider *  _Nonnull itemProvider, NSUInteger idx, BOOL * _Nonnull stop) {
	
	            if ([itemProvider hasItemConformingToTypeIdentifier:@"public.url"])
	            {
	                [itemProvider loadItemForTypeIdentifier:@"public.url" options:nil completionHandler:^(id<NSSecureCoding>  _Nullable item, NSError * _Null_unspecified error) {
	
	                    if ([(NSObject *)item isKindOfClass:[NSURL class]])
	                    {
	                        dispatch_async(dispatch_get_main_queue(), ^{
	
	                            label.text = ((NSURL *)item).absoluteString;
	
	                        });
	                    }
	
	                }];
	
	                hasGetUrl = YES;
	                *stop = YES;
	            }
	
	            *stop = hasGetUrl;
	
	        }];
	
	    }];
	}
	
	- (void)cancelBtnClickHandler:(id)sender
	{
	    //取消分享
	    [self.extensionContext cancelRequestWithError:[NSError errorWithDomain:@"CustomShareError" code:NSUserCancelledError userInfo:nil]];
	}
	
	- (void)postBtnClickHandler:(id)sender
	{
	    //执行分享内容处理
	    [self.extensionContext completeRequestReturningItems:@[] completionHandler:nil];
	}

share Extension 的基本内容就是这样了，
下面是Demo的地址；[shareP](https://github.com/wang22290/shareDemo.git)





	
	


    

    
   

