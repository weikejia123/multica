# multica — 多编码 Agent 编排工作台（Multica）

| 维度 | 内容 |
|------|------|
| **用途** | 开源多 agent 编排工作台：像分配同事一样给 AI 编码 agent（Claude Code/Codex/Cursor/Copilot/Kimi/OpenCode 等 20+ CLI）分配 issue，agent 自动领取、报告进度、提交回审 |
| **应用场景** | 团队/个人同时使用多个编码 agent 时的统一管理：看板（board）+ issue 分配 + 执行日志回放 + token 用量 + 审批门 + 定时 autopilot + 多渠道通知（Slack/飞书/钉钉/企微） |
| **标签** | Agent 编排 · 多 Agent · Go · 看板 · 自托管 · 20+ CLI |
| **技术栈** | Go 后端（Chi/sqlc/gorilla-websocket）+ pnpm monorepo 前端（Next.js web + Electron desktop + Expo RN mobile）；Turborepo |
| **内部版本** | V1-20260814 |
| **关联** | 上游 `multica-ai/multica`（★45,892）；fork `weikejia123/multica`；本地 `projects/coder-agent/multica/`；Gitea `dzsoft/multica` |

## 目录位置

`projects/coder-agent/multica/` — 归入 coder-agent 分类目录（AI 编程 Agent 集合）。

## Fork 策略

- **Add-only**：不修改上游代码文件，仅新增 my-docs/ 本地文档
- 分支：`main` = upstream/main（纯净，同步上游）；`wkj-dev` = 二开分支
- 三远程：upstream（官方上游）/ origin（个人 fork）/ gitea（本地备份）

## ⚠️ 许可注意

**Multica License**：Apache-2.0 + Part I 附加条件（两者合为 "Multica License"，均不可单独授权）。非纯 Apache-2.0，二次分发需遵守附加条件。fork 仅本地研究。

## 核心能力

| 能力 | 说明 |
|------|------|
| 20+ agent CLI | Claude Code/Codex/Cursor/Copilot/Kimi/OpenCode 等，agent 以"团队成员"形态出现在看板 |
| issue 分配 | 像分配同事一样指派 agent，自动领取/执行/评论/交回审查 |
| Squads | agent + 人混合团队，leader 路由工作 |
| Skills | 把解决过的问题变成所有 agent 复用的 playbook |
| 自托管运行 | daemon 跑在你自己的机器/云盒，代码不出本地 |
| 执行日志 | 回放每次工具调用/命令/错误（带时间戳）+ token 用量 |
| 审批门 | 工作落在 review 而非 main，人类决定什么上线 |
| Autopilots | 站会/审计/报告 cron 定时跑 |
| VCS 集成 | GitHub/GitLab/Gitea/Forgejo，含自托管 |
| 渠道 | Slack/飞书/钉钉/企微（钉钉企微社区维护） |
| 多端 | Web/Desktop(macOS/Win/Linux)/Mobile(iOS 源码构建) |

## 构建 / 运行

```bash
# 开发（自动 setup + 启动全部）
make dev

# 自托管（Docker）
docker compose -f docker/docker-compose.yml up -d

# 验证
make check
```

## 与目录内其他项目的定位

- **multica** — 多 agent 编排层（看板/issue/审批），对接各家 CLI
- **AionUi** — agent 驾驶舱（Electron + ACP 协议），也对接 20+ CLI —— 两者高度可比，见 `my-docs/00-项目分析/20260814-multica-vs-aionui.md`
- **codex/claude-code/pi 等** — 被编排的单个 agent CLI

## 评估记录

- 2026-08-14: 加入 fork（weikejia123/multica），存 coder-agent/ 分类目录，分支 wkj-dev，Gitea 备份完成
