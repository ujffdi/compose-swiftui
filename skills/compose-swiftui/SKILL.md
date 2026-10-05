---
name: compose-swiftui
description: >
  Translate Jetpack Compose into SwiftUI or SwiftUI into Jetpack Compose
  by reversing the modifier chain and mapping layout, state, effects, and
  platform containers. Use when the user says 互相转换, 翻译成 SwiftUI,
  翻译成 Compose, Compose 转 SwiftUI, SwiftUI 转 Compose, port or convert
  between Compose and SwiftUI, or runs /compose-swiftui.
---

# Compose ↔ SwiftUI

翻译视图树、状态归属和副作用。按下面的顺序做，做完用「完成标准」自检。

## 范围

用户贴出哪一段，就译哪一段，并只输出目标端代码。用户要求对照时，再并排给出原文和译文。

目标端是 SwiftUI，且用户没有给出文件路径时，给出可编译片段，不创建文件。

目标端要写入 Compose 源文件时，先读目标路径向上最近的 `AGENTS.md` 并遵守其中的界面约定。用户只要片段时，直接给出片段。

用户要的是两端共用一套界面时，先让用户选定一条路再继续：Compose Multiplatform（iOS 上用 Skia 绘制 Compose），或 Skip（SwiftUI 源码译成 Compose，iOS 仍是 SwiftUI）。

## 顺序

1. 列出状态、持有者和事件回调。
2. 写出容器树，对齐全部写明。
3. 修饰符整链倒序，然后核对点击热区、`clip` 和 `shadow`。
4. 按进入、key 变化、离开对齐副作用。
5. 顶栏、列表、弹层、警告框换成目标平台自己的容器。

## 修饰符

同一书写顺序的视觉结果相反。翻译时把整条链倒过来。

SwiftUI 里，靠近视图的修饰符在内层：

```swift
Text("Hi")
    .padding(8)
    .background(.red)
    .padding(16)
```

Compose 里，靠前的修饰符在外层：

```kotlin
Text(
    "Hi",
    modifier = Modifier
        .padding(16.dp)
        .background(Color.Red)
        .padding(8.dp),
)
```

倒序之后核对三件事：点击区域是该修饰符作用之后的尺寸；`clip` 会裁掉它之后绘制的阴影；阴影落在裁切的哪一侧与原文一致。

Compose 自定义组件把调用方传入的 `modifier` 加在根节点链的最前面。

## 布局

Compose 的 `Column`、`Row`、`Box` 默认靠起始边。SwiftUI 的 `VStack`、`HStack`、`ZStack` 默认居中。两边都写出对齐。

| 意图 | Compose | SwiftUI |
|---|---|---|
| 纵向 | `Column` | `VStack` |
| 横向 | `Row` | `HStack` |
| 重叠 | `Box` | `ZStack` |
| 剩余空间 | `Modifier.weight(1f)`（仅 `Row` / `Column`） | `Spacer()`，或 `frame(maxWidth: .infinity)` |
| 子项对齐 | `Modifier.align(...)` | `frame(maxWidth: .infinity, maxHeight: .infinity, alignment: ...)` |
| 间距 | `Arrangement.spacedBy(8.dp)` | `VStack(spacing: 8)` / `HStack(spacing: 8)` |
| 固定尺寸 | `.size(48.dp)` | `.frame(width: 48, height: 48)` |
| 撑满宽度 | `.fillMaxWidth()` | `.frame(maxWidth: .infinity)` |
| 滚动 | `verticalScroll` / `horizontalScroll` | `ScrollView` |

`dp` 与 `pt` 使用同一个数字。

## 状态

把 SwiftUI 的 `$binding` 拆成 Compose 的当前值和写入回调。把 Compose 的 `(value, onValueChange)` 收成 `Binding(get:set:)`。

| SwiftUI | Compose |
|---|---|
| `@State` | `remember { mutableStateOf(...) }` |
| `@Binding` | `(T, (T) -> Unit)` |
| `@StateObject`，或由页面持有的 `@Observable` | `viewModel()` / `hiltViewModel()` |
| 计算属性 | 组合期直接计算，或 `derivedStateOf` |
| `@Environment` | `CompositionLocal` |
| `Text("key")`（查本地化表） | `stringResource(R.string.key)` |
| `Text(verbatim:)` | `Text("...")` |

## 副作用

`LaunchedEffect(key)` 对应 `.task(id:)`：进入时执行，key 变化时取消再执行。

源码是 `.onChange(of:)` 时，译文在首次出现时不执行。

离开界面的清理用 `DisposableEffect` 的 `onDispose`，对应 `.onDisappear`。

## 容器与绘制

| Compose | SwiftUI |
|---|---|
| `LazyColumn` | 外观接近用 `ScrollView { LazyVStack }`；系统列表用 `List` |
| `items(..., key = { it.id })` | `ForEach(..., id: \.id)` |
| `Scaffold` + `topBar` | `NavigationStack` + `.toolbar` |
| `ModalBottomSheet` | `.sheet` + `.presentationDetents` |
| `AlertDialog` | `.alert` |
| `Snackbar` | 自定义 overlay |
| `AnimatedVisibility` | `if` + `.transition` |
| `animate*AsState` | `.animation(.easeInOut, value:)` 或 `withAnimation` |
| `ContentScale.Fit` | `.resizable().scaledToFit()` |
| `ContentScale.Crop` | `.resizable().scaledToFill().clipped()` |
| `ContentScale.FillBounds` | `.resizable()` 后拉伸到 `frame` |
| `clickable` | 控件用 `Button`；自定义热区用 `.contentShape(Rectangle()).onTapGesture` |
| `AndroidView` | `UIViewRepresentable` |

`List` 自带分隔线、行高和内边距。会增删改的列表两边都用稳定 id。

## 完成标准

- 对齐已经写明。
- 倒序之后，背景和 padding 的内外关系与原文一致。
- 列表 id 稳定。
- 首次进入时副作用是否执行，与原文一致。
- 改动范围就是用户给出的那一段。
