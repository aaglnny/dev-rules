# Android Kotlin 语言开发规范

## 适用范围

本规范只负责 Kotlin 语言写法和 Kotlin 实现示例。Android 项目的公共架构、组件职责、文件语言边界、Room、依赖、Manifest 和包结构统一读取 [android-project.md](android-project.md)；XML、Drawable 和资源规则统一读取 [android-xml.md](android-xml.md)。

使用本规范前必须先加载 `E:\rules\android\android-project.md`。涉及布局或资源时再加载 `E:\rules\android\android-xml.md`。

`dao`、`entitys` 包必须使用 Java，其实现示例读取 [android-java.md](android-java.md)，不得在本文件中复制一套 Kotlin Entity 或 Dao 示例。

# Kotlin 基础规则

## 版本与语法边界

- Kotlin 版本以项目根目录配置为准，当前项目使用 Kotlin `1.6.21` 和 JVM `1.8`。
- 不在普通业务需求中擅自升级 Kotlin 或 JVM 版本。
- 只使用当前版本和项目依赖能够稳定支持的语法。
- 不因为使用 Kotlin 就主动引入新的架构、异步框架或 Kotlin 扩展库。

## `val` 与 `var`

- 默认使用 `val`，只有变量确实需要重新赋值时才使用 `var`。
- 集合内容可变但引用不变时使用 `val` 声明可变集合。
- 不为了“统一”把所有成员都声明为 `var`，也不为了绝对不可变创建大量临时对象。

```kotlin
private val list = mutableListOf<XxxEntity>()
private var currentIndex = 0
```

## 属性

- Kotlin 属性自带 getter/setter，不手写 Java 风格访问器。
- 页面状态和 Kotlin 业务对象直接使用属性表达。
- 私有成员使用 `private`。
- 不使用无意义的 `m` 前缀；已有 Java 基类字段如 `mContext`、`mDatas` 必须保持原名。

```kotlin
private var selectedType = TYPE_DEFAULT
```

## 空安全

- 根据数据真实来源决定是否可空，不机械地把所有属性声明为可空。
- 用户输入、Intent 参数、网络响应、数据库可空列和 Java 平台类型在语言边界处处理空值。
- 内部流程已经保证非空的数据不重复层层判空。
- 优先使用 `orEmpty()`、安全调用、Elvis 运算符、`takeIf` 和提前返回。
- 禁止随意使用 `!!`；只有生命周期和调用链能严格证明非空且无法用更清晰的类型表达时才允许使用。
- 简单 `if` 更清楚时，不连续嵌套多层 `let`、`run`、`also`。

```kotlin
val name = intent.getStringExtra("name").orEmpty()

val entity = dao.queryById(id) ?: return

imageUrl?.takeIf { it.isNotBlank() }?.let {
    Glide.with(this).load(it).into(binding.ivCover)
}
```

## `lateinit` 与 `lazy`

- Activity、Fragment、自定义 Dialog 中由生命周期初始化的非空对象可使用 `lateinit var`。
- 使用前必须能由生命周期顺序保证已经初始化，不用 `lateinit` 掩盖本应可空的业务状态。
- 只读且首次使用时才创建的对象可使用 `by lazy`。
- 简单对象不为了展示 Kotlin 特性强行使用延迟初始化。

```kotlin
private lateinit var adapter: XxxAdapter

private val dao by lazy {
    DatabaseManager.getInstance(applicationContext).getXxxDao()
}
```

## 集合

- 只读参数和只读返回值优先使用 `List<T>`。
- 需要增删改时使用 `MutableList<T>`。
- 调用 Java 基类时以真实签名为准；基类要求 `MutableList<T>?` 时按其签名传入。
- 不在同一段逻辑中无意义地反复调用 `toList()`、`toMutableList()`。

## 函数

- 简单返回值可以使用表达式函数。
- 包含较长业务逻辑、分支或副作用时使用普通函数体。
- 不把两三行且只使用一次的逻辑机械拆成辅助函数。
- 单一业务文件中可使用 `load()`、`save()`、`delete()` 等短方法名；对外或跨模块方法使用完整业务名称。

```kotlin
override fun onSetLayoutId(): Int = R.layout.fra_xxx

private fun hasData(): Boolean = list.isNotEmpty()
```

## 静态成员与 Java 互操作

- 使用 `companion object` 表达类级成员。
- 需要让 Java 以静态方式调用时添加 `@JvmStatic`。
- 需要向 Java 暴露字段时按实际需要使用 `const val` 或 `@JvmField`。
- 不无目的地给所有 companion 方法添加 `@JvmStatic`。
- Java SAM 接口优先使用 Lambda；多方法接口使用 `object : Interface {}`。

```kotlin
companion object {
    @JvmStatic
    fun newInstance(): XxxFragment = XxxFragment()
}
```

## Java 平台类型

- 在调用 Java API 的边界处尽早明确可空性。
- Java 返回的字符串用于展示时按需使用 `orEmpty()`。
- Java 返回的 Entity 可能为空时先按可空值接收并判断。
- 不对未知 Java 返回值直接使用 `!!`。
- Java 方法名、基类字段名和历史拼写保持原样，例如 `BaseRecylerAdapter`、`setOnItemClickLitener()`。

## 语法克制

- 不为简单页面创建无必要的扩展函数、密封类、委托、DSL 或操作符重载。
- 不使用复杂作用域函数链制造“简洁感”。
- 不把普通回调全部包装为高阶函数；已有 Java 接口或项目公共接口能满足需求时直接复用。

# Kotlin 代码风格

## 常量

```kotlin
companion object {
    private const val TYPE_DEFAULT = "0"
    private const val REQUEST_CODE_SELECT = 1001
}
```

- 类内常量使用 `private const val`。
- 多处共享的常量复用项目已有常量类。
- 不为只使用一次的简单字符串创建常量。
- Intent Key 被多个入口使用时声明为常量。

## 格式

- 以 Android Studio/Kotlin 默认格式为基础，不顺手格式化整个旧文件。
- `when` 的每个分支都必须使用 `{}` 包裹，即使分支内只有一行代码。
- 链式调用较长时按调用层级换行。
- Lambda 简短时放在调用处，过长时提取为有业务含义的方法。
- 不使用分号。

```kotlin
when (view.id) {
    R.id.iv_title_back -> {
        finish()
    }
    R.id.tv_title_right -> {
        saveData()
    }
}
```

# Android 组件 Kotlin 示例

以下示例只说明如何用 Kotlin 落实 [android-project.md](android-project.md) 的公共规则，不重新定义组件职责。

## Activity

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
        // 按项目数据库或网络规则加载数据
    }

    private fun saveData() {
        // 先验证，再按项目异步规则保存
    }
}
```

网格列表只替换布局管理器和间距实现：

```kotlin
binding.rv.layoutManager = GridLayoutManager(this, 2)
binding.rv.addItemDecoration(
    GridSpacesItemDecoration(2, SizeUtils.dp2px(8f), false)
)
```

## Fragment

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

带参数实例：

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

## Adapter

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

设置基类 Item 监听：

```kotlin
adapter.setOnItemClickLitener { _, position, data ->
    val entity = data as XxxEntity
    handleItemClick(entity, position)
}

adapter.setOnLongItemClickLitener { _, position ->
    delete(adapter.getItem(position))
}
```

如果 Kotlin 已正确推断 `data` 为具体 Entity，不做多余强制转换。

Item 内按钮回调：

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

需要语义明确的专用回调时使用 `fun interface`：

```kotlin
fun interface OnDeleteClickListener {
    fun onDeleteClick(entity: XxxEntity, position: Int)
}
```

# Room 与 RxJava3 Kotlin 示例

Entity、Dao 和 `DatabaseManager` 的 Java 定义见 [android-java.md](android-java.md)。Kotlin 调用层按以下方式实现异步操作。

## 查询

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

## 保存

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

## 删除

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

# 项目组件 Kotlin 调用示例

## Toast、确认弹窗和工具类

```kotlin
ToastUtils.showShort("操作成功")

SizeUtils.dp2px(12f)

val text = VTBTimeUtils.formatDateTime(System.currentTimeMillis())
```

```kotlin
DialogUtil.showConfirmRreceiptDialog(
    this,
    "标题",
    "确认要执行此操作吗？",
    object : ConfirmDialog.OnDialogClickListener {
        override fun confirm() {
            deleteData(entity)
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
                deleteData(entity)
            }

            override fun cancel() {
            }
        }
    )
    .create(true)
    .show()
```

## 图片加载

圆角控件和资源配置读取 [android-xml.md](android-xml.md)。Kotlin 只负责正常加载图片，不使用 `RoundedCorners` 模拟项目圆角方案：

```kotlin
Glide.with(context)
    .load(imageUrl)
    .into(binding.ivCover)
```

## RadioButton 与 RadioGroup 监听

```kotlin
binding.rbOption1.setOnCheckedChangeListener { _, isChecked ->
    if (isChecked) {
        currentOption = OPTION_1
        updateUi()
    }
}
```

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

# Presenter 网络请求 Kotlin 示例

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
        adapter.addAllAndClear(list)
    }

    override fun hideLoading() {
        hideLoadingDialog()
    }
}
```

# 页面跳转 Kotlin 示例

无参数跳转：

```kotlin
skipAct(XxxActivity::class.java)
```

携带参数：

```kotlin
val intent = Intent(this, XxxActivity::class.java)
intent.putExtra("data", entity)
startActivity(intent)
```

多入口页面可提供：

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

# 自定义 Dialog Kotlin 示例

Dialog 的公共职责读取 [android-project.md](android-project.md)，布局读取 [android-xml.md](android-xml.md)。

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
        // 初始化当前 Dialog 需要的数据
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

Activity 中显示：

```kotlin
XxxSelectDialog(this) { _, which ->
    selectItem(which)
}.show()
```

Fragment 中显示：

```kotlin
XxxSelectDialog(requireContext()) { _, which ->
    selectItem(which)
}.show()
```

# 表单处理 Kotlin 示例

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

- 字符串必填验证优先使用 `isBlank()`；只判断空字符串时可使用 `isEmpty()`。
- 数字解析使用 `toIntOrNull()`、`toLongOrNull()`、`toFloatOrNull()`、`toDoubleOrNull()`，不用异常代替 Kotlin 已提供的安全解析。
- Java 兼容边界可继续使用 `TextUtils.isEmpty()`，但同一方法不混用多套空值判断。
