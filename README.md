# tianshu-plugin-wit

[中文](README.zh.md)

This is the shape of a plugin: WIT, JSON Schema, and the `host.read` / `host.apply` / Play extension notes. There is no wasm here, so this repo is not an installable plugin. To write a plugin, clone the TypeScript or Rust template; those repos vendor this contract and pack without this repo.

Language-template maintainers copy `plugin.wit` and `schema/` from here.

A plugin is three files:

| File | Role |
|---|---|
| `plugin.json` | Identity: id, title, version, description, icon and README filenames |
| `ui.json` | Canvas nodes, fields, and optional toolbox buttons |
| `plugin.wasm` | Logic at Play time; exports are in `plugin.wit` |

The workbench keys the plugin by `plugin.json` `id` and nodes by `ui.json` `nodes[].kind`. The kind on the graph is `p:<id>:<kind>`.

On disk a community plugin is one directory named after `id`:

```
plugins/community/<id>/
  plugin.json
  ui.json
  plugin.wasm
  README.md    # if plugin.json names a readme
  icon.svg     # if it names an icon
```

`id` must match wasm `id()`. `ui.json` must match `ui-json()`. `abi` and `abi-version()` are `3`. Use Settings → Add local plugin on the packed folder. The workbench copies the files, then loads that copy. Extra MCP tools from `plugin.json` `mcpTools` are listed only when the plugin id is official (`tianshu.*`).

## plugin.json

See `schema/plugin.schema.json`.

| Field | Required | Meaning |
|---|---|---|
| `id` | yes | Plugin id. Starts with a letter; then letters, digits, `.` `_` `-` |
| `title` | yes | Name in settings and lists |
| `version` | yes | Your version string |
| `abi` | yes | `3` |
| `description` | no | One-line description |
| `readme` | no | Filename in the same directory, e.g. `README.md` |
| `icon` | no | Filename in the same directory, e.g. `icon.svg` |

```json
{
  "id": "example.community.template",
  "title": "Template",
  "description": "Echo inbound pipe or the text param.",
  "readme": "README.md",
  "version": "0.1.0",
  "abi": 3
}
```

## ui.json

See `schema/ui.schema.json`. The root object has `nodes` and optional `tools`.

Each node:

| Field | Required | Meaning |
|---|---|---|
| `kind` | yes | Node kind. Letters, digits, `.` `_` `-`. First argument to `step` |
| `title` | yes | Title on the canvas |
| `category` | yes | Left-library group |
| `categoryLabel` | no | Label for that group; defaults to `category` |
| `fields` | no | Parameters on the node |
| `output` | no | Play data output. `data` means downstream can pull this cell. Default `none` |
| `write` | no | Play attribute writes. `attr` means the host applies `set` from the step return. Default `none` |

Each field:

| Field | Required | Meaning |
|---|---|---|
| `id` | yes | Param name. Key in `params-json`, or `host.read("param", id, "")` |
| `label` | yes | UI label |
| `type` | yes | The template uses `"string"` |
| `default` | no | Default value |

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

Each tool is a left-side toolbox button with its own SVG:

| Field | Required | Meaning |
|---|---|---|
| `id` | yes | Button id |
| `title` | yes | Tooltip |
| `icon` | yes | `.svg` in the same directory |
| `op` | yes | Graph-edit verb (for example `add_node`, `add_graph`, `connect_pipe`). Full table: [host.apply](docs/host-apply.md) |
| `args` | no | Arguments. `$selected` / `$selected2` become the current selection |

If those icons would collide with toolbox text, the text hides. At most 16 tools. `icon` must be an `.svg`.

## wasm exports

`guest` in `plugin.wit`:

| Export | Meaning |
|---|---|
| `abi-version` | Return `3` |
| `id` | Same as `plugin.json` `id` |
| `ui-json` | Full `ui.json` text |
| `step(kind, params-json)` | One pipe step. Return value: [What an extension node is in Play](docs/play-node.md) |
| `on-tool(id)` | Toolbar click. May call `host.apply` |

`host.read(kind, a, b)` asks the host for data during a step. Empty string means nothing. The host usually returns IR JSON such as `{"type":"String","value":"ping"}`. Argument tables: [host.read](docs/host-read.md) ([中文](docs/host-read.zh.md)).

`host.apply(op, args-json)` edits the graph from `on-tool`. Table: [host.apply](docs/host-apply.md) ([中文](docs/host-apply.zh.md)).
