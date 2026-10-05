# Compose ↔ SwiftUI

把 Jetpack Compose 和 SwiftUI 按语义互译的 Agent 技能。它翻译视图树、状态归属和副作用。

同一串修饰符，两边的内外层相反：SwiftUI 里靠近视图的修饰符在内层，Compose 里写在前面的修饰符在外层。技能会把整条链倒过来，并写明对齐、Binding、列表和副作用。

技能是通用的 `SKILL.md`（frontmatter + Markdown），可用于 Grok、Claude Code，以及其他认这个格式的 Agent。

## 技能

| 技能 | 路径 |
|---|---|
| `compose-swiftui` | [`skills/compose-swiftui/SKILL.md`](./skills/compose-swiftui/SKILL.md) |

## 安装

把技能目录复制到 Agent 的 skills 目录：

```bash
# Grok — 仓库内（推荐给团队）
mkdir -p .grok/skills && cp -r skills/compose-swiftui .grok/skills/

# Grok — 当前用户，所有项目可用
cp -r skills/compose-swiftui ~/.grok/skills/

# Claude Code — 仓库内
mkdir -p .claude/skills && cp -r skills/compose-swiftui .claude/skills/

# Claude Code — 当前用户
cp -r skills/compose-swiftui ~/.claude/skills/
```

## 用法

装好之后可以直接调用，或用自然语言：

```text
/compose-swiftui
```

> 把这段 Compose 翻译成 SwiftUI
>
> 把这段 SwiftUI 翻译成 Compose

贴出哪一段，就只译哪一段。需要原文和译文并排时，说明要对照。

## 翻译时守住的几点

1. 先列出状态和事件，再写容器树，然后把修饰符整链倒序。
2. Compose 的 `Column` / `Row` / `Box` 默认靠起始边，SwiftUI 的 `VStack` / `HStack` / `ZStack` 默认居中。两边都写出对齐。
3. SwiftUI 的 `$binding` 拆成 Compose 的当前值和写入回调；`(value, onValueChange)` 收成 `Binding(get:set:)`。
4. `LaunchedEffect(key)` 对应 `.task(id:)`。源码是 `.onChange(of:)` 时，译文在首次出现时不执行。
5. `dp` 和 `pt` 使用同一个数字。
6. 顶栏、列表、弹层和警告框换成目标平台自己的容器。外观要接近 `LazyColumn` 时用 `ScrollView { LazyVStack }`；要用系统列表时用 `List`（自带分隔线、行高和内边距）。

完整对照表在 [`skills/compose-swiftui/SKILL.md`](./skills/compose-swiftui/SKILL.md)。

两端共用一套界面时，技能会先请你在 Compose Multiplatform（iOS 上用 Skia 绘制 Compose）和 Skip（SwiftUI 源码译成 Compose，iOS 仍是 SwiftUI）之间选定。

## English

An agent skill that translates Jetpack Compose and SwiftUI by meaning: the view tree, who owns the state, and the effects.

Modifier order runs in opposite directions. In SwiftUI the modifier closest to the view is the inner layer. In Compose the first modifier is the outer layer. The skill reverses the whole chain, then writes alignment, bindings, lists, and effects so the result matches.

The skill is a `SKILL.md` file (frontmatter + Markdown). It works with Grok, Claude Code, and other agents that load this format.

### Skill

| Skill | Path |
|---|---|
| `compose-swiftui` | [`skills/compose-swiftui/SKILL.md`](./skills/compose-swiftui/SKILL.md) |

### Install

Copy the skill folder into the agent's skills directory:

```bash
# Grok — project-scoped (recommended for teams)
mkdir -p .grok/skills && cp -r skills/compose-swiftui .grok/skills/

# Grok — user-scoped (all projects)
cp -r skills/compose-swiftui ~/.grok/skills/

# Claude Code — project-scoped
mkdir -p .claude/skills && cp -r skills/compose-swiftui .claude/skills/

# Claude Code — user-scoped
cp -r skills/compose-swiftui ~/.claude/skills/
```

### Usage

```text
/compose-swiftui
```

> Translate this Compose into SwiftUI
>
> Translate this SwiftUI into Compose

Only the pasted region is translated. Ask for a side-by-side when you want the source and the translation together.

### Rules the skill keeps

1. List state and events, write the container tree, then reverse the modifier chain.
2. Compose `Column` / `Row` / `Box` default to the start edge. SwiftUI `VStack` / `HStack` / `ZStack` default to center. Alignment is written on both sides.
3. A SwiftUI `$binding` becomes Compose's value plus change callback. `(value, onValueChange)` becomes `Binding(get:set:)`.
4. `LaunchedEffect(key)` matches `.task(id:)`. When the source is `.onChange(of:)`, the translation does not run on first appearance.
5. `dp` and `pt` use the same number.
6. Top bars, lists, sheets, and alerts use the target platform's own containers. Use `ScrollView { LazyVStack }` when the look should stay close to `LazyColumn`. Use `List` for a system list (it brings separators, row height, and insets).

The full tables are in [`skills/compose-swiftui/SKILL.md`](./skills/compose-swiftui/SKILL.md).

When one shared UI should run on both platforms, the skill asks you to choose Compose Multiplatform (Compose drawn with Skia on iOS) or Skip (SwiftUI transpiled to Compose; iOS stays SwiftUI).

## License

[MIT](./LICENSE)
