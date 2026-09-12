# HarmonyOS 系统能力开发规范

## 适用范围

本规范负责日志、异步、文件、权限、数据库、事件、图片、相册、分享、Preferences、Kit 导入和资源释放。使用任何系统 API 前先按根入口核对当前 SDK 文档。

## Kit 导入

优先使用当前 SDK 推荐的 Kit 导入方式，并沿用目标项目已有稳定导入。

| Kit | 常用能力 |
|---|---|
| `@kit.AbilityKit` | `common.UIAbilityContext` |
| `@kit.ArkUI` | `componentSnapshot`、UI 能力 |
| `@kit.ArkData` | Preferences、统一数据类型 |
| `@kit.CoreFileKit` | `fileIo`、`fileUri` |
| `@kit.ImageKit` | 图片编码与处理 |
| `@kit.MediaLibraryKit` | 保存相册 |
| `@kit.ShareKit` | 系统分享 |
| `@kit.BasicServicesKit` | `emitter` |
| `@kit.NetworkKit` | 网络与 Socket |
| `@kit.ConnectivityKit` | Wi-Fi 与连接信息 |

- 不同时混用同一能力的新旧导入路径。
- 新增 API 前确认 `module.json5`、产品兼容版本和系统能力要求。

## 日志

- 统一使用 `VtbLogger`，不直接使用 `console.log`、`console.error` 或 `hilog`。
- `TAG` 使用当前类名或模块名。
- 调试细节使用 `debug`，关键业务节点使用 `info`，可恢复问题使用 `warn`，失败使用 `error`。
- 日志包含定位需要的业务标识，不记录密码、Token、隐私内容或完整敏感数据。

```typescript
VtbLogger.error(TAG, `保存失败: ${JSON.stringify(error)}`)
```

## Toast

- `entry` 模块统一使用 `entry/src/main/ets/common/Constants.ets` 导出的 `showToast`。
- 页面、组件和普通工具类不直接调用全局 `promptAction.showToast()`。
- Toast 文案面向用户，使用简短中文，不展示异常堆栈或内部错误码。

## 异步与错误处理

- 网络、文件、数据库、权限和系统 API 使用 `async/await` 时明确返回 `Promise<T>`。
- `try-catch` 只包住可能失败的外部操作；内部可控逻辑不层层捕获。
- 异常记录使用 `VtbLogger.error`，并恢复 Loading、按钮状态或已打开资源。
- 异常不能静默转换为成功结果。
- 多个有顺序依赖的初始化任务按业务顺序等待；互不依赖时再考虑并发。

## Preferences

- 优先复用项目已有 `VtbPreferenceUtil`，不重复封装 Preferences。
- 存储 key 统一定义，避免页面散落字符串。
- 写入后是否同步刷新按工具类现有实现处理。
- 用户数据、文件内容或复杂关系数据不使用 Preferences 代替数据库。

## 文件系统

- 文件能力优先复用脚手架 `entry/src/main/ets/utils/FileUtils.ets`，不要在页面或业务类中重复封装 `fileIo`、`fileUri` 和 `DocumentViewPicker`。
- 文件类型判断使用 `FileUtils.isImage()`、`isVideo()`、`isAudio()`；文件名、扩展名和大小处理使用 `getFileNameFromPath()`、`getFileExtension()`、`formatFileSize()`。
- 文件复制和写入优先使用 `FileUtils.copyFile()`、`copyRawFileToFile()`、`copyFileToSandbox()` 和 `writeFile()`；批量保存到用户选择的公共目录使用 `saveFilesToPublic()`。
- 持久化用户数据使用 `context.filesDir`，可清理临时数据使用 `context.cacheDir`。
- 只有 `FileUtils` 没有覆盖当前需求时才直接使用底层文件 API；`fileIo.openSync()` 打开的文件必须在成功和异常路径都关闭。
- 多资源操作优先使用 `try-finally` 保证释放。
- 路径转 URI 使用 `fileUri.getUriFromPath()`；访问前根据业务需要检查文件是否存在。
- 不把临时文件写入持久目录，也不把需要长期保存的用户数据只放在缓存目录。

完整复制示例读取 [examples/file-operation.md](examples/file-operation.md)。

## 权限

- 敏感操作前使用项目已有 `VtbPermissionUtil` 检查并申请权限。
- 新增权限同步维护 `module.json5` 的 `requestPermissions`。
- 需要用户授权的权限提供准确的 `reason` 和 `usedScene`。
- 用户拒绝后停止对应操作并给出明确提示，不循环触发申请。
- 权限名称、授权方式和可申请范围必须核对当前 SDK 文档。

## 数据库

- 使用数据库时必须复用项目统一的 `Rdb.ets`、`RdbCommon.ets` 和 `RdbTableImplGlobal.ets`，不得另起一套数据库初始化、表配置或全局表实例管理方式。
- `Rdb.ets` 负责数据库基础封装、数据库实例获取、初始化回调和公共资源处理；业务表不得重复封装同一套 `RdbStore` 初始化逻辑。
- `RdbCommon.ets` 集中维护表名、建表 SQL、列定义和数据库相关公共配置；业务表不得散落硬编码表结构。
- `RdbTableImplGlobal.ets` 负责业务表实例的统一注册和获取；页面、ViewModel 和其他业务类通过它取得 `XxxTable`，不得自行 `new XxxTable()` 或维护全局单例。
- 每个业务表对应一个 `XxxTable` 访问类，负责具体字段映射、CRUD、谓词和业务查询；`XxxTable` 通过 `Rdb` 获取数据库能力，并通过 `RdbCommon` 获取表配置。
- 页面不得直接创建或管理 `RdbStore`，不得直接实例化 `Rdb`、`XxxTable`，也不得绕过 `RdbTableImplGlobal` 操作数据库。
- `XxxModel` 只描述业务实体和嵌套数据结构；`XxxTable` 负责 `ValuesBucket` 组装、`RdbPredicates`、增删改查和结果映射；页面只负责触发查询和更新 UI 状态。
- 数据表初始化、复杂网络数据加载和网络数据入库由 `MainViewModel` 或同类统一入口负责，页面不重复拉取和写入同一份网络数据；数据库查询可以由页面通过 `RdbTableImplGlobal` 获取业务表访问类后直接调用。
- 单条插入、同步插入、批量插入、按主键查询、按业务字段查询、更新和删除都通过表访问类暴露明确方法。
- 更新和删除必须使用主键或明确业务条件构造 `RdbPredicates`，不能在页面中拼接 SQL 或传递不明确条件。
- 查询必须有稳定排序；文章列表沿用 `id` 升序，其他业务按项目真实展示顺序定义。
- `ResultSet` 使用列名获取列索引，再转换为明确的 Model；查询结果为空时返回空数组或明确的 `undefined`，不要让页面处理原始 `ResultSet`。
- 查询结果遍历遵循当前 SDK 的游标规则，处理完成后按 SDK 要求释放结果集和其他数据库资源。
- 复杂嵌套字段可以使用 `JSON.stringify` 存储，读取时通过已声明的模型类型解析；解析失败返回该字段的业务默认值，并记录必要日志。
- 布尔字段与数据库整数列转换统一，例如 `false/true` 对应 `0/1`，不要在页面散落转换逻辑。
- 可空主键用于区分新建和已有数据：新建时不写入无效主键，更新和删除前必须确认主键有效。
- 数据库异常只在数据库边界捕获并交给统一日志或回调处理，不在页面中层层包裹 `try-catch`。
- 修改表结构前先读取目标项目当前数据库版本、迁移策略和已有表实现，不自行猜测；需要升级时明确处理版本和迁移。
- `ArticleListPage -> RdbTableImplGlobal -> ArticleTable -> Rdb -> RdbCommon` 是文章数据库调用链；文章类 Model、Table 和列表页的完整分层示例见 [examples/database-rdb.md](examples/database-rdb.md)。

## emitter 事件

- 仅在缺少更直接的数据流方式时使用 `emitter` 进行跨组件通知。
- 事件 ID 定义为统一常量。
- 在 `aboutToAppear` 或对应生命周期注册，在 `aboutToDisappear` 或对应生命周期注销。
- 回调函数需要精确注销时，保留同一个函数引用。
- 不用 emitter 承载大量业务数据或替代明确的状态所有权。

## 图片、相册与分享

- 图片、视频、音频和文档选择优先使用脚手架 `entry/src/main/ets/utils/PhotoPickerUtils.ets`。
- 选择图片使用 `PhotoPickerUtils.selectImage()`，选择视频使用 `selectVideo()`，选择音频使用 `selectAudio()`，按后缀选择文档使用 `openDocumentPicker()`。
- 单个本地图片或视频保存到相册使用 `FileUtils.saveMediaToGallery()`；多张图片保存使用 `FileUtils.saveImageList()`；不要在页面中重复实现 `photoAccessHelper` 和文件复制流程。
- `PixelMap` 需要先写入缓存文件时使用 `FileUtils.savePixelMap()`，再调用 `saveMediaToGallery()`；组件截图仍由页面通过 `componentSnapshot` 获取。
- 组件截图前确保目标组件设置稳定 `id`。
- 页面持有的 PixelMap 等资源按当前 SDK 要求释放；工具类内部创建的图片打包器和文件由工具类负责释放。
- 选择媒体和保存相册仍需按脚手架调用方式及当前 SDK 核对权限、API 版本和用户取消结果。
- 分享 URI 使用系统可访问的 URI，不直接暴露内部路径。
- 完整流程读取 [examples/media-share.md](examples/media-share.md)。

## 自定义弹窗

- 项目仍使用 `@CustomDialog` 时，组件必须声明 `CustomDialogController`。
- 弹窗只负责 UI、输入和结果回调，不在弹窗内部执行页面跳转或复杂数据操作。
- 确认、取消和关闭路径明确，避免重复触发回调。
- 完整结构读取 [examples/custom-dialog.md](examples/custom-dialog.md)。

## 检查清单

- [ ] API 已按当前 SDK 文档确认版本、权限和返回类型。
- [ ] 日志与 Toast 使用项目统一封装。
- [ ] 文件、图片和监听器在所有路径正确释放或注销。
- [ ] 文件与媒体功能已优先复用 `FileUtils` 和 `PhotoPickerUtils`，没有在页面重复实现底层能力。
- [ ] 权限已声明，并在敏感操作前检查。
- [ ] 外部操作有必要的异常处理，内部逻辑没有大范围重复捕获。
- [ ] 没有在日志、Toast 或分享内容中泄露敏感数据。
