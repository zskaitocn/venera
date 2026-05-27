# Venera iOS 12 兼容构建文档

本分支从 v1.0.0-beta 派生，修改了 iOS 部署目标，使其可在 iOS 12.5.8 (iPad 5th gen) 上运行。

## 修改内容

### 1. Podfile (`ios/Podfile`)
- `platform :ios, '14.0'` → `platform :ios, '12.0'`
- 新增 `post_install` hook，强制所有 pod target 为 12.0

### 2. Xcode 项目 (`ios/Runner.xcodeproj/project.pbxproj`)
- 全部 3 处 `IPHONEOS_DEPLOYMENT_TARGET = 14.0` → `IPHONEOS_DEPLOYMENT_TARGET = 12.0`

### 3. Info.plist (`ios/Runner/Info.plist`)
- 添加 `<key>MinimumOSVersion</key><string>12.0</string>`
- 移除 iOS 14+ 专属键：`CADisableMinimumFrameDurationOnPhone`, `UIApplicationSuppportsIndirectInputEvents`

### 4. AppDelegate.swift (`ios/Runner/AppDelegate.swift`)
- `import UniformTypeIdentifiers` → `import MobileCoreServices`（iOS 8-14 兼容）
- `UIDocumentPickerViewController(forOpeningContentTypes:...)` → `UIDocumentPickerViewController(documentTypes:..., in: .open)`（iOS 14 新方法在 iOS 12 不可用）

### 5. pubspec.yaml
- `url_launcher: ^6.3.0` → `url_launcher: ^6.2.0`（v6.3.5+ 要求 iOS 13+）

## 环境要求

| 工具 | 最低版本 | 备注 |
|------|-----------|------|
| Flutter SDK | 3.24.4 | v1.0.0-beta 原始版本，支持 iOS 12 |
| Xcode | 13.x 或 14.x | 需支持 iOS 12 SDK（Xcode 15 移除了 iOS 12 SDK） |
| macOS | 12.x (Monterey) 或更高 | 需要能运行兼容 Xcode 的 macOS 版本 |
| CocoaPods | 1.15+ | `pod install` 需正常运行 |

> ⚠️ Xcode 15+ 不再包含 iOS 12 SDK，如需真机调试，建议使用 Xcode 14.x。

## 构建步骤（在 Mac 虚拟机上执行）

### 第一步：安装 Flutter 3.24.4
```bash
# 下载 Flutter 3.24.4
cd ~/sdk
git clone https://github.com/flutter/flutter.git -b 3.24.4 --depth=1 flutter-3.24.4
export PATH="$HOME/sdk/flutter-3.24.4/bin:$PATH"
flutter --version  # 应显示 3.24.4
```

### 第二步：安装依赖
```bash
cd /path/to/venera
git checkout ios12-compat
flutter pub get
```

### 第三步：iOS 构建
```bash
cd ios
pod deintegrate
pod install --repo-update
cd ..
flutter build ios --release --no-codesign
```

### 第四步：签名并安装到 iPad
```bash
# 在 Xcode 中打开 ios/Runner.xcworkspace
# 登录你的 Apple ID
# 设置 Bundle Identifier（修改为你的唯一 ID）
# 连接 iPad（iOS 12.5.8）
# 选择设备 → Product → Run
```

## 已知限制

1. **Swift 运行时**：Flutter 3.24.4 在 iOS 12 上会自动 bundle Swift 运行时库（较大），这是正常的
2. **`webview` 功能**：`flutter_inappwebview` 在 iOS 12 上部分 API 可能不可用，但基础渲染应工作
3. **深色模式**：iOS 12 不支持系统级深色模式，App 内深色主题可以正常使用
4. **性能**：iPad 5th gen (A9 芯片) 性能有限，超长漫画列表可能卡顿

## 在 iOS 12 上测试的功能点

- [ ] 本地漫画文件读取
- [ ] 图片渲染与缩放
- [ ] 翻页手势
- [ ] 设置页面
- [ ] WebView 内嵌网页
- [ ] 文件选择器（目录选择）
- [ ] 深色模式切换
- [ ] 收藏功能

## 后续升级建议

如果未来 Apple 强制要求更高 iOS 版本，建议：
- 在 `v1.0.x` 分支上继续维护 iOS 12 兼容版本
- 主线版本可安全升级到 `IPHONEOS_DEPLOYMENT_TARGET = 13.0`（覆盖 99%+ 用户）
