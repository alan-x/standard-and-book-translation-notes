### 对分层 DI 系统的支持

一些应用使用一个分层依赖注入（DI）系统。比如，Angular 2.0 应用使用它自己的[分层依赖系统](https://angular.io/docs/ts/latest/guide/hierarchical-dependency-injection.html)

在层级 DI 系统中，一个容器可以有一个父容器，多个容器可以用于一个应用。这些容器组成了一个分层架构。

当一个分层结构的底部的容器请求一个依赖，容器尝试去使用它自己的绑定去满足这个依赖。如果容器缺少这个绑定，它向上传递请求到父容器。如果容器不能满足请求，它直接传递到它的父容器。请求保持向上冒泡知道我们发现一个容器可以处理请求或者抛出容器祖先。


```ts
let weaponIdentifier = "Weapon";

@injectable()
class Katana {}
 
let parentContainer = new Container();
parentContainer.bind(weaponIdentifier).to(Katana);
 
let childContainer = new Container();
childContainer.parent = parentContainer;

expect(childContainer.get(weaponIdentifier)).to.be.instanceOf(Katana); // true
```