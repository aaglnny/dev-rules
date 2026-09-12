# LazyDataSource 完整示例

```typescript
@Observed
export default class LazyDataSource<T> implements IDataSource {
  private listeners: DataChangeListener[] = []
  dataArray: T[] = []

  totalCount(): number {
    return this.dataArray.length
  }

  getData(index: number): T {
    return this.dataArray[index]
  }

  registerDataChangeListener(listener: DataChangeListener): void {
    if (this.listeners.indexOf(listener) < 0) {
      this.listeners.push(listener)
    }
  }

  unregisterDataChangeListener(listener: DataChangeListener): void {
    const index: number = this.listeners.indexOf(listener)
    if (index >= 0) {
      this.listeners.splice(index, 1)
    }
  }

  pushArrayData(data: T[]): void {
    this.dataArray = data
    this.listeners.forEach((listener: DataChangeListener): void => {
      listener.onDataReloaded()
    })
  }
}
```

如果项目已经有 `LazyDataSource` 或 `LazyDataSourceV2`，直接复用已有实现，不在业务页面重复定义数据源。
