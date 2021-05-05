> [原文](http://blog.wolksoftware.com/the-current-state-of-dependency-inversion-in-javascript)


# JavaScript 中依赖反转当前的状态

**了解 JavaScript 依赖反转的过去，现在和未来。**

在过去的一年半，我一直阅读大量关于依赖反转，并查阅大量 JavaScript 开源 IoC 容器的源码。同时，我也致力于 [InversifyJS](http://blog.wolksoftware.com/introducing-inversify-2)（一个 JavaScript 应用威力强大的 IoC 容器，由 TypeScript 驱动）。

我花了大量的时间去思考 JavaScript 中的 DI，还和很多开发者讨论，我发现这个话题有很大的争议。我编写这个文章是为了分享我学习到的。

### 揭穿 JavaScript IoC 容器谎言（myths）

在我开始详细说明 JavaScript 中的依赖反转的过去，现在和将来的之前，我将会尝试揭穿一些谎言。

![](https://svbtleusercontent.com/hzgdooytzzbpyg.png)

### 谎言1: JavaScript 中没有 IoC 的位置

当我谈论 JavaScript 应用中关于解耦和模块化的时候，很多开发者辩解道 IoC 容器在 JavaScript 中没有容身之处，因为他是一个非常动态的编程语言。

JavaScript 是一个多范式编程语言，他有一些元素来自函数式编程，一些元素来自面向对象编程，但是 JavaScript 的优点是他如此的动态，你可以扩展它去迎合你最喜欢的风格。

ES3 和 ES5 中，面向对象元素不像在 ES6 中一样容易看见。同时，函数式编程开始成为一个主流话题，我们将在未来渐渐看到越来越多函数式编程框架和库，比如[ramdajs](http://ramdajs.com/)

一些人将会跟随函数式的路，可能 IoC 将不会成为必须。

> ["如果你将 SOLID 原则坚持到底，你会得到一些让函数式编程看起来很有吸引力的东西"](http://blog.ploeh.dk/2014/03/10/solid-the-next-step-is-functional/) - Mark Seemann

然而，其他人将会遵循面向对象方向，IoC 容器将会成为复杂度日渐增长的 JavaScript 应用一个绝对必须的结果。

> 注意： 术语复杂度在这个上下文用于描述多个实体间的相互作用。如果多个实体和他们的交互增加，我们将会到达一个点，那就是不太可能知道和理解他们所有。

当使用 C# 和 Java 编码的时候，我们中的大部分永远不会说什么类似“我不需要遵循[solid 原则](https://en.wikipedia.org/wiki/SOLID_(object-oriented_design))”。那为什么你会在 JavaScript 中选择使用 OO 编程范式的时候说相同的话呢？

我们当然也有第三类框架和库，将会从函数式编程和 OO 编程接受一些思想，并混合他们。在这种架构中，一个 IoC 容器是否必须是不清晰的，我猜测这取决于它遵循函数式风格比较很多还是 OO 风格比较多。

### 谎言2: 我们不需要 IoC 容器，我们已经有模块加载器了！

另一种常见错误概念是 JavaScript 中我们不需要 IoC 容器，因为我们有模块加载器。

我们对于模块都非常开心。我们 JavaScript 代码在他们到来之前简直是噩梦，现在他们可用了，我们可以享受某些程度的解耦了。

如果你的项目不是很大，模块足够管理你的应用的复杂度（实体和交互的数量），但是模块不足够去遵循依赖倒转原则：

> [“依赖抽象。不要依赖实现。”](https://en.wikipedia.org/wiki/Dependency_inversion_principle)

当你引入一个模块时，你引入了一个实现，这个实现是你将要使用的，你没有引入这个模块的抽象。考虑下面的代码片段：
![](https://svbtleusercontent.com/ppyvblr4vv6r0w.png)

`DataService`依赖`Http`类。我们使用可模块，但是我们硬编码了对`Http`类的依赖。如果`Http`类的路径或者名称改变，我们需要更新所有依赖它的模块。这意味着我们的代码紧密耦合，不可维护，测试或者重用。

在过去，我们在相同的文件声明这些实体，或者使用多个文件和`<script>`标签以正确的顺序加载文件。我们需要避免污染全局空间，我们开始使用命名空间和闭包去解决这个问题。模块帮助我们去解决这些问题，但是我们不能认为，因为两个实体定义在不同文件，他们就不紧密耦合。

### 谎言3: 依赖倒转 === 注入依赖

我们已经提到，我么不能认为两个实体定义在不同文件，他们就没有紧密耦合。同样的方式，我们不应该认为我们注入依赖，依赖就不是紧密耦合。

考虑下面来自一个 Angular 2.0 例子的代码片段： 

![](https://svbtleusercontent.com/8k0pjfhiyupoq.png)

这一次，我们从`DataService`注入了依赖，通过它的构造器。这将有助于编写单元测试，因为我们可以简单的注入一个`Http`类到`DataService`。

然而，我们依旧引入了`Http`声明的模块。我们实现了依赖注入，但是我们没有达到依赖倒置。如果`Http`类改变了它的名字或者定位，我们将需要更新所有依赖它的模块。

David Heinemeier Hansson（ruby on Rails 的作者）认为[它不需要依赖注入](http://david.heinemeierhansson.com/2012/dependency-injection-is-not-a-virtue.html)，因为 Ruby 是一个动态编程语言。他解释说，当他但愿测试一个类，与其通过它的构造器注入一个类依赖，他可以直接在运行态替换真正的实现。我不认为这是一个非常好的实践，但是对于测试，这是没毛病的。但是对于可维护性和可重用来说，这是完全错误的。假设你在其他应用或者其他模块引入一个类。你需要在运行态替换类依赖，你的代码将会变得难以维护和阅读。

> 注意：前面是一个 TypeScript 例子。TypeScript 能够知道被注入的服务，因为我们已经指出了它的类型。如果我们使用 ES5 替代，我们需要手动指出被注入的类型，使用类似下面的方式：
```ts
DataService.parameters = [new ng.core.Inject(Http)];
```
> 这意味着甚至使用 ES5，我们依旧需要包含一个对 `HTTP` 模块的引用。

注入一个依赖不是最重要的东西。最重要的是依赖一个抽象。

![](https://svbtleusercontent.com/jsvtlhyzljilyg_retina.png)

只有这么做，我们才能移除对`Http`模块的硬索引，在这个场景，如果`Http`模块改变它的名字或者定位，所有依赖它的模块都保持原样，因为他们感知不到这个。

依赖一个抽象允许我们达到真正的解耦，但美好的是这解耦不仅仅是依赖和依赖。我们也可以解耦一些[横切关注点](https://en.wikipedia.org/wiki/Cross-cutting_concern)，比如缓存或者日志。使用[拦截](https://en.wikipedia.org/wiki/Cross-cutting_concern)。

> 拦截是一个高级编程技术，允许你去拦截一个对象的调用，这样你可以在调用前后添加一些额外的逻辑。这个拦截过程应该是抽象和透明的，这样，调用对象和目标对象可以观察到拦截过程。

总之，依赖反转不仅仅是注入依赖。

### JavaScript 中 SOLID 的旅途

现在我们知道 JavaScript 中的依赖倒置是可能的，也是必要的，我们将检验我们离编写真正解耦的 JavaScript 应用还有多远。

> SOLID（单一职责，开闭，李式代换，接口隔离和依赖倒置）是一个助记符缩写，表示面向对象编程的五个基本原则，设计它的目的是让它更可能让一个程序员创建一个更容易维护和扩展的系统。

> SOLID (single responsibility, open-closed, Liskov substitution, interface segregation and dependency inversion) is a mnemonic acronym that stands for five basic principles of object-oriented programming and design that intend to make it more likely that a programmer will create a system that is easy to maintain and extend over time.(翻译不能)

#### 以前：模块加载器

正如我们已经说过的，模块加载器是迈向 JavaScript 真正解耦的第一步，但也只是这样：仅仅是第一步。

#### 近期：angular 1.x DI 方式

我们可以说 Angular 1.x 是第一个包含依赖注入的 JavaScript 框架，它的 IoC（`$injector`）是他的其中一个价值点。

Angular 1.x 允许我们用两个不同方式去声明模块的依赖：

![](https://svbtleusercontent.com/v1zsads0ksnmog_retina.png)

这两种方式，我们使用字符串字面量引用注入的依赖。我们可以认为字符串字面量就是依赖的抽象。这意味着 Angular 1.x 允许我们去依赖抽象，并使用真正的解耦去声明模块。

不幸的是，不是任何东西都是好消息，正如 Pascal Precht 描述在[http://blog.thoughtram.io/](http://blog.thoughtram.io/) 的，Angular 1.x DI 实现有一些问题：

- 内部缓存 - 依赖作为单例服务。无论何时，我们请求一个服务，在每一个应用声明周期，它只创建一次。创建工厂机器非常麻烦。

- 命名空间冲突 - 一个应用只能有一个“类型”词素。如果我们有一个 car 服务，有一个第三方扩展也引入一个有相同名字的服务，就会出现问题。

- 内置到框架 - Angular 1 的 DI 被内置进框架。无法让我们使用它解耦为一个单独的系统。

#### 现在：Aurelia DI 实现

一个包含依赖注入作为特性的现代框架是 Aurelia。Aurelia 为 JavaScript 提出了以下的提案：

![](https://svbtleusercontent.com/up7oowq2cpsuxg.png)

下面是 TypeScript：

![](https://svbtleusercontent.com/3glra41sco57da_retina.png)

正如我们已经学到的，我们不应该认为因为我们注入依赖，依赖和依赖没有紧密耦合。在前面两个代码例子中，存在对`HttpClient`的硬编码引用：

```ts
import {HttpClient} from "aurelia-fetch-client";

```

然而，它[看起来](https://github.com/aurelia/dependency-injection/blob/master/src/container.js)可以使用字符串字面量解决这个问题：

![](https://svbtleusercontent.com/fcbtdunt79sca.png)

在依赖一个抽象之后，我们需要链接抽象（字符串字面量/接口）到一个具体实现（类）。我们可以在渲染过程通过 [Aurelia IoC 容器](https://github.com/aurelia/dependency-injection) 的`registerTransient`方法做到这个。

![](https://svbtleusercontent.com/f0uiqnmfwvoknq.png)

这意味着 IoC 容器知道`HttpClient`类，但是`User`类只知道它的抽象。另一个好东西是 Aurelia 应用中的所有解耦可以包含在整个应用中一个单独的地方：引导过程。

引导过程允许我们去配置 IoC 容器。IoC 容器知道系统所有的实体，但是实体之间相互隔离。

> 注意：不要认为引导过程（IoC 配置）是[组合根](http://blog.ploeh.dk/2011/07/28/CompositionRoot/)，因为他们不是相同的东西。

#### 现在： Angular 2.x DI 实现

Angular 2.x  IoC 容器创建用于解决 Angular 1.x IoC 容器的问题。

我在线上找到的大部分例子看起来像这样：

![](https://svbtleusercontent.com/u1kxaftjkyixzq_retina.png)

再一次，我们声明在文件的`DataService`包含一个它的依赖的硬编码索引：`Http`类。

就像 Aurelia，我们可以绑定一个字符串字面量到一个类来解决这个问题：

![](https://svbtleusercontent.com/caurimefnz0atg_retina.png)

我们可以使用提供者设置注入`DataService`类到一个组件：

![](https://svbtleusercontent.com/fkcu5vtiuplrqa.png)

这很棒，但是最好将整个应用所有的耦合放在一个单独的文件。这在 Angular 2.x 可以做到，感谢引导进程：

![](https://svbtleusercontent.com/tiyadfirg9zduw.png)

### 当前： InversifyJS 实现

InversifyJs 设计用于鼓励对 SOLID 原则的遵守，允许你去依赖抽象（接口）：

![](https://svbtleusercontent.com/5j7vntnu23ulfg.png)

抽象（接口）在运行时无法使用，因此我们使用字符串字面量替代。就像 Angular 和 Aurelia，我们可以期待未来字符串字面量通过 TypeScript 编译期产生。

![](https://svbtleusercontent.com/3ckuaeh3dsxubq_retina.png)

正如前面我们看到的，InversifyJS 尝试通过泛型类型充分利用 TypeScript 类型安全。InversifyJS 是框架未知的，支持高级 IoC 容器特性，比如[上下文绑定]()和[拦截]()

InversifyJS 应用所有耦合可以包含在唯一的地方：核心配置。

### InversifyJS 和 Angular 2.0 DI

Pascal Precht 关于[Angular 2 中多 Provider](http://blog.thoughtram.io/angular2/2015/11/23/multi-providers-in-angular-2.html)的文章有如下解释：

![](https://svbtleusercontent.com/6n8bh2zfalzeg_retina.png)

当我第一次看到这篇文章的标题的时候，我想这就是我在 InversifyJS 叫做“多注入”，但这是错的：

![](https://svbtleusercontent.com/b15tzdls6kdzog_retina.png)

我们可以说多提供者就是 Angular 2.x 处理我在 InversifyJS 中叫做“上下文绑定”的方式。

![](https://svbtleusercontent.com/m7lfmttlke60qa.gif)

我个人更加喜欢 InversifyJS 实现，因为它确保所有的绑定（抽象和实现的映射）都声明在单一的地方。InversifyJS 文档鼓励命名某个文件为`inversify.config.ts`。文件包含 IoC 容器配置：所有的声明和约束（上下文绑定）。IoC 容器知道所有的实体，但是他们不知道彼此。这让我们可以将整个应用的耦合集中到单一的地方：`inversify.config.ts`文件。

如果我们有一个巨大的应用，我们可以创建多 IoC 容器模块，让`inversify.config.ts `文件更容易维护。

![](https://svbtleusercontent.com/gpk7acpxmhhca.png)

### 未来

未来是关于**设计态类型注解在运行态作为元数据可用**。我知道这个，是因为从 TypeScript 添加装饰器之后，已经看到一些开发者在互联网四处询问他们，也因为我已知在 GitHUb 上关注 TypeScript 问题。

![](https://svbtleusercontent.com/d8nq5mf1illw.png)

我们可以猜测元数据将会使用[Metadata Reflection API](https://www.npmjs.com/package/reflect-metadata)存储。但是这时候我们没有足够的信息去知道类型声明在运行时会如何表示。然而，他将会是下面的一种：

- 字符串字面量
- Symbols
- 对象字面量（JSON）

这意味着我们可以期待，在未来，使用 Angular 2，Aurelia 或者 InversifyJS，达到真正的解耦，字符串字面量将不再需要。元数据将会被编译器生成，因此我们也可以期望看到越来越多的 JavaScript 转换器。

### 总结

小心 Augular 和 Aurelia 文档。他们依赖注入例子不展示怎样使用抽象（字面量直接量）替换实现（类）去避免依赖紧密耦合。我假设他们不展示它是因为字符串字面量是丑陋的，但是他们是达到真正解耦的唯一方式。

我们需要字符串字面量是因为 TypeScript 接口在运行时是不可访问的。这是因为 TypeScript 编译器处理[运行时类型元数据](http://blog.wolksoftware.com/decorators-metadata-reflection-in-typescript-from-novice-to-expert-part-4#3-basic-type-serialization_1)生成，我们可以期待他会在未来修复。

与此同时，轻避免在你的代码使用魔法字符串。尝试声明你所有的字符串字面量在一个常量文件，类似[Redux 中的 action](https://github.com/reactjs/redux/blob/master/examples/todomvc/constants/ActionTypes.js)：

![](https://svbtleusercontent.com/mafc6qmb6t4xg.png)

我们将去实现真正的 JavaScript IoC 容器，不带任何妥协（字符串字面量），两个最有前途的 JavaScript 框架，Angular 2.0 和 Aurelia 包都含一个 IoC 容器并鼓励 SOLOD 原则的实现。

我们当然有其他框架未知的 IoC 容器，比如[InversifyJS](http://inversify.io/)，支持高级 IoC 特性，比如[上下文绑定](https://github.com/inversify/InversifyJS#contextual-bindings--paramnames)或者[拦截](https://github.com/inversify/InversifyJS#activation-handler)。

Google 或者 Microsoft 等大公司支持大框架支持都在 JavaScriot 中使用 IoC。是时候停止认为 IoC 容器在 JavaScript 应用中没有容身之地了，并开始编写走兽 SOLID 原则的 OO JavaScript 代码。

感谢陪伴我，这是一个超长的阅读！

请自由和我们分享这个文章的想法，通过[@OweR_ReLoaDeD](https://twitter.com/OweR_ReLoaDeD)，[@InversifyJS ](https://twitter.com/inversifyjs)，[@WolkSoftwareLtd](https://twitter.com/WolkSoftwareLtd)。

[不要忘了订阅](http://blog.wolksoftware.com/feed)，如果你不像失去未来的文章。