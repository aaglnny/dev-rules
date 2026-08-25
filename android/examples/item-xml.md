# RecyclerView Item XML 完整示例

适用于 `BaseRecylerAdapter` 的普通列表 Item。Item 布局只表达展示结构，点击和长按由页面在 `bindEvent()` 中设置。

## 完整示例

文件路径：`res/layout/list/layout/item_xxx.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:layout_marginStart="16dp"
    android:layout_marginTop="8dp"
    android:layout_marginEnd="16dp"
    android:layout_marginBottom="8dp"
    android:background="#F5F6F7"
    android:padding="12dp">

    <com.makeramen.roundedimageview.RoundedImageView
        android:id="@+id/iv_cover"
        android:layout_width="100dp"
        android:layout_height="66dp"
        android:scaleType="centerCrop"
        app:riv_corner_radius="8dp"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

    <TextView
        android:id="@+id/tv_name"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_marginStart="12dp"
        android:ellipsize="end"
        android:maxLines="1"
        android:textColor="@color/base_text_color"
        android:textSize="16sp"
        android:textStyle="bold"
        app:layout_constraintEnd_toStartOf="@id/iv_delete"
        app:layout_constraintStart_toEndOf="@id/iv_cover"
        app:layout_constraintTop_toTopOf="@id/iv_cover" />

    <TextView
        android:id="@+id/tv_content"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_marginTop="4dp"
        android:maxLines="2"
        android:textColor="#666666"
        android:textSize="13sp"
        app:layout_constraintEnd_toStartOf="@id/iv_delete"
        app:layout_constraintStart_toStartOf="@id/tv_name"
        app:layout_constraintTop_toBottomOf="@id/tv_name" />

    <ImageView
        android:id="@+id/iv_delete"
        android:layout_width="28dp"
        android:layout_height="28dp"
        android:src="@android:drawable/ic_menu_delete"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        app:tint="#FFFF5C5C" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

Item 内按钮由 Adapter 的局部按钮回调绑定，不在普通 Item XML 中声明 DataBinding 点击变量。系统删除图标只用于保证模板完整，目标项目有对应资源时优先替换。

## 实施检查

- [ ] 文件是否位于 `res/layout/list/layout/`。
- [ ] 图片是否使用项目已有圆角控件和默认资源。
- [ ] Item 是否没有执行数据库、网络和页面跳转。
- [ ] 是否根据实际点击方案选择 DataBinding 点击或 Adapter 按钮回调，不维护两套重复监听。
