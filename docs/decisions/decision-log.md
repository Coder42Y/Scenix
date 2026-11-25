# Scenix 决策日志

更新：2026-11-25

| # | 议题 | 决策 |
|---|------|------|
| 1 | 核心用户 | 非技术知识工作者的通用 agent |
| 2 | Runtime 来源 | 启动时**自动探测**本地已装 CLI（类 multica），多 provider 并列；每 session 可切 model + thinking effort |
| 3 | 冷启动 | ~~引导安装 CLI~~ → 被 #10/#11 取代：MVP 内置 Pi，冷启动只需引导配置模型凭据；探测其他 CLI 是后续能力 |
| 4 | 权限 | 项目绑定工作文件夹为边界；CLI 权限请求翻译成「人话审批卡片」（允许一次/始终/拒绝） |
| 5 | UI 形态 | 聊天内联 + 可展开到侧边 Canvas（持久化） |
| 6 | MVP hero 场景 | 数据分析 + 可交互仪表盘 |
| 7 | 图表 | 自定义 catalog：Chart 组件包 Vega-Lite spec（vega-embed 渲染） |
| 8 | 终局定位 | 个人本地 agent 工作台 + 可扩展 catalog 生态 |
| 8b | 数据执行环境 | App 托管预装数据 runtime（uv + pandas/duckdb），以 Pi extension 工具暴露 |
| 9 | 产品取向（修正） | **不追求轻量**：配齐必要环境，macOS/Windows/Linux 都要「完美跑通」，可接受大安装包 |
| 10 | MVP runtime（修正） | **内置 Pi**（badlogic/earendil-works pi，MIT，有 TS SDK + RPC 模式 + extension 自定义工具）；Claude Code/Codex 等后移 |
| 11 | Runtime 分发（修正） | **Pi 随 app 内置安装，用户不做选择**；runtime 独立于 app 持续自动更新 |
| 12 | Runtime 更新策略 | 独立更新通道（不等 app 发版）+ 每版先过 Scenix 兼容性测试再放行 + 用户端启动自检失败自动回滚上一好版本 |
| 13 | 模型凭据 | 订阅登录 + API key 双轨，首屏引导式配置；凭据仅存本机系统 keychain，Scenix 不自建后端；可切 model + thinking effort |
| 14 | 桌面壳 | Electron + React（早先已选；与 #9「不追求轻量、全平台一致」相符，内置 Chromium 保证渲染一致） |
| 15 | MVP 平台 | macOS + Windows 同时首发；Linux 后置 |
| 16 | MVP 验收 | 非技术用户**零指导**跑通一次完整分析：安装→配模型→丢 CSV→得到可筛选仪表盘→Canvas 调整并保存；macOS 与 Windows 一致 |
| 17 | 数据出境 | 默认只发 schema + 少量样本行 + 聚合查询结果给模型；全量原始数据留本地 duckdb；UI 提供「已发给模型的内容」面板，用户可关闭样本行或手动放开全量 |
| 18 | 持久化对象 | 「活面板」：保存 A2UI surface（组件树 + dataModel）+ 数据源引用 + 查询定义；数据更新后点刷新即可重算，无需再找 agent |
| 19 | MVP non-goals | ❌ 其他 CLI runtime 探测 ❌ 团队协作 / 账号 / 云同步 ❌ 数据库 / API 数据源（仅本地 CSV / Excel）❌ Catalog 市场 / 模板分享导入（内部留扩展口，不对外）。Linux 沿用 #15 后置。 |
| 20 | 会话模型（修正） | **单一 agent runtime（Pi）+ 多 session 并行进 MVP**：不同项目/工作文件夹的会话可同时运行、互不阻塞，可回看与切换。❌ 多 agent 协作 / 编排不在产品范围内 |
| 21 | 后台 session 提醒 | 侧栏状态徽标（运行 / 等审批 / 完成 / 报错）+ 系统通知；后台审批请求暂停该 session 等用户确认，**永不因在后台而自动放行** |

## Pi 核实结果（2026-11-25）
- 集成方式：SDK（进程内，首选）/ RPC（子进程 JSONL）。来源：[README](https://raw.githubusercontent.com/earendil-works/pi/HEAD/README.md)
- 自定义工具走 **extension**，**无官方 MCP** → A2UI 的 `a2ui_render` 与数据 runtime 工具应写成 Pi extension 工具，不走 MCP server。
- npm 两个 scope：`@mariozechner/pi-coding-agent` 停在 0.73.1，`@earendil-works/pi-coding-agent` 已 1.6.0（MIT）→ 应选后者。
- **已验证**：SDK 有 `session.model` / `session.thinkingLevel`；RPC 有 `set_model` / `set_thinking_level` / `get_available_thinking_levels`；会话默认持久化（`SessionManager`，支持分支树）；有 OAuth + API key 凭据示例。

## 待决
- 并发上限（暂定默认值：同时运行 3 个，其余排队；PRD 里标注为可调，待 spike 实测内存后定）
