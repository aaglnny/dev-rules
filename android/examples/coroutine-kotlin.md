# Kotlin 协程数据库异步完整示例

Kotlin 数据库异步统一使用协程，不使用 RxJava3。Dao 继续使用 Java 普通返回类型，Kotlin 调用层负责生命周期、IO 线程切换、UI 更新和异常处理。

## 依赖

依赖使用项目 `androidApi` 统一配置，不在业务模块硬编码版本：

```groovy
implementation androidApi.library.lifecycleruntime
implementation androidApi.library.kotlincoroutines
```

## Activity 公共成员

```kotlin
import androidx.lifecycle.lifecycleScope
import kotlinx.coroutines.CancellationException
import kotlinx.coroutines.Dispatchers
import kotlinx.coroutines.launch
import kotlinx.coroutines.withContext

private val dao by lazy {
    DatabaseManager
        .getInstance(applicationContext)
        .getXxxDao()
}
```

## 查询

```kotlin
private fun getData() {
    lifecycleScope.launch {
        try {
            val list = withContext(Dispatchers.IO) {
                dao.queryAll()
            }

            adapter.addAllAndClear(list)
            binding.tvEmpty.visibility =
                if (adapter.itemCount > 0) View.GONE else View.VISIBLE
        } catch (e: CancellationException) {
            throw e
        } catch (e: Exception) {
            ToastUtils.showShort("数据加载失败")
        }
    }
}
```

## 保存

```kotlin
private fun save() {
    val name = binding.etName.text.toString().trim()
    if (name.isBlank()) {
        ToastUtils.showShort("名称不能为空")
        return
    }

    val entity = XxxEntity().apply {
        this.name = name
    }

    lifecycleScope.launch {
        try {
            withContext(Dispatchers.IO) {
                dao.insert(entity)
            }

            ToastUtils.showShort("保存成功")
            finish()
        } catch (e: CancellationException) {
            throw e
        } catch (e: Exception) {
            ToastUtils.showShort("保存失败")
        }
    }
}
```

## 删除

删除前仍需按项目规则显示确认弹窗，用户确认后再调用本方法：

```kotlin
private fun delete(entity: XxxEntity) {
    lifecycleScope.launch {
        try {
            withContext(Dispatchers.IO) {
                dao.delete(entity)
            }

            ToastUtils.showShort("删除成功")
            getData()
        } catch (e: CancellationException) {
            throw e
        } catch (e: Exception) {
            ToastUtils.showShort("删除失败")
        }
    }
}
```

## Fragment 生命周期

Fragment 中涉及 Binding 的协程必须使用 `viewLifecycleOwner.lifecycleScope`，避免 View 销毁后继续更新界面：

```kotlin
private fun getData() {
    viewLifecycleOwner.lifecycleScope.launch {
        try {
            val list = withContext(Dispatchers.IO) {
                dao.queryAll()
            }

            adapter.addAllAndClear(list)
            binding.tvEmpty.visibility =
                if (adapter.itemCount > 0) View.GONE else View.VISIBLE
        } catch (e: CancellationException) {
            throw e
        } catch (e: Exception) {
            ToastUtils.showShort("数据加载失败")
        }
    }
}
```

## 实施检查

- [ ] Kotlin 文件是否没有使用 RxJava3 类型、订阅和线程调度代码。
- [ ] Activity 是否使用 `lifecycleScope`。
- [ ] Fragment 更新 Binding 时是否使用 `viewLifecycleOwner.lifecycleScope`。
- [ ] Dao 操作是否放在 `withContext(Dispatchers.IO)` 中。
- [ ] UI 是否在协程返回主线程后更新。
- [ ] 是否禁止使用 `GlobalScope` 和脱离生命周期的自定义 `CoroutineScope`。
- [ ] 是否单独处理并重新抛出 `CancellationException`。
- [ ] 是否使用安全空值处理，没有使用 `!!`。
