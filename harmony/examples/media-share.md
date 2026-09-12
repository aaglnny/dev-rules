# 媒体选择、相册与分享完整示例

媒体选择和相册保存优先使用脚手架 `PhotoPickerUtils.ets` 与 `FileUtils.ets`，导入路径按目标文件所在目录调整。

```typescript
import { common } from '@kit.AbilityKit'
import { FileUtils } from '../utils/FileUtils'
import { PhotoPickerUtils } from '../utils/PhotoPickerUtils'

private async selectImages(): Promise<void> {
  const uris: string[] = await PhotoPickerUtils.selectImage(9)
  if (uris.length === 0) {
    return
  }
  this.imageList = uris
}

private async selectVideo(): Promise<void> {
  const uris: string[] = await PhotoPickerUtils.selectVideo(1)
  if (uris.length > 0) {
    this.videoUri = uris[0]
  }
}

private async selectAudio(): Promise<void> {
  const uris: string[] = await PhotoPickerUtils.selectAudio()
  if (uris.length > 0) {
    this.audioUri = uris[0]
  }
}

private async saveImage(imagePath: string): Promise<boolean> {
  const context = this.getUIContext().getHostContext() as common.UIAbilityContext
  return await FileUtils.saveMediaToGallery(context, imagePath)
}
```

其他常用能力：

- 多张图片保存到相册使用 `FileUtils.saveImageList(this.getUIContext(), paths)`。
- 文档选择使用 `PhotoPickerUtils.openDocumentPicker(context, filters, max)`。
- 查询相册图片或视频使用 `getPhotosFromAlbum()`、`getVideosFromAlbum()`。
- 获取视频缩略图和时长使用 `getVideoThumbnail()`、`getAssetDuration()`。
- 删除媒体资产使用 `deleteAssets()`，调用前必须有明确的用户操作。

媒体选择需要处理用户取消后返回空数组的情况。保存相册和读取媒体资产前，按脚手架调用方式及当前 SDK 核对权限和 API 版本。系统分享继续使用当前项目已有分享封装和系统可访问 URI，不直接暴露应用内部路径。
