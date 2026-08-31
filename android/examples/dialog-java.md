# Java 原生 DataBinding Dialog 完整示例

自定义弹窗统一使用 Android 原生 `Dialog` 和 DataBinding。Dialog 只负责 UI 初始化与结果回调，不执行页面跳转、数据库写入或网络请求。

## 完整示例

```java
package com.xxx.project_name.widget.dialog;

import android.app.Dialog;
import android.content.Context;
import android.content.DialogInterface;
import android.graphics.Color;
import android.graphics.drawable.ColorDrawable;
import android.os.Bundle;
import android.view.Gravity;
import android.view.LayoutInflater;
import android.view.View;
import android.view.Window;
import android.view.WindowManager;

import androidx.databinding.DataBindingUtil;

import com.blankj.utilcode.util.ScreenUtils;
import com.blankj.utilcode.util.SizeUtils;
import com.xxx.project_name.R;
import com.xxx.project_name.databinding.DialogXxxSelectBinding;

public class XxxSelectDialog extends Dialog {
    private final DialogInterface.OnClickListener listener;
    private DialogXxxSelectBinding binding;

    public XxxSelectDialog(Context context, DialogInterface.OnClickListener listener) {
        super(context);
        this.listener = listener;
    }

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        Window window = getWindow();
        if (window == null) {
            return;
        }
        window.setSoftInputMode(WindowManager.LayoutParams.SOFT_INPUT_ADJUST_RESIZE | WindowManager.LayoutParams.SOFT_INPUT_STATE_HIDDEN);
        window.setGravity(Gravity.CENTER);
        window.getDecorView().setPadding(0, 0, 0, 0);
        window.setBackgroundDrawable(new ColorDrawable(Color.TRANSPARENT));
        window.requestFeature(Window.FEATURE_NO_TITLE);

        WindowManager.LayoutParams params = window.getAttributes();
        params.width = ScreenUtils.getScreenWidth() - SizeUtils.dp2px(40f);
        window.setAttributes(params);

        binding = DataBindingUtil.inflate(
                LayoutInflater.from(getContext()),
                R.layout.dialog_xxx_select,
                null,
                false);
        setContentView(binding.getRoot());

        initView();
        binding.setOnClickListener(this::onItemViewClick);
    }

    private void initView() {
        // 初始化弹窗展示数据。
    }

    private void onItemViewClick(View view) {
        int id = view.getId();
        if (id == R.id.tv_option_1) {
            if (listener != null) {
                listener.onClick(this, 0);
            }
        } else if (id == R.id.tv_option_2) {
            if (listener != null) {
                listener.onClick(this, 1);
            }
        }
        dismiss();
    }
}
```

## 显示弹窗

```java
new XxxSelectDialog(this, (dialog, which) -> {
    if (which == 0) {
        selectFirstItem();
    } else if (which == 1) {
        selectSecondItem();
    }
}).show();
```

## 实施检查

- [ ] 是否使用 DataBinding inflate 并通过根视图设置内容。
- [ ] 是否配置 Window 位置、宽度、软键盘模式和透明背景。
- [ ] 选项点击后是否默认关闭 Dialog。
- [ ] Dialog 是否没有执行页面跳转、网络或数据库操作。

