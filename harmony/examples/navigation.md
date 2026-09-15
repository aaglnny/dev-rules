# 声明式路由完整示例

## 路由配置

`module.json5`：

```json5
{
  "routerMap": "$profile:route_map"
}
```

`route_map.json`：

```json
{
  "routerMap": [
    {
      "name": "main://DetailPage",
      "pageSourceFile": "src/main/ets/pages/detail/DetailPage.ets",
      "buildFunction": "DetailPageBuilder"
    }
  ]
}
```

页面导出 Builder：

```typescript
import { BaseTitleBar_V2 } from '../view/BaseTitleBar_V2'

@Builder
export function DetailPageBuilder(): void {
  DetailPage()
}

@ComponentV2
export struct DetailPage {
  private pathStack?: NavPathStack = undefined

  build(): void {
    NavDestination() {
      Column() {
        BaseTitleBar_V2({
          titleBarAttribute: {
            backShow: true,
            title: '详情',
            backCallback: () => {
              this.pathStack?.pop()
            }
          }
        }).margin({ top: this.getUIContext().px2vp(globalThis.statusHeight) })

        Text('详情内容')
      }
    }
    .hideTitleBar(true)
    .onReady((context: NavDestinationContext) => {
      this.pathStack = context.pathStack
    })
  }
}
```

## 普通跳转与结果回传

```typescript
private openDetail(id: number, title: string): void {
  this.pathStack?.pushPath({
    name: BuilderNameConstants.DETAIL_PAGE,
    param: {
      'id': id,
      'title': title
    } as Record<string, Object>
  })
}
```

需要回传结果时统一使用带 `onPop` 的 `this.pathStack?.pushPath({ name, param, onPop })`。普通跳转不增加额外回调包装。

```typescript
interface SaveResult {
  saved: boolean
}

private openEditor(): void {
  this.pathStack?.pushPath({
    name: BuilderNameConstants.EDIT_PAGE,
    param: {
      'mode': 'edit'
    } as Record<string, Object>,
    onPop: (popInfo: PopInfo) => {
      const result: SaveResult = popInfo.result as SaveResult
      if (result.saved) {
        this.refresh()
      }
    }
  })
}
```

返回当前页：`this.pathStack?.pop()`；返回结果：`this.pathStack?.pop(result)`。使用 `@Provider()` 和 `@Consumer()` 时不设置别名。
