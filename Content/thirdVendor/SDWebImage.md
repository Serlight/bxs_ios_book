# SDWebImage使用
#### SDWebImage介绍
<i>&#160;&#160;&#160;&#160;&#160;SDWebImage图片加载框架占据iOS的大半壁江山，它支持网络下载，缓存。获取到的图片设置给对应的UIImageView和UIButton控件。使用SDWebImage管理图片可以提高开发效率。</i>
    
SDWebImage框架github托管地址：**[https://github.com/rs/SDWebImage](https://github.com/rs/SDWebImage)**
****
#### SDWebImage通常使用方法
###### 一、下面是UIImageView的例子 （UIImageView和UIButton用法差不多）
`1. sd_setImageWithURL:`
		
	//图片缓存的基本代码，就是这么简单
	[self.imageView sd_setImageWithURL:imageURL];
    	
`2. sd_setImageWithURL:  completed:`

	//用block 可以在图片加载完成之后做些事情
	[self.imageView sd_setImageWithURL:imageURL completed:^(UIImage *image, NSError *error,SDImageCacheType cacheType, NSURL *imageURL) {
	    //注意image有可能会为空
	  	 NSLog(@"这里可以在图片加载完成之后做些事情");
	}];
    
`3. sd_setImageWithURL:  placeholderImage:`

	//给一张默认图片，先使用默认图片，当图片加载完成后再替换
	[self.imageView sd_setImageWithURL:imageURL placeholderImage:[UIImage 				imageNamed:@"default"]];

`4. sd_setImageWithURL:  placeholderImage:  completed:`

	//使用默认图片，而且用block 在完成后做一些事情
	[self.imageView sd_setImageWithURL:imageURL placeholderImage:[UIImage imageNamed:@"default"] completed:^(UIImage *image, NSError *error, SDImageCacheType cacheType, NSURL *imageURL) {
		//注意image有可能会为空
		NSLog(@"图片加载完成后做的事情"); 
	}];
    
`5. sd_setImageWithURL:  placeholderImage:  options:`
    
	//options 选择加载方式 （SDWebImageRetryFailed）
	[self.imageView sd_setImageWithURL:imageURL placeholderImage:[UIImage imageNamed:@"default"] options:SDWebImageRetryFailed];
	
###### 二、SDWebImage的options的所有选项解释
	
	//失败后重试
	SDWebImageRetryFailed = 1 << 0,
	//UI交互期间开始下载，导致延迟下载比如UIScrollView减速。
	SDWebImageLowPriority = 1 << 1,
	//只进行内存缓存
	SDWebImageCacheMemoryOnly = 1 << 2,
	//这个标志可以渐进式下载,显示的图像是逐步在下载
	SDWebImageProgressiveDownload = 1 << 3,
	//刷新缓存
	SDWebImageRefreshCached = 1 << 4,
	//后台下载
	SDWebImageContinueInBackground = 1 << 5,
	//NSMutableURLRequest.HTTPShouldHandleCookies = YES;
	SDWebImageHandleCookies = 1 << 6,
	//允许使用无效的SSL证书
	SDWebImageAllowInvalidSSLCertificates = 1 << 7,
	//优先下载
	SDWebImageHighPriority = 1 << 8,
	//延迟占位符
	SDWebImageDelayPlaceholder = 1 << 9,
	//改变动画形象
	SDWebImageTransformAnimatedImage = 1 << 10,
	//下载完成后手动设置图片，默认是下载完成后自动放到ImageView上
	SDWebImageAvoidAutoSetImage = 1 << 11
	
====
#### SDWebImage内部实现过程

1. 入口setImageWithURL:PlaceholderImage:options:会先把placeholderImage显示，然后SDWebImageManager根据URL开始处理图片。
2. 进入SDWebImageManager-downloadWithURL:delegate:options:userInfo:,交给SDImageCach从缓存中查找图片是否已经下载queryDiskCacheForKey:delegate:userInfo:。
3. 先从内充图片缓存查找是否有拖，如果内充中已经有图片缓存，SDImageCacheDelegate回调imageCache:didFindImage:forKey:userInfo:到SDWebImageManager。
4.  SDWebImageManageDelegate 回调 webImageManager:didFinishWithImage:到UIImageView+WebCache等前端展示图片。
5.  如果内存缓存中没有过，生成NSInvocationOperation 添加到队里开始从硬盘查找图片是否已经缓存。
6.  根据URLKey在硬盘缓存目录下尝试图区图片文件。这一步是在NSOperation进行操作，所以回到主线程进行结果回调notifyDelegate:.
7.  如果上一操作从硬盘读取到了图片，将图片添加到内存缓存中（如果空闲内存过小，会先清空内存缓存）。SDImageCacheDelegate 回调 imageCache:didFindImage:forKey:userInfo:。进而回调展示图片。
8.  如果从硬盘缓存目录读取不到图片，说明所有缓存都不存在该图片，需要下载图片，回调 imageCache:didNotFindImageForKey:userInfo:
9.  共享或重新生成一个下载器 SDWebImageDownloader 开始下载图片
10.  图片下载由 NSURLConnection 来做，实现相关 delegate 来判断图片下载中、下载完成和下载失败。
11.  connection:didReceiveData: 中利用 ImageIO 做了按图片下载进度加载效果。
12.  connectionDidFinishLoading: 数据下载完成后交给 SDWebImageDecoder 做图片解码处理。
13.  图片解码处理在一个 NSOperationQueue 完成，不会拖慢主线程 UI。如果有需要对下载的图片进行二次处理，最好也在这里完成，效率会好很多。
14.  在主线程 notifyDelegateOnMainThreadWithInfo: 宣告解码完成，imageDecoder:didFinishDecodingImage:userInfo: 回调给 SDWebImageDownloader
15. imageDownloader:didFinishWithImage: 回调给 SDWebImageManager 告知图片下载完成。
16. 通知所有的 downloadDelegates 下载完成，回调给需要的地方展示图片.
17. 将图片保存到 SDImageCache 中，内存缓存和硬盘缓存同时保存。写文件到硬盘也在以单独 NSInvocationOperation 完成，避免拖慢主线程。
18. SDImageCache 在初始化的时候会注册一些消息通知，在内存警告或退到后台的时候清理内存图片缓存，应用结束的时候清理过期图片。
19. SDWI 也提供了 UIButton+WebCache 和 MKAnnotationView+WebCache，方便使用。
20. SDWebImagePrefetcher 可以预先下载图片，方便后续使用。 

====
#### SDWebImage底层实现原理
SDWebImage沙盒缓存机制，主要有三块组成

1. 内存图片缓存
2. 内存操作缓存
3. 磁盘沙盒缓存

可以有图片解释：

![SDWebImage原理图片](http://images.cnitblog.com/blog/721839/201502/081450075631345.png)

=========