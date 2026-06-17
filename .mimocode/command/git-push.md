---
description: 一键提交并推送当前项目的 git 更改。自动检查状态、生成 commit message、推送并验证。
---

# Git Commit & Push

将当前工作目录（或 `$ARGUMENTS` 指定的子目录）的 git 更改提交并推送到远程仓库。

## 流程

1. **检查状态** — `git status` 查看未跟踪/已修改文件，`git diff --stat` 查看变更统计
2. **检查远程** — `git remote -v` 确认远程仓库配置
3. **查看最近提交** — `git log --oneline -3` 了解提交风格
4. **暂存更改** — `git add -A`
5. **生成 commit message** — 根据 `git diff --cached --stat` 和实际变更内容，撰写简洁的中文 commit message：
   - 格式：`<type>: <简短描述>`，换行后列出要点
   - type 使用 feat / fix / refactor / docs / chore
   - 如果用户提供了 `$ARGUMENTS`，结合参数内容生成
6. **提交** — `git commit -m "..."`
7. **推送** — `git push origin main`（或当前分支）
8. **验证** — `git status && git log --oneline -3` 确认提交成功

## 注意事项

- 如果没有变更，直接告知用户"没有需要提交的更改"并停止
- 如果远程仓库未配置，提示用户先设置 remote
- commit message 使用中文，与项目现有风格保持一致
- 不要使用 `--force` 推送
