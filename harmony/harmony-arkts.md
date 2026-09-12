# HarmonyOS ArkTS 语言开发规范

## 适用范围

本规范负责 ArkTS 类型、语法、命名、对象建模和函数写法。页面、组件和状态管理读取 [harmony-arkui.md](harmony-arkui.md)。

## 类型安全

- 业务代码禁止使用 `any` 和 `unknown` 逃避类型检查。
- 所有成员、参数和返回值声明明确类型；复杂局部值也应补充类型。
- `JSON.parse()` 的结果必须断言为已定义的 `interface`、`class` 或模型类型。
- 对象字面量必须有明确的上下文类型，例如已声明的 `interface`、SDK 类型或 `Record<string, Object>`。
- 禁止使用匿名对象类型作为业务类型，例如 `{ name: string }`；先声明命名类型。
- 不使用 `eval()`、`with` 或 `delete` 动态改变对象结构。

```typescript
interface ApiResponse {
  code: number
  data: string
}

const result: ApiResponse = JSON.parse(content) as ApiResponse
```

## ArkTS 语法限制

- 禁止解构声明，使用命名类型和点语法读取字段。
- 箭头函数显式声明参数类型和返回类型。
- `ForEach`、`map`、`filter`、监听器和异步回调中的函数类型都要明确。
- 不依赖 TypeScript 专属或当前 ArkTS 版本不支持的动态语法。

```typescript
interface YearMonthResult {
  year: string
  month: string
}

const dateInfo: YearMonthResult = this.getYearMonth()
const year: string = dateInfo.year
const month: string = dateInfo.month

const ids: number[] = list.map((item: DataModel): number => item.id)
```

## 对象与路由参数

- 页面跳转参数统一命名为 `param`。
- 路由参数统一使用对象字面量，并通过 `as Record<string, Object>` 明确类型。
- 不使用 `new Object()` 后逐项赋值；是否将参数结构抽取到 `model/XxxModel.ets`，由项目根据实际复用范围和业务职责自行判断。

```typescript
const param = {
  'id': id,
  'title': title
} as Record<string, Object>
```

## 空值处理

- 根据真实数据来源选择 `undefined`、`null` 或明确默认值，不同时引入多套空值状态。
- 可选链和空值合并用于确实可能为空的数据。
- 项目内部已保证存在的数据不重复判空；系统 API、网络、文件、数据库和跨模块参数需要按真实返回类型处理。
- 不使用非空断言绕过生命周期或外部数据检查。

## 异步函数

- 异步函数明确返回 `Promise<T>`。
- 网络、文件、数据库和系统能力调用捕获实际可能发生的异常。
- 不用大范围 `try-catch` 包裹无关同步逻辑。
- `finally` 只恢复 Loading、临时资源等无论成功失败都必须处理的状态。
- 捕获异常后使用 `VtbLogger` 记录，不返回伪造成功结果。

## 命名规范

| 类型 | 规范 | 示例 |
|---|---|---|
| 类、结构体、枚举、命名空间 | `UpperCamelCase` | `UserModel`、`PageState` |
| 变量、方法、参数 | `lowerCamelCase` | `userName`、`load()` |
| 常量、枚举值 | `UPPER_SNAKE_CASE` | `MAX_COUNT`、`TEXT` |
| 布尔值 | `is`、`has`、`can`、`should` 等前缀 | `isLoading` |

- 短生命周期局部值允许使用 `item`、`list`、`data`、`result`、`info`、`param`。
- 当前文件已经表达业务时，私有方法只表达动作；存在多个相似动作或跨文件公开时补充业务语义。
- 保留项目广泛使用的历史命名，不为形式统一批量重命名。

## 模型与枚举

- 需要抽取模型时统一放在 `model/XxxModel.ets`；是否抽取由项目根据实际复用范围和业务职责自行判断。
- 工具类私有数据结构可以就近定义。
- 需要持久化、网络传输或跨页面传递的枚举优先使用字符串枚举。

```typescript
export enum HistoryType {
  TEXT = 'TEXT',
  WEBSITE = 'WEBSITE',
  IMAGE = 'IMAGE'
}
```

## 检查清单

- [ ] 没有 `any`、业务层 `unknown` 或匿名对象类型。
- [ ] 参数、返回值和回调类型明确。
- [ ] 没有解构声明和不受支持的动态语法。
- [ ] `JSON.parse()` 已恢复为明确模型类型。
- [ ] 外部数据进行了必要校验，内部流程没有重复防御。
