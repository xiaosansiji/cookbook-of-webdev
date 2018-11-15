# Web 站点性能优化

- 优化什么
- 通用优化方法
- React 技术栈下的优化方法



## 优化什么？

从浏览器角度看待网页加载的过程：

![image-20181109142635337](https://github.com/xiaosansiji/cookbook-of-webdev/blob/master/performance-optimization/image-20181109142635337.png)

换个角度，从用户体验角度看待网页加载：

![image-20181109144732772](https://github.com/xiaosansiji/cookbook-of-webdev/blob/master/performance-optimization/google-performance.png)

由此我们可以得到一些常见性能指标：白屏时间、首次有效渲染时间、用户可交互时间、整页时间、DNS时间、CPU占用率、动画帧率。

当然这些指标并非同样重要，很多时候需要根据产品形态、用户需求等赋予权重（构建用户为中心的性能模型）。

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

为了减少同一域下网络请求数，我们可以采用如下几种方式：

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

[^注意]: 在保证用户浏览体验的前提下，我们应该尽可能的减小站点请求资源大小，但这并不是说资源小了页面渲染性能一定会提升：由于一个 TCP 请求窗口在大部分情况下是 1480*10/1024 = 14.45K ，所以下载一个 15K 的文件和一个 28K 的文件从网络传输角度看时间几乎是一样的，因为这两个文件都需要两次往返传输。所以在优化静态资源大小时，14K 及它的倍数可以看作是一个个级别，级别之间的文件大小传输时间没有太大区别。当然尽量减小资源大小可以在移动场景下可以减少用户流量消耗，并非没有意义。

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

- 多路复用，降低了重复建立 TCP 连接的性能损耗，也可以减少 TCP 慢启动带来的性能影响
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

##### 尽早开始渲染

能够尽早的拿到静态资源构建 render 树才能尽早开始页面渲染，除了优化网络传输等方法外，我们应当尽量减少 CSS/js 资源请求对渲染过程对影响。

![image-20181112161111566](https://github.com/xiaosansiji/cookbook-of-webdev/blob/master/performance-optimization/browser-render.png)

这里有必要指出：各浏览器内引擎一般会做优化，不是等整个 HTML 解析完才开始绘制，而是部分构建成功 render 树后就开始绘制，以求能更快的展现页面。

- 不要让外部样式或脚本 block 渲染过程
- 优先加载关键 CSS

link 尽量在 head 中，script 尽量在 body 底部，js 执行并不会影响首屏时间，但可能截断首屏的内容。

收集开始渲染页面的第一个可见部分所需的所有 CSS（关键CSS）并将其内联添加到页面的 `<head>` 中，从而减少网络往返。 由于在慢启动阶段交换包的大小有限，所以关键 CSS 的预算大约是 14 KB。或者通过 HTTP/2 的 Sever Push 在请求 HTML 文档时推送关键 CSS 文件到客户端浏览器，这样可以避免更改关键 CSS 后 HTML 文档缓存失效的问题。

JS 文件可以尝试使用 `async` 和 `defer` ：

```javascript
<script type="text/javascript" src="demo_async.js" async="async"></script>
```

这两个属性的标签都不会阻塞 DOM 树的更新，都是在页面渲染完成后执行，不同点在于：

- async 属性的脚本会按加载完成后就执行 
- defer 当有多个 defer 属性的脚本时会按照标签出现的先后顺序执行

##### 懒加载与预加载

懒加载：

懒加载主要为了提升用户可交互时间这个指标，比如，我们可以优先加载用户可视范围内的图片资源，其他的等用户滚动到可视范围内了再进行加载：

```html
<img src="" data-original="images/xx.png"/>
```

预加载：

预加载其实会牺牲一部分首屏渲染性能，换来后续用户操作的流畅性，在访问站点时有些页面会加载一些本页面没有用到的图片等资源，这些资源会缓存在浏览器内存或硬盘上，这样在后续访问其他页面时就可以加快页面展现，利用 js、CSS 或 ajax 等都可以实现，如利用 js 构建 `Image` 对象：

```javascript
var img = new Image();
img.src = "img/example.jpg";
```

##### 骨架屏

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

##### 消抖与节流

如果一个事件频发的被触发：

- 消抖 debounce：固定一次事件发生 n 毫秒后执行一次，n 毫秒内又触发1次事件，则重新延后 n 毫秒执行
- 节流 throttle：预设一个延迟周期，在一个周期内只执行一次事件

举例：输入联想、滚动事件

underscore、lodash 等工具库中都有消抖与节流函数。

### 减少内存泄漏

不同于 C/C++ 的手动管理内存，JS 有自动垃圾回收机制，但是在某些情况下也会出现内存不能及时回收的情况：如果我们使用了闭包后未将相关资源加以释放，或者引用了外链后未将其置空（比如给某DOM元素绑定了事件回调，后来却remove了该元素），及某些低版本浏览器下的循环引用等都会造成内存泄漏的情况发生。

针对 React 技术栈，在 shouldComponentUpdate 或者 componentWillUpdate 方法中调用 setState 会造成 setState 的循环调用，也会造成浏览器内存占用快速上升，最终使页面卡死。

## React 技术栈下的优化方法

### 默认的优化策略

#### Virtual DOM 与 Diff 算法

![flight-status](https://github.com/xiaosansiji/cookbook-of-webdev/blob/master/performance-optimization/flight-status.jpeg)

假设我们有一个展示航班信息的列表，大约1000行，每5s自动刷新一次，如果我们每次拿到后台接口返回的列表数据后都暴力的直接 `targetDOM.innerHtml() ` 会有比较严重的性能问题。

在 jQuery 时代我们主要的解决思路是两点：

1. 不在 DOM 元素上直接进行添加或修改操作
2. 在内存中维护一个一维数组结构的对象当作一层缓存，每次拿到新的数据时与原先对象做比对，确定需要更新的元素后做精确更新，从而尽量复用 HTML 文档中原有的 DOM 结构

通过以上两种方式我们可以尽量减少 reflow、repaint 从而能够针对这一特定场景做出性能优化，但一方面这样做大大增加了我们实现业务的复杂度，另一方面思路二有缺陷，我们要面对的是 DOM 的树形结构，仅仅考虑一维数组结构并不具有通用性。

React 提出的通用解决方案是：

1. Virtual DOM  用轻量的 JS 对象替代原生 DOM，隔离对原生 DOM 的直接操作
2. Diff 算法针对树形结构提供了优化过的比对策略

![tree-dom](https://github.com/xiaosansiji/cookbook-of-webdev/blob/master/performance-optimization/tree-dom.png)

传统树结构比对算法是循环递归的方式对节点进行遍历，其算法时间复杂度为 O(n^3)，而 React 中实现的 Diff 算法的时间复杂度优化到了 O(n)，具体实现思路为针对前端场景做了三个假设：

1. DOM 节点跨层级的移动操作特别少，可以忽略不计
2. 拥有相同类的两个组件会形成相似的树形结构，拥有不同类的两个组件会形成不同的树形结构
3. 同一层级的一组子节点，可以通过唯一 id 进行区分

针对这三个假设，React 在 Tree、Component、Element 三个粒度的对比上进行了特殊优化

![react-diff](https://github.com/xiaosansiji/cookbook-of-webdev/blob/master/performance-optimization/react-diff.png)

#### setState 的特殊处理

浏览器对原生 DOM 的修改操作已经做了一层优化：临近的几次对某个 DOM 对象的操作会合并成一次

实际上在一些 MVVM 的框架甚至小程序等都提供类似 setState 这样的接口来批量/延迟更新数据。

React 中的 setState 方法通过一个队列机制实现 state 更新，当调用 setState 的时候，会将需要更新的 state 合并之后放入状态队列，而不会立即更新。这与节流/消抖策略的思路一致：减少频繁发生的事件对页面渲染性能带来的影响。反应在组件更新上，每次在一个组件中调用 setState ，React 都会将这个组件标记为dirty。在一次事件循环结束后，React会搜索所有被标记为 dirty 的组件，并对它们进行重新渲染。

![setstate-dirty](https://github.com/xiaosansiji/cookbook-of-webdev/blob/master/performance-optimization/setstate-dirty.png)

当然我们有必要说明，setState 并不是完全异步的，只是说有一个队列机制。React 内部在处理 React 合成事件和生命周期函数中的 setState 调用时是延后处理更新操作，而对于原生事件和特殊的 setTimeout 中的 setState 调用则会立即更新。

```javascript
class App extends React.Component {
  state = { val: 0 }

  componentDidMount() {
    this.setState({ val: this.state.val + 1 })
    console.log(this.state.val)

    this.setState({ val: this.state.val + 1 })
    console.log(this.state.val)

    setTimeout(_ => {
      this.setState({ val: this.state.val + 1 })
      console.log(this.state.val);

      this.setState({ val: this.state.val + 1 })
      console.log(this.state.val)
    }, 0)
  }

  render() {
    return <div>{this.state.val}</div>
  }
}
// 0
// 0
// 2
// 3
```

#### React 的事件机制

setState 一节中我们提到了 React 合成事件与原生事件的概念，如果 DOM 上绑定了过多的事件处理函数，整个页面响应以及内存占用可能都会受到影响。React 为了避免这类 DOM 事件滥用，同时屏蔽底层不同浏览器之间的事件系统差异，实现了一个中间层——SyntheticEvent（合成事件）。

其实在原生 JS 中已经在利用事件冒泡机制来实现事件委托，从而提高页面性能；

```javascript
var oUl = document.getElementById("ul1");
　　oUl.onclick = function(ev){
　　　　var ev = ev || window.event;
　　　　var target = ev.target || ev.srcElement;
　　　　if(target.nodeName.toLowerCase() == 'li'){
　　　　　　　  alert(target.innerHTML);
　　　　}
　　}
```

jQuery 中提供 on 方法简化了声明事件委托的方式。

在 React 组件中声明事件时并没有直接绑定到对应的原生 DOM 上，而是绑定在 document 节点上，依靠 React 自己实现的一套事件冒泡机制进行事件委托，同时为了减少频繁创建合成事件对象，在内部维护了一个事件对象池。同时因为合成事件机制屏蔽各浏览器事件实现机制的差异，也提升了兼容性。

#### React Fiber

我们先来看一下 React 16 以前完成一次组件渲染的过程是怎样的：

假设有如下结构的组件

![react-fiber-1](https://github.com/xiaosansiji/cookbook-of-webdev/blob/master/performance-optimization/react-fiber-1.png)

在 mount 阶段渲染过程中各组件声明周期调用顺序如下

![react-fiber-2](https://github.com/xiaosansiji/cookbook-of-webdev/blob/master/performance-optimization/react-fiber-2.png)

当我们的组件树很深时，这一次 mount 过程可能需要几百毫秒，而在这一过程中 React 的渲染会一直占用 js 引擎，这时如果有用户点击/输入等操作浏览器将不会处理，一直到渲染结束才会响应，从而造成页面“卡顿”的感觉。

为了解决这个问题，React 16 中引入了新的 Fiber 机制将 React 的渲染过程“分片”，在每个分片任务结束后检查是否有更高级的任务需要执行（如用户操作），若有责优先处理高级任务否则继续执行下一分片的任务。这样 React 渲染的过程就变成了下面这样：

![react-fiber-3](https://github.com/xiaosansiji/cookbook-of-webdev/blob/master/performance-optimization/react-fiber-3.jpg)

### 需要手动干预的方法

#### 绑定事件处理函数 this

```javascript
class Link extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      count: 0
    }
  }
  handleClick(e) {
    e.preventDefault();
    this.setState({ count: this.state.count + 1 })
  }
  render() {
    return <a href="#" onClick={this.handleClick}>Clicked me {this.state.count} times.</a>    
  }

}

ReactDOM.render(<Link/>, document.querySelector("#root"))
// Uncaught TypeError: Cannot read property 'setState' of undefined
```

我们在开始使用 React 的时候经常出现上面的错误，这其实是因为调用事件处理函数时 this 指向发生了变化，详情参考[这里](https://github.com/mqyqingfeng/Blog/issues/7)。而为了避免这种情况我们有以下四种方式来绑定 this：

**render 方法中使用 bind：**

```javascript
<a href="#" onClick={this.handleClick.bind(this)}>
    Clicked me {this.state.count} times.
</a>
```

但这种方式其实存在性能问题：JS 中 bind 方法会生成一个新的函数，对于 render 中的自组件来说 props 中的属性有更新，会造成一次额外的渲染。

**render 方法中使用肩头函数：**

```javascript
<a href="#" onClick={e => this.handleClick(e)}>
    Clicked me {this.state.count} times.
</a>
```

与使用 bind 一样，每次都都会生成一个新函数对象，造成额外渲染。

**在构造函数内提前绑定：**

```javascript
class Link extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      count: 0
    }
    // 重点在这里
    this.handleClick = this.handleClick.bind(this);
  }
  handleClick(e) {
    e.preventDefault();
    this.setState({ count: this.state.count + 1 })
  }
  render() {
    return <a href="#" onClick={this.handleClick}>Clicked me {this.state.count} times.</a>    
  }
}

ReactDOM.render(<Link/>, document.querySelector("#root"))
```

这样提前确定 this 指向可以避免多次绑定带来的性能问题，但是函数的定义、使用和 this 绑定分在三个不同地方，降低了代码可读性。

**在 class properties 中使用箭头函数：**

```javascript
class Link extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      count: 0
    }
  }
  handleClick = e => {
    e.preventDefault();
    this.setState({ count: this.state.count + 1 })
  }
  render() {
    return <a href="#" onClick={this.handleClick}>Clicked me {this.state.count} times.</a>    
  }
}
ReactDOM.render(<Link/>, document.querySelector("#root"))
```

推荐做法，在 React 16 和 ES6 条件下可以使用。

#### componentShouldUpdate

```javascript
shouldComponentUpdate(nextProps, nextState) {
    if (this.props.color !== nextProps.color) {
      return true;
    }
    if (this.state.count !== nextState.count) {
      return true;
    }
    return false;
  }
```

#### PureComponent

声明组件时，使用 `React.PureComponent` 替代 `React.Component`，PureComponent 相当于在 shouldComponentUpdate 中做了一次“浅比较”。

对于简单的数据结构来说，浅比较已经足够，但是对于拥有复杂数据结构的对象来说就会有问题：

```javascript
class WordAdder extends React.PureComponent {
  state = {
    user: ['aaa']
  };

  handleAddClick() {
    const users = this.state.users;
    users.push('bbb');
    this.setState({ users });
  }

  render() {
    ...
  }
}
```

这里当我们触发点击事件回调向用户数组中加入一个用户时，组件并没有如我们想象一样的进行一次新的渲染，因为我们直接在原来的数组对象上增加新用户数据，浅比较时会认为两者相等。

这里有必要简单介绍一下 JS 中的数据类型分为原始/值类型和对象/引用类型，它们的内存示意图如下：

![type-memory](https://github.com/xiaosansiji/cookbook-of-webdev/blob/master/performance-optimization/type-memory.png)

当修改原始类型的值时会改变在栈内存上对应的值，而改变对象类型时其栈上存储的地址却不会改变，造成浅比较想等，深比较不等的结果。

为了解决这个问题我们会要求针对复杂数据结构的对象的修改，不要直接在原对象上操作，而是生成一个新对象：

```javascript
handleAddClick() {
    this.setState({ users: [ ...this.state.users, 'bbb' ] });
  }
```

利用 ES6 的扩展运算符，我们可以很便捷的生成一个全新的对象，这样在做浅比较时就得到不想等的结果，从而触发新的渲染。

#### Immutable Data

新的问题来了，对拥有复杂结构的对象类型来说，每次新创建一个新对象会有一定的性能损耗，每次深比较性能开销也比较大，有没有什么方式既可以浅比较又能不完全创建新对象呢？

Immutable 即是这样的解决方案：对 Immutable 对象的修改每次都会返回新的 Immutable 对象，与完全创建新对象不同的是在这一过程中数据结构是共享的。

![immutable](https://github.com/xiaosansiji/cookbook-of-webdev/blob/master/performance-optimization/immutable.gif)

#### Fragments

`React.Fragment` 也是 React 16 中新引入的一个特性，在 React 16 之前，因为 React 要求 render 函数返回的 elements  必须有根节点，所以在渲染数组类型的数据结构时我们经常会用一个空 div 包裹我们的返回内容：

```javascript
function () {
    return (
        <div>
            <div>xxx</div>
            <div>yyy</div>
            <div>zzz</div>
        </div>
    );
}
```

但这样增加了 HTML 文档上的 DOM 对象，对渲染性能也会造成影响。

React 16 针对这种场景提出了两种解决方案：

可以使用`React.Fragment` 

```javascript
function () {
    return (
        <React.Fragment>
            <div>xxx</div>
            <div>yyy</div>
            <div>zzz</div>
        </React.Fragment>
    );
}
```

当然 16 版本中也开始支持返回 elements 数组

```javascript
function () {
    return [
         <div>xxx</div>
         <div>yyy</div>
         <div>zzz</div>
    ];
}
```

#### Context 传值

![](https://github.com/xiaosansiji/cookbook-of-webdev/upload/master/performance-optimization/react-context.png)

对站点全局语言切换或者像兄弟组件间消息通讯等场景，通过父子组件的 props 属性一层层向下传递不但非常繁琐，而且也会有多次渲染带来的性能问题。这时我们可以利用 React 提供的 Context 接口传递数据，实际上 react-redux/react-router 等都是通过 Context 传递数据：

```javascript
function getRouter() {
    return (
    <LocaleProvider locale={zhCN}>
      <ConnectedRouter>
        <AuthorizedRoute
          path="/"
          render={props => <BasicLayout {...props} />}
          authority={['admin', 'user']}
          redirectPath="/user/login"
         />
      </ConnectedRouter>
    </LocaleProvider>
  );
}
```

当然要注意的是在 React 16 中对 Context 接口进行了重构，在使用时注意兼容性修改。

## 参考链接

[WebP 图片格式简介](https://www.cnblogs.com/upyun/p/7813319.html)

[Vue项目骨架屏注入实践](https://www.jianshu.com/p/3da35af1e4c2)

[网站性能优化实战——从12.67s到1.06s的故事](https://juejin.im/post/5b0b7d74518825158e173a0c#heading-7)

[React 源码剖析系列 － 不可思议的 react diff](https://zhuanlan.zhihu.com/p/20346379?spm=a2c4e.11153940.blogcont586669.10.7aa21308YhIW92)

[图解React Diff算法及新架构Fiber](https://yq.aliyun.com/articles/586669)

[React 是怎样炼成的](https://segmentfault.com/a/1190000013365426)

[React Fiber是什么](https://zhuanlan.zhihu.com/p/26027085)

[深入理解React16之：（一）.Fiber架构](https://www.jianshu.com/p/bf824722b496)

[玩转 React（六）- 处理事件](https://segmentfault.com/a/1190000011877137)