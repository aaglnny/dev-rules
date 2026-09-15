# @CustomDialog 完整示例

`@CustomDialog` 使用 V1 状态装饰器规则，不能把内部状态改成 `@Local`。

```typescript
@CustomDialog
export struct ConfirmDialog {
  controller: CustomDialogController
  @State title: string = ''
  @State message: string = ''
  onConfirm?: () => void

  build(): void {
    Column() {
      Text(this.title)
      Text(this.message)
      Row() {
        Button('取消')
          .onClick(() => {
            this.controller?.close()
          })
        Button('确定')
          .onClick(() => {
            this.onConfirm?.()
            this.controller?.close()
          })
      }
    }
  }
}
```

弹窗只处理 UI 和结果回调。数据库写入、网络请求和页面跳转由调用页面负责。
