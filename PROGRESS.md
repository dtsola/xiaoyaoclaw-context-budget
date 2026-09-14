---
type: project
status: active
progress: 70
created: 2026-09-14
updated: 2026-09-14
docs:
  - path: docs/DESIGN.md
    desc: 设计文档（问题背景 + 源码取证的机制事实 + 项目口径 9 条 + 三段式流程 + 纯指令式实现 + 路线）
  - path: SKILL.md
    desc: 技能主体 v0.1（口径 + 检测/决策/执行 + 动态取数策略 + 落配置硬规则自检 + 反触发）
  - path: README.md
    desc: 用户视角 README（触发方式 / 三段式流程 / 边界 / 可选建议配置 / FAQ）
  - path: docs/INTERACTION.md
    desc: 用户视角交互流程（含全周期使用流程）
  - path: docs/TRIGGERS.md
    desc: 触发词设计（主触发词「上下文检查」+ 反触发清单 + description 建议文本）
  - path: docs/EVAL-2026-09-14.md
    desc: 首次全流程演练记录（官网 L1 取证 + 冲突 + 执行 + 校验）
  - path: docs/archive/
    desc: 设计原稿（系统视角方案与交互流程设计，仅溯源用）
---

# xiaoyaoclaw-context-budget（OpenClaw Context Budget）

## 目标 / 背景

生态「检查」件：**上下文检查**。解决「模型上下文窗口该配多大」这件事——

- 各厂商窗口差别大（200K / 1M…），OpenClaw 在模型未声明窗口时落回兜底默认 `DEFAULT_CONTEXT_TOKENS = 200000`
- 窗口被低估 → **过早自动摘要、长任务被打断、压缩卡顿**；被高估 → **注意力分散**
- 本技能口径：**有效窗口 = 厂商标称窗口 × 60%**；压缩阈值保持系统默认

**核心设计（用户 2026-09-14 拍板）**
- 流程固定三段：**检测（自动）→ 决策（回一个数字）→ 执行（自动）**
- 只配置**已启用**模型（在用 + 图像/PDF 这类旁路在用）；未在用不动
- 只做窗口；`maxTokens` 等其它参数、压缩阈值**一律不碰**
- **不留痕**：不写审计、不记录修改历史（仅保留改前快照供一步回退）
- **纯指令式**：不用脚本、不维护本地数据文件；厂商标称窗口**运行时动态联网取数**
- 60% 是技能内部默认，**不作为用户输入项**

## 当前状态

**设计定稿 + 技能主体就绪（45%）**。已完成：

1. 机制源码取证（窗口解析链 / 压缩触发公式 / 能力边界：无 per-model、无 per-agent 阈值）
2. 设计文档 + 用户视角流程 + 触发词 + README
3. 实机落地并验证（deepseek 与 minimax/M3 = 600k；压缩阈值回默认；`/status` = `239k/600k`）
4. `SKILL.md` v0.1（**纯指令式**，零脚本、零数据文件）
5. 曾在 `scripts/` 下实现过辅助脚本 + `vendors.json` → **已按用户口径移除**（改运行时动态取数，避免维护成本与运行时依赖）

**待开发**：英文 README → README hero 图 → 发布（GitHub + ClawHub，走全流程确认制）

## 进度日志

- 2026-09-14 11:05–11:16：机制取证 + 方案落地（deepseek 500k→600k、压缩阈值自设 120k）；踩坑记录：`config.patch` 仅就地重载需 restart、`/status` 读会话级缓存需清理、`agent/models.json` 按 agent 运行时刷新
- 2026-09-14 11:53–11:59：官网 L1 取证（DeepSeek 1M；MiniMax M3 1M / M2.x 204,800）+ 全流程演练（发现 M2.x 触发点受全局 reserve 约束的呈现缺陷，已修设计）
- 2026-09-14 12:00：用户口径改定 —— 压缩阈值回默认，只按 60% 设置已启用模型窗口；`minimax/M3` 200000 → 600000，校验通过
- 2026-09-14 12:04–12:10：用户视角流程重写（三段式）、触发词设计（技能名 `xiaoyaoclaw-context-budget`、主触发词「上下文检查」）、README 草稿（可选建议配置）
- 2026-09-14 12:11：确定**不留痕**（去审计/历史）；流程固定为「检测→决策→执行」
- 2026-09-14 12:13：**正式立项**（本项目目录建立，设计文档迁入）
- 2026-09-14 12:2x：辅助脚本一度开发完成（`scripts/ctxbudget.py` + `vendors.json`，5 子命令实测跑通），期间抓到两个真问题：① minimax 模型 id 自带 provider 前缀 → 全限定名重复 ② **`models` 数组在 `config.patch` 中是整体替换**，载荷只带被改模型会误删同 provider 其它模型
- 2026-09-14 12:17：**用户否决脚本与数据文件** —— 理由：技能本质是指令集，用内置工具即可；数据文件需要持续维护、必然过期 → 改为**运行时动态联网取数**
  - 已执行：`scripts/` 整体移出项目（备份于 `tmp/ctxbudget-scripts-removed-20260914/`）；`SKILL.md` 重写「取数策略 / 硬规则自检 / 纯指令式实现」；`DESIGN.md` §十 改写并记录代价
  - 原脚本承担的防错，改为 `SKILL.md` 内**自检清单**（完整数组 + 长度/id 核对 + 数值校验）
  - 进度 55 → **45**（按当前实际完成度：设计定稿 + SKILL v0.1，发布未做）
- 2026-09-14 12:35：**GitHub 发布完成**（用户授权「项目上传 github」）
  - 发布前清理：公开面文案将「指挥官」统一替换为「用户」（7 个 md）；README 头部按生态惯例补齐语言切换 + hero 嵌入 + 徽章（license / ClawHub 链接）；`audit_readme.py` 静态审计通过
  - 建仓：`dtsola/xiaoyaoclaw-context-budget`（public / main / 7 topics / 中文 description，走 Python 写 API 避开 PS 的 GBK 坑）
  - 推送：`dcf3f9c4` → main（走代理 `127.0.0.1:22307`，一次成功）
  - 远端核验：根目录 5 文件 + `assets/`、`docs/`；`assets/readme/hero.svg` 4292 B ✅
  - 项目进度 45 → **70**（剩：ClawHub 发布，按全流程确认制需用户确认后提交）
- 2026-09-14 13:1x–13:2x：**ClawHub 已提交 v1.0.0 并公开**（用户手动公开）；但线上安全扫描判定 **ClawScan = suspicious**（静态扫描 clean；LLM 复核 + AIG 1 项 finding）→ 按意见收窄权限（**v1.0.1 待重发**）
  - 报告来源：`clawhub scan download xiaoyaoclaw-context-budget --version 1.0.0` → zip 内 `clawscan.json` / `static-analysis.json` / `skillspector.json`
  - 核心意见：① 清会话缓存未限定范围（Medium finding）② 行为超出声明（重启 + 清会话状态 = `environment_proportionality: concern`）③ 触发面过宽（mutation-capable 技能）④ 无审计被判削弱可追责性 ⑤ hero.svg 注释被判"隐藏指令"（降噪项）
  - **v1.0.1 修改**：SKILL.md 新增「权限与写操作声明（权限透明）」8 行表；执行第 4/5 步改为「只提示不擅自执行 / 会话状态默认不动 + 限定当前会话 `contextTokens` + 先备份 + 需确认」；新增「触发分级（读 / 写分离）」；不留痕表述统一（不写审计/历史；仅记本次旧值；用户可要求变更摘要）；README 中英同步（写操作透明 / 联网只取数 / 不做自动化）；hero.svg **去掉全部注释**（组 id 已表意）并把版本号改为 v1.0.1；`docs/README-DRAFT.md` 移入 `docs/archive/`；DESIGN/INTERACTION/EVAL 同步口径（EVAL 加"历史记录"声明）
  - 校验：`audit_readme.py` 通过（2 张本地图）；`visual_verify.py` 全绿
- 2026-09-14 13:47–13:59：**v1.0.2 扫描结果（仍 suspicious，已公开 latest=1.0.2）→ 出 v1.0.3 并提交**
  - v1.0.2 意见：*"documentation mixes local configuration writes with broad or automatic triggers"* + *"one rollback path contradicts its no-local-files promise"*
  - **根因**：子指令写作「撤销**上次**上下文调整」= 暗示跨会话留档，与「记录仅会话内、不落盘」自相矛盾
  - **v1.0.3 改动**：① 「撤销上次」→「**撤销本次调整**」（并注明仅同会话内有效）② **去自动化语义** —— 流程统一为「检测（只读）→ 决策（回一个数字）→ 执行（确认后写入）」，README 明示「**本技能不会自动运行**」③ 可选 cron 段落标题改为「可选，需你自行配置；技能本身不会自动运行」④ **触发词收窄**为精确意图清单 + 明确「不要因泛泛谈到 context / memory / token 就激活」
  - 提交：v1.0.3（`source-commit 965fd02`）；挂了 14:12 的一次性自动检查
- 2026-09-14 13:36：**v1.0.1 扫描结果 = 仍 suspicious，但收窄到一条自相矛盾** → 出 **v1.0.2**
  - 扫描原文：*"it also permits session-storage edits and backups despite claiming it writes no files"*
  - 根因：权限表「绝不写 = 不写任何文件」与「会话状态可编辑 + 先备份」互相打架
  - **v1.0.2 修改**：① **删除"改会话存储"能力** —— 会话状态行改为「不处理、不修改任何会话存储；只解释显示滞后、由用户自行处置」② 执行第 5 步同步改为「不处理」（不改动即无失败面）③「绝不写」精确化：除窗口字段外不写任何配置项、不创建定时任务、**不写任何本地文件（含不生成备份/审计文件）** ④ 回退记录明确为「**仅本次会话内**记录旧值，不落盘、不写文件」⑤ README 中英 / DESIGN / INTERACTION 同步（去掉"备份/快照"字样）⑥ `docs/EVAL-2026-09-14.md` 移入 `docs/archive/`（历史记录，避免旧版行为描述被当作现行能力）⑦ hero.svg 版本号 → v1.0.2
- 2026-09-14 12:5x：**本地全局安装完成**（用户要求「本地全局安装一下，我来测试」）
  - 安装路径：`state/skills/xiaoyaoclaw-context-budget/SKILL.md`（与项目同源）
  - 关键发现：**技能是白名单制** —— 每个 agent 的 `agents.list[].skills` 数组 = 允许清单，与全局技能目录逐条对应（29 项）；因此"全局安装" = ①拷文件到 `state/skills/` ②把技能名加进各 agent 白名单（缺任一步都不可用）
  - 执行：`config.patch` 一次性重写 `agents.list`（载荷基于实况配置生成后追加，保留各 agent 原有数组，含 xiaogang/xiaoguang 的 workspace 私有技能）→ 7 个 agent 全部含本技能（30/31/40/32/30/30/30 项）
  - 待用户实测；测试建议：说「上下文优化」；想看差异链路用「比例 50%」再「撤销本次调整」


- 2026-09-14 12:31–12:34：**英文 README**（`README.en.md`，与中文版 1:1 对齐）+ **hero 图**（`assets/readme/hero.svg`，纯 SVG，1200×360）
  - hero 由 `xiaoyaoclaw-beautify-github-readme` 技能生产：走它的「确认模式 → 勘察（参照同系列 memory-distill hero 视觉语言）→ 确认实现方式（纯 SVG）→ 生产 → 渲染级校验」流程
  - 校验结果：`visual_verify.py` **全绿**（无静态问题 / 无对比度问题 / 无边缘贴边）；视觉模型复核判定「可发布」，中文渲染正常、无裁切
  - 途中修掉一个误报：校验器按「文字所属组内最近填充矩形」判定背景，容量条与其后文字同组 → 被误判为文字背景；已将容量条移入独立分组置于末尾（视觉不变，报告干净）
  - 设计取舍：沿用生态视觉语言（1200×360 深色卡 + 左标题块 + 右终端面板），但用本技能独有的「**60% 容量条**」作为母题
- 2026-09-14 12:2x：修 PROGRESS.md 编码事故（PS 5.1 `Get-Content`/`Set-Content` 中文往返导致双重编码 + 写入 BOM）→ 已重建，教训记入工作区日志
- 2026-09-14 12:21：**用户追加通用性要求** —— SKILL.md 需同时支持 **OpenClaw 与 小遥Claw**；模型必须**运行时动态读取**，禁止写死本机模型/数值
  - `SKILL.md` 重写：新增「通用性要求（硬约束）」表（不硬编码路径 / 模型名 / 窗口数值 / 运行态数值 / 生效机制；示例仅作示例）；配置读写改为一律走 `gateway config.get` / `config.patch`；取数改为「运行时检索策略」而非内置厂商 URL 表；决策卡与回执模板全部占位符化
  - `DESIGN.md` 新增 §三之二「通用性要求」；§八 注明为**勘察期实例记录**（非技能内置）
  - `README.md` 首屏去掉平台宣言（保留「示例仅为示例」注记）；同一轮还把 SKILL.md 正文与 frontmatter 里的平台宣言一并删除（用户 12:27 确认）

## 文档索引

| 文档 | 说明 | 更新 |
|------|------|------|
| docs/DESIGN.md | 设计文档（机制事实 + 9 条口径 + 三段式流程 + 纯指令式实现 + 路线） | 2026-09-14 |
| SKILL.md | 技能主体 v0.1（纯指令式） | 2026-09-14 |
| README.md | 用户视角 README | 2026-09-14 |
| docs/INTERACTION.md | 用户视角交互流程 + 全周期使用流程 | 2026-09-14 |
| docs/TRIGGERS.md | 触发词 + 反触发清单 | 2026-09-14 |
| docs/EVAL-2026-09-14.md | 首次全流程演练记录 | 2026-09-14 |
| docs/archive/ | 设计原稿（仅溯源） | 2026-09-14 |

<!--
使用说明（agent 维护，用户可忽略）：
- status: active | paused | archived
- progress: 0-100，时刻维护（每次更新进度日志时同步调整）
- 进度日志只追加不删除
- 重要文档：移入 docs/ 或记录路径，追加到 docs 数组（机器可读）+ 本表格（人可读）
- 项目完结：status → archived + 关键结论记入 MEMORY.md（供 memory-distill 蒸馏）
-->
