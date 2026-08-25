# Kotlin Adapter 完整示例

Adapter 只负责列表展示和局部按钮回调，Item 点击、长按和复杂业务由页面层处理。

## 完整示例

```kotlin
package com.xxx.project_name.ui.adapter

import android.content.Context
import android.view.View
import com.bumptech.glide.Glide
import com.xxx.project_name.R
import com.xxx.project_name.entitys.XxxEntity
import com.viterbi.common.base.BaseRecylerAdapter
import com.viterbi.common.base.MyRecylerViewHolder

class XxxAdapter(
    private val context: Context,
    list: MutableList<XxxEntity>?,
    layoutId: Int
) : BaseRecylerAdapter<XxxEntity>(context, list, layoutId) {

    private var buttonClickListener: BaseRecylerAdapter.ButtonClickListener<XxxEntity>? = null

    fun setButtonClickListener(
        listener: BaseRecylerAdapter.ButtonClickListener<XxxEntity>?
    ) {
        buttonClickListener = listener
    }

    override fun convert(holder: MyRecylerViewHolder, position: Int) {
        val entity = mDatas[position]
        holder.setText(R.id.tv_name, entity.name.orEmpty())

        Glide.with(context)
            .load(entity.imageUrl)
            .into(holder.getImageView(R.id.iv_cover))

        holder.getView<View>(R.id.btn_action).setOnClickListener { view ->
            buttonClickListener?.onButtonClick(view, position, entity)
        }
    }
}
```

## 页面 `bindEvent()` 中设置监听

```kotlin
adapter.setOnItemClickLitener { _, position, data ->
    val entity = data as XxxEntity
    openDetail(entity)
}

adapter.setOnLongItemClickLitener { _, position ->
    showDeleteDialog(adapter.getItem(position))
}
```

Adapter 初始化和 RecyclerView 绑定放在 `initView()`，Item 点击和长按监听放在页面的 `bindEvent()`。

## 实施检查

- [ ] 是否按基类真实签名使用 `MutableList` 和 `mDatas`。
- [ ] 是否没有在 `convert()` 内执行数据库、网络或页面跳转。
- [ ] 是否使用 `addAllAndClear()` 刷新外部数据。
