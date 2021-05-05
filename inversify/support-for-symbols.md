### 对 Symbol 的支持

在大型应用中，使用字符串作为被 InversifyJS 注入的类型标识符会导致命名冲突。InversifyJS 支持并且推荐使用[Symbols](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Symbol)替代字符串字面量。

> symbol 是唯一并且不可变的数据类型，可以作为对象属性标识符。symbol 对象是一个 symbol 原始数据类型的隐式对象装箱。

```ts
import { Container, injectable, inject } from "inversify";

let Symbols = {
	Ninja : Symbol.for("Ninja"),
	Katana : Symbol.for("Katana"),
	Shuriken : Symbol.for("Shuriken")
};

@injectable()
class Katana implements Katana {
    public hit() {
        return "cut!";
    }
}

@injectable()
class Shuriken implements Shuriken {
    public throw() {
        return "hit!";
    }
}

@injectable()
class Ninja implements Ninja {

    private _katana: Katana;
    private _shuriken: Shuriken;

    public constructor(
	    @inject(Symbols.Katana) katana: Katana,
	    @inject(Symbols.Shuriken) shuriken: Shuriken
    ) {
        this._katana = katana;
        this._shuriken = shuriken;
    }

    public fight() { return this._katana.hit(); };
    public sneak() { return this._shuriken.throw(); };

}

var container = new Container();
container.bind<Ninja>(Symbols.Ninja).to(Ninja);
container.bind<Katana>(Symbols.Katana).to(Katana);
container.bind<Shuriken>(Symbols.Shuriken).to(Shuriken);
```