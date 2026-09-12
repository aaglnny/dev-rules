# 文件操作完整示例

文件操作优先使用脚手架 `entry/src/main/ets/utils/FileUtils.ets`，导入路径按目标文件所在目录调整。

```typescript
import { common } from '@kit.AbilityKit'
import { FileUtils } from '../utils/FileUtils'

private async copyToCache(sourcePath: string): Promise<string> {
  const context = this.getUIContext().getHostContext() as common.UIAbilityContext
  return await FileUtils.copyFileToSandbox(context, sourcePath)
}

private async saveFiles(paths: string[]): Promise<boolean> {
  const context = this.getUIContext().getHostContext() as common.UIAbilityContext
  return await FileUtils.saveFilesToPublic(context, paths)
}
```

按实际功能复用以下方法：

- `copyFile()`：复制文件。
- `copyRawFileToFile()`：把资源文件写入沙箱。
- `copyFileToSandbox()`：把文件复制到应用缓存目录。
- `writeFile()`：写入 `ArrayBuffer` 或字符串。
- `saveFilesToPublic()`：通过系统文件保存界面把一个或多个文件保存到公共目录。
- `isImage()`、`isVideo()`、`isAudio()`：判断文件类型。
- `getFileNameFromPath()`、`getFileExtension()`、`formatFileSize()`：处理文件信息。

持久化文件使用 `context.filesDir`，临时文件使用 `context.cacheDir`。只有 `FileUtils` 没有覆盖当前需求时才直接调用 `fileIo`；直接打开的文件必须在成功和异常路径都关闭。
