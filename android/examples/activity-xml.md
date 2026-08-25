# Activity XML 完整示例

适用于项目现有 DataBinding Activity。页面代码通过生成的 Binding 访问控件，点击事件统一绑定 `onClickListener`。

## 固定结构

- 根标签使用 `<layout>`，根布局优先使用 `ConstraintLayout`。
- `<data>` 只声明当前布局使用的变量。
- 标题栏复用 `layout_title_bar`。
- 列表控件设置明确 ID，并由代码设置 `LayoutManager` 和 Adapter。

## 完整示例

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

        <TextView
            android:id="@+id/tv_empty"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="暂无数据"
            android:textColor="#999999"
            app:layout_constraintBottom_toBottomOf="parent"
            app:layout_constraintEnd_toEndOf="parent"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintTop_toBottomOf="@id/include" />

    </androidx.constraintlayout.widget.ConstraintLayout>

</layout>
```

## 实施检查

- [ ] 是否没有使用 `findViewById()` 对应的重复代码绑定。
- [ ] `RecyclerView` 是否使用 `0dp` 高度并约束到页面边界。
- [ ] 空态控件是否根据实际业务决定是否保留。
