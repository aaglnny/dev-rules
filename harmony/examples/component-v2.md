# ArkUI V2 子组件完整示例

组件只负责展示传入状态和回传用户操作，业务请求和页面跳转由父组件处理。

```typescript
@ComponentV2
export struct ItemCard {
  @Param title: string = ''
  @Param selected: boolean = false
  @Event onItemClick: () => void

  build(): void {
    Row() {
      Text(this.title)
        .fontColor(this.selected ? $r('app.color.mainPage_selected') : $r('app.color.mainPage_normal'))
    }
    .onClick((): void => {
      this.onItemClick()
    })
  }
}
```

如果组件需要根据自身状态独立刷新，使用 `@Local`；如果只是接收父组件状态，不把父状态复制成另一份本地状态。
