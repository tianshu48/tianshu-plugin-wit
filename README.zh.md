# tianshu-plugin-wit

[English](README.md)

插件对外的形状写在这里：WIT、JSON Schema、`host.read` / `host.apply` / Play 扩展槽说明。本仓没有 wasm，不能当插件安装。写插件请单独克隆 TypeScript 或 Rust 模板；那些仓已经钉了一份本合同，打包不依赖本仓。

语言模板维护者从这里拷 `plugin.wit` 和 `schema/`。

一个插件就是三个文件一起工作：

| 文件 | 干什么 |
|---|---|
| `plugin.json` | 插件自己是谁：id、标题、版本、说明、图标和 README 文件名 |
| `ui.json` | 往图上放哪些节点、工具栏按钮 |
| `plugin.wasm` | Play 时跑的逻辑，导出见 `plugin.wit` |

工作台按 `plugin.json` 的 `id` 认插件，按 `ui.json` 的 `nodes[].kind` 认节点。图上的 kind 是 `p:<id>:<kind>`。

磁盘上社区插件是一个以 `id` 为名的目录：

```
plugins/community/<id>/
  plugin.json
  ui.json
  plugin.wasm
  README.md    # 若 plugin.json 写了 readme
  icon.svg     # 若写了 icon
```

`id` 必须和 wasm 里 `id()` 相同。`ui.json` 必须和 `ui-json()` 返回的 JSON 相同。`abi` 和 `abi-version()` 现在都是 `3`。在「设置 → 添加本地插件」里选打包好的目录，工作台会拷一份再加载。`plugin.json` 的 `mcpTools` 只有官方 id（`tianshu.*`）会登记到 MCP。

## plugin.json

对照 `schema/plugin.schema.json`。

| 字段 | 必填 | 含义 |
|---|---|---|
| `id` | 是 | 插件 id。字母开头，后面可以是字母数字和 `.` `_` `-` |
| `title` | 是 | 设置页、列表里显示的名字 |
| `version` | 是 | 你自己的版本号，字符串 |
| `abi` | 是 | 填 `3` |
| `description` | 否 | 一句话说明 |
| `readme` | 否 | 同目录下说明文件名，如 `README.md` |
| `icon` | 否 | 同目录下图标文件名，如 `icon.svg` |

```json
{
  "id": "example.community.template",
  "title": "Template",
  "description": "把入边或参数回声到管道。",
  "readme": "README.md",
  "version": "0.1.0",
  "abi": 3
}
```

## ui.json

对照 `schema/ui.schema.json`。根上是 `nodes`，可加 `tools`。

每个节点：

| 字段 | 必填 | 含义 |
|---|---|---|
| `kind` | 是 | 节点种类。字母数字和 `.` `_` `-`。`step` 收到的第一个参数就是它 |
| `title` | 是 | 画布上的标题 |
| `category` | 是 | 出现在左侧库的哪一组 |
| `categoryLabel` | 否 | 这组在库里显示的名字；不写就用 `category` |
| `fields` | 否 | 节点上可填的参数 |
| `output` | 否 | Play 数据输出。`data` 表示下游可以拉这一格；默认 `none` |
| `write` | 否 | Play 写属性。`attr` 表示步进时宿主按返回值改主体属性；默认 `none` |

每个 field：

| 字段 | 必填 | 含义 |
|---|---|---|
| `id` | 是 | 参数名。`step` 的 `params-json` 里用这个键，也可以 `host.read("param", id, "")` |
| `label` | 是 | 界面上的标签 |
| `type` | 是 | 现在模板里用 `"string"` |
| `default` | 否 | 默认值 |

```json
{
  "nodes": [
    {
      "kind": "echo",
      "title": "Echo",
      "category": "echo",
      "categoryLabel": "Echo",
      "fields": [
        { "id": "text", "label": "Text", "type": "string", "default": "" }
      ]
    }
  ]
}
```

每个 tool 是工具栏左侧一颗自备 SVG 的按钮：

| 字段 | 必填 | 含义 |
|---|---|---|
| `id` | 是 | 按钮 id |
| `title` | 是 | 提示文字 |
| `icon` | 是 | 同目录 `.svg` |
| `op` | 是 | 图编辑动词（例如 `add_node`、`add_graph`、`connect_pipe`）。全表见 [host.apply](docs/host-apply.zh.md) |
| `args` | 否 | 参数。`$selected` / `$selected2` 会换成当前选中 |

图标把工具栏文字挤着了，文字会藏起来。`tools` 最多 16 项，`icon` 必须是 `.svg`。

## wasm 要导出什么

`plugin.wit` 里的 `guest`：

| 导出 | 含义 |
|---|---|
| `abi-version` | 返回 `3` |
| `id` | 返回和 `plugin.json` 一样的 id |
| `ui-json` | 返回 `ui.json` 的全文 |
| `step(kind, params-json)` | 这个节点在管道上走一步。返回值见 [扩展槽在 Play 里是什么](docs/play-node.zh.md) |
| `on-tool(id)` | 工具栏点击。可以调 `host.apply` |

`host.read(kind, a, b)` 是步进时向宿主要数据。空字符串表示没有。返回多半是 IR JSON，例如 `{"type":"String","value":"ping"}`。参数表见 [host.read](docs/host-read.zh.md)（[English](docs/host-read.md)）。

`host.apply(op, args-json)` 只在 `on-tool` 里改图。动词表：[host.apply](docs/host-apply.zh.md)（[English](docs/host-apply.md)）。
