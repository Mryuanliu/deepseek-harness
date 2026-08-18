# 同步与提交速查（deepseek-harness fork）

本仓库：本地 `dev` 是你自己的迭代分支，`master` 是上游镜像，`upstream` 指向官方 `deepseek-ai/deepseek-harness`，`origin` 是你的 fork。

## 1. 拉取上游最新代码

```sh
# 1) 拉上游更新（只更新远端引用，不影响你的工作区）
git fetch upstream

# 2) 把 master 快进到上游最新（master 上不要有自己的提交）
git switch master
git merge --ff-only upstream/master

# 3) 回到你的开发分支
git switch dev
```

## 2. 把最新代码合并到你的改动（dev）

**推荐：rebase（历史线性、干净）**

```sh
git switch dev
# 先保证工作区干净：提交或暂存未提交的改动
git add -A && git commit        # 或：git stash -u
git rebase master               # 把你的提交重放到最新上游之上
# 有冲突时：解决后 git add <file> 再 git rebase --continue
# dev 已推送到 fork 且被 rebase 过，需要带保护地强推：
git push --force-with-lease origin dev
```

**保守：merge（不重写历史）**

```sh
git switch dev
git merge master                # 生成一个合入点
# 有冲突时：解决后 git add <file> 再 git merge --continue
git push origin dev
```

## 3. 提交你的改动

```sh
git switch dev
git add <文件>                  # 或 git add -A 全部
git commit -m "feat(scope): 说明"   # 遵循 type(scope): subject
git push                        # dev 已跟踪 origin/dev，直接推即可
```

## 4. 常用查看命令

```sh
git status                                     # 当前工作区
git log --oneline master..dev                  # 你比上游多了哪些提交
git log --oneline upstream/master..master      # master 待吸收的上游提交
git fetch upstream && git log --oneline master..upstream/master   # 上游有没有新提交
```

## 常见坑

- 永远不要在 `master` 上提交，否则 `--ff-only` 同步会失败。
- `rebase` 前确保工作区干净（先 `commit` 或 `stash`）。
- 上游改动依赖后先 `pnpm install` 再继续开发。
- 强推只对自己的分支用 `--force-with-lease`，绝不用裸 `--force`。
- 完整工作流与质量门槛见根目录 `FORK_DEV_GUIDE.md`。
