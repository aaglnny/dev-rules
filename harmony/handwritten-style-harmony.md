# 鸿蒙手写风格补充规范

通用代码表达、封装克制、注释、命名和修改范围遵循 [handwritten-style-general.md](../handwritten-style-general.md)。ArkUI 状态、组件、事件和 Builder 的平台规则由 [harmony-arkui.md](harmony-arkui.md) 维护，项目工具复用由 [harmony-project.md](harmony-project.md) 和 [harmony-system-api.md](harmony-system-api.md) 维护。

## ArkUI 代码表达

- 组件树保持业务层级清晰，业务判断靠近对应 UI，不为形式统一拆出只使用一次的 Builder 或子组件。
- ArkUI 属性链每个方法单独一行，缩进和换行节奏沿用当前文件。
- 页面已有稳定布局写法时局部延续，不借功能修改批量重排组件层级或属性顺序。
- 颜色、字号、间距和图片优先使用项目资源；已有模块稳定使用数字时不做无关统一。
- 注释使用中文，只说明平台限制、兼容原因、生命周期约束或容易误删的取舍。
- 不把中文硬编码改成 Unicode 转义，不通过多余中间变量和辅助方法制造整齐感。

本文件只维护鸿蒙代码的表达方式，不重复定义平台行为、工程结构或具体系统 API。
