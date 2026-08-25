# Java Fragment 完整示例

本模板用于无参数 Fragment。带参数时使用 `arguments`，不要依赖自定义构造参数。

## 固定结构

- 继承项目现有 `BaseFragment<Binding, Presenter>`。
- 实现 `onSetLayoutId()`、`initView()`、`bindEvent()` 和点击回调。
- 只在 Binding 有效的生命周期内访问视图。
- Fragment 无参数入口提供 `newInstance()`。

## 完整示例

```java
package com.xxx.project_name.ui.xxx;

import android.view.View;

import com.xxx.project_name.R;
import com.xxx.project_name.common.base.BaseFragment;
import com.xxx.project_name.common.base.BasePresenter;
import com.xxx.project_name.databinding.FraXxxBinding;

public class XxxFragment extends BaseFragment<FraXxxBinding, BasePresenter> {

    public static XxxFragment newInstance() {
        return new XxxFragment();
    }

    @Override
    public int onSetLayoutId() {
        return R.layout.fra_xxx;
    }

    @Override
    public void initView() {
        binding.include.setTitleStr("标题");
        getData();
    }

    @Override
    public void bindEvent() {
        binding.setOnClickListener(this::onClickCallback);
    }

    @Override
    public void onClickCallback(View view) {
        int id = view.getId();
        if (id == R.id.iv_title_back && getActivity() != null) {
            getActivity().finish();
        } else if (id == R.id.tv_title_right) {
            save();
        }
    }

    private void getData() {
        // 按页面数据来源加载内容。
    }

    private void save() {
        // 执行当前 Fragment 的保存业务。
    }
}
```

## 带参数入口

```java
private static final String ARG_ID = "arg_id";

public static XxxFragment newInstance(long id) {
    XxxFragment fragment = new XxxFragment();
    Bundle args = new Bundle();
    args.putLong(ARG_ID, id);
    fragment.setArguments(args);
    return fragment;
}
```

## 实施检查

- [ ] 是否没有定义带业务参数的自定义构造方法。
- [ ] 异步回调中是否确认 Fragment 仍处于有效生命周期。
- [ ] 是否没有在 `onDestroyView()` 后访问 Binding。

