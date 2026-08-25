# 鸿蒙项目开发规则

这份规则根据你当前正在学习的鸿蒙项目整理，目标是让后续开发其他鸿蒙项目时，能够按同一套框架、同一套路由方式、同一套数据加载方式快速推进。

## 1. 项目入口规则

- 启动页只负责初始化、合规处理和跳转主页面。
- 不要让启动页承担复杂业务展示。
- 首屏建议保持单一入口，避免多个入口分散初始化逻辑。

参考实现：
- [EntryAbility.ets](E:/DevEcoStudioProjects/hm_study_english_260423_1916/entry/src/main/ets/entryability/EntryAbility.ets)
- [LaunchPage.ets](E:/DevEcoStudioProjects/hm_study_english_260423_1916/entry/src/main/ets/pages/launcher/LaunchPage.ets)

## 2. 主框架规则

- 主页面负责底部导航和页面容器，不负责承载大量业务逻辑。
- 主页面建议统一提供两个核心上下文：
  - 页面栈 `pathStack`
  - 数据刷新信号 `refreshNetworkData`
- 底部 Tab 的数量、顺序和名称建议先定死，再围绕它扩展子页面。

参考实现：
- [MainPage.ets](E:/DevEcoStudioProjects/hm_study_english_260423_1916/entry/src/main/ets/pages/main/MainPage.ets)

## 3. 路由跳转规则

- 路由名统一放在常量文件里管理，不要直接散落在页面代码中。
- 主框架内部页面跳转优先使用 `NavDestination + pathStack.pushPath()`。
- 独立页面、启动页、合规页可以使用 `router.pushNamedRoute()` 或 `router.replaceUrl()`。
- 返回统一用 `pathStack.pop()`。
- 跳转参数优先通过 `param` 传递，不要依赖全局变量。
- 使用声明式路由时，新增页面必须同步维护 `route_map.json`。
- `route_map.json` 需要把 `name`、`pageSourceFile`、`buildFunction` 三项对应起来。
- 新增页面时建议按这个顺序处理：
  1. 先补 `BuilderNameConstants`
  2. 再补 `route_map.json`
  3. 最后在页面里调用 `pathStack.pushPath({ name: ... })`
- 页面跳转统一先构造 `Record<string, Object>` 类型的 `param`，再传给 `pathStack.pushPath()`。
- `Record<string, Object>` 不要使用 `new Object() as Record<string, Object>` 后再逐项赋值，统一使用带类型声明的对象字面量。
- 参数对象变量名建议统一为 `param`，避免同一项目里混用 `routeParam`、`params` 等名称。

代码示例：

```typescript
private toAudioList(sort: number, type: number, title: string): void {
  const param: Record<string, Object> = {
    'sort': sort,
    'type': type,
    'title': title,
  }

  this.pathStack?.pushPath({
    name: BuilderNameConstants.HARA_AUDIO_LIST,
    param: param
  })
}
```

```typescript
private toAudio(index: number): void {
  const param: Record<string, Object> = {
    'audioList': this.audioList,
    'position': index,
    'type': this.audioType,
  }

  this.pathStack?.pushPath({
    name: BuilderNameConstants.HARA_AUDIO,
    param: param
  })
}
```

参考实现：
- [RouterConstants.ets](E:/DevEcoStudioProjects/hm_study_english_260423_1916/entry/src/main/ets/common/RouterConstants.ets)
- [Home02.ets](E:/DevEcoStudioProjects/hm_study_english_260423_1916/entry/src/main/ets/pages/main/Home02.ets)
- [VideoListPage.ets](E:/DevEcoStudioProjects/hm_study_english_260423_1916/entry/src/main/ets/pages/video/VideoListPage.ets)
- [StoryListPage.ets](E:/DevEcoStudioProjects/hm_study_english_260423_1916/entry/src/main/ets/pages/story/StoryListPage.ets)
- [route_map.json](E:/DevEcoStudioProjects/hm_study_english_260423_1916/entry/src/main/resources/base/profile/route_map.json)

## 4. 页面职责规则

- 每个页面只负责一类职责。
- 首页负责聚合展示。
- 列表页负责数据列表。
- 详情页负责单条内容展示。
- 设置页负责说明和配置。

推荐拆法：
- 页面壳负责布局和路由容器。
- `@Builder` 负责拆分子块。
- 业务方法负责数据读取和跳转。

## 5. 数据加载规则

- 数据加载统一交给 `MainViewModel` 或同类 ViewModel。
- 页面不要重复拉同一份网络数据。
- 数据初始化推荐分两步：
  1. 先初始化本地数据库表
  2. 再按顺序请求网络数据并写入本地表
- 页面展示时优先查本地表，不直接依赖网络返回。
- 初始化完成后必须有明确完成信号，方便子页面刷新。

参考实现：
- [MainViewModel.ets](E:/DevEcoStudioProjects/hm_study_english_260423_1916/entry/src/main/ets/viewModel/MainViewModel.ets)

## 6. 数据表规则

- 每一种业务实体单独建表。
- 模型字段尽量和原始数据源保持一致，减少转换成本。
- 复杂内容字段建议用 JSON 字符串存储，读取时再还原。
- 列表页和详情页尽量读取同一张表，避免数据不一致。

参考实现：
- [RdbTableImplGlobal.ets](E:/DevEcoStudioProjects/hm_study_english_260423_1916/entry/src/main/ets/database/RdbTableImplGlobal.ets)
- [StoryTable.ets](E:/DevEcoStudioProjects/hm_study_english_260423_1916/entry/src/main/ets/database/tables/StoryTable.ets)
- [MediaTable.ets](E:/DevEcoStudioProjects/hm_study_english_260423_1916/entry/src/main/ets/database/tables/MediaTable.ets)
- [ArticleTable.ets](E:/DevEcoStudioProjects/hm_study_english_260506_1694/entry/src/main/ets/database/tables/ArticleTable.ets)

## 7. 状态管理规则

- 页面状态优先使用 `@Local`。
- 父子共享信号优先使用 `@Provider / @Consumer`。
- 页面需要随着外部信号刷新时，优先使用监听式刷新，而不是手动全局通知。
- 列表更新尽量通过重新赋值触发刷新。

参考实现：
- [MainPage.ets](E:/DevEcoStudioProjects/hm_study_english_260423_1916/entry/src/main/ets/pages/main/MainPage.ets)
- [Home02.ets](E:/DevEcoStudioProjects/hm_study_english_260423_1916/entry/src/main/ets/pages/main/Home02.ets)

## 8. 页面布局规则

```typescript
//基本文字样式
Text('内容')
  .fontSize($r('app.float.fp_16'))
  .fontColor($r('app.color.base_text_color'))

```

```typescript
//基本页面背景颜色设置
Column() {

}
.backgroundColor($r('app.color.base_bg_color'))

```


- 常见页面结构建议是：
  - 顶部标题区
  - 中部卡片区
  - 列表区或功能入口区
  - 底部留白
- 大背景图 + 白色圆角卡片是这个项目常用视觉风格。
- 列表页和详情页建议保持统一的间距、字号和圆角。
- 空状态必须存在，至少提供一张占位图和一句提示。

参考实现：
- [Home02.ets](E:/DevEcoStudioProjects/hm_study_english_260423_1916/entry/src/main/ets/pages/main/Home02.ets)
- [VideoListPage.ets](E:/DevEcoStudioProjects/hm_study_english_260423_1916/entry/src/main/ets/pages/video/VideoListPage.ets)
- [StoryListPage.ets](E:/DevEcoStudioProjects/hm_study_english_260423_1916/entry/src/main/ets/pages/story/StoryListPage.ets)
- [HomeMy.ets](E:/DevEcoStudioProjects/hm_study_english_260423_1916/entry/src/main/ets/pages/main/HomeMy.ets)

## 9. 组件拆分规则

- 复杂页面必须拆成多个 `@Builder` 小块。
- 每个 Builder 只负责一块 UI，命名要直接表达用途。
- 可复用卡片、菜单、标题栏尽量做成独立组件。
- 点击事件尽量通过参数传递，不要把业务逻辑写死在 Builder 里。

参考实现：
- [Home01.ets](E:/DevEcoStudioProjects/hm_study_english_260506_1694/entry/src/main/ets/pages/main/Home01.ets)
- [BaseTitleBar_V2.ets](E:/DevEcoStudioProjects/hm_study_english_260423_1916/entry/src/main/ets/view/BaseTitleBar_V2.ets)
- [CustomConfirmDialog.ets](E:/DevEcoStudioProjects/hm_study_english_260423_1916/entry/src/main/ets/view/dialog/CustomConfirmDialog.ets)

## 10. Toast 提示规则

- `entry` 模块内需要显示 Toast 时，统一使用 `entry/src/main/ets/common/Constants.ets` 中导出的 `showToast` 方法。
- 页面、组件、工具类不要直接调用 `promptAction.showToast()`，避免上下文获取方式分散。
- `showToast` 支持 `string | Resource`，普通提示直接传字符串即可。

代码示例：

```typescript
import { showToast } from '../../common/Constants'

private saveData(): void {
  if (this.name.length === 0) {
    showToast('请输入名称')
    return
  }

  showToast('保存成功')
}
```
## 11. 代码风格规则

- 变量和方法名要直观，避免过度缩写。
- 页面结构建议保持“先 build，后 helper，后数据方法”的顺序。
- 一页文件太长时，优先拆辅助 Builder 或子组件。
- 字符串优先直接写在页面里，除非项目明确要求做国际化。

## 12. 事件和跳转规则

- 列表项点击后只负责跳转或触发回调，不要顺手做额外业务。
- 详情页进入时优先接收上页参数，再按 `id` 或关键字段重新查本地表。
- 返回按钮行为建议统一由标题栏或页面统一处理。
- 需要结果回传时，明确使用 `pop(result)`。

参考实现：
- [StoryListPage.ets](E:/DevEcoStudioProjects/hm_study_english_260423_1916/entry/src/main/ets/pages/story/StoryListPage.ets)
- [VideoListPage.ets](E:/DevEcoStudioProjects/hm_study_english_260423_1916/entry/src/main/ets/pages/video/VideoListPage.ets)
- [NoteEditPage.ets](E:/DevEcoStudioProjects/hm_study_english_260423_1916/entry/src/main/ets/pages/note/NoteEditPage.ets)

## 13. 资源使用规则

- 图片、图标、背景、占位图要先整理成稳定资源命名。
- 同一类资源尽量统一尺寸和用途。
- 非必要时，展示图标资源的 `Image` 组件不要同时设置固定宽度和固定高度。
- 图标资源只固定一个方向的尺寸，另一个方向使用 `'auto'`，避免资源比例被拉伸或压缩。
- 常规图标推荐写法：固定 `.width(...)`，`.height('auto')`；如果业务场景按高度对齐，则固定 `.height(...)`，`.width('auto')`。
- 只有在明确需要裁剪、铺满固定容器，或设计稿要求强制变形时，才允许同时设置固定宽高。
- 页面封面图和占位图建议提前规划好。

## 14. 业务扩展规则

- 新增一个业务模块时，建议按下面顺序补齐：
  1. 模型
  2. 数据表
  3. 数据初始化
  4. 列表页
  5. 详情页
  6. 路由常量
  7. `route_map.json`
  8. 跳转入口
- 先保证能查到数据并展示，再补交互和优化。
- 先保证闭环，再补动画和细节。

## 15. 推荐开发顺序

1. 确认项目入口和页面流向。
2. 整理路由常量和页面树。
3. 补数据模型和本地表。
4. 补 ViewModel 初始化和网络加载链路。
5. 完成列表页和详情页。
6. 补点击跳转和参数传递。
7. 最后统一视觉和细节。

## 16. 日常检查清单

- 这个页面有没有明确职责？
- 这个页面的数据是不是来自本地表？
- 路由名是不是统一管理？
- 这个跳转是不是走 `pathStack`？
- 初始化是不是由主页面统一触发？
- 数据有没有“先入库，再展示”？
- 列表页有没有空状态？
- 详情页有没有重新按 `id` 查一次完整数据？
- 布局是不是遵循这个项目的卡片化风格？
- 这个功能是不是已经形成完整闭环？

## 17. 一句话原则

- 入口统一，路由统一，数据统一，展示分层，页面专职，列表查库，详情重查，状态上收，组件下沉。


