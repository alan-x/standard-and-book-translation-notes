[已校对]
# Jest

> [Pro egghead 关于 Jest/TypeScript 的课程](https://egghead.io/lessons/typescript-getting-started-with-jest-using-typescript)

没有一个测试解决方案是完美的。也就是说，jest 是一个优秀的单元测试选项，提供非常好的 TypeScript 支持。

> 注意：我们假设你从一个简单的 node package.json 设置开始。当然，所有的 TypeScript 文件应该在一个`src`文件夹，这也是一个干净的项目推荐的设置（就算没有 Jest）。


### 步骤1:安装

使用 npm 安装下列：
```ts
npm i jest @types/jest ts-jest typescript -D
```

解释：
- 安装`jest`框架（`jest`）
- 安装`jest`的 types（`@types/jest`）
- 为 jest 安装 TypeScript 预处理器（`ts-jest`），允许 jest 飞快转化 TypeScript，并且有内置的 source-map 支持
- 安装 TypeScript 编译器（`typescript`），他是`ts-jest`的先觉条件。
- 将这些都保存在你的开发依赖中（测试应该总是一个 npm 开发依赖）

### 步骤2：配置 Jest

添加下面的`jest.config.js`文件到你的项目的跟目录：
```ts
module.exports = {
  "roots": [
    "<rootDir>/src"
  ],
  "testMatch": [
    "**/__tests__/**/*.+(ts|tsx|js)",
    "**/?(*.)+(spec|test).+(ts|tsx|js)"
  ],
  "transform": {
    "^.+\\.(ts|tsx)$": "ts-jest"
  },
}
```
（如果你的`package.json`文件包含`"type": "module"`，会导致 Node 假设模块是 es6 格式，你可以转化前面的到 es6 格式，通过替换第一行为`export default {`）。

解释：
- 我们总是推荐将所有的 TypeScript 文件放到你的项目的一个`src`文件夹。我们假设这是真的，并且使用`roots`选项指定这个。

- `testMatch`配置是一个全局模式匹配器，用来发现 ts/tsx/js 格式的 .test/.spec 文件

- `transform`配置只是告诉`jest`为 ts/tsx 文件使用`ts-jest`

### 步骤3：运行测试

从你的项目根目录运行`npx jest`，jest 将会你拥有的所有测试。

#### 可选的：为 npm 脚本添加一个脚本目标

添加`package.json`：
```ts
{
  "test": "jest"
}
```

- 这允许你去运行一个简单的`npm t`
- 使用`npm t -- --watch`启用监听模式

#### 可选的：在监听模式运行 jest

- `npx jest --watch`

#### 例子
- 对于文件`foo.ts`
```ts
  export const sum
    = (...a: number[]) =>
      a.reduce((acc, val) => acc + val, 0);
```
- 一个简单的`foo.test.ts`:
```ts
  import { sum } from '../foo';

  test('basic', () => {
    expect(sum()).toBe(0);
  });

  test('basic again', () => {
    expect(sum(1, 2)).toBe(3);
  });
```
注意：
- Jest 提供了全局的`test`函数
- Jest 带来预定义的断言，以全局`expect`的形式

#### 异步例子

Jest 内建了 async/await 支持。比如：
```ts
test('basic',async () => {
  expect(sum()).toBe(0);
});

test('basic again', async () => {
  expect(sum(1, 2)).toBe(3);
}, 1000 /* optional timeout */);
```

#### enzyme 例子

> [Pro egghead  关于 Enzyme / Jest / TypeScript 的课程](https://egghead.io/lessons/react-test-react-components-and-dom-using-enzyme)

Enzyme 允许你使用 dom 支持去测试 react 组件。设置 enzyme 有三个步骤：

1. 安装 enzyme，enzyme 的 type，一个更好的 enzyme 快照序列器，针对你的 react 版本的 enzyme-adapter-react `npm i enzyme @types/enzyme enzyme-to-json enzyme-adapter-react-16 -D`。

2. 添加`"snapshotSerializers"`和`"setupTestFrameworkScriptFile"`到你的`jest.config.js`。

```ts
 module.exports = {
   // OTHER PORTIONS AS MENTIONED BEFORE

   // Setup Enzyme
   "snapshotSerializers": ["enzyme-to-json/serializer"],
   "setupFilesAfterEnv": ["<rootDir>/src/setupEnzyme.ts"],
 }
```
3. 创建`src/setupEnzyme.ts`文件
```ts
 import { configure } from 'enzyme';
 import EnzymeAdapter from 'enzyme-adapter-react-16';
 configure({ adapter: new EnzymeAdapter() });
```

现在，这是一个 react 组件和测试的版本：

- `checkboxWithLabel.tsx`
```ts
  import * as React from 'react';

  export class CheckboxWithLabel extends React.Component<{
    labelOn: string,
    labelOff: string
  }, {
      isChecked: boolean
    }> {
    constructor(props) {
      super(props);
      this.state = { isChecked: false };
    }

    onChange = () => {
      this.setState({ isChecked: !this.state.isChecked });
    }

    render() {
      return (
        <label>
          <input
            type="checkbox"
            checked={this.state.isChecked}
            onChange={this.onChange}
          />
          {this.state.isChecked ? this.props.labelOn : this.props.labelOff}
        </label>
      );
    }
  }
```
- `checkboxWithLabel.test.tsx`
```ts
  import * as React from 'react';
  import { shallow } from 'enzyme';
  import { CheckboxWithLabel } from './checkboxWithLabel';

  test('CheckboxWithLabel changes the text after click', () => {
    const checkbox = shallow(<CheckboxWithLabel labelOn="On" labelOff="Off" />);

    // Interaction demo
    expect(checkbox.text()).toEqual('Off');
    checkbox.find('input').simulate('change');
    expect(checkbox.text()).toEqual('On');

    // Snapshot demo
    expect(checkbox).toMatchSnapshot();
  });
```

### 为什么我们喜欢 jest 的原因

> [关于这些特性的细节，可以查阅 jest 网站](http://facebook.github.io/jest/ )

- 内置断言库
- 好的 TypeScript 支持
- 非常可靠的测试观察者
- 镜像测试
- 内置覆盖工具
- 内置 async/await 支持