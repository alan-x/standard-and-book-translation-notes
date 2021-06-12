## 第一部分，JavaScript 起源

2. 前史

万维网的理念和基础技术在 1989-1991 被 Tim Berners-Lee[2003] 在 CERN 研发。Berners-Lee 的网页技术流传在高能无力社区多年。然而，他没有在社区外接到更多注意，直到 Marc Andressen，一个大学生，和 Eric Bina，在 Illinois 大学的 Urbana-Champaign National Center for Supercomputing Applications（NCSA）工作，在 1992-1993 年开发了 Mosaic。

NCSA Mosaic 是一个安装简单，使用简单的网页客户端，有一个图形化的用户接口。它本质上定义了软件类型“网页浏览器”，并在物理社区外普及了王维网的概念。Mosaic 被广泛分发，到了 1993 年早期，商业利益争先恐后追逐浏览器潮流，通过授权 NCSA Mosaic 代码或者完全构建扩展自 Mosaic 启发的浏览器。Jim Clark，Silicon Graohic Inc. 的创始人，获得了风险投资，并雇佣了 Mac Andreessen 和 Eric Bina。在 1994 年 4 月，他们共同创立了最终被命名为 Netscape Communications Corporation 的公司。Netscape 将它的目标设置为替代 NCSA Mosaic，成为世界上最流行的浏览器。它是从头开始开发开发并增强的下一代 Mosaic 类似的浏览器，他在 1994 年 9 月开始广泛分发。到了 1995 年早期，Netscape Mavigator 达到了他的初始目标，并加速取代 Mosaic。

Tim Berners-Lee 的网页技术核心是使用声明性的 HTML 标记语言去描述文档的展示为网页。于此相反，业界对使用脚本语言[Ousterhout 1997] 去让终端用户去编排他们应用的操作非常有兴趣。类似 Microsoft Office 的 Visual Basic 和 Apple-Script[Cook 2007]不是为了是为了实现复杂数据结构和主要应用的核心算法组件。相反，他们提供用户一个方法以艺术的方式去粘合这类应用。正如 Netscape 扩展了王维网的观众，一个重要的问题是脚本应该如何继承到网页。


### 2.1 Brendan Eich 加入 Netscape

Brendan Eich，1985 年，完成了他在 University of Illinois Urbana-Champaign 的教授生涯，并立即加入 Silicon Graphics, Inc 工作。他主要研究 Unix 内核和网络层。在 1992 年他离开 SGI 并加入 MicroUnity，一个资金充足的新创公司，开发视频媒体处理器。在两个公司，他实现了一个小的指定目的的语言，支持内核和网络程序任我。在 MicroUnity，他也为 GCC 编译器做了一些工作。

在 1995 年早期，Brendan Eich 被 Netscape 使用“来这里做浏览器 Schema”的诱饵雇佣。但是当 Eich 在 1995 年 4 月 3 日加入 Netscape，他发现了一个复杂产品市场和编程语言场景。Netscape 在 1994 年拒绝了 Microsoft 的低价收购，之后 Netscape 管理者预期会遭到 Microsoft 的“拥抱，扩展，扑灭”策略[Wikipedia 2019]的直接攻击。Microsoft，在 Bill Gate 的直接领导下，很快意识到他将推出的专有花园墙信息工具，Project Blackbird[Anderson 2007]，将会无缘于网页作为一个跨系统平台的兴起。盖茨的“Internet Tidal Wave”备忘[Gates 1995]将 Microsoft 重新启动到 Blackbird 到 Internet Explorer，和完整的服务端产品套件，因为 Netscape 急于占据相同市场。

网页脚本语言的候选包括研究语言，比如 Schema；实用的基于 Unix 专属语言，比如 Perl，Python，和 Tcl；专属语言，比如 Microsoft 的 Visual Basic。Brendan Eich 期待在浏览器中实现 Schema。但是在 1995 年早期，Sub Microsystem 开始了一个游击营销活动[Byous 1998]，因为他还没有发布 Java 语言。Sun 和 Netscape 快接触达成一个协议，那就是将 Java 集成到 Netscape 2 中。Eich 回忆说，Marc Andreessen 在 Netscape 集会上的发言是“Netscape 加上 Java 杀死 Windows”。在 1995 年 5 月 23 日，在 Sun 的 Java 公开发布中，Netscape 声明他授权 Sun 的 Java 技术[Netscape 1995a]在浏览器中使用。

Netscape 内部快速制定策略去选择一个脚本语言，有很多缺陷，Schema，Perl，Python，Tcl，和 Visual Basic 都不可用，因为商业利益和/或市场考虑。Netscape 和 Sun 高级管理者 Marc Andreessen 和 Sun 的 Bill Joy 认为唯一可行的方案设计和实现一个“小语言”去完善 Java。

怀疑者，在 Sun 中主导，在 Netscape 占多数，怀疑是否需要一个简单的脚本语言：Java 不适合脚本吗；能否解释为什么两个语言比一个语言好；Netscape 是否有创建一个新语言的专业知识。

第一个反对很容易反驳。Java 在 1995 年春天对于新手不太适合。意识需要包裹一个主程序代码体在一个包中的一个类中命名为 main 静态方法中。一是需要为所有参数声明静态类型，返回值，和变量。基于 Vsiual BASIC 补充 Visual C++，和很多 Unix 语言补充基于原生代码组件的经验，很明显，对于“胶水”脚本，Java 不够简单。

第二个反对通过引用 Microsoft 的产品来克服。对于专业 Window 应用程序员，Microsoft 售卖 Visual C++。对于爱好者，业余程序员，设计师，会计师，和其他。Microsoft 提供 Visual Basic 作为脚本语言，这些经验较少的，业余的程序员可以和使用 Visual C++ 构建的自定义组件”粘合“在一起。有一个叫做”应用的 Versual Basic“的 Visual Basic 版本被集成到 Microsoft Office 应用去支持用户扩展和驱动这些程序。

克服了前面两个反对，Marc Andreessen 提议将浏览器脚本语言叫做“Mocha”，根据 Eric 说，希望在适当的时候改名为“JavaScript”。这个 Java 的伴侣语言必须“看起来像 Java”，但是依旧保持使用简单，并且是“基于对象”而不是“基于类”，类似 Java。

此时依旧留下一个最后的反对：Netscape 是否有经验去创建一个有效的脚本语言，还有是否能在 1995 年九月为 Netscape 2 准备好。Brendan Eich 的任务是通过创建 Mocha 来证明他确实做到的。

### Mocha 的故事

随着 Java 通告的临近，Brendan Eich 将时间视为本质，一只鸟在手胜过灌木丛的许多可能；因此他在 1995 年 5 月连续的十天内构建了第一个 Mocha 原型实现。这个工作被赶在可行性论证之前。这个案例由最低限度语言实现，最小集成到 Netscape 2 pre-alpha 浏览器。

Eich 的原型开发在 Silicon Graphics Indy Unix[Netfeak 2019]。原型使用手写词法分析器和递归下降解析器。转化器发送字节码指令而不是不是解析树。字节码解释非常简单和慢。

字节码是 Netscape 的 LiveWire 服务器的需要，他们的开发者十分期待嵌入 Mocha，甚至在他原型之前。团队的前 Borland 管理和工程师职员非常相信动态语言，但是想要字节码，而不是源码解析，为了更妙的服务应用加载。

Marc Andreessen 强调 Mocha 使用应该非常简单，任何人都可以直接在一个 HTML 文档中直接写几行，Sun 和 Netscape 的上级管理人重申 Mocha 的需求是“看起来像 Java”，明确的要移除任何类似 BASIC 的东西。但是类似 Java 的外表创建一个类似 Java 的行为的期望，包括对象模型和类似 boolean，int，double，和 string 之类的语意。

除了看起来像 Java，Brendan Eich 可以自由选择语言设计的细节。在加入 Netscape 之后，他探索了“简单使用”或者“教学”语言，包括 HyperTalk[Appler Compoter 1998]，Logo[Papert 1980]，和 Self[ungar 和 Smith, 1987]。每个人都同意  Mocha 应该“基于类”，但是没有类，因为支持类花费较长时间，并且会和 Java 竞争。处于对 Self 的敬佩，Eich 选择使用一个单一的原型链动态去代理对象模型。他相信这回解决时间，尽管直到最后，都没有时间在 Mocha 原型中暴露这个机制。

对象通过应用一个新的操作到一个构造器函数创建。一个默认对象构造器函数叫做 Object，和其他内置对象一起内置到环境。每一个对象都由零个或者多个属性构成。每一个属性都有名字（也叫做属性 key）和一个值，可以是一个函数，一个对象，或者多个其他内置数据类型中的一个。属性通过赋值一个值到一个未使用的属性键。属性没有可见性或者赋值约束。一个构造器函数可以提供一个属性的初始集合；其他属性可以在它创建之后添加到一个对象。这中非常动态的是想非常受 LiveWire 团队喜欢。

尽管 Schema 的诱惑已经过去了，Brendan Eich 依旧找到了类似 Lisp 的一等函数实现。没有类去包含方法，一等函数为受 Schema 启发的习语提供了一个工具箱：顶级程序，传递函数作为参数，对象方法，和事件处理器。时间约束要求函数表达式延迟（也叫做 lambda expression，或者就是 lambdas），但是他们保留了语法。事件处理器和对象方法统一从 Java（在 C++ 之后）借用 this 关键字在任何函数中贡献上下文对象，函数被作为一个方法调用。

在 Mac Andreessen 和一些早期 Netscape 工程师的非正式讨论推动下，原型支持一个 eval 函数，可以解析和支持一个字符串，包含一个程序。直觉是这种类型的动态字符串到程序编程对于一些网页浏览器和服务器非常重要。但是支持 eval 的决定立马达成一直。一些使用需要函数提供他们的源代码作为一个字符串，通过类似一个类似 Java 的 toString 方法。Eich 选择在他的十天短跑去实现一个字符串反编译器，因为源代码主要从第二存储存储或者读取，对于一些目标架构来说泰国昂贵。特别是 Window 3.1 个人电脑，受限于 Intel 8086 16 位分段记忆模型，需要为未包裹或者大内存结构覆盖和手动管理多段内存。

在十天的末尾，原型在一个包含 Netscape 全部工程师职员的会议上展示（图片 2）。他是一个成功，这导致对于集成一个更完善和完全融合版本到预计九月份发行的第一个测试发行 Netscape 2 发行过于乐观。Brendan Eich 的主要关注在于夏天完全集成 Mocha 到浏览器。这需要设计和实现让 Mocha 程序去和网页交互的 API。同时，他还让语言原型实现到可交付软件并响应早期内部用户的缺陷报告，修改意见和特性要求。

Mocha 10 天创建的更多细节在 Brendan Eich 对这个故事的复述[Eich 2008c, 2011d；JavaScript Jabber 2014；Walker 2018]。这个产品版本的 Mocha 源码可以通过 Internet Archive[Netscape 1997b]。Jamie Zawinski 的[1999]“netscape 宿舍”记录了这个时期 Netscape 软件开发者的经验。

