# 代码规范

规范的主要目的在于统一代码风格，提高代码质量，保证开发的一致性，与 lint 的目的基本相同。此规范主要是对 lint 无法涉及部分的补充，其它规则请参照相关 lint 设置。

**通用规范**: 适用项目中的所有代码，优先级低于具体规范。

**具体规范**: 包括 Component，Service，Directive，Css 等。

**释义**:

  bad： 不好的写法

  good：好的写法

  order: \<number\>：以数字顺序表示书写时的顺序，数字越小越靠前。

---

## 编辑器

统一使用 vscode，如需使用其它编辑器，请自行搜索具有相似功能的插件或工具。

以下为项目开发必须安装的 vscode 插件：

- Angular Language Service - Angular 官方插件。
- TypeScript Import Sorter - 让 Import 按次序排列，便于阅读，此外还可以移除未使用的 import 等。
- Code Spell Checker - 帮助检查单词拼写错误，提高代码质量。
- Better Comments - 提高注释的可读性，直观的分辨注释类型。

## 通用规范

### 所有的函数，方法都必须遵循单一职责原则，函数的代码量最大不超过 45 行

在 13.5 英寸屏幕上，一屏的显示行数正常情况下就是 45 行，这样无论多大屏幕你都可以在一屏内阅读完整个函数的内容。

### 缩进层级应该保持在 3 级以下

这里的缩进层级更准确的说是由大括号引发的缩进层级，一般情况下缩进越多，也就意味着流程控制越复杂，可以根据函数体内大括号的嵌套层级来表示。示例如下：

以下函数的缩进层级视为 0 级

```ts
function launchShareStrategy(paramsObs: Observable<ShareStrategyStateSnapshot>): Subscription {
  return this.process
    .processShareStrategy(
      paramsObs.pipe(
        switchMap(params =>
          this.confirmStrategyShare(params, ShareConfirmComponent).pipe(
            tap(confirmType => confirmType === ConfirmType.INNER && this.genKey$.next([params, confirmType])),
            filter(confirmType => confirmType !== ConfirmType.INNER),
            mapTo({ id: params.id, type: params.type })
          )
        )
      )
    )
    .add(this.launchGenKey());
}
```

你可以将上面的函数改造成以下形式, 其缩进层级仍为 0 级

```ts
function launchShareStrategy(paramsObs: Observable<ShareStrategyStateSnapshot>): Subscription {
  const mapFn = params =>
    this.confirmStrategyShare(params, ShareConfirmComponent).pipe(
      tap(confirmType => confirmType === ConfirmType.INNER && this.genKey$.next([params, confirmType])),
      filter(confirmType => confirmType !== ConfirmType.INNER),
      mapTo({ id: params.id, type: params.type })
    );

  return this.process.processShareStrategy(paramsObs.pipe(switchMap(mapFn))).add(this.launchGenKey());
}
```

以下函数的缩进层级为 3 级

```ts
function getArgSelectedItem(id: number): VariableTypeDes {
  if (id > 5) {
    try {
      if (id < 100) {
        const res = getUserById(id);

        return res.des;
      } else {
        const realId = transformId(id);
        const res = getUserById(realIda);

        return res.des;
      }
    } catch (e) {
      throw new Error('Can not find user by Id');
    }
  }

  return null;
}
```

### 函数的圈复杂度原则上不超过 3 级，最高不得超 5 级

圈复杂度是指示函数中复杂程度的代码度量。高圈复杂度表示可能容易出错或难以修改的易混淆的代码。圈复杂度实际上就是等于判定节点的数量再加上 1，判定节点包括 if else 、switch case 、 for 循环、三元运算符，逻辑或、与、非等等，示例如下:

```ts
/**
 * 以下函数的圈复杂度为: 1(if) + 1(for) + 2(case) + 1(ternary) + 1 = 6
 */
function testComplexity(param) {
  let result = 1;

  if (param > 0) {
    result--;
  }

  for (let i = 0; i < 10; i++) {
    result += Math.random();
  }

  switch (parseInt(result)) {
    case 1:
      result += 20;
      break;
    case 2:
      result += 30;
      break;
    default:
      result += 10;
      break;
  }

  return result > 20 ? result : result;
}
```

### import 语句格式化

必须对 import 语句进行排序，这样查找和阅读起来更加方便。使用方法：ctrl + shift + p (mac: command + shift + p), 输入 import sort
js和ts中，所有的 import 语句必须处于文件的最上方，禁止在文件中的其它地方 import（除懒加载路由）

### class 上的所有字段和方法必须有注释

compodoc 会根据注释生成项目文档。属性或方法的语义特别明显时使用 @ignore 显式忽略。

### 变量声明优先使用 const，其次是 let，禁用 var

const 和 let 在绝大多数情况下都可以满足需求，唯一可能用到 var 的地方是 declare 声明全局变量，这种情况应尽量避免。因此在引用第三方依赖里尽量选择具有声明文件的依赖。

### 声明语句与逻辑主体间保留一个空行, 不同逻辑主体之间保留一个空行

这里的声明语句一般情况下指的就是变量声明，示例如下：

```ts
function calc(a: number, b: number): number {
  const radio = Math.random() * 10;
  const base = 20;
                                  // 这个空行表示声明语句结束
  if (a < 0 || b < 0) {
    return 0;
  }
                                  // 这个空行表示第一段逻辑结束
  if (radio < 5) {
    return 1;
  }
                                  // 这个空行表示第二段逻辑结束
  return (a + b) * radio + base;
}
```

### 命名规范

1. 变量声明必须使用**名词**;函数和方法必须以**动词**开头。特例：谓词变量或函数，指那些表达逻辑 true 和 false 的变量，或返回值为 true 或 false 的函数，此类变量或函数必须以**谓词**开头，常用的有 is,should, need, can 等。示例：

```ts
const sendProduct = 2; // bad 动词开头

const product = 'xx'; // good 名词

/**
 * bad 名词开头
 */
function chartConfig() {
  // ....
}

/**
 * good 以动词 generate 开头
 */
function generateChartConfig() {
  // ...
}

class Person {
  showTable = false; // bad
  shouldShowTable = false; // good
}
```

2. ts，js 文件中变量的命名采用 camelCase（小驼峰） 方式。常量命名使用纯大写字母，单词间以下划线'_'分隔。示例：

```ts
const useid = 'xxx'; // bad
const userId = 'xxx'; // good

const datetime = '2019-12-13'; //bad
const dateTime = '2019-12-13'; // good

const default_url = '/api/v1'; // bad
const DEFAULT_URL = '/api/v1'; // good
```

3. 禁止滥用缩写，尤其类似于'BTC'这种脱离业务后根本无法推断出语义的缩写。业务专有名词缩写应单独建立文档（TODO）进行释义。示例：

```ts
const wx = 20838198; // bad 阅读代码人的只能猜测其表达的是微信帐号
const wechatAccount = 2839283; // good 语义明确， 微信帐号
```

4. 变量要命名长度最少 3 个字符；无论何种场景下禁止使用单字母，符号进行命名，示例:

```ts
const ary = [1, 2, 3, 4];
const result = ary.map(n => n * 3); // bad
const result = ary.map(item => item * 3); //good

const de = window.document; // bad
const doc = window.document; // good
```

5. 仅以下划线开头的命名方式表示私有变量，示例：

```ts
@Injectable()
export class ApiService {
  constructor(public _chartService: ChartService) {} // bad! 声明为public
}

@Injectable()
export class ApiService {
  constructor(private _http: HttpClient) {} // good！ 声明为private
}
```

6. 严禁一切形式的拼音命名方式，包括拼音简写，使用拼音首字母等。示例：

```ts
export class DataLsdBuildComponent {
  // 风险区域统计
  fxqyEchartsOption = null; // bad

  riskAreaEchartOptions = null; // good;

  fzgsType = 'zdfx'; // bad! What does the 'fzgs' meaning?
}
```

7. 在正确的位置使用正确的单、复数形式

```ts
export class DataLsdBuildComponent {
  // 风险区域统计
  riskAreaEchartsOption = null; // bad echarts 为复数，表示多个图表

  riskAreaEchartOptions = null; // good options 为复数，表示多个选项
}

const companys = 'companys'; // bad 错误的复数
const companies = 'companies'; // good 正确的复数
```

8. 所有的 subject 类型的字段或变量均以'\$'结尾

```ts
const userId: BehaviorSubject<number> = new BehaviorSubject<0>; // bad 变量

const userId$: BehaviorSubject<number> = new BehaviorSubject<0>; // good 变量

class AuthService {
  user: Subject<UserInfo> = new Subject<null>; // bad 字段名称
}

class AuthService {
  user$: Subject<UserInfo> = new Subject<null>; // good 字段名称
}

```

### 类型声明必须写并且原则上不能出现 any

所有变量、函数必须书写正确的类型参数，不写类型或 everything is any 的话我们为什么要使用 ts？

### 除 HTML，所有使用引号的地方优先使用单引号

只在必须使用双引号的地方使用双引号（如 JSON 字段等）。 HTML 中优先使用双引号。

## Component 规范

1. 一个 component.ts 中只应该存在一个 component, 禁止将多个 component 写在同一个 ts 文件下，更不允许 service，module 等混写在同一个 ts 文件中。
2. 禁止使用内联模板。
3. 禁止使用内联 css。
4. 变更检测策略优先使用 OnPush 策略。
5. 原则上禁止使用 ViewEncapsulation.None
6. 属性的 decorator 独占一行。
7. component 内代码顺序依次为：被 Input 修饰的属性，被 Output 修饰的属性，其它 decorator 修饰的属性，其它属性，constructor,钩子方法（按钩子所处的生命周期位置次序书写），组件的其它方法。
8. constructor 函数主体内除初始化代码外禁止书写其它逻辑，constructor 函数无需注释。
9. 在 OnInit 阶段调用 initialModel 和 launch 方法。组件需要通过 service 获取的数据均在 initialModel 方法中获取，组件需要通过 service 发出的数据均在 launch 方法中发出。各方法的次序根据调用次序列排列。
10. 组件中所有的方法和属性之间保留一个空行。

示例如下：

```ts
import { Component, OnDestroy, OnInit, Input, Output, EventEmitter, ViewChild } from '@angular/core';

import { Observable, of, Subject } from 'rxjs';
import { map, takeWhile } from 'rxjs/operators/';

@Component({
  selector: 'app-demo',
  templateUrl: './demo.component.html',
  styleUrls: ['./demo.component.scss'],
})
export class DemoComponent implements OnInit, OnDestroy {
  /**
   * 独占一行，被修饰的字段另起一行
   * order: 1
   */
  @Input()
  inputField: InputType;
                              // 此空行 仅用于提高可读性， 下同
  /**
   * 独占一行，被修饰的字段另起一行
   * order: 2
   */
  @Output()
  remove: EventEmitter<number> = new EventEmitter();

  /**
   * 独占一行，被修饰的字段另起一行
   * order: 3
   */
  @ViewChild()
  element: HTMLDivElement;

  /**
   * 4, 5都属于其它属性，可根据实际情况调整其顺序
   * order: 4
   */
  isBind: Observable<boolean>;

  /**
   * order: 5
   */
  isAlive = true;

  /**
   * constructor 的位置在属性这后
   * order: 6
   */
  constructor(private accountService: AccountService) {}

  /**
   * ngOnInit 紧跟在constructor 之后
   * order: 7
   *x
  ngOnInit() {
    this.initModel();
                    // 此空行表示一段逻辑结束
    this.launch();
  }

  /**
   * 在 ngOnInit中先调用 initModel 所以此方法在 launch 之前
   * order: 8
   */
  initialModel() {
    this.isBind = this.accountService.getGoogleAuthKey().pipe(map(res => !res.snskey));
  }

  /**
   * @ignore
   * order: 9
   */
  launch() {
    this.accountService.launchGetGoogleAuthKey(of(null));
                                          // 此空行表示一段逻辑结束 , 下同
    this.accountService.handleGoogleAuthKeyError(() => this.isAlive);

    this.accountService.launchUnbindSNS(this.unbind$.pipe(takeWhile(() => this.isAlive)));

    this.accountService.handleUnbindSNSError(() => this.isAlive);
  }

  /**
   * @ignore
   * order: 10
   */
  ngOnDestroy() {
    this.isAlive = false;
  }
}
```

## Service 规范

1. 除 guard.service.ts 外，一个 service.ts 文件内只应该存在一个 service，换名话说，一个 service.ts 文件只能存在一个使用 Injectable 修饰的 class。
2. 全局 service 优先使用向 Injectable 修饰器传入{ provideIn：'root' }的方式，而不是在根模块的 providers 字段中声明。其它模块内部使用的 service 在相应模块的 providers 字段中声明。
3. service.ts 文件内的 interface，type 等应该位于 service 之前，import 语句之后。
4. class 代码顺序规范：
   - class 代码顺序由上到下依次为：属性，constructor，方法。
   - service 中的代码量较大时应该添加适当的分隔注释，具有相同功能的代码应该添加在相同的区域内，例如对于 business service 其方法大致可以分为几类：api 交互的方法，处理数据逻辑的方法，处理错误的方法，其它工具方法等。
5. 属性与属性，属性与方法间必须保留一个空行，constructor 与 class 语句之间保留一个空行，注释与 class 语句之间没有空行。
6. 命名：发送 api 请求的方法以 get， delete, add, update 开头。Error handler 部分的方法必须 handle 开头，Error 结尾，即 handleXXXError。其它方法的命名参照命名规范部分。
7. 所有的方法和属性优先声明为 private，其次为 protected，如果属于 public 类型，请省略 public 类型声明（构造函数中的参数除外）。
8. contractor 内注入的其它服务必须声明为 private，命名必须以'\_'开头(通过继承需要传递给父类的情况除外，可以声明为 public，不以下划线开头),constructor 主体部分原则上不应该有逻辑，如果必须有逻辑的话请尽量保持简单。

示例：

```ts
import { MessageService } from '../service/message.service';

/**
 * interface 位于import语句之后
 */
interface RequestParams {
  // ...
}

export type UserType = 'vip' | 'superVip';

@Injectable({
  providedIn: 'root',
})
export class AuthService extends BaseService {
  /**
   * 与class语句间没有空行
   */
  private readonly githubConfigPath = 'github/config';

  /**
   * public 字段， 省略 'public’声明符
   * 命名以 $ 结尾
   */
  user$: BehaviorSubject<User> = new BehaviorSubject(null);

  constructor(
    private _http: HttpClient, // 命名以 _  开头
    private _error: ErrorService, // 命名以 _ 开头
    public messageService: MessageService, // 父类上需要使用的 service，声明为public类型，不以 _ 开头
    @Inject(PLATFORM_ID) private _platformId: Object // 命名以 _ 开头
  ) {
    super(messageService);
  }

  /* -------------------------------Api Request----------------------------------- */

  /**
   * @ignore
   */
  private getHiddenStatisticsByDepartment(): Observable<fromRes.ShareStrategyResponse> {
    // ...
  }

  /**
   * @ignore
   */
  private updateUserName(): Observable<RequestParams> {
    // ...
  }

  /* ------------------------------------Logic methods---------------------------- */

  /**
   * @ignore
   */
  private confirmStrategyShare(
    param: ShareStrategyStateSnapshot,
    component: any
  ): Observable<number | InnerShareFormModel> {
    // ...
  }

  /* -----------------------------------------Error handler----------------------- */

  /**
   * 处理相关错误的方法，以 handle 开头，Error 结尾
   */
  handleShareStrategyError(keepAlive: keepAliveFn): Subscription {
    // ...
  }
}
```

## css 规范

1. 编码使用 UTF-8，每个 SCSS 文件的第一行必须是定义编码的 @charset "UTF-8"。
2. 代码顺序：编码声明，import 语句，变量声明(包括 mixin，function 等)，宿主元素样式声明，其它元素样式声明，deep query 样式声明，media query 样式声明。（有时为了覆盖样式，需要将 import 语句放在其它位置，要灵活调整）

```scss
@charset 'UTF-8'; // order: 1

@import '...'; // order: 2
@import '...';

$space: 20px; // order: 3
$line-height: 32px;

$color: #00e1fe;
$bcg-color: #ffee00;

@mixin vertical-middle {
  // ...
}

/**
 * component 宿主元素样式 order: 4
 */
:host {
}

/**
 * component样式 order: 5
 */

/**
 * deep query 样式 order: 6
 */
:host::ng-deep {
}

/**
 * media query 样式 order: 7
 */
@media screen and (max-width: 1440px) {
  // ...
}
```

3. import 块与其它部分间均留一个空行，import 语句间没有空行。

```scss
@charset 'UTF-8';
// 空行
@import '...';
@import '...';
// 空行
$space: 20px;
```

4. 无论是变量名称还是 class 名称，统一使用一个中划线分隔的命名方式。

```scss
$verticalSpace: 20px; // bad
$vertical-space: 20px; // good

.leftHeader {
  // bad
}

.left--header {
  // bad
}

.left-header {
  // good
}
```

5. 属性书写顺序：
   - include
   - Positioning Model 布局方式、位置；相关属性包括：position / top / right / bottom / left / z-index / display / float / ...
   - Box model 盒模型；相关属性包括：width / height / padding / margin / border / overflow / ...
   - Typographic 文本排版；相关属性包括：font / line-height / text-align / word-wrap / ...
   - Visual 视觉外观；相关属性包括：color / background / list-style / transform / animation / transition / ...
   - 如果包含 content 属性，应放在最前面。

```scss
...
.left-header {
  @include vertical-middle(); // order: 1
  display: flex; // order: 2
  justify-content: center;
  width: 200px; // order: 3
  overflow-x: hidden;
  color: green; // order: 4
  transform: translateX(2px);
}
...
```

6. 类似 margin，padding，background， border-radius 之类的属性，应该优先使用其简写形式。如:

 ```css
 .left-header {
   margin: 0 2px 2px 0; // good
 }

 .left-header {
   margin-right: 2px; // bad
   margin-bottom: 2px;
 }
 ```

7. 颜色值统一使用小写，禁止使用 16 进制的简写形式。

```scss
.left-header {
  color: #0fe; // bad
  background: #FFEE22 // bad
}

.left-header {
  color: #00ffee; // good
  background: #ffee22; // good
}
```

8. 只在属性语句末尾使用 '//' 注释，前后留一个空格;其它情况一律使用 '/* */' 注释，内容与星号之间保留一个空格。

```scss
// bad use inline-comment
.left-header {
  color: red; /* bad use block-comment */
}

/**
 * good
 */
 .left-header {
   color: red: // good
 }
```

8. scss 结构应与 html 结构尽量保持一致，公共 class 单独书写。

```html
<div class="container">
  <div class="header"></div>
  <div class="content">
    <div class="content-left bcg-white"></div>
    <div class="content-right bcg-white"></div>
  </div>
</div>
```

```scss
.bcg-white {
  // ... 这是一个公共样式
}

.container {
  .header {
    // ...
  }
  .content {
    //...
    .content-left {
      //...
    }
    .content-right {
      // ...
    }
  }
}

```

9. 样式块之间保留一个空行。嵌套的样式块之间没有空行

```scss
.bcg-white {
  //...
}                     // bad 没有留空行
.container {
  .header {
    // ...
  }
                     // bad 多余的空行
  .content {
    //...
                     // bad 多余的空行
    .content-left {
      //...
    }
                     // bad 多余的空行
    .content-right {
      // ...
    }
  }
}
```

```scss
.bcg-white {
  //...
}
                      // good 样式块间留有空行
.container {          // good container 内部的所有代码块间都没有空行
  .header {
    // ...
  }
  .content {
    //...
    .content-left {
      //...
    }
    .content-right {
      // ...
    }
  }

```

10. 数值与单位：介于 0-1 之间的数值省略 0；属性值'0'后面不要加单位。

```scss
  .left-header {
    margin: 0px; // bad
    opacity: 0.5; // bad
  }

  .left-header {
    margin: 0; // good
    opacity: .5; // good
  }
```

11. 所有 meta 样式优先使用 tailwind 中已定义好的 class。所有与 flex 布局相关的优先使用@angular/flex-layout。

## 注释规范

1. 所有 ts，js 文件均采用 jsDoc 风格的注释。
2. 对块级代码的注释，函数参数注释及其它用于提高代码阅读性的注释均使用块级注释，行内注释以及对单行语句或变量的注释采用单行注释。
3. 无论是块级注释还是单行注释，注释与被注释内容之间不能有空行。用于提高代码可读性的注释应与前后上下文保留一个空行。行内注释 '//' 前后均保留一个空格。函数参数注释与被注释参数之间不能有空格。
4. html 注释独占一行，注释内容与'-'之间保留一个空格。
5. scss 注释参数 css 规范。
6. 不要滥用注释，只在必须时添加注释。语义明确，功能清晰或自释性强的代码无需添加注释。
7. 用于生成文档的注释在没 ts 要书写注释内容时使用'@ignore'替代。
8. 由于 ts 是强类型语言，本身就具有很强的自释性，在类型声明良好的情况下，无须再在注释中说明参数的类型，再配合合理的命名就可以很清楚的表达参数的类型及作用，因此形如 '@param {number} b‘ 这种注释在代码中可以省略。

示例：

```ts
export class Auth {
  /**
   * 是否管理员
   */
  isAdmin: boolean = true;

  constructor() {}

  /**
   * @description 这个注释除了解释函数外，还可以用来生成文档。
   * @returns { Observable<string> } 以Observable的形式返回用户的github地址。
   */
  getGithubAddress(path: string/* 注意没有空格 */, id: string): Observable<string> {
    const githubAddress = 'http://auth.github.com/xxx'; // 这是行内注释，也是一个多余的注释，因为从变量名称上很容易理解这是 github 的 url 地址；
    // 这是个一单行注释，与上下文间没有空行。
    const redirect = getUrl(path);

    return this._http.get<GithubAuthConfig>(this.completeApiUrl(this.path, this.githubConfigPath)).pipe(
      map(
        config =>
          `${this.githubAuthURI}?client_id=${config.clientId}&redirect_uri=${environment.githubAuthRedirectAddress +
            redirectPath}&scope=user&state=${config.state}`,
      ),
      catchError(this._error.handleHttpError),
    );
  }

  /* ------------------------------------------这个注释只是为了提高代码可读性，与上下文之间应保留一个空行--------------------------------------- */

  /**
   * @ignore
   */
  getUser() {
    ....
  }

}

/**
 * @function getName
 * @returns { string } 用户名称
 * @description 通过用户id获取用户名称
 * @deprecated v1.0.1 将会移除
 */
function getName(id: number): string {

}
```

```html
<ul>
  <!-- 这是一个html注释，单独占一行 -->
  <li></li>
  <li></li>
</ul>
```

```scss
/* 这是css中的注释 */
.name {
  display: inline-block; // 说明为什么需要这个属性
}
```
