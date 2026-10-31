# XDUCraft Wordcloud

基于 NoneBot 2 与 OneBot v11 的群聊天词云图生成与定时推送插件。按群记录聊天文本并过滤停用词，支持每日 00:00 自动定时推送或管理员按需生成词云图。

运行整套机器人请看 [FlyingPig278/XDUCraft-bot](https://github.com/FlyingPig278/XDUCraft-bot)。

## 安装与加载

本软件包未发布至公共 PyPI 仓库，请使用 Git 地址安装。

### 1. 安装核心包与插件

在现有的 NoneBot 2.4.x 环境中，先安装 Core 核心包，再安装本插件：

```bash
pip install "nonebot-plugin-xducraft-core[onebot] @ git+https://github.com/FlyingPig278/nonebot-plugin-xducraft-core.git@v0.3.0"
pip install "nonebot-plugin-xducraft-wordcloud @ git+https://github.com/FlyingPig278/nonebot-plugin-xducraft-wordcloud.git@v0.1.0"
```

### 2. 配置插件加载

在 NoneBot 项目的 pyproject.toml 中向现有 plugins 列表追加以下模块（请保留已有的其他插件和适配器）：

```toml
[tool.nonebot]
plugins = [
    "xducraft_config",
    "xducraft_wordcloud",
]
```

## 启用与首次使用

插件安装后，全局总开关与各群开关默认处于关闭状态。

### 1. 开启插件

方式一：使用 Web 管理面板

如果配置了 [nonebot-plugin-xducraft-admin](https://github.com/FlyingPig278/nonebot-plugin-xducraft-admin)，访问管理面板（默认地址 http://127.0.0.1:8765）：
1. 在“插件”页面开启“群聊词云”全局开关。
2. 在“群聊”页面选择对应群，开启本群的“群聊词云”插件开关。

方式二：不使用 Web 管理面板

如果不运行管理面板，请参考 Core 文档中的[不使用网页时启用插件](https://github.com/FlyingPig278/nonebot-plugin-xducraft-core#不使用网页时启用插件)。通过一次性脚本调用 Core 配置接口，同时开启全局开关与目标群的插件开关。由于 Core 会在前置检查中拦截未在群中启用的插件消息，必须在 Core 中同时配置全局与群开关。

### 2. 群内使用与指令列表

开启全局与群开关后，即可在群内使用。管理指令均需要群管理员、群主或超级用户权限：

- `/wc on`：开启本群聊天记录与每日词云推送（如果群开关已开启，此指令可用于同步启用群功能）。
- `/wc off`：关闭本群聊天记录与每日词云推送。
- `/wc status`：查看本群词云参数、当前记录条数与启用状态。
- `/wc gen [today|yesterday|YYYY-MM-DD]`：生成并发送指定日期的词云图（默认为昨天；首次测试可发送 `/wc gen today` 即时查看今日词云）。
- `/wc ban add [QQ号|@用户]`：将指定用户加入本群屏蔽列表，不记录且不统计其发言。
- `/wc ban del [QQ号|@用户]`：从本群屏蔽列表中移除指定用户。
- `/wc ban list`：查看本群已屏蔽的用户列表。
- `/wc clear [QQ号|@用户]`：清除指定用户在本群的历史聊天记录。

## 数据隐私与存储说明

- 仅在已开启词云的群中收集聊天文本，未开启的群不会记录任何聊天数据。
- 当群成员撤回发言时，如果所在群已开启词云，插件会捕获撤回事件并立即从本地数据库中删除对应消息记录。
- 被屏蔽的成员发言不会被记入数据库，也不会出现在生成的词云图中。
- 聊天记录保存在本地独立 SQLite 数据库 `<DATA_DIR>/plugins/wordcloud/chat_logs.db` 中，不与 Core 配置数据库混用。
- 聊天记录默认保存 1095 天（可通过 `retention_days` 调整），系统会在每日 00:00 自动清理超出保留天数的过期记录。
- 如需自定义数据存放目录，请在启动 Python 进程前配置操作系统环境变量 `XDUCRAFT_DATA_DIR` 为绝对路径。

## 配置项说明

### 全局配置

- `retention_days`：聊天记录最大保留天数，默认为 1095 天。
- `ban_user_ids`：全局屏蔽的 QQ 号列表，所有群统一排除其发言。

### 群级配置

可在 Web 管理面板群卡片中调整，保存在 Core 数据库中：

- `ban_user_ids`：本群额外屏蔽的 QQ 号列表，与全局屏蔽列表合并生效。
- `min_word_length`：分词最短词长，默认为 2 个字符。
- `max_words`：词云图最多展示词数，默认为 200。
- `additional_stopwords`：本群额外停用词列表。
- `footer_subtitle`：词云图底部副标题。
- `footer_branding_enabled` 与 `footer_branding_text`：底部署名开关与展示文本。

## 本地开发与测试

克隆本仓库后，可以在本地虚拟环境中安装开发依赖并运行测试：

```bash
pip install "nonebot-plugin-xducraft-core[onebot] @ git+https://github.com/FlyingPig278/nonebot-plugin-xducraft-core.git@v0.3.0"
pip install -e ".[dev]"
pytest
python -m build
```

## 数据存储与历史迁移

历史配置文件 `wordcloud_config.json` 仅用于首次启动时单次导入。Core 首次启动会自动将其导入 Core SQLite 数据库的 `wordcloud` 命名空间中，导入完成后旧文件不再保存新设置，后续修改以数据库为准。

## 许可证与资产说明

本项目源码采用 MIT 许可证开源，详见 [LICENSE](LICENSE)。

内置思源黑体历史版本字体文件 `resources/fonts/SourceHanSansCN-Medium.otf` 版权归 Adobe Systems Incorporated 所有，其内嵌声明遵循 Apache License 2.0，许可文本参见 [LICENSES/SourceHanSansCN-Apache-2.0.txt](LICENSES/SourceHanSansCN-Apache-2.0.txt)，详细说明见 [NOTICE](NOTICE)。该历史二进制资产未重新标记为后续新版所采用的 OFL 许可。本项目不主张该字体文件的 MIT 授权。
