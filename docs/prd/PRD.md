# Scenix 产品需求文档（PRD）

> 状态：需求对齐完成，待评审｜日期：2026-11-25
> 可视化版本：[prd.html](prd.html)｜决策原始记录：[../decisions/decision-log.md](../decisions/decision-log.md)

## 1. 一句话

Scenix 是一个桌面工作台：内置 Pi 作为 agent runtime，把 agent 的输出渲染成原生可交互界面（基于 Google A2UI 协议），而不是一段文字。

MVP 面向不写代码的知识工作者，用一件事证明价值：**丢进一份 CSV，零指导得到一个可筛选、可保存、可刷新的仪表盘。**

## 2. 目标用户与场景

核心用户是非技术的知识工作者（运营、分析、研究）。他们没有 Python，没有 API key，也不会装命令行工具，所以所有环境必须内置并自动配好。

| 场景 | 主要用到的界面 | 阶段 |
|---|---|---|
| 数据分析与可交互仪表盘 | 图表、筛选控件、数据表、Canvas 活面板 | MVP 主场景 |
| 文件批量整理，先审批再执行 | 操作清单表格、勾选、审批卡片 | MVP 之后 |
| 调研写作，用表单补全需求 | 表单、选择器、进度面板 | MVP 之后 |

## 3. 目标

**MVP 目标**：非技术用户零指导，在 macOS 与 Windows 上跑通完整分析路径（见第 7 节验收标准）。

**终局目标**：个人本地 agent 工作台，加可扩展的 catalog 生态（仪表盘、审批、表单都可成为可复用的界面组件）。MVP 只留扩展口，不对外开放。

## 4. 架构概要

```
渲染进程（React）                主进程（Electron）                  外部
  聊天流                           会话管理（多 session 并行）           模型 provider（用户自带凭据）
  A2UI 渲染器（含 Chart）   <──>    Pi runtime（内置，每 session 一个）   数据 runtime（uv + pandas + duckdb）
  Canvas 活面板                      ├ a2ui_render 工具（校验后推给界面）   工作文件夹（CSV、Excel）
  审批卡片                           ├ 数据查询工具
  会话侧栏                           └ 权限桥（权限请求 → 人话卡片）
                                  独立更新通道（兼容测试 + 自动回滚）
```

要点：

- 界面由 agent 的工具调用产生，用户的每次操作又作为消息回到同一个 session，所以筛选和调整不需要重新开始对话。
- Pi 没有官方 MCP，A2UI 与数据能力以 **Pi extension 工具**接入，不起 MCP server。
- 图表是自定义 catalog：Chart 组件的 props 是 Vega-Lite spec。A2UI basic catalog 没有图表组件。

## 5. 功能范围

### MVP 要做

- 内置 Pi 与独立自动更新通道
- 订阅登录与 API key 的引导式配置，可切换 model 与 thinking effort
- 聊天内联 A2UI 加侧边 Canvas
- Chart 自定义 catalog（Vega-Lite）
- 托管的本地数据 runtime
- 人话审批卡片与工作文件夹边界
- 活面板的保存与刷新
- 多 session 并行与后台提醒
- macOS 与 Windows 同时首发

### MVP 明确不做

| 不做 | 原因 |
|---|---|
| 其他 CLI runtime 探测（Claude Code、Codex 等） | MVP 只有内置 Pi，先把一条路径做到零指导可用 |
| 团队协作、账号、云同步 | 定位是个人本地工作台，不自建后端 |
| 数据库、在线 API 数据源 | 只支持本地 CSV 和 Excel |
| catalog 市场、模板分享 | 内部留扩展口，不对外开放 |
| 多 agent 协作与编排 | 不属于本产品范围；多 session 并行是单 runtime 的并行 |
| Linux | 后置 |

## 6. 关键产品规则

- **Runtime 与更新**：Pi 随 app 内置，用户不做选择。runtime 走独立更新通道，每个新版先过 Scenix 兼容测试再放行；用户端启动自检失败，自动回滚到上一个好版本。
- **模型凭据**：订阅登录与 API key 双轨，首屏引导配置。凭据只存本机系统 keychain，Scenix 不自建后端。
- **权限**：每个项目绑定一个工作文件夹作为边界。写文件、跑命令、联网等请求变成人话卡片，选项为允许一次、始终允许、拒绝。
- **数据隐私**：全量数据留在本地。模型默认只收到 schema、少量样本行和聚合结果。界面有「已发给模型的内容」面板，用户可关掉样本行，或手动放开全量。
- **活面板**：保存的是组件树、数据模型、数据源引用和查询定义。数据更新后点刷新即可重算，不用再找 agent。
- **多 session**：不同项目的会话同时运行、互不阻塞，暂定同时运行 3 个，其余排队。后台 session 用侧栏徽标和系统通知提醒；审批永不因在后台而自动放行。
- **产品取向**：不追求轻量。配齐必要环境，各平台一致、完美跑通，可接受较大的安装包。桌面壳用 Electron + React。

## 7. 验收标准

一个不写代码的人，零指导，在 macOS 和 Windows 上都能走完下面的路径，且两个平台表现一致：

1. 安装应用，首屏引导完成模型配置
2. 把一份 CSV 拖进来，用一句话提出分析要求
3. 得到带图表和筛选控件的仪表盘，筛选能联动
4. 展开到 Canvas 调整布局并保存
5. 换一份更新后的 CSV，点刷新得到新结果

## 8. 风险与待验证

| 待验证项 | 为什么重要 | 怎么验证 |
|---|---|---|
| Pi extension 能否把结构化 A2UI 消息实时推到界面 | 整个渲染闭环的前提 | 最小 extension 在 Electron 里端到端跑通 |
| 同一进程内多个 Pi 会话能否并行且互相隔离 | 决定用进程内 SDK 还是每 session 一个子进程，也决定并发上限 | 并发起多个会话，实测内存与隔离性 |
| A2UI 渲染器与自定义 Chart 组件的契合度 | 仪表盘是主场景 | 用 Vega-Lite 做一个自定义组件，验证与数据模型联动 |
| 模型生成合法 A2UI 的成功率 | 决定校验与自修的投入 | 用 5 到 10 份真实数据集做基准 |
| 内置数据 runtime 的跨平台打包 | 非技术用户不能有任何环境步骤 | 干净的 Windows 与 macOS 机器上安装验证，含签名与公证 |
| Pi 上游变动 | 包名已换过 scope，版本从 0.x 跳到 1.x | 兼容测试门控更新，固定使用新 scope 的包 |

## 9. 推进顺序

1. 三个 spike：extension 推送、多会话并行、Chart 自定义组件
2. 渲染闭环：单 session 的对话、A2UI 内联渲染、操作回传
3. 数据 runtime 与 Chart，跑通 hero 场景
4. 活面板、多 session、审批卡片与后台提醒
5. 更新通道、跨平台打包与零指导验收

## 10. 参考

- A2UI 规范与 SDK：<https://a2ui.org>、<https://github.com/a2ui-project/a2ui>（Apache-2.0）
- Pi：<https://github.com/earendil-works/pi>（npm 包 `@earendil-works/pi-coding-agent`）
- multica（自动探测本地 runtime 的参照）：<https://github.com/multica-ai/multica>
