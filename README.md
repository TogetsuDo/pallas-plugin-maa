<p align="center">
  <img src="./assets/brand-avatar.png" width="220" height="220" alt="MAA 远控">
</p>

<h1 align="center">MAA 远控 pallas-plugin-maa</h1>

<p align="center">提供 MAA 设备绑定、任务排队与结果回传能力。</p>

<p align="center">
  <img alt="官方插件" src="https://img.shields.io/badge/%E5%AE%98%E6%96%B9%E6%8F%92%E4%BB%B6-FE7D37">
  <img alt="控制台插件商店" src="https://img.shields.io/badge/%E6%8E%A7%E5%88%B6%E5%8F%B0-%E6%8F%92%E4%BB%B6%E5%95%86%E5%BA%97-4EA94B">
  <img alt="安装命令" src="https://img.shields.io/badge/uv%20run%20pallas%20ext%20install%20pallas--plugin--maa-586069">
  <img alt="PyPI 版本" src="https://img.shields.io/pypi/v/pallas-plugin-maa?label=%E7%89%88%E6%9C%AC&color=2563EB">
</p>

## 安装方式

需已安装 [Pallas-Bot](https://github.com/PallasBot/Pallas-Bot) **≥ 4.0**。

推荐直接在控制台插件商店安装，或在本体项目中执行：

```bash
uv run pallas ext install pallas-plugin-maa
```

也可单独安装本包：

```bash
uv pip install pallas-plugin-maa
```

开发联调：clone 本仓库后 `uv pip install -e .`（`pyproject.toml` 可配置本体 path 依赖）。

## 多进程分片

Pallas-Bot 支持单进程，也支持 **hub + 多个 worker** 的多进程部署。启用分片时：

- **hub 与每个 worker 须安装相同版本的本扩展包**；
- 各进程共享同一路径的 **`data/`**（注册表、协调状态、WebUI 落盘等）；
- MAA 任务队列与路由状态走 Redis 协调层；对外 HTTP 基址通常配置在 **hub**。

本插件通过本体 **`plugin_coord`** 与启动时的 **`register_maa_coord()`** 接入协调层；hub 角色加载 **`pallas_plugin_maa_hub`** 转发 MAA HTTP。

详见：[多进程分片 · 架构说明](https://PallasBot.github.io/Pallas-Bot-Docs/architecture/bot-process-sharding)

## 怎么使用

[MAA 远程控制协议](https://docs.maa.plus/zh-cn/protocol/remote-control-schema.html)：`getTask` / `reportStatus` + QQ 绑定、口令排队、结果回传。

### 用户命令

| 类型 | 口令示例 |
| --- | --- |
| 绑定 | `牛牛绑定MAA`、`牛牛MAA状态`、`牛牛切换MAA设备`、`牛牛MAA设备名`、`牛牛清空MAA队列` |
| 任务 | `牛牛长草`、`牛牛作战`、`牛牛公招`、`牛牛基建`、`牛牛截图`、`牛牛停止` 等 |
| 高级 | `牛牛MAA任务 <type> [params]` |

**上手**：配置 `maa_public_base_url` → MAA 填帮助页 URL（用户标识符 = QQ）→ 私聊绑定设备 → 群聊发口令。完整表见 **牛牛帮助 → MAA 远控**。

#### 多台设备

| 口令 | 说明 |
| --- | --- |
| `牛牛MAA状态` | 列表与当前选用 |
| `牛牛切换MAA设备` | 改远控目标 |
| `牛牛MAA设备名` | 设置别名 |

### 命令权限

| 命令 ID | 默认等级 |
| --- | --- |
| `maa.bind` | everyone |
| `maa.control` | everyone |
| `maa.status` | everyone |

> 详细用法、限制条件和可用范围以帮助为主。

## 配置项

一般只需 **`maa_public_base_url`**（WebUI **服务网关 / 连通性** 亦可编辑）。

| 键 | 默认 | 说明 |
| --- | --- | --- |
| `maa_public_base_url` | 空 | 对外 HTTP 基址 |
| `maa_attach_screenshot` | true | 指令后附加截图 |
| `maa_combat_auto_prepare` | true | 作战前自动排队关卡设置 |

完整键见本仓库 [`config.py`](src/pallas_plugin_maa/config.py)。改 `maa_get_task_path` 等会重挂路由并清帮助缓存。

## 排障

| 现象 | 处理 |
| --- | --- |
| 未检测到轮询 | MAA 端点不可达或 URL 错误；分片须 hub 配置 `maa_public_base_url` 且各 worker 共用 `data/` |
| 状态有待拉取、MAA 无任务 | 分片时队列走 Redis `pallas:coord:maa_pending:*`；须 hub 能访问各 worker 端口并保证 Redis 可用 |
| 下发后无任务 | 未绑定或用户标识符非 QQ；查 `牛牛MAA状态` |
| 队列有、MAA 无 | 设备 id 与「当前选用」不一致；可清空队列重试 |
| 截图失败 | 调大反代 `client_max_body_size` |

## 维护者说明

#### 任务分类

| 分类 | type 示例 | MAA 行为 |
| --- | --- | --- |
| 顺序任务 | `LinkStart`、`CaptureImage`、`Settings-*` | 按队列顺序执行 |
| 立即任务 | `CaptureImageNow`、`StopTask`、`HeartBeat` | 可插队 |

#### 唤醒与子项

- `LinkStart`（牛牛长草）：含唤醒 + 按勾选跑子模块
- `LinkStart-WakeUp`：仅唤醒
- 其它 `LinkStart-*`：不含唤醒；游戏需已在主界面
- 牛牛作战当前临时下发 `LinkStart`（`COMBAT_COMMAND_TASK_TYPE`），上游修复后改回 `LinkStart-Combat`

#### 作战与关卡

- `牛牛设置关卡`：最多 4 候选，仅下发 `Settings-Stage1`
- `maa_combat_auto_prepare`：作战前可自动排队已保存主关卡

#### 多 Bot 同群

群内远控口令与 `牛牛MAA状态` 等命令经 `claim_group_handler("maa", …)`，同一条群消息仅一只牛响应。私聊绑定/切换设备不受影响。

#### 代码索引

| 逻辑 | 位置 |
| --- | --- |
| 口令 → type | `tasks.py` |
| HTTP | `http_api.py`、`http_routes.py` |
| 队列/绑定 | `store.py` |

## 实现

源码位置：

- worker：[`src/pallas_plugin_maa/`](src/pallas_plugin_maa/)
- hub 入口：[`src/pallas_plugin_maa_hub/`](src/pallas_plugin_maa_hub/)

实现要点：

- worker 侧负责命令解析、状态管理与任务下发。
- hub 侧负责对外 HTTP 暴露与分片路由转发。
- 分片部署时依赖共享 `data/` 与 Redis 协调层保持队列一致。

## 相关链接

| 说明 | 链接 |
| --- | --- |
| MAA 远控 · 用户文档 | [文档站 · maa](https://PallasBot.github.io/Pallas-Bot-Docs/plugins/maa) |
| 插件开发入门 | [develop/plugin/getting-started](https://PallasBot.github.io/Pallas-Bot-Docs/develop/plugin/getting-started) |
| 多进程分片 | [architecture/bot-process-sharding](https://PallasBot.github.io/Pallas-Bot-Docs/architecture/bot-process-sharding) |
