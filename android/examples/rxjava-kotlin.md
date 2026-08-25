# Kotlin RxJava3 数据库异步完整示例

Dao 使用 Java 普通返回类型，Kotlin 调用层负责封装 Observable、线程切换和 UI 更新。

## 查询

```kotlin
private fun getData() {
    Observable.create<List<XxxEntity>> { emitter ->
        try {
            val list = DatabaseManager
                .getInstance(applicationContext)
                .getXxxDao()
                .queryAll()
            emitter.onNext(list)
            emitter.onComplete()
        } catch (e: Exception) {
            emitter.tryOnError(e)
        }
    }
        .subscribeOn(Schedulers.io())
        .observeOn(AndroidSchedulers.mainThread())
        .subscribe(object : SimpleObserver<List<XxxEntity>>() {
            override fun onNext(list: List<XxxEntity>) {
                adapter.addAllAndClear(list)
                binding.tvEmpty.visibility =
                    if (adapter.itemCount > 0) View.GONE else View.VISIBLE
            }

            override fun onError(e: Throwable) {
                super.onError(e)
                ToastUtils.showShort("数据加载失败")
            }
        })
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

    Observable.create<Unit> { emitter ->
        try {
            DatabaseManager
                .getInstance(applicationContext)
                .getXxxDao()
                .insert(entity)
            emitter.onNext(Unit)
            emitter.onComplete()
        } catch (e: Exception) {
            emitter.tryOnError(e)
        }
    }
        .subscribeOn(Schedulers.io())
        .observeOn(AndroidSchedulers.mainThread())
        .subscribe(object : SimpleObserver<Unit>() {
            override fun onNext(value: Unit) {
                ToastUtils.showShort("保存成功")
                finish()
            }

            override fun onError(e: Throwable) {
                super.onError(e)
                ToastUtils.showShort("保存失败")
            }
        })
}
```

## 实施检查

- [ ] 是否使用安全空值处理，不用 `!!`。
- [ ] 是否没有使用 `Observable.just()` 直接包裹 Dao 调用。
- [ ] 是否完成 IO 线程和主线程切换。
