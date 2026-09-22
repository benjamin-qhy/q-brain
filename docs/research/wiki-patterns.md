# LLM Wiki 原方案与两个工作流实现：调研

核查日期：2026-09-22。范围：官方 Gist、仓库主页、README、协议文件、Python 辅助工具、LICENSE 与提交 Atom。只读源码，没有安装或运行这些项目；功能存在与实际效果分开判断。

## 结论

**Karpathy 适合作为设计原则；Astro-Han 适合作为最小知识编译流程；nvk 适合借鉴较完整的研究、收集、会话整理与质量管理。三个都不能单独承担“所有个人信息持续汇聚 + 正式本体”的完整中枢。** 以下是源码检查后的架构判断。

把个人大脑想成自己的图书馆：Raw 是原始藏书，Wiki 是整理后的知识卡片，Schema 是整理规矩。链接图告诉你“两张卡片有关”；本体还要明确“张三是人、项目 A 是项目、张三负责项目 A”，规定哪些关系合法，并保留证据及有效时间。**提示词里的 Schema 不自动等于本体，Obsidian 连线图也不自动等于带类型的知识图谱。**

## 核查总表

| 项目 | 此次 Stars | 许可证 | 活动快照 | 实际形态 | 对个人大脑的价值 |
| --- | ---: | --- | --- | --- | --- |
| [Karpathy LLM Wiki](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f) | 页面只显示 5,000+；无法确认用户表格中的 48.3k | Gist 正文未见明确许可证 | 创建于 2026-04-04，页面显示 1 次修订；评论活跃不代表方案代码更新 | 一份设计说明，不是应用或服务 | 定义原始资料、知识页、整理规范的分层 |
| [Astro-Han/karpathy-llm-wiki](https://github.com/Astro-Han/karpathy-llm-wiki) | 2,331 | [MIT](https://github.com/Astro-Han/karpathy-llm-wiki/blob/main/LICENSE) | 默认分支最新提交为 [2026-07-23 eafcc770](https://github.com/Astro-Han/karpathy-llm-wiki/commit/eafcc77001e496cc43499e4923b663aec722c813)，主页 28 commits | Agent Skill + 模板 + Python 证据检查 | 简洁、可移植的摄入和整理规范 |
| [nvk/llm-wiki](https://github.com/nvk/llm-wiki) | 1,332 | [MIT](https://github.com/nvk/llm-wiki/blob/master/LICENSE) | 默认分支最新提交为 [2026-09-15 1224fbcd](https://github.com/nvk/llm-wiki/commit/1224fbcdf3827f4ba56d225a9e359f5e8a5594e5)，README 为 v0.25.0 | 多 Agent 运行环境插件、工作流协议和本地确定性工具 | 研究、批量收集、长期维护、会话知识晋升 |

Stars 来自 GitHub 公开 HTML 的精确标题，Gist 是展示上限标签，不应把 5,000+ 当成真实精确数量。REST API 返回限流 403，故未获取创建时间等完整 API 元数据。原始元数据见 [patterns-metadata.json](evidence/patterns-metadata.json)。活动依据默认分支提交，不推断维护承诺。

## 1. Karpathy LLM Wiki

[原文](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)明确说这是交给 Agent 的思路文件。提出三个层次和 Ingest / Query / Lint：资料到来时整理知识，回答引用已有页面，定期检查矛盾和断链。Obsidian 用于浏览，Agent 修改 Markdown；索引和日志承载导航、历史。原文认为约百份来源的中等规模可先用文件索引，大规模再接搜索工具。

**适合借鉴：** 保留原始资料、持续更新知识页、回答可以沉淀为新页，避免每次从零分析。**缺口：** 没有连接器、同步进程、任务队列、重试、身份合并或正式本体实现；这些要另外开发。原文所谓 Schema 是 Agent 工作规则。不要把“理论上可接 Slack/会议记录”写成已经提供集成。

## 2. Astro-Han/karpathy-llm-wiki

### 存在的实现

[README](https://github.com/Astro-Han/karpathy-llm-wiki/blob/main/README.md)明确标为非官方社区实现。[SKILL.md](https://github.com/Astro-Han/karpathy-llm-wiki/blob/main/SKILL.md)规定不可修改的 raw、整理后的 wiki、索引和追加日志；新资料先判断新增、更新、冲突、无新增知识，再编译。受到影响的旧页要连带更新，旧观点被替代或有争议时保留状态。查询默认不写文件，用户要求保存时才归档。

实际代码 [check_evidence.py](https://github.com/Astro-Han/karpathy-llm-wiki/blob/main/scripts/check_evidence.py)检查数字、日期、较长引文能否在已关联原文中找到，并报告缺失原文链接及未使用来源。它是范围有限的字面检查，不是事实真伪或语义推理验证；小整数等也明确不在机械覆盖范围内。

### 适配与边界

普通 Markdown 和相对链接可以用于 Obsidian，但这里不是 Obsidian 插件。当前规则对 wiki 只允许一层主题子目录；若用户已有复杂 Vault，不能直接强套目录。

最重要的核查：README 的 **Design Boundaries 明确不做** typed relationship ontology、向量/图搜索、MCP、UI、自动定时任务；也暂不建立自动撤回坏来源的机制。这不是隐藏待发现功能，而是有意缩小范围。故适合参考知识整理纪律，不适合作为本体/全来源同步底座。[边界原文](https://github.com/Astro-Han/karpathy-llm-wiki#design-boundaries)

README 的 94 篇知识页、99 份来源是作者自报使用规模，本次未独立复现，不能证明它能承载多年全量个人数据。

## 3. nvk/llm-wiki

### 已超过“纯提示词”，但仍需要 Agent 运行环境

[README](https://github.com/nvk/llm-wiki/blob/master/README.md)和 [AGENTS.md](https://github.com/nvk/llm-wiki/blob/master/AGENTS.md)直接归功于 Karpathy，并提供多运行环境插件。研究、论题正反论证、摄入、编译、问答、审查、输出等主要是 Agent 协议；[scripts/llm-wiki](https://github.com/nvk/llm-wiki/blob/master/scripts/llm-wiki)确实存在 Python 实现，负责结构检查、归档、适配器注册、数据集清单和其他确定性操作。不能描述为只有一份提示词，也不能描述为装好就自动运行的独立知识服务。

值得借鉴的具体模块：

- **多来源批量入口：** 文档列出 Git、MediaWiki、CSV 消息归档、Wayback 等 collection adapter，以及外部私有 adapter 协议；CSV 聊天导入不代表实时同步微信、所有邮箱和所有网盘。
- **会话与知识分层：** `.sessions/` 保存经删减的操作记忆，提炼并显式 promote 到 raw 后才编译为知识，避免完整聊天直接成为事实库。
- **大型资料不强塞 Markdown：** datasets 保存外部数据的位置、格式、结构和查询方法；inventory 保存需要后续行动的对象。
- **质量与时效：** lint 有本地结构检查；librarian / refresh 有旧知识检查和更新流程；需要 Agent 判断的事实审核仍不等于机器保证正确。

上述能力依据 README 命令说明与 Python 入口静态检查，没有执行真实连接器、LLM 或规模测试。

### Obsidian 与本体

[Obsidian Integration](https://github.com/nvk/llm-wiki#obsidian-integration)明确提供 wikilinks、别名、标签和可选 `.obsidian` 配置，每个 topic 默认是独立 Vault；hub 默认不配置 Vault。对“一个统一个人中枢”的目标，需要主动统一跨主题实体和导航，不能默认多个主题 Vault 自然组成统一图谱。

[协议](https://github.com/nvk/llm-wiki/blob/master/AGENTS.md)包含 category、sources、confidence 以及 inventory kind/status 等结构字段；但 README 将 schema.md 定义为人维护的主题指南，本次未见 RDF/OWL、SHACL 或全局类型关系推理实现。**有元数据和双链，尚不足以认定已有正式本体系统。**

## 建议如何参考开发

以下为针对用户目标的建议，不是上游已实现承诺：

1. 用 Karpathy 的分层作为基础，但把个人原始笔记和 AI 生成页面分开，避免自动整理改写用户原意。
2. 借 Astro-Han 的摄入分流、冲突状态、来源检查和串行编译规则，形成最小可靠知识编译器。
3. 从 nvk 挑选批量入口、会话晋升、外部数据清单、研究正反证据流程，不必整套搬入其主题 Vault 和大量命令。
4. 单独设计全局实体 ID、类型与关系：人、组织、项目、事件、目标、资料、主张；关系至少带 source_id、有效时间、生成方式和审核状态。知识图谱作为可重建索引，Obsidian 页作为可阅读视图。
5. 全来源入口另外补可靠同步：首次全量、增量游标、去重、失败重试、删除传播、权限与脱敏。试点应覆盖“同一个人跨聊天和会议出现”“来源撤回后结论失效”“项目事实随时间变化”，通过后再扩大导入。

三者中，若只选一个工程参考：**nvk 覆盖更广，但应作为工作流参考而非图谱引擎；若只选一个最小起步模板，选 Astro-Han。** 最终图谱引擎的取舍需结合其他候选项目调查结果。
