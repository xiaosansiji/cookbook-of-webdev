# React Virtual DOM

## 是什么

###HTML DOM

HTML DOM 定义了访问和操作 HTML 文档的标准方法。

DOM 提供了对 HTML 文档的结构化（树）表述，JavaScript 可以通过 DOM 访问和操作页面元素。



多说一句，DOM 并不是只能被 JavaScript 调用。

###React Virtual DOM

React Virtual DOM 与 HTML DOM 有什么区别？



###Vue Virtual DOM

```javascript
var child1 = React.createElement('li', null, 'First Text Content');
var child2 = React.createElement('li', null, 'Second Text Content');
var root = React.createElement('ul', { className: 'my-list' }, child1, child2);
```

```javascript
<div ng-if="person != null">
    Welcome back, <b>{{person.firstName}} {{person.lastName}}</b>!
</div>
<div ng-if="person == null">
    Please log in.
</div>
```

模板引擎 VS JSX：各种指令 => JavaScript

## 为什么

关于 Virtual DOM 快不快的讨论：[网上都说操作真实 DOM 慢，但测试结果却比 React 更快，为什么？](https://www.zhihu.com/question/31809713/answer/53544875)

![webkit-painting](/Users/sunzhe/code/cookbook-of-webdev/React/webkit-painting.png)

### DOM Diff 算法

为了解答 React Virtual DOM 为什么快及性能调优方案，我们有必要了解渲染过程背后的 Diff 算法

## show me the code

源码解读

## 性能调优

## 注意事项

[React合成事件和DOM原生事件混用须知](https://juejin.im/post/59db6e7af265da431f4a02ef)

class => className

## Diff

[深入浅出React（四）：虚拟DOM Diff算法解析](http://www.infoq.com/cn/articles/react-dom-diff)

