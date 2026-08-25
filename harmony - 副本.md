# API 文档检查（强制自动执行）

> **文档优先**：在编写或修改 HarmonyOS ArkTS 代码前，必须检查 API 用法，不能仅凭历史知识。

## 本地文档路径（优先使用，更快）
```
D:\software\Huawei\DevEco Studio\plugins\openharmony\ohos-info-center-view\static\hos\JsEtsAPIReference
```
- 使用 `grep_search` 搜索 API 名称
- 使用 `view_file` 查看匹配的文档

## 在线文档（最新）
- 华为开发者文档：https://developer.huawei.com/consumer/cn/doc/
- 状态管理 V2：https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkts-new-provider-and-consumer
- 导航 API：https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkts-navigation-navigation

## 重点检查项
- 状态管理 V2：@ComponentV2、@Local、@Provider/@Consumer（必须指定别名并提供默认值）
- 导航 API：主框架内部页面跳转优先使用 pathStack.pushPath({ name, param })；需要 onPop 回调时再使用 pushPathByName(name, param, onPop)
- 数组更新：@Local 数组需重新赋值触发更新（使用 filter/map 创建新数组）
- px2vp 转换：使用 `this.getUIContext().px2vp()` 替代全局函数

## 常见问题速查
| 问题 | 解决方案 |
|------|----------|
| Provider/Consumer 不生效 | 确保使用相同别名：`@Provider('key')` 和 `@Consumer('key')`，V2 必须提供默认值 |
| onPop 回调不触发 | 仅需要 onPop 回调时使用 pushPathByName，onPop 作为第三个参数；普通跳转使用 pathStack.pushPath({ name, param })
| UI 不刷新 | @Local 数组用 filter/map 创建新数组，嵌套对象用 @ObservedV2 和 @Trace |
| @Builder 无法响应 @Local 变化 | 不要把依赖 @Local 状态变化的交互控件写成 @Builder，改为 @ComponentV2 子组件，并用 @Param/@Event 传递数据和事件 |
| getContext(this) 废弃 | ❌ `getContext(this)` → ✅ `this.getUIContext().getHostContext() as common.UIAbilityContext` |
| promptAction.showToast 废弃 | ❌ `promptAction.showToast()` → ✅ `this.getUIContext().getPromptAction().showToast()` |
| 解构声明报错 | ArkTS 不支持解构：❌ `const { a, b } = obj` → ✅ `const result = obj; result.a; result.b` |
| 对象字面量类型报错 | 必须先定义 interface，不能直接用匿名对象类型：`{ a: string }` |
| 箭头函数返回类型报错 | 必须显式声明返回类型：❌ `(item) => item.id` → ✅ `(item: Model): number => item.id` |
| ForEach 缺少 keyGenerator | 必须提供 keyGenerator：❌ `ForEach(list, (item) => {})` → ✅ `ForEach(list, (item): void => {}, (item): string => item.id)` |

# HarmonyOS ArkTS 开发规范

## 项目概述

这是一个 HarmonyOS NEXT 项目，使用 ArkTS 语言和 ArkUI 声明式 UI 框架开发。

## ArkTS 类型约束（强制）

> **官方文档**: [从TypeScript到ArkTS的适配规则](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/typescript-to-arkts-migration-guide-0000001820879585)

ArkTS 是静态强类型语言，比 TypeScript 更严格。**必须遵守以下规则**：

### 禁止 any 和 unknown 类型 (`arkts-no-any-unknown`)

```typescript
// ❌ 错误：JSON.parse 返回 any
const result = JSON.parse(str)

// ✅ 正确：定义接口并使用类型断言
interface ApiResponse { code: number; data: string }
const result: ApiResponse = JSON.parse(str) as ApiResponse
```

### 对象字面量必须有类型声明 (`arkts-no-untyped-obj-literals`)

```typescript
// ❌ 错误：无类型对象字面量
this.pathStack?.pop({ fileName: 'test', filePath: '/path' })

// ✅ 正确：先定义接口
interface FileResult { fileName: string; filePath: string }
const result: FileResult = { fileName: 'test', filePath: '/path' }
this.pathStack?.pop(result)
```

### 禁止解构声明 (`arkts-no-destruct-decls`)

```typescript
// ❌ 错误：解构声明
const { year, month } = this.getYearMonth()

// ✅ 正确：使用接口和点语法
interface YearMonthResult { year: string; month: string }
const dateInfo: YearMonthResult = this.getYearMonth()
const year = dateInfo.year
const month = dateInfo.month
```

### 箭头函数必须显式声明类型 (`arkts-no-implicit-return-types`)

```typescript
// ❌ 错误：隐式返回类型
const weekMap = this.aggregateByKey(list, item => item.iToast d)

// ✅ 正确：显式声明参数和返回类型
const weekMap = this.aggregateByKey(list, (item: FlowingWaterModel): number => item.id)
```

### 废弃 API 替换

```typescript
// ❌ 废弃：getContext(this)
const context = getContext(this)

// ✅ 推荐：使用新 API
import common from '@ohos.app.ability.common'
const context = this.getUIContext().getHostContext() as common.UIAbilityContext
```

### 其他重要约束

| 规则 | 说明 |
|------|------|
| 禁止 `eval()` | 不能使用动态代码执行 |
| 禁止 `with` 语句 | 不支持 with 语法 |
| 禁止 `delete` 删除属性 | 不能动态删除对象属性 |
| 所有变量必须声明类型 | 不能依赖类型推断为 any |

## 命名规范

### 标识符命名

| 类型 | 规范 | 示例 |
|------|------|------|
| 类名、枚举名、命名空间 | `UpperCamelCase` | `UserManager`, `PageState` |
| 变量名、方法名、参数名 | `lowerCamelCase` | `userName`, `loadData()` |
| 常量名、枚举值 | `UPPER_SNAKE_CASE` | `MAX_COUNT`, `STATUS_SUCCESS` |
| 布尔型变量 | 加 `is`/`has`/`can`/`should` 前缀 | `isLoading`, `hasPermission` |

### 文件命名

- **页面文件**：`XxxPage.ets`（如 `HomePage.ets`、`LoginPage.ets`）
- **组件文件**：`XxxComponent.ets` 或直接 `Xxx.ets`（如 `HeaderComponent.ets`）
- **工具类**：`VtbXxxUtil.ets`（如 `VtbPreferenceUtil.ets`、`VtbLogger.ets`）
- **模型/实体**：`XxxModel.ets`（如 `UserModel.ets`）
- **服务类**：`XxxService.ets`（如 `HttpService.ets`）
- **常量类**：`XxxConstants.ets`（如 `CommonConstants.ets`、`RouterConstants.ets`）
- **数据源类**：`XxxDataSource.ets`（如 `LazyDataSource.ets`）

## 项目结构（多模块架构）

```
├── AppScope/                    # 应用全局配置
│   ├── app.json5               # 应用配置（名称、版本、权限）
│   └── resources/              # 全局资源
├── entry/                       # 主模块（HAP）
│   └── src/main/
│       ├── ets/
│       │   ├── entryability/   # 应用入口 Ability
│       │   ├── abilityStage/   # AbilityStage
│       │   ├── pages/          # 页面文件（按功能模块分子目录）
│       │   ├── models/         # 数据模型
│       │   ├── view/           # 自定义组件
│       │   ├── common/         # 模块级常量、路由定义
│       │   └── utils/          # 模块级工具类
│       ├── resources/          # 模块资源
│       └── module.json5        # 模块配置
├── oh_modules/                  # 第三方依赖（含 @vhm/basecommon HAR 包）
├── build-profile.json5          # 编译配置
└── oh-package.json5             # 依赖管理
```

## 组件开发规范

### 页面组件结构（使用 @ComponentV2）

```typescript
import CommonConstants from '@vhm/basecommon/src/main/ets/common/CommonConstants'

@Entry({ routeName: 'XxxPage' })  // 定义路由名称
@ComponentV2
export struct XxxPage {
  // 1. 状态变量声明（使用 @Local 替代 @State）
  @Local isLoading: boolean = false
  @Local dataList: string[] = []
  
  // 2. 生命周期方法
  aboutToAppear(): void {
    this.loadData()
  }
  
  onPageShow(): void {
    // 页面显示时调用
  }
  
  // 3. 业务方法
  private loadData(): void {
    // 加载数据逻辑
  }
  
  // 4. UI 构建方法（必须实现）
  build() {
    Column() {
      // UI 结构
    }
    .padding({ top: this.getUIContext().px2vp(globalThis.statusHeight) })
    .width(CommonConstants.COMPONENT_PROPORTION_100)
    .height(CommonConstants.COMPONENT_PROPORTION_100)
  }
}
```

### 子组件结构

```typescript
@ComponentV2
export struct ItemCard {
  // 使用 @Param 接收外部传入的属性（替代 @Prop）
  @Param title: string = ''
  @Param content: string = ''
  
  // 回调事件
  onItemClick?: () => void
  
  build() {
    Column() {
      Text(this.title)
        .fontSize($r('app.float.fp_16'))
        .fontWeight(CommonConstants.FONT_WEIGHT_BOLD)
      Text(this.content)
        .fontSize($r('app.float.fp_14'))
    }
    .onClick(() => {
      this.onItemClick?.()
    })
  }
}
```

### 导航与路由

```typescript
// 父组件提供导航栈
@Entry
@ComponentV2
struct MainPage {
  @Provider() pathStack: NavPathStack = new NavPathStack()
  
  build() {
    Navigation(this.pathStack) {
      // 页面内容
    }
    .hideTitleBar(true)
    .mode(NavigationMode.Stack)
  }
}

// 子组件消费导航栈
@ComponentV2
export struct ChildPage {
  @Consumer() pathStack: NavPathStack | undefined = undefined
  
  private navigateToDetail(): void {
    this.pathStack?.pushPathByName('DetailPage', { id: 123 }, (popInfo: PopInfo) => {
      // 处理返回结果
      let result = popInfo.result as string
    })
  }
}
```

### 声明式路由规范（推荐）

> 使用 `route_map.json` 声明式配置路由，避免在 MainPage 中手写 if-else 逻辑。

**1. module.json5 中声明路由文件**
```json5
"routerMap": "$profile:route_map"
```

**2. route_map.json 中配置路由映射**
```json
{
  "routerMap": [
    {
      "name": "模块名://页面名",
      "pageSourceFile": "src/main/ets/pages/xxx/XxxPage.ets",
      "buildFunction": "XxxPageBuilder"
    }
  ]
}
```

**3. 页面中导出 Builder 函数**
```typescript
@Builder
export function XxxPageBuilder() {
  XxxPage()
}

@ComponentV2
export struct XxxPage {
  // 页面内容
}
```

**路由名称规范**

格式：`{模块名}://{页面名}`

| 模块 | 示例 |
|------|------|
| main | `main://CalendarPage` |
| record | `record://AddRecordPage` |
| movie | `movie://MovieDetailPage` |

**注意事项**
- 无需在 MainPage 中手动 import 路由页面
- 无需在 MainPage 中编写 routerMap if-else 逻辑
- 新增页面只需：① 创建页面并导出 Builder ② 在 route_map.json 添加配置

### 页面导航栏规范

> 仅当页面顶部设计为**标准布局（返回键 + 标题）**时，使用 `BaseTitleBar_V2` 作为系统导航栏。
> 
> **不适用场景**：
> - 顶部有 Tab 切换
> - 其他自定义布局
> 
> **注**：`BaseTitleBar_V2` 支持右侧按钮（图片/文字），通过 `menuShow`/`titleRightShow` 属性配置。

```typescript
import { BaseTitleBar_V2 } from '../view/BaseTitleBar_V2'

build() {
  NavDestination() {
    Column() {
      Column() {
          BaseTitleBar_V2({
            titleBarAttribute: {
              backShow: true,
              backImg: $r('app.media.ic_back'),
              title: this.pageTitle,
              backCallback: (): void => {
                this.pathStack?.pop()
              }
            }
          })
        }
        .padding({
          top: this.getUIContext().px2vp(globalThis.statusHeight)
        })
      
      // 页面内容
    }
  }
  .hideTitleBar(true)
}
```

## 状态管理装饰器（V2 版本）

| 装饰器 | 用途 | 说明 |
|--------|------|------|
| `@Local` | 组件内部状态 | 替代 V1 的 `@State`，状态变化触发 UI 刷新 |
| `@Param` | 父→子单向传递 | 替代 V1 的 `@Prop`，子组件不可修改 |
| `@Provider` | 祖先组件提供数据 | 跨层级向下传递数据 |
| `@Consumer` | 后代组件消费数据 | 接收祖先组件通过 `@Provider` 提供的数据 |

### 使用示例

```typescript
// 父组件
@Entry
@ComponentV2
struct ParentPage {
  @Local count: number = 0
  @Provider() sharedData: string = 'Hello'
  
  build() {
    Column() {
      Text(`计数: ${this.count}`)
      ChildComponent({ count: this.count })
      Button('增加').onClick(() => this.count++)
    }
  }
}

// 子组件
@ComponentV2
struct ChildComponent {
  @Param count: number = 0
  @Consumer() sharedData: string = ''
  
  build() {
    Column() {
      Text(`接收的计数: ${this.count}`)
      Text(`共享数据: ${this.sharedData}`)
    }
  }
}
```

## 常用装饰器

| 装饰器 | 用途 |
|--------|------|
| `@Builder` | 轻量级 UI 复用，抽取可复用的 UI 片段 |
| `@Styles` | 复用通用样式 |
| `@Extend` | 扩展原生组件样式 |
| `@Observed` | 标记可观察类（用于 LazyDataSource 等） |

### @Builder 使用

> 注意：@Builder 适合静态 UI 片段复用，不适合承载需要观察 @Local 变化的交互控件。若控件点击后需要根据 @Local 更新选中态、样式或内容，应抽成 @ComponentV2 子组件，通过 @Param 接收状态，通过 @Event 回传事件。

```typescript
@ComponentV2
struct DemoPage {
  @Local currentIndex: number = 0
  
  @Builder
  TabBuilder(title: string, index: number, selectedImg: Resource, normalImg: Resource) {
    Column() {
      Image(this.currentIndex === index ? selectedImg : normalImg)
        .width($r('app.float.vp_25'))
        .height($r('app.float.vp_25'))
      Text(title)
        .margin({ top: $r('app.float.vp_4') })
        .fontSize($r('app.float.fp_10'))
        .fontColor(this.currentIndex === index 
          ? $r('app.color.mainPage_selected') 
          : $r('app.color.mainPage_normal'))
    }
    .justifyContent(FlexAlign.Center)
    .onClick(() => this.currentIndex = index)
  }
  
  build() {
    Column() {
      this.TabBuilder('首页', 0, $r('app.media.tab_01'), $r('app.media.tab_02'))
      this.TabBuilder('我的', 1, $r('app.media.tab_my_01'), $r('app.media.tab_my_02'))
    }
  }
}
```

## 工具类开发规范

### 单例模式导出

```typescript
import preferences from '@ohos.data.preferences'

const STORE_NAME = 'myStore'
let preference: preferences.Preferences

class VtbPreferenceUtil {
  // 同步初始化
  getPreferencesOnSync(context: Context): void {
    try {
      let options: preferences.Options = { name: STORE_NAME }
      preference = preferences.getPreferencesSync(context, options)
    } catch (err) {
      console.error(`error: ${err}`)
    }
  }

  // 同步写入
  putPreferenceOnSync(context: Context, key: string, value: preferences.ValueType): void {
    if (!key) return
    if (!preference) this.getPreferencesOnSync(context)
    try {
      preference.putSync(key, value)
      preference.flushSync()
    } catch (err) {
      console.error('TAG', `Failed to put value, Cause: ${err}`)
    }
  }

  // 同步读取
  getPreferenceOnSync(context: Context, key: string, defValue: preferences.ValueType) {
    if (!key) return defValue
    if (!preference) this.getPreferencesOnSync(context)
    try {
      return preference.getSync(key, defValue)
    } catch (err) {
      console.error('TAG', `Failed to get value, Cause: ${err}`)
    }
    return defValue
  }
}

// 单例导出
export default new VtbPreferenceUtil()
```

### 使用方式

```typescript
import VtbPreferenceUtil from '@vhm/basecommon/src/main/ets/utils/VtbPreferenceUtil'

// 读取
let value = VtbPreferenceUtil.getPreferenceOnSync(context, 'key', 'defaultValue')

// 写入
VtbPreferenceUtil.putPreferenceOnSync(context, 'key', 'value')
```

## 常量类开发规范

```typescript
export default class CommonConstants {
  // 比例常量
  static readonly COMPONENT_PROPORTION_100: string = '100%'
  static readonly COMPONENT_PROPORTION_95: string = '95%'
  static readonly COMPONENT_PROPORTION_90: string = '90%'
  
  // 字体权重
  static readonly FONT_WEIGHT_LIGHT: number = 400
  static readonly FONT_WEIGHT_NORMAL: number = 500
  static readonly FONT_WEIGHT_BOLD: number = 700
  
  // Tab 索引
  static readonly HOME_TAB_INDEX_00: number = 0
  static readonly HOME_TAB_INDEX_01: number = 1
  static readonly HOME_TAB_INDEX_02: number = 2
  static readonly HOME_TAB_INDEX_03: number = 3
  
  // 布局常量
  static readonly BORDER_RADIUS_12: number = 12
  static readonly DIALOG_BORDER_RADIUS: number = 16
}
```

## 数据源开发规范（LazyDataSource）

```typescript
import { ObservedArray } from './ObservedArray'

class BasicDataSource<T> implements IDataSource {
  private listeners: DataChangeListener[] = []

  public totalCount(): number { return 0 }
  public getData(index: number): T | undefined { return undefined }

  registerDataChangeListener(listener: DataChangeListener): void {
    if (this.listeners.indexOf(listener) < 0) {
      this.listeners.push(listener)
    }
  }
  
  unregisterDataChangeListener(listener: DataChangeListener): void {
    const pos = this.listeners.indexOf(listener)
    if (pos >= 0) this.listeners.splice(pos, 1)
  }
  
  notifyDataReload(): void {
    this.listeners.forEach(listener => listener.onDataReloaded())
  }
  
  notifyDataAdd(index: number): void {
    this.listeners.forEach(listener => listener.onDataAdd(index))
  }
}

@Observed
export default class LazyDataSource<T> extends BasicDataSource<T> {
  dataArray: T[] = []

  public totalCount(): number { return this.dataArray.length }
  public getData(index: number): T { return this.dataArray[index] }
  
  public pushData(data: T): void {
    this.dataArray.push(data)
    this.notifyDataAdd(this.dataArray.length - 1)
  }
  
  public pushArrayData(newData: ObservedArray<T>): void {
    this.clear()
    this.dataArray.push(...newData)
    this.notifyDataReload()
  }
  
  public clear(): void {
    this.dataArray.splice(0, this.dataArray.length)
  }
  
  public isEmpty(): boolean {
    return this.dataArray.length === 0
  }
}
```

## 公共库统一导出规范（Index.ets）

```typescript
// baseCommon/Index.ets

// 页面导出
export { UserPrivacyOrAgreementPage } from './src/main/ets/pages/feedback/UserPrivacyOrAgreementPage'
export { FeedbackPage } from './src/main/ets/pages/feedback/FeedbackPage'
export { AboutUsPage } from './src/main/ets/pages/feedback/AboutUsPage'

// 组件导出
export { PermissionsDialog } from './src/main/ets/view/dialog/PermissionsDialog'
export { BaseTitleBar } from './src/main/ets/view/BaseTitleBar'
export { BaseTitleBarV2 } from './src/main/ets/view/BaseTitleBarV2'

// 动态加载函数
export function harInit(pageName: string) {
  switch (pageName) {
    // case RouterInfo.login_LoginPage.pageName:
    //   import('./src/main/ets/pages/LoginPage')
    //   break
  }
}
```

## 路由常量定义规范

```typescript
// 路由协议格式：{moduleName}://{PageName}
export class BuilderNameConstants {
  static readonly HAR_ABC: string = 'crop://ABCPage'
  static readonly TEST_ABC: string = 'test://ABCTestPage'
}
```

## 资源引用规范

```typescript
// ✅ 正确：使用资源文件
.fontSize(18)
.fontColor($r('app.color.black_333'))
.width($r('app.float.vp_25'))
.margin({ left: $r('app.float.vp_16') })

// ❌ 避免：硬编码数值（除非是特殊情况）
.fontColor('#333333')
.width(25)
```

## 沉浸式状态栏处理

```typescript
// EntryAbility.ts 中配置
onWindowStageCreate(windowStage: window.WindowStage) {
  let mainWindow = windowStage.getMainWindowSync()
  mainWindow.setWindowSystemBarEnable(['status'])
  mainWindow.setWindowSystemBarProperties({ statusBarColor: '#00000000' })
  mainWindow.setWindowLayoutFullScreen(true)
  
  // 获取安全区域高度
  windowStage.getMainWindow((err, data) => {
    if (!err.code) {
      globalThis.windowClass = data
      globalThis.statusHeight = data.getWindowAvoidArea(window.AvoidAreaType.TYPE_SYSTEM).topRect.height
      globalThis.bottomAvoidAreaHeight = data.getWindowAvoidArea(
        window.AvoidAreaType.TYPE_NAVIGATION_INDICATOR
      ).bottomRect.height
    }
  })
}

// 页面中使用
Column() {
  // 内容
}
.padding({ 
  top: this.getUIContext().px2vp(globalThis.statusHeight),
  bottom: this.getUIContext().px2vp(globalThis.bottomAvoidAreaHeight)
})
```

## 代码风格

- **缩进**：使用 2 个空格
- **每行一个语句**：每个语句单独一行
- **属性链式调用**：每个属性方法换行，提高可读性
- **类型声明**：明确声明所有变量类型，避免 `any`

```typescript
// ✅ 推荐：链式调用换行
Text('内容')
  .fontSize($r('app.float.fp_16'))
  .fontColor($r('app.color.black_333'))
  .margin({ top: $r('app.float.vp_8'), bottom: $r('app.float.vp_8') })

// ❌ 不推荐：挤在一行
Text('内容').fontSize(16).fontColor('#333333').margin({ top: 8 })
```

## 空值安全

```typescript
// 可选类型使用 ?
let userName: string | undefined = undefined

// 可选链操作符
let length = userName?.length

// 空值合并
let displayName = userName ?? '未知用户'

// 回调安全调用
this.onItemClick?.()
```

## @vhm/basecommon 公共库速查表

### 工具类

| 名称 | 导入方式 | 用途 | 方法 |
|------|----------|------|------|
| `VtbLogger` | `import VtbLogger from '@vhm/basecommon/src/main/ets/utils/VtbLogger'` | **统一日志工具**，封装 hilog | `debug/info/warn/error(tag, ...args)` |
| `VtbPreferenceUtil` | `import VtbPreferenceUtil from '@vhm/basecommon/src/main/ets/utils/VtbPreferenceUtil'` | **轻量级 key-value 本地存储**（等同安卓 SharedPreferences） | `putPreference`、`getPreference`、`getPreferenceBoolean`、`putPreferenceOnSync`、`getPreferenceOnSync` |
| `VtbPermissionUtil` | `import { VtbPermissionUtil } from '@vhm/basecommon/src/main/ets/utils/VtbPermissionUtil'` | **运行时权限申请**，检查+申请一步完成 | `checkPermissions`、`checkRequestPermissions`、`requestPermissions`、`requestPermissionsList` |
| `VtbBaseStringUtil` | `import VtbBaseStringUtil from '@vhm/basecommon/src/main/ets/utils/VtbBaseStringUtil'` | 字符串工具，判空等 | `stringIsEmpty(str)` |

### 组件

| 名称 | 导入方式 | 用途 |
|------|----------|------|
| `BaseTitleBarV2` | `import { BaseTitleBarV2 } from '@vhm/basecommon'` | **页面标准导航栏**（返回键 + 标题），支持右侧按钮 |
| `PermissionsDialog` | `import { PermissionsDialog } from '@vhm/basecommon/src/main/ets/view/dialog/PermissionsDialog'` | **隐私协议弹窗**，App 首次启动时弹出，包含同意/拒绝/查看隐私政策/使用条款四个回调 |

### 数据源

| 名称 | 导入方式 | 用途 |
|------|----------|------|
| `LazyDataSource` | `import LazyDataSource from '@vhm/basecommon/src/main/ets/datasource/LazyDataSource'` | V1 版懒加载数据源（配合 `LazyForEach`） |
| `LazyDataSourceV2` | `import LazyDataSourceV2 from '@vhm/basecommon/src/main/ets/datasource/LazyDataSourceV2'` | V2 版懒加载数据源 |

### 常量与配置

| 名称 | 导入方式 | 用途 |
|------|----------|------|
| `CommonConstants` | `import CommonConstants from '@vhm/basecommon/src/main/ets/common/CommonConstants'` | 公共常量（宽高比例、字体权重等） |
| `AppConfigInfo` | `import AppConfigInfo from '@vhm/basecommon/src/main/ets/common/AppConfigInfo'` | 应用配置信息 key（APP_NAME、VERSION_CODE 等），配合 `AppStorage` 使用 |

## 日志规范（VtbLogger）

**统一使用 `VtbLogger`**，不要直接使用 `hilog` 或 `console.log`。

```typescript
import VtbLogger from '@vhm/basecommon/src/main/ets/utils/VtbLogger'
```

### 日志级别

| 级别 | 方法 | 用途 | 示例 |
|------|------|------|------|
| 调试 | `VtbLogger.debug` | 开发调试信息 | `VtbLogger.debug(TAG, '变量值: ' + value)` |
| 信息 | `VtbLogger.info` | 关键业务节点 | `VtbLogger.info(TAG, '二维码生成成功')` |
| 警告 | `VtbLogger.warn` | 非致命但需关注 | `VtbLogger.warn(TAG, '未初始化，跳过操作')` |
| 错误 | `VtbLogger.error` | 异常/失败 | `VtbLogger.error(TAG, `操作失败: ${JSON.stringify(error)}`)` |

### 使用规范

```typescript
const TAG = 'ImageUtil'  // TAG = 当前类名或模块名

// ✅ 正确：使用 VtbLogger
VtbLogger.error(TAG, `保存失败: ${JSON.stringify(error)}`)

// ❌ 错误：直接使用 hilog
hilog.error(0x0001, TAG, `保存失败: ${JSON.stringify(error)}`)

// ❌ 错误：使用 console.log
console.log('保存失败')
```

## 异步编程规范

### async/await 标准模板

所有异步操作使用 `async/await` + `try-catch-finally`：

```typescript
private async doSomething(): Promise<void> {
  try {
    let result = await someAsyncApi()
    VtbLogger.info(TAG, '操作成功')
  } catch (error) {
    VtbLogger.error(TAG, `操作失败: ${JSON.stringify(error)}`)
  } finally {
    this.isLoading = false
  }
}
```

### 异步初始化模式

工具类使用 `init()` + 空值守卫：

```typescript
class XxxUtil {
  private resource: SomeType | null = null

  async init(context: common.Context): Promise<void> {
    try {
      this.resource = await getResourceAsync(context)
    } catch (error) {
      VtbLogger.error(TAG, `初始化失败: ${JSON.stringify(error)}`)
    }
  }

  async doWork(): Promise<void> {
    if (!this.resource) {
      VtbLogger.warn(TAG, '未初始化，无法执行操作')
      return
    }
    // 正常执行
  }
}
```

## 文件系统操作规范

```typescript
import { fileIo, fileUri } from '@kit.CoreFileKit'
```

### 沙箱路径选择

| 路径 | 用途 | 获取方式 |
|------|------|----------|
| `filesDir` | 持久化文件（用户数据） | `context.filesDir` |
| `cacheDir` | 临时文件（可被系统清理） | `context.cacheDir` |

### 文件复制标准写法

```typescript
const srcFile = fileIo.openSync(srcUri, fileIo.OpenMode.READ_ONLY)
const destFile = fileIo.openSync(destPath, fileIo.OpenMode.CREATE | fileIo.OpenMode.WRITE_ONLY)

const stat = fileIo.statSync(srcFile.fd)
const buffer = new ArrayBuffer(stat.size)
fileIo.readSync(srcFile.fd, buffer)
fileIo.writeSync(destFile.fd, buffer)

fileIo.closeSync(srcFile)
fileIo.closeSync(destFile)
```

### 文件存在检查 & URI 转换

```typescript
// 检查文件是否存在
let isExist = fileIo.accessSync(filePath)

// 路径 → URI
const uri = fileUri.getUriFromPath(filePath)
```

## @CustomDialog 弹窗规范

### 弹窗组件定义

```typescript
@CustomDialog
export struct ConfirmDialog {
  controller: CustomDialogController  // 必须声明
  title: string = ''
  message: string = ''
  onConfirm?: () => void
  onCancel?: () => void

  build() {
    Column() {
      Text(this.message)
      Row() {
        Button('取消').onClick(() => { this.onCancel?.(); this.controller.close() })
        Button('确定').onClick(() => { this.onConfirm?.(); this.controller.close() })
      }
    }
  }
}
```

### 调用方

```typescript
private dialogController: CustomDialogController = new CustomDialogController({
  builder: ConfirmDialog({
    title: '删除确认',
    message: '确定要删除吗？',
    onConfirm: () => { this.deleteItem() }
  }),
  autoCancel: true,
  alignment: DialogAlignment.Center,
  customStyle: true
})

// 显示
this.dialogController.open()
```

> **注意**：`@CustomDialog` 内部状态使用 V1 装饰器（`@State`），不能用 `@Local`

## 权限管理规范

### 标准流程

```typescript
import { VtbPermissionUtil } from '@vhm/basecommon/src/main/ets/utils/VtbPermissionUtil'

const context = this.getUIContext().getHostContext() as common.UIAbilityContext

// 单个权限：检查 + 申请一步完成
const hasPermission = await VtbPermissionUtil.checkRequestPermissions(context, 'ohos.permission.CAMERA')
if (!hasPermission) {
  this.getUIContext().getPromptAction().showToast({ message: '未授权相机权限' })
  return
}

// 多个权限：批量申请
const granted = await VtbPermissionUtil.requestPermissionsList(context, [
  'ohos.permission.CAMERA',
  'ohos.permission.MICROPHONE'
])
```

### module.json5 权限声明

```json5
"requestPermissions": [
  { "name": "ohos.permission.INTERNET" },                    // 无需弹窗
  { "name": "ohos.permission.GET_WIFI_INFO" },               // 无需弹窗
  { "name": "ohos.permission.VIBRATE" },                     // 无需弹窗
  {                                                            // 需要弹窗
    "name": "ohos.permission.CAMERA",
    "reason": "$string:camera_reason",
    "usedScene": { "abilities": ["EntryAbility"], "when": "inuse" }
  }
]
```

## 事件通知机制（emitter）

使用 `emitter`（`@kit.BasicServicesKit`）实现跨组件通信：

```typescript
import { emitter } from '@kit.BasicServicesKit'

// 定义事件 ID 常量
export const EVENT_HISTORY_UPDATE = 'event_history_update'

// 发送事件
emitter.emit(EVENT_HISTORY_UPDATE)

// 订阅（aboutToAppear 中）
emitter.on(EVENT_HISTORY_UPDATE, () => { this.refreshData() })

// 取消订阅（aboutToDisappear 中）
emitter.off(EVENT_HISTORY_UPDATE)
```

## 图片处理与分享规范

### 组件快照保存到相册

```typescript
import { componentSnapshot } from '@kit.ArkUI'
import { image } from '@kit.ImageKit'
import { photoAccessHelper } from '@kit.MediaLibraryKit'

// 1. 截图（组件须设置 id）
const pixelMap = await componentSnapshot.get('componentId')

// 2. 打包
const packer = image.createImagePacker()
const buffer = await packer.packing(pixelMap, { format: 'image/png', quality: 100 })

// 3. 存相册
const helper = photoAccessHelper.getPhotoAccessHelper(context)
const uri = await helper.createAsset(photoAccessHelper.PhotoType.IMAGE, 'png')
const file = fileIo.openSync(uri, fileIo.OpenMode.WRITE_ONLY)
fileIo.writeSync(file.fd, buffer)
fileIo.closeSync(file)
```

### 系统分享

```typescript
import { systemShare } from '@kit.ShareKit'
import { uniformTypeDescriptor as utd } from '@kit.ArkData'

const shareRecord: systemShare.SharedRecord = {
  utd: utd.UniformDataType.PNG,
  uri: fileUri.getUriFromPath(filePath),
  title: '分享图片'
}
const shareData = new systemShare.SharedData(shareRecord)
const controller = new systemShare.ShareController(shareData)
controller.show(context, {
  previewMode: systemShare.SharePreviewMode.DETAIL,
  selectionMode: systemShare.SelectionMode.SINGLE
})
```

## 模型与枚举定义规范

### 模型文件组织

| 场景 | 位置 | 示例 |
|------|------|------|
| 多页面共享的模型 | `models/XxxModels.ets` | `CodeModels.ets` |
| 仅工具类内部使用 | 就近定义在工具类中 | `HistoryUtil.ets` 中的 `MakeHistory` |

### 枚举定义

使用**字符串枚举**，便于序列化和调试：

```typescript
export enum HistoryType {
  TEXT = 'TEXT',
  WEBSITE = 'WEBSITE',
  IMAGE = 'IMAGE'
}
```

## Kit 依赖清单

| Kit | 导入路径 | 用途 |
|-----|----------|------|
| CoreFileKit | `@kit.CoreFileKit` | `fileIo`、`fileUri` 文件操作 |
| ArkData | `@kit.ArkData` | `preferences` 轻量存储、`utd` 统一数据类型 |
| ArkUI | `@kit.ArkUI` | `componentSnapshot` 截图、`promptAction` Toast |
| AbilityKit | `@kit.AbilityKit` | `common.UIAbilityContext` 上下文 |
| ImageKit | `@kit.ImageKit` | `image.createImagePacker` 图片处理 |
| MediaLibraryKit | `@kit.MediaLibraryKit` | `photoAccessHelper` 保存到相册 |
| ScanKit | `@kit.ScanKit` | `generateBarcode` 生成二维码、`scanBarcode` 扫码 |
| ShareKit | `@kit.ShareKit` | `systemShare` 系统分享 |
| NetworkKit | `@kit.NetworkKit` | `socket.TCPSocketServer` 本地服务器 |
| ConnectivityKit | `@kit.ConnectivityKit` | `wifiManager` 获取 WiFi/IP 信息 |
| BasicServicesKit | `@kit.BasicServicesKit` | `emitter` 事件通知 |
| PerformanceAnalysisKit | `@kit.PerformanceAnalysisKit` | `hilog` 底层日志（由 VtbLogger 封装） |

## globalThis 全局变量管理

### EntryAbility 中初始化

```typescript
globalThis.statusHeight = data.getWindowAvoidArea(window.AvoidAreaType.TYPE_SYSTEM).topRect.height
globalThis.bottomAvoidAreaHeight = data.getWindowAvoidArea(
  window.AvoidAreaType.TYPE_NAVIGATION_INDICATOR).bottomRect.height
```

### 页面中使用（需 px → vp 转换）

```typescript
.padding({
  top: this.getUIContext().px2vp(globalThis.statusHeight),
  bottom: this.getUIContext().px2vp(globalThis.bottomAvoidAreaHeight)
})
```

| 变量名 | 类型 | 用途 |
|--------|------|------|
| `globalThis.windowClass` | `window.Window` | 主窗口对象 |
| `globalThis.statusHeight` | `number`（px） | 状态栏高度 |
| `globalThis.bottomAvoidAreaHeight` | `number`（px） | 底部导航栏高度 |

## 代码审查检查清单

生成或修改代码后，按以下清单逐项自检：

### ArkTS 类型安全
- [ ] 所有变量、参数、返回值都有显式类型声明（无 `any` / `unknown`）
- [ ] 箭头函数参数和返回值都有显式类型：`(item: Model): string => item.id`
- [ ] 对象字面量都有对应的 `interface` 定义，不使用匿名对象类型
- [ ] 无解构声明（`const { a, b } = obj`），改用点语法
- [ ] `JSON.parse()` 结果通过 `as` 断言为已定义的 interface

### 废弃 API 检查
- [ ] 未使用 `getContext(this)` → 改用 `this.getUIContext().getHostContext()`
- [ ] 未使用全局 `promptAction.showToast()` → 改用 `this.getUIContext().getPromptAction().showToast()`
- [ ] 未使用全局 `px2vp()` → 改用 `this.getUIContext().px2vp()`
- [ ] 未使用 `console.log/error` → 改用 `VtbLogger`
- [ ] 未使用 `hilog` → 改用 `VtbLogger`

### 组件与装饰器
- [ ] `@ComponentV2` 组件使用 `@Local`（非 `@State`）
- [ ] `@CustomDialog` 组件使用 `@State`（非 `@Local`，V1 装饰器）
- [ ] `@Provider` / `@Consumer` 使用相同别名且提供默认值
- [ ] `ForEach` 提供了 `keyGenerator` 第三个参数
- [ ] `@Param` 属性提供了默认值

### 资源管理与内存安全
- [ ] 文件操作（`fileIo.openSync`）都有对应的 `fileIo.closeSync`
- [ ] `emitter.on()` 在 `aboutToAppear` 注册，`emitter.off()` 在 `aboutToDisappear` 注销
- [ ] 图片/颜色/尺寸优先使用资源引用 `$r('app.xxx')`，避免硬编码
- [ ] 异步操作都有 `try-catch` 包裹，`catch` 中使用 `VtbLogger.error`

### 权限与安全
- [ ] 敏感操作前调用 `VtbPermissionUtil.checkRequestPermissions()` 检查权限
- [ ] 新增权限已在 `module.json5` 的 `requestPermissions` 中声明
- [ ] 敏感权限（相机/麦克风/存储）提供了 `reason` 和 `usedScene`

### 代码风格
- [ ] UI 属性链式调用每个方法占一行
- [ ] 每行一个语句，缩进 2 空格
- [ ] 新增代码注释为中文
- [ ] 代码审查或自动修复时，不得以“格式统一”为理由把中文硬编码改成 Unicode



