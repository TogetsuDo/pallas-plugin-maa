# 更新日志

本文件依据 git tag 历史整理，版本号遵循[语义化版本](https://semver.org/lang/zh-CN/)。
新提交合入后请在 `## [Unreleased]` 下记录，发布时随版本 tag 归档。

## [Unreleased]

## [4.0.13] - 2026-06-27
- docs(readme): 命令权限默认等级改用中文展示

## [4.0.12] - 2026-06-27

- feat(maa): 补齐清空队列/切换设备等命令权限声明，冷却接入 command_limits 并显示中文名

## [4.0.11] - 2026-06-27
- docs(readme): 「怎么使用」口令统一加行内代码标记

## [4.0.10] - 2026-06-27
- fix(help): 二级帮助用法「下表」改为「上方功能一览」，适配 help v3 布局

## [4.0.9] - 2026-06-27
- fix(help): 帮助页用法文案去掉 Markdown 加粗星号，适配 PIL v3 成图

## [4.0.8] - 2026-06-25
- feat(metadata): 补充绑定 MAA 命令冷却声明

## [4.0.7] - 2026-06-24
- feat(knowledge): 声明 knowledge_sources FAQ 供 LLM 注入

## [4.0.6] - 2026-06-19
- docs(assets): 更新头像资源并改用 PyPI 版本徽章
- chore(assets): 替换品牌头像为透明背景版本

## [4.0.5] - 2026-06-18
- docs(readme): 统一官方插件卡片模板

## [4.0.4] - 2026-06-18
- docs(readme): 更新官方扩展安装命令

## [4.0.3] - 2026-06-18
- migrate: src.* → pallas.api.* / pallas.product.* / pallas.core.*
- release: bump to 4.0.3 for pallas import migration

## [4.0.2] - 2026-06-18
- docs(readme): 添加 Pallas-Bot hero 图
- chore(release): 4.0.2 同步 README 进 PyPI 包

## [4.0.1] - 2026-06-17
- feat: Pallas-Bot 4.0 官方扩展首包
- fix(build): 修正 hatch wheel 的 src 包路径
- feat(release): PyPI 发版 workflow 与 4.0.1
