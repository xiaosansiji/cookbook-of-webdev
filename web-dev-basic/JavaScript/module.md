# JavaScript 模块机制

## 为什么需要模块机制

- **提升可维护性**：“意大利面条”式的杂糅在一起的代码是很难读懂的，经过合理拆分的低耦合、高内聚模块是构建大型应用的基础。
- **独立命名空间** ：建议结合《你不知道的 JavaScript》作用域相关章节看，在模块外不能改变内部未开放的状态。
- **方便复用** ：借助于独立命名空间特性，我们可以放心的复用模块提供的功能，而不需要担心引入位置同名变量覆盖等问题。

在模块机制大规模应用前，我们经常使用 IIFE（立即执行函数表达式）来达到上述目的，如在 jQuery 大行其道的时代我们经常使用 IIFE 来封装我们自己的模块。以下例子来自《你不知道的 JavaScript》函数作用域章节：

```javascript
var a = 2;

(function IIFE( global ) {

    var a = 3;
    console.log( a ); // 3
    console.log( global.a ); // 2

})( window );

console.log( a ); // 2
```

大家可以自行验证如果不用 IIFE 方式处理，仅依靠 JavaScript 的函数作用域[^1]的代码执行结果会怎样。

[^1]: ES6 之前的规范中主要提供了函数作用域，只有 `with`、`try/catch` 这样的不常用用法中才有块级作用域，不过 ES6 中增加了 `let` 、 `const`  关键字来提供块级作用域。

IIFE 只能解决命名空间污染问题，它的缺点除了要手动写这样怪异的代码外，主要是没有提供一个明确的模块加载解决方案。以前我们主要依赖于浏览器中 `<script>` 的加载机制，当一个页面中有多个相互依赖的 JS 模块时，就会遇到以下问题：

- 我们需要手动管理各模块的加载先后顺序：比如众多 jQuery 插件都要求必须先加载 jQuery 本身等要求
- 我们也不能在同一页面使用模块的两个版本：如页面中使用了两个版本的 jQuery，因为 jQuery 使用 `$` 和 `jQuery` 全局对象来暴露接口，同时存在两个版本时我们就不能再直接使用这两个对象来调用接口了[^2]。

[^2]: jQuery 提供了 noConflict 方法来解决这个问题，但仍显得比较怪异。

除了上述问题，我们还需要考虑服务器端 JavaScript 运行环境的问题，毕竟服务器端可没有浏览器环境让你用 `<script>` 标签引入模块，随着 Node.js 的出现，CommonJS 规范应运而生。

## CommonJS

2009 年 Node.js 上线，NPM 包管理 + CommonJS 模块机制带来的简单快捷的模块封装/复用体验是最令人激动的特性之一，这也使得 JS 这门前端语言能够像 Java/C# 等一样构建大型后端服务应用。

CommonJS 规范主要提供了如下特性：

1. 一个单独的文件就是一个模块
2. 使用 `exports` 或 `modul.exports` 关键字来暴露模块中的内容
3. 未暴露的对象都是私有，模块外无法访问
4. 使用 `require` 关键字来加载模块
5. 同步加载模块文件
6. 加载过后的文件会被缓存中，下次加载只需要从内存中读取

可以这样封装和使用模块：

```javascript
// hi-module.js
var str = 'Hi';

function sayHi(name) {
  console.log(str + ', ' + name + '!');
}

module.exports = sayHi;

// main.js
var Hi = require('./hi');
Hi('Jack');     // Hi, Jack!
```

一切看起来很完美了，但是有一个问题：CommonJS 无法在浏览器端使用。

问题出在第 5 条，Node.js 运行在服务器端，所有模块文件都存储在本地，同步去读取当然没有问题，况且还有缓存可以利用。但是在浏览器端，所有资源都来自于远端，需要通过网络传输，同步加载会增大页面响应时间。

## AMD

> AMD: Asynchronous Module Definition 异步模块定义规范

为了解决浏览器环境下模块加载的问题，在 CommonJS 基础上社区提出了 AMD 规范，对比 CommonJS 它主要提供了如下不同的特性：

1. 使用 `define` 来定义模块
2. 使用 `require` 来加载模块，与 CommonJS 不同的是，这里 `require` 支持同时加载多个模块，且可以配置加载结束后的事件回调函数

详情可以参见 [文档](https://github.com/amdjs/amdjs-api/wiki/AMD-(%E4%B8%AD%E6%96%87%E7%89%88)) 。

示例：

```javascript

```



## 参考链接

- [javascript中模块的发展历程](http://blog.liucunyang.cn/2017/12/01/module/#more)
- [JavaScript 模块化简述](https://juejin.im/post/58882a42128fe100684ad9de)
- [JavaScript 模块化入门Ⅰ：理解模块](https://zhuanlan.zhihu.com/p/22890374)