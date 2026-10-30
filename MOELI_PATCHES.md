# Moeli 维护说明

本分支供 Moeli 移动端与鸿蒙端共同使用，保留上游 Git 历史与 MIT LICENSE。

- 上游：https://github.com/AhmeedGamil/liquid_glass_easy
- 基线：`v5.1.0`，提交 `60de7a660a048a203d1011d9bdde93be3ba607ad`。
- 对应 pub.dev 发布包 SHA-256：`c2a476d4813d572661f2f8817d32183f4b4bc73db95bc491c6cea29b90e479d8`。
- 维护分支：`moeli-fixes`。应用依赖固定完整提交 SHA，不跟随分支名称自动变化。

## 本地补丁

修复以独立提交保留，只修改 `lib/src/widgets/lens/liquid_glass_shape_cache.dart`：

1. 几何缓存使用 `TargetPixelFormat.rgbaFloat32`。缓存的 RGBA 保存距离和梯度等精确数值，普通颜色目标可能使用的有损压缩会破坏数据，在部分 Android GPU 上表现为固定的脏污轮廓。
2. `maxTexels` 从 `4 << 20` 调为 `1 << 20`，保持单张 RGBA32F 纹理基础层的名义 16 MiB 上限。超限仍采用上游实时计算路径。

16 MiB 不是全局显存预算；Blender 的多张纹理与 mipmap 另计。缓存编码、解码、背景采样、批处理和视觉参数均沿用原实现。

该补丁需要支持上述 `toImageSync` 参数的 Flutter SDK。当前客户端使用 Flutter 3.44.8 与 Flutter OHOS 3.41.10；本 fork 未发布到 pub.dev。

## 升级流程

1. 获取上游选定的正式发布 tag，在 `moeli-fixes` 上合并。保留已有补丁提交及其历史，不强制覆盖维护分支；旧的应用固定提交应持续可获取。
2. 检查形状缓存的格式、编码和内存预算变化。发生冲突时按新版实现迁移修复；若上游已有等效修复，验证后采用上游实现。
3. 在移动端与鸿蒙端分别用各自工具链验证依赖解析、shader 加载、顶部 batch / 菜单 / 触摸相关定向用例；在适用的真实设备上检查静止、按压、菜单形变边缘。渲染实现有变化时再进行有针对性的性能复验。
4. 推送并验证新提交可获取后，将两个应用的 `pubspec.yaml` 一起固定到同一完整 SHA，分别运行依赖解析并提交更新后的 `pubspec.lock`。鸿蒙不要保留将此包指回本地副本的 override。

应用的菜单布局、交互和 batch 使用方式在应用仓库中维护，不混入本 fork。不要直接修改 Pub 缓存，也不要通过关闭缓存撤销已有修复或性能优化。
