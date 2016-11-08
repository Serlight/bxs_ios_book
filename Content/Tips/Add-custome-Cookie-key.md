# UIWebview添加多个自定义的cookie

在官方文档很容易可以找到通过如下代码可以添加一个自定义的cookie key，vlaue，

    NSHTTPCookie *mainDomainCOMCookie = [NSHTTPCookie cookieWithProperties: cookieProperties];
                                         [NSDictionary dictionaryWithObjectsAndKeys:
                                          domain, NSHTTPCookieDomain,
                                         path, NSHTTPCookiePath,
                                          customeKey,  NSHTTPCookieName,
                                          customeValue, NSHTTPCookieValue,
                                          nil]];

这样添加了一个自定义的key：customeKey， value： customeValue。因为添加自定义的 cookie value 的时候， key 的 property 的关键字是 NSHTTPCookieName, Value 的关键字是 NSHTTPCookieValue； 所以一次只能添加一对。 

那么怎么去添加多个自定义的key-value呢

	NSDictionary *customeProperties = @{key1: value1, key2: value2};
	        [customeProperties enumerateKeysAndObjectsUsingBlock:^(NSString * key, id  _Nonnull obj, BOOL * _Nonnull stop) {
	            NSHTTPCookie *mainDomainCOMCookie = [NSHTTPCookie cookieWithProperties:
	                                                         [NSDictionary dictionaryWithObjectsAndKeys:
	                                                          domain, NSHTTPCookieDomain,
	                                                          path, NSHTTPCookiePath,
	                                                          key,  NSHTTPCookieName,
	                                                          obj, NSHTTPCookieValue,
	                                                          nil]];
	            
	            [[NSHTTPCookieStorage sharedHTTPCookieStorage] setCookie:mainDomainCOMCookie];
	        }];

通过上面的代码可以添加两组key-value. 其实在 sharedHTTPCookieStorage 里面产生了两个 NSHTTPCookie。