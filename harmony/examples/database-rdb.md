# RDB 数据库完整示例

本示例参考 `hm_data_travel_260716_2014` 项目的 `ArticleModel`、`ArticleTable` 和 `ArticleListPage`，约束实体、表访问类和页面之间的职责边界。

## 数据库三层公共类

数据库功能必须接入项目已有的三个公共类，调用关系如下：

```text
ArticleListPage
    -> RdbTableImplGlobal.getInstance().getArticleTable()
    -> ArticleTable
    -> Rdb
    -> RdbCommon
```

- `Rdb.ets`：数据库基础封装，负责获取 `RdbStore`、执行统一初始化和处理公共数据库资源。
- `RdbCommon.ets`：数据库公共配置，负责集中定义表名、建表 SQL、列定义和表结构相关常量。
- `RdbTableImplGlobal.ets`：业务表统一入口，负责创建、注册和返回 `ArticleTable` 等业务表实例。
- `ArticleTable.ets`：具体业务表访问类，负责 Article 字段映射、`ValuesBucket`、`RdbPredicates`、CRUD 和查询结果转换。
- `ArticleListPage.ets`：页面通过 `RdbTableImplGlobal` 获取 `ArticleTable`，只负责触发查询和更新页面状态。

新增业务表时，先在 `RdbCommon.ets` 补充表配置，再创建对应的 `XxxTable.ets`，最后在 `RdbTableImplGlobal.ets` 注册统一获取方法。页面和 ViewModel 不得直接创建 `RdbStore`、`Rdb` 或 `XxxTable`。

## Model

Model 只描述业务字段和嵌套结构。字段默认值与表字段类型保持一致，新增数据使用可空主键区分新建记录。

```typescript
@ObservedV2
export class ArticleContentModel {
  type: string = ''
  ct: string = ''
  kind: number = 0
  cont: string = ''
}

@ObservedV2
export class ArticleModel {
  id: number | null = null
  type: number = 0
  kind: string = ''
  title: string = ''
  titleUrl: string = ''
  imgUrl: string = ''
  audioUrl: string = ''
  banner: string = ''
  desc: string = ''
  content: ArticleContentModel[] = []
  collect: boolean = false
  watchCount: number = 0
}
```

- Model 不直接持有 `RdbStore`，不在字段 getter 中查询数据库。
- 实体是否抽取到 `model/XxxModel.ets` 由项目根据实际复用范围和业务职责自行判断。
- 仅用于界面展示的计算值不要写回数据库字段；需要持久化的值必须有明确表列。

## Table 访问类

每个业务表使用一个 `XxxTable` 类，通过项目已有 `Rdb` 和 `RdbCommon` 获取表配置及数据库实例。

```typescript
import relationalStore from '@ohos.data.relationalStore'

function generateBucket(model: ArticleModel): relationalStore.ValuesBucket {
  const bucket: relationalStore.ValuesBucket = {}
  if (model.id !== null && model.id !== 0) {
    bucket.id = model.id
  }
  bucket.type = model.type
  bucket.kind = model.kind
  bucket.title = model.title
  bucket.content = JSON.stringify(model.content)
  bucket.collect = model.collect ? 1 : 0
  bucket.watchCount = model.watchCount
  return bucket
}
```

表访问类负责以下内容：

- `getRdbStore()` 和表初始化回调。
- Model 到 `ValuesBucket` 的字段映射。
- 单条插入、同步插入和批量插入。
- 按主键更新和删除。
- 按主键、分类等业务条件查询。
- `ResultSet` 到 Model 的字段映射。

更新和删除使用主键构造 `RdbPredicates`：

```typescript
const predicates = new relationalStore.RdbPredicates(tableName)
predicates.equalTo('id', model.id as number)
```

嵌套数组字段读取时恢复为明确类型，解析失败使用空数组：

```typescript
const contentText: string = resultSet.getString(resultSet.getColumnIndex('content'))
try {
  const content: ArticleContentModel[] = JSON.parse(contentText) as ArticleContentModel[]
  model.content = Array.isArray(content) ? content : []
} catch (error) {
  model.content = []
}
```

查询列表时固定排序并逐行移动游标：

```typescript
predicates.orderByAsc('id')
resultSet.goToFirstRow()
const result: ArticleModel[] = []
for (let index: number = 0; index < resultSet.rowCount; index++) {
  result.push(getArticleModel(resultSet))
  resultSet.goToNextRow()
}
```

实际项目中应按当前 SDK 文档和 `Rdb` 封装处理结果集释放、异常回调和同步/异步接口，不把原始 `ResultSet` 传给页面。

## 列表页面

列表页通过项目统一表实例查询，不直接调用 `relationalStore`：

```typescript
@ComponentV2
export struct ArticleListPage {
  @Local articleList: ArticleModel[] = []
  @Local kind: string = ''
  @Local keyword: string = ''

  private loadData(): void {
    const articleTable = RdbTableImplGlobal.getInstance().getArticleTable()
    articleTable.getRdbStore((): void => {
      articleTable.queryByKind(this.kind, (result: ArticleModel[]): void => {
        this.articleList = result
      })
    })
  }
}
```

- 路由参数在 `NavDestination` 的 `onReady` 中读取，拿到 `kind`、标题和关键词后再查询。
- 不要在 `aboutToAppear` 和 `onReady` 同时触发同一份数据加载，避免重复查询。
- 查询结果必须重新赋值给 `@Local` 数组，确保列表刷新。
- 分类筛选优先下沉到 `ArticleTable.queryByKind()`；小规模本地数据的关键词搜索可以先 `queryAll()` 再在页面过滤。
- 数据量增大后，关键词匹配应下沉到数据库谓词或专用查询方法。
- 空列表显示空状态；列表项点击只传递主键并跳转，详情页按主键重新查询完整文章。

## 检查清单

- [ ] Model、Table、ListPage 三层职责没有混在一起。
- [ ] 表字段与 Model 字段映射完整，布尔值和 JSON 字段转换统一。
- [ ] 新增数据不写入无效主键，更新和删除使用明确主键。
- [ ] 查询有稳定排序，空结果返回空数组或明确的 `undefined`。
- [ ] `ResultSet` 已按 SDK 要求遍历和释放。
- [ ] 列表结果重新赋值给 `@Local`，没有原地修改后等待 UI 猜测变化。
- [ ] 已复用 `Rdb.ets`、`RdbCommon.ets` 和 `RdbTableImplGlobal.ets`，没有另起数据库基础封装、表配置或表实例管理方式。
- [ ] 页面没有直接创建 `RdbStore`、`Rdb`、`XxxTable`，没有绕过 `RdbTableImplGlobal`、拼接 SQL 或重复加载同一数据。
- [ ] 详情页根据主键重新查询，不依赖列表页传递完整实体。
