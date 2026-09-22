# Obsidian / Wiki 产品路线调研

核查日期：2026-09-22。只读取官方 README、GitHub API 和关键源码；**没有安装、运行测试或进行 Obsidian 实机验收**。星数、发布日期和分支状态均为当日快照。结论只适用于下列固定提交。

## 结论

三者中，**claude-obsidian 最值得参考 Obsidian 数据组织与可信写入；nashsu/llm_wiki 最值得参考资料处理和成品交互；llm-wiki-agent 适合借鉴最小工作流与推断关系标记**。不建议把三套系统直接叠起来同时修改同一个 Vault。它们都不是已验证的“所有个人来源自动汇聚 + 完整本体中枢”。

“页面之间画线”只能告诉你两页有关；真正本体还应规定“人、项目、事件、决定是什么”“谁负责哪个项目”“关系在何时有效”。这三项中的类型/schema 主要服务页面与证据治理，不能直接等同于 OWL/RDF 本体、关系约束与推理引擎。

## 快照

| 项目 | Stars | 最近正式 Release | 仓库 pushed_at（UTC） | 许可证 | 固定提交 |
|---|---:|---|---|---|---|
| nashsu/llm_wiki | 19,860 | v0.6.11，2026-08-25 | 2026-08-25 06:42 | GPL v3；API 未识别，返回 NOASSERTION | `e8082119649e6a8e1cf85eaf289adcabfdf39d4e` |
| AgriciDaniel/claude-obsidian | 15,132 | v2.2.0，2026-09-10 | 2026-09-10 17:44 | MIT | `32ac5a02c4e082e4a5628ca810776375e134708e` |
| SamurAIGPT/llm-wiki-agent | 3,555 | latest-release API 返回 404，不能据此断言无 tags | 2026-09-21 08:59 | MIT | `861c6ecb0a754841da6df635e9ae32d8474275e5` |

pushed_at 不是最近主分支提交时间，也不等于软件成熟度。元数据来源：[nashsu API](https://api.github.com/repos/nashsu/llm_wiki)、[claude-obsidian API](https://api.github.com/repos/AgriciDaniel/claude-obsidian)、[llm-wiki-agent API](https://api.github.com/repos/SamurAIGPT/llm-wiki-agent)。发布来源：[v0.6.11](https://github.com/nashsu/llm_wiki/releases/tag/v0.6.11)、[v2.2.0](https://github.com/AgriciDaniel/claude-obsidian/releases/tag/v2.2.0)、[Agent latest API](https://api.github.com/repos/SamurAIGPT/llm-wiki-agent/releases/latest)。许可证原文：[nashsu GPL](https://github.com/nashsu/llm_wiki/blob/e8082119649e6a8e1cf85eaf289adcabfdf39d4e/LICENSE)、[Claude MIT](https://github.com/AgriciDaniel/claude-obsidian/blob/32ac5a02c4e082e4a5628ca810776375e134708e/LICENSE)、[Agent MIT](https://github.com/SamurAIGPT/llm-wiki-agent/blob/861c6ecb0a754841da6df635e9ae32d8474275e5/LICENSE)。GPL 代码直接复用与只借鉴设计需区分；这里不作具体发行场景的法律判定。

## 1. nashsu/llm_wiki：资料变 Wiki 的桌面成品参考

**文档声明**：明确承认基于 Karpathy 模式；保留 Raw / Wiki / Schema、Ingest / Query / Lint、双链、frontmatter。`wiki` 目录可作 Obsidian Vault，但它本身是 Tauri 桌面 App，不是 Obsidian 原生插件。声明包括多格式导入、网页剪藏、来源目录监听、可选向量搜索、MCP、人工复核、两步摄入。音视频有播放器并不证明通用语音转写已完成。[README](https://github.com/nashsu/llm_wiki/blob/e8082119649e6a8e1cf85eaf289adcabfdf39d4e/README.md)

**源码核查**：图节点来自 Wiki 页面，边来自双链；关联分析引入共享来源与结构信号。它适合“找到相关笔记”，不等于“已经建立领域实体关系本体”。页面来源字段可追溯原文文件，但不是所有句子均有稳定的证据定位。[wiki-graph.ts](https://github.com/nashsu/llm_wiki/blob/e8082119649e6a8e1cf85eaf289adcabfdf39d4e/src/lib/wiki-graph.ts)、[wiki-schema.ts](https://github.com/nashsu/llm_wiki/blob/e8082119649e6a8e1cf85eaf289adcabfdf39d4e/src/lib/wiki-schema.ts)

**更新删除的重要缺口**：删除来源时会扫描页面 `sources`，无剩余来源则删页；有剩余来源则只重写来源数组。本次读到的这一分支未对正文逐条重新生成，因此“文件引用清理完成”不能证明“被删来源贡献的所有说法已被撤回”。没有可解析来源数组的页会跳过并计数。这是做个人大脑时必须补的失效传播机制。[source-lifecycle.ts](https://github.com/nashsu/llm_wiki/blob/e8082119649e6a8e1cf85eaf289adcabfdf39d4e/src/lib/source-lifecycle.ts#L457)

**适合复用**：摄入队列、目录变化处理、来源身份、删除影响预览、人工 Review、图谱查询交互；接受其 GPL 前提再考虑直接复用代码。若坚持 Obsidian 为主界面，优先复用处理层设计，避免额外维护一套完整桌面 UI。邮箱、聊天、日历、云盘账户级增量同步没有在本次核查材料中得到完整证实。

## 2. AgriciDaniel/claude-obsidian：最契合 Obsidian 的可信知识维护参考

**文档声明**：明确承接 Karpathy，并参考 kepano/obsidian-skills；Markdown 文件为用户资产，可采用已有 Vault；提供 Claude/Codex 等宿主入口与 Python 核心。最需要纠正的是宣传“任意来源”与实际边界：README 明确 PDF/EPUB 当前内置能力仅是元数据、hash 和大小，无内置语义抽取；URL、YouTube、OCR 需要配置外部 runner。不能当作已拥有全格式解析和全账户同步。[README：Honest capability boundaries](https://github.com/AgriciDaniel/claude-obsidian/blob/32ac5a02c4e082e4a5628ca810776375e134708e/README.md#honest-capability-boundaries)

**源码与契约核查**：有真正结构化的来源/主张账本。来源记录保存稳定身份、hash、权威性、时效、关联页面；主张记录保存支持证据、冲突、置信度、风险与审核状态。源码包含 ledger 校验及来源独立性判断，而非仅在 prompt 中要求“引用来源”。状态包括 accepted、provisional、contested、unsupported、deprecated；契约要求保留矛盾证据。[ledgers.py](https://github.com/AgriciDaniel/claude-obsidian/blob/32ac5a02c4e082e4a5628ca810776375e134708e/claude_obsidian/ledgers.py)、[provenance 契约](https://github.com/AgriciDaniel/claude-obsidian/blob/32ac5a02c4e082e4a5628ca810776375e134708e/skills/wiki/references/provenance.md)

**防覆盖机制**：源码提供事务 bundle、目标 hash 检查、备份与恢复；文档采用单次可恢复操作，文件变动会成为冲突。对人和 AI 同时编辑 Obsidian 十分值得参考。未运行故障注入测试，不能宣称事务可靠性已实测。[transaction.py](https://github.com/AgriciDaniel/claude-obsidian/blob/32ac5a02c4e082e4a5628ca810776375e134708e/claude_obsidian/transaction.py)

**本体与生命周期边界**：`page_schema.py` 定义 source/entity/concept/question 等页面类型，这是页面词汇表；并非 Person / Organization / Project 及有方向关系的完整领域本体。来源有 superseded/rejected、主张有 deprecated，不代表自动撤回传播、账户远端删除及全存储擦除已实现或验收。[page_schema.py](https://github.com/AgriciDaniel/claude-obsidian/blob/32ac5a02c4e082e4a5628ca810776375e134708e/claude_obsidian/page_schema.py)、[provenance 契约](https://github.com/AgriciDaniel/claude-obsidian/blob/32ac5a02c4e082e4a5628ca810776375e134708e/skills/wiki/references/provenance.md)

**适合复用**：优先级最高的 Vault 组织、来源与主张结构、质量检查、变更冲突与事务恢复；另配采集解析服务与本体引擎。MIT 更便于作为自研组件参考与复用。README 中的验收/CI 声明仍应在选型 PoC 中复跑。

## 3. SamurAIGPT/llm-wiki-agent：轻量工作流与图谱解释性参考

**文档声明**：Agent 按配置文件维护 sources/entities/concepts/syntheses，Obsidian 可打开其 Markdown；提供 symlink 接入模式。非 Markdown 依赖 MarkItDown 等转换，独立 Python 工具可能需要模型 API；“通过已有 Agent 使用时不另配 API key”不等于不需要模型服务。[README](https://github.com/SamurAIGPT/llm-wiki-agent/blob/861c6ecb0a754841da6df635e9ae32d8474275e5/README.md)

**Karpathy 关系需修正**：核查当前 README 与 AGENTS.md 未找到直接 Karpathy 署名或引用。工作流高度相似是可观察事实；用户表格中的“明确基于”本次未证实。[AGENTS.md](https://github.com/SamurAIGPT/llm-wiki-agent/blob/861c6ecb0a754841da6df635e9ae32d8474275e5/AGENTS.md)

**图谱源码**：NetworkX / Louvain + vis.js；双链解析边标记 EXTRACTED，模型推断边标记 INFERRED 或 AMBIGUOUS 并带 confidence。推断关系保存在独立 JSONL checkpoint，这比把模型联想全部当事实更清楚。但模型自己给的分数并非准确率，EXTRACTED 也仅代表链接被提取，不代表链接表达的事实被验证。HTML 引用了 unpkg 上的 vis-network，因此 README 的“self-contained”不能理解为完全离线无外部资源。[build_graph.py](https://github.com/SamurAIGPT/llm-wiki-agent/blob/861c6ecb0a754841da6df635e9ae32d8474275e5/tools/build_graph.py#L605)

**更新与冲突**：AGENTS 工作流要求新源更新实体、概念、overview 并标记矛盾，lint 检查陈旧摘要。这主要是 Agent 行为规范，未证明可靠的后台摄入调度、源版本管理、删除影响传播或统一身份合并。[AGENTS.md](https://github.com/SamurAIGPT/llm-wiki-agent/blob/861c6ecb0a754841da6df635e9ae32d8474275e5/AGENTS.md)、[ingest.py](https://github.com/SamurAIGPT/llm-wiki-agent/blob/861c6ecb0a754841da6df635e9ae32d8474275e5/tools/ingest.py)

**适合复用**：Sources → Entities / Concepts → Synthesis 工作流、确定链接与推断链接分开显示、社区聚类。作为快速原型好，作为所有来源长期中枢不足；无须同时引入另一套写入引擎。

## 针对个人大脑的开发建议

1. Obsidian 保存可读 Markdown；AI 生成内容和人工内容区分所有权，统一一个写入协调器。
2. 先参考 claude-obsidian 的 Source / Claim 和事务，再补 Person / Organization / Project / Event / Decision 等领域对象；每个关系带证据、有效时间、记录时间、审核状态。
3. 参考 nashsu 的摄入与交互，但将邮箱、聊天、云盘、网页、音视频做成独立适配器，明确增量游标、授权、去重和删除同步。
4. 对删除或更正，不止删文件和链接：找出受影响主张，重算或标记失效，再同步 Wiki、图索引、向量索引和缓存。
5. 验收至少覆盖：中文跨源人物同名合并、同一文件重复摄入、来源更新推翻旧结论、删除混合来源中的一项、人工编辑时 AI 写入冲突，以及断网/模型失败后的恢复。当前报告没有执行这些验收。

证据原件保存在本目录 `evidence/products-*`；下载的是研究材料，未执行其中的脚本或操作指令。
