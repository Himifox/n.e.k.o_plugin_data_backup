# 猫娘备份插件

该插件把备份能力集成到 N.E.K.O 的原生 SDK v2 插件运行时中，不启动额外的 Flask 服务。它既可作为独立 `.neko-plugin` 安装，也与 N.E.K.O 主仓的内置版本保持同源。

## 备份范围

`core` 核心数据组：

| 目录 | 内容 |
| --- | --- |
| `config` | 保存在 N.E.K.O 数据目录内的程序设置、角色配置等普通配置文件 |
| `character_cards` | 角色卡文件及其中保存的人设、模型引用等角色信息 |
| `memory` | 各角色的长期记忆数据库；SQLite 数据库通过在线备份接口处理，包含尚未 checkpoint 的 WAL 数据 |

`assets` 模型资源组：

| 目录 | 内容 |
| --- | --- |
| `card_faces` | 角色头像和卡面资源 |
| `live2d` | Live2D 模型及其附属文件 |
| `vrm` | VRM 模型、动画等资源 |
| `mmd` | MMD 模型、动作等资源 |
| `pngtuber` | PngTuber 立绘资源 |
| `workshop` | 创意工坊下载或管理的资源 |

两个备份组彼此独立。只创建 `core` 快照不会同时备份模型资源，只创建 `assets` 快照也不会包含配置和记忆。

## 不会备份

- N.E.K.O 程序本体、源代码、前端文件和运行依赖。
- `plugins` 目录中的其他插件源码、插件配置和插件运行数据；本插件的快照目录也不会递归备份自身。
- 日志、缓存、临时文件，以及不在上述固定目录中的其他数据。
- 操作系统钥匙串、环境变量、浏览器或外部程序保存的凭证。实际写在 `config` 目录普通文件中的内容仍会进入快照。
- 符号链接及其指向的数据，或当前 N.E.K.O 数据根目录之外的文件。

## 快照与恢复

- 快照保存在当前 N.E.K.O 数据根目录的 `plugins/data_backup/data/snapshots` 下，每组默认保留最近 10 份。
- 恢复会把整个备份组还原为快照时的精确状态，因此快照创建后新增到该组目录中的文件也会被移除。
- 恢复前会校验快照内容并自动创建一份当前状态的安全快照；恢复完成后需要立即重启 N.E.K.O。
- 快照与原数据默认位于同一数据根目录，适合误删或配置损坏后的回滚，不能替代针对磁盘损坏的异盘备份。重要快照应另行复制到其他磁盘或云存储。
- 配置和记忆可能含有隐私信息，请按敏感数据保护快照目录。

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

推送与 `plugin.toml` 版本一致的标签（例如 `v0.1.1`）后，发布工作流会生成 `data_backup.neko-plugin` Release 附件。
