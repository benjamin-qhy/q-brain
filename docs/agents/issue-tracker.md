# Issue tracker: GitHub

本项目的需求和任务使用 benjamin-qhy/q-brain 的 GitHub Issues。
使用 gh CLI，执行前核对 git remote，避免操作参考仓库。

## 常用操作

- 创建：gh issue create --title "标题" --body-file <正文文件>
- 阅读：gh issue view <编号> --comments
- 列表：gh issue list --state open
- 评论：gh issue comment <编号> --body-file <正文文件>
- 标签：gh issue edit <编号> --add-label <标签> 或 --remove-label <标签>
- 关闭：gh issue close <编号>

多行正文写入临时文件，再通过 --body-file 传入。

技能要求“发布到任务跟踪系统”时，创建 GitHub Issue。
技能要求“读取任务”时，读取对应 Issue 及评论。

## Pull requests as a triage surface

PRs as a request surface: no.

## Wayfinding

规划总任务使用 wayfinder:map 标签，子任务使用 wayfinder:<type>。
优先使用 GitHub 子任务和原生依赖；不可用时用任务清单、
Part of #编号 和 Blocked by: #编号 表达。

按总任务中的顺序选择未分配、没有未完成阻塞项的子任务。
领取时分配给当前用户；完成后记录结果、关闭子任务，
并更新总任务中的决定与链接。
