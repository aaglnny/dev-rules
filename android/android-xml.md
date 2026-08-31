# Android XML 与资源开发规范

## 适用范围

本规范只负责 Android XML 布局、DataBinding/ViewBinding 的 XML 写法、控件属性、Drawable、图片控件和资源组织方式，不负责 Java、Kotlin 语法、业务组件职责、依赖版本或 Gradle 配置。

- Android 项目公共规则读取 [android-project.md](android-project.md)。
- Kotlin 调用示例读取 [android-kotlin.md](android-kotlin.md)。
- Java 调用示例读取 [android-java.md](android-java.md)。
- 页面、控件、布局和资源实现优先参考项目中已经调整完成的同类页面，不自行套用通用模板。

## XML 完整示例索引

根据当前布局类型读取对应文件，不一次性加载全部示例：

- Activity：`E:\rules\android\examples\activity-xml.md`
- Fragment：`E:\rules\android\examples\fragment-xml.md`
- RecyclerView Item：`E:\rules\android\examples\item-xml.md`
- Dialog：`E:\rules\android\examples\dialog-xml.md`

完整布局示例只维护在 `examples` 目录。本文件保留 XML 规则和局部属性示例，不再复制完整页面布局。

# DataBinding 与 ViewBinding XML

## DataBinding 根结构

- DataBinding 布局根标签使用 `<layout>`。
- 新创建的 XML 布局文件中，`<data>` 只能声明 `onClickListener`，禁止添加 Entity、ViewModel、状态值、文本等其他 `<variable>`。
- 布局不需要点击监听时，`<data>` 中不声明任何 `<variable>`。
- 点击事件统一声明 `onClickListener`，代码侧使用 Binding 生成的 setter；具体调用语法见对应语言规范。

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
        android:layout_height="match_parent">

        <!-- 页面内容 -->

    </androidx.constraintlayout.widget.ConstraintLayout>

</layout>
```

## ViewBinding 布局

- ViewBinding 页面使用普通 XML 根布局，不为了 ViewBinding 额外包裹 `<layout>`。
- 需要由代码访问的控件设置明确 ID。
- 同一布局不同时维护 DataBinding 变量和另一套重复点击绑定。
- Binding 的选择和生命周期属于项目及语言代码职责，不在本文件重复约定。

# 资源文件组织与命名

## 布局文件

- Activity：`activity_xxx.xml`
- Fragment：`fra_xxx.xml`
- RecyclerView Item：`item_xxx.xml`，存放在 `res/layout/list/layout/`
- Dialog：`dialog_xxx.xml`，存放在 `res/layout/dialog/layout/`
- 自定义 View：`view_xxx.xml`，存放在 `res/layout/view/layout/`

## ID 与图片资源

- 资源 ID 使用带控件类型前缀的小写下划线命名，例如 `tv_name`、`et_name`、`iv_icon`、`rv_list`。
- 布局内部的 ID 根据局部上下文简洁命名，不重复添加布局文件名已经表达的业务前缀。
- 图标以 `ic_` 开头，避免重复添加页面名和 `icon` 等冗余词。
- PNG、JPG、WEBP 等图片格式的图标统一存放在 `res/mipmap-xxhdpi/`。
- XML 格式的图标资源统一存放在 `res/drawable/`。
- 大插图可使用 `_art` 后缀区分。
- `res` 下的资源文件名默认只表达资源自身用途，不添加页面、模块等业务名称。
- 例如进度背景命名为 `shape_progress`，不要命名为 `shape_home_progress`。
- 只有通用名称已经存在、资源无法复用且确实需要区分时，才添加最小必要的业务名称。

## 颜色、字符串与尺寸

- 颜色优先复用项目中已经存在的 `@color` 资源；已有合适颜色时直接引用，无需改为硬编码。
- 项目中没有可复用颜色时，直接在 XML 中使用 `#RRGGBB` 或 `#AARRGGBB`，不为其在 `res/values/colors.xml` 中新增条目。
- 字符串和尺寸直接填写实际文本及 `dp`、`sp` 等值，不为其在 `res/values/` 下新增 `strings.xml`、`dimens.xml` 条目。
- 需要表达选中、按下、禁用等多状态时，可以继续使用 selector 资源；selector 优先复用已有颜色，没有可复用颜色时直接填写颜色值。

## Drawable

- 相同样式优先复用已有 Drawable，禁止重复创建近似资源。
- 圆角背景根据圆角尺寸复用对应 Drawable。
- `shape_bg_white_xx` 固定表示白色背景；需要其他颜色时使用合适的已有资源或 `backgroundTint`，不改变该资源原本语义。

# Android XML 布局编写规则

- 页面必须优先参考项目中已经调整完成的同类布局，保持现有层级、命名、间距和属性写法，不得自行套用通用模板。
- `<layout>` 下的根布局优先使用 `ConstraintLayout`。
- 滚动页面使用 `NestedScrollView + ConstraintLayout`。`NestedScrollView` 下只保留一个直接子布局，禁止增加无业务意义的中间层级。
- 布局属性遵循最小化原则。没有明确裁剪、越界绘制或 padding 绘制需求时，禁止主动添加 `clipChildren`、`clipToPadding`；`rowCount`、`useDefaultMargins`、`alignmentMode` 等属性也只在实际需要时设置。
- `ConstraintLayout` 的直接子节点自行设置外边距和约束；内部子节点只负责自身内容间距，禁止通过额外父布局重复传递边距。
- `GridLayout` 子项明确设置 `layout_row`、`layout_column`、`layout_columnWeight` 和必要边距。宽度使用 `0dp` 配合权重，禁止复制多套相同 View 结构。
- `GridLayout` 中由多个 View 组成且重复出现的结构，必须提取到 `res/layout/view/layout/` 下的独立布局，通过 `include` 复用；一个 View 即可完成的布局不需要提取独立布局。
- 被复用布局只保留公共视觉结构；行列、权重、边距、ID 和点击监听由主布局中的 `include` 设置。
- `include` 需要点击时，直接在 `include` 标签设置 `android:onClickListener="@{onClickListener}"`，禁止在被 include 的布局中重复声明点击变量或设置点击监听。
- 各 include 的差异内容统一由页面通过 Binding 设置，禁止为了不同文字或图标复制布局，也禁止依赖 `findViewById()`。
- 单选切换使用 `RadioGroup + RadioButton`。选中背景和文字颜色优先使用 selector，布局只表达视觉状态，不在代码中重复手动设置背景和文字颜色。
- `EditText` 和 `TextView` 必须设置 `android:textColor`；使用 `hint` 属性时必须设置 `android:textColorHint`。
- `ImageView` 不设置 `android:contentDescription`；使用 tint 属性时设置 `app:tint`，不使用 `android:tint`。
- 除非设计明确要求，`ImageView` 和 `TextView` 使用 `wrap_content`。
- 文本默认不指定字体和字体边距。
- 容器不随意固定高度或设置统一外边距，间距按设计由对应直接子控件承担。
- 所有布局实现以 KISS 为原则，只保留当前页面真正需要的层级、属性和代码，不主动增加预防性配置。

# 常用控件属性示例

任意可点击文本：

```xml
<TextView
    android:id="@+id/tv_confirm"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:onClickListener="@{onClickListener}"
    android:text="确定"
    android:textColor="@color/base_text_color" />
```

带 hint 的输入框：

```xml
<EditText
    android:id="@+id/et_name"
    android:layout_width="0dp"
    android:layout_height="wrap_content"
    android:hint="请输入名称"
    android:textColor="@color/base_text_color"
    android:textColorHint="@color/base_text_color_grey" />
```

# 复用布局与 include

被复用布局只保留公共视觉结构：

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content">

    <ImageView
        android:id="@+id/iv_icon"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content" />

    <TextView
        android:id="@+id/tv_name"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:textColor="@color/base_text_color" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

主布局中的 `include` 负责 ID、Grid 行列、权重、间距和点击：

```xml
<include
    android:id="@+id/include_option_1"
    layout="@layout/view_option"
    android:layout_width="0dp"
    android:layout_height="wrap_content"
    android:layout_column="0"
    android:layout_columnWeight="1"
    android:layout_row="0"
    android:layout_marginEnd="8dp"
    android:onClickListener="@{onClickListener}" />
```

# 圆角图片资源规范

当前项目中 Glide 的 `RoundedCorners` 不作为圆角图片方案。需要圆角图片时，XML 使用项目已依赖的 `RoundedImageView`；依赖声明由 [android-project.md](android-project.md) 负责，图片加载调用见对应语言规范。

```xml
<com.makeramen.roundedimageview.RoundedImageView
    android:id="@+id/iv_cover"
    android:layout_width="match_parent"
    android:layout_height="120dp"
    android:scaleType="centerCrop"
    app:riv_corner_radius="8dp"
    app:riv_oval="false" />
```

# RadioButton 与 RadioGroup 资源规范

- 使用 `RadioGroup` 包裹同一组 `RadioButton`。
- 选中背景使用 selector Drawable，文字颜色使用 selector Color。
- RadioButton 的按钮图标、背景和文字颜色只声明项目当前页面实际需要的属性。
- 不复制多套仅文字或图标不同的单选布局。

```xml
<RadioGroup
    android:id="@+id/rg_option"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="horizontal">

    <RadioButton
        android:id="@+id/rb_option_1"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_weight="1"
        android:background="@drawable/selector_option_bg"
        android:button="@null"
        android:gravity="center"
        android:text="选项一"
        android:textColor="@color/selector_option_text" />

    <RadioButton
        android:id="@+id/rb_option_2"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_weight="1"
        android:background="@drawable/selector_option_bg"
        android:button="@null"
        android:gravity="center"
        android:text="选项二"
        android:textColor="@color/selector_option_text" />

</RadioGroup>
```

# Dialog 布局

Dialog 布局文件命名为 `dialog_xxx.xml`，存放在 `res/layout/dialog/layout/`。完整布局读取 `E:\rules\android\examples\dialog-xml.md`，Dialog 类读取对应 Java 或 Kotlin 示例。
