# ArkUI V2 页面完整示例

适用于使用 `@ComponentV2` 的普通数据库列表页。页面在 `onReady` 中获取 `NavPathStack` 和路由参数，直接通过 `RdbTableImplGlobal` 查询数据库，列表项内容使用 `@Builder` 抽取。

```typescript
import CommonConstants from '@vhm/basecommon/src/main/ets/common/CommonConstants'

import { BuilderNameConstants } from '../common/RouterConstants'
import { RdbTableImplGlobal } from '../database/RdbTableImplGlobal'
import { ArticleModel } from '../model/ArticleModel'
import { BaseTitleBar_V2 } from '../view/BaseTitleBar_V2'

@Entry
@ComponentV2
export struct ArticleListPage {
  private pathStack?: NavPathStack = undefined
  @Local list: ArticleModel[] = []
  @Local pageTitle: string = '文章列表'
  @Local kind: string = ''

  build(): void {
    NavDestination() {
      Column() {
        BaseTitleBar_V2({
          titleBarAttribute: {
            backShow: true,
            title: this.pageTitle,
            backCallback: () => {
              this.pathStack?.pop()
            }
          }
        }).margin({ top: this.getUIContext().px2vp(globalThis.statusHeight) })

        if (this.list.length === 0) {
          this.buildEmptyState()
        } else {
          this.buildList()
        }
      }
      .width(CommonConstants.COMPONENT_PROPORTION_100)
      .height(CommonConstants.COMPONENT_PROPORTION_100)
    }
    .hideTitleBar(true)
    .onReady((context: NavDestinationContext) => {
      this.pathStack = context.pathStack
      const param: Record<string, Object> = context.pathInfo.param as Record<string, Object>
      this.pageTitle = param['title'] as string
      this.kind = param['kind'] as string
      this.loadData()
    })
  }

  @Builder
  private buildList(): void {
    List() {
      ForEach(this.list, (item: ArticleModel) => {
        ListItem() {
          this.buildListItem(item)
        }
      }, (item: ArticleModel): string => (item.id as number).toString())
    }
    .width(CommonConstants.COMPONENT_PROPORTION_100)
    .layoutWeight(1)
    .scrollBar(BarState.Off)
  }

  @Builder
  private buildListItem(item: ArticleModel): void {
    Text(item.title)
      .fontSize(14)
      .width(CommonConstants.COMPONENT_PROPORTION_100)
      .onClick(() => {
        this.openDetail(item)
      })
  }

  @Builder
  private buildEmptyState(): void {
    Column() {
      Text('暂无文章数据')
        .fontSize(13)
    }
    .width(CommonConstants.COMPONENT_PROPORTION_100)
    .alignItems(HorizontalAlign.Center)
    .padding({
      top: 80,
      bottom: 24
    })
  }

  private loadData(): void {
    const table = RdbTableImplGlobal.getInstance().getArticleTable()
    table?.getRdbStore(() => {
      table.queryByKind(this.kind, (result: ArticleModel[]) => {
        this.list = result
      })
    })
  }

  private openDetail(item: ArticleModel): void {
    this.pathStack?.pushPath({
      name: BuilderNameConstants.DETAIL_PAGE,
      param: {
        'id': item.id as number
      } as Record<string, Object>
    })
  }
}
```

页面直接查询数据库时，仍必须通过 `RdbTableImplGlobal` 获取业务表访问类；页面只负责触发查询、更新 `@Local` 列表和处理交互。实际项目中标题栏优先沿用 `BaseTitleBar_V2.ets`，数据库调用沿用项目已有的 `Rdb.ets`、`RdbCommon.ets` 和 `RdbTableImplGlobal.ets`。
