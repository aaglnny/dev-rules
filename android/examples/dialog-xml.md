# Dialog XML 完整示例

适用于 Android 原生 DataBinding Dialog。Dialog 类的 Window 配置和结果回调读取 `dialog-java.md` 或 `dialog-kotlin.md`。

## 完整示例

文件路径：`res/layout/dialog/layout/dialog_xxx_select.xml`

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
        android:background="@drawable/shape_base_dialog_bg"
        android:padding="20dp">

        <ImageView
            android:id="@+id/iv_close"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:onClickListener="@{onClickListener}"
            android:src="@android:drawable/ic_menu_close_clear_cancel"
            app:layout_constraintEnd_toEndOf="parent"
            app:layout_constraintTop_toTopOf="parent" />

        <TextView
            android:id="@+id/tv_title"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="请选择"
            android:textColor="@color/base_text_color"
            android:textSize="18sp"
            android:textStyle="bold"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintTop_toTopOf="parent" />

        <TextView
            android:id="@+id/tv_option_1"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_marginTop="24dp"
            android:gravity="center"
            android:onClickListener="@{onClickListener}"
            android:text="选项一"
            android:textColor="@color/base_text_color"
            app:layout_constraintEnd_toEndOf="parent"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintTop_toBottomOf="@id/tv_title" />

        <TextView
            android:id="@+id/tv_option_2"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_marginTop="16dp"
            android:gravity="center"
            android:onClickListener="@{onClickListener}"
            android:text="选项二"
            android:textColor="@color/base_text_color"
            app:layout_constraintEnd_toEndOf="parent"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintTop_toBottomOf="@id/tv_option_1" />

    </androidx.constraintlayout.widget.ConstraintLayout>

</layout>
```

## 实施检查

- [ ] Dialog 根布局是否使用项目已有背景资源。
- [ ] 是否只声明当前布局实际使用的 `onClickListener`。
- [ ] 关闭按钮是否只关闭 Dialog，不返回业务选项。
- [ ] 选项是否通过单一职责回调返回结果。
