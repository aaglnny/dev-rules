# Android 项目公共开发规范

## 适用范围

本规范负责 Android 项目中与 Java、Kotlin 语法无关的公共约束，包括项目工作流、文件语言边界、基类体系、组件职责、Room、数据库异步、网络、Dialog、依赖、Manifest 和包结构。

本规范不维护 Java/Kotlin 业务实现或页面布局、资源示例；只保留必要的 Manifest、Gradle 和包结构等项目配置片段：

- Kotlin 语法和 Kotlin 示例读取 [android-kotlin.md](android-kotlin.md)。
- Java 语法和 Java 示例读取 [android-java.md](android-java.md)。
- XML 布局、Drawable、控件属性和资源示例读取 [android-xml.md](android-xml.md)。

## 规范加载顺序

处理 Kotlin 代码时依次加载：

1. `E:\rules\handwritten-style-general.md`
2. `E:\rules\android\android-project.md`
3. `E:\rules\android\android-kotlin.md`
4. 涉及 XML 或资源时加载 `E:\rules\android\android-xml.md`

处理 Java 代码时依次加载：

1. `E:\rules\handwritten-style-general.md`
2. `E:\rules\android\android-project.md`
3. `E:\rules\android\android-java.md`
4. 涉及 XML 或资源时加载 `E:\rules\android\android-xml.md`

只修改 XML 或资源时加载 `E:\rules\android\android-project.md` 和 `E:\rules\android\android-xml.md`，无需加载两份语言规范。

规范之间发生冲突时，按以下顺序处理：

1. 编译正确和业务正确
2. Android 平台及当前项目基类的真实约束
3. 本规范中的项目强制规则
4. 当前模块已经稳定使用的人工代码风格
5. 对应语言或 XML 规范
6. 通用手写风格规则

同一层级出现冲突时，以项目真实代码和当前需求为准。本文件从原 Java、Kotlin 规范迁移公共规则时，冲突项以迁移前的 `android-kotlin.md` 为准。

# 核心原则与工作流

## 核心原则

- 遵循 KISS 原则，优先保证实现简洁、直接、可维护。
- 先理解现有项目结构、基类、依赖版本和业务调用链，再进行设计和编码。
- 以项目真实代码和运行约束为准，不凭空引入项目未使用的框架或模式。
- 只修改当前需求涉及的文件，不顺手重构、重命名或格式化无关代码。

## 开发流程

- 进行设计或编码前先完成项目调研，厘清会影响实现的疑点。
- 需要方案确认的任务遵循“构思方案 → 提请审核 → 分解任务 → 实施”的顺序。
- 新增实现优先复用当前模块已有基类、工具类、组件和同类页面写法。

## 验证规则

- 完成修改后默认只进行静态检查，不执行 Gradle 构建。
- 默认不安装 APK，不使用模拟器或真机验证。
- 默认不使用 Android CLI、ADB 等命令行工具验证。
- 只有用户明确要求时，才执行上述构建或运行验证。

## 输出规范

- 所有回复、分析说明、实施方案、任务清单和新增 Markdown 文档使用中文。

# 项目环境与技术边界

这是一个 Android Java/Kotlin 混合项目。继续使用项目已有的 DataBinding/ViewBinding、Room、RecyclerView、Presenter、RxJava3、Glide、Android 原生 Dialog，以及现有 `BaseActivity`、`BaseFragment`、`BaseRecylerAdapter`、`SimpleObserver` 等公共能力。

- 不因为新增 Kotlin 代码就自动引入 Compose、Flow、Hilt、Navigation、MVI 等框架。
- 只有目标模块已经使用协程，或用户明确要求时，才使用协程完成对应功能。
- 不改变现有业务架构、基类体系、资源组织方式和组件选型，除非需求明确要求。

# 文件语言边界

本节为强制规则。

## 现有文件

- 严禁将任何现有 `.java` 文件重命名、迁移或重写为 `.kt` 文件。
- 修改现有 Java 文件时继续使用 Java，只做当前需求需要的修改。
- 不以“统一语言”“Kotlin 化”或“减少样板代码”为理由转换 Java 文件。
- Java 与 Kotlin 在同一项目和同一业务包中正常共存，不要求建立平行模块或平行包结构。

## 新增文件

- 创建新的代码文件时优先使用 Kotlin。
- `dao`、`entitys` 包及其子包中的代码文件必须使用 Java，严禁创建 `.kt` 文件。
- 文件语言以实际 `package` 归属为准，不能通过更换目录或创建近似包名绕过限制。
- 除 `dao`、`entitys` 外，如果目标模块已有明确且必须保持的 Java 实现约束，继续使用 Java。
- 新增 Kotlin 文件放在模块现有源码目录中，不单独创建 Kotlin 专用源码树。

# Android 组件公共规则

## Activity

- Activity 继承项目现有 `BaseActivity<Binding, Presenter>`。
- 普通页面的 Presenter 通常使用 `BasePresenter`；网络页面使用对应 Contract 的 Presenter 并实现 View 接口。
- 在 `onCreate()` 中调用 `setDataBindingLayout()`，不使用 `setContentView()` 绕过基类流程。
- 方法默认按 `onCreate()`、`initView()`、`bindEvent()`、点击回调、Presenter/接口回调、私有业务方法的顺序组织。
- 页面标题栏统一复用 `layout_title_bar`，通过 Binding 对应的 `include` 设置标题，不创建重复标题栏。
- 页面初始化数据默认在 `initView()` 中调用 `getData()`。
- 点击事件集中在基类约定的点击回调中处理。

## Fragment

- Fragment 继承项目现有 `BaseFragment<Binding, Presenter>`。
- 无参数 Fragment 提供 `newInstance()`；有参数 Fragment 通过 `arguments` 传递，不依赖可能丢失的自定义构造参数。
- 实现基类要求的 `onSetLayoutId()`、`initView()`、`bindEvent()` 和点击回调。
- 只在 Binding 有效的生命周期内访问视图。
- 异步回调返回时，如果 Fragment 已脱离 Activity，不执行依赖 Activity 的 UI 操作。
- Context 和 Activity 的获取方式必须符合当前生命周期，不在无把握的异步回调中直接强取 Activity。

## RecyclerView 与 Adapter

- RecyclerView 必须设置 `LayoutManager`，并按现有同类页面设置对应的 `ItemDecoration`。
- Adapter 继承项目现有 `BaseRecylerAdapter<Entity>`，实现 `convert(holder, position)`。
- 继续使用基类的 `MyRecylerViewHolder` 和 `mDatas`，不擅自替换为其他 Adapter 基类、`ListAdapter` 或 Paging。
- 使用基类提供的 `holder.setText()`、`holder.getTextView()`、`holder.getImageView()`、`holder.getView()` 等方法绑定数据。
- 外部刷新数据使用 `adapter.addAllAndClear(list)`，不直接修改 Adapter 的 `mDatas`。
- `convert()` 只负责列表展示、局部 UI 状态和按钮回调，不执行数据库、网络、页面跳转或复杂业务。
- 整个 Item 的点击和长按使用基类的 `setOnItemClickLitener()`、`setOnLongItemClickLitener()`，监听在 Activity、Fragment 或 Dialog 外部设置。
- 不为整个 Item 重复定义点击接口，也不在 Adapter 构造方法中设置 Item 监听。
- Item 内部有删除、编辑等按钮时，复用 `ButtonClickListener<T>`，或定义单一职责的专用接口；Adapter 只触发回调。

## Binding

- 使用项目基类的页面通过 `binding` 访问视图，禁止使用 `findViewById()`。
- DataBinding 和 ViewBinding 的选择沿用当前模块，不为同一页面建立两套 Binding。
- ViewBinding 的创建和销毁必须符合 Activity/Fragment 生命周期；Fragment 在 `onDestroyView()` 后清理 View 引用，基类已封装时直接复用基类。
- Binding 的 XML 声明与资源规则见 [android-xml.md](android-xml.md)，调用示例见对应语言规范。

# Entity、Dao 与 Room

## Entity

- `entitys` 包及其子包中的 Entity 必须使用 Java。
- Room Entity 使用无参数的 `@Entity`，不设置 `tableName`，表名使用 Entity 类名。
- Room Entity 不使用 `@NonNull`、`@Nullable`；字段、构造参数和 getter/setter 均不添加这两个注解。
- 主键使用 `@PrimaryKey(autoGenerate = true)`。
- Entity 实现 `Serializable`，字段使用 `private`，并提供 getter/setter。
- 时间字段使用 `long` 时间戳。
- 非 Room Entity 也按项目约定实现 `Serializable`，空值由调用层按实际来源处理。
- Java 实现示例见 [android-java.md](android-java.md)。

## Dao

- `dao` 包及其子包中的 Dao 必须使用 Java。
- Dao 的 SQL 表名必须与 Entity 类名一致，不能引用自定义或已移除的 `tableName`。
- Dao 只返回 `List<T>`、Entity、基础包装类型、`void` 等普通类型，不返回 `Observable`、`Single` 或 `Completable`。
- 插入默认使用 `OnConflictStrategy.REPLACE`。
- 列表查询默认按 `id DESC` 排序，除非业务明确要求其他顺序。
- 使用 `DatabaseManager.getInstance(context).getXxxDao()` 获取 Dao。
- Java 接口示例见 [android-java.md](android-java.md)。

## Room 配置

- 开发阶段数据结构变更可卸载 App 清除旧数据后重新安装，不强制处理 Migration。
- 用户明确要求保留线上数据、升级正式数据库版本或提供发布迁移方案时，必须实现并验证 Migration。
- Java/Kotlin 混合模块的 Room 注解处理器统一使用 `annotationProcessor androidApi.library.roomprocessor`，不使用 kapt 处理 Room 注解。
- Room 相关依赖统一使用项目 `androidApi` 配置：

```groovy
implementation androidApi.library.room
annotationProcessor androidApi.library.roomprocessor
implementation androidApi.library.roomRxjava3
```

- `@Database` 的 `entities` 包含全部 Room Entity，并为每个 Dao 提供抽象 getter。
- `DatabaseManager` 使用线程安全单例，并使用 Application Context 创建数据库。
- 当前项目允许保留 `allowMainThreadQueries()` 配置，但业务代码仍必须按本规范使用 RxJava3 在 IO 线程执行数据库操作。
- Java `DatabaseManager` 示例见 [android-java.md](android-java.md)。

# 数据库与 RxJava3 异步规则

数据库操作统一采用以下方式：

1. Dao 只暴露普通返回类型。
2. 调用层使用 `Observable.create()` 包裹 Dao 操作。
3. 使用 `subscribeOn(Schedulers.io())` 执行数据库操作。
4. 使用 `observeOn(AndroidSchedulers.mainThread())` 更新 UI。
5. 捕获数据库异常并传递给 `onError()`，向用户展示友好提示。
6. 成功路径正确发送 `onNext()` 和 `onComplete()`，保存、删除等无业务返回值操作使用对应语言的空结果类型。

补充要求：

- 不在主线程直接调用 Dao。
- 不在 Dao 内部切换线程。
- 不使用 `Observable.just(dao.queryAll())`，因为 Dao 会在创建 Observable 时立即执行。
- 页面销毁后的订阅管理沿用项目现有方式；基类已管理 Disposable 时必须复用。
- 查询、保存和删除的完整实现见对应语言规范。

# 工具类与通用组件

## Toast

- 统一使用 `com.blankj.utilcode.util.ToastUtils.showShort()`。
- 如果当前模块统一封装了同名 `ToastUtils`，以模块真实导入为准，不混用不同实现。

## 确认对话框

- 删除等不可逆操作执行前必须显示确认弹窗。
- 默认复用 `DialogUtil.showConfirmRreceiptDialog()`；需要更多配置时使用 `ConfirmDialog.Builder`。

## 时间与尺寸

- 时间格式化优先复用 `VTBTimeUtils.formatDateTime()`、`VTBTimeUtils.getCurrentTime()`、`VTBTimeUtils.strToDate()`。
- dp 转 px 使用 `SizeUtils.dp2px()`，不重复创建同类工具或扩展函数。

# 网络请求规则

- 继续使用项目现有 Presenter 模式，不在 Activity 或 Fragment 中绕过 Presenter 直接创建 Retrofit 请求。
- 网络页面使用对应 Contract 的 Presenter，并实现 View 接口。
- Presenter 在 `initView()` 中按项目方式初始化并发起请求。
- Gson 泛型解析使用对应语言的匿名 `TypeToken`。
- 网络错误提示和 Loading 显隐复用现有基类。
- 成功回调中不执行复杂耗时计算；耗时逻辑切换到后台线程。
- Java、Kotlin 完整示例见对应语言规范。

# Activity 注册与页面跳转

- 新增 Activity 必须在 `AndroidManifest.xml` 注册，并沿用项目现有屏幕方向和 `tools:ignore` 写法。

```xml
<activity
    android:name=".ui.xxx.XxxActivity"
    android:screenOrientation="portrait"
    tools:ignore="DiscouragedApi,LockedOrientationActivity" />
```

- 无参数跳转优先使用基类 `skipAct()`。
- 携带参数时使用 `Intent`；入口被多处调用时，可由目标 Activity 提供简洁的 `start()` 方法。
- Entity 通过 Intent 传递时必须实现 `Serializable`。
- Intent Key 在多个入口复用时定义常量。
- 从非 Activity Context 启动页面时，按 Android 规则处理 `FLAG_ACTIVITY_NEW_TASK`。
- 不为单一入口建立复杂路由封装。

# 自定义 Dialog

- 自定义弹窗统一使用 Android 原生 `Dialog` 和 DataBinding。Kotlin 实现参考 [android-kotlin.md](android-kotlin.md) 中的 `XxxSelectDialog`，Java 实现参考 [android-java.md](android-java.md) 中的 `XxxSelectDialog`。
- 不使用 XPopup，不为自定义 Dialog 引入新的第三方弹窗框架。
- Binding 使用 `DataBindingUtil.inflate()` 创建，并通过 Binding 根视图设置内容。
- Window 的位置、宽度、软键盘模式和透明背景按设计配置；居中弹窗宽度按屏幕宽度减去两侧间距计算。
- 选项较少且只需返回索引时使用 `DialogInterface.OnClickListener`；返回 Entity、日期等复杂数据时定义单一职责接口。
- Dialog 只处理 UI 初始化和结果回调，不执行页面跳转、数据库写入或网络请求。
- 点击选项后默认关闭；只有业务明确要求连续操作时才保持显示。
- 关闭按钮只关闭 Dialog，不返回业务选项结果。
- 日期、月份等选择界面也按相同原生 Dialog 结构创建。
- Dialog 布局统一参考 [android-xml.md](android-xml.md) 中的 Dialog 布局示例。

# 表单验证与错误处理

## 表单验证

- 保存前先完成同步验证，验证通过后再执行异步保存。
- 依次校验必填值、数据类型、业务范围和条件必选项。
- 验证失败时给出明确中文提示并立即返回。
- 空值判断和安全数字解析使用对应语言的惯用写法，示例见语言规范。

## 错误处理

- 数据库异常在异步源中捕获并传递给观察者，不直接向用户展示堆栈或敏感内部信息。
- 文件读写、Bitmap 转换和第三方图片处理只包住实际可能失败的操作，不用大范围 `try/catch` 包裹整个页面方法。
- 失败后恢复必要 UI 状态，例如关闭 Loading，不返回伪造成功结果。
- 项目内部可控数据不层层捕获异常；能用类型和明确分支表达时不使用异常。

# 依赖管理

项目当前主要使用：

- Kotlin
- DataBinding / ViewBinding
- RecyclerView
- Room + Room RxJava3
- Glide
- Retrofit + RxJava3
- Gson
- AndroidUtilCode
- Android 原生 Dialog
- 项目现有基类、Contract、Presenter 和 Observer

可按需求使用的现有约定依赖：

```groovy
implementation 'com.haibin:calendarview:3.7.1'
implementation 'io.github.lucksiege:pictureselector:v3.11.2'
implementation 'io.github.lucksiege:compress:v3.11.2'
api 'com.makeramen:roundedimageview:2.3.0'
```

依赖规则：

- 先确认项目已有依赖，禁止重复添加。
- 版本优先使用 `androidApi` 等项目统一配置，不在业务模块硬编码冲突版本。
- 新增依赖前必须确认现有工具或组件无法满足需求。
- 不因语言切换增加与需求无关的 Kotlin 扩展库。

# 通用代码风格、命名与包结构

## 通用代码风格

- 变量和方法使用小驼峰命名。
- Adapter 默认命名为 `adapter`，单一列表默认命名为 `list`，Binding 固定使用 `binding`。
- Context 使用 `context`；基类已有 `mContext` 时沿用基类字段。
- 布尔值优先使用 `is`、`has`、`can`、`need` 前缀。
- 不创建 `EnhancedXxx`、`OptimizedXxx`、`UnifiedXxx` 等机器感命名。
- 使用 import 后直接写类名，只有同名类冲突时才使用全限定名。
- 不使用 `*` 通配导入，除非项目格式规则明确允许。
- 删除本次修改产生的未使用 import，不为整理 import 修改无关代码。
- 新增注释使用中文，解释业务原因、兼容逻辑、线程要求或第三方库限制。
- 不写解释显而易见语句的注释，不添加模板化作者、日期和版本注释。

## 组件命名

- Activity：`XxxActivity`、`XxxListActivity`、`AddXxxActivity`
- Fragment：`XxxFragment`
- Adapter：`XxxAdapter`、`XxxListAdapter`
- Entity：`XxxEntity`
- Dao：`XxxDao`
- Dialog：`XxxDialog`、`XxxSelectDialog`

资源命名和目录规则见 [android-xml.md](android-xml.md)。

## 包结构

继续沿用现有项目包结构：

```text
com.xxx.project_name/
├── ui/
│   ├── adapter/
│   ├── mime/
│   │   ├── main/
│   │   └── .../
│   └── .../
├── widget/
│   └── dialog/
├── dao/
├── entitys/
├── utils/
└── common/
```

- Java 与 Kotlin 同包共存时保持包名一致。
- 不主动修正历史包名 `entitys`，除非用户明确要求整体迁移。
- 新文件放在对应业务包，不建立无业务意义的 `helper`、`manager`、`extension` 包。
