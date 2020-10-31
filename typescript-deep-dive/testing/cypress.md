# Cypress

Cypress 是一个很好的 E2E 测试工具。这是考虑它的一些很好的原因：

- 独立安装的可能性。
- 附带开箱即用的 TypeScript 支持。
- 提供 google chrome 好的调试交互体验。折合 UI 开发的方式基本一样。
- 有命令-执行分离，允许威力更强大的调试和测试稳定性（更多在下面）
- 使用隐式断言去提供更有意义的调试体验和更少的脆弱（跟多的提示在下面）
- 提供 mock 和观测后端的能力，不需要改变你的应用代码（更多提示在下面）

### 安装

这个安装过程提供的步骤将会给你一个很好的`e2e`文件夹，你可以复制/粘贴或者作为你的组织的样板。

> 以视频格式存在的相同的步骤在我的[youtube 频道]()

创建 e2e 文件夹，安装 cypress，TypeScript 和设置 typescript 和 cypress 配置文件：
```ts
mkdir e2e
cd e2e
npm init -y
npm install cypress typescript
npx tsc --init --types cypress --lib dom,es6
echo {} > cypress.json
```

> 这是专门为 cypress 创建一个分离的`e2e`文件夹的一些原因：

> - 创建一个分离的文件夹或者`e2e`让你从项目分离`package.json`依赖更加简单。这让依赖冲突更少。
> - 测试框架有污染全局命名空间的习惯，使用类似`describe``it``expect`。最好保持 e2e 的`tsconfig.json`和`node_modules`在特定的`e2e`文件夹，放置全局类型定义冲突。

添加一些脚本到`e2e/package.json`文件：
```ts
  "scripts": {
    "cypress:open": "cypress open",
    "cypress:run": "cypress run"
  },
```
在`cypress/integration/basic.ts`编写你的第一个测试文件夹：
```ts
it('should perform basic google search', () => {
  cy.visit('https://google.com');
  cy.get('[name="q"]')
    .type('subscribe')
    .type('{enter}');
});
```

现在在开发的时候运行`npm run cypress:open`，并在你的构建服务器运行`npm run cypress:run`。


### 关键文件的更多描述

在`e2e`文件夹下，你现在有了这些文件：

- `/cypress.json`：配置 cypress。默认是空的，这就是你所需要的。
- `/cypress`子文件夹：
    - `/integration`：所有测试
        - 为了更好的组织，自由在子文件夹下创建测试，比如`/someFeatureFolder/something.spec.ts`。

### 第一个测试

- 使用下面的内容创建一个文件`/cypress/integration/first.ts`：
```ts
describe('google search', () => {
  it('should work', () => {
    cy.visit('http://www.google.com');
    cy.get('#lst-ib').type('Hello world{enter}')
  });
});
```

### 在开发时候运行

使用下面的命令打开 cypress IDE：
```ts
npm run cypress:open
```
然后选择一个测试运行

### 在一个构建服务器运行

你可以使用下面的命令以 ci 模式去运行 cypress 测试
```ts
npm run cypress:run
```

### 提示：在 UI 和测试之间分享代码

Cypress 测试被编译/打包并运行在浏览器。因此自由的导入任何项目到你的测试。

比如你可以在 UI 和测试之间共享 Id，让 CSS 选择器不被破坏：
```ts
import { Ids } from '../../../src/app/constants';

// Later
cy.get(`#${Ids.username}`)
  .type('john')
```

### 提示：创建 Page 对象

创建为各种需要对所有交互的测试提供便利的处理器是一种常见的的测试约定。你可以使用 带 getter 和方法的 TypeScript 类创建一个页面对象，比如：
```ts
import { Ids } from '../../../src/app/constants';

class LoginPage {
  visit() {
    cy.visit('/login');
  }

  get username() {
    return cy.get(`#${Ids.username}`);
  }
}
const page = new LoginPage();

// Later
page.visit();

page.username.type('john');
```

### 提示：明确断言

Cypress 自带（内置）chai 和 chai-query 断言库去帮助测试网页，你使用`.should`命令使用他们，将连接器作为字符串传递，使用`should('foo')`替换`.to.foo `。比如，使用 chai-query 你应该`expect($(#foo)).to.have.text('something')`，使用 cypress，你可以`cy.get('#foo').should('have.text', 'something')`：
```ts
cy.get('#foo')
  .should('have.text', 'something')
```

> 你将会得到`should`连接器的智能提示，因为 cypress 懈怠了正确的 TypeScript 定义。

完整的连接器列表在这里可以得到：[https://docs.cypress.io/guides/references/assertions.html]()

如果你想要一些复杂的东西，你甚至可以使用`should(callback)`，比如：
```ts
cy.get('div')
  .should(($div) => {
    expect($div).to.have.length(1);
    expect($div[0].className).to.contain('heading');
  })
// This is just an example. Normally you would `.should('have.class', 'heading')
```

> 提示：cypress 会自动在回调重试，因此他们只是作为标准字符串连接器的面具。

### 提示：命令和链

一个 cypress 链中的每一个函数调用都是一个`command`。`should`命令是一个断言。它是一个独立类别的链的便捷开始，并且独立执行。比如：
```ts
// Don't do this
cy.get(/**something*/)
  .should(/**something*/)
  .click()
  .should(/**something*/)
  .get(/**something else*/)
  .should(/**something*/)

// Prefer separating the two gets
cy.get(/**something*/)
  .should(/**something*/)
  .click()
  .should(/**something*/)

cy.get(/**something else*/)
  .should(/**something*/)
```

一些其他库同时求值和运行代码。这些库强制你去使用单独的链，其中混合了选择器和断言，可能是调试的噩梦。

Cypress 命令基本上是声明式的，cypress 运行时会在之后运行命令。换句话说：Cypress 让它更简单。

### 提示：使用`contains`做简单的查询

限免展示了一个例子：
```ts
cy.get('#foo')
  // Once #foo is found the following:
  .contains('Submit')
  .click()
  // ^ will continue to search for something that has text `Submit` and fail if it times out.
  // ^ After it is found trigger a click on the HTML Node that contained the text `Submit`.
```

### 提示：智能延迟和重试

Cypress 将会自动等待很多异步的东西。比如：
```ts
// If there is no request against the `foo` alias cypress will wait for 4 seconds automatically
cy.wait('@foo')
// If there is no element with id #foo cypress will wait for 4 seconds automatically and keep retrying
cy.get('#foo')
```

这让你避免添加任意定时器（和重试）逻辑在你的测试代码流。

### 提示：隐式断言

Cypress 有隐式断言的概念。如果一个未来的命令因为前面的命令失败，这就会触发。比如，下面将会在`contains`（当然在自动重试之后）失败，因为没有东西可以被`click`：
```ts
cy.get('#foo')
  // Once #foo is found the following:
  .contains('Submit')
  .click()
  // ^ Error: #foo does not have anything that `contains` `'Submit'`
```

在传统框架，你将会得到一个糟糕的错误，比如`click`不存在于`null`。在 Cypress，你会得到一个漂亮的错误，`#foo`不包含`Submit`。这个错误是一个隐式断言的形式。

### 提示：等待一个 HTTP 请求

很多的测试因为应用发起的 XHR 的任何定时器而变得脆弱。`cy.server`让他更简单：

- 创建一个后端调用的别名
- 等待他们发生

比如：
```ts
cy.server()
  .route('POST', 'https://example.com/api/application/load')
  .as('load') // create an alias

// Start test
cy.visit('/')

// wait for the call
cy.wait('@load')

// Now the data is loaded
```

### 提示：mock 一个 HTTP 请求响应

你也可以使用`route`轻松模拟一个请求响应：
```ts
cy.server()
  .route('POST', 'https://example.com/api/application/load', /* Example payload response */{success:true});
```

#### 提示： 断言一个 HTTP 请求响应

你可以使用`route``onRequest`/`onResponse`断言请求，而不实用模拟，比如：
```ts
cy.route({
  method: 'POST',
  url: 'https://example.com/api/application/load',
  onRequest: (xhr) => {
    // Example assertion
    expect(xhr.request.body.data).to.deep.equal({success:true});
  }
})
```

### 提示：mock 时间

你可以使用`wait`去暂停一个测试一段时间，比如，测试一个自动的“you are about to be logged out”通知在屏幕上：
```ts
cy.visit('/');
cy.wait(waitMilliseconds);
cy.get('#logoutNotification').should('be.visible');
```

然而，推荐使用`cy.clock`去模拟时间，并使用`cy.tick`去前进。比如：
```ts
cy.clock();

cy.visit('/');
cy.tick(waitMilliseconds);
cy.get('#logoutNotification').should('be.visible');
```

### 提示：单元测试应用代码

你也可以使用 cypress 独立去单元测试你的应用代码，比如：
```ts
import { once } from '../../../src/app/utils';

// Later
it('should only call function once', () => {
  let called = 0;
  const callMe = once(()=>called++);
  callMe();
  callMe();
  expect(called).to.equal(1);
});
```

### 提示：单元测试中的 mock

如果你在单元测试你的应用的模块，你可以使用`cy.stub`提供 mock。比如，如果你想要确保`navigate`在一个函数`foo`调用：

- `foo.ts`
```ts
import { navigate } from 'takeme';

export function foo() {
  navigate('/foo');
}
```

- 你可以像在`some.spec.ts`这样做：
```ts
/// <reference types="cypress"/>

import { foo } from '../../../src/app/foo';
import * as takeme from 'takeme';

describe('should work', () => {
  it('should stub it', () => {
    cy.stub(takeme, 'navigate');
    foo();
    expect(takeme.navigate).to.have.been.calledWith('/foo');
  });
});
```

### 提示：命令行 - 执行分离

当你调用一个 cypress 命令（或者断言），比如`cy.get('#something')`，函数立即返回而没有真实执行动作。它做了什么，是通知 cypress 测试运行器你将会在某个时刻需要携带（执行）一个动作（在这个场景是`get`）。

你基本上是构建了一个命令列表，运行器将会前进和执行。你可以检查这个命令-使用一个简单的测试分离运行，你会发现`start / between / end``console.log`语句立即执行，在运行期开始运行这个命令之前：
```ts
/// <reference types="cypress"/>

describe('Hello world', () => {
  it('demonstrate command - execution separation', () => {
    console.log('start');
    cy.visit('http://www.google.com');
    console.log('between');
    cy.get('.gLFyf').type('Hello world');
    console.log('end');
  });
});
```

命令执行分离有两个好处：

- 运行器可以耐剥落的方式运行命令，并自动重试和隐式断言。

### 提示：断点

cypress 自动生成的快照 + 命令日志对于调试帮助非常大。也就是说你可以停止测试执行，如果你愿意。

首先确保你在测试运行期打开了 chrome 开发者工具（mac 上按`CMD + ALT + i`/window 按 F12）。一旦开发工具打开，你可以重新运行测试，开发工具将保持打开。如果你打开了开发工具，你可以以两种方式暂停测试执行：

- 应用代码断言：在你的应用使用一个`debugger`语句，测试运行器将会像标准 web 应用一样停止运行。

- 测试代码断言：聂可以使用`.debug()`命令，cypress 测试运行将会在这里停止。或者，你可以使用一个`debugger`语句在`.then`命令回调中去制造一个暂停。比如`.then(() => { debugger })`。你甚至可以使用它去捕获一些元素`cy.get('#foo').then(($ /* a reference to the dom element */) => { debugger; })`或者网络调用，比如：`cy.request('https://someurl').then((res /* network response */) => { debugger });`。然而，惯用的方式是`cy.get('#foo').debug()`，然后当测试运行器会在`debug`停止，你可以在命令日志中点击`get`去自动`console.log`任何你需要的关于`.get('#foo')`命令（其他你想要调试的命令也类似）。

### 提示：开始服务器和测试

如果你需要在你的测试运行之前启动本地服务，你可以添加`start-server-and-test`[https://github.com/bahmutov/start-server-and-test]()作为一个依赖。它接受下面的参数

- 一个 npm 脚本去运行服务器(aka 服务)
- 一个后端去检查，如果服务器启动了（aka start）
- 一个 npm 脚本去初始化测试（aka test）


package.json 例子：
```ts
{
    "scripts": {
        "start-server": "npm start",
        "run-tests": "mocha e2e-spec.js",
        "ci": "start-server-and-test start-server http://localhost:8080 run-tests"
    }
}
```

### 资源

- 网站：[https://www.cypress.io/]()
- 编写你的第一个 cypress 测试（给你一个漂亮的 cypress IDE 旅程）：[https://docs.cypress.io/guides/getting-started/writing-your-first-test.html]()
- 设置一个 CI 环境（比如，`cypress run`提供的开箱即用的 docker 镜像）：[https://docs.cypress.io/guides/guides/continuous-integration.html]()
- 方法（列出食谱和描述。点击标题去导航到方法的源代码）：[https://docs.cypress.io/examples/examples/recipes.html]()
- 虚拟测试：[https://docs.cypress.io/guides/tooling/visual-testing.html]()
- 可选的设置 cypress.json 的`baseUrl`去[防止第一次`visit`之后的一个初始化重载]()
- 使用 cypress 的代码覆盖：[Webcase]()