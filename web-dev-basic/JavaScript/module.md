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

## 常见模块规范

### CommonJS

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

### AMD

> AMD: Asynchronous Module Definition 异步模块定义规范

为了解决浏览器环境下模块加载的问题，在 CommonJS 基础上社区提出了 AMD 规范，对比 CommonJS 它主要提供了如下不同的特性：

1. 使用 `define` 来定义模块：`define(id?: String, dependencies?: String[], factory: Function|Object);`
2. 使用 `require` 来加载模块，与 CommonJS 不同的是，这里 `require` 可以配置加载结束后的事件回调函数: `require(modules: String|[], callback?: Function)`

详情可以参见 [文档](https://github.com/amdjs/amdjs-api/wiki/AMD-(%E4%B8%AD%E6%96%87%E7%89%88)) 。从设计思想上来说，AMD 采取了浏览器优先策略，CommonJS 则采取了服务器优先的策略。

示例：

**定义模块**

```javascript
// 创建一个名为"alpha"的模块，依赖于 require、exports、beta 模块:
define("alpha", ["require", "exports", "beta"], function (require, exports, beta) {
  exports.verb = function() {
    return beta.verb();
    //Or:
    return require("beta").verb();
  }
});
```



`"alpha"` 是可以省略，当未指定模块名时，默认取模块所在文件的名称，如在 alpha.js 中则会被命名为 `alpha` 模块；最后的 `factory` 模块初始化要执行的函数或对象，若为函数需要返回一个对象或函数供外部使用。

**使用模块**

```javascript
require(['foo', 'bar'], function ( foo, bar ) {
  foo.doSomething();
});
```

如上，在成功加载了 `foo` 和 `bar` 两个模块后执行了指定方法。

AMD 规范充分考虑了浏览器端环境，提供了异步加载的机制，这并不是什么黑魔法，AMD 的实现库如 RequireJS 等实际上在加载的过程利用了 `<script>` 标签：

对于未被加载过的模块，创建一个 `<script>` 标签，在 `src` 属性中填入转换为 url 的远端模块地址，设置 `onload`、 `onerror` 等事件，然后将这个标签加入 DOM 树交由浏览器处理。

可以看到，`require` 的 callback 回调正是依赖 `<script>` 上的 `onload` 事件实现的。

### UMD

> Universal Module Definition 通用模块定义

[UMD](https://github.com/umdjs/umd) 是对 AMD 和 CommonJS 规范的一个兼容适配，这意味着以 UMD 封装的模块可以同时应用于浏览器和服务器运行环境。对于很多工具类模块，如时间处理库 [Moment.js](http://momentjs.cn/) 等，无论前端显示处理还是后端时间戳转换都会用到，对于这样的模块我们就需要将其打包为 UMD 模式的文件。

听起来好像很神奇，但实际上我们只需要实现一个 Adapter 逻辑就好了，如下是比较简单的一种实现：

```javascript
((root, factory) => {
  if (typeof define === 'function' && define.amd) {
    //AMD
    define(['jquery'], factory);
  } else if (typeof exports === 'object') {
    //CommonJS
    var $ = requie('jquery');
    module.exports = factory($);
  } else {
    //都不是，浏览器全局定义
    root.testModule = factory(root.jQuery);
  }
})(this, ($) => {
  //do something...  这里是真正的函数体
});
```

你仍然使用 `define` 关键字定义模块，在使用时只需要增加上面这样的判断就可以愉快的在各种环境使用模块了。

### CMD

> Common Module Definition

看到这里你可能已经抓狂了：怎么又来个 CMD ？！！

我们来捋一捋，这里借用 [Sea.js](https://github.com/seajs) 作者玉伯大佬的 [issue](https://github.com/seajs/seajs/issues/588) 说明：

> 大概 09 年 - 10 年期间，[CommonJS](http://wiki.commonjs.org/wiki/CommonJS) 社区大牛云集。CommonJS 原来叫 ServerJS，推出 [Modules/1.0](http://wiki.commonjs.org/wiki/Modules) 规范后，在 Node.js 等环境下取得了很不错的实践。
>
> 09年下半年这帮充满干劲的小伙子们想把 ServerJS 的成功经验进一步推广到浏览器端，于是将社区改名叫 CommonJS，同时激烈争论 Modules 的下一版规范。分歧和冲突由此诞生，逐步形成了三大流派：
>
> 1. **Modules/1.x 流派**。这个观点觉得 1.x 规范已经够用，只要移植到浏览器端就好。要做的是新增 [Modules/Transport](http://wiki.commonjs.org/wiki/Modules/Transport) 规范，即在浏览器上运行前，先通过转换工具将模块转换为符合 Transport 规范的代码。主流代表是服务端的开发人员。现在值得关注的有两个实现：越来越火的 [component](https://github.com/component/component) 和走在前沿的 [es6 module transpiler](https://github.com/square/es6-module-transpiler)。
> 2. **Modules/Async 流派**。这个观点觉得浏览器有自身的特征，不应该直接用 Modules/1.x 规范。这个观点下的典型代表是 [AMD](http://wiki.commonjs.org/wiki/Modules/AsynchronousDefinition) 规范及其实现 [RequireJS](http://requirejs.org/)。这个稍后再细说。
> 3. **Modules/2.0 流派**。这个观点觉得浏览器有自身的特征，不应该直接用 Modules/1.x 规范，但应该尽可能与 Modules/1.x 规范保持一致。这个观点下的典型代表是 [BravoJS](https://code.google.com/p/bravojs/) 和 FlyScript 的作者。BravoJS 作者对 CommonJS 的社区的贡献很大，这份 [Modules/2.0-draft](http://www.page.ca/~wes/CommonJS/modules-2.0-7/) 规范花了很多心思。FlyScript 的作者提出了 [Modules/Wrappings](http://wiki.commonjs.org/wiki/Modules/Wrappings) 规范，这规范是 CMD 规范的前身。...

简单来说 CommonJS、AMD、CMD 都是社区对模块化标准 Modules 的一些实践，因为一些认知的分歧大家逐渐形成了自己的社区。

CMD 规范实现主要是 Sea.js ，国内社区用的多一些，国外社区相对较少一些。

CMD 与 AMD 一样，都是异步加载模块，使用场景都偏向于浏览器端（虽然服务端这两种也都可以用）。其与 AMD 的主要区别是 CMD 更推崇**依赖就近**，而AMD 推崇**依赖前置**：

```javascript
// AMD 默认推荐的是
define(['./a', './b'], function(a, b) { 
  // 依赖必须一开始就写好 
  a.doSomething();
  // 此处略去 100 行 
  b.doSomething();
  ...
})
// CMD
define(function(require, exports, module) {
  var a = require('./a');
  a.doSomething();
  // 此处略去 100 行
  var b = require('./b');
  // 依赖可以就近书写
  b.doSomething();
  ... 
})
```

如上，CMD 规范认为依赖就近，即“用到时再声明引入对应模块“更符合代码书写习惯，这在书写方式上也与 CommonJS 更接近。

以上内容主要来自玉伯在知乎上的回答，我个人对此持保留意见，在那个没有 ES6 module，也没有各种打包工具可以把模块代码编译为浏览器直接运行的 JS 文件的时代，CMD 绽放了它应有风采，但行业在不断进步，在当下这个时间点 CMD 及其主要实现 Sea.js 应用的已经不太多了。

### ES6 module

在经历了上述各种模块规范后，ES6 终于从语言层面上正式规范了 JavaScript 的模块机制，详情可查看阮一峰老师的《[ECMAScript 6 入门](http://es6.ruanyifeng.com/)》相关章节。

简单来说 ES6 module 提供了 import 和 export 关键字来做模块的导入、导出操作，其与 CommonJS、AMD 最大的不同在于：

- `import`命令是编译阶段执行的，在代码运行之前
- CommonJS 模块输出的是一个值的拷贝，ES6 module 输出的是值的引用

长远角度看我们希望能用 ES6 module 替代所有类型的模块机制，但是一方面 CommonJS 还有一些更适应服务端环境的特性，另一方面并不是所有浏览器都支持 ES6 module，所以我们在封装自己的模块时还是倾向于使用 UMD 等兼容规范。

## 工具

上述模块规范在实际项目开发中并不需要我们手动或者使用对应实现工具处理，而是在使用 Webpack、Parcel、Rollup 等工具时调用 Babel 等插件进行自动化处理，详细内容可以参看《工具》相关章节。

## 参考链接

- [javascript中模块的发展历程](http://blog.liucunyang.cn/2017/12/01/module/#more)
- [JavaScript 模块化*七日谈*](http://huangxuan.me/js-module-7day/#/)
- [JavaScript 模块化简述](https://juejin.im/post/58882a42128fe100684ad9de)
- [JavaScript 模块化入门Ⅰ：理解模块](https://zhuanlan.zhihu.com/p/22890374)
- [前端模块化开发那点历史](https://github.com/seajs/seajs/issues/588)
- [AMD 和 CMD 的区别有哪些？玉伯的回答](https://www.zhihu.com/question/20351507)
- [使用 AMD、CommonJS 及 ES Harmony 编写模块化的 JavaScript](http://justineo.github.io/singles/writing-modular-js/)