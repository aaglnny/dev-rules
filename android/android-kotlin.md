# Android Kotlin 语言开发规范

## 适用范围

本规范负责 Kotlin 语言写法、Java 互操作边界和小型语法示例。Android 公共架构、组件职责、文件语言边界、Room、依赖、Manifest 和包结构读取 [android-project.md](android-project.md)；完整组件结构读取 `examples` 目录；XML、Drawable 和资源规则读取 [android-xml.md](android-xml.md)。

使用本规范前必须先加载 `E:\rules\android\android-project.md`。涉及布局或资源时再加载 `E:\rules\android\android-xml.md`。

`dao`、`entitys` 包必须使用 Java，其完整结构读取 `E:\rules\android\examples\room-java.md`，不得创建 Kotlin Entity 或 Dao 示例。

## Kotlin 完整示例索引

- Activity：`E:\rules\android\examples\activity-kotlin.md`
- Fragment：`E:\rules\android\examples\fragment-kotlin.md`
- Adapter：`E:\rules\android\examples\adapter-kotlin.md`
- Room 的 Java Entity、Dao 和 DatabaseManager：`E:\rules\android\examples\room-java.md`
- RxJava3 数据库异步：`E:\rules\android\examples\rxjava-kotlin.md`
- 自定义 Dialog：`E:\rules\android\examples\dialog-kotlin.md`

完整示例的强制程度、可替换范围和真实项目核对顺序遵守 [android-project.md](android-project.md) 中的“完整示例使用规则”。

# Kotlin 基础规则

## 版本与语法边界

- Kotlin 版本以项目根目录配置为准，当前项目使用 Kotlin `1.6.21` 和 JVM `1.8`。
- 不在普通需求中擅自升级 Kotlin 或 JVM 版本。
- 只使用当前版本和项目依赖能够稳定支持的语法。
- 不因为使用 Kotlin 就主动引入新的架构、异步框架或 Kotlin 扩展库。

## `val`、属性与集合

- 默认使用 `val`，只有确实需要重新赋值时使用 `var`。
- 集合内容可变但引用不变时使用 `val` 声明可变集合。
- Kotlin 属性自带 getter/setter，不手写 Java 风格访问器。
- 私有成员使用 `private`，不使用无意义的 `m` 前缀；已有 Java 基类字段如 `mContext`、`mDatas` 保持原名。
- 只读参数和返回值优先使用 `List<T>`，调用 Java 基类时以真实签名为准。

```kotlin
private val list = mutableListOf<XxxEntity>()
private var currentIndex = 0
private var selectedType = TYPE_DEFAULT
```

## 空安全

- 根据数据真实来源决定是否可空，不机械地把所有属性声明为可空。
- 用户输入、Intent 参数、网络响应、数据库可空列和 Java 平台类型在边界处处理空值。
- 优先使用 `orEmpty()`、安全调用、Elvis 运算符、`takeIf` 和提前返回。
- 禁止随意使用 `!!`。
- 简单 `if` 更清楚时，不连续嵌套多层作用域函数。

```kotlin
val name = intent.getStringExtra("name").orEmpty()
val entity = dao.queryById(id) ?: return
imageUrl?.takeIf { it.isNotBlank() }?.let {
    Glide.with(this).load(it).into(binding.ivCover)
}
```

## `lateinit`、`lazy` 与函数

- Activity、Fragment、自定义 Dialog 中由生命周期初始化的非空对象可以使用 `lateinit var`。
- 只读且首次使用时才创建的对象可以使用 `by lazy`。
- 简单返回值可以使用表达式函数；较长业务逻辑使用普通函数体。
- 不把两三行且只使用一次的逻辑机械拆成辅助函数。

```kotlin
private lateinit var adapter: XxxAdapter

private val dao by lazy {
    DatabaseManager.getInstance(applicationContext).getXxxDao()
}

override fun onSetLayoutId(): Int = R.layout.fra_xxx
```

## Java 互操作与类级成员

- 使用 `companion object` 表达类级成员。
- 需要让 Java 静态调用时添加 `@JvmStatic`，不无目的地添加。
- Java SAM 接口优先使用 Lambda；多方法接口使用 `object : Interface {}`。
- Java 返回的字符串展示时按需使用 `orEmpty()`；Java 返回的 Entity 可能为空时先判断。
- Java 方法名、基类字段名和历史拼写保持原样，例如 `BaseRecylerAdapter`、`setOnItemClickLitener()`。

```kotlin
companion object {
    @JvmStatic
    fun newInstance(): XxxFragment = XxxFragment()
}
```

## 语法克制与格式

- 不为简单页面创建无必要的扩展函数、密封类、委托、DSL 或操作符重载。
- 不使用复杂作用域函数链制造“简洁感”。
- 以 Android Studio/Kotlin 默认格式为基础，不顺手格式化整个旧文件。
- `when` 的每个分支使用 `{}` 包裹，不使用分号。
- 链式调用较长时按调用层级换行。

```kotlin
when (view.id) {
    R.id.iv_title_back -> {
        finish()
    }
    R.id.tv_title_right -> {
        save()
    }
}
```

# Kotlin 调用规则

## Item 监听

- Adapter 初始化、`LayoutManager`、`ItemDecoration` 和 Adapter 绑定放在 `initView()`。
- `setOnItemClickLitener()`、`setOnLongItemClickLitener()` 及 Item 内按钮监听的页面侧设置放在 `bindEvent()`。
- Adapter 内部只负责展示和触发回调，不执行页面跳转、数据库或网络业务。

## Room 与 RxJava3

- Entity、Dao 和 `DatabaseManager` 的完整写法读取 `room-java.md`。
- 查询、保存和删除的完整写法读取 `rxjava-kotlin.md`。
- Dao 返回普通类型，不返回 RxJava 类型。
- 使用 `Observable.create()` 延迟执行 Dao，不使用 `Observable.just(dao.queryAll())`。
- 数据库操作在 `Schedulers.io()` 执行，UI 更新切换到主线程。

## Toast、确认弹窗和图片

```kotlin
ToastUtils.showShort("操作成功")
SizeUtils.dp2px(12f)
val text = VTBTimeUtils.formatDateTime(System.currentTimeMillis())

Glide.with(context)
    .load(imageUrl)
    .into(binding.ivCover)
```

删除等不可逆操作复用 `DialogUtil.showConfirmRreceiptDialog()`；自定义 Dialog 的完整结构读取 `dialog-kotlin.md`。

## Presenter 网络请求

- 网络页面使用对应 Contract 的 Presenter，并实现 View 接口。
- Presenter 在 `initView()` 中按当前模块方式初始化并发起请求。
- Gson 泛型解析使用匿名 `TypeToken`。
- 编码前必须读取当前模块中一个真实的 Presenter 页面。

```kotlin
val type = object : TypeToken<List<XxxEntity>>() {}.type
val list: List<XxxEntity> = Gson().fromJson(jsonStr, type)
adapter.addAllAndClear(list)
```

## 页面跳转与表单

无参数跳转优先使用 `skipAct(XxxActivity::class.java)`。Entity 通过 Intent 传递时必须实现 `Serializable`。

```kotlin
val value = binding.etField.text.toString().trim()
if (value.isBlank()) {
    ToastUtils.showShort("请输入必填字段")
    return
}

val number = value.toFloatOrNull()
if (number == null) {
    ToastUtils.showShort("请输入有效的数值")
    return
}
```
