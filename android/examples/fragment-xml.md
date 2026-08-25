# Fragment XML 完整示例

适用于项目现有 DataBinding Fragment。Fragment 使用基类在 `onCreateView()` 中创建 Binding。

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

        <androidx.core.widget.NestedScrollView
            android:id="@+id/scroll"
            android:layout_width="match_parent"
            android:layout_height="0dp"
            app:layout_constraintBottom_toBottomOf="parent"
            app:layout_constraintTop_toBottomOf="@id/include">

            <androidx.constraintlayout.widget.ConstraintLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:padding="16dp">

                <TextView
                    android:id="@+id/tv_content"
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:text="内容"
                    android:textColor="@color/base_text_color"
                    app:layout_constraintStart_toStartOf="parent"
                    app:layout_constraintTop_toTopOf="parent" />

            </androidx.constraintlayout.widget.ConstraintLayout>

        </androidx.core.widget.NestedScrollView>

    </androidx.constraintlayout.widget.ConstraintLayout>

</layout>
```

## 实施检查

- [ ] 滚动容器是否只有一个直接子布局。
- [ ] 是否只在 Binding 有效期间访问控件。
- [ ] 是否复用项目标题栏而不是重复创建标题栏。
