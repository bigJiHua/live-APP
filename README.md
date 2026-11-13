# Gold 财管 · 原生 Android App

「Gold 财管」个人财务管理 App 的原生 Android 客户端，使用 Kotlin + Jetpack Compose 编写，
界面与交互对齐 web 端（Vue 3 + Vant），业务覆盖账户、银行卡、账单、流水、报表、预算、资产、
待办、日记与数据管理。

## 特性

- **纯原生**：Jetpack Compose + Navigation Compose，业务页面无 WebView 套壳。
- **1:1 对齐 web**：`vant-ui` 模块用 Compose 复刻项目实际使用的 45 类 Vant 组件与 7 类函数式 API，
  主题色、图标、字号与 web 端保持一致。
- **自带后端（BYO-backend）**：API 地址由用户在登录页「接入站点」输入并保存在本机，
  APK 内不内置任何默认/占位地址。
- **安全存储**：token 与会话使用 EncryptedSharedPreferences（Android Keystore）加密保存，
  日记缓存使用 EncryptedFile 加密落盘。
- **发布构建**：R8 混淆 + 资源压缩，支持可选证书锁定（默认关闭）。

## 功能模块

| 模块 | 内容 |
| --- | --- |
| 登录与账户 | 登录、站点接入、PIN 设置/校验、个人信息 |
| 账户 | 账户结构、余额流水、快速记账、批量登记、卡流水 |
| 银行卡 | 卡包/卡管理、账单与明细、分期、还款、额度、外币、信用中心 |
| 流水 | 日历、流水列表/详情、周期记账 |
| 报表 | 报表中心、月度趋势、分类占比、负债总览、资金日历/日报/收益/趋势、转账 |
| 预算 | 预算列表、详情、表单 |
| 资产 | 资产列表/编辑/趋势、固定资产与回收站 |
| 效率 | 待办日历、日记（图文编辑/分享）、工资日历与统计 |
| 设置与数据 | 分类管理、缓存清理、数据导入/导出/备份/校验、登录日志 |

## 工程结构

```
live-APP/
├── app/          # 业务 App（com.live.finance）：core / data / di / theme / ui
├── vant-ui/      # 自研 Compose 组件库（com.live.vant）：Vant 风格组件与主题
├── demo/         # 组件可视化演示 App
├── docs/         # 架构、构造书、组件清单等文档
├── tests/        # 工程校验脚本
├── build.gradle.kts
└── settings.gradle.kts
```

`app` 通过 `implementation(project(":vant-ui"))` 依赖组件库。

## 技术栈

| 项 | 版本 |
| --- | --- |
| 语言 | Kotlin 1.9.24（JVM 17） |
| UI | Jetpack Compose BOM 2024.06.00（编译器 1.5.14） |
| 构建 | AGP 8.5.2 / Gradle 8.7 / Kotlin DSL |
| Android | compileSdk 34 / minSdk 24 / targetSdk 34 |
| 导航 | Navigation Compose 2.7.7 |
| 网络 | Retrofit 2.11 + OkHttp 4.12 + Gson 2.11 |
| 存储 | EncryptedSharedPreferences + DataStore Preferences 1.1.1 + security-crypto |
| 图片 | Coil（经 `vant-ui` 传递引入） |

## 构建与运行

环境：JDK 17、Android SDK 34。用 Android Studio 打开仓库根目录，
或在 `local.properties` 配置 `sdk.dir` / 设置 `ANDROID_HOME`。

```bash
./gradlew :app:assembleDebug       # 调试包
./gradlew :app:assembleProdRelease # 正式变体（R8）
./gradlew :demo:assembleDebug      # 组件演示
```

两个产品变体 `fake` / `prod` 当前均连接真实后端；`fake` 仅包名追加 `.fake`、版本名追加 `-fake`。
首次启动需在登录页输入后端站点地址。

## 发布签名

签名文件不入库（`.gitignore` 已忽略 `*.jks` / `*.keystore`）。在 `gradle.properties`
或同名环境变量中配置：

```properties
RELEASE_KEYSTORE=release.jks
RELEASE_STORE_PASSWORD=
RELEASE_KEY_ALIAS=live
RELEASE_KEY_PASSWORD=
```

未配置时 release 产物为未签名，可安装调试但不可分发。

## 安全

- 仅允许 HTTPS，禁用明文流量（`usesCleartextTraffic=false` + networkSecurityConfig）。
- 可选证书锁定：`API_HOST` + `API_PINNED_SHA256`，默认关闭，避免证书轮换导致全网失效。
- Release 开启 R8 混淆与资源压缩。

## 文档

- [项目构造书](docs/项目构造书.md)：文件↔源文件对照、改造手册、P0–P3 坑清单
- [项目架构与接手指南](docs/PROJECT-ARCHITECTURE.md)
- [Vant 组件复刻清单](docs/vant-components-inventory.md)
- [web 页面参考](docs/web-audit-pages.md)
- [组件库说明](vant-ui/README.md)

## 版本

versionName `2.1.0` / versionCode `20100`。
