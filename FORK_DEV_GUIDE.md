# Fork 开发与迭代指南

本指南面向在本仓库基础之上做二次迭代的场景：**自己的改动长期保留，同时持续跟进官方上游最新代码**。适用于当前仓库（fork 自 deepseek-ai/deepseek-harness，MIT 许可）。

## 分支模型

- `master`：上游镜像，永远只做快进跟进（`--ff-only`），绝不提交自研代码。保持干净才能用同步流程一键对齐。
- `dev`：长期开发分支，你的自研迭代都沉淀在这里，可以长命存在。
- `feat/xxx`、`fix/xxx`：具体功能/修复分支，从 `dev` 切出，完成后合回 `dev`。
- 你的 fork（`origin`）只推 `dev` 和功能分支做备份/协作，不推 `master`。

## 一次性初始化（已完成）

- `upstream` 已指向 `https://github.com/deepseek-ai/deepseek-harness.git`（对齐用 GitHub API 查到的 fork parent）。
- 本地 `master` 已与 `upstream/master` 对齐（commit `47f943859b`）。
- `dev` 分支已创建并处于当前分支；此前未提交的 `docs/user/guide/*` 与 `website/docs.ts` 改动已随分支带过去，请尽快作为你的第一个 dev 提交落盘。

## 日常开发流程

1. 开工前先同步上游（见下节）。
2. 从 `dev` 切功能分支：`git switch dev && git switch -c feat/xxx`。
3. 小步提交，提交信息遵循仓库习惯：`type(scope): subject`（如 `feat(agent-loop): ...`）。
4. 本地验证最小相关检查：`pnpm run typecheck`、`pnpm run lint`、`pnpm run test:coverage`（CI 门槛为 packages 下源文件逐文件 100%）；
   模型或用户可见行为按仓库测试政策补 keyless snapshot，非平凡改动按 AGENTS.md 写 Agent Note。
5. 合回 `dev`：`git switch dev && git merge feat/xxx`（想保持单条记录可用 `--squash`）。
6. 备份推送：`git push origin dev`；若 `dev` 被 rebase 过则 `git push --force-with-lease origin dev`。

## 同步上游（核心流程）

```sh
# 1. 拉上游更新（只更新远端引用，不影响工作区）
git fetch upstream

# 2. 把 master 快速前进到上游最新（master 上必须没有本地提交）
git switch master
git merge --ff-only upstream/master

# 3. 回到 dev，把你的自研改动重放到最新上游之上
git switch dev
git rebase master        # 线性历史；冲突逐个解决
# 保守替代：git merge master   # 不重写历史，保留合入点（代价是历史里有同步点）

# 4. dev 已推送且被 rebase 后，用 lease 保护地推回 fork
git push --force-with-lease origin dev
```

`rebase` 前提是工作区干净。有未提交改动时先收起来：

```sh
git stash -u && git rebase master && git stash pop
```

## 冲突处理

- 冲突时 `git status` 查看冲突文件，解决后 `git add` 再 `git rebase --continue`（或 `git merge --continue`）。
- 冲突大概率集中在你自定义过的模块；用 `git log` 查看上游对该文件的改动以判断取舍。
- 拿不准时宁可保守对齐上游行为，你的自定义差异单独用 commit 记录，便于下次同步复用判断。

## 质量门槛（遵循仓库 AGENTS.md）

- 每次改动带单测；模型/用户可见行为的变化必须配可回放的真实示例与 keyless snapshot。
- 非平凡改动必须同时在 `.agents/notes/` 里写 Agent Note，与改动同 PR 提交。
- 文档改动遵循 `docs/AGENTS.md`：双语文档、每段一行、一个事实一个归属；提交前跑 `pnpm run doc-sync`。
- 推送前按 `.agents/skills/dsh-pre-push-checks/SKILL.md` 选择最小必要检查，不要无谓跑全量。

## 版本与发布（可选）

- 自研阶段保持 `"private": true`，或改用你自己的 npm scope；避免占用 `@deepseek-ai/*` 命名空间。
- fork 是公开仓库（`Mryuanliu/deepseek-harness`），可在 README 中声明与上游的关系与差异。

## 常见坑

- 别在 `master` 上提交：一旦分叉，`--ff-only` 同步会失败，还得手工对齐。
- `rebase` 前确保工作区干净（commit 或 stash）。
- 上游改动依赖后先 `pnpm install` 再开发。
- 永远不要对其他人的仓库 `--force`；自己的 fork 用 `--force-with-lease`。
- 官方上游现阶段不接受外部 PR（见 `CONTRIBUTING.md`）；你的二次迭代在本 fork 生态内进行即可。
