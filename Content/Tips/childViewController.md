# childViewController

在项目中很多时候使用addChildViewController去加载子的View, 近来项目发现了一个问题， childViewController
不会被回收。那么在childVC里面的timer， 或者notifications就没地方去释放了。会造成内存泄漏。

这样就需要主动释放了。在parentViewController 的dealloc里面去主动调用willMoveToParentViewController， 参数为nil，

	- (void)dealloc {
	    [_childVC willMoveToParentViewController:nil];
	}

然后在childVC里面重写willMoveToParentViewController， 

	- (void)willMoveToParentViewController:(UIViewController *)parent {
	    [super willMoveToParentViewController:parent];
	    if (!parent) {
	        [[NSNotificationCenter defaultCenter] removeObserver:self];
	        [_requestRoomListTimer invalidate];
	        _requestRoomListTimer = nil;
	    }
	}