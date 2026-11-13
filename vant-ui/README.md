# vant-ui — 项目内部自研 Compose 组件库

`vant-ui` 是本项目（安卓原生 APP）的**内部 UI 组件库**，使用 Jetpack Compose 独立编写，
提供与项目 web 端（Vue3 + Vant 4.9.22）一致的外观与交互行为，供 `:app` 业务层直接调用。

## 声明与第三方素材（避嫌说明）

- 本模块为**独立实现**：全部 Compose 代码为本项目原创编写，**不包含、未移植 Vant 的任何
  JS / CSS / Vue 源码**，仅参照其公开文档实现视觉与交互效果，用于和本项目 web 端保持一致。
- 本模块与有赞（YouZan）/ Vant 官方**无任何隶属或合作关系**，非官方库，**不对外发布、不分发**。
- 组件命名（`Van*` 前缀）仅用于项目内部与 web 端组件名的对照映射，便于维护，别无他意。
- 唯一使用的第三方素材：`src/main/res/font/vant_icon.ttf` 图标字体，提取自
  [Vant 4.9.22](https://github.com/youzan/vant)（MIT License，Copyright (c) YouZan），
  依 MIT 协议条款使用并在此致谢与声明。
- 若计划将本仓库公开或商用，请自行复核第三方素材的许可条款。

## 组件清单（45 类组件 + 7 类函数式 API）

| 分组 | 组件 |
| --- | --- |
| theme | VantColors / VantDimens / VantTheme（项目 CSS 变量 1:1 映射） |
| icon | VanIcon + VantIconCodes（259 个图标码点，字体渲染） |
| basic | Button Cell CellGroup Loading Overlay Divider Tag Progress Circle NoticeBar Skeleton Image Empty |
| form | Field Stepper Switch Radio Checkbox Picker DatePicker NumberKeyboard PasswordInput Uploader Search |
| feedback | Popup Toast Dialog ActionSheet ImagePreview PullRefresh List |
| nav | NavBar Tabs Tabbar DropdownMenu Collapse |
| layout | Row / Col（24 栅格） |
| other | Swipe Calendar |

组件盘点与各组件在业务侧的使用统计见 [docs/vant-components-inventory.md](../docs/vant-components-inventory.md)。

## 构建与验证

版本矩阵：AGP 8.5.2 + Kotlin 1.9.24 + Compose BOM 2024.06.00（compileSdk 34 / minSdk 24），
导入 Android Studio 打开 `APP/` 即可随工程构建；`:demo` 模块提供全部组件的可视化演示页。

## 主题

`web/src/assets/css/style.css` 用 `--van-*` 覆盖了 Vant 出厂主题，本库的默认主题 = 覆盖后的结果：
primary **#3a66e0**、文字 #323233/#646566/#969799、页面底 #f7f8fa、卡片 #fff、边框 #ebedf0 等。

```kotlin
// App 根部包一层即可
VantTheme {                       // 浅色（项目主题）
    VantTheme(VantDarkColors.dark()) { ... }   // 深色（对应 data-theme-mode=dark）
    // 需要 Vant 出厂蓝时：VantTheme(VantColors.classic())
}
```

## 图标

`van-icon` 的全部使用都来自同一套字体。`vant_icon.ttf` 直接提取自 `vant/lib/index.css`
内嵌的 `@font-face vant-icon`（woff2 → fontTools 转 TTF），259 个名称→码点映射自动生成于
`VantIconCodes.kt`，因此任意图标名（arrow、cross、location-o、credit-pay…）渲染结果与 web 一致。

再生成映射表（修改 vant 版本后执行，需 `pip install fonttools brotli`）：

```python
import re, base64, io, json
css = open('web/node_modules/vant/lib/index.css', encoding='utf-8').read()

# 1) 字体：内嵌 woff2 → TTF
m = re.search(r'font-family:vant-icon;[^}]*base64,([A-Za-z0-9+/=]+)', css)
from fontTools.ttLib import TTFont
font = TTFont(io.BytesIO(base64.b64decode(m.group(1)))); font.flavor = None
font.save('APP/vant-ui/src/main/res/font/vant_icon.ttf')

# 2) 码点表 → temp/vant_icon_map.json
mapping = {}
for mm in re.finditer(r'\.van-icon-([a-z0-9-]+):before\{content:"', css):
    s = mm.end(); e = css.index('"', s)
    mapping[mm.group(1)] = int(css[s:e].lstrip(chr(92)), 16)
json.dump(mapping, open('temp/vant_icon_map.json', 'w'), separators=(',', ':'))

# 3) JSON → Kotlin（把映射值以真实 PUA 字符嵌入 '…' 字面量，生成
#    APP/vant-ui/src/main/java/com/live/vant/icon/VantIconCodes.kt，见 VantIconCodes 文件头注释）
```

## Vant → Compose API 映射（迁移时按此表替换）

| Vant | Compose | 关键 props/事件 |
|------|---------|----------------|
| `<van-button type size block plain round loading disabled @click>` | `VanButton(text, onClick, type=, size=, block=, plain=, round=, loading=, disabled=)` | 枚举 `VanButtonType/Size` |
| `<van-cell title value label icon is-link center @click>` | `VanCell(title=, value=, label=, icon=, isLink=, center=, onClick=, …Slot)` | 插槽 → 尾随/具名 composable |
| `<van-cell-group title inset>` | `VanCellGroup(title=, inset=) { … }` | |
| `<van-icon name size color @click :badge>` | `VanIcon(name, size=, color=, badge=, dot=, onClick=)` | 图像型 name（URL）自动切 Coil |
| `<van-field v-model label placeholder type readonly clearable>` | `VanField(value, onValueChange, label=, placeholder=, type=, readonly=, clearable=)` | |
| `<van-picker columns title @confirm @cancel>` | `VanPicker(columns=, onConfirm=, onCancel=, title=)` | |
| `<van-date-picker v-model min-date max-date type>` | `VanDatePicker(type=, value=, minDate=, maxDate=, onConfirm=)` | LocalDate |
| `<van-switch v-model size active-color>` | `VanSwitch(checked, onCheckedChange, size=, activeColor=)` | |
| `<van-stepper v-model min max>` | `VanStepper(value, onValueChange, min=, max=)` | |
| `<van-radio-group v-model direction>` / `van-radio` | `VanRadioGroup(value, onValueChange) { VanRadio(name=, label=) }` | |
| `<van-checkbox-group v-model>` / `van-checkbox` | `VanCheckboxGroup(value, onValueChange) { VanCheckbox(name=) }` | |
| `<van-number-keyboard v-model show theme extra-key close-button-text @input @delete @close>` | `VanNumberKeyboard(show, text, onTextChange, theme=, extraKey=, onClose=)` | |
| `<van-password-input value length focused gutter>` | `VanPasswordInput(value, length=, focused=, gutter=)` | |
| `<van-uploader v-model max-count multiple @delete after-read>` | `VanUploader(fileList, onPick=, maxCount=, onDelete=)` | after-read 上传 = onPick 回调接业务 |
| `<van-search v-model placeholder show-action @search @cancel @clear>` | `VanSearch(value, onValueChange, placeholder=, showAction=, onSearch=, onCancel=, onClear=)` | |
| `<van-popup v-model:show position round>` | `VanPopup(show, onDismissRequest, position=, round=)` | |
| `<van-overlay show z-index @click>` | `VanOverlay(show, onClick=)` | |
| `<van-loading size type color vertical>` | `VanLoading(size=, type=, color=, vertical=)` | |
| `<van-toast>` 系列 | `LocalVanToastController.current.show/success/fail/loading/close` | 根部挂 `VanToastHost` |
| `showConfirmDialog({title,message})` | `VanConfirmDialog(show, title=, message=, onConfirm=, onCancel=)` | |
| `showDialog` | `VanAlertDialog` / `VanDialog` | |
| `<van-action-sheet actions title cancel-text @select>` | `VanActionSheet(show, onDismissRequest, actions=, onSelect=)` | |
| `<van-image-preview images closeable @change>` | `VanImagePreview(show, images=, closeable=, onChange=, onClose=)` | |
| `<van-pull-refresh v-model @refresh success-text>` | `VanPullRefresh(state, onRefresh, successText=)` + `state.finishRefresh()` | |
| `<van-list v-model:loading finished finished-text @load>` | `VanList(state=, loading=, finished=, onLoad=)` + `VanListFooter(...)` | |
| `<van-nav-bar title left-arrow @click-left @click-right>` | `VanNavBar(title=, leftArrow=, onClickLeft=, onClickRight=)` | |
| `<van-tabs v-model:active><van-tab title name>` | `VanTabs(active, onActiveChange, tabs=) { name -> }` | |
| `<van-swipe :autoplay :loop vertical show-indicators>` | `VanSwipe(count=, autoplay=, loop=, vertical=)` | |
| `<van-collapse v-model><van-collapse-item name title icon>` | `VanCollapse(activeNames, onActiveNamesChange) { VanCollapseItem(name, title) }` | |
| `<van-notice-bar text left-icon mode scrollable>` | `VanNoticeBar(text, leftIcon=, mode=)` | |
| `<van-calendar v-model:show type min-date max-date @confirm>` | `VanCalendar(show, onClose, type=, minDate=, maxDate=, onConfirm=)` | |
| `<van-skeleton row title loading>` | `VanSkeleton(loading, row=, title=) { 内容 }` | |
| `<van-empty description image>` | `VanEmpty(description=, image=)` | 内置插画含 default/error/search/network |
| `<van-tag type plain round closeable>` | `VanTag(text, type=, plain=, round=, closeable=)` | |
| `<van-progress percentage stroke-width color show-pivot>` | `VanProgress(percentage, strokeWidth=, color=)` | |
| `<van-circle rate text>` | `VanCircle(rate, text=, size=)` | |
| `<van-divider content-position dashed>` | `VanDivider(content=, dashed=, contentPosition=)` | |
| `<van-row gutter><van-col span>` | `VanRow(gutter=) { VanCol(span=) }` | |

## 已知实现差异（如实声明）

1. **点击态**：Vant 用 CSS `:active` 变色，本组件用等价的按压叠色（无 Material 水波纹）——视觉 1:1；
   如团队要求水波纹可另加 `ripple()`。
2. **Popup / ActionSheet / Calendar / NumberKeyboard / ImagePreview** 走独立 Window（Dialog）实现，
   天然盖全屏；`z-index` 语义由层级顺序表达。
3. **Toast** 无 Vue 的全局命令式调用，采用 `VanToastController` 状态宿主（见上表），
   行为（duration/forbidClick/loading 不自动关）与 Vant 一致。
4. **DropdownMenu** 的全屏灰遮罩简化为内联展开 + 点选收起；菜单栏/选项样式 1:1。
5. **Lazyload**：Coil 按可见性加载，语义覆盖 `lazy-load`；网络权限由调用方声明。
6. **van-uploader** 的 `before-read/after-read`（压缩+上传链路）属于业务逻辑：
   `onPick(List<Uri>)` 回调交业务处理（对应 web 里 browser-image-compression + 上传后写回 fileList）。
7. 空状态 `albums-o / computer-o / palette` 三个图标名在 Vant 4.9.22 字体中本就不存在，
   两端同样渲染为空，非复刻缺失。
