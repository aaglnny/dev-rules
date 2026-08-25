# Android Java 语言开发规范

## 适用范围

本规范负责 Java 语言写法、互操作边界和小型语法示例。Android 公共架构、组件职责、文件语言边界、Room、依赖、Manifest 和包结构读取 [android-project.md](android-project.md)；完整组件结构读取 `examples` 目录；XML、Drawable 和资源规则读取 [android-xml.md](android-xml.md)。

使用本规范前必须先加载 `E:\rules\android\android-project.md`。涉及布局或资源时再加载 `E:\rules\android\android-xml.md`。

`dao`、`entitys` 包必须使用 Java。Entity、Dao 和 `DatabaseManager` 的完整结构只维护在 `E:\rules\android\examples\room-java.md`。

## Java 完整示例索引

- Activity：`E:\rules\android\examples\activity-java.md`
- Fragment：`E:\rules\android\examples\fragment-java.md`
- Adapter：`E:\rules\android\examples\adapter-java.md`
- Entity、Dao、DatabaseManager：`E:\rules\android\examples\room-java.md`
- RxJava3 数据库异步：`E:\rules\android\examples\rxjava-java.md`
- 自定义 Dialog：`E:\rules\android\examples\dialog-java.md`

完整示例的强制程度、可替换范围和真实项目核对顺序遵守 [android-project.md](android-project.md) 中的“完整示例使用规则”。

# Java 基础规则

## 版本与语法边界

- Java 版本以项目 Gradle 配置为准，不在普通需求中擅自升级。
- 优先使用项目已经稳定采用的 Java 写法，不为展示新语法改写旧代码。
- Java 与 Kotlin 互调时保持现有公开方法名、字段名和历史拼写不变。

## 字段、变量与空值

- 成员字段声明在类顶部，默认使用 `private`。
- 不会重新赋值的字段按实际情况使用 `final`，不机械补全。
- Adapter 默认命名为 `adapter`，单一列表默认命名为 `list`，Binding 固定使用 `binding`。
- 根据真实数据来源处理 `null`，不为项目内部已保证非空的数据重复判空。

```java
String displayName = entity.getName() == null ? "" : entity.getName();

if (entity == null) {
    return;
}
```

## Lambda、分支与方法组织

- 单方法接口优先使用 Lambda 或方法引用，多方法接口使用匿名内部类。
- Lambda 逻辑较长时提取为有业务含义的方法。
- 点击事件按分支数量选择 `if/else` 或 `switch`。
- 方法默认按生命周期方法、基类重写方法、接口回调、私有业务方法组织。
- 不顺手格式化整个旧文件，不整理与当前需求无关的 import。

```java
@Override
public void onClickCallback(View view) {
    int id = view.getId();
    if (id == R.id.iv_title_back) {
        finish();
    } else if (id == R.id.tv_title_right) {
        save();
    }
}
```

# Java 调用规则

## Item 监听

- Adapter 初始化、`LayoutManager`、`ItemDecoration` 和 `setAdapter()` 放在 `initView()`。
- `setOnItemClickLitener()`、`setOnLongItemClickLitener()` 及 Item 内按钮监听的页面侧设置放在 `bindEvent()`。
- Adapter 内部只负责展示和触发回调，不执行页面跳转、数据库或网络业务。

## Room 与 RxJava3

- Entity、Dao 和 `DatabaseManager` 的完整写法读取 `room-java.md`。
- 查询、保存和删除的完整写法读取 `rxjava-java.md`。
- Dao 返回普通类型，不返回 RxJava 类型。
- 使用 `Observable.create()` 延迟执行 Dao，不使用 `Observable.just(dao.queryAll())`。
- 数据库操作在 `Schedulers.io()` 执行，UI 更新切换到主线程。

## Toast、确认弹窗和图片

```java
ToastUtils.showShort("操作成功");
int padding = SizeUtils.dp2px(12);
String text = VTBTimeUtils.formatDateTime(System.currentTimeMillis());

Glide.with(context)
        .load(imageUrl)
        .into(binding.ivCover);
```

删除等不可逆操作复用 `DialogUtil.showConfirmRreceiptDialog()`；自定义 Dialog 的完整结构读取 `dialog-java.md`。

## Presenter 网络请求

- 网络页面使用对应 Contract 的 Presenter，并实现 View 接口。
- Presenter 在 `initView()` 中按当前模块方式初始化并发起请求。
- Gson 泛型解析使用匿名 `TypeToken`。
- 编码前必须读取当前模块中一个真实的 Presenter 页面。

```java
List<XxxEntity> list = new Gson().fromJson(
        jsonStr,
        new TypeToken<List<XxxEntity>>() {
        }.getType());
adapter.addAllAndClear(list);
```

## 页面跳转与表单

无参数跳转优先使用 `skipAct(XxxActivity.class)`。Entity 通过 Intent 传递时必须实现 `Serializable`。

```java
String value = binding.etField.getText().toString().trim();
if (TextUtils.isEmpty(value)) {
    ToastUtils.showShort("请输入必填字段");
    return;
}

float number;
try {
    number = Float.parseFloat(value);
} catch (NumberFormatException e) {
    ToastUtils.showShort("请输入有效的数值");
    return;
}
```
