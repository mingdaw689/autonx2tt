# Issue tracker：GitHub

本仓库的 issue 与 spec 以 GitHub issue 形式存放。所有操作使用 `gh` CLI。

> 前置条件：当前目录尚未初始化为 git 仓库，也没有 GitHub remote。需要先执行 `git init`、添加 GitHub remote（`git remote add origin ...`）并完成 `gh auth login`，`gh` 才能在仓库内自动推断目标仓库。

## 约定

- **创建 issue**：`gh issue create --title "..." --body "..."`。多行正文使用 heredoc。
- **读取 issue**：`gh issue view <number> --comments`，用 `jq` 过滤评论，并同时获取标签。
- **列出 issue**：`gh issue list --state open --json number,title,body,labels,comments --jq '[.[] | {number, title, body, labels: [.labels[].name], comments: [.comments[].body]}]'`，按需配合 `--label` 与 `--state` 过滤。
- **评论 issue**：`gh issue comment <number> --body "..."`
- **添加 / 移除标签**：`gh issue edit <number> --add-label "..."` / `--remove-label "..."`
- **关闭**：`gh issue close <number> --comment "..."`

仓库信息从 `git remote -v` 推断；在克隆仓库内运行时，`gh` 会自动完成推断。

## 将 PR 作为 triage 入口

**将 PR 作为请求入口：否。** _（若本仓库把外部 PR 当作功能请求，改为 `yes`；`/triage` 会读取该标志。）_

设为 `yes` 时，PR 与 issue 走相同的标签和状态流程，使用对应的 `gh pr` 命令：

- **读取 PR**：`gh pr view <number> --comments`，diff 用 `gh pr diff <number>`。
- **列出待 triage 的外部 PR**：`gh pr list --state open --json number,title,body,labels,author,authorAssociation,comments`，仅保留 `authorAssociation` 为 `CONTRIBUTOR`、`FIRST_TIME_CONTRIBUTOR` 或 `NONE` 的项（丢弃 `OWNER`/`MEMBER`/`COLLABORATOR`）。
- **评论 / 打标签 / 关闭**：`gh pr comment`、`gh pr edit --add-label` / `--remove-label`、`gh pr close`。

GitHub 的 issue 与 PR 共用同一编号空间，因此裸写的 `#42` 可能是其中任意一种：先用 `gh pr view 42` 解析，失败再回退到 `gh issue view 42`。

## 当某个 skill 说「发布到 issue tracker」

创建一个 GitHub issue。

## 当某个 skill 说「获取相关 ticket」

运行 `gh issue view <number> --comments`。

## Wayfinding 操作

供 `/wayfinder` 使用。**map** 是一个 issue，**child** 是其下的 ticket。

- **Map**：单个带 `wayfinder:map` 标签的 issue，正文承载 Notes / Decisions-so-far / Fog 内容。`gh issue create --label wayfinder:map`。
- **Child ticket**：作为 map 的 GitHub 子 issue 关联（通过 `gh api` 调用 sub-issues 端点）。若未启用子 issue，则在 map 正文的任务列表中加入该 child，并在 child 正文顶部写 `Part of #<map>`。标签为 `wayfinder:<type>`（`research`/`prototype`/`grilling`/`task`）。认领后，ticket 分配给执行的开发者。
- **阻塞关系**：使用 GitHub 原生 issue 依赖，这是规范且 UI 可见的表示方式。通过 `gh api --method POST repos/<owner>/<repo>/issues/<child>/dependencies/blocked_by -F issue_id=<blocker-db-id>` 添加边，其中 `<blocker-db-id>` 是阻塞者的数字**数据库 id**（`gh api repos/<owner>/<repo>/issues/<n> --jq .id`，_不是_ `#number` 或 `node_id`）。GitHub 通过 `issue_dependencies_summary.blocked_by` 报告（仅未关闭的阻塞者，即实时门禁）。若依赖功能不可用，回退为在 child 正文顶部写 `Blocked by: #<n>, #<n>`。当所有阻塞者都关闭时，ticket 解除阻塞。
- **Frontier 查询**：列出 map 下所有未关闭的 child（`gh issue list --state open`，限定在 map 的子 issue / 任务列表范围内），丢弃存在未关闭阻塞者（`issue_dependencies_summary.blocked_by > 0`，或 `Blocked by` 行中仍有未关闭 issue）或已有 assignee 的项；按 map 顺序取第一个。
- **认领**：`gh issue edit <n> --add-assignee @me`，这是本次会话的第一次写操作。
- **解决**：先 `gh issue comment <n> --body "<answer>"`，再 `gh issue close <n>`，最后把上下文指针（要点 + 链接）追加到 map 的 Decisions-so-far。
