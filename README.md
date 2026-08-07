# 数据备份插件

该插件把备份能力集成到 N.E.K.O 的原生 SDK v2 插件运行时中，不启动额外的 Flask 服务。它既可作为独立 `.neko-plugin` 安装，也与 N.E.K.O 主仓的内置版本保持同源。

- `core`：`config`、`character_cards`、`memory`
- `assets`：`card_faces`、`live2d`、`vrm`、`mmd`、`pngtuber`、`workshop`
- 快照保存在当前 N.E.K.O 数据根目录的 `plugins/data_backup/data/snapshots` 下。
- 恢复前会自动创建一份安全快照；恢复后需要重启 N.E.K.O。
- 符号链接不会进入快照，所有操作仅接受固定组名和插件生成的快照 ID。

设计参考 [MemoryCat](https://github.com/JohnChiao75/MemoryCat) 的快照与分组备份思路。MemoryCat 以 Apache License 2.0 发布；本实现针对 N.E.K.O SDK v2 重新组织，并保留本说明作为来源致谢。

## 安装

从 GitHub Releases 下载 `data_backup.neko-plugin`，然后在 N.E.K.O 插件管理页面选择导入插件包。插件启用后，从插件列表的“打开备份管理”进入管理页面。

恢复会替换当前备份组中的现有数据。插件会先自动创建安全快照，但恢复完成后仍应立即重启 N.E.K.O。

## 开发与验证

本仓库使用 Python 3.11。在仓库根目录运行：

```bash
uv run --group dev pytest -q
uv run --group dev ruff check .
```

在 N.E.K.O 主仓中挂载为 `plugin/plugins/data_backup` 后，可使用官方插件工具生成安装包：

```bash
uv run python -m plugin.neko_plugin_cli.cli check -r data_backup
```

推送与 `plugin.toml` 版本一致的标签（例如 `v0.1.0`）后，发布工作流会生成 `data_backup.neko-plugin` Release 附件。
