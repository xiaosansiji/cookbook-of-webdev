> **特别声明：**此篇文章由[David](http://www.codingserf.com/)根据《[Block, Element, Modifier](http://bem.info/method/definitions/)》进行翻译，整个译文带有我们自己的理解与思想，如果译得不好或不对之处还请同行朋友指点。如需转载此译文，需注明英文出处：<http://bem.info/method/definitions/>以及作者相关信息
>
> ——作者：[BEM官网](http://bem.info/)
>
> ——译者：[David](http://www.codingserf.com/)

## 什么是BEM？

BEM代表块（Block），元素（Element），修饰符（Modifier）。这些术语的含意将在本文进一步阐述。

编程方法论中一个最常见的例子就是面向对象编程（OOP）。这一编程范例出现在许多语言中。在某种程度上，BEM和OOP是相似的。它是一种用代码和一系列模式来描述现实情况的方法，它只考虑程序实体而无所谓使用什么编程语言。

我们使用BEM的原则创建了一个前端开发技巧和工具的集合，这样我们就能快速构建一个网站，并且保证他们长久的可维护性。

## 统一数据域

想象一个如下图所示的普通网站。

![Sass调试](http://www.w3cplus.com/sites/default/files/styles/print_image/public/blogs/2013/Definitions-BEM-1.jpg)

就这样的一个网站来说，指出它是由哪些“块”构成的，对开发是很有帮助的。

例如，在上图中有`Head`，`Main Layout`和`Foot`块。`Head`由`Logo`，`Search`，`Auth`块和`Menu`组成。`MainLayout`包括一个`Page Title`和一个`Text`块。

![Sass调试](http://www.w3cplus.com/sites/default/files/styles/print_image/public/blogs/2013/Definitions-BEM-2.jpg)

对于团队沟通来说给页面的每一部分起个名字是很有用的。

这样的话一个项目经理就可以这么说：

- 让`Head`再大点;
- 创建一个`Head`中不带`Search`的页面。

一个HTML开发人员可以对一个JavaScript开发人员说：

- 给`Auth`块来点动画，等等

现在让我们仔细了解一下什么是BEM:

## 块（Block）

一个块是一个独立的实体，就像应用的一块“积木”。一个块既可以是简单的也可以是复合的（包含其他块）。

例如搜索表单块：

![BEM的定义](http://www.w3cplus.com/sites/default/files/styles/print_image/public/blogs/2013/Definitions-BEM-3.jpg)

## 元素（Element）

一个元素是块的一部分，具有某种功能。元素是依赖上下文的：它们只有处于他们应该属于的块的上下文中时才是有意义的。

例如一个输入域和一个按钮是Search块的中的元素。

![BEM的定义](http://www.w3cplus.com/sites/default/files/styles/print_image/public/blogs/2013/Definitions-BEM-4.jpg)

## 描述页面和模板的意义

块和元素构成了页面的内容。它们不仅仅是被呈现在一个页面上，它们的排列顺序也同样重要。

块（或元素）彼此之间可能遵循着某种顺序。

例如，电商网站上的一个商品列表：

![BEM的定义](http://www.w3cplus.com/sites/default/files/styles/print_image/public/blogs/2013/Definitions-BEM-5.jpg)

……或者菜单项：

![BEM的定义](http://www.w3cplus.com/sites/default/files/styles/print_image/public/blogs/2013/Definitions-BEM-6.jpg)

块里也有可能再嵌套块。

例如，一个Head块会包含其他块：

![BEM的定义](http://www.w3cplus.com/sites/default/files/styles/print_image/public/blogs/2013/Definitions-BEM-7.jpg)

除了我们自己创建的一些块之外 ，我们还需要一种途径来描述页面上那些纯文本的布局。为此，每一个块和元素都应该有一个可以识别的关键字。

用来标识一个具体块的关键字其实就是这个块的名字（`block name`）。

例如，`menu`可以作为`Menu`块的关键字，`head`可以作为`Head`块的关键字。

用来标识一个元素的关键字也是这个元素的名字。

例如，菜单中的每个菜单项就是`menu`块的`item`元素。

一个项目中的块名必须是唯一的，明确指出它所描述的是哪个块。相同块的实例可以有相同的名字。在这种情况下一个块在一个页面上可以出现2（3，4，……）次。

一个块范围内的一种元素的名字也必须是唯一的。一种元素可以重复出现多次。

例如，菜单项：

![BEM的定义](http://www.w3cplus.com/sites/default/files/styles/print_image/public/blogs/2013/Definitions-BEM-8.jpg)

关键字应该按一定的顺序摆放。任何支持嵌套的数据格式（XML，JSON）都可以:

```
<b:page>
  <b:head>
    <b:menu>
      …
    </b:menu>
    <e:column>
      <b:logo/>
    </e:column>
    <e:column>
      <b:search>
        <e:input/>
        <e:button>Search</e:button>
      </b:search>
    </e:column>
    <e:column>
      <b:auth>
        …
      </b:auth>
    <e:column>
  </b:head>
</b:page>

```

在这个例子中，`b`和`e`两个命名空间用来区分块和元素两种节点。

同样可以用JSON格式来表示：

```
{
  block: 'page',
  content: {
    block: 'head',
    content: [
      { block: 'menu', content: … },
      {
        elem: 'column',
        content: { block: 'logo' }
      },
      {
        elem: 'column',
        content: [
          {
            block: 'search',
            content: [
              { elem: 'input' },
              { elem: 'button', content: 'Search' }
            ]
          }
        ]
      },
      {
        elem: 'column',
        content: {
          block: 'auth', content: …
        }
      }
    ]
  }
}

```

上面的例子展示了一个块和元素相互嵌套的对象模型。这个结构中也可以包含任意多的自定义数据字段。

我们把这种结构叫BEM树（和DOM树类似）。

通过把模板转换（使用XSL或是JavaScript）应用到BEM树上生成最终的浏览器标签。

如果开发人员需要把一个块移动到页面的其他地方，他只需要改变一下BEM树就好了。模板会生成最终的视图。

你可以使用任何格式来描述BEM树，也可以使用任何模板引擎。

我们把JSON作为页面的描述格式。然后通过一个基于JS的模板引擎BEMHTML来把它转换为HTML。

## 块的独立性

随着一个项目的发展，我们常常会在页面中添加，删除，或者是移动一些块。例如，你可能想要互换`Logo`和`Auth`块，或者把`Menu`放到`Search`块下面。

![BEM的定义](http://www.w3cplus.com/sites/default/files/styles/print_image/public/blogs/2013/Definitions-BEM-9.jpg)

为了让这个过程更加简化，块必须是独立的。

一个独立的块可以放置在页面的任意位置 ，包括嵌套在其他块里。

## 独立的CSS

从CSS的角度来看：

- 一个块（或者一个元素）必须有一个唯一的“名字”（一个CSS类）这样才能被CSS规则所作用。
- HTML元素不能用作CSS选择器（如`.menu td`）因为这样的选择器并非是完全上下文无关的。
- 避免使用级联（cascading）选择器（译注：如`.menu .item`）。

## 为独立的CSS类命名

下面是一种可能的CSS类命名方案：

一个块的CSS类名就是这个块的名字（block name）。

```
<ul class="menu">
...
</ul>

```

一个元素的CSS类名是一个块名和一个元素名的组合，它们中间用一些符号隔开。

```
<ul class="menu">
    <li class="menu__item">…</li>
    <li class="menu__item">…</li>
</ul>

```

在一个元素的CSS类名中包含一个块名是必要的，这样可以让级联最小化。

我们在长名称中使用连字符分隔单词（例如，`block-name`），使用两个下划线来分隔块名和元素名（`block-name__element-name`）。

例如：

- `block-name--element-name`
- `blockName-elementName`

## 独立的模板

对于模板引擎来说，块的独立性意味着：

- 输入的数据要描述清楚块和元素:块（或元素）必须有个唯一的“名字”，在模板中告诉我们诸如“Menu应该放到这里”这样的事情。
- 块可以出现在BEM树的任意地方。

## 独立块模板

当模板引擎在某个模板中遇到一个块的时候可以准确地把这个块转换成HTML。因此每个块都应该有自己的模板。

例如，下面就是一个XSL模板：

```
<xsl:template match="b:menu">
  <ul class="menu">
    <xsl:apply-templates/>
  </ul>
</xsl:template>

<xsl:template match="b:menu/e:item">
  <li class="menu__item">
    <xsl:apply-templates/>
  </li>
<xsl:template>

```

我们的产品正在逐渐弃用[XSLT](https://github.com/veged/xjst)，我们用我们自己开发的基于JavaScript的模板引擎XJST来解析它。这个模板引擎吸收了所有XSLT的优点（我们是声明性编程的粉丝），并且在服务端和客户端都可以用JavaScript来实现。

现在我们用一种叫做[BEMHTML](http://clubs.ya.ru/bem/replies.xml?item_no=992)的特定领域语言来编写我们的模板，它是基于XJST的。BEMHTML的主要思想在Ya.Ru（在俄罗斯）的BEM俱乐部里提出。

## 块的重复

一个网站的第二个`Menu`块可能出现在`Foot`块里。或者，一个`Text`会变成两个，中间被一个广告分开。

即使一个块被开发成单独的单元，那么相同的块也可能在任何时候出现在这个页面上。

在CSS相关的术语中，这意味着：

- 一定不能使用ID选择器:只有类选择器才能满足我们非唯一性的需求

在JavaScript里意味着：

- 可以准确地检测到具有相同行为的块，因为它们有相同的CSS类名:使用CSS类选择器，可以拣选出所有相同名字的块，方便给它们定义动态行为

## 块修饰符

我们经常需要创建一个和已存在的块非常相似的块，只是外观或行为有些许改变。

比如说我们有一个这样的任务：

给Footer添加另外一个布局不一样的Menu。

![BEM的定义](http://www.w3cplus.com/sites/default/files/styles/print_image/public/blogs/2013/Definitions-BEM-10.jpg)

为了避免开发一个和现有的块只稍微有点不同的另一个块，我们引入修饰符（modifier）的概念。

修饰符作为一个块或是一个元素的一种属性，代表这个块或这个元素在外观或是行为上的改变。

一个修饰符有一个名字和一个值。多个修饰符可以同时使用。

例如：一个用来指定背景颜色的块修饰符

![BEM的定义](http://www.w3cplus.com/sites/default/files/styles/print_image/public/blogs/2013/Definitions-BEM-11.jpg)

例如：一个改变“当前”选项的元素修饰符

![BEM的定义](http://www.w3cplus.com/sites/default/files/styles/print_image/public/blogs/2013/Definitions-BEM-12.jpg)

## 从输入数据的角度来看修饰符

在BEM树里，修饰符是用来描述一个块或者是一个元素实体的属性的。

例如，它们可以作为XML的属性节点：

```
<b:menu m:size="big" m:type="buttons">
  …
</b:menu>

```

用JSON也可以：

```
{
  block: 'menu',
  mods: [
   { size: 'big' },
   { type: 'buttons' }
  ]
}

```

## 从HTML/CSS的角度来看修饰符

对于一个块或者是一个元素来说修饰符是一个附加的CSS类。

```
<ul class="menu menu_size_big menu_type_buttons">
…
</ul>

```

```
.menu_size_big {
  // CSS code to specify height
}
.menu_type_buttons .menu__item {
  // CSS code to change item's look
}

```

我们用一个下划线来分隔块名（或元素名）和修饰符名，再用另一个下划线来分隔修饰符名和它对应的值。（译注：此处原文是俄文，大致就是这个意思）。

## 元素修饰符

元素修饰符以相同的方式实现。

当手写CSS代码的时候，为了编程的可访问性，始终使用分隔符非常重要。

例如，当前菜单项就可能被一个修饰符标记：

```html
<b:menu>
  <e:item>Index<e:item>
  <e:item m:state="current">Products</e:item>
  <e:item>Contact<e:item>
</b:menu>

```

```css
{
  block: 'menu',
  content: [
    { elem: 'item', content: 'Index' },
    {
      elem: 'item',
      mods: { 'state' : 'current' },
      content: 'Products'
    },
    { elem: 'item', content: 'Contact' }
  ]
}

```

```css
.menu__item_state_current
{
  font-weight: bold;
}

```

上面的结构在HTML中代表如下：

```html
<ul class="menu">
    <li class="menu__item">Index</li>
    <li class="menu__item menu__item_state_current">Products</li>
    <li class="menu__item">Contact</li>
</ul>

```

或使菜单类独立于它布局的实现细节之外：

```html
<div class="menu">
    <ul class="menu__layout">
        <li class="menu__layout-unit">
            <div class="menu__item">Index</div>
        </li>
        <li class="menu__layout-unit">
            <div class="menu__item menu__item_state_current">Products</div>
        </li>
        <li class="menu__layout-unit">
            <div class="menu__item">Contact</div>
        </li>
    </ul>
</div>


<div class="menu">
    <table class="menu__layout">
        <tr>
            <td class="menu__layout-unit">
                <div class="menu__item">Index</div>
            </td>
            <td class="menu__layout-unit">
                <div class="menu__item menu__item_state_current">Products</div>
            </td>
            <td class="menu__layout-unit">
                <div class="menu__item">Contact</div>
            </td>
        </tr>
    </table>
</div>

```

## 主题抽象

当许多人开发同一个项目他们应该在数据域上保持一致，在给块和元素命名的时候用上它。例如，一个标签云的块始终命名为tags。其中的每个元素就是一个tag。这种约定贯穿所有的语言：CSS，JavaScript，XSL等等。

从开发处理的角度来看：所有参与都基于同样的约定 。

从CSS角度开看：• 块和元素的CSS可以写成一种伪代码，然后根据命名约定编译成CSS。

```css
.menu {
    __layout {
      display: inline;
    }
    __layout-item {
      display: inline-block;
      …
    }
    __item {
      _state_current {
        font-weight: bold;
      }
    }
  }

```

相应JavaScript：可以使用专门的辅助库来替代使用类名直接选择DOM元素:

```javascript
$('menu__item').click( … );
$('menu__item').addClass('menu__item_state_current');
$('menu').toggle('menu_size_big').toggle('menu_size_small');

```

块和元素的CSS类命名约定可能在一段时间内发生改变。如果命名约定改变了，只需要修改那些要访问块和元素，以及可能处理它们的修饰符的方法（function）。

```
block('menu').elem('item').click( … );
block('menu').elem('item').setMod('state', 'current');
block('menu').toggleMod('size', 'big', 'small');

```

上面的代码是抽象的，现实中我们使用bem-bl块中的i-bem块的JavaScript核心和[库](http://bem.github.com/bem-bl/sets/common-desktop/i-bem/i-bem.en.html)。

## 块的一致性

一个网站有一个带有某种动态行为的Button块。

![BEM的定义](http://www.w3cplus.com/sites/default/files/styles/print_image/public/blogs/2013/Definitions-BEM-13.jpg)

当鼠标滑过这样的块，它的外观就会改变。

![BEM的定义](http://www.w3cplus.com/sites/default/files/styles/print_image/public/blogs/2013/Definitions-BEM-14.jpg)

经理可能会要求在别的页面也使用相同的按钮。

一个块光有CSS实现还是不够的。复用一个块也意味着复用它的行为，也就是它所绑定的JavaScript。

所以一个块必须“知道”关于它自己的一切。为了实现一个块，要用我们所用到的所有技术来描述清楚它的外观和行为，我们把这叫多语言机制。

多语言描述一个块的时候涉及实现这个块的外观和功能的所有编程语言（技术）。

想让一个块像一个UI元素一样展现在页面上，我们需要用下面这些技术来实现它：

- 模板（XSL，TT2，JavaScript等等），把块的声明转换成HTML代码
- CSS，描述块的外观
- JavaScript，如果块有动态行为的话
- 图片
- 文档

构成一个块的每样东西都可以作为一种块技术。

## 真实案例

[Yandex](http://company.yandex.ru/)是一个大公司（在俄罗斯），它使用BEM方法论来开发它的服务。

BEM方法论并不要求你使用某种框架。你也不必在你用到的所有页面技术中都使用BEM（但是那会是最有效的）。

[Yandex的所有服务](http://www.yandex.ru/all)都在它们页面的CSS和JavaScript代码以及XSL模板中用到了BEM。例如：

- [Yandex.Maps](http://maps.yandex.ru/?text=%D0%A0%D0%BE%D1%81%D1%81%D0%B8%D1%8F%2C%20%D0%9C%D0%BE%D1%81%D0%BA%D0%B2%D0%B0&sll=37.609218%2C55.753559&ll=37.609218%2C55.753563&spn=2.570801%2C0.884460&z=9&l=map)
- [Yandex.Images](http://images.yandex.ru/yandsearch?text=Yandex+office&rpt=image):提供在因特网上搜索图片的服务。
- [Yandex.Video](http://video.yandex.ru/#search?text=yac%202011):同时提供托管和搜索图片的服务。
- [Yandex.Auto](http://auto.yandex.ru/)
- [Turkish Yandex](http://www.yandex.com.tr/)

一此服务并没有使用XSL模板而是使用我们最新的模板技术来构建他们的页面。就是我们前面提到的BEMHTML模板引擎。以下就是这些服务：

- [Yandex Search](http://yandex.ru/yandsearch?text=BEM+methodology+front-end&lr=213)或者[Yandex Search in English](http://yandex.com/yandsearch?text=%22What+is+BEM%3F%22+front-end&lr=213)
- [Mobile Apps Search](http://apps.yandex.ru/):这个网站是呈现在智能手机上的。

也有一些其他公司在使用BEM。

比如，mail.us就在他们的部分服务中用到了BEM。他们页面上的一些块的CSS代码就是基于BEM的。他们也有自己的C++模板引擎，并根据BEM方法论来编写块模板。

更多的案例：

- [Rambler.News](http://beta.news.rambler.ru/)
- [HeadHunter](http://hh.ru/)
- [TNK Racing Team](http://futurecolors.ru/tnkracing/)

你或许也对那些使用bem-bl块库的网站感兴趣：

- [基于BEM的web表单JZ验证](http://form.dev.eot.su/)
- [Mikhail Troshev vCard](http://mishanga.pro/):源代码托管在[GitHub](https://github.com/mishanga/bem-vcard)。

## 了解更多

- [定义](http://bem.info/method/definitions/)
- [文件系统组织](http://bem.info/method/filesystem/)
- [历史](http://bem.info/method/history/)

**译者手语：**整个翻译依照原文线路进行，并在翻译过程略加了个人对技术的理解。如果翻译有不对之处，还烦请同行朋友指点。谢谢！

> #### 关于David
>
> 2009年开始接触前端开发，2011年组建创业团队——[[五维互动](http://www.fi5e-d.com/)]，2012年团队被“收编”并更名[创影互动]，遂只身来上海发展，现在就职于[FlipScript](http://www.flipscript.com.cn/)。欢迎交流共勉：[腾讯微博](http://t.qq.com/lybluesky0110)、[个人博客](http://www.codingserf.com/)。

如需转载烦请注明出处：

英文原文：<http://bem.info/method/definitions/>

中文译文：<http://www.w3cplus.com/css/bem-definitions.html>

著作权归作者所有。

商业转载请联系作者获得授权,非商业转载请注明出处。

原文: 

https://www.w3cplus.com/css/bem-definitions.html

