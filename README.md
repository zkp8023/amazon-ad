# amazon-ad

亚马逊广告智能投放 Claude Code / Codex项目插件。

当前仓库同时兼容两种使用方式：

- Claude Code：通过 `.claude-plugin` 市场机制安装，可使用插件提供的命令与技能
- Codex：通过原生 skill discovery 安装，只会发现仓库 `skills/` 目录下的技能；不会使用 `.claude-plugin` 元数据或 `commands/` 目录

## 安装

### Claude Code 安装方式

添加 marketplace：

```bash
/plugin marketplace add zkp8023/amazon-ad
```

再安装插件：

```bash
/plugin install amazon-ad@amazon-ad-marketplace
```

### Codex 安装方式

推荐直接让 Codex 拉取并执行本仓库的安装说明：

```text
Fetch and follow instructions from https://raw.githubusercontent.com/zkp8023/amazon-ad/refs/heads/main/.codex/INSTALL.md
```