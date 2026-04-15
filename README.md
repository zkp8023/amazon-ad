# amazon-ad

亚马逊广告智能投放 Claude Code 插件。

提供与 Amazon 广告优化相关的命令与技能，帮助你更方便地组织广告投放、优化思路与开发辅助能力。

## 安装

### Claude Code 安装方式

先添加 marketplace：

```bash
/plugin marketplace add zkp8023/amazon-ad
```

再安装插件：

```bash
/plugin install amazon-ad@amazon-ad-marketplace
```

## 包含内容

- `commands/`：插件命令
- `skills/`：技能定义
- `agents/`：代理能力（如有）
- `.claude-plugin/plugin.json`：插件元信息
- `.claude-plugin/marketplace.json`：marketplace 配置

## 使用

安装完成后，可在 Claude Code 中使用 `/plugin` 查看已安装插件，并使用该插件提供的命令与技能。