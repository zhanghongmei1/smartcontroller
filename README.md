# App 提醒更新 — 版本配置文件

两个 App 的「提醒更新」功能读取这里的 JSON 来判断是否有新版本。
客户端逻辑在各自项目的 `utils/updateChecker.js`（启动 8 秒后静默检查，按当前平台取
`android` / `ios` 段比较版本号，弹窗确认后跳对应应用商店：安卓 Google Play，iOS App Store）。

## JSON 结构

顶层分 `android` / `ios` 两段，**两个平台的版本号各自独立**，哪边发版就改哪边。
每段字段相同：

| 字段 | 说明 |
|---|---|
| `versionName` | 展示用的版本号，如 `1.5.1` |
| `versionCode` | 比较用的整数版本号，**大于**用户手机上已装的 versionCode 时弹窗 |
| `notes` | 更新说明，显示在弹窗第二行（建议英文，用户主要在海外） |
| `forceUpdate` | `true` = 强制更新，弹窗没有"稍后"按钮，不能跳过 |
| `downloadUrl` | 商店协议链接（见下），点"立即更新"后直接跳商店 |
| `downloadUrlWeb` | 网页版商店链接。设备没有商店应用时（如无 GMS 的海外华为机）自动退回这个 |

## 商店链接格式

- **Google Play**（安卓）：
  `downloadUrl` → `market://details?id=安卓包名`（App 内直接唤起 Play 商店）
  `downloadUrlWeb` → `https://play.google.com/store/apps/details?id=安卓包名`
- **App Store**（iOS）：
  `downloadUrl` → `itms-apps://itunes.apple.com/app/id你的AppleID`（数字 ID）
  `downloadUrlWeb` → `https://apps.apple.com/app/id你的AppleID`
  Apple ID 在 App Store Connect 的应用信息里（商店网页网址 `apps.apple.com/app/idXXXXXXXXXX` 里就是）。

### 占位符待填项

| 文件 | 占位符 | 去哪查 |
|---|---|---|
| `smart-controller-pro.json` | 无（安卓包名 `com.controller.myapp` 已填好） | — |
| `smart-controller-pro.json` | `TODO_APPSTORE_ID`（ios 段） | App Store Connect，上架后填 |
| `brake-ctrl.json` | `TODO_ANDROID_PACKAGE`（android 段） | DCloud 开发者后台该应用的包名 |
| `brake-ctrl.json` | `TODO_APPSTORE_ID`（ios 段） | App Store Connect，上架后填 |

没上架的平台保持占位符即可：只要那一段的 `versionCode` 不大于线上版本，就永远不会弹窗
（模板里已按当前线上版本号设置，iOS 没上架也弹不出来）。

## 一次性准备

1. **建 GitHub 仓库**：注册/登录 GitHub，新建一个 **公开(Public)** 仓库，名字如 `app-update`，
   把本目录两个 JSON 传上去（文件名保持不变）。手机 App 拉取的 raw 地址格式：
   `https://raw.githubusercontent.com/<你的用户名>/app-update/main/<文件名>`
2. **改客户端地址**：把 `tuoche/utils/updateChecker.js` 和 `tuoche/newUI3/utils/updateChecker.js`
   里的 `YOUR_GITHUB_USERNAME` 替换成你的 GitHub 用户名（两个文件都要改）。
3. **填占位符**：按上面表格把包名 / Apple ID 填进 JSON（可以先传 GitHub 再网页上改）。

## 每次发版流程（按平台独立）

安卓发版：
1. HBuilderX/离线打包改 `versionName` / `versionCode`（**必须 +1**），提交 Google Play
2. 改 JSON 的 `android` 段：`versionCode` 和商店里新版本一致，`notes` 写更新内容
3. GitHub 网页上提交，App 端立即生效（客户端请求带时间戳，绕过 CDN 缓存）

iOS 发版：同理改 `ios` 段（前提是已上架 App Store）。

## 验证方法

把 JSON 里当前平台的 `versionCode` 随手改大（比如 999），重新打开 App，约 8 秒后应弹更新提示，
点"立即更新"应跳对应商店；改回真实版本号后不再弹。检查失败（无网/文件缺失）会静默跳过，
只在 HBuilderX 控制台打 log。
