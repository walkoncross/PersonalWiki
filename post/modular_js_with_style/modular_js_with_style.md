## [轉載] 使用 AMD、CommonJS 及 ES Harmony 编写模块化的 JavaScript [Back](./../post.md)

> - Author: [Addy Osmani](http://twitter.com/addyosmani)
- Translate: [Gu YiLing](http://weibo.com/justineo)
- Origin: http://justineo.github.io/singles/writing-modular-js/

### 模块化（解耦应用程序的重要性）

当我们称一个应用程序是**模块化**的的时候，我们通常是指它由一组高度解耦的、存放在不同模块中的独特功能构成。你可能已经知道，[松散耦合](http://arguments.callee.info/2009/05/18/javascript-design-patterns--mediator/)通过尽可能地去除依赖性来让可维护性更加简单易得。当这一点被有效实现的时候，系统中某一部分的变化将如何影响其它部分就会变得显而易见。

然而，与一些更传统的编程语言不同的是，JavaScript 的当前版本（[ECMA-262](http://www.ecma-international.org/publications/standards/Ecma-262.htm) ）并没有为开发者们提供以一种简洁、有条理地的方式来引入模块的方法。规范的一大问题，就是未曾在这方面投入足够多的考量。直到近年来，人们对更为有序组织的 JavaScript 应用的需求变得越来越显著，这一情况才有所改观。

作为代替，当前的开发者们只能被迫降级使用[模块模式](http://www.adequatelygood.com/2010/3/JavaScript-Module-Pattern-In-Depth)或是[对象字面量模式](http://rmurphey.com/blog/2009/10/15/using-objects-to-organize-your-code/)的各种变体。通过很多这样的方法，各模块的脚本被串在一起注入到 DOM 中（译注：感谢 [@lkaihua](https://twitter.com/lkaihua/) 的确认，指的是作为 `script` 标签[注入](https://twitter.com/addyosmani/status/278514027892703232)到 DOM 中），其命名空间是由单一的全局对象来描述的。你的整个体系架构在这种模式下，仍然有可能发生命名冲突。想要简洁地管理依赖关系，不通过一些手工处理或借助第三方库往往是不可能的。

尽管这些问题的原生解决方案在 [ES Harmony](http://wiki.ecmascript.org/doku.php?id=harmony:modules) 中才会被引入，但好消息是，编写模块化的 JavaScript 目前已经变得极为简单，甚至今天就可以开始动手。

在本文中，我们将会考察三种编写模块化 JavaScript 的格式：**AMD**、**CommonJS** 以及 JavaScript 下一个版本——**Harmony** 中的提案。

### 序言（关于脚本加载器的注解）

要讨论 AMD 和 CommonJS 模块，我们必然会谈及一个显而易见的话题——脚本加载器。目前，脚本加载是为了让我们能在现今的各种应用中都能使用模块化的 JavaScript 这个目标而服务的。为此，使用一个兼容的脚本加载器不得不成为必须。为使大家能更好地读懂本文，我推荐大家先**大致**了解一下主流的脚本加载工具的工作原理，这样在这儿对模块格式所作的讨论也就更有意义了。

用来处理基于 AMD 和 CJS 格式进行模块加载的加载器为数众多，但我个人最偏爱的还是 [RequireJS](http://requirejs.org/) 和 [curl.js](https://github.com/unscriptable/curl)。这些工具完整的入门教程已经在本文的讨论范围之外，但我还是推荐大家读一读 John Hann 关于 [curl.js](http://unscriptable.com/index.php/2011/03/30/curl-js-yet-another-amd-loader/) 的文章以及 James Burke 的 [RequireJS](http://requirejs.org/docs/api.html) 的 API 文档来了解更多相关内容。

从生产的角度，在处理这样的模块时，使用优化工具（例如 RequireJS Optimizer，即 RequireJS 优化器）来拼合脚本来进行部署是值得推荐的。有趣的是，只要有了 [Almond](https://github.com/jrburke/almond) AMD shim，RequireJS 甚至无须加入到部署完的站点之中，你当做脚本加载器的那个东西可以被轻松地从开发过程中移除出去。

尽管如此，James Burke 还是很可能会说，在页面加载完毕以后还能动态加载脚本的使用场景依然存在，并且 RequireJS 在这些场景下也有用武之地。带着上面的这些注解，让我们开始吧。

### AMD（一种在浏览器中编写模块化 JavaScript 的格式）

AMD（异步模块定义，Asynchronous Module Definition）格式总体的目标是为现在的开发者提供一个可用的模块化 JavaScript 的解决方案。它诞生于 Dojo 在使用 XHR+eval 时的实践经验，其支持者希望未来的解决方案都可以免受由于过去方案的缺陷所带来的麻烦。

AMD 模块格式本身是一个关于如何定义模块的提案，在这种定义下模块和依赖项都能够[异步](http://dictionary.reference.com/browse/asynchronous)地进行加载。它有很多独特的优势，包括天生的异步及高度灵活等特性，这些特性能够解除常见的代码与模块标识间的那种紧密耦合。它被许多开发者所青睐，可以认为它是迈向 ES Harmony 中提出的[模块系统](http://wiki.ecmascript.org/doku.php?id=harmony:modules)的一块可靠基石。

AMD 最初是一份 CommonJS 列表中模块格式的规范草案，但因为没能达成完整的共识，格式的后续开发就被移到了 amdjs 讨论组中。

目前它已经被包括 Dojo (1.7)、MooTools (2.0)、Firebug (1.8) 甚至 jQuery (1.7) 这样的项目所接纳。尽管我们时不时还是会遇到 CommonJS AMD 格式 这个说法，但由于不是所有 CJS 列表上的参与者都愿意支持它，我们最好还是称它为 AMD 或是异步模块支持。