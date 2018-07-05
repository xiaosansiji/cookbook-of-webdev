# 跨域处理

## 同源策略

> 同源策略：Same Origin Policy

在讨论跨域之前，我们有必要解释下“同源策略”：

同源策略是浏览器的一种安全限制，它限制了从同一个源加载的文档或脚本如何与来自另一个源的资源进行交互，这是一个用于隔离潜在恶意文件的重要安全机制。

我们在前端开发过程中遇到的某些资源，如 DOM、Ajax/Fetch 返回的异步数据及浏览器本地存储的 Cookie、localStorage 等，一般会比较敏感。

想象以下场景，我们登录在线银行，这个站点依赖 Cookie 来保护我们的用户信息：只有正常登录认证过的用户，服务器端会在浏览器中存储一个生成的 Cookie 来标识这个用户，允许正常进行转账操作。而在浏览完这个站点后，我们又访问了一个恶意站点，如果没有同源策略限制，这个恶意网站也可以拿到银行站点存储的 Cookie 并传输给别人，那么在这个 Cookie 过期之前，别人可以把你的银行卡搬空。。

关于 DOM 操作也是一样的，虚假钓鱼网站以 iframe 的方式嵌入了真实网站，我们在访问时可能根本发现不了，如果没有同源策略限制，主页面可以任意操作子页面中 DOM 元素，比如在用户密码输入框上增加了额外的事件监听，同样可以诱导我们暴露个人敏感信息[^1]。

[^1]: 当然为了防止我们的站点被嵌入到钓鱼站点的 iframe 中，我们也可以通过 meta 声明、后端服务器设置等规避，如在 HTML 中增加如下 meta: `<meta http-equiv="X-FRAME-OPTIONS" content="DENY">`，其他设置将在 web 安全章节详细说明。

而我们经常见到的另一些资源，如 CSS 样式表文件、JS 文件、字体文件、图片等因为不具备敏感信息，所以不受同源策略限制，这些静态资源我们可以依赖 CDN 来部署和分发以提高用户体验：一方面 CDN 节点众多，可以就近响应用户请求；另一方面，浏览器在发起请求的时候会默认带上本域的 Cookie，而请求静态资源时明显不需要 Cookie 保护，每次请求都带 Cookie 增大了网络带宽消耗，我们的 CDN 域一般与我们的站点域不一致，可以规避这个问题。

## 判断跨域

同源：协议、域名、端口都相同。详情可以参见[这里](https://developer.mozilla.org/en-US/docs/Archive/Misc_top_level/Same-origin_policy_for_file:_URIs) 。

示例：

| URL                                      | 结果   | 原因                  |
| ---------------------------------------- | ---- | ------------------- |
| `http://store.company.com/dir2/other.html` | 成功   |                     |
| `http://store.company.com/dir/inner/another.html` | 成功   |                     |
| `https://store.company.com/secure.html`  | 失败   | 不同协议 ( https和http ) |
| `http://store.company.com:81/dir/etc.html` | 失败   | 不同端口 ( 81和80)       |
| `http://news.company.com/dir/other.html` | 失败   | 不同域名 ( news和store ) |

## 跨域数据存储访问

### localStorage 与 IndexedDB

[localStorage](https://developer.mozilla.org/zh-CN/docs/Web/Guide/API/DOM/Storage) 和 [IndexedDB](https://developer.mozilla.org/zh-CN/docs/IndexedDB) 均遵循上述同源策略限制，它们不允许 JavaScript 对跨域数据进行读写操作。这限制了我们在不同子站点上共享数据的能力，如我们想在我们产品的两个站点见共享风格配置，用户切换了一个站点的风格（如导航为深色等）后，在同一浏览器中打开的另一个站点也保持同样的风格设置，依靠 localStorage 是做不到的，一般做法是存储在后端 Redis 等服务中，页面加载的时候读取下配置接口。

### Cookie

[Cookie](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Cookies) 的作用域限制与 localStorage、IndexedDB 这些数据存储方案不同，它使用`Domain` 和 `Path` 来设置作用域。

因此我们可以使用 Cookie 来实现 localStorage 与 IndexedDB 章节中用户配置共享的例子，但需要注意，虽然我们可以用 Cookie 可以实现上述需求，但由于浏览器的每次请求都会携带符合条件的 Cookie 数据，这样无疑增大了带宽消耗，所以在实际项目中仍然建议配置信息存储在后端，由指定接口提供。

我们可以利用 Cookie 的 Domain 设置配合 Oauth 来实现指定子域名下所有站点的单点登录：`*.xiaosansiji.com` 是我们的一个泛域名解析，其下有 `a.xiaosansiji.com`、`b.xiaosansiji.com` 等多个站点，我们希望登录某个站点后，就可以默认登录所有 `xxx.xiaosansiji.com` 站点。

具体实现为：用户访问 `a.xiaosansiji.com` 站点时，跳转到单点登录服务器提供的统一登录页面，用户验证通过后重定向跳转回 `a.xiaosansiji.com` 主页面，在这次 HTTP 请求的 Response Header 中向浏览器中存储 Cookie 并设置 Domain 为 `xiaosansiji.com` ：

```javascript
Response Header
HTTP/1.1 302
Server: nginx/1.13.1
Date: Wed, 04 Jul 2018 06:48:43 GMT
Content-Length: 0
Location: http://a.xiaosansiji.com/
Connection: keep-alive
Set-Cookie: csid=D6ACED5C8AB99D5A3DB57594112AB00F;domain=xiaosansiji.com;path=/;HttpOnly
Content-Language: zh-CN
```

这样当我们在同一浏览器中新打开的一个 Tab 中访问 `b.xiaosansiji.com` 站点时，系统提示我们已经登录成功。

注意我们在 `Set-Cookie` 时使用了 `HttpOnly` 标记，这主要是从安全角度禁止 JavaScript 操作该 Cookie，详细内容参见 [《Web 安全》]()章节。

## 跨域 HTTP 访问

![cross-domain-error.png](./cross-domain-error.png)

在日常前端开发工作中，我们遇到的更多是上面这样的报错：我们从 `localhost:8000` 发起了一个 Ajax 请求到 `www.google.com.hk` 站点，然后我们没能拿到想要的数据，反而在 console 中提示了上图的错误。

这其实就是因为同源策略限制，我们的跨域请求失败了。需要注意其实我们的请求是已经发送到目标服务器了，对方接口也正常返回了数据，只是在返回到浏览器时被 block [^1]了（回想本篇开头的同源策略定义，它是**浏览器**的安全策略）。

[^1]: 高版本的 Chrome 和 Firefox 等会在站点采用 HTTPS 的情况下直接阻止请求发出去，这时候后端其实不会接收到该请求。

我们常见的解决方案有三种：

- JSONP ：是把请求伪装成标签去请求。因为标签是浏览器自己发送请求，所以不受同源策略影响啊
- 后端 Proxy：这个也很好理解，我把所有请求都发送到不跨域的代理服务器上，服务器上可是我们说的算，只要经过处理把数据返回给浏览器就好。
- CORS：这是我们今天主要讲。因为解铃还须系铃人，既然是你限制的，那么你总得给我一个解决办法吧。浏览器给出的解决办法就是（CORS）

### JSONP

> JSONP：JSON with Padding

我们在使用 jQuery 等封装的 Ajax 库时经常这样使用 JSONP：

```javascript
$.Ajax({
  type: "get",
  async: false,
  url: "http://demo.com/api/current?user=1",
  dataType: "jsonp",
  jsonp: "callback",//传递给请求处理程序或页面的，用以获得jsonp回调函数名的参数名(一般默认为:callback)
  success: function(json){
    alert(json);
  },
});
```

这让我们产生了一种错觉，仿佛 JSONP 也是 Ajax 的一种，毕竟我们正常发起请求时也只是将 `dataType` 改为 `json` 而已。

但 JSONP 其实跟 Ajax/Fetch 都没什么关系，它用一种比较 hack 的方式实现跨域 HTTP 接口交互：在同源策略章节我们已经说过，在页面中使用 `script` 标签加载 CDN 上的 JavaScript 资源文件是不受同源策略限制的。程序员们就想到如果我们请求的 JavaScript 文件是后端动态生成的，在浏览器端执行了这一段 JavaScript 代码后岂不是就拿到了后端接口的数据！

于是上述代码的底层实现其实是这样的：

浏览器端我们需要增加如下代码发起请求，并声明了请求成功后处理函数

```javascript
var callback = function(data){
    alert(data);
};
var url = "http://demo.com/api/current?user=1";
var script = document.createElement('script'); 
script.setAttribute('src', url); 
document.getElementsByTagName('head')[0].appendChild(script);
```

在服务器端我们需要把正常返回的数据转化为字符串，并用 callback 包裹

```javascript
server.on('request', function(req, res) {
  var params = qs.parse(req.url.split('?')[1]);
  // jsonp返回设置
  res.writeHead(200, { 'Content-Type': 'text/javascript' });
  res.write('callback' + '(' + JSON.stringify(params) + ')');
  res.end();
});
```

这样页面上请求到后端数据后就可以直接调用 `callback` 方法进行后续数据展示处理了。

可以看到 JSONP 的底层实现跟 Ajax 的 XHR 完全不同，只是 jQuery 帮我们封装了这些操作而已。

JSONP 方案可以运行在任何允许执行 JavaScript 脚本的浏览器上，兼容性较好，但也存在很多限制：

- 需要后端同学的支持：正常返回的 JSON 数据需要处理一下，变成浏览器可以执行的脚本
- 仅支持 GET 方法的请求：如代码示例中展示的，我们只能发起 GET 请求，且所有参数只能添加到 url 中
- 对于错误的支持不好：不像 Ajax 和 Fetch 都有较好的错误处理接口，JSONP 失败了只会在控制台输出一句 error 而已

总的来说，我们在日常与后端接口交互中较少采用 JSONP 方案，在投放广告等领域用的比较多。

###后端 Proxy 

这是我们日常开发中最常用的方案，具体来说就是让后端启一个代理服务，来代替直接由浏览器对其他域的接口发起请求。

如，我们将站点部署在 Nginx 上，有两个单独部署的后端接口地址（这在后端微服务应用中很常见）需要访问：

```nginx
server {  
    listen       80;  
    server_name  localhost;  

    #反向代理地址为http://a.demo.com/api的后端接口，需要根据实际情况改为自己的地址
    location ^~ /api {
        proxy_pass http://a.demo.com;
        proxy_set_header        X-Real-IP $remote_addr;
        proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header        Host $http_host;
    }
    location ^~ /bpi {
        rewrite ^/bpi(.+) /api/$1 break;
        proxy_pass http://b.demo.com;
        proxy_set_header        X-Real-IP $remote_addr;
        proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header        Host $http_host;
    }
    ...
}
```

这样我们在浏览器上发起这两类请求时其实都可以请求到这个 Nginx 节点上，由 Nginx 根据 url 路径的不同（api/bpi）来分别请求两个站点的接口。

后端 Proxy 的方案唯一的缺点就是需要后端开发或者使用 Nginx/Apach 等配置代理，只能用于后端能够改造的前提下。

### CORS

> CORS：cross-origin sharing 跨域资源共享

对于 Ajax 和 Fetch 来说，CORS 是现在最可靠的跨域解决方案，它本身已经加入 [W3C 规范](https://www.w3.org/TR/cors/) 。现在各主流浏览器都允许服务器端返回 HTTP 请求时增加相应 header 设置，以声明浏览器端发出的请求有权限做哪些操作。上节说过，同源策略会使得浏览器在服务端返回数据时阻塞该响应，而当我们在 Response header 中增加了相关 CORS 声明后，浏览器就可以放行该响应，让 JavaScript 继续后续的数据处理和展示工作。这相当于让服务器端为我们的请求“背书”，该服务器明确告知浏览器：我允许该站点跨域获取我的接口数据。

在 CORS 中的配置分为“简单请求”和“需预检的请求”两种。

#### 简单请求

满足如下条件，视为“简单请求”：

- 使用下列方法之一：

  - `GET`
  - `HEAD`
  - `POST`

- 未设置下面列出之外的其他 header ：

  - [`Accept`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Accept)

  - [`Accept-Language`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Accept-Language)

  - [`Content-Language`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Content-Language)

  - [`Content-Type`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Content-Type) 只能设置三种类型：

    - `text/plain`
    - `multipart/form-data`
    - `application/x-www-form-urlencoded`

    还有如下几个不太常用 header 设置

  - [`DPR`](http://httpwg.org/http-extensions/client-hints.html)

  - [`Downlink`](http://httpwg.org/http-extensions/client-hints.html#downlink)

  - [`Save-Data`](http://httpwg.org/http-extensions/client-hints.html#save-data)

  - [`Viewport-Width`](http://httpwg.org/http-extensions/client-hints.html#viewport-width)

  - [`Width`](http://httpwg.org/http-extensions/client-hints.html#width)

- 请求中的任意[`XMLHttpRequestUpload`](https://developer.mozilla.org/zh-CN/docs/Web/API/XMLHttpRequestUpload) 对象均没有注册任何事件监听器；[`XMLHttpRequestUpload`](https://developer.mozilla.org/zh-CN/docs/Web/API/XMLHttpRequestUpload) 对象可以使用 [`XMLHttpRequest.upload`](https://developer.mozilla.org/zh-CN/docs/Web/API/XMLHttpRequest/upload) 属性访问。

- 请求中没有使用 [`ReadableStream`](https://developer.mozilla.org/zh-CN/docs/Web/API/ReadableStream) 对象。

满足以上条件的“简单请求”，我们只需要要求服务器端在返回响应时提供 `Access-Control-Allow-Origin` 的 header 设置就可以了，在线例子可以查看[这里](http://arunranga.com/examples/access-control/simpleXSInvocation.html)

```http
HTTP/1.1 200 OK
Date: Thu, 05 Jul 2018 02:18:58 GMT
Server: Apache
Access-Control-Allow-Origin: http://arunranga.com
Keep-Alive: timeout=2, max=100
Connection: Keep-Alive
Transfer-Encoding: chunked
Content-Type: application/xml
```

如上，我们在跨域的另一个服务器返回中设置了 `Access-Control-Allow-Origin: http://arunranga.com` ，即明确告知浏览器允许 `http://arunranga.com` 站点的请求获取本服务器的数据。

当然，有些提供天气/地理查询服务的网站可能会允许任何其他站点获取数据，则会设置 header 为 `Access-Control-Allow-Origin: *` 。

在有一种情况下，不允许设置 `Access-Control-Allow-Origin: *`：跨域请求的接口依赖 header 中的 Cookie 信息来做用户验证时，根据 CORS 规范，请求不会自动携带该域下的 Cookie，对于 Fetch 请求来说，我们需要设置

```javascript
fetch(url, {
  credentials: 'include'  
})
```

以允许携带 Cookie，具体情况可参考 [`Request.credentials`](https://developer.mozilla.org/zh-CN/docs/Web/API/Request/credentials)。同时，我们只能设置`Access-Control-Allow-Origin: http://arunranga.com` ，即只能允许指定域的访问，而不能使用 `*` 允许来自所有域的访问。

#### 需预检的请求

不满足“简单请求”条件的所有请求我们都视为“需预检的请求”，如 `PUT` 请求等。其特殊之处在于：我们不仅需要设置 `Access-Control-Allow-Origin`，还需要设置 `Access-Control-Allow-Methods` 等以允许这些请求。

在简单请求过程中，该跨域的请求与非跨域请求其实是一样的：都从浏览器端发送请求到了目标服务器，在浏览器接收到响应时再判断是否允许跨域处理。

而需预检的请求则不是这样：在发起的正式的 `PUT` 等请求前，浏览器会先使用 [`OPTIONS`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Methods/OPTIONS) 方法发起一个预检请求，以询问服务器是否允许该请求，如果允许才会发送实际请求，可以查看这个在线[例子](http://arunranga.com/examples/access-control/preflightInvocation.html)：

![option-request.png](option-request.png)

在这个例子中，我们发起了一个 `POST` 请求，但因为我们使用了非标准的的 header 设置，所以仍属于需预检的请求。这也是比较常见的做法，在 header 中加入某些我们自己定义头，如标识用户所属的组织等：`x-current-group: 'dev'`，相应的在 CORS 处理中我们需要在增加允许该 header 的设置：`Access-Control-Allow-Headers': 'x-current-group'` 。

当然如果我们每发起一个需预检的请求都要实际上发送两次 HTTP 请求（一个 OPTIONS 预请求，一个正式请求），对于服务器来说增大了访问压力，我们可以在 Response header 中设置 `'Access-Control-Max-Age': 600` 来让验证通过后的10分钟内不再发送预请求。

#### 常见 header 设置

总结以上简单请求和需预检的请求使用场景，常用的 CORS Response header 设置有:

- [`Access-Control-Allow-Origin`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Access-Control-Allow-Origin):  `<origin> | *` 例，`http://arunranga.com`
- [`Access-Control-Expose-Headers`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Access-Control-Expose-Headers) : `<field-name>[, <field-name>]*` 例，`x-current-group`
- [`Access-Control-Allow-Methods`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Access-Control-Allow-Methods) : `<method>[, <method>]*` 例，`PUT, OPTIONS`
- [`Access-Control-Allow-Credentials`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Access-Control-Allow-Credentials) : bool 设置为 `true` 时允许浏览器端携带 Cookie 等 Credentials 访问接口，这个除了服务端要设置外，浏览器请求的中也要设置，Ajax 是 `xhr.withCredentials = true;` Fetch 中是 `credentials: 'include'`
- [`Access-Control-Max-Age`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Access-Control-Max-Age) : <delta-seconds> 注意单位是秒

### 其他 CORS 用法

注意 CORS 其实不仅限于跨域接口的访问，HTML 5 规范中也对 CORS 提供了支持，比如我们在做前端全局异常采集时经常要求引入的 Scripts 做如下处理：

```javascript
<script src="https://example.com/example.js" crossorigin="anonymous"></script>
```

它允许我们可以获取该跨域脚本执行出错时的详细信息，否则我们就只能在监听 window.onerror 时得到这样的提示：`Script error.`

其他标签支持 CORS 的情况，请参看[这里](https://developer.mozilla.org/zh-CN/docs/Web/HTML/CORS_settings_attributes) 。

## 参考链接：

- [浏览器跨域方法与基于Fetch的Web请求最佳实践](https://segmentfault.com/a/1190000006095018)
- [浏览器的同源策略](https://developer.mozilla.org/zh-CN/docs/Web/Security/Same-origin_policy)
- [HTTP cookies](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Cookies)
- [JSONP 教程](http://www.runoob.com/json/json-jsonp.html)
- [HTTP访问控制（CORS）](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Access_control_CORS)



