# HarmonyOS ArkUI 与状态管理开发规范

## 适用范围

本规范负责 `@ComponentV2`、状态管理 V2、页面与子组件、Builder、导航、列表刷新、资源和安全区。完整结构根据任务读取 `examples` 目录。

## 页面与组件

- 新页面和普通子组件统一使用 `@ComponentV2`。
- 页面使用 `@Entry` 或声明式路由导出的 Builder 入口，具体形式沿用当前项目。
- 页面内部状态使用 `@Local`，父组件传入的只读参数使用 `@Param`。
- 跨层级共享使用 `@Provider` 和 `@Consumer`，二者使用相同别名并提供符合业务的默认值。
- 子组件向父组件回传事件使用 `@Event`；简单且项目已有稳定普通回调写法时沿用现状。
- `@CustomDialog` 仍按 V1 组件约束使用 `@State`，读取 [examples/custom-dialog.md](examples/custom-dialog.md)。

## 状态管理 V2

- `@Local` 数组增加、删除、排序或替换元素后，优先重新赋值新数组触发刷新。
- 深层对象需要细粒度观察时，按当前 SDK 文档使用 `@ObservedV2` 和 `@Trace`。
- `@Param`、`@Provider` 和 `@Consumer` 提供明确类型和默认值。
- 共享状态只上提到实际需要的最近公共祖先，不建立无边界全局状态。
- 页面需要响应统一刷新信号时使用当前项目已有监听方式，不重复建立全局通知机制。

完整示例读取 [examples/state-v2.md](examples/state-v2.md)。

## Builder 与组件拆分

- `@Builder` 用于无独立生命周期的轻量 UI 片段。
- `@Builder` 内部可以直接读取所属 `@ComponentV2` 的 `@Local`，并随该状态变化刷新 UI。
- 不要把 `@Local` 作为普通方法参数传入 `@Builder`；需要传递状态时使用明确的普通值参数，或改用 `@ComponentV2` 的 `@Param` 和 `@Event`。
- 需要独立状态、独立生命周期或复杂交互的可复用区域，优先拆成 `@ComponentV2` 子组件。
- 简单页面可以直接写 UI，不要求每个区域都拆 Builder 或组件。
- 同一 UI 被多处复用、拥有独立状态或事件、或者当前页面已明显过长时再抽组件。
- Builder 只负责 UI 表达和轻量回调，不直接执行数据库、网络或复杂业务。
- `@ComponentV2` 的 `@Event`、`@Param` 和业务成员不得命名为 `onClick`、`onTouch` 等基类或 `CommonAttribute` 已占用的事件方法；使用 `onItemClick`、`handleClick` 等业务语义名称。
- 组件内部的点击手势仍使用 `.onClick((): void => {})`；该链式 API 与组件成员回调名称分开处理。

## 声明式路由

- 主框架内部统一使用 `NavPathStack`。
- 普通跳转优先使用 `this.pathStack?.pushPath({ name, param })`。
- 需要处理返回结果时统一使用 `this.pathStack?.pushPath({ name, param, onPop })`，通过 `onPop` 接收目标页面返回的数据。
- 返回使用 `this.pathStack?.pop()`；需要回传结果时使用 `this.pathStack?.pop(result)`。
- `@Provider('pathStack')` 与 `@Consumer('pathStack')` 使用相同别名。
- 页面内的 `pathStack` 声明为 `NavPathStack | undefined`，初始值为 `undefined`，并在 `NavDestination.onReady` 中赋值为 `context.pathStack`。
- 路由名从 `BuilderNameConstants` 读取，`param` 统一使用对象字面量并断言为 `Record<string, Object>`；是否抽取模型由项目自行判断。

完整路由配置和代码读取 [examples/navigation.md](examples/navigation.md)。

## 页面导航栏

- 标准“返回键 + 标题”页面优先使用项目统一的 `BaseTitleBar_V2.ets` 中的 `BaseTitleBar_V2`。
- 返回回调统一调用 `this.pathStack?.pop()`。
- 顶部存在 Tab、搜索栏或其他复杂结构时，沿用当前项目对应页面写法。
- 页面使用自定义标题栏时隐藏 `NavDestination` 默认标题栏。

## ForEach 与列表

- `ForEach` 必须提供稳定且唯一的 `keyGenerator`。
- 不使用数组下标作为可增删、可排序业务列表的长期 key。
- `LazyForEach` 使用项目已有 `LazyDataSource` 或 `LazyDataSourceV2`，完整结构读取 [examples/lazy-data-source.md](examples/lazy-data-source.md)。
- 列表为空时显示明确空状态。

```typescript
ForEach(this.list, (item: DataModel): void => {
  Text(item.title)
}, (item: DataModel): string => item.id.toString())
```

## UIContext 与废弃 API

- 获取 UIAbility 上下文使用 `this.getUIContext().getHostContext() as common.UIAbilityContext`。
- px 转 vp 使用 `this.getUIContext().px2vp(value)`。
- `entry` 模块 Toast 使用项目 `showToast()` 封装；系统能力层需要直接操作时使用当前 `UIContext` 的 PromptAction。
- 禁止新增全局 `getContext(this)`、全局 `px2vp()` 或全局 `promptAction.showToast()` 调用。

## 安全区与沉浸式布局

- `EntryAbility` 统一初始化主窗口和系统安全区高度。
- `globalThis.statusHeight`、`globalThis.bottomAvoidAreaHeight` 保存 px 值，页面使用前通过当前 `UIContext` 转换为 vp。
- 页面不重复查询同一安全区，也不在多个页面建立不同全局变量名。
- 已使用窗口适配新 API 的项目，以当前项目真实实现和 SDK 文档为准。

## 资源和布局

- ArkUI 属性链每个方法单独一行。
- 颜色、字号、常用尺寸优先使用资源引用；简单局部间距可以沿用项目现有数字写法。
- 图标通常固定一个方向，另一方向使用 `'auto'`。
- 不因整理代码把中文文本转换为 Unicode 转义。

## 检查清单

- [ ] V2 组件使用了正确的状态装饰器。
- [ ] `@Provider` 与 `@Consumer` 的别名一致并有默认值。
- [ ] `@Local` 没有作为普通方法参数传入 `@Builder`。
- [ ] 需要独立状态、独立生命周期或复杂交互的区域已使用 `@ComponentV2`，简单 UI 复用才使用 `@Builder`。
- [ ] `ForEach` 有稳定唯一 key。
- [ ] 路由常量、Builder 和 `route_map.json` 一致。
- [ ] 未新增废弃的全局上下文、Toast 或单位转换 API。
- [ ] `@ComponentV2` 没有使用 `onClick` 等基类事件方法名称声明业务成员。
