# multica vs AionUi — 多 Agent 管理平台深度对比

> 分析日期：2026-08-14
> 对比对象：`multica-ai/multica`（★45,892，Go） vs `iOfficeAI/AionUi`（★31.3K，Electron/TS）
> 分析基准：两项目本地 wkj-dev 分支源码（2026-08-14 克隆）

---

## 0. 一句话总览

| | **multica** | **AionUi** |
|---|---|---|
| **本质** | **任务编排工作台**——把 agent 当同事，issue 分配 → 看板 → 执行 → 审批 | **Agent 对话驾驶舱**——统一聊天界面接入 20+ CLI agent，可远程遥控 |
| **驱动模型** | 看板/issue 驱动（Kanban + 任务状态机） | 会话/聊天驱动（Conversation + 消息流） |
| **工作产出** | PR / diff / 审批评审门 | 对话回复 / 办公文档（PPT/Word/Excel） |
| **适合谁** | 小团队管理多 agent 干活、要可审计的执行流水 | 个人/团队统一操控多 agent、要远程和定时 |

---

## 1. 项目定位对比

### multica — "Agents that show up on the board"
- **核心痛点**：你装了 Claude Code、Codex、pi 等多个 agent，每个活在自己的终端里、会话结束就忘、上下文反复重讲。
- **解法**：把 agent 变成**看板上的一等成员**——给它分配 issue，它自己领取、自己跑（在你控制的 runtime 上）、边干边评论、交回审查。
- **关键承诺**：意图、运行、决策、diff 全部关联到同一 issue——**没人需要重建上下文，没有东西未经人类批准就上线**。
- 20+ agent CLI 运行时（claude/codex/copilot/opencode/cursor-agent/kimi/reasonix/pi/hermes/dsh/grok/qwen/trae 等）。

### AionUi — "AI Agent Cowork 工作台"
- **核心痛点**：装了多个 CLI agent，各自终端/配置/交互方式不同，无法统一查看切换。
- **解法**：一个桌面 App 统一接入 20+ CLI agent（自动检测本机已装），内置完整 Agent 引擎（零配置开跑），WebUI/TG/飞书/钉钉/微信多渠道远程访问。
- **关键承诺**：Agent Client Protocol（ACP）统一协议抽象——新增 agent 不重写 UI；内置 21 个办公文档助手。

---

## 2. 架构对比

| 维度 | multica | AionUi |
|------|---------|--------|
| 后端 | **Go**（Chi router + sqlc + gorilla/websocket + PostgreSQL/pgvector） | **Node/Electron**（内置引擎 aioncore 为 Rust 后端，见 AionCore fork） |
| 前端 | Next.js web + Electron desktop + Expo RN mobile + Fumadocs（pnpm monorepo + Turborepo） | Electron 37 + React 19 + Arco + UnoCSS（packages/desktop + web-cli + web-host + mobile） |
| 状态管理 | TanStack Query（server state）+ Zustand（client state）严格分层 | 进程双轨（main/renderer）经 IPC 桥 |
| 数据层 | PostgreSQL + sqlc 生成代码，**禁用外键/级联**（应用层事务） | SQLite/文件（客户端本地为主） |
| 部署 | 自托管 Docker Compose / Helm + 云端 SaaS | 本地桌面 + 自托管 web-host + 远程渠道 |
| 许可 | **Multica License**（Apache-2.0 + Part I 附加条件，非纯开源） | Apache-2.0 |

### multica 架构亮点（源码实证）
- `server/internal/` 30+ 模块：`daemon/`（本地执行）+ `dispatch/`（任务派发）+ `realtime/`（WS 事件）+ `scheduler/`（cron）+ `issueguard/`（防重复自指派）+ `selfexec/`（自执行）
- **daemon 模型**：CLI 轮询 server（默认 3s）→ 领取任务 → 建隔离工作目录 → spawn agent CLI → 流式回传；心跳 15s
- **无外键硬规则**：关系校验/清理全在应用层，索引必须 `CONCURRENTLY`——PostgreSQL 生产迁移纪律教科书
- **内置 skills**：`server/internal/service/builtin_skills/` 9 个（onboarding/creating-agents/squads/autopilots/working-on-issues 等），agent 干活时自读

### AionUi 架构亮点（源码实证）
- **ACP 协议抽象**（`@agentclientprotocol/sdk`）：统一 Agent 客户端协议，20+ 异构 agent 同一接口接入——全项目最有价值的设计
- **双进程纪律**：Main 无 DOM API / Renderer 无 Node API，只能走 IPC 桥——Electron 防腐化硬约束
- **双轨 agent**：内置引擎（零配置）+ 外部 CLI（自动检测），覆盖新手和老手
- **办公文档链**：OfficeCLI 渲染可编辑 .pptx/.docx/.xlsx

---

## 3. 核心能力矩阵

| 能力 | multica | AionUi | 胜者 |
|------|---------|--------|------|
| 接入 agent 数量 | 22+（含 Hermes/pi/reasonix/dsh/grok/qwen） | 20+（含 Hermes/OpenClaw） | 平 |
| 任务/issue 管理 | ★★★★★ 看板 + issue + assignee + 状态机 | ★★ 会话为主，无 issue 概念 | **multica** |
| 执行可审计性 | ★★★★★ 每次工具调用/命令/错误回放 + token 用量 | ★★★ 对话历史可见，无逐调用审计 | **multica** |
| 审批门 | ★★★★★ 工作落 review 而非 main，人工决定上线 | ★★ 权限弹窗级 | **multica** |
| 定时自动化 | ★★★★★ Autopilots（站会/审计/报告 cron） | ★★★★ Cron 计划任务 | multica 略胜 |
| 内置 agent 引擎 | ★（依赖外部 CLI） | ★★★★★ 零配置开跑 | **AionUi** |
| 办公文档生成 | ✗ | ★★★★★ 21 个助手（PPT/Word/Excel） | **AionUi** |
| 远程/IM 渠道 | ★★★★ Slack/飞书/钉钉/企微 | ★★★★★ WebUI/TG/飞书/钉钉/微信 | AionUi 略胜 |
| 团队协作 | ★★★★★ Squads（agent+人混编，leader 路由） | ★★★★ Team 模式（Leader 拆任务给 Teammate） | multica 略胜 |
| 应用内浏览器 | ✗ | ★★★★★ CDP 桥 + 口令护栏 | **AionUi** |
| 自托管 | ★★★★★ Docker/Helm + 任意 Git host（含 Gitea） | ★★★ web-host 自托管 | **multica** |
| 跨端 | ★★★★★ Web/Desktop/Mobile(iOS 源码) | ★★★★★ Desktop/Web/Mobile | 平 |
| 许可友好度 | ★★ 自定义许可（Apache+条件） | ★★★★★ 纯 Apache-2.0 | **AionUi** |

---

## 4. 与 my-agent-group 生态的关联

| 关联点 | 说明 |
|--------|------|
| **Hermes 被两者支持** | multica runtimes 列表含 `hermes`；AionUi README 明确列出 Hermes Agent——两个工作台都可以当 Hermes 的图形前端 |
| **pi / reasonix / dsh 被 multica 支持** | multica 运行时表格含 Pi、DeepSeek-Reasonix、DeepSeek Harness——与 coder-agent/ 目录 8+ 项目直接呼应 |
| **AionCore 配套** | AionUi 的 Rust 后端已在 `projects/coder-agent/AionCore/`（2026-08-09 加入） |
| **Gitea 集成** | multica 支持 Gitea/Forgejo VCS——与本项目本地 Gitea 基础设施天然契合 |

---

## 5. 结论与选型建议

### 何时选 multica
- 需要**可审计的多 agent 生产流水线**：issue 分配 → 看板跟踪 → 执行日志回放 → 审批门
- 团队场景：多人 + 多 agent 混编（Squads），leader 路由工作
- 要自托管（Docker/Helm）+ 自托管 Git（Gitea）集成
- 关注 token 成本（逐 agent 逐 issue 用量）

### 何时选 AionUi
- 需要**统一对话界面**操控多个 agent，快速切换、远程（手机/IM）查看
- 要开箱即用的内置 agent（不装 CLI 直接跑）+ 办公文档产出
- 个人/小团队轻量使用，无强审计需求

### 两者互补性
- **multica = 任务编排层**（怎么分活、怎么验收），**AionUi = 交互控制层**（怎么聊、怎么遥控）——理论上可组合：AionUi 接 Hermes，multica 也接 Hermes，甚至 multica 的 issue 驱动 + AionUi 的界面可以并存于同一 agent 栈
- 若 my-agent-group 未来做统一 agent 接入层：**ACP 协议抽象**（AionUi 思路）解决"异构 agent 统一接口"，**issue 状态机 + 审计**（multica 思路）解决"工作怎么流转和验收"

---

## 附：评测局限

- 本报告基于 README/CLAUDE.md/AGENTS.md/CLI_AND_DAEMON.md + server 目录结构分析，未逐文件深读全部源码
- multica 处于快速迭代期（最近 commit b5a48eca1，MUL-6168）；AionUi 版本 2.1.45（2026-08-03 分析）
- multica 自定义许可的具体限制需查阅 LICENSE Part I 全文（fork 仅限本地研究，勿二次分发）
