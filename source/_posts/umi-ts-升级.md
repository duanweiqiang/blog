---
title: umi ts 升级
date: 2021-01-22 20:07:17
img: /medias/featureimages/2.jpg
top: true
cover: true
coverImg: /medias/featureimages/7.jpg
toc: true
mathjax: false
summary: TypeScript是强类型的语言，可通过编译成纯JS，提高代码的质量，TS编译工具也可运行在任何服务器和系统上。
categories: 全站
tags:
- ts
- umi
---

### umi配置ts的过程

#### 背景（为什么要配置TS）

  1.本来项目是一个老的项目，之前有过升级但是由于大家技术水平的不统一，TS学习成本高，导致一直在搁浅中。
  2.由于现在ts的普及以及最近代码质量越来越难以把控，所以引入ts是控制质量的一种方式。

##### ts有啥好呢？
  1.可读性：类型明确，不需要额外注释类型；看到类型就知道怎么用；
  2.可维护性：在编译间断就可以发现错误，避免认为错误出现；
  3.兼容性：兼容js语法；可以直接将`*.js`改为`*.ts`使用；完全支持ES6规范；
  4.第三方库：兼容第三方库，即使第三方库不是用tst写的，也可以编写单独的类型文件供ts读取；

##### ts有啥不好呢？
  1.从纯js转过来有一定的难度，好多写法类似java；
  2.类型定义比较繁琐和js相比；
  3.一些库兼容性不是很好；
  4.需要额外的构建工作量；

#### umi怎么配置

  - 新的umi 3 create工具包会自动帮你配好。老的项目需要手动配置；我们这里主要介绍老项目的配置；

  - 不管是老项目还是新项目，我们至少都要懂他的配置，这样便于我们后期如果项目变庞大复杂了，这是对项目进行优化；
  
##### umi3脚手架配置
  按umi标准文档配置就可以了；
  umi `create-app`文档：https://umijs.org/zh-CN/docs
  ```
    $ yarn create @umijs/umi-app
      Copy:  .editorconfig
      Write: .gitignore
      Copy:  .prettierignore
      Copy:  .prettierrc
      Write: .umirc.ts
      Copy:  mock/.gitkeep
      Write: package.json
      Copy:  README.md
      Copy:  src/pages/index.less
      Copy:  src/pages/index.tsx
      Copy:  tsconfig.json
      Copy:  typings.d.ts
      .
      .
      .
  ```
  目录结构
  ```
    .
    ├── package.json
    ├── .umirc.ts
    ├── .env
    ├── dist
    ├── mock
    ├── public
    └── src
        ├── .umi
        ├── layouts/index.tsx
        ├── pages
            ├── index.less
            └── index.tsx
        └── app.ts
  balabala ... 这里省略一万字（具体看文档就可以了
  ```

##### umi老的项目配置

  (1).umirc.ts
  官方解释： 配置文件，包含 umi 内置功能和插件的配置。
  话不多说，和老的`.js`一样并且可以共用。即运行时配置。

  (2)tsconfig.json
  官方解释： TypeScript项目的根目录；
  我的解释：用来配置编译ts项目的根文件和编译配置；
  ```
  {
    "compilerOptions": {
      "target": "esnext",
      "module": "esnext",
      "moduleResolution": "node",
      "importHelpers": true,
      "jsx": "react",
      "esModuleInterop": true,
      "sourceMap": true,
      "baseUrl": ".",
      "strict": true,
      "allowUnreachableCode":false, // 不报告执行不到的代码错误。
      "allowJs": true, //允许编译javascript文件。
      "allowSyntheticDefaultImports": true, //允许从没有设置默认导出的模块中默认导入。这并不影响代码的输出，仅为了类型检查。
      "experimentalDecorators": true,
      "outDir": "dist/",
      "paths": {
        "@/*": ["src/*"],
        "@*": ["src/*"],
        "@SDVariableJS": ["config/variableConfig.js"],
      }
    },
    "files": [
      "core.ts",
      "sys.ts",
      "types.ts",
    ],
    "include":[],
    "exclude": ["node_modules", "dist"]
  }
  ```

  常用的参数配置：

  `allowUnreachableCode`:默认值-`false` 解释：不报告执行不到的代码错误。
  `allowJs`:默认值-`true` 解释：允许编译javascript文件。
  `allowSyntheticDefaultImports`:默认值-`true` 解释：允许从没有设置默认导出的模块中默认导入。这并不影响代码的输出，仅为了类型检查。
  `noImplicitUseStrict`:默认值-`alse` 解释：模块输出中不包含 "use strict"指令。
  `alwaysStrict`:默认值-`false` 解释：以严格模式解析并为每个源文件生成 "use strict"语句
  `checkJs`:默认值-`false` 解释：在`.js`文件中报告错误。与 `allowJs`配合使用。

  更多参数配置见：https://www.tslang.cn/docs/handbook/compiler-options.html

  (3)typings.d.ts

  释义：存放一些声明，类似于C/C++的.h头文件。(不知道大家理解么，写过C的应该懂这个)

  - TypeScript相比JavaScript增加了类型声明。这些类型声明帮助编译器识别类型，从而防止开发者“搬起石头砸自己的脚”。原则上TypeScript 需要开发者做到先声明后使用。这就导致开发者在调用很多原生接口（浏览器、Node.js）或者第三方模块的时候，因为某些全局变量或者对象的方法并没有声明过，导致编译器的类型检查失败。

  - 用ts写的模块在发布的时候仍然是用js发布;
  这就导致一个问题：ts 那么多类型数据都没了，所以需要一个`*.d.ts`文件来标记某个js库里面对象的类型然后typings就是一个网络上的d.ts数据库;
  
  - `*.d.ts`类型定义文件，我感觉现在对我的用处就是编辑器的智能提示

  (4)tslint.json

  释义：保存了要使用的代码检查器的设置。
  TSLint 对TypeScript 支持得很好，并且如果你使用的是 VsCode IDE，还有出色的插件支持。

  tslint.json
  ```
  {
    "defaultSeverity": "error",
    "extends": [
      "tslint:latest",
      "tslint-react",
      "tslint-config-prettier" // 安装tslint-config-prettier后，tslint-config-prettier禁用TSLint的所有格式设置规则, TSLint 和 prettier在代码格式化规则上就不会有冲突了
    ],
    "jsRules": {},
    "rules": {
      "object-literal-sort-keys": false,
      "no-console": true,
      "jsx-no-lambda": false,
      "no-submodule-imports": false,
      "no-implicit-dependencies": false
    }
  }
  ```

  tslint保存时校验配置
  ```
  "editor.codeActionsOnSave": {
      
      "source.fixAll.eslint": true, // For ESLint
      
      "source.fixAll.tslint": true, // For TSLint
      
      "source.fixAll.stylelint": true， // For Stylelint
  }
  ```
  prettier保存时校验配置
  ```
  "editor.formatOnSave": true,
  ```
  - ps: 使用`tslint-config-prettier`关闭tslint中有关格式的规则，避免tslint与prettier在格式规则上产生冲突。目前我们的项目中（.jsx）中没有开启这一项。
  

  当按ctrl+s保存代码时，tslint插件会自动按照默认配置文件（项目根目录下的`tslint.json`）检查代码错误，prettier插件会自动按照默认配置文件（项目根目录下的`.prettierrc`）检查代码风格，并自动矫正。(不建议使用)

#### VSCode配置
  建议安装tsLint 插件插件，vsCode中直接搜索安装；
  - ps：这个库可`esLint`库可以共存，但是需要进行配置，保证在`.jsx`/`.tsx`文件中只有一个生效，不然错误提示会有误报的情况。
  具体配置地址：https://github.com/Microsoft/typescript-tslint-plugin；

  ![](https://img.imgdb.cn/item/600ee2ed3ffa7d37b3236844.png)

  到此时，我们的环境就配好了，接下来写一个小的React的demo.tsx吧；

#### React typescript写法 
  几个概念React.FC：
  -FC = Functional Component
  -SFC = Stateless Functional Component (已弃用)

  学习typescript的基本语法,具体ts见：https://www.runoob.com/typescript/ts-variables.html
  
  - ps: 我有视频，如有需要可找我要哈😄；

  有了以上typescrt姿势，我们就可以开始写ts代码了。⛽️

  下面是我写的两种常见写法的demo；

##### 泛型组件写法(对标Class写法)

```javascript

import React from 'react';
import UseInfo from "./UseInfo"; // 引用其他组件（可以是.tsx，也可以是.jsx）
import style from './test.less'; // 引用样式文件

function getExclamationMarks(numChars: number) {
  return Array(numChars + 1).join('!');
}

interface Props {
  name: string;
  enthusiasmLevel: number;
}
interface State {
  now: string,
}

class Test extends React.Component<Props, State> {
  constructor(props:Props) {
    super(props);
    this.state = {
      now: "fff",
    };
  }
  public clickTest = (x:string) => {
    console.log(x)
    this.setState({now: '###'});
  }
  render() {
    const { name, enthusiasmLevel = 1 } = this.props;
    const { now } = this.state;

    console.log('@@@@@');
    const hello : string = 111
    console.log(hello)

    if (enthusiasmLevel <= 0) {
      throw new Error('You could be a little more enthusiastic. :D');
    }

    return (
      <div className="hello">
        <UseInfo />
        <div className="greeting">
          {`Hello ${name + getExclamationMarks(enthusiasmLevel)}`}
        </div>
        <div className={style.button} onClick={()=>this.clickTest(now)}></div>
      </div>
    );
  }
}
export default Test;

```

##### Hooks写法

```javascript
import React, { Fragment, useState } from 'react';
import UseInfo from "./UseInfo"; // 引用其他组件（可以是.tsx，也可以是.jsx）
import style from './base.less'; // 引用样式文件

interface IProps {
    changeTitle?: () => void;
}

const Auth = (props: IProps) => {
    const [houseTitle, setHoueseTitle] = useState<string>('')
    return (<Fragment>
      <useInfo />
      <div>
        <input
            type="text"
            className={style.demo1}
            placeholder={'please input'}
            value={houseTitle}
            onChange={changeTitle}
        />
    </div>
    </Fragment>)
    function changeTitle(e:any) { // 房屋标题
        setHoueseTitle(e.target.value)
    }
}
export default Auth
```
好了，我们的一个完整的`umi`支持`ts`写法到这里就结束了，希望大家都学会了。

#### 结语
随着前端技术的发展。弱类型基本不能满足我们日益复杂的业务，强类型语言是未来的趋势。
Framework - Angular2、UI - ant-design、library - RxJS等项目已经都迁移到typescript了，我们还不跟进吗。。。