### 属性注入

InversifyJS 支持属性注入，因为有时候构造器注入不是最好的输入模式。然而，你应该尝试避免使用属性注入，并且在大部分场景，推荐使用构造器注入。

> 如果类没有依赖就无法执行它的工作，则添加他到构造器。类需要新的依赖，因此你想要你的改变破坏东西。因此，创建一个没有完成初始化的类（“双步构造”）是一个反模式（IMHO）。如果类一个类没有这个依赖可以工作，一个 setter 就够了。

来源：[http://stackoverflow.com/](http://stackoverflow.com/questions/1503584/dependency-injection-through-constructors-or-property-setters)

有两种场景你可能想要使用属性注入
- 当我们可以使用 InversifyJS 去创建一个类的实例。
- 当我们不能使用 InversifyJS 去创建一个类的实例。

这些场景非常不同，需要不同的属性注入实现。

### 当我们可以使用 InversifyJS 去创建一个类的实例

如果你正在编写一个库或者框架，允许 InversifyJS 在应用中创建类的实例，则你可以使用`@inject`装饰器注入一个属性：
```ts
import { injectable, inject, container } from "inversify";

@injectable()
class PrintService {
    // ...
}

@injectable()
class Summary {
    // ...
}

@injectable()
class Author {
    // ...
}

@injectable()
class Book {

  private _author: Author;
  private _summary: Summary;

  @inject("PrintService")
  private _printService: PrintService;

  public constructor(
      @inject("Author") author: Author,
      @inject("Summary") summary: Summary
) {
    this._author = author;
    this._summary = summary;
  }

  public print() {
     this._printService.print(this);
  }

}

let container = new Container();
container.bind<PrintService>("PrintService").to(PrintService);
container.bind<Author>("Author").to(Author);
container.bind<Summary>("Summary").to(Summary);
container.bind<Book>("Book").to(Book);

// Book instance is created by InversifyJS
let book = container.get<Book>("Book");
book.print();
```

### 当我们不能使用 InversifyJS 去创建一个类的实例

InversifyJS 设计上让它和很多库和框架的集成称为可能。然而，很多特性需要能够在应用中创建实例。

问题是一些框架控制了实例的创建。比如，React 控制给定 Component 的创建。

我们开发了一个工具允许你去注入到一个属性，甚至当 InversifyJS 没有创建它的实例

```ts
import getDecorators from "inversify-inject-decorators";
import { Container, injectable  } from "inversify";

@injectable()
class PrintService {
    // ...
}

let container = new Container();
container.bind<PrintService>("PrintService").to(PrintService);
let { lazyInject } = getDecorators(container);

class Book {

  private _author: string;
  private _summary: string;

  @lazyInject("PrintService")
  private _printService: PrintService;

  public constructor(author: string, summary: string) {
    this._author = author;
    this._summary = summary;
  }

  public print() {
     this._printService.print(this);
  }

}

// Book instance is NOT created by InversifyJS
let book = new Book("Title", "Summary");
book.print();
```

工具模块称为`inversify-inject-decorators`并提供下面的装饰器：
- `@lazyInject`用于注入一个属性，不携带元数据
- `@lazyInjectNamed`用于注入一个属性，不携带命名元数据
- `@lazyInjectTagged`用于注入一个属性，不携带标签元数据
- `@lazyMultiInject`用于多注入

请访问[Github 项目](https://github.com/inversify/inversify-inject-decorators)了解更多。