# Java Adapter 完整示例

Adapter 只负责 Item 展示和局部按钮回调。列表点击、长按和数据库操作由 Activity、Fragment 或 Dialog 设置和处理。

## 固定结构

- 继承 `BaseRecylerAdapter<XxxEntity>`。
- 使用基类的 `mDatas`、`MyRecylerViewHolder` 和绑定方法。
- `convert()` 不执行数据库、网络、页面跳转或复杂业务。
- 外部刷新使用 `adapter.addAllAndClear(list)`。

## 完整示例

```java
package com.xxx.project_name.ui.adapter;

import android.content.Context;
import android.view.View;

import com.bumptech.glide.Glide;
import com.xxx.project_name.R;
import com.xxx.project_name.common.base.BaseRecylerAdapter;
import com.xxx.project_name.entitys.XxxEntity;

import java.util.List;

public class XxxAdapter extends BaseRecylerAdapter<XxxEntity> {
    private final Context context;
    private ButtonClickListener<XxxEntity> buttonClickListener;

    public XxxAdapter(Context context, List<XxxEntity> list, int layoutId) {
        super(context, list, layoutId);
        this.context = context;
    }

    public void setButtonClickListener(
            ButtonClickListener<XxxEntity> buttonClickListener) {
        this.buttonClickListener = buttonClickListener;
    }

    @Override
    public void convert(MyRecylerViewHolder holder, int position) {
        XxxEntity entity = mDatas.get(position);
        String name = entity.getName() == null ? "" : entity.getName();

        holder.setText(R.id.tv_name, name);
        Glide.with(context)
                .load(entity.getImageUrl())
                .into(holder.getImageView(R.id.iv_cover));

        holder.getView(R.id.btn_action).setOnClickListener(view -> {
            if (buttonClickListener != null) {
                buttonClickListener.onButtonClick(view, position, entity);
            }
        });
    }
}
```

## 外部设置监听

```java
adapter.setOnItemClickLitener((view, position, data) -> {
    XxxEntity entity = (XxxEntity) data;
    openDetail(entity);
});

adapter.setOnLongItemClickLitener((view, position) -> {
    showDeleteDialog(adapter.getItem(position));
});
```

## 实施检查

- [ ] 是否设置了 `LayoutManager` 后再绑定 Adapter。
- [ ] 是否没有直接修改 `mDatas`。
- [ ] 是否没有在 `convert()` 内执行异步业务。

