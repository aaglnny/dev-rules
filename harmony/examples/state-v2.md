# 状态管理 V2 完整示例

```typescript
interface ListItem {
  id: number
  title: string
}

@Entry
@ComponentV2
export struct StatePage {
  @Local list: ListItem[] = []
  @Local selectedId: number = 0

  private select(id: number): void {
    this.selectedId = id
  }

  private remove(id: number): void {
    this.list = this.list.filter((item: ListItem): boolean => item.id !== id)
  }

  build(): void {
    Column() {
      ForEach(this.list, (item: ListItem) => {
        ItemCard({
          title: item.title,
          selected: this.selectedId === item.id,
          onItemClick: () => this.select(item.id)
        })
      }, (item: ListItem): string => item.id.toString())
    }
  }
}
```

数组更新使用 `filter`、`map` 或新数组赋值触发刷新。`@Builder` 内部可以直接读取所属组件的 `@Local`，但不能把 `@Local` 作为普通方法参数传入 `@Builder`；需要独立状态、独立生命周期或复杂交互的可复用区域，优先拆成 `@ComponentV2` 子组件。
