# 交接说明（2026-11-19）

Minglu 换 Claude 账号前写的。新账号的 Claude 从这里开始读，然后读 `CLAUDE.md` 和 `README.md`。

## 1. 先做这几件事

1. **确认 GitHub 能访问本仓库。** 仓库现在是 `minglu-prime/ibkr-agent`（旧名 `Sam18069272581/ibkr-agent`，GitHub 会自动跳转）。新账号要在 claude.ai 里连上能访问这个仓库的 GitHub 账号，并把仓库加进项目。
2. **读最新分支，不要读 `main`。** `main` 只有一个初始提交（只有 `.gitignore`）。所有代码都在三个叠起来、还没合并的草稿 PR 里：

   | PR | 分支 | 内容 | 叠在 |
   | --- | --- | --- | --- |
   | #1 | `claude/scaffold-agent` | 美股 IBKR agent 骨架、模拟盘闭环、Windows 支持 | `main` |
   | #2 | `claude/project-thread-hkch6v` | 币圈 OKX 现货、一年持有期、Opus 每周边界 + Haiku 快哨兵 | #1 |
   | #3 | `claude/project-thread-ewswv6` | 防御性修复（OKX 手续费币种等），已合并 #2 的最新提交 | #2 |

   最新、最完整的代码在 `claude/project-thread-ewswv6`（`dfb0167`）。建议 Minglu 按 #1 → #2 → #3 的顺序合并进 `main`，以后新账号直接从 `main` 开始。
3. **把项目说明重新填上。** 新账号的项目说明（project instructions）是空的，把下面“质量规则”那段原样贴进去。
4. **确认 Anthropic API key 还能用。** 本地 `.env` 里的 `ANTHROPIC_API_KEY` 是 Minglu 自己填的。如果它属于旧账号的组织，换账号后可能被停用，需要换一个新的 key（同样只填在 `.env` 里）。

## 2. 用户偏好和规则

- 所有回复用中文。
- 质量规则（Minglu 2026-11-05 定的，也写在 `CLAUDE.md`）：
  - 禁止用空 catch 或静默 fallback 掩盖错误；错误必须显示或记录日志
  - 禁止使用 mock/假数据/写死数值，除非我明确要求
  - 每个功能必须处理：加载中、空数据、错误三种状态
  - 改完代码必须运行 lint + 测试，全部通过才能说"完成"
  - 修 bug 时先写一个能复现该 bug 的失败测试，再修复
  - 修复必须说明根本原因，不接受只消除症状的补丁
  - 新增逻辑前先搜索是否已有类似函数，禁止重复实现
  - 完成后列出：你做的假设、没处理的边界情况
- 目前的理解（Minglu 没明确确认过）：单元测试里的假交易所、假 Claude 客户端是测试替身，不算“假数据”。演示页必须保留“演示 · 模拟数据 · 非真实交易”标签。
- 不经 Minglu 明确同意：不下真单，不改 `mode: live`，不改 `dry_run: false`，不设 `CRYPTO_ALLOW_LIVE=yes`。

## 3. 设计文档

原来只在旧账号里的文档，已导出到仓库：

- `docs/design/ibkr-architecture.md`：美股 agent 最初的架构设计（2026-10-29），以及按小资金调整后的方案。
- `docs/design/crypto-migration.md`：币圈迁移方案（2026-11-05），包括借鉴的开源项目和 X 帖子、不借鉴的做法、价格变化怎么处理。
- `docs/crypto-desk/trading-desk.html`：交易流程可视化页“理性交易台”（像素办公室里的 Q 版小人演每笔交易）。现在用的是**演示数据**，还没读真实日志。小人是原创形象，不是 Claude 吉祥物，因为 Minglu 想发到网上。

旧账号里这几份文档的在线链接（claude.ai/artifact/...）换账号后打不开，以仓库里的为准。

## 4. Minglu 的电脑（Windows 11，DESKTOP-3G27JBR）

- 项目目录 `C:\Users\COLORFUL\ibkr-agent`，2026-11-05 新 clone 的，虚拟环境 `.venv`（Python 3.11.9）。
- **本地落后于 GitHub。** 2026-11-19 检查：本地分支 `claude/project-thread-hkch6v` 停在 `7f2b4c0`，没有没推送的提交，也没有改过的代码文件。下次在那台电脑上跑之前先 `git fetch`，切到 `claude/project-thread-ewswv6`（或合并后的 `main`）。
- 只在本地、不在 GitHub 的文件（都在 `.gitignore` 里，换 Claude 账号不影响它们）：
  - `.env`：Minglu 自己填的密钥。**永远不要读取、打印或提交。**
  - `config\settings.yaml`：从模板复制，只开了 `crypto.enabled` 和 `crypto.paper_local`，没有 `crypto.ai` 段。
  - `data\journal.sqlite`、`data\crypto_paper.json`：主日志和本地模拟账户。
  - `data\ai-test\`：AI 测试专用的配置和日志。`settings.yaml` 是主配置的副本，`journal_path` 指向 `data/ai-test/journal.sqlite`，开了 `crypto.ai` 的 `envelope_enabled`、`gate_enabled`，`monthly_budget_usd: 2`。用法：`.venv\Scripts\ibkr-agent --config data/ai-test/settings.yaml crypto ...`（`--config` 写在 `crypto` 前面）。主配置和主日志不要动。
- 那台电脑上 Claude Code 的自动模式不允许 `pip install`，安装依赖要 Minglu 自己跑。`ruff` 可能还没装。
- `D:\claude\ibkr-trader` 是另一个无关项目，不要动。Minglu 原来的美股项目文件夹在哪里不清楚。
- IB Gateway 跑在 Docker Desktop 里，跑美股前 Docker 要开着。电脑每周二 16:30（CET）开机；每周任务脚本 `deploy\install_task_windows.ps1` 还没装。
- OKX 公开行情用默认域名就能连。

## 5. 现在做到哪了

### 美股（状态停在 2026-10-29，之后没再跑过）

- 网关能登录模拟账户，程序能读到账户。模拟盘余额已重置，净值约 $3,468。`dry_run: true`，没下过单。
- **未解决：IBKR Error 10197**（所有股票拿不到行情）。原因是模拟盘共享真实账户的行情，而真实账户在别处有登录。推荐的根本解决：在 Client Portal 里关掉“与模拟账户共享实时行情”。代码已有后备：拿不到快照就用历史日线收盘价。
- 下一步见 `CLAUDE.md` 的“下一步”。

### 币圈（状态停在 2026-11-05）

- 决定：OKX（币安在欧盟没有 MiCA 牌照）、本金 €300、回撤熔断 35%、核心仓 BTC/ETH、分 4 周建仓。
- 第二阶段（Opus 每周边界 + Haiku 快哨兵）在 `7f2b4c0` 时用真 Claude API 在 `data\ai-test` 里跑过一次：生成了 Opus 边界，快哨兵放行 4 次，花了约 $0.05。之后加了分批建仓和质量修复（`e25d4f6`、PR #3），**还没重跑**。
- PR #3 的主要发现：OKX 买入手续费按买到的币扣，不是欧元。已修。
- 下一步：在 Minglu 电脑上更新代码后重跑一次 `data\ai-test`；然后让可视化页读真实日志，替换演示数据。

## 6. 否决过的想法（不要再提）

- 炒 meme 币、每笔 8% 仓位、高频、永续合约、杠杆。
- 用 70% 胜率作为转实盘门槛（Minglu 选了“稳定性优先”的门槛，见 README）。
- 拿别人无法核实的单次战绩当依据。

## 7. 没弄清的事

- Minglu 提到过“马维斯”这个参考，一直没弄清指的是什么。
- 本地 `.env` 里的 Anthropic key 属于哪个账号，换账号后是否还能用。
