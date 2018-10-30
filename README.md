## 网络篇

### webpack打包 和 Gzip

1.dll 第三方库打包

2.按需加载

3.Gzip。开启服务端Gzip。在request的请求头部 ```accept-encoding:gzip```

#### Gzip 压缩背后的原理，是在一个文本文件中找出一些重复出现的字符串、临时替换它们，从而使整个文件变小。根据这个原理，文件中代码的重复率越高，那么压缩的效率就越高，使用 Gzip 的收益也就越大。反之亦然



### 图片优化

图片一直是性能优化中必不可少的环节。与其说图片优化不如说 各种场景下的权衡利弊的选择更好，在保证图片能满足要求的情况下，尽量让图片小。

#### 不同业务场景下的图片方案选型

常用的图片格式有。JPG/JPGE ，PNG，SVG，webP，base64等，经常在嘴边说的，雪碧图，也在一线开发中发光发热。

##### 前置知识：二进制位数与色彩的关系

在计算机中，像素用二进制数来表示。不同的图片格式中像素与二进制位数之间的对应关系是不同的。一个像素对应的二进制位数越多，它可以表示的颜色种类就越多，成像效果也就越细腻，文件体积相应也会越大。



### JPEG/JPG

关键字：**有损压缩(最大的特点)，体积小、加载快、不支持透明**， 

使用场景：

JPG 适用于呈现色彩丰富的图片，在我们日常开发中，JPG 图片经常作为大的**背景图、轮播图或 Banner 图**出现。



### PNG-8 与 PNG-24

关键字：**无损压缩、质量高、体积大、支持透明**

PNG8/PNG24. 后面的数字代表二进制位数，如果不考虑性能，推荐PNG24；考虑性能优化的，推荐使用PNG8；他唯一的bug就是体积太大。具体使用PNG8/24 就需要去跟UI小姐姐去沟通了。

使用场景：

PNG适用于小的 Logo、颜色简单且对比强烈的图片或背景等。



### SVG

关键字：**文本文件、体积小、不失真、兼容性好**

SVG 与 PNG 和 JPG 相比，**文件体积更小，可压缩性更强，可以无限放大而不失真**

使用场景

一般是用来做icon，会有就行。[在线矢量图形库](http://www.iconfont.cn/)



### Base64

关键字：**文本文件、依赖编码、小图标解决方案**

Base64 是一种用于传输 8Bit 字节码的编码方式，通过对图片进行 Base64 编码，我们可以直接将编码结果写入 HTML 或者写入 CSS，从而减少 HTTP 请求的次数。

使用场景：

图片的实际尺寸很小；
图片无法以雪碧图的形式与其它小图结合（合成雪碧图仍是主要的减少 HTTP 请求的途径，Base64 是雪碧图的补充）；
图片的更新频率非常低（不需我们重复编码和修改文件内容，维护成本较低）；



### WebP

关键字：**年轻的全能型选手，兼容问题**

使用场景

现在不是哪种场景适合使用WebP，而是说哪种浏览器支持这种格式。

如果选择了WebP 就要考虑他的兼容问题了。

提供一种解决方案：

把判断工作交给后端，由服务器根据 HTTP 请求头部的 Accept 字段来决定返回什么格式的图片。当 Accept 字段包含 image/webp 时，就返回 WebP 格式的图片，否则返回原图。这种做法的好处是，当浏览器对 WebP 格式图片的兼容支持发生改变时，我们也不用再去更新自己的兼容判定代码，只需要服务端像往常一样对 Accept 字段进行检查即可。





## 浏览器缓存机制和缓存策略介绍。 **划重点，必考！！！**

缓存可以减少网络 IO 消耗，提高访问速度。浏览器缓存是一种操作简单、效果显著的前端性能优化手段。对于这个操作的必要性，Chrome 官方给出的解释似乎更有说服力一些：

> 通过网络获取内容既速度缓慢又开销巨大。较大的响应需要在客户端与服务器之间进行多次往返通信，这会延迟浏览器获得和处理内容的时间，还会增加访问者的流量费用。因此，缓存并重复利用之前获取的资源的能力成为性能优化的一个关键方面。

浏览器缓存机制有四个方面，从获取资源时请求的优先级依次排列如下：

1. **Memory Cache **
2. **Service Worker Cache**
3. **HTTP Cache**
4. **Pash Cache**



先来一张Network图，

![线上Network图](https://user-gold-cdn.xitu.io/2018/9/20/165f714800e5be49?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

给size栏一个特写

![size栏](https://user-gold-cdn.xitu.io/2018/9/20/165f715425bd73b6?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

看size栏中，形如(from xxx)即来自相应的资源 , “from memory cache”对标到 Memory Cache 类型，“from ServiceWorker”对标到 Service Worker Cache 类型。至于 Push Cache，这个比较特殊，是 HTTP2 的新特性。



##### HTTP Cache

HTTP缓存 分 **强缓存** 和 **协商缓存** 。在强缓存没有被命中的情况下，才会走协商缓存。

###### 强缓存特征

强缓存 是通过 Cache-Control 和 expires来实现的，浏览器根据Cache-Contol和expires来判断是否命中强缓存，如果命中，**不与服务端发生通信**

**Cache-Control 相对于 expires 更加准确，它的优先级也更高。当 Cache-Control 与 expires 同时出现时，我们以 Cache-Control 为准**。

Cache-Control 可以视作是 expires 的**完全替代方案**。在当下的前端实践里，我们继续使用 expires 的唯一目的就是**向下兼容**。



在响应头中，

![响应头](https://user-gold-cdn.xitu.io/2018/9/20/165f52bf6e844b85?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

```cache-control: max-age=31536000
expires: Wed, 11 Sep 2019 16:12:18 GMT
cache-control: max-age=31536000
```

在 Cache-Control 中，我们通过 max-age 来控制资源的有效期。max-age 不是一个时间戳，而是一个时间长度。在本例中，max-age 是 31536000 秒，它意味着该资源在 31536000 秒以内都是有效的，完美地规避了时间戳带来的潜在问题。



Cache-Control 以下用法非常常见：

```
cache-control: max-age=3600, s-maxage=31536000
```

**s-maxage 优先级高于 max-age，两者同时出现时，优先考虑 s-maxage。如果 s-maxage 未过期，则向代理服务器请求其缓存内容。**  s-maxage 仅在代理服务器生效，客户端仅考虑max-age即可。

在依赖各种**代理**的大型架构中，我们不得不考虑**代理服务器**的缓存问题。s-maxage 就是用于表示 cache 服务器上（比如 cache CDN）的缓存的有效时间的，并只对 public 缓存有效。

**Cache-control 中一些相对概念的理解**

#### public 与 private

public 表示 该资源即在客户端缓存又在服务端缓存。

private 默认值 仅在客户端缓存。

#### no-store与no-cache

no-store 绕过浏览器端缓存直接向服务端去确认该资源是否过期（即走我们下文即将讲解的协商缓存的路线）；

no-cache 绕过浏览器缓存和服务端缓存，直接去服务端发送请求，并下载完整的响应。



### 协商缓存：浏览器与服务器合作之下的缓存策略

依赖于服务器和浏览器之间的通信。

在协商缓存的机制下，浏览器发送询问信息，根据服务器返回的信息再去决定是下载完整的响应，还是从本地获取缓存的资源。

在服务端资源没变的情况下，返回304状态码，调用浏览器缓存。



实现

Etag和If-None-Match

Last-Modified 是一个时间戳，如果启用了协商缓存，会随着首次请求的响应头返回，

```
Last-Modified: Fri, 27 Oct 2017 06:35:57 GMT
```

随后每次请求都会带上，一个叫 If-Modified-Since 的时间戳字段，它的值正是上一次 响应 返回给它的 last-modified 值：

```
If-Modified-Since: Fri, 27 Oct 2017 06:35:57 GMT
```

服务器接收到这个时间戳后，会比对该时间戳和资源在服务器上的最后修改时间是否一致，从而判断资源是否发生了变化

Etag 和 Last-Modified 类似，当首次请求时，我们会在响应头里获取到一个最初的标识符字符串

```
ETag: W/"2a3b-1602480f459"
```

那么下一次请求时，请求头里就会带上一个值相同的、名为 if-None-Match 的字符串供服务端比对了：

```
if-None-Match:W/"2a3b-1602480f459"
```

**Etag 并不能替代 Last-Modified，它只能作为 Last-Modified 的补充和强化存在。 Etag 在感知文件变化上比 Last-Modified 更加准确，优先级也更高。当 Etag 和 Last-Modified 同时存在时，以 Etag 为准**



### HTTP 缓存指南 (这个图是精髓啊)

![](https://user-gold-cdn.xitu.io/2018/9/20/165f701820fafcf8?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

当我们的资源内容不可复用时，直接为 Cache-Control 设置 no-store，拒绝一切形式的缓存；否则考虑是否每次都需要向服务器进行缓存有效确认，如果需要，那么设 Cache-Control 的值为 no-cache；否则考虑该资源是否可以被代理服务器缓存，根据其结果决定是设置为 private 还是 public；然后考虑该资源的过期时间，设置对应的 max-age 和 s-maxage 值；最后，配置协商缓存需要用到的 Etag、Last-Modified 等参数。











原文链接：

https://juejin.im/book/5b936540f265da0a9624b04b

