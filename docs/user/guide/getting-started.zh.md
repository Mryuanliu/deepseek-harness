# DeepSeek Harness 新手上手指南

[English](getting-started.md) | 中文

本教程带你从一台干净的机器启动 Web UI 并跑通第一个 agent 会话。假定你可以自行安装软件；只有 agent 调用模型时才需要 DeepSeek API 密钥。

## DeepSeek Harness 是什么

DeepSeek Harness（`dsh`）是 DeepSeek AI 开发的开源 agent 框架。它的每一项能力都是运行在 Cordis 上的插件：agent 主循环、工具、shell 与文件系统能力、网络搜索、LLM 提供方、Web UI 本身，都可以替换或扩展。各部分的组合方式见[架构总览](../architecture.md)。

## 环境要求

- Node.js 22.19+ 或 24+。
- pnpm（本仓库锁定 pnpm 11.7.0；corepack 会按 lockfile 激活）。
- Git 2.26+。
- 可选但推荐：真实 agent 任务需要 DeepSeek API 密钥。

检查工具：

```sh
node --version
git --version
```

若缺少 `pnpm`，用 corepack 激活锁定版本：

```sh
corepack enable
corepack prepare pnpm@11.7.0 --activate
pnpm --version
```

如果 corepack 的 shim 不在你的 PATH 中，把上述命令改成通过 `corepack pnpm` 执行即可。

## 获取源码

```sh
git clone https://github.com/deepseek-ai/deepseek-harness.git
cd deepseek-harness
```

## 安装与构建

```sh
pnpm install
pnpm run build
```

`pnpm install` 会安装所有 workspace 包并配置 Lefthook Git 钩子；`pnpm run build` 编译 TypeScript 包并构建 Web 前端。源码检出必须成功构建后才能运行。

## 启动 Web UI

```sh
pnpm dsh web
```

命令会打印访问地址；UI 默认跑在 `http://127.0.0.1:3080`。保持该进程运行，在浏览器中打开地址。

## 配置模型

打开**设置 → 模型**，输入 DeepSeek API 密钥并保存。模型路由立即可用，无需重启。其他提供方与自定义 OpenAI 兼容端点见[模型配置指南](./providers.md)。

命令行模式改用仓库根目录的 `.env` 存放密钥：

```sh
DEEPSEEK_API_KEY=sk-...
```

## 运行第一个任务

点击**选择工作区**，添加一个项目目录并选中。启动一个会话并发送提示，例如：

> Summarize this repository and identify its main packages.

agent 可以读取和编辑工作区文件、运行命令、委派工作并维护计划。当前权限策略下需要审批的操作会先询问你。

## 其他入口

- [Web UI 指南](./index.md)更详细地介绍模型与工作区配置。
- 无头模式执行一次任务后退出：`pnpm dsh --profile headless "run the tests"`。
- ACP 自动化服务与自指 Cordis demo 需要密钥：`pnpm run demo:acp`、`pnpm run demo:cordis`。
- [Python SDK](./python-sdk.md)从 Python 驱动 agent。
- 插件开发从[第一个插件教程](../develop/basic/)开始。
- [CLI 参考](../../../apps/cli/README.md)说明 `--profile`、`--patch`、`--dump-config` 等 profile 与启动参数。

## 常用开发命令

修改代码后运行聚焦的检查：

```sh
pnpm run typecheck
pnpm run test
pnpm run test:snapshot
pnpm run lint
pnpm run build
```

非平凡改动需要配套 Agent Notes、测试与快照；参与贡献前请读 [CONTRIBUTING.md](../../../CONTRIBUTING.md) 和 [docs/development.md](../../development.md)。

## 常见问题

- `pnpm: command not found` — 按上文用 corepack 激活 pnpm，或用 `corepack pnpm` 执行相同命令。
- `Port 3080 already in use` — 停掉占用该端口的进程（`lsof -iTCP:3080 -sTCP:LISTEN`），或查看 web profile 暴露的端口参数（`pnpm dsh web --help`）。
- 启动时创建 `~/.dsh` 报 `EPERM` — 进程无法写入 profile 目录；检查目录权限，或不要在受限沙箱里运行。
- UI 能打开但任务因没有模型而失败 — 在**设置 → 模型**或根目录 `.env` 配置 API 密钥；无需重启服务器。

## 下一步

- [Web UI 指南](./index.md) — 模型与工作区配置。
- [架构总览](../architecture.md) — 插件系统如何组合。
- [开发指南](../../development.md) — 贡献者环境与日常流程。
- [Cookbook](../../cookbook/adding-a-package.md) — 分步操作手册。
