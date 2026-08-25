# Java RxJava3 数据库异步完整示例

Dao 保持同步普通返回类型，调用层使用 `Observable.create()` 包裹数据库操作，并在 IO 线程执行。

## 查询

```java
private void getData() {
    Observable.create((ObservableOnSubscribe<List<XxxEntity>>) emitter -> {
                try {
                    List<XxxEntity> list = DatabaseManager
                            .getInstance(getApplicationContext())
                            .getXxxDao()
                            .queryAll();
                    emitter.onNext(list);
                    emitter.onComplete();
                } catch (Exception e) {
                    emitter.tryOnError(e);
                }
            })
            .subscribeOn(Schedulers.io())
            .observeOn(AndroidSchedulers.mainThread())
            .subscribe(new SimpleObserver<List<XxxEntity>>() {
                @Override
                public void onNext(List<XxxEntity> list) {
                    adapter.addAllAndClear(list);
                    binding.tvEmpty.setVisibility(
                            adapter.getItemCount() > 0 ? View.GONE : View.VISIBLE);
                }

                @Override
                public void onError(Throwable e) {
                    super.onError(e);
                    ToastUtils.showShort("数据加载失败");
                }
            });
}
```

## 保存

```java
private void save() {
    String name = binding.etName.getText().toString().trim();
    if (TextUtils.isEmpty(name)) {
        ToastUtils.showShort("名称不能为空");
        return;
    }

    XxxEntity entity = new XxxEntity();
    entity.setName(name);

    Observable.create((ObservableOnSubscribe<Boolean>) emitter -> {
                try {
                    DatabaseManager
                            .getInstance(getApplicationContext())
                            .getXxxDao()
                            .insert(entity);
                    emitter.onNext(Boolean.TRUE);
                    emitter.onComplete();
                } catch (Exception e) {
                    emitter.tryOnError(e);
                }
            })
            .subscribeOn(Schedulers.io())
            .observeOn(AndroidSchedulers.mainThread())
            .subscribe(new SimpleObserver<Boolean>() {
                @Override
                public void onNext(Boolean success) {
                    ToastUtils.showShort("保存成功");
                    finish();
                }

                @Override
                public void onError(Throwable e) {
                    super.onError(e);
                    ToastUtils.showShort("保存失败");
                }
            });
}
```

## 实施检查

- [ ] 是否使用 `Observable.create()`，而不是 `Observable.just(dao.queryAll())`。
- [ ] 是否设置 `subscribeOn(Schedulers.io())` 和主线程 `observeOn()`。
- [ ] 异常是否通过 `onError()` 传递并恢复必要 UI 状态。
- [ ] 页面销毁时是否沿用项目现有 Disposable 管理。

