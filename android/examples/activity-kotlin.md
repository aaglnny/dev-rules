# Kotlin Activity 完整示例

本模板用于普通 DataBinding Activity。复制后替换业务类型和资源名称，保持生命周期方法顺序、Binding 使用方式和项目基类不变。

## 固定结构

- 默认使用 `val`，只有确实需要重新赋值时使用 `var`。
- 页面对象使用 `lateinit` 只承载生命周期内必然初始化的对象。
- 点击事件集中在 `onClickCallback()`，分支使用带大括号的 `when`。
- 不使用 `!!`，不使用 `findViewById()`。

## 可替换内容

- Activity、Binding、Adapter、Entity 和布局资源名称。
- Presenter 类型及 `getData()`、`save()` 的业务实现。
- 标题和控件 ID。

## 完整示例

```kotlin
package com.xxx.project_name.ui.xxx

import android.os.Bundle
import android.view.View
import androidx.recyclerview.widget.LinearLayoutManager
import com.blankj.utilcode.util.SizeUtils
import com.xxx.project_name.R
import com.xxx.project_name.databinding.ActivityXxxBinding
import com.xxx.project_name.entitys.XxxEntity
import com.xxx.project_name.ui.adapter.XxxAdapter
import com.viterbi.common.base.BaseActivity
import com.viterbi.common.base.BasePresenter
import com.viterbi.common.widget.view.SimplePaddingDecoration

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
        binding.rv.addItemDecoration(SimplePaddingDecoration(mContext, SizeUtils.dp2px(0f)))
        binding.rv.adapter = adapter

        getData()
    }

    override fun bindEvent() {
        binding.setOnClickListener(this::onClickCallback)

        adapter.setOnItemClickLitener { _, _, entity ->
            openDetail(entity as XxxEntity)
        }
    }

    override fun onClickCallback(view: View) {
        when (view.id) {
            R.id.iv_title_back -> {
                finish()
            }
            R.id.tv_title_right -> {
                save()
            }
        }
    }

    private fun getData() {
        // 按当前页面的数据来源调用数据库或 Presenter。
    }

    private fun openDetail(entity: XxxEntity) {
        // 打开详情页或处理当前 Item。
    }

    private fun save() {
        // 先完成同步校验，再按项目异步规则保存。
    }
}
```

## 实施检查

- [ ] 是否使用项目现有 `BaseActivity` 和 `binding`。
- [ ] 是否在 `initView()` 初始化列表并调用 `getData()`。
- [ ] Item 点击和长按监听是否在 `bindEvent()` 中注册。
- [ ] 是否避免无意义的扩展函数和作用域函数链。
- [ ] 是否没有使用 `!!` 或绕过 Presenter 直接请求网络。
