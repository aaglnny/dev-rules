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

## 普通跳转、刷新通知与结果回传

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

普通返回使用 `this.pathStack?.pop()`，不会触发跳转页面注册的 `onPop`。系统侧滑返回不携带结果，同样不能依赖 `onPop` 通知上一个页面。

只需要通知上一个页面刷新、不需要回传业务数据时，上一个页面在跳转时注册 `onPop`，收到回调后直接刷新，不读取 `popInfo.result`：

```typescript
private openEditor(): void {
  this.pathStack?.pushPath({
    name: BuilderNameConstants.EDIT_PAGE,
    param: {} as Record<string, Object>,
    onPop: (_popInfo: PopInfo) => {
      this.refresh()
    }
  })
}
```

目标页面完成操作后，使用固定标记触发 `onPop`：

```typescript
private save(): void {
  // 保存成功后通知上一个页面刷新
  this.pathStack?.pop('update')
}
```

需要回传业务数据时，上一个页面通过 `popInfo.result` 读取结果：

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

目标页面使用 `this.pathStack?.pop(result)` 返回业务结果。使用 `@Provider()` 和 `@Consumer()` 时不设置别名。
