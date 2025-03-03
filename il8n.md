---

## 多语言

Midway 提供了多语言组件，让业务可以快速指定不同的语言，展示不同的文案，也可以在 HTTP 场景配合请求参数，请求头等方式来使用。

相关信息：

| 描述              |    |
| ------------------- | ---- |
| 可用于标准项目    | ✅ |
| 可用于 Serverless | ✅ |
| 可用于一体化      | ✅ |
| 包含独立主框架    | ❌ |
| 包含独立日志      | ❌ |

## 安装组件

```bash
$ npm i @midwayjs/i18n@3 --save
```

或者在 `package.json` 中增加如下依赖后，重新安装。

```json
{
  "dependencies": {
    "@midwayjs/i18n": "^3.0.0",
    // ...
  },
}
```

## 使用组件

将 i18n 组件配置到代码中。

```typescript
import { Configuration } from '@midwayjs/core';
import * as i18n from '@midwayjs/i18n';

@Configuration({
  imports: [
    // ...
    i18n
  ]
})
export class MainConfiguration {
  //...
}
```

## 使用

组件提供了 `MidwayI18nService`  服务，用于翻译多语言文本。

使用 `translate` 方法，传入不同的文本关键字和参数，返回不同语言的文本内容。

```typescript
@Controller('/')
export class UserController {

  @Inject()
  i18nService: MidwayI18nService;

  @Get('/')
  async index(@Query('username') username: string) {
    return this.i18nService.translate('HELLO_MESSAGE', {
      args: {
        username
      },
    });
  }
}
```

## 配置多语言文案

你可以在配置文件中直接配置，但是大多数情况下，文案会很多，有时候甚至可能文案在远端服务上，这个时候直接配置就不太现实。

一般来说，我们会将文案单独放到某个文案配置目录中，比如 `src/locales` 。

以 `src/locale` 这个目录为例，我们举个例子，结构如下：

```text
.
├── src
│   ├── locales
|   │   ├── en_US.json
|   │   └── zh_CN.json
│   └── controller
│       └── home.controller.ts
├── package.json
└── tsconfig.json
```

这里我们建了两个多语言的文件，`en_US.json` 和 `zh_CN.json`，分别代表英文和中文。

文件内容分别如下：

```json
// src/locales/en_US.json
{
  "hello": "Hello {username}",
  "email": "email id",
  "login": "login account",
  "createdAt": "register date"
}
```

```json
// src/locales/zh_CN.json
{
  "hello": "你好 {username}",
  "email": "邮箱",
  "login": "帐号",
  "createdAt": "注册时间"
}
```

每行一个字符串对，是一个标准的 JSON 格式内容，也可以使用 js/ts 文件，花括号中是可替换的参数占位。

同时，需要在配置中加入这两个 JSON，其中 `default` 是语言的默认分组。

```typescript
// src/config/config.default.ts
export default {
  // ...
  i18n: {
    // 把你的翻译文本放到这里
    localeTable: {
      en_US: {
        default: require('../locale/en_US'),
      },
      zh_CN: {
        default: require('../locale/zh_CN'),
      }
    },
  }
}
```

这样就可以使用了，使用输出如下。

```typescript
this.i18nService.translate('hello', {
  args: {
    username: 'harry',
  },
  locale: 'en_US',
});

// output: Hello harry.

this.i18nService.translate('hello', {
  args: {
    username: 'harry',
  },
  locale: 'zh_CN',
});

// output: 你好 harry.

```

## 多语言文案分组

在如下配置中，用户配置的多语言文案在 `default` 分组中。

```typescript
// src/config/config.default.ts
export default {
  // ...
  i18n: {
    // 把你的翻译文本放到这里
    localeTable: {
      en_US: {
        default: require('../locale/en_US'),
      },
      zh_CN: {
        default: require('../locale/zh_CN'),
      }
    },
  }
}
```

这样做的好处是，在其他组件或者业务代码中，我们也可以使用不同的分组名，来添加其他的多语言文案。

比如：

```typescript
// src/config/config.default.ts
export default {
  // ...
  i18n: {
    // 把你的翻译文本放到这里
    localeTable: {
      en_US: {
        default: require('../locale/en_US'),
        user: require('../locale/user_en_US'),
      },
      zh_CN: {
        default: require('../locale/zh_CN'),
        user: require('../locale/user_zh_CN'),
      }
    },
  }
}
```

在代码中，如果调用非默认分组，需要指定分组参数。

```typescript
this.i18nService.translate('user.hello', {
  args: {
    username: 'harry',
  },
  group: 'user',        // 指定其他分组
  locale: 'en_US',
});

```

## 多语言文案格式

多语言文本中可以添加参数，参数可以有 `对象` 和 `数组` 两种形式。

对象形式如下，使用花括号作为占位符。

```text
Hello {username}
```

使用时，通过配置传递，按对象 key 覆盖变量。

```typescript
async index(@Query('username') username: string) {
  return this.i18nService.translate('hello', {
    args: {
      username
    },
  });
}
```

数组形式如下，使用数字作为占位符。

```text
Hello {0}
```

使用时，通过配置传递，格式是数组形式，按数组顺序覆盖数字变量。

```typescript
async index(@Query('username') username: string) {
  return this.i18nService.translate('hello', {
    args: [username]
  });
}
```

## 动态添加多语言文案

有时候，多语言文案可能放在远端，比如数据库等，我们可以通过 `addLocale` 方法进行动态添加。

比如，在配置加载后，代码使用前。

```typescript
// configuration.ts

// ...
@Configuration({
  imports: [
    koa,
    i18n
  ]
})
export class MainConfiguration {

  @Inject()
  i18nService: MidwayI18nService;

  async onReady() {
    this.i18nService.addLocale('zh_TW', {
      hello: '你好，{username} 美麗的世界'
    });
  }


  // ...
}
```

在代码中就可以使用。

```typescript
async index(@Query('username') username: string) {
  return this.i18nService.translate('hello', {
    args: [username],
    locale: 'zh_TW'
  });
}
```

## 通过参数指定当前语言

一般情况下，默认语言为 `en_US`，用户的浏览器访问一般会自带 `Accept-Language` 头，所以会正确识别语言。比如用中文浏览器访问，就能正常显示中文。

除此之外，在 HTTP 场景下可以通过 URL Query，Cookie，Header 来指定语言。

优先级从上到下：

* query: /?locale\=en-US
* cookie: locale\=zh-TW
* header: Accept-Language: zh-CN,zh;q\=0.5

当传递了这些参数之后，多语言数据会自动保存到当前用户的 Cookie 中，下次请求会直接用该设定好的语言。

## 手动设置语言

可以通过调用 `saveRequestLocale` 设置当前语言。

```typescript
async index() {
  // ...
  this.i18nService.saveRequestLocale('zh_CN');
}
```

如果开启了 `writeCookie` 配置，设置后会保存到当前用户的 Cookie 中，下次请求会使用该设置。

## 语言选择优先级

这些多种设置语言的方式，有着不同的优先级，如下优先级从高到低：

* 1、`i18nService.translate` 方法显式指定的语言
* 2、通过其他装饰器设置的语言，比如 `@Validate` 装饰器的参数（本质是调用了`i18nService.translate` 方法）
* 3、通过 `saveRequestLocale` API 直接设置的当前语言
* 4、通过浏览器 Query，Cookie，Header 设置的语言（本质是调用了 `saveRequestLocale`）
* 5、i18n 组件配置中的默认语言

## 关于语言大小写

在代码内部，我们会将所有的多语言，fallback 规则，写入的文本串，返回的 locale 结果，使用下面的规则替代

* 1、使用中划线代替下划线
* 2、使用小写代替大写

即所有的 `en_US` 都会变成 `en-us`，`zh_CN` 会变成 `zh-cn`。

这样做会安全的适配 URL 和 Cookie。

## View 中使用

在 Web 类型的框架中，我们默认添加了 locals 变量支持，可以在模板引擎中使用。

假设我们使用的模板引擎是 [Nunjucks](https://midwayjs.org/docs/extensions/render)，可以直接引用到 `i18n` 方法。

多语言文案如下：

```json
{
  "hello": "Hello {username}",
}
```

模板如下：

```html
<span>{{ i18n('hello', user) }}</span>
```

示例如下：

```typescript
// ...

@Controller('/')
export class UserController {

  @Inject()
  ctx: Context;

  @Get('/')
  async index() {
    await this.ctx.render('index', {
      // 注意这里是整个对象传递给模板
      user: {
        username: 'harry',
      }
    });
  }
}
```

i18n 方法定义如下：

```typescript
function i18n(templateName: string, args: Record<string, any>) {
  // ...
}
```

方法名可以通过配置修改。

```typescript
// src/config/config.default.ts
export default {
  // ...
  i18n: {
    localsField: 'i18n',
  }
}
```

## 配置

### 默认配置

大部分情况下，你只需要在配置 `localeTable` 添加你自己的多语言翻译即可。

下面是完整的配置，你可以在配置定义中找到。

```typescript
// src/config/config.default.ts
export default {
  // ...
  i18n: {
    // 默认语言  "en_US"
    defaultLocale: 'en_US',

    // 把你的翻译文本放到这里
    localeTable: {
      en_US: {
        // group name
        default: {
          // hello: 'hello'
        }
      },
      zh_CN: {
        // group name
        default: {
          // hello: '你好'
        }
      },
    },

    // 语言映射，可以用 * 号通配
    fallbacks: {
      //   'en_*': 'en_US',
      //   pt: 'pt-BR',
    },
    // 是否将请求参数写入 cookie
    writeCookie: true,
    resolver: {
      // url query 参数，默认是 "locale"
      queryField: 'locale',
      cookieField: {
        // Cookie 里的 key，默认是 "locale"
        fieldName: 'locale',
        // Cookie 域名，默认为空，代表当前域名有效
        cookieDomain: '',
        // Cookie 默认的过期时间，默认一年
        cookieMaxAge: FORMAT.MS.ONE_YEAR,
      },
    },
    localsField: 'i18n',
  }
}
```

### 回写 Cookie

默认情况下，多语言组件会将当前用户的语言回写到 Cookie 中，避免下次请求再进行查找以提高性能，我们可以通过配置关闭这个行为。

```typescript
// src/config/config.default.ts
export default {
  // ...
  i18n: {
    writeCookie: false,
  }
}
```

### 请求解析配置

HTTP 场景下，我们提供了通过参数指定当前语言的能力。

默认情况下，组件通过下面的字段来查找。

* query 的 `locale` 字段
* cookie 的 `locale` 字段
* header 的 `Accept-Language` 部分

我们可以通过配置修改查询的字段。

比如，修改 Query 的字段。

```typescript
// src/config/config.default.ts
export default {
  // ...
  i18n: {
    resolver: {
      queryField: 'abc'
    },
  }
}
```

我们就可以通过 `/?abc=en-US` 来请求修改语言。

如果不希望通过请求来设置语言，可以将整个 `resolver` 解析关闭，对 Cookie 的回写也将同时停止。

```typescript
// src/config/config.default.ts
export default {
  // ...
  i18n: {
    resolver: false,
  }
}
```

## 常用语言

| 语言             | 语言包名  |
| ------------------ | ----------- |
| 阿拉伯           | ar\_EG |
| 亞美尼亞         | hy\_AM |
| 保加利亚语       | bg\_BG |
| 加泰罗尼亚语     | ca\_ES |
| 捷克语           | cs\_CZ |
| 丹麦语           | da\_DK |
| 德语             | de\_DE |
| 希腊语           | el\_GR |
| 英语             | en\_GB |
| 英语（美式）     | en\_US |
| 西班牙语         | es\_ES |
| 爱沙尼亚语       | et\_EE |
| 波斯语           | fa\_IR |
| 芬兰语           | fi\_FI |
| 法语（比利时）   | fr\_BE |
| 法语             | fr\_FR |
| 希伯来语         | he\_IL |
| 印地语           | hi\_IN |
| 克罗地亚语       | hr\_HR |
| 匈牙利           | hu\_HU |
| 冰岛语           | is\_IS |
| 印度尼西亚语     | id\_ID |
| 意大利语         | it\_IT |
| 日语             | ja\_JP |
| 格鲁吉亚语       | ka\_GE |
| 卡纳达语         | kn\_IN |
| 韩语/朝鲜语      | ko\_KR |
| 库尔德语         | ku\_IQ |
| 拉脱维亚语       | lv\_LV |
| 马来语           | ms\_MY |
| 蒙古语           | mn\_MN |
| 挪威             | nb\_NO |
| 尼泊尔语         | ne\_NP |
| 荷兰语（比利时） | nl\_BE |
| 荷兰语           | nl\_NL |
| 波兰语           | pl\_PL |
| 葡萄牙语(巴西)   | pt\_BR |
| 葡萄牙语         | pt\_PT |
| 斯洛伐克语       | sk\_SK |
| 塞尔维亚         | sr\_RS |
| 斯洛文尼亚       | sl\_SI |
| 瑞典语           | sv\_SE |
| 泰米尔语         | ta\_IN |
| 泰语             | th\_TH |
| 土耳其语         | tr\_TR |
| 罗马尼亚语       | ro\_RO |
| 俄罗斯语         | ru\_RU |
| 乌克兰语         | uk\_UA |
| 越南语           | vi\_VN |
| 简体中文         | zh\_CN |
| 繁体中文         | zh\_TW |

## 常见问题

### 1、测试配置全局语言不生效

一般场景下，你 **无需** 配置全局语言，因为浏览器访问会自动带上语言信息，比如中文浏览器自动返回中文，英文浏览器自动返回英文。

假如你明确希望测试全局设置的效果，请务必按下面操作：

* 1、如果使用的是浏览器，请清空页面 cookie 再访问，因为 cookie 中会记录上一次用户的语言信息
* 2、如果使用的是 Postman 等工具，请不要带上 cookie 以及语言相关的 Header，Query 等字段

### 2、测试返回的语言非预期

请使用浏览器进行测试，不要使用 Postman。

由于 Postman 请求不会带有浏览器语言相关的 Header，所以服务端无法自动判断语言。

如果你一定要使用 Postman，请参考浏览器请求，加上 `Accept-Language` Header。
