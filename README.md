# Scenix

把 agent 的输出变成原生可交互界面的桌面工作台。

Scenix 内置 [Pi](https://github.com/earendil-works/pi) 作为 agent runtime，使用 Google 开源的 [A2UI](https://a2ui.org) 协议，让 agent 输出的不只是文字，而是图表、表单、仪表盘和审批卡片，由 Scenix 用原生组件渲染。

> **状态：需求阶段。** 目前还没有代码，仓库里是已对齐的产品需求与决策记录。

## 第一个版本要做到什么

面向不写代码的知识工作者：**丢进一份 CSV，零指导得到一个可筛选、可保存、可刷新的仪表盘**，并且在 macOS 与 Windows 上表现一致。

## 核心设计

- **内置 runtime**：Pi 随 app 安装，用户不用装任何命令行工具；runtime 走独立更新通道，先过兼容测试再放行，失败自动回滚
- **原生界面**：agent 通过工具调用输出 A2UI，聊天内联渲染，也可展开到持久化的侧边 Canvas
- **数据留在本地**：全量数据在本地 duckdb；模型默认只收到 schema、少量样本行和聚合结果
- **人话审批**：权限请求翻译成人话卡片，每个项目绑定一个工作文件夹作为边界
- **多 session 并行**：不同项目的会话同时运行，后台提醒，审批永不自动放行

## 文档

| 文档 | 说明 |
|---|---|
| [docs/prd/PRD.md](docs/prd/PRD.md) | 产品需求文档（Markdown，评审与修改以此为准） |
| [docs/prd/prd.html](docs/prd/prd.html) | PRD 的可视化版本，含架构图与时序图，下载后双击打开 |
| [docs/decisions/decision-log.md](docs/decisions/decision-log.md) | 21 项已定决策的原始记录与 Pi 调研结果 |

## 目录结构

```
Scenix/
├── README.md
├── docs/
│   ├── prd/                 产品需求
│   │   ├── PRD.md             Markdown 正文（主）
│   │   ├── prd.src.html       可视化版本源文件
│   │   └── prd.html           可视化版本（由 prd.src.html 构建，勿手改）
│   └── decisions/
│       └── decision-log.md    决策日志
└── （代码目录待定，spike 完成后再确定）
```

## 下一步

先做三个 spike，验证后再开始搭建工程：

1. Pi extension 能否把结构化 A2UI 消息实时推到界面
2. 同一进程内多个 Pi 会话能否并行且互相隔离
3. 基于 Vega-Lite 的 Chart 自定义组件能否与 A2UI 数据模型联动

## 许可

暂未确定。
