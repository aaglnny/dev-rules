# Java Activity 完整示例

本模板用于普通 DataBinding Activity。复制后只替换业务类型、布局资源、标题和具体业务逻辑；保留基类、生命周期顺序和 Binding 访问方式。

## 固定结构

- 继承项目现有 `BaseActivity<Binding, Presenter>`。
- 在 `onCreate()` 中调用 `setDataBindingLayout()`。
- 方法顺序保持为 `onCreate()`、`initView()`、`bindEvent()`、点击回调、私有业务方法。
- 使用 `binding` 访问控件，不使用 `findViewById()`。
- 数据库和网络请求沿用项目已有 Presenter、RxJava3 和基类能力。

## 可替换内容

- `XxxActivity`、`ActivityXxxBinding`、`XxxEntity`、`XxxAdapter`。
- 布局资源、标题、控件 ID 和具体业务方法。
- Presenter 类型；无网络页面可使用 `BasePresenter`。

## 完整示例

```java
package com.xxx.project_name.ui.xxx;

import android.os.Bundle;
import android.view.View;

import androidx.recyclerview.widget.LinearLayoutManager;

import com.xxx.project_name.R;
import com.xxx.project_name.databinding.ActivityXxxBinding;
import com.xxx.project_name.ui.adapter.XxxAdapter;
import com.xxx.project_name.common.base.BaseActivity;
import com.xxx.project_name.common.base.BasePresenter;

public class XxxActivity extends BaseActivity<ActivityXxxBinding, BasePresenter> {
    private XxxAdapter adapter;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setDataBindingLayout(R.layout.activity_xxx);
    }

    @Override
    public void initView() {
        binding.include.setTitleStr("标题");

        adapter = new XxxAdapter(this, null, R.layout.item_xxx);
        binding.rv.setLayoutManager(new LinearLayoutManager(this));
        binding.rv.setAdapter(adapter);

        getData();
    }

    @Override
    public void bindEvent() {
        binding.setOnClickListener(this::onClickCallback);
    }

    @Override
    public void onClickCallback(View view) {
        int id = view.getId();
        if (id == R.id.iv_title_back) {
            finish();
        } else if (id == R.id.tv_title_right) {
            save();
        }
    }

    private void getData() {
        // 按当前页面的数据来源调用数据库或 Presenter。
    }

    private void save() {
        // 先完成同步校验，再按项目异步规则保存。
    }
}
```

## 实施检查

- [ ] 是否保留 `setDataBindingLayout()` 和 Binding 访问。
- [ ] 是否复用标题栏和项目已有 Presenter 基类。
- [ ] 是否在 `initView()` 完成列表初始化并调用数据加载。
- [ ] 是否没有把网络或数据库调用放进 Adapter。
