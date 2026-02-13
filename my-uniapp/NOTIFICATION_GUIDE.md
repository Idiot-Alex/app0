# 通知功能使用说明

## 功能概述

主界面已添加"发送系统通知"按钮，支持在不同平台发送通知：

- **App（Android/iOS）**：使用 5+ 引擎发送本地系统通知
- **H5**：使用浏览器 Notification API
- **小程序**：提示使用服务端订阅消息

## 运行测试

### 1. H5 测试（浏览器）
```bash
cd my-uniapp
npm run dev:h5
```
在浏览器中打开后，点击按钮会请求通知权限并发送测试通知。

### 2. App 测试（需要 HBuilderX）

**真机调试：**
1. 手机开启 USB 调试模式
2. 连接电脑
3. 在 HBuilderX 中点击"运行" → "运行到手机或模拟器"
4. 选择你的设备

**打包 APK/IPA：**
1. 在 HBuilderX 中点击"发行" → "原生App-云打包"
2. 配置应用信息（应用名称、包名等）
3. 配置证书（Android 可用公共测试证书，iOS 需要开发者证书）
4. 等待打包完成，安装到手机测试

## 配置说明

已配置的关键权限和模块：

### Android
- `POST_NOTIFICATIONS` - 发送通知权限（Android 13+ 必需）
- `Push` 模块 - 5+ 推送模块

### iOS
- `aps-environment` - 推送环境配置
- `UIBackgroundModes` - 后台通知模式

## API 说明

```typescript
// 发送本地通知（App 端）
plus.push.createMessage(content: string, payload: string, options: {
  title?: string,
  icon?: string,
  sound?: string
})
```

## 注意事项

1. **Android 13+** 需要动态申请通知权限，首次使用时系统会弹出权限请求
2. **iOS** 打包时需要有效的开发者证书才能使用推送功能
3. 本地通知不需要服务器，直接在设备上显示
4. 如果需要远程推送，需要配置 uni-push 或第三方推送服务

## 下一步扩展

如需添加远程推送功能：
1. 申请 DCloud 开发者账号
2. 在 manifest.json 中配置 uni-push
3. 后端集成推送服务
