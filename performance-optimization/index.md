# Web 站点性能优化

## 优化什么？

一次网页加载的过程：

![image-20181109142635337](https://github.com/xiaosansiji/cookbook-of-webdev/blob/master/performance-optimization/image-20181109142635337.png)

常见性能指标：白屏时间、首次有效渲染时间、用户可交互时间、整页时间、DNS时间、CPU占用率、动画帧率。

换个角度，从用户浏览角度看待网页加载：

![image-20181109144732772](https://github.com/xiaosansiji/cookbook-of-webdev/blob/master/performance-optimization/google-performance.png)

构建用户为中心的性能模型：这些指标并非同样重要，很多时候需要根据产品形态、用户需求等赋予权重。

至于怎么获取这些指标，需要一些监控的手段和工具（Lighthouse/主动监控等），可以参见之前分享的《手摸手，教你做一个前端监控系统》。

## 通用优化方法

推荐书单《高性能网站建设指南》、《[雅虎性能优化建议](https://developer.yahoo.com/performance/rules.html?guccounter=1)》

### 核心思想

- 优化网络传输
- 优化渲染性能
- 减少内存泄露

### 优化网络传输

#### 减少网络请求次数

浏览器针对同一域名的并行网络请求数是有限制的，根据浏览器厂家/版本或站点使用的 HTTP 协议版本的的不同限制数也不相同，大概在 2-8 个。

假如一个页面有120个静态资源（css、js、img），并且所有资源都在一个域名下，使用的浏览器最大网络并行请求资源数是6，如果所有请求时间都是一样的，每个文件加载需要500ms，则所有资源加载完成需要 120/6 * 0.5 = 10s 的时间。

- 使用 Webpack、Gulp 等工具合并 JS/CSS 资源
- Sprite 图
- base 64
- 使用 CDN

##### Sprite 图

将很多图片整合到一个图片文件中，利用 CSS 的 `background-position` 属性在渲染时定位到指定图片的位置。

![image-20181109150646552](https://github.com/xiaosansiji/cookbook-of-webdev/blob/master/performance-optimization/sprite-image.png)

##### base 64

```html
<img src=“data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAAkCAYAAABIdFAMAAAAGXRFWHRTb2Z0d2FyZQBBZG9iZSBJbWFnZVJlYWR5ccllPAAAAHhJREFUeNo8zjsOxCAMBFB/KEAUFFR0Cbng3nQPw68ArZdAlOZppPFIBhH5EAB8b+Tlt9MYQ6i1BuqFaq1CKSVcxZ2Acs6406KUgpt5/LCKuVgz5BDCSb13ZO99ZOdcZGvt4mJjzMVKqcha68iIePB86GAiOv8CDADlIUQBs7MD3wAAAABJRU5ErkJggg%3D%3D”/>
```

有些页面上会有这样的 `img` 标签，对于比较小的图片可以通过这种方式内联到 HTML/CSS 文件中，减少一次网络请求。但是要慎重，跟 Sprite 图一样，base 64 处理后会带来维护困难，及不能有效利用缓存等问题（更改了一个图片的 base 64 编码意味着这个 HTML/CSS 文件就过期了）。

##### CDN

我们刚才说过浏览器对同一域名的并行网络请求数有限制的，而 CDN 的域名地址跟我们的业务主站域名一般时不一致的，这样我们在使用 CDN 时就可以加快并行请求资源的速度。

#### 减小资源大小

- Uglify 压缩代码
- Tree-shaking
- Gzip
- 图片压缩
- 使用 CSS 动画/Canvas 代替 GIF

##### Tree-shaking

与 Uglify 压缩代码的方式不同，Tree-shaking 可以“剔除无用代码”。

##### Gzip

也是一种压缩传输方案：用压缩/解压需要的时间换取网络传输时间。

```http
HTTP/1.1 200 OK
Server: nginx/1.0.15
Date: Sun, 26 Aug 2012 18:21:25 GMT
Content-Type: text/css
Last-Modified: Sun, 26 Aug 2012 15:17:07 GMT
Connection: keep-alive
Expires: Mon, 27 Aug 2012 06:21:25 GMT
Cache-Control: max-age=43200
Content-Encoding: gzip
```

`Content-Encoding: gzip` 标示开启了 Gzip，Nginx 中开启 Gzip 需要如下配置：

```nginx
gzip on;
gzip_min_length 1k;
gzip_buffers 4 16k;
#gzip_http_version 1.0;
gzip_comp_level 2;
gzip_types text/plain application/javascript application/x-javascript text/css application/xml text/javascript application/x-httpd-php image/jpeg image/gif image/png;
gzip_vary off;
gzip_disable "MSIE [1-6]\.";
```

##### 图片压缩

我们通常根据 PC 还是移动站来选择不同格式或压缩比率的图片文件。

在此我们特别介绍下 WebP 图片格式：

WebP 图片格式是由 Google 提出的一种新的网络图片格式，它提供有损和无损两种压缩方式处理原始图片：

![webp-vs-jpg](https://github.com/xiaosansiji/cookbook-of-webdev/blob/master/performance-optimization/webp-vs-jpg.png)

![webp-vs-png](https://github.com/xiaosansiji/cookbook-of-webdev/blob/master/performance-optimization/webp-vs-png.png)

可以看到无论有损还是无损，WebP 都有相当大的压缩优势，缺点是这种格式还未成为标准，兼容性存在问题。

#### 提升传输效率

- CDN
- 有效利用缓存
- HTTP/2

CDN 可以在网络距离更近的节点上提供资源文件访问服务，而且使用比较大众的 CDN 服务商如七牛云等，有较大几率可以复用其他站点的资源。

HTTP 缓存不详细说了，主要是利用 HTTP Header 中的 Etags、Expires 等做强缓存和协商缓存，不从服务器而是本机内存或磁盘上获取资源文件。

这里主要说下 HTTP/2 带来的性能上的提升：

- 多路复用，降低了重复建立 TCP 连接的性能损耗
- HPACK 压缩了 HTTP Header 体积
- Server Push 可以预先将一些资源主动推送到浏览器端，而不是等需要加载的时候才向服务端请求

当然 HTTP/2 需要现升级协议到 HTTPS，切换上有一点成本。

### 优化渲染性能

#### 提升首屏渲染性能

- 尽早开始渲染
- 按需加载/[Code-splitting](https://webpack.github.io/docs/code-splitting.html)
- 懒加载与预加载
- 骨架屏
- SSR

##### 懒加载与预加载

懒加载主要为了提升用户可交互时间这个指标，比如，我们可以优先加载用户可视范围内的图片资源，其他的等用户滚动到可视范围内了再进行加载：

```html
<img src="" data-original="images/xx.png"/>
```

预加载：

![image-20181112161111566](https://github.com/xiaosansiji/cookbook-of-webdev/blob/master/performance-optimization/browser-render.png)



这里有必要指出：各浏览器内引擎一般会做优化，不是等整个 HTML 解析完才开始绘制，而是部分构建成功 render 树后就开始绘制，以求能更快的展现页面。

- 不要让外部样式或脚本 block 渲染过程
- 优先加载关键 CSS

link 尽量在 head 中，script 尽量在 body 底部。

收集开始渲染页面的第一个可见部分所需的所有 CSS（关键CSS）并将其内联添加到页面的 `<head>` 中，从而减少网络往返。 由于在慢启动阶段交换包的大小有限，所以关键 CSS 的预算大约是 14 KB。或者通过 HTTP/2 的 Sever Push 在请求 HTML 文档时推送关键 CSS 文件到客户端浏览器，这样可以避免更改关键 CSS 后 HTML 文档缓存失效的问题。

JS 文件可以尝试使用 async`和`defer ：

```javascript
<script type="text/javascript" src="demo_async.js" async="async"></script>
```

这两个属性的标签都不会阻塞 render 树的构建，不同点在于：

- async 属性的脚本会按加载完成后就执行 
- defer 会一直等到页面渲染完成再执行，并且当有多个 defer 属性的脚本时会按照标签出现的先后顺序执行

#####骨架屏

![skeleton-vs-loading](https://github.com/xiaosansiji/cookbook-of-webdev/blob/master/performance-optimization/skeleton-vs-loading.png)

之前大家为了优化用户体验一般会使用一些 Loading 动画，现在在移动端上更倾向于用骨架屏来提升用户体验。

实现逻辑为预先渲染出结构布局，数据加载完成后再填充数据显示，这样的好处在于不干扰用户操作，使用户对于加载的内容有一个大致的预期。

以下是 facebook 在弱网环境下的表现：

![skeleton-facebook](https://github.com/xiaosansiji/cookbook-of-webdev/blob/master/performance-optimization/skeleton-facebook.gif)

##### SSR

为了减少 SPA 类站点首屏需要浏览器端 JS 动态渲染带来的体验差异，我们可以使用后端渲染的方式首页。

#### 减少 reflow 与 repaint

- 减少 DOM 操作
- 分层渲染与 GPU 加速
- 消抖与节流

减少 DOM 操作的相关知识我们会在 React 部分详细说明，Virtual DOM/Diff 等很多设计都是围绕减少 DOM 操作来的。

##### 分层渲染与 GPU 加速

![image-20181109173423432](https://github.com/xiaosansiji/cookbook-of-webdev/blob/master/performance-optimization/layer.png)

##### ![layers-firfox](https://github.com/xiaosansiji/cookbook-of-webdev/blob/master/performance-optimization/layers-firfox.png)

浏览器在得到 render tree 后渲染页面的大体过程如下：

1. 浏览器根据 Render 树确定多个独立的渲染层
2. CPU将每个层绘制进绘图中
3. 将位图作为纹理上传至GPU（显卡）绘制
4. GPU将所有的渲染层缓存（如果下次上传的渲染层没有发生变化，GPU就不需要对其进行重绘）并复合多个渲染层最终形成我们的图像

（本章节参考[网站性能优化实战——从12.67s到1.06s的故事](https://juejin.im/post/5b0b7d74518825158e173a0c#heading-10)）

有没有可能将绘制带来的影响减少到最小？

```css
transform: translateZ(0);
```

通过使用 CSS transform 属性可以将 DOM 提取到单独的渲染层上，每次更改时影响的范围更小，更精确。

有没有可能减少在 CPU 内计算 DOM 位置，直接使用 GPU 绘制？

CSS 动画就是 GPU 直接绘制，所以我们在实现某些动画效果时应当尽量使用 CSS 动画，而非 JS 动画。

#####消抖与节流

如果一个事件频发的被触发：

- 消抖 debounce：固定一次事件发生 n 毫秒后执行一次，n 毫秒内又触发1次事件，则重新延后 n 毫秒执行
- 节流 throttle：预设一个延迟周期，在一个周期内只执行一次事件

举例：输入联想、滚动事件

underscore、lodash 等工具库中都有消抖与节流函数。

### 减少内存泄漏

不同于 C/C++ 的手动管理内存，JS 有自动垃圾回收机制，但是在某些情况下也会出现内存不能及时回收的情况：如果我们使用了闭包后未将相关资源加以释放，或者引用了外链后未将其置空（比如给某DOM元素绑定了事件回调，后来却remove了该元素），及某些低版本浏览器下的循环引用等都会造成内存泄漏的情况发生。

## React 技术栈下的优化方法

### 默认的优化策略

#### Virtual DOM 与 Diff 算法

#### React 的事件机制

#### setState 的特殊处理

#### React Fiber

### 需要手动干预的方法

#### PureRender

#### componentShouldUpdate

#### Immutable

#### List 下的 key

#### Fragment

#### Context 传值

## 参考链接

[WebP 图片格式简介](https://www.cnblogs.com/upyun/p/7813319.html)

[Vue项目骨架屏注入实践](https://www.jianshu.com/p/3da35af1e4c2)

[网站性能优化实战——从12.67s到1.06s的故事](https://juejin.im/post/5b0b7d74518825158e173a0c#heading-7)



