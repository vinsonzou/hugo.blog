+++
description = ""
date = "2021-06-14T11:18:08+08:00"
title = "巧用 cache-control: s-maxage 优化CDN和浏览器缓存同步"
tags = ["cdn"]

+++

在实际使用中会发现，通常源站资源发生变化后，我们会尝试刷新CDN。但是由于用户浏览器依旧保留旧资源，造成访问不一致的问题。解决思路如下:

在源站添加如下header:

> cache-control: max-age=0,s-maxage=604800

**解释说明**

CDN 通常会遵循这个头，如果仅仅设置cache-control: max-age=0，固然每次浏览器会向 CDN 请求验证资源新鲜度，但是也会造成 CDN 每次都回源验证，会引起缓存击穿的问题。而静态资源通常更新并不频繁，我们可能会期望浏览器仅仅找 CDN 验证新鲜度就够了，CDN 不需要回源。对于这种场景，在cache-control头中添加s-maxage参数就够了。这个参数 CDN 通常会处理，优先级比max-age高。这样就实现了我们的需求。

Ps: 最理想的方案仍然是[cache busting](https://www.keycdn.com/support/what-is-cache-busting)。此方案仅适用于实在无法做到的静态资源。对于一些不需要太重视新鲜度问题的资源，仅仅max-age参数就够了，这样可能尽可能在浏览器段缓存资源，减少 CDN 的请求流量。

**CDN厂商 s-maxage 支持情况**
- [阿里云CDN【支持】](https://help.aliyun.com/document_detail/27136.htm)
- [腾讯云CDN【不支持】](https://cloud.tencent.com/document/product/228/47672)
- [华为云CDN【不支持】](https://support.huaweicloud.com/usermanual-cdn/cdn_01_0116.html)
- [Akamai CDN【支持】](https://developer.akamai.com/blog/2020/10/23/configure-caching-easily-s-maxage-now-ga)
- [Google cloud CDN【支持】](https://cloud.google.com/cdn/docs/caching?hl=zh_cn#maximum-size)
- [AWS cloudfront CDN【支持】](https://docs.amazonaws.cn/AmazonCloudFront/latest/DeveloperGuide/Expiration.html#ExpirationDownloadDist)

### Cache-Control 示例说明
- Cache-Control: public max-age=3600 //本地缓存和 CDN 缓存均缓存 1 小时；
- Cache-Control: private immutable   //不能缓存在 CDN，只能缓存在本地。并且一旦被缓存了，则不能被更新；
- Cache-Control: no-cache //不能缓存。如果一定要缓存的话，确保对其进行了二次验证；
- Cache-Control: public max-age=3600 s-maxage=7200  //本地缓存 1 小时，CDN 上缓存 2 小时；
- Cache-Control: public max-age=3600 proxy-revalidate   //本地和 CDN 均缓存 1 小时。但是如果 CDN 收到请求，则尽管已经缓存了 1 小时，还是要检查源中文档是否已经被改变。

### 参考

- https://blog.dteam.top/posts/2021-06/use-s-maxage-for-cdn.html
