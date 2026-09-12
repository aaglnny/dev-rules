# HarmonyOS 项目公共开发规范

## 适用范围

本规范负责所有鸿蒙项目共用的工程架构、页面职责、数据流、路由组织、公共库、资源策略、工作流和验证规则。ArkTS 语法读取 [harmony-arkts.md](harmony-arkts.md)，ArkUI 与状态管理读取 [harmony-arkui.md](harmony-arkui.md)，系统能力读取 [harmony-system-api.md](harmony-system-api.md)。

## 核心原则

- 遵循 KISS 原则，先完成清晰、可维护的业务闭环。
- 先读取当前模块的真实代码，再决定命名、目录、组件和调用方式。
- 开发公共功能前检查主入口指定脚手架及目标项目已有能力，不重复创建同类组件和工具。
- 只修改当前需求涉及的文件，不顺手重构、重命名或格式化无关代码。
- 不为未来可能出现的需求提前引入框架、目录、基类或扩展点。

## 开发流程

- 开始设计或编码前，确认项目入口、页面流向、数据来源、路由方式、已有公共能力和相关 SDK API。
- 需要方案审核的任务依次进行：构思方案、提请审核、分解任务、实施。
- 开发完成后执行静态检查和与修改范围相称的验证。
- 除非用户明确要求，不主动执行 `hvigorw`、`assembleApp`、打包、安装或真机运行。

## 项目结构

沿用项目现有多模块结构：

```text
├── AppScope/
│   ├── app.json5
│   └── resources/
├── entry/
│   └── src/main/
│       ├── ets/
│       │   ├── entryability/
│       │   ├── abilityStage/
│       │   ├── pages/
│       │   ├── model/
│       │   ├── view/
│       │   ├── viewModel/
│       │   ├── database/
│       │   ├── common/
│       │   └── utils/
│       ├── resources/
│       └── module.json5
├── build-profile.json5
└── oh-package.json5
```

- 新文件放入现有业务目录，不创建无明确职责的 `helper`、`manager` 或平行目录。
- 需要抽取模型时放在 `model/XxxModel.ets`；是否抽取由项目根据实际复用范围和业务职责自行判断。
- 页面、组件、工具、模型、服务和常量沿用 `XxxPage.ets`、`XxxComponent.ets`、`VtbXxxUtil.ets`、`XxxModel.ets`、`XxxService.ets`、`XxxConstants.ets`。

## 应用入口与主框架

- 启动页只负责初始化、合规处理和跳转主页面，不承载复杂业务展示。
- 首屏保持单一入口，避免多个页面分别执行同一初始化逻辑。
- 主页面负责底部导航、页面容器和全局上下文，不承载大量具体业务。
- 主页面统一提供 `pathStack` 和数据刷新信号 `refreshNetworkData`。
- 底部 Tab 的数量、顺序和名称先确定，再围绕它扩展子页面。

## 页面职责

- 首页负责聚合展示，列表页负责列表，详情页负责单条完整内容，设置页负责说明和配置。
- 页面壳负责布局和路由容器；业务方法负责数据读取、保存和跳转。
- 列表项点击只执行跳转或回调，不顺带执行无关业务。
- 详情页接收上页关键参数后，优先按 `id` 或业务主键重新查询完整数据。
- 页面标题栏、状态和交互组件遵循 [harmony-arkui.md](harmony-arkui.md)。

## 数据加载与本地表

- 复杂网络数据加载和网络数据入库统一交给 `MainViewModel` 或同类 ViewModel，页面不重复请求同一份网络数据；数据库查询可以由页面通过业务表访问类直接调用。
- 初始化先建立本地数据库表，再按业务顺序请求网络数据并写入本地表。
- 页面展示优先读取本地表，不直接依赖网络响应对象。
- 初始化完成后提供明确刷新信号，让子页面按需重新查询。
- 每一种业务实体单独建表；模型字段尽量与原始数据源一致。
- 复杂内容可以保存为 JSON 字符串，读取时恢复为已声明类型。
- 列表页和详情页读取同一张表，避免维护两套数据状态。
- 新增业务模块按“模型、数据表、初始化、列表页、详情页、路由、入口”的顺序完成闭环。
- 数据库实体、单表访问类和列表页面分别维护，页面只调用业务查询方法，不直接操作 `relationalStore`。
- RDB 的 Model、Table、字段映射和列表查询方式参考 [examples/database-rdb.md](examples/database-rdb.md)。

## 路由组织

- 主框架内部页面使用 `Navigation`、`NavDestination` 和 `NavPathStack`。
- 启动页、合规页等主框架外页面可以沿用项目现有 `router.pushNamedRoute()` 或 `router.replaceUrl()`。
- 路由名统一放入 `BuilderNameConstants`，页面中不散落字符串路由名。
- 路由协议统一为 `{moduleName}://{PageName}`。
- 声明式路由在 `module.json5` 配置 `routerMap`，并在 `route_map.json` 中保持 `name`、`pageSourceFile`、`buildFunction` 一致。
- 新增页面时依次补充 Builder、路由常量、`route_map.json` 和跳转入口。
- 普通跳转、参数传递和结果回传读取 [examples/navigation.md](examples/navigation.md)。

## 公共库与统一出口

- 公共 HAR 的页面、组件和稳定能力通过 `Index.ets` 统一导出；项目已经使用直接源码路径导入时沿用现状。
- 不为已有公共能力重新创建同类工具或组件。
- 目标项目内的工具类已经随业务更新时使用目标项目版本；脚手架作为缺失能力和调用方式的统一参考。
- `CommonConstants` 保存项目反复使用的比例、字体权重、索引和布局常量；只使用一次的简单值可留在局部。
- 日志、Toast、权限、文件和媒体能力遵循 [harmony-system-api.md](harmony-system-api.md)。

## 资源与页面视觉

- 图片、颜色、字号和尺寸优先使用 `$r('app.xxx')` 资源，简单局部间距可以沿用项目中稳定的数字写法。
- 字符串默认沿用当前项目直接书写方式；项目已经国际化时使用字符串资源。
- 图片、图标、背景和占位图使用稳定、可识别的资源名称。
- 普通图标只固定一个方向，另一方向使用 `'auto'`，避免拉伸；裁剪或铺满固定容器时可以同时设置宽高。
- 列表页必须有符合当前产品设计的空状态。
- 列表页和详情页保持一致的间距、字号、圆角和背景风格。

## 代码组织与手写风格

- 通用代码表达遵循 `E:\rules\handwritten-style-general.md`，鸿蒙特有表达遵循 [handwritten-style-harmony.md](handwritten-style-harmony.md)。
- 编译正确、平台约束和业务正确性优先于代码风格。

## 日常检查

- 页面职责是否明确，是否复用了项目已有组件和工具。
- 数据是否由统一入口初始化并先入库，再由页面查询展示。
- 路由名、Builder 和 `route_map.json` 是否一致。
- 列表是否有空状态，详情是否重新查询完整数据。
- 新增代码是否只覆盖当前需求，没有无关重构。
- 涉及的 SDK API 是否已核对当前文档。
