# Kotlin 原生 DataBinding Dialog 完整示例

自定义弹窗使用 Android 原生 `Dialog` 和 DataBinding。Dialog 只负责初始化视图和返回选择结果。

## 完整示例

```kotlin
package com.xxx.project_name.widget.dialog

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
import com.xxx.project_name.R
import com.xxx.project_name.databinding.DialogXxxSelectBinding

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
        // 初始化弹窗展示数据。
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

## 显示弹窗

```kotlin
XxxSelectDialog(this) { _, which ->
    selectItem(which)
}.show()
```

## 实施检查

- [ ] 是否通过 DataBinding 创建布局并设置根视图。
- [ ] 是否避免 `!!` 和不必要的作用域函数链。
- [ ] 是否只在 Dialog 内处理 UI 和结果回调。
- [ ] 是否在选项点击后关闭弹窗。

