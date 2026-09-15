# HarmonyOS 开发规范入口

本文件是鸿蒙项目规范的唯一入口。处理 HarmonyOS NEXT、ArkTS 或 ArkUI 代码时，按本文定义的顺序加载规则。

## 适用范围

当前规范适用于同一类 HarmonyOS NEXT 项目，所有项目均以 `E:\DevEcoStudioProjects\zzz_hm_vtbbase` 为脚手架基线，统一使用 ArkTS、ArkUI 声明式 UI、状态管理 V2、`Navigation`、`NavDestination` 和声明式路由。

开发公共功能前先检查脚手架已有组件、工具类和调用方式。目标项目已经包含脚手架能力时使用目标项目内的版本；目标项目尚未同步对应能力时，以脚手架当前实现为参考，不重复创建同类封装。

## 规范加载顺序

所有鸿蒙代码任务依次加载：

1. `E:\rules\handwritten-style-general.md`
2. `E:\rules\harmony\handwritten-style-harmony.md`
3. `E:\rules\harmony\harmony-project.md`
4. `E:\rules\harmony\harmony-arkts.md`
5. 涉及页面、组件、状态或路由时加载 `E:\rules\harmony\harmony-arkui.md`
6. 涉及权限、文件、日志、数据库、分享或其他系统能力时加载 `E:\rules\harmony\harmony-system-api.md`
7. 根据当前任务读取 `E:\rules\harmony\examples` 下对应的完整示例

只修改配置、资源或文档时，按实际内容加载相关规则，不需要读取无关示例。

## 完整示例加载路由

| 当前任务 | 读取的完整示例 |
|---|---|
| 新建页面或页面壳 | `E:\rules\harmony\examples\page-v2.md` |
| 新建 V2 子组件或交互组件 | `E:\rules\harmony\examples\component-v2.md` |
| 声明式路由、页面跳转、结果回传 | `E:\rules\harmony\examples\navigation.md` |
| 状态管理 V2、列表刷新 | `E:\rules\harmony\examples\state-v2.md` |
| LazyForEach 数据源 | `E:\rules\harmony\examples\lazy-data-source.md` |
| 自定义弹窗 | `E:\rules\harmony\examples\custom-dialog.md` |
| 文件复制、路径、URI 和公共目录保存 | `E:\rules\harmony\examples\file-operation.md` |
| RDB 数据库、`Rdb`、`RdbCommon`、`RdbTableImplGlobal`、Model、Table 和列表页 | `E:\rules\harmony\examples\database-rdb.md` |
| 媒体选择、保存相册与系统分享 | `E:\rules\harmony\examples\media-share.md` |

同一任务涉及多个能力时，分别读取对应示例。不要一次性加载整个 `examples` 目录。

## 示例使用规则

- 开发对应能力前读取完整示例，用它确认组件结构、装饰器、生命周期位置和核心调用方式。
- 类名、路由名、资源名、提示文本、字段和具体业务逻辑可以按需求替换。
- 不复制当前业务不需要的状态、方法、监听、权限或依赖。
- 编码前读取目标项目中至少一个职责最接近的真实文件，优先沿用现有基类、公共组件、工具类、导入方式和书写节奏。
- 示例与目标项目真实代码冲突时，按下面的优先级处理。

## 冲突优先级

1. 编译正确、运行正确和业务正确
2. 当前 SDK 文档与目标项目真实 API
3. 当前项目已有架构、公共组件和稳定调用方式
4. `harmony-project.md` 中的项目强制规则
5. `harmony-arkts.md`、`harmony-arkui.md`、`harmony-system-api.md`
6. `handwritten-style-harmony.md`
7. 对应完整示例
8. 通用手写风格规则

同一层级出现冲突时，以目标项目当前需求和稳定代码为准。

## API 文档检查

编写或修改 HarmonyOS API 调用前，必须检查当前 SDK 文档，不能只凭历史知识。

本地文档路径：

```text
D:\software\Huawei\DevEco Studio\plugins\openharmony\ohos-info-center-view\static\hos\JsEtsAPIReference
```

优先使用 `rg` 搜索 API 名称，再读取匹配文档。只有本地文档没有对应内容或需要确认最新行为时，再查华为开发者文档。

重点核对：API 版本、废弃标记、参数类型、返回类型、权限、系统能力、异常码和生命周期要求。
