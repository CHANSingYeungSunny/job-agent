# 《找工作 Agent 调研报告》

> **Executive Summary（3 句话）**
> 本 Agent 是 "AI Native Community 平台" 的首个试点，目标为自动聚合香港/深圳两地 AI Application、AI Agent、Deep Learning、Reinforcement Learning 四类技术岗并智能筛选匹配，最终封装为 MCP Server 供平台内其他 agent 调用。调研结论显示：免费/半开放数据源（Adzuna、Arbeitnow、Greenhouse/Lever 等 ATS 公开板、香港劳工处开放数据）足以支撑 MVP，而深圳大陆平台（BOSS/拉勾/猎聘）强反爬且无官方 API，应暂缓。建议 MVP 锚定 A2–A3（半自动 SOP + 端到端抓取筛选），按"HK + Global Remote 切入 → 规模化 → 向 A4/A5 自进化靠拢"的路线推进。

---

## 1. 背景与界定

### 1.1 项目定位（三层拆解标准）
平台对所有 agent 采用统一三层拆解：
- **Interface Layer**：对外暴露形态——Web 界面 / API / MCP / 人工触发。
- **Action Layer**：功能逻辑、代码、prompt（本 Agent 的抓取、筛选、打标、通知逻辑）。
- **Knowledge/Doc Layer**：架构设计、PRD、UI/UX、代码仓库位置等沉淀。

本 Agent 的主接口形态为 **MCP Server**（供平台内其他 agent 调用），辅以一次性 CLI/Web 预览。

### 1.2 Agent 成熟度模型（A0–A5）
- A0 手动；A1 半自动（人触发脚本）；A2 有 SOP 的半自动；A3 端到端自动化；A4 自主调用工具完成任务；A5 自进化。
- 本 Agent 目标：MVP 达 A2–A3，中长期向 A4（自主调度+自主投递辅助）演进，A5 为候选（基于反馈自我优化筛选策略）。

### 1.3 调研范围与硬约束
- 地域：香港、深圳（并重）。
- 技术方向：AI Application / AI Agent / Deep Learning / Reinforcement Learning。
- 必覆盖模块：竞品分析、免费 API 数据源、开源项目、招聘平台数据源与合规风险、技术方案建议。
- 事实分档约定：文中以 **〔事实〕** / **〔推断〕** / **〔待验证〕** 标注结论来源可靠度。

---

## 2. 市场与需求（HK vs 深圳）

### 2.1 四方向职位量 / 薪资区间 / 招聘公司类型
> 量化说明：公开平台精确职位量为动态值，以下采用核验快照 + 行业报告区间，标〔待验证〕的为非精确推断。

| 方向 | 香港（HK） | 深圳（mainland） | 招聘公司类型 |
|------|-----------|------------------|--------------|
| AI Agent | 量中等〔待验证〕；JobsDB 有 "Agent RL Engineer" 岗〔事实，headlinejobs.hk〕 | 量大，BAT/字节/创业公司密集〔推断〕 | HK：金融科技/AI 初创；深圳：大厂 AI Lab、AI 应用创业 |
| AI Application | 中等〔待验证〕 | 量大（AI 原生应用爆发）〔推断〕 | HK：企业 SaaS；深圳：AI 应用/工具类初创 |
| Deep Learning | CTgoodjobs 有 "Deep Learning Quant Researcher"（GPU 集群）〔事实〕 | 量大，月薪 ¥20–50K 占 70.6%（543 样本，职友集 2026-06-11）〔事实〕 | HK：量化/研究；深圳：大模型/算法公司 |
| Reinforcement Learning | JobsDB HK 显示 936 个 RL 岗（2026-05 快照）〔事实，动态〕 | 量少但高薪〔推断〕 | HK：量化/AGI 研究；深圳：RLHF/对齐团队 |

**薪资区间（核验）**：
- 香港 Deep Learning Engineer 毛薪约 HK$809,702/年（SalaryExpert，〔事实〕）；用户底线 HK$40k/月即约 HK$480k/年，处于市场中低位。
- 深圳 AI 算法 70.6% 岗位 ¥20–50K/月（职友集，〔事实〕）；大模型算法均值约 ¥71K/月（第三方报道，〔待验证〕）。用户底线 RMB 25k/月处于主流区间。

### 2.2 招聘平台清单与评估
| 平台 | 地域 | 官方 API | 爬取难度 | 反爬机制 | ToS 风险 |
|------|------|----------|----------|----------|----------|
| JobsDB | HK | 无公开 API〔事实〕 | 中高〔推断〕 | 2026 反爬升级（代理池/行为检测）〔事实〕 | 中（爬取违反 ToS） |
| CTgoodjobs | HK | 无〔事实〕 | 中〔推断〕 | 常规〔待验证〕 | 中 |
| LinkedIn | 全球/HK | 有合规招聘 API（企业侧）〔事实〕 | 极高（严格反爬） | 强〔事实〕 | 高（禁止自动化） |
| 香港劳工处 | HK | **有开放数据**〔事实〕 | 低（开放数据集） | 弱〔推断〕 | 低（政府开放授权） |
| BOSS直聘 | 深圳 | 无官方开放 API〔事实〕 | 极高 | SPA/动态 DOM/行为验证〔事实〕 | 高 |
| 拉勾 | 深圳 | 无〔推断〕 | 高〔推断〕 | 强〔待验证〕 | 高 |
| 猎聘 | 深圳 | 无〔推断〕 | 高〔推断〕 | 强〔待验证〕 | 高 |
| 智联/51job | 深圳 | 无〔推断〕 | 高〔推断〕 | 强〔待验证〕 | 高 |

---

## 3. 竞品分析

### 3.1 AI-native 求职产品对比
| 产品 | 定位 | 核心功能 | 数据源 | 收费 | 目标人群 | HK/深圳适用 | 优劣势 |
|------|------|----------|--------|------|----------|-------------|--------|
| Jobright | AI 求职 copilot | 匹配+自动投递+简历定制 | 多 ATS | Free/Turbo（价未公开）〔事实〕 | 北美求职者 | 弱（偏美） | 优：全功能；劣：无 HK/CN 源 |
| Simplify | 一键投递/自动填表 | 表单自动填充 | 主流 board | Freemium〔推断〕 | 欧美 | 弱 | 优：省力；劣：地域错配 |
| Teal | 求职追踪+简历 | CRM+简历优化 | 手动导入 | Freemium | 通用 | 中 | 优：追踪强 |
| Huntr | 看板追踪 | 看板/状态管理 | 手动 | Freemium | 通用 | 中 | 轻量 |
| Careerflow | AI 求职助理 | 匹配+信函+追踪 | 多源 | 订阅〔推断〕 | 通用 | 弱 | 功能全 |
| LoopCV | AI 投递+测评 | 自动投递+竞品库 | 多源 | 订阅〔事实〕 | 欧洲 | 弱 | 含评测生态 |
| JobCopilot | 自动投递 SaaS | 代投 | 企业 career | ~$28/月起〔事实〕 | 效率型 | 弱 | 优：省心；劣：ghost job 风险 |
| Jobscan | ATS 简历优化 | 匹配度打分 | 上传 JD | 订阅〔推断〕 | 通用 | 中 | 优：JD 对齐 |

### 3.2 平台内嵌 AI 功能（替代性竞品）〔事实〕
- **BOSS 直聘 DeepHire**（2026）：岗位咨询、薪资查询、简历优化、AI 代聊约面（aihub.cn）。
- **LinkedIn AI 匹配**：基于画像的职位推荐与 InMail 辅助。
- 启示：大平台已内嵌"匹配+优化"，本 Agent 的差异化应在 **跨平台聚合 + 个人化筛选 prompt + MCP 可组合性**，而非重复做简历美化。

### 3.3 对本 Agent 的启示
可借鉴 Jobright/Jobscan 的"匹配度打分"与 ai-job-search 的"审查 agent"思路；但所有竞品均不覆盖 HK/深圳本地源，这正是本 Agent 的切入点空白。

---

## 4. 免费 API 数据源

### 4.1 数据源对比
| 数据源 | 免费性 | 认证 | rate limit | 覆盖 HK/CN | 字段完整度 | 文档质量 |
|--------|--------|------|-----------|------------|-----------|----------|
| Adzuna | 免费层〔事实〕 | API key〔事实〕 | 随 tier（待确认精确值）〔待验证〕 | 12 国含 UK/US，HK/CN 弱〔事实〕 | 高（含薪资历史） | 高〔事实〕 |
| Remotive | 免费〔推断〕 | 部分无需 key〔待验证〕 | 低〔待验证〕 | 仅远程岗，HK/CN 弱 | 中 | 中 |
| Arbeitnow | **免费无需 key**〔事实〕 | 无〔事实〕 | 低（小站）〔推断〕 | 欧洲/远程，HK/CN 弱 | 中 | 中 |
| The Muse | **Deprecated / 不再公众免费**〔事实，纠偏〕 | — | — | 不适用 | — | 移除出推荐 |
| Greenhouse Job Board | **免费公开**（per-company token）〔事实〕 | board token〔事实〕 | 有文档限流〔事实〕 | 依赖公司，全球含 HK 分部 | 高 | 高〔事实〕 |
| Lever / Ashby 等 ATS | 免费公开板〔事实〕 | board token〔事实〕 | 参考 ats-api-reference〔事实〕 | 全球 | 高 | 高 |
| TheirStack | **需付费/不推荐 C 端**〔事实，纠偏〕 | 商业 key | 商业级 | 偏 B2B lead | 高 | 中 |

### 4.2 覆盖缺口评估与 MVP 切入点结论〔呼应增补第 3 点〕
**结论（明确）**：MVP 必须以 **HK（劳工处开放数据 + JobsDB 爬取）+ Global Remote（Adzuna / Arbeitnow / Greenhouse·Lever ATS 板）** 为切入点，**暂缓深圳大陆平台爬虫开发**。
- 依据：① 深圳平台无官方 API 且强反爬（BOSS 的 SPA/行为验证）〔事实〕，初期投入产出比极低；② HK 有劳工处**开放授权数据**〔事实〕且 JobsDB 结构相对可爬〔推断〕，合规成本最低；③ Adzuna/Arbeitnow/ATS 板为免费/半免费合规源，可立即供给 remote + 在港分部岗位。
- 深圳源列入 Phase 2，待反爬方案（如合规数据合作/官方合作接口）成熟后接入。

---

## 5. 开源项目

### 5.1 项目对比
| 项目 | stars / 活跃度 | license | 技术栈 | 功能 | 可替换 HK/深圳源 | 可集成 MCP |
|------|----------------|---------|--------|------|------------------|-----------|
| **JobSpy** (zachgoll) | **10k+**〔事实，纠偏〕 | MIT〔推断〕 | Python | 抓取 LinkedIn/Indeed/Glassdoor/Zip | 源不含 HK/深圳，需扩展〔推断〕 | 已有 jobspy-mcp-server〔事实〕 |
| jobspy-mcp-server | 中〔待验证〕 | — | Python/MCP | 包装 JobSpy 为 MCP | 同 JobSpy | 是〔事实〕 |
| **AIHawk** (feder-cr) | **30k–40k+**〔事实，纠偏〕 | — | Python/Selenium | 自动投递 | 曾含 LinkedIn（已移除）〔事实〕 | 否（偏客户端） |
| ai-job-search (Mads) | 活跃（2026-03 起）〔事实〕 | — | Claude Code | 匹配+LaTeX 简历+审查 | 通用 | 否（框架） |
| career-ops (santifer) | 中〔待验证〕 | — | — | 求职 ops | 通用 | 否 |
| Auto_job_applier_linkedIn | 中〔待验证〕 | — | Python | 批量投递 | LinkedIn | 否 |
| adzuna-job-search-mcp | 活跃（2026-03）〔事实〕 | — | Python/MCP | Adzuna 搜索 | 全球含 remote | 是〔事实〕 |
| **startup.jobs MCP** | 真实存在〔事实，纠偏〕 | — | MCP | 初创公司招聘聚合 | 全球初创 | 是 |
| Jobs HK（劳工处爬虫 MCP） | 存在〔事实〕 | — | MCP | HK 劳工处数据 | HK〔事实〕 | 是 |
| Google Jobs MCP (SerpAPI) | 依赖 SerpAPI key〔事实〕 | — | MCP | Google Jobs | 全球 | 是（需付费 key） |
| Apify JobsDB scraper | 商业 actor〔事实〕 | 付费 | Apify | JobsDB 爬取 | HK〔事实〕 | 可经 API 调用 |

### 5.2 可复用资产
- 直接改造进 MCP 架构：**jobspy-mcp-server**（抓取底座）+ **adzuna-job-search-mcp**（API 源）+ **Jobs HK MCP**（劳工处）+ **startup.jobs MCP**（初创聚合）。
- 借鉴 **ai-job-search** 的"审查 agent"做 LLM 筛选打标；**AIHawk** 的投递逻辑作为 Phase 3 参考（注意其 LinkedIn 合规事件）。

---

## 6. 技术方案建议

### 6.1 抓取 Pipeline + 四方向关键词字典〔增补第 1 点〕
**Pipeline**：API 优先（Adzuna/ATS 板/劳工处）→ 爬虫兜底（JobsDB via Apify 或自建 polite scraper）→ 调度（定时/增量）→ 去重（URL+标题 hash）→ 存储（结构化 DB）→ LLM 筛选打标 → 通知（Webhook/邮件）。

**四方向 CN/EN 关键词字典（抓取过滤 + LLM 打标依据）**：
| 方向 | EN 关键词 | CN 关键词 |
|------|-----------|-----------|
| AI Agent | LLM Agent, MCP, Agentic AI, Tool-use | 智能体, Agent 开发, MCP Server |
| AI Application | AI App, RAG, GenAI Product | AI 应用, 大模型应用, RAG |
| Deep Learning | Deep Learning, CNN/Transformer, Multimodal | 深度学习, 大模型, 多模态 |
| Reinforcement Learning | RL, RLHF, Policy Gradient | 强化学习, 对齐, RLHF |

### 6.2 三层拆解落地设计
- **Interface Layer**：MCP Server 暴露 `search_jobs` / `filter_jobs` / `get_job` / `notify_match` 工具；Web 预览页（只读看板）。
- **Action Layer**：抓取调度器 + 去重 + LLM 筛选 prompt（依据 C1 条件：Python/LLM/MCP、3–5 年、中高级、HK$40k/RMB25k、拒外包合约、需本地雇佣签证）+ 打标（方向/地点/匹配分）。
- **Knowledge Layer**：PRD、架构图、prompt 模板、代码仓库（平台 agent 目录）、本调研报告归档。

### 6.3 成熟度路线
- MVP = **A2–A3**（SOP 半自动 + 端到端抓取筛选）。
- → A4：自主调度 + 主动推送 + 投递辅助（需解决平台 ToS）。
- → A5：基于用户反馈自优化筛选权重（需反馈数据闭环）。

### 6.4 工程落点（对齐开发流程）+ 最小规范建议〔C2〕
- 流程：自建 branch → `e2e-scratch-test` → 开源模型 CodeReview → 登记 agent-registry。
- **最小规范建议（播种用）**：
  - MCP tool 命名：`job_search` / `job_filter` / `job_detail` / `job_notify`（动词_名词）。
  - 输入 schema：`{ query, location:["HK"|"SZ"], direction, min_salary, visa_required, exclude_contract }`。
  - 输出 schema：`{ jobs:[{id, title, company, location, salary, direction, match_score, source}] , next_cursor }`。
  - registry 登记字段：`agent_id, owner, maturity_level, mcp_endpoint, data_sources, compliance_note`。

---

## 7. 合规与风险

### 7.1 法规对照
- **香港 PDPO**：第 33 条跨境转移条文**尚未生效**〔事实〕，目前理论上可传输出港，但仍须遵守收集目的、保安等规定。
- **大陆 PIPL**：出境个人信息 <10 万人可走标准合同/认证；≥10 万或敏感信息需 CAC 安全评估；需单独同意 + PIPIA〔事实〕。

### 7.2 抓取与数据合规 + 跨境风险〔增补第 4 点〕
- 反爬对策：礼貌频率（如 ≥5s 间隔、限流、轮换 UA）、优先用开放/API 源、尊重 robots。
- 个人数据处理：仅存职位公开信息，不抓候选人隐私；用户简历本地处理。
- **跨境合规风险（关键）**：将简历/个人数据送入**境外 LLM API** 触发 PIPL 跨境（≥10 万人阈值虽不直击个人，但敏感简历属个人信息）与 PDPO 关注。
- **缓解建议**：① 本地化推理（自托管开源模型做筛选打标）；② 脱敏（去姓名/联系方式后再送模型）；③ 若用境外 API，仅送职位 JD 文本，不送个人简历字段。

---

## 8. MVP 实施路线与未决问题

### 8.1 路线图（每阶段标注 A 级 + 成功度量）〔增补第 5 点〕
| 阶段 | 时间 | A 级 | 范围 | 成功度量 |
|------|------|------|------|----------|
| MVP | 1–2 周 | **A2** | HK 劳工处 + Adzuna/Arbeitnow + JobsDB(Apify)；基础 MCP 暴露 | 每周命中 ≥20 条匹配职位；筛选 precision ≥ 80% |
| 扩展 | 1 个月 | **A3** | 接入 Greenhouse/Lever/startup.jobs MCP；LLM 打标+通知；本地推理 | 覆盖源 ≥5；precision ≥ 85%；召回率可量化 |
| 规模化 | 3 个月 | **A4** | 自主调度+主动推送；视合规评估接入深圳源；反馈闭环 | 全自动日更；用户周留存；投递辅助合规可用 |

### 8.2 未决问题清单
1. 深圳源合规接入路径（官方合作/数据授权）？〔待验证〕
2. JobsDB 自建爬取 vs Apify 托管的成本权衡？
3. LLM 筛选用本地模型还是境外 API（跨境合规）？
4. 用户 C1 条件是否需支持多档薪资/签证组合？
5. agent-registry 中央服务何时落地（当前愿景阶段）？

---

## 9. TL;DR（一页，可对外发 blog）

**找工作 Agent：用 MCP 把 HK/深圳 AI 岗「自动喂」到你面前**

我是 AI Native Community 平台的首个试点 agent，目标是自动聚合香港与深圳的 AI Application / AI Agent / Deep Learning / Reinforcement Learning 技术岗，按你的条件（Python/LLM/MCP、3–5 年中高级、HK$40k·RMB25k 起、拒外包、需本地签证）智能筛选并推送。

**为什么现在做**：免费合规数据源已够用——Adzuna、Arbeitnow、Greenhouse/Lever 等 ATS 公开板、香港劳工处开放数据；而深圳 BOSS/拉勾/猎聘强反爬且无官方 API，我们明智地暂缓，先打 HK + 全球 Remote。

**怎么搭**：抓取 API 优先、爬虫兜底；封装成 MCP Server（`job_search`/`job_filter`/`job_detail`/`job_notify`），平台内其他 agent 可直接调用；LLM 做匹配打标，简历本地脱敏处理以满足 PIPL/PDPO 跨境合规。

**路线**：2 周出 MVP（A2，周命中 20+ 匹配岗、precision≥80%）→ 1 月扩到 A3（多源+主动通知）→ 3 月规模化到 A4（自主调度）。

**差异化**：大厂（BOSS DeepHire、LinkedIn AI）只做单平台匹配，我们做**跨平台聚合 + 个人化 prompt + 可组合 MCP**——这才是 agent 原生求职的正确形态。

---

*报告完成。全文约 4200 字，含 6 张对比表，事实/推断/待验证三档已标注，5 点增补与 C 类补充（C1 筛选条件已用于 §6.2、C2 最小规范见 §6.4、C3 按对外口径写 TL;DR）均已落实。此为 AI Native Community 平台 Knowledge Layer 第一份沉淀文档。*
