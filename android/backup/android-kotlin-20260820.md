# 核心理念与原则

> **简洁至上**：恪守 KISS（Keep It Simple, Stupid）原则，优先保证代码简洁、直接、可维护，避免过度工程化和不必要的防御性设计。
>
> **深度分析**：立足于第一性原理分析问题，先理解现有项目结构、基类和业务调用链，再进行设计与编码。
>
> **事实为本**：以项目真实代码、依赖版本和运行约束为准，不凭空引入项目未使用的框架或模式。

# 开发工作流

> **渐进式开发**：通过多轮对话明确需求。在进行设计或编码前，必须先完成项目调研并厘清影响实现的疑点。
>
> **结构化流程**：严格遵循“构思方案 → 提请审核 → 分解为具体任务”的顺序。
>
> **最小改动**：只修改当前需求涉及的文件，不顺手重构、重命名或格式化无关代码。

## 验证规则

> 完成代码修改后默认只进行静态检查，不执行 Gradle 构建。
>
> 不安装 APK，不使用模拟器或真机验证。
>
> 不使用 Android CLI、ADB 等命令行工具验证。
>
> 只有用户明确要求时，才执行上述构建或运行验证。

# 输出规范

> **语言要求**：所有回复、分析说明、实施方案、任务清单和 Markdown 文档均使用中文。

---

# Android Kotlin 项目开发规范

# 项目概述

这是一个 Android Java/Kotlin 混合项目，新增页面和普通业务代码优先使用 Kotlin，继续使用现有项目中的 DataBinding/ViewBinding、Room 数据库、RecyclerView、Presenter、RxJava3、Glide 和 Android 自定义 Dialog 等组件。

本规范约束 Kotlin 写法和新增文件的语言边界，不改变现有项目的业务架构、基类体系、资源组织方式和组件选型。

本规范与以下通用规则同时生效：

- `E:\rules\handwritten-style-general.md`
- 当前项目已有的人工代码风格
- 当前模块使用的基类、工具类和公共组件约束

如规范之间存在冲突，按以下优先级处理：

1. 编译正确和业务正确
2. Android 平台及当前项目基类约束
3. 本规范中的文件语言强制规则
4. 当前模块已有代码风格
5. 本规范中的 Kotlin 推荐写法
6. 通用手写风格规则

# 文件语言规则

本节为强制规则，优先于后续 Kotlin 推荐写法和当前模块的语言使用习惯。

## 现有文件

- 严禁将任何现有 `.java` 文件重命名、迁移或重写为 `.kt` 文件。
- 修改现有 Java 文件时必须继续使用 Java，只做当前需求需要的修改。
- 不以“统一语言”“Kotlin 化”或“减少样板代码”为理由转换 Java 文件。
- Java 与 Kotlin 必须在同一项目中正常共存，不要求按语言拆分模块或建立平行包结构。

## 新增文件

- 创建新的代码文件时优先使用 Kotlin。
- `dao`、`entitys` 包及其子包中的代码文件必须使用 Java，严禁创建 `.kt` 文件。
- 文件语言以实际 `package` 归属为准，不能通过更换目录或新建近似包名绕过限制。
- 除 `dao`、`entitys` 外，如果目标模块已有明确且必须保持的 Java 实现约束，继续使用 Java。
- 新增 Kotlin 文件继续放在模块现有源码目录中，不单独创建 Kotlin 专用源码树。

# Room Entity 项目约束

- Room Entity 只使用无参数的 `@Entity` 注解，不设置 `tableName`，数据库表名使用 Entity 类名。
- Room Entity 不使用 `@NonNull`、`@Nullable`，字段、构造方法参数、getter/setter 均不添加这两个注解。
- Dao 的 SQL 表名必须与 Entity 类名一致，不能继续引用已移除的自定义表名。

# Kotlin 基础规范

## Kotlin 与项目版本

- Kotlin 版本以项目根目录配置为准，不在业务需求中擅自升级。
- 当前项目使用 Kotlin `1.6.21` 和 JVM `1.8`。
- 不因为使用 Kotlin 就自动引入 Compose、Flow、Hilt、Navigation、MVI 等新框架。
- 只有当前模块已经使用协程，或用户明确要求时，才继续使用协程实现对应功能。

## `val` 与 `var`

- 默认使用 `val`。
- 只有变量确实需要重新赋值时才使用 `var`。
- 集合内容可变但引用不变时，使用 `val list = mutableListOf<XxxEntity>()`。
- 不要为了“统一”把所有成员变量都声明为 `var`。
- 不要为了追求不可变而创建大量临时对象，按业务实际需要选择。

```kotlin
private val list = mutableListOf<XxxEntity>()
private var currentIndex = 0
```

## 属性声明

- Kotlin 属性自带 getter/setter，不要手写 Java 风格的访问器。
- Kotlin 页面状态和普通业务对象直接使用属性表达；`entitys` 包中的 Entity 按 Java 规范编写 getter/setter。
- 私有成员使用 `private`。
- 不使用无意义的 `m` 前缀；但已有基类字段如 `mContext`、`mDatas` 必须沿用。

```kotlin
private var selectedType = TYPE_DEFAULT
```

## 空安全

- 根据数据真实来源决定是否可空，不要机械地把所有属性声明为可空。
- 用户输入、Intent 参数、网络响应、数据库可空列和 Java 平台类型必须按需处理。
- 内部流程已经保证非空的数据，不要层层重复判空。
- 优先使用 `orEmpty()`、`?.let {}`、Elvis 运算符和提前返回。
- 禁止随意使用 `!!`。
- 只有生命周期和调用链能严格证明非空，且无法通过更清晰的类型声明表达时，才允许使用 `!!`。
- 不要连续嵌套多层 `let`、`run`、`also`，简单 `if` 更清楚时直接使用 `if`。

```kotlin
val name = intent.getStringExtra("name").orEmpty()

val entity = dao.queryById(id) ?: return

imageUrl?.takeIf { it.isNotBlank() }?.let {
    Glide.with(this).load(it).into(binding.ivCover)
}
```

## `lateinit` 与延迟初始化

- Activity、Fragment、自定义 Dialog 中由生命周期初始化的非空对象可使用 `lateinit var`。
- 使用前必须能由生命周期顺序保证已经初始化。
- 不要用 `lateinit` 掩盖本应可空的业务状态。
- 只读且首次使用时才创建的对象可使用 `by lazy`。
- 简单对象不要为了展示 Kotlin 特性强行使用 `lazy`。

```kotlin
private lateinit var adapter: XxxAdapter

private val dao by lazy {
    DatabaseManager.getInstance(applicationContext).getXxxDao()
}
```

## 集合类型

- 只读参数和只读返回值优先使用 `List<T>`。
- 需要增删改时使用 `MutableList<T>`。
- 调用 Java 基类构造函数时，以基类真实签名为准；`BaseRecylerAdapter` 若要求 `MutableList<T>?`，按其签名传入。
- 不要在同一段逻辑中无意义地反复调用 `toList()`、`toMutableList()`。

## 函数写法

- 简单返回值可以使用表达式函数。
- 业务逻辑较长或包含分支、副作用时使用普通函数体。
- 不要把两三行且只使用一次的逻辑拆成辅助函数。
- 单一业务文件中允许使用 `load()`、`save()`、`delete()` 等短方法名。
- 对外暴露、跨模块调用或容易歧义的方法必须使用完整业务名称。

```kotlin
override fun onSetLayoutId(): Int = R.layout.fra_xxx

private fun hasData(): Boolean = list.isNotEmpty()
```

## 静态成员与 Java 互操作

- Kotlin 中使用 `companion object` 替代 Java `static`。
- 需要让 Java 代码以静态方式调用时添加 `@JvmStatic`。
- 需要暴露静态字段给 Java 时，按实际需要使用 `const val` 或 `@JvmField`。
- 不要无目的地给所有 companion 方法添加 `@JvmStatic`。
- 调用 Java SAM 接口时优先使用 Lambda；包含多个方法的接口使用 `object : Interface {}`。

```kotlin
companion object {
    @JvmStatic
    fun newInstance(): XxxFragment = XxxFragment()
}
```

## Java 平台类型

项目包含大量 Java 基类和公共组件。调用 Java API 时：

- 在边界处尽早明确可空性。
- Java 返回的字符串用于展示时可使用 `orEmpty()`。
- Java 返回的 Entity 可能为空时，先赋给可空变量并判断。
- 不要对未知 Java 返回值直接使用 `!!`。
- Java 方法名、基类字段名和历史拼写必须保持原样，例如 `BaseRecylerAdapter`、`setOnItemClickLitener()`。

## Kotlin 语法克制

- 不为简单页面创建无必要的扩展函数、密封类、委托、DSL 或操作符重载。
- 不使用复杂作用域函数链制造“简洁感”。
- 不把普通回调全部包装为高阶函数，已有 Java 接口或项目公共接口能解决时优先复用。

# Activity 开发规范

## 基类使用

- 所有 Activity 必须继承 `BaseActivity<Binding, Presenter>()`。
- 第一个泛型参数是对应的 DataBinding 类，如 `ActivityContactListBinding`。
- 第二个泛型参数是 Presenter，普通页面通常使用 `BasePresenter`。
- 需要网络请求时使用对应的 `MainContract.Presenter`，并实现 `MainContract.View`。

## 生命周期与方法顺序

方法默认按以下顺序组织：

1. `onCreate()`
2. `initView()`
3. `bindEvent()`
4. `onClickCallback()`
5. Presenter 或接口回调
6. 当前页面私有业务方法

```kotlin
class XxxActivity : BaseActivity<ActivityXxxBinding, BasePresenter>() {

    private lateinit var adapter: XxxAdapter

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setDataBindingLayout(R.layout.activity_xxx)
    }

    override fun initView() {
        binding.include.setTitleStr("标题")

        adapter = XxxAdapter(this, null, R.layout.item_xxx)
        binding.rv.layoutManager = LinearLayoutManager(this)
        binding.rv.addItemDecoration(
            SimplePaddingDecoration(this, SizeUtils.dp2px(0f))
        )
        binding.rv.adapter = adapter

        getData()
    }

    override fun bindEvent() {
        binding.setOnClickListener(this::onClickCallback)
    }

    override fun onClickCallback(view: View) {
        when (view.id) {
            R.id.iv_title_back -> {
                finish()
            }
            R.id.tv_title_right -> {
                saveData()
            }
        }
    }

    private fun getData() {
        // 按数据库异步规范加载数据
    }

    private fun saveData() {
        // 按表单验证和数据库异步规范保存数据
    }
}
```

## 布局设置

- Activity 必须在 `onCreate()` 中调用 `setDataBindingLayout(R.layout.activity_xxx)`。
- 不要把 `setDataBindingLayout()` 移到 `initView()`。
- 不使用 `setContentView()` 绕过现有基类初始化流程。

## 工具栏初始化

- 使用 `binding.include.setTitleStr("标题")` 设置标题。
- 如果当前项目同类页面普遍使用 Kotlin 属性语法，`binding.include.titleStr = "标题"` 也可沿用。
- 标题栏统一复用 `layout_title_bar`，不要另建重复标题栏。

## 点击事件

- DataBinding 的点击绑定方式遵循“DataBinding 点击变量”章节。
- 点击事件集中在 `onClickCallback()` 中处理。
- Kotlin 中优先使用 `when (view.id)`。
- 简单分支也允许使用 `if/else`，以当前文件风格为准。

## RecyclerView 设置

线性列表：

```kotlin
adapter = XxxAdapter(this, null, R.layout.item_xxx)
binding.rv.layoutManager = LinearLayoutManager(this)
binding.rv.addItemDecoration(
    SimplePaddingDecoration(this, SizeUtils.dp2px(0f))
)
binding.rv.adapter = adapter
```

网格列表：

```kotlin
adapter = XxxAdapter(this, null, R.layout.item_xxx)
binding.rv.layoutManager = GridLayoutManager(this, 2)
binding.rv.addItemDecoration(
    GridSpacesItemDecoration(2, SizeUtils.dp2px(8f), false)
)
binding.rv.adapter = adapter
```

要求：

- 必须设置 `LayoutManager`。
- 按项目页面风格设置对应的 `ItemDecoration`。

## 数据加载

- 页面初始化数据默认在 `initView()` 中调用 `getData()`。
- 数据库加载遵循“数据库与 RxJava3 异步规范”，网络加载遵循“网络请求规范”。

# Fragment 开发规范

## 基类使用

- 所有 Fragment 必须继承 `BaseFragment<Binding, Presenter>()`。
- 第一个泛型参数是对应 Binding 类。
- 第二个泛型参数通常为 `BasePresenter` 或具体 Presenter。
- 继续使用基类提供的 `binding`，不要重复创建 `_binding`，除非目标模块的 BaseFragment 明确要求该模式。

## 实例创建

无参数 Fragment 使用 `newInstance()`：

```kotlin
class XxxFragment : BaseFragment<FraXxxBinding, BasePresenter>() {

    companion object {
        @JvmStatic
        fun newInstance(): XxxFragment = XxxFragment()
    }

    override fun onSetLayoutId(): Int = R.layout.fra_xxx

    override fun initView() {
        // 初始化视图
    }

    override fun bindEvent() {
        binding.setOnClickListener(this::onClickCallback)
    }

    override fun onClickCallback(view: View) {
        when (view.id) {
            R.id.iv_title_back -> {
                activity?.finish()
            }
            R.id.tv_title_right -> {
                saveData()
            }
        }
    }

    private fun saveData() {
        // 保存业务数据
    }
}
```

需要参数时通过 `arguments` 传递，不直接依赖可丢失的 Fragment 构造参数：

```kotlin
companion object {
    private const val ARG_ID = "arg_id"

    @JvmStatic
    fun newInstance(id: Long): XxxFragment {
        return XxxFragment().apply {
            arguments = Bundle().apply {
                putLong(ARG_ID, id)
            }
        }
    }
}
```

## 必须实现的方法

- `onSetLayoutId()`
- `initView()`
- `bindEvent()`
- 基类要求的 `onClickCallback()`

## 生命周期安全

- 只在 Binding 有效的生命周期内访问 `binding`。
- 异步回调返回时，如果 Fragment 已脱离 Activity，不执行依赖 Activity 的 UI 操作。
- 需要 Context 时按实际生命周期使用 `requireContext()` 或 `context ?: return`。
- 不在无把握的异步回调里直接使用 `requireActivity()`。

# Adapter 开发规范

## 基类使用

- 所有 RecyclerView Adapter 必须继承 `BaseRecylerAdapter<Entity>`。
- 必须实现 `convert(holder, position)`。
- 继续使用基类的 `MyRecylerViewHolder` 和 `mDatas`。
- 不擅自替换为新的 Adapter 基类、ListAdapter 或 Paging，除非用户明确要求。

## 标准结构

```kotlin
class XxxListAdapter(
    private val context: Context,
    list: MutableList<XxxEntity>?,
    layoutId: Int
) : BaseRecylerAdapter<XxxEntity>(context, list, layoutId) {

    override fun convert(holder: MyRecylerViewHolder, position: Int) {
        val entity = mDatas[position]

        holder.setText(R.id.tv_name, entity.name.orEmpty())
        Glide.with(context)
            .load(entity.imageUrl)
            .into(holder.getImageView(R.id.iv_cover))
    }
}
```

如果当前同类 Adapter 使用次构造函数，可保持项目现有写法，不强制批量改为主构造函数。

## 数据绑定

- 使用 `holder.setText()`、`holder.getTextView()`、`holder.getImageView()`、`holder.getView()` 等现有方法。
- 使用 `mDatas[position]` 获取当前数据。
- 外部刷新数据使用 `adapter.addAllAndClear(list)`，不直接修改 Adapter 的 `mDatas`。
- UI 展示时按实体真实空值情况使用 `orEmpty()`。
- 不在 `convert()` 中执行数据库、网络或复杂业务操作。
- Adapter 只负责列表展示、局部 UI 状态和按钮回调。

## Item 点击监听

`BaseRecylerAdapter` 已提供 Item 点击和长按监听：

- Item 点击使用 `setOnItemClickLitener()`。
- Item 长按使用 `setOnLongItemClickLitener()`。
- 监听必须在 Activity、Fragment 或 Dialog 等外部设置。
- 不要在 Adapter 构造函数中设置 Item 监听。
- 不要为整个 Item 重复定义 `OnItemClickListener`。
- Java 基类回调中的 `data` 如果被推断为 `Any`，在外部按具体类型转换。

```kotlin
adapter = XxxListAdapter(this, null, R.layout.item_xxx)

adapter.setOnItemClickLitener { _, position, data ->
    val entity = data as XxxEntity
    handleItemClick(entity, position)
}

adapter.setOnLongItemClickLitener { _, position ->
    delete(adapter.getItem(position))
}
```

如果 Kotlin 能正确推断 `data` 为 `XxxEntity`，不要做多余强制转换。

## Item 内部按钮点击

Item 中存在删除、编辑等多个按钮时，可使用项目已有 `ButtonClickListener<T>`，或按按钮业务定义专用接口。

优先复用 `ButtonClickListener<T>`：

```kotlin
class XxxAdapter(
    context: Context,
    list: MutableList<XxxEntity>?,
    layoutId: Int
) : BaseRecylerAdapter<XxxEntity>(context, list, layoutId) {

    private var buttonClickListener: ButtonClickListener<XxxEntity>? = null

    fun setButtonClickListener(listener: ButtonClickListener<XxxEntity>?) {
        buttonClickListener = listener
    }

    override fun convert(holder: MyRecylerViewHolder, position: Int) {
        val entity = mDatas[position]

        holder.getView<View>(R.id.btn_action).setOnClickListener { view ->
            buttonClickListener?.onButtonClick(view, position, entity)
        }
    }
}
```

需要语义明确的专用接口时：

```kotlin
fun interface OnDeleteClickListener {
    fun onDeleteClick(entity: XxxEntity, position: Int)
}

private var deleteClickListener: OnDeleteClickListener? = null

fun setOnDeleteClickListener(listener: OnDeleteClickListener?) {
    deleteClickListener = listener
}
```

要求：

- Adapter 内部只触发回调，不直接执行页面跳转、数据库删除等业务。
- 回调为空时使用安全调用，不使用 `!!`。
- 简单 Item 点击继续使用基类监听，不创建重复接口。

# Entity 开发规范

## Room 数据库实体

```java
@Entity
public class XxxEntity implements Serializable {
    
    @PrimaryKey(autoGenerate = true)
    private long id;
    
    private String name;
    private long createTime;  // 创建时间，格式为时间戳
    
    // getter/setter 方法
    public long getId() { return id; }
    public void setId(long id) { this.id = id; }
    
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    
    // ... 其他 getter/setter
}
```

## Entity 规范

- 使用 `@Entity` 注解
- 使用 `@PrimaryKey(autoGenerate = true)` 标记自增主键
- 实现 `Serializable` 接口
- 字段使用 `private` 修饰，提供 getter/setter 方法
- 时间字段使用 `long` 类型，格式为时间戳

## 非 Room 实体类规范

对于不需要存储到数据库的普通实体类，遵循以下规范：

- 实现 `Serializable` 接口（便于传递）
- 字段使用 `private` 修饰，提供 getter/setter 方法
- 空值处理由调用方（如 Adapter、UI 层）按需处理

```java
public class XxxEntity implements Serializable {
    private long id;
    private String name;
    
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
}
```

# Dao 开发规范

## Room DAO 接口

```java

@Dao
public interface XxxDao {
    
    @Insert(onConflict = OnConflictStrategy.REPLACE)  
	void insert(XxxEntity... beans);  
  
	@Insert(onConflict = OnConflictStrategy.REPLACE)  
	void insert(List<XxxEntity> list);
    
    @Insert(onConflict = OnConflictStrategy.REPLACE)
    void insertOrReplace(XxxEntity... entities);
    
    @Query("SELECT * FROM XxxEntity ORDER BY id DESC")
    List<XxxEntity> queryAll();
    
    @Query("SELECT COUNT(*) FROM XxxEntity")  
	int queryCount();  
  
	@Query("SELECT * FROM XxxEntity WHERE id = :id")  
	XxxEntity queryById(long id);
    
    @Update
    void update(XxxEntity... entities);
    
    @Delete
    void delete(XxxEntity... entities);
}
```

## Dao 使用

- 使用 `DatabaseManager.getInstance(context).getXxxDao()` 获取 Dao 实例

- 查询方法使用 `@Query` 注解

- 插入使用 `insert` 方法，使用 `OnConflictStrategy.REPLACE`

- 查询结果按 `id DESC` 排序（最新的在前）

# Room 数据库配置规范

## 开发阶段说明

当前项目处于开发阶段，数据结构变更时可通过卸载 App 清除旧数据后重新安装，不强制处理数据库迁移。

如果用户明确要求保留线上数据、升级正式数据库版本或发布迁移方案，则必须实现并验证 Migration，不能继续使用卸载清库方案。

## Gradle 配置

模块使用 Groovy Gradle 时：

```groovy
plugins {
    id 'com.android.application'
    id 'kotlin-android'
}

android {
    buildFeatures {
        dataBinding = true
        viewBinding true
    }

    kotlinOptions {
        jvmTarget = '1.8'
    }
}

dependencies {
    implementation androidApi.library.room
    annotationProcessor androidApi.library.roomprocessor
    implementation androidApi.library.roomRxjava3
}
```

要求：

- 当前 Java/Kotlin 混合模块的 Room 注解处理器统一使用 `annotationProcessor`。
- DataBinding 和 ViewBinding 开关继续沿用项目配置。

## DatabaseManager 实现

数据库管理类采用单例模式设计，统一提供数据库实例访问入口，便于全局调用。需注意在实际使用时，将泛型占位符及数据库名称替换为项目真实信息。

```java

@Database(entities = {XxxEntity.class, YyyEntity.class}, version = 1, exportSchema = false)
public abstract class DatabaseManager extends RoomDatabase {

    // 数据库名称，可根据业务需求修改
    public static final String DB_NAME = "data.db";

    // 单例实例，使用 volatile 保证多线程可见性
    private volatile static DatabaseManager instance;

    /**
     * 获取数据库单例实例
     * @param context 上下文，建议使用 Application 上下文避免内存泄漏
     * @return 数据库实例
     */
    public static DatabaseManager getInstance(Context context) {
        if (instance == null) {
            synchronized (DatabaseManager.class) {
                if (instance == null) {
                    instance = DatabaseManager.create(context);
                }
            }
        }
        return instance;
    }

    /**
     * 创建数据库实例
     */
    private static DatabaseManager create(final Context context) {
        return Room.databaseBuilder(
                context.getApplicationContext(),
                DatabaseManager.class,
                DB_NAMA).allowMainThreadQueries().build();
    }
    
    // 必须为每个 Dao 接口提供抽象的 getter 方法
    public abstract XxxDao getXxxDao();
    public abstract YyyDao getYyyDao();

}
```

## 使用要点

- `@Database` 注解中 `entities` 参数需包含所有 Room 实体类，用逗号分隔

- **允许使用 `allowMainThreadQueries()`**，同时也使用 **RxJava3** 进行异步数据库操作（项目已集成 RxJava3）

- 获取实例时建议传入 `Application` 上下文，防止因 Activity 上下文销毁导致的内存泄漏

- 每个 Dao 接口都需在 DatabaseManager 中定义对应的抽象 getter 方法，如 `getXxxDao()`

# 数据库与 RxJava3 异步规范

## 唯一标准

1. Dao 只返回普通类型。
2. UI 层使用 `Observable.create()` 包裹 Dao 操作。
3. 数据库操作使用 `subscribeOn(Schedulers.io())`。
4. UI 回调使用 `observeOn(AndroidSchedulers.mainThread())`。
5. 数据库异常传递给 `onError()`，并给用户友好提示。
6. `Observable.create()` 必须正确发送 `onNext()` 和 `onComplete()`，避免保存成功后没有回调。

## 查询标准写法

```kotlin
private fun getData() {
    Observable.create<List<XxxEntity>> { emitter ->
        try {
            val list = DatabaseManager
                .getInstance(applicationContext)
                .getXxxDao()
                .queryAll()

            emitter.onNext(list)
            emitter.onComplete()
        } catch (e: Exception) {
            emitter.tryOnError(e)
        }
    }
        .subscribeOn(Schedulers.io())
        .observeOn(AndroidSchedulers.mainThread())
        .subscribe(object : SimpleObserver<List<XxxEntity>>() {
            override fun onNext(list: List<XxxEntity>) {
                adapter.addAllAndClear(list)
                binding.tvEmpty.visibility =
                    if (adapter.itemCount > 0) View.GONE else View.VISIBLE
            }

            override fun onError(e: Throwable) {
                super.onError(e)
                ToastUtils.showShort("数据加载失败")
            }
        })
}
```

## 保存标准写法

```kotlin
private fun saveData() {
    val name = binding.etName.text.toString().trim()
    if (name.isBlank()) {
        ToastUtils.showShort("名称不能为空")
        return
    }

    val entity = XxxEntity()
    entity.name = name

    Observable.create<Unit> { emitter ->
        try {
            DatabaseManager
                .getInstance(applicationContext)
                .getXxxDao()
                .insert(entity)

            emitter.onNext(Unit)
            emitter.onComplete()
        } catch (e: Exception) {
            emitter.tryOnError(e)
        }
    }
        .subscribeOn(Schedulers.io())
        .observeOn(AndroidSchedulers.mainThread())
        .subscribe(object : SimpleObserver<Unit>() {
            override fun onNext(value: Unit) {
                ToastUtils.showShort("保存成功")
                finish()
            }

            override fun onError(e: Throwable) {
                super.onError(e)
                ToastUtils.showShort("保存失败")
            }
        })
}
```

## 删除标准写法

删除前按“确认对话框”章节显示确认弹窗，用户确认后再执行异步删除。

异步删除：

```kotlin
private fun deleteData(entity: XxxEntity) {
    Observable.create<Unit> { emitter ->
        try {
            DatabaseManager
                .getInstance(applicationContext)
                .getXxxDao()
                .delete(entity)

            emitter.onNext(Unit)
            emitter.onComplete()
        } catch (e: Exception) {
            emitter.tryOnError(e)
        }
    }
        .subscribeOn(Schedulers.io())
        .observeOn(AndroidSchedulers.mainThread())
        .subscribe(object : SimpleObserver<Unit>() {
            override fun onNext(value: Unit) {
                ToastUtils.showShort("删除成功")
                getData()
            }

            override fun onError(e: Throwable) {
                super.onError(e)
                ToastUtils.showShort("删除失败")
            }
        })
}
```

## 异步注意事项

- 不在主线程直接调用 Dao。
- 不在 Dao 内部切线程。
- 不用 `Observable.just(dao.queryAll())` 包裹查库，因为参数会在创建 Observable 时立即执行。
- 使用 `Observable.create()` 时必须捕获数据库异常并调用 `tryOnError()`。
- 保存、删除等无业务返回值操作可使用 `Unit` 作为事件。
- 页面销毁后的订阅管理沿用当前项目既有方式；不要在单个需求中擅自引入另一套生命周期框架。
- 如果当前基类已经统一管理 Disposable，必须复用基类能力。

# 工具类使用规范

## Toast

统一使用项目指定的 Toast：

```kotlin
ToastUtils.showShort("操作成功")
```

优先导入：

```kotlin
import com.blankj.utilcode.util.ToastUtils
```

如果当前模块统一封装了同名 `ToastUtils`，以当前模块实际导入为准，不混用不同实现。

## 确认对话框

删除等不可逆操作默认使用：

```kotlin
DialogUtil.showConfirmRreceiptDialog(
    this,
    "标题",
    "确认要执行此操作吗？",
    object : ConfirmDialog.OnDialogClickListener {
        override fun confirm() {
            // 执行已确认的操作
        }

        override fun cancel() {
        }
    }
)
```

需要更多配置时：

```kotlin
ConfirmDialog.Builder(this)
    .setTitleName("标题")
    .setMessage("确认要删除吗？")
    .setConfirmBtn("确定")
    .setCancelBtn("取消")
    .setOnDialogClickListener(
        object : ConfirmDialog.OnDialogClickListener {
            override fun confirm() {
                // 执行删除
            }

            override fun cancel() {
            }
        }
    )
    .create(true)
    .show()
```

## 时间工具

- `VTBTimeUtils.formatDateTime(System.currentTimeMillis())`：格式化时间戳。
- `VTBTimeUtils.getCurrentTime()`：获取 `yyyy-MM-dd` 格式日期。
- `VTBTimeUtils.strToDate(dateStr)`：字符串转 Date。

## 尺寸转换

```kotlin
SizeUtils.dp2px(12f)
```

优先复用项目已有工具，不重复创建 `dpToPx()` 扩展函数。

# 网络请求规范

## Presenter 模式

需要网络请求的 Activity：

1. 继承 `BaseActivity<Binding, MainContract.Presenter>()`。
2. 实现 `MainContract.View`。
3. 在 `initView()` 中初始化 Presenter。
4. 使用现有 Presenter 方法发起请求。
5. 在 View 回调中处理结果。

## 标准结构

```kotlin
class XxxListActivity :
    BaseActivity<ActivityXxxListBinding, MainContract.Presenter>(),
    MainContract.View {

    private lateinit var adapter: XxxAdapter
    private var dataUrl = ""

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setDataBindingLayout(R.layout.activity_xxx_list)
    }

    override fun initView() {
        binding.include.setTitleStr("标题")

        dataUrl = intent.getStringExtra("dataUrl").orEmpty()

        adapter = XxxAdapter(this, null, R.layout.item_xxx)
        binding.rv.layoutManager = LinearLayoutManager(this)
        binding.rv.addItemDecoration(
            SimplePaddingDecoration(this, SizeUtils.dp2px(0f))
        )
        binding.rv.adapter = adapter

        presenter = MainPresenter(this)
        presenter.queryJson(dataUrl)
    }

    override fun bindEvent() {
        binding.setOnClickListener(this::onClickCallback)
    }

    override fun onClickCallback(view: View) {
        when (view.id) {
            R.id.iv_title_back -> {
                finish()
            }
        }
    }

    override fun queryJsonSuccess(requestUrl: String, jsonStr: String) {
        val type = object : TypeToken<List<XxxEntity>>() {}.type
        val list: List<XxxEntity> = Gson().fromJson(jsonStr, type)
        showList(list)
    }

    override fun hideLoading() {
        hideLoadingDialog()
    }

    private fun showList(list: List<XxxEntity>) {
        adapter.addAllAndClear(list)
        binding.tvEmpty.visibility =
            if (adapter.itemCount > 0) View.GONE else View.VISIBLE
    }
}
```

## 网络规则

- 不因使用 Kotlin 就绕过现有 Presenter 层直接在 Activity 创建 Retrofit。
- Presenter 初始化和调用方式沿用当前项目。
- Gson 泛型解析使用匿名 `TypeToken`。
- Java 返回的可空字符串在边界处处理。
- 网络错误提示和 Loading 显隐复用现有基类。
- 不在接口成功回调中执行复杂耗时计算；耗时逻辑切换到后台线程。

# 代码风格规范

## 命名规范

- Activity：`XxxActivity`、`XxxListActivity`、`AddXxxActivity`
- Fragment：`XxxFragment`
- Adapter：`XxxAdapter`、`XxxListAdapter`
- Entity：`XxxEntity`
- Dao：`XxxDao`
- Dialog：`XxxDialog`、`XxxSelectDialog`

## 变量命名

- 使用小驼峰命名。
- Adapter 默认命名 `adapter`。
- 单一列表默认命名 `list`。
- Binding 固定使用 `binding`。
- Context 使用 `context`；基类已有 `mContext` 时直接复用。
- 布尔值优先使用 `is`、`has`、`can`、`need` 前缀。
- 不创建 `EnhancedXxx`、`OptimizedXxx`、`UnifiedXxx` 等机器感命名。

## 常量

```kotlin
companion object {
    private const val TYPE_DEFAULT = "0"
    private const val REQUEST_CODE_SELECT = 1001
}
```

- 类内常量使用 `private const val`。
- 多处共享的常量复用项目现有常量类。
- 不为只使用一次的简单字符串创建常量。
- Intent Key 在多个入口复用时应声明为常量。

## 代码组织

- 相关方法放在一起。
- Activity 的方法顺序遵循“生命周期与方法顺序”章节，其他类型参考当前模块同类文件。
- 不用大量区域注释把文件切成模板化区块。
- 不为了绝对顺序移动无关旧代码。
- 修改旧文件时保持原有书写节奏。

## 导入规范

- 使用 import 后直接写类名。
- 只有同名类冲突时才使用全限定名。
- 不使用 `*` 通配导入，除非项目自动格式化规则明确允许。
- 删除本次修改产生的未使用 import。
- 不为了整理 import 修改无关代码。

## 注释规范

- 新增注释必须使用中文。
- 注释解释“为什么”，不解释显而易见的语句。
- 复杂业务规则、兼容逻辑、线程要求、数据库约束和第三方库坑点需要注释。
- 不写“设置标题”“遍历列表”“点击按钮”等无价值注释。
- 不添加模板化作者、日期和版本注释，除非项目明确要求。

## Kotlin 格式

- 使用 Android Studio/Kotlin 默认格式作为基础。
- `when` 的每个分支都必须使用 `{}` 包裹，即使分支内只有一行代码。
- 链式调用较长时按调用层级换行。
- Lambda 简短时放在调用处，过长时提取有业务含义的方法。
- 不用分号。
- 不故意制造格式不一致来模拟手写感。
- 不顺手格式化整个旧文件。

# DataBinding / ViewBinding 规范

## 使用原则

- 使用现有 `BaseActivity`、`BaseFragment` 的页面默认沿用 DataBinding。
- DataBinding 布局根标签使用 `<layout>`。
- ViewBinding 只用于当前模块已经明确采用 ViewBinding 的页面。
- 同一页面不要同时创建 DataBinding 和 ViewBinding 两套 Binding。
- 不使用 Kotlin synthetic。
- 所有视图通过 `binding.xxx` 访问。

## DataBinding 点击变量

布局中定义：

```xml
<data>

    <variable
        name="onClickListener"
        type="android.view.View.OnClickListener" />

</data>
```

Activity 或 Fragment 中绑定：

```kotlin
binding.setOnClickListener(this::onClickCallback)
```

不要使用：

```kotlin
binding.onClickListener = this::onClickCallback
```

## ViewBinding

如果目标页面按模块约定使用 ViewBinding：

- Binding 创建和销毁必须符合 Activity/Fragment 生命周期。
- Fragment 必须在 `onDestroyView()` 后清理对 View 的引用。
- 如果模块基类已经封装 ViewBinding 生命周期，直接复用基类。
- 不为已有 DataBinding 基类页面另建 ViewBinding 初始化逻辑。

# 布局文件规范

## 文件组织

- Activity：`activity_xxx.xml`
- Fragment：`fra_xxx.xml`
- RecyclerView Item：`item_xxx.xml`，存放在 `res/layout/list/layout/`
- Dialog：`dialog_xxx.xml`，存放在 `res/layout/dialog/layout/`
- 自定义 View：`view_xxx.xml`，存放在 `res/layout/view/layout/`

## Activity 布局示例

```xml
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">

    <data>

        <variable
            name="onClickListener"
            type="android.view.View.OnClickListener" />

    </data>

    <androidx.constraintlayout.widget.ConstraintLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:background="@color/base_bg_color">

        <include
            android:id="@+id/include"
            layout="@layout/layout_title_bar"
            android:onClickListener="@{onClickListener}"
            app:layout_constraintEnd_toEndOf="parent"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintTop_toTopOf="parent" />

        <androidx.recyclerview.widget.RecyclerView
            android:id="@+id/rv"
            android:layout_width="match_parent"
            android:layout_height="0dp"
            app:layout_constraintBottom_toBottomOf="parent"
            app:layout_constraintTop_toBottomOf="@id/include" />

    </androidx.constraintlayout.widget.ConstraintLayout>

</layout>
```

任意 View 点击：

```xml
<TextView
    android:id="@+id/tv_confirm"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:textColor="@color/base_text_color"
    android:onClickListener="@{onClickListener}" />
```

## Android XML 布局编写规则

- 页面必须优先参考项目中已经调整完成的同类布局，保持现有层级、命名、间距和属性写法，不得自行套用通用模板。
- `<layout>` 下的根布局优先使用 `ConstraintLayout`。
- 滚动页面使用 `NestedScrollView + ConstraintLayout`。`NestedScrollView` 下只保留一个直接子布局，禁止增加无业务意义的中间层级。
- 布局属性遵循最小化原则。没有明确裁剪、越界绘制或 padding 绘制需求时，禁止主动添加 `clipChildren`、`clipToPadding`；`rowCount`、`useDefaultMargins`、`alignmentMode` 等属性也只在实际需要时设置。
- `ConstraintLayout` 的直接子节点自行设置外边距和约束；内部子节点只负责自身内容间距，禁止通过额外父布局重复传递边距。
- `GridLayout` 子项应明确设置 `layout_row`、`layout_column`、`layout_columnWeight` 和必要边距。宽度使用 `0dp` 配合权重，禁止复制多套相同 View 结构。
- `GridLayout` 中重复出现的 View 必须提取到 `res/layout/view/layout/` 下的独立布局，通过 `include` 复用。复用布局只保留公共视觉结构，行列、权重、边距、ID 和点击监听由主布局中的 `include` 设置。
- `include` 需要点击时，直接在 `include` 标签设置 `android:onClickListener="@{onClickListener}"`，禁止在被 include 的布局中重复声明点击变量或设置点击监听。
- 复用布局内部的 ID 根据局部上下文简洁命名，例如 `tv_name`、`tv_desc`、`iv_icon`，禁止重复添加 `grade`、`special` 等布局文件本身已经表达的业务前缀。
- 各 include 的差异内容统一在 Activity/Fragment 中通过 DataBinding 设置，禁止为了不同文字或图标复制布局，也禁止使用 `findViewById()`。
- 单选切换使用 `RadioGroup + RadioButton`。选中背景和文字颜色优先使用 selector，Activity/Fragment 只处理布局显隐和业务状态，禁止手动重复设置背景及文字颜色。
- `EditText` 和 `TextView` 必须设置 `android:textColor`；使用 `hint` 属性时必须设置 `android:textColorHint`。
- `ImageView` 不设置 `android:contentDescription`；使用 tint 属性时设置 `app:tint`，不要使用 `android:tint`。
- 资源 ID 使用带控件类型前缀的小写下划线命名。
- 图标以 `ic_` 开头，避免重复添加页面名和 `icon` 等冗余词。
- 大插图可使用 `_art` 后缀区分。
- 除非设计明确要求，`ImageView` 和 `TextView` 使用 `wrap_content`。
- 文本默认不指定字体和字体边距。
- 容器不随意固定高度或设置统一外边距，间距按设计由内部控件控制。
- 相同样式优先复用已有 drawable，禁止重复创建近似资源。
- 圆角背景根据圆角尺寸复用对应 drawable。
- `shape_bg_white_xx` 固定为白色背景，需要其他颜色时使用 `backgroundTint`。
- 所有布局实现以 KISS 为原则，只保留当前页面真正需要的层级、属性和代码，不主动增加预防性配置。
- XML 规则与代码语言无关，Kotlin 项目继续完全沿用。

# 圆角图片规范

Glide 的 `RoundedCorners` 在当前项目中不生效，圆角图片必须使用 `RoundedImageView`。

## 依赖

```groovy
api 'com.makeramen:roundedimageview:2.3.0'
```

## XML

```xml
<com.makeramen.roundedimageview.RoundedImageView
    android:id="@+id/iv_cover"
    android:layout_width="match_parent"
    android:layout_height="120dp"
    android:scaleType="centerCrop"
    app:riv_corner_radius="8dp"
    app:riv_oval="false" />
```

## Kotlin 加载

```kotlin
Glide.with(context)
    .load(imageUrl)
    .into(binding.ivCover)
```

Adapter 中：

```kotlin
Glide.with(context)
    .load(entity.imageUrl)
    .into(holder.getImageView(R.id.iv_cover))
```

禁止：

```kotlin
val options = RequestOptions()
    .transform(RoundedCorners(24))

Glide.with(context)
    .load(imageUrl)
    .apply(options)
    .into(binding.ivCover)
```

# RadioButton / RadioGroup 规范

- 单选布局结构、选中样式和页面职责遵循“Android XML 布局编写规则”。
- 监听单个 RadioButton 时，只有 `isChecked == true` 才处理。
- 监听 RadioGroup 时根据 `checkedId` 更新状态。
- 不在取消选中回调中重复执行相同业务。

单个 RadioButton：

```kotlin
binding.rbOption1.setOnCheckedChangeListener { _, isChecked ->
    if (isChecked) {
        currentOption = OPTION_1
        updateUi()
    }
}
```

RadioGroup：

```kotlin
binding.rgOption.setOnCheckedChangeListener { _, checkedId ->
    when (checkedId) {
        R.id.rb_option_1 -> {
            currentOption = OPTION_1
        }
        R.id.rb_option_2 -> {
            currentOption = OPTION_2
        }
    }
    updateUi()
}
```

# Activity 注册与页面跳转

## Manifest 注册

新增 Activity 必须在 `AndroidManifest.xml` 注册：

```xml
<activity
    android:name=".ui.xxx.XxxActivity"
    android:screenOrientation="portrait"
    tools:ignore="DiscouragedApi,LockedOrientationActivity" />
```

## 无参数跳转

继续使用基类封装：

```kotlin
skipAct(XxxActivity::class.java)
```

## 携带参数跳转

```kotlin
val intent = Intent(this, XxxActivity::class.java)
intent.putExtra("data", entity)
startActivity(intent)
```

如果入口会被多处调用，可在目标 Activity 中提供 `start()`：

```kotlin
companion object {

    @JvmStatic
    fun start(context: Context, entity: XxxEntity) {
        val intent = Intent(context, XxxActivity::class.java)
        intent.putExtra("data", entity)
        context.startActivity(intent)
    }
}
```

要求：

- Entity 通过 Intent 传递时必须实现 `Serializable`。
- Key 多处使用时定义常量。
- 不为了一个单一入口创建复杂路由封装。
- 从非 Activity Context 启动页面时，按 Android 规则处理 `FLAG_ACTIVITY_NEW_TASK`。

# 自定义 Dialog 开发规范

## 基础方式

自定义弹窗统一参考项目 `PixelStyleDialog` 的实现方式：

## Kotlin 标准结构

```kotlin
class XxxSelectDialog(
    context: Context,
    private val listener: DialogInterface.OnClickListener? = null
) : Dialog(context) {

    private lateinit var binding: DialogXxxSelectBinding

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        val dialogWindow = window ?: return
        dialogWindow.setSoftInputMode(
            WindowManager.LayoutParams.SOFT_INPUT_ADJUST_RESIZE or
                WindowManager.LayoutParams.SOFT_INPUT_STATE_HIDDEN
        )
        dialogWindow.setGravity(Gravity.CENTER)
        dialogWindow.decorView.setPadding(0, 0, 0, 0)
        dialogWindow.setBackgroundDrawable(ColorDrawable(Color.TRANSPARENT))
        dialogWindow.requestFeature(Window.FEATURE_NO_TITLE)

        val params = dialogWindow.attributes
        params.width = ScreenUtils.getScreenWidth() - SizeUtils.dp2px(60f)
        dialogWindow.attributes = params

        binding = DataBindingUtil.inflate(
            LayoutInflater.from(context),
            R.layout.dialog_xxx_select,
            null,
            false
        )
        setContentView(binding.root)

        initView()
        binding.setOnClickListener(this::onItemViewClick)
    }

    private fun initView() {
        // 只初始化当前 Dialog 需要的视图和数据
    }

    private fun onItemViewClick(view: View) {
        when (view.id) {
            R.id.tv_option_1 -> {
                listener?.onClick(this, 0)
            }
            R.id.tv_option_2 -> {
                listener?.onClick(this, 1)
            }
        }

        dismiss()
    }
}
```

需要导入的主要类型：

```kotlin
import android.app.Dialog
import android.content.Context
import android.content.DialogInterface
import android.graphics.Color
import android.graphics.drawable.ColorDrawable
import android.os.Bundle
import android.view.Gravity
import android.view.LayoutInflater
import android.view.View
import android.view.Window
import android.view.WindowManager
import androidx.databinding.DataBindingUtil
import com.blankj.utilcode.util.ScreenUtils
import com.blankj.utilcode.util.SizeUtils
```

## Dialog 布局

布局文件命名为 `dialog_xxx.xml`，存放在 `res/layout/dialog/layout/`。根标签使用 `<layout>`，点击控件统一绑定 `onClickListener`：

```xml
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">

    <data>

        <variable
            name="onClickListener"
            type="android.view.View.OnClickListener" />

    </data>

    <androidx.constraintlayout.widget.ConstraintLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:background="@drawable/shape_dialog_bg">

        <ImageView
            android:id="@+id/iv_close"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:onClickListener="@{onClickListener}"
            app:layout_constraintEnd_toEndOf="parent"
            app:layout_constraintTop_toTopOf="parent" />

        <TextView
            android:id="@+id/tv_option_1"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:textColor="@color/base_text_color"
            android:onClickListener="@{onClickListener}"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintTop_toTopOf="parent" />

        <TextView
            android:id="@+id/tv_option_2"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:textColor="@color/base_text_color"
            android:onClickListener="@{onClickListener}"
            app:layout_constraintEnd_toEndOf="parent"
            app:layout_constraintTop_toTopOf="parent" />

    </androidx.constraintlayout.widget.ConstraintLayout>

</layout>
```

`iv_close` 不需要回调业务结果。它触发 `onItemViewClick()` 后直接执行 `dismiss()`。

## 显示 Dialog

Activity 中：

```kotlin
XxxSelectDialog(this) { _, which ->
    selectItem(which)
}.show()
```

Fragment 中：

```kotlin
XxxSelectDialog(requireContext()) { _, which ->
    selectItem(which)
}.show()
```

## Dialog 实现要求

- `super.onCreate(savedInstanceState)` 必须最先调用。
- `Window` 的位置、宽度、软键盘模式和透明背景按设计图配置。
- 居中弹窗可参考 `PixelStyleDialog`，宽度使用屏幕宽度减去两侧间距。
- Binding 必须使用 `DataBindingUtil.inflate()` 创建，并通过 `setContentView(binding.root)` 设置内容。
- Binding 初始化完成后再调用 `binding.setOnClickListener(this::onItemViewClick)`。
- 选项较少且只需返回索引时，使用 `DialogInterface.OnClickListener`。
- 需要返回 Entity、日期或其他复杂数据时，可定义单一职责的 `fun interface`。
- 点击选项后默认关闭 Dialog；业务要求连续操作时才保留弹窗。
- 不在 Dialog 中执行页面跳转、数据库写入或网络请求。
- 日期、月份等选择界面也按本节方式创建对应的 `XxxSelectDialog`，样式放在 XML 中处理。
- 不为自定义 Dialog 额外引入第三方弹窗框架。

# 表单验证规范

保存前必须先完成同步验证，再执行异步保存。

```kotlin
private fun saveData() {
    val fieldValue = binding.etField.text.toString().trim()
    if (fieldValue.isBlank()) {
        ToastUtils.showShort("请输入必填字段")
        return
    }

    val numberValue = fieldValue.toFloatOrNull()
    if (numberValue == null) {
        ToastUtils.showShort("请输入有效的数值")
        return
    }

    if (numberValue <= 0f) {
        ToastUtils.showShort("数值必须大于0")
        return
    }

    if (someType == CURRENT_TYPE && selectedItem.isNullOrBlank()) {
        ToastUtils.showShort("请选择必选项")
        return
    }

    saveToDatabase()
}
```

要求：

- 字符串必填验证优先使用 `isBlank()`。
- 只需要判断空字符串时可使用 `isEmpty()`。
- Java 兼容代码中允许继续使用 `TextUtils.isEmpty()`，但不要同一方法混用多套空值判断。
- 数字解析优先使用 `toIntOrNull()`、`toLongOrNull()`、`toFloatOrNull()`、`toDoubleOrNull()`。
- 不用异常处理代替 Kotlin 已提供的安全解析。
- 业务范围校验在解析成功后执行。
- 验证失败后给出明确中文提示并立即返回。

# 错误处理规范

## 数据库错误

- 不直接向用户展示完整堆栈或敏感内部信息。
- 调试阶段可使用项目现有日志方式记录异常。

## 文件和图片错误

- 文件读写、Bitmap 转换和第三方图片处理按实际风险捕获异常。
- 捕获范围只包住可能失败的操作，不把整个 Activity 方法放进大 `try/catch`。
- 失败后恢复必要 UI 状态，如关闭 Loading。
- 不返回伪造成功结果。

## 普通内部逻辑

- 项目内部可控数据不层层 `try/catch`。
- 能用类型系统和明确分支表达的情况不使用异常。
- 不为每个安全调用都补一层日志或 Toast。

# 依赖库

## 项目已有依赖

- Kotlin
- DataBinding / ViewBinding
- RecyclerView
- Glide
- Retrofit + RxJava3
- Gson
- AndroidUtilCode
- Android 原生 Dialog
- 项目现有 BaseActivity、BaseFragment、BaseRecylerAdapter、SimpleObserver
- 项目当前模块已引入的协程依赖

## CalendarView

```groovy
implementation 'com.haibin:calendarview:3.7.1'
```

## 图片选择器

```groovy
implementation 'io.github.lucksiege:pictureselector:v3.11.2'
```

## 图片压缩

```groovy
implementation 'io.github.lucksiege:compress:v3.11.2'
```

## 依赖规则

- 先确认项目已有依赖，禁止重复添加。
- 版本优先使用 `androidApi` 等项目统一配置。
- 不在业务模块硬编码另一个冲突版本。
- 新增依赖前必须确认现有工具或组件无法满足需求。
- 不因语言切换增加与需求无关的 Kotlin 扩展库。

# 包结构

继续沿用现有项目包结构：

```text
com.xxx.project_name/
├── ui/
│   ├── adapter/          # Adapter
│   ├── mime/             # 主要业务模块
│   │   ├── main/         # 主界面
│   │   └── .../          # 其他业务模块
│   └── ...
├── widget/
│   └── dialog/           # 自定义 Dialog
├── dao/                  # Room Dao
├── entitys/              # Room 和普通 Entity
├── utils/                # 工具类
└── common/               # 基类、契约和公共类
```

要求：

- Java 与 Kotlin 同包共存时保持包名一致。
- 不主动修正历史包名 `entitys`，除非用户明确要求整体迁移。
- 新文件放在对应业务包，不建立无业务意义的 `helper`、`manager`、`extension` 包。
