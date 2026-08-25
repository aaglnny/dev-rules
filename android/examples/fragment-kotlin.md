# Kotlin Fragment 完整示例

本模板用于无参数 Fragment。带参数时通过 `arguments` 传递，Binding 生命周期沿用项目基类封装。

## 完整示例

```kotlin
package com.xxx.project_name.ui.xxx

import android.view.View
import com.xxx.project_name.R
import com.xxx.project_name.common.base.BaseFragment
import com.xxx.project_name.common.base.BasePresenter
import com.xxx.project_name.databinding.FraXxxBinding

class XxxFragment : BaseFragment<FraXxxBinding, BasePresenter>() {

    companion object {
        @JvmStatic
        fun newInstance(): XxxFragment = XxxFragment()
    }

    override fun onSetLayoutId(): Int = R.layout.fra_xxx

    override fun initView() {
        binding.include.setTitleStr("标题")
        getData()
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
                save()
            }
        }
    }

    private fun getData() {
        // 按页面数据来源加载内容。
    }

    private fun save() {
        // 执行当前 Fragment 的保存业务。
    }
}
```

## 带参数入口

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

## 实施检查

- [ ] 是否使用 `arguments` 而不是自定义构造参数。
- [ ] 是否只在 Binding 有效时访问视图。
- [ ] 异步回调是否避免对已脱离 Activity 的 Fragment 操作 UI。

