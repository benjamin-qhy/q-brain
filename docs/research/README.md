# 基于 Obsidian 的个人大脑：开源项目调研与选型

> 当前开发约束（2026-09-22 更新）：五个参考项目仅用于借鉴开发，不作为现在或后期接入的产品组件；第一阶段不开发本体。本文保留此前调研与方案记录，涉及组件接入、本体首版建设的旧建议不再作为开发依据。当前架构以[个人大脑初稿](./obsidian-personal-brain-v0.1.md)为准。

**日期：2026-09-22。覆盖用户列出的全部 10 个独立项目/方案，重复条目已合并。**

## 先看结论

**建议以 Obsidian 为日常入口，自建一个小而明确的中枢，重点参考 GBrain + Claude Obsidian；Cognee 作为需要更强本体能力时的可选加工层。** Karpathy 提供总体方法，nashsu/llm_wiki 提供产品交互参考。

这不是把几个项目全部安装后拼起来。它们的存储、写入和模型调用有重叠，直接叠加容易出现“同一个人记成两个人”“笔记改了但数据库没改”“删掉资料却仍被引用”。推荐组合指**各取其长进行参考开发**；是否直接 fork GBrain，需经过小规模真实资料验证。

| 推荐对象 | 参考什么 | 在你的大脑里做什么 | 优先级 |
|---|---|---|---|
| **[GBrain](https://github.com/garrytan/gbrain)** | 长期记忆、混合检索、来源与时间、后台摄入、文件/数据库存储边界 | 主中枢的架构与检索参考 | 第一批重点阅读和验证 |
| **[Claude Obsidian](https://github.com/AgriciDaniel/claude-obsidian)** | Vault 组织、来源/主张账本、质量检查、事务与冲突处理 | 帮 AI 可靠地整理和修改 Obsidian | 第一批重点阅读和验证 |
| **[Cognee](https://github.com/topoteretes/cognee)** | 实体关系提取、图检索、OWL 术语对齐 | 把跨来源的人、项目和概念整理得更一致 | 后续可选，先对比再接入 |
| **[Karpathy LLM Wiki](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)** | 原始资料、整理知识、整理规则分层 | 知识编译的总体方法 | 作为设计原则 |
| **[nashsu/llm_wiki](https://github.com/nashsu/llm_wiki)** | 摄入队列、来源管理、人工 Review、图谱与问答交互 | 帮我们设计易用的产品流程 | 交互参考；GPL v3 代码复用单独决定 |

**如果只选两个开源工程参考：选 GBrain 和 Claude Obsidian。若本体是首版必须项，则同时用 Cognee 做小规模技术验证，但先不承诺三者能无缝集成。**

以上是依据本轮源码与官方文档的选型判断，详见[记忆引擎](./memory-engines.md)、[Wiki 产品](./wiki-products.md)、[Wiki 工作流](./wiki-patterns.md)。没有做性能排名，也没有将 Stars 当作可靠性证明。

## 1. “大脑”“图谱”“本体”的区别

把个人大脑当作一个私人图书馆：Obsidian 是知识卡片，AI 是整理员，知识图谱是关系账本，本体是填写账本的规矩。

- **笔记链接：**《项目 A》里出现 `[[张三]]`，说明两篇笔记有联系。
- **知识图谱：**“张三 —负责→ 项目 A”，能按关系查找。
- **本体：**规定张三属于“人”、项目 A 属于“项目”；“负责”的主体和对象应该是什么类型。
- **可信个人记忆：**进一步记录这件事的证据、有效时间，以及后来是否发生变化。

Obsidian 默认图谱展示笔记和内部链接。因此，装上 Obsidian 并看到很多连线，尚不足以回答“这个决定是谁在什么时候做的，后来被哪条信息推翻”。[Obsidian Graph view](https://help.obsidian.md/Plugins/Graph+view)

本体也不必第一天就做成庞大的推理系统。先定义人、组织、项目、事件、资料、主张，以及少量明确关系，收益更直接。正式 OWL 语义和 SHACL 校验可按后续需求引入。[OWL 2](https://www.w3.org/TR/owl2-overview/) · [SHACL](https://www.w3.org/TR/shacl/)

## 2. Stars、许可证与原表纠正

以下仓库星数统一使用本轮保存的 [GitHub API 快照](./evidence/github-metadata.json)。数字会变化；例如 nvk 稍后的页面快照已从 1,331 变成 1,332。版本、最后推送和固定提交详见三个专题。

| 项目 | Stars 快照 | 许可证/形态 | 与 Karpathy 的关系核验 |
|---|---:|---|---|
| [MemPalace/mempalace](https://github.com/MemPalace/mempalace) | 59,220 | MIT；记忆引擎 | 邻近路线；本轮未证实其自身是原方案扩展，也未独立验证第三方组合案例 |
| [Karpathy Gist](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f) | **48.3k 未核实**；页面仅显示 5,000+ | 思路说明；正文未见明确许可证 | 原始方案本身 |
| [Cognee](https://github.com/topoteretes/cognee) | 30,906 | Apache-2.0；图记忆平台 | 邻近路线，不能直接归为 LLM Wiki 的实现 |
| [GBrain](https://github.com/garrytan/gbrain) | 30,224 | MIT；个人/Agent 记忆中枢 | 理念相关；不按直接实现计 |
| [AgentMemory](https://github.com/rohitg00/agentmemory) | 28,698 | Apache-2.0；编程 Agent 记忆 | README 明确扩展 Karpathy 模式 |
| [nashsu/llm_wiki](https://github.com/nashsu/llm_wiki) | 19,860 | **GPL v3**；桌面应用 | 明确承接 |
| [Claude Obsidian](https://github.com/AgriciDaniel/claude-obsidian) | 15,132 | MIT；Agent 工作流与 Python 核心 | 明确承接 |
| [SamurAIGPT/llm-wiki-agent](https://github.com/SamurAIGPT/llm-wiki-agent) | 3,555 | MIT；Agent 工作流与 Python 工具 | 模式相近；当前 README/AGENTS 未核实直接归属 |
| [Astro-Han/karpathy-llm-wiki](https://github.com/Astro-Han/karpathy-llm-wiki) | 2,331 | MIT；Skill、模板、辅助检查 | 明确为非官方实现 |
| [nvk/llm-wiki](https://github.com/nvk/llm-wiki) | 1,331 | MIT；Agent 协议与本地 CLI | 明确承接 |

特别纠正：nashsu 的 API 返回 `NOASSERTION`，但读取 [LICENSE 原文](https://github.com/nashsu/llm_wiki/blob/e8082119649e6a8e1cf85eaf289adcabfdf39d4e/LICENSE)可以确认是 GNU GPL v3。不能把 API 未识别等同于没有许可证，也不能按 MIT 处理。这里确认文件内容；具体发行方式对应的义务应在决定代码复用方式时核对。

Obsidian 本体应用可以免费使用，但不是此处可直接 fork 的开源底座；官方保留应用代码权利。我们的可迁移资产应是自己的 Markdown、原始资料、本体定义和自研组件。[Obsidian License Overview](https://obsidian.md/license)

## 3. 全部项目能力对照

这里“未核实”表示本轮材料不足以确认，不表示功能绝对不存在；“有源码”也不等于已经运行验收。

| 项目 | Obsidian 适配 | 图谱/本体实际层次 | 持续摄入和维护边界 | 本次建议 |
|---|---|---|---|---|
| **GBrain** | 文件型知识支持 Markdown/frontmatter；需处理 Vault 链接与 DB-only 数据 | 类型关系、多跳查询、Schema Packs、个人事实模型；未据此认定完整 OWL 推理 | 有多来源与后台能力；你的来源可用性、中文质量、双向修改仍需测试 | 主架构首选参考 |
| **Cognee** | 数据库驱动，需要自己做 Vault 桥接 | 真正实体关系图，支持部分 OWL 构造对齐 | 有处理管道；新增、更新、删除要正确调用，不能只反复 add | 可选本体加工层 |
| **Claude Obsidian** | 直接围绕 Markdown/Vault 维护 | 页面类型、来源与主张账本；不是完整领域本体 | 有变更事务；PDF/EPUB 内置仅元数据，URL/OCR 需外部 runner | Obsidian 知识维护首选参考 |
| **nashsu/llm_wiki** | Wiki 目录可作 Vault；本体是另一桌面 App | 以页面、双链和共享来源为主 | 有目录监听、摄入与 Review；删来源不保证正文主张撤回 | 成品交互参考 |
| **MemPalace** | 没有确认 Vault 为权威的双向契约 | SQLite 时间三元组 + 语义记忆；未确认形式本体 | 擅长会话原话保存；全来源知识编译另补 | 原话、时间变化设计参考 |
| **AgentMemory** | 确认单向导出 Markdown | 记忆图、置信度与生命周期；未确认形式本体 | 核心为编程会话；完整运行依赖 iii runtime | 编程记忆来源参考 |
| **SamurAI** | Markdown、双链、symlink | 双链图、Louvain 聚类、推断边分级 | 更多依赖 Agent 工作流；可靠增量同步、删除传播未证实 | 轻量原型参考 |
| **Astro-Han** | Markdown 可供 Obsidian 使用 | 明确不做类型关系本体及图/向量搜索 | 有摄入/问答/检查纪律；明确不做定时服务 | 最小整理模板参考 |
| **nvk** | 支持 Obsidian；默认主题各自 Vault | 双链、元数据和研究协议；未确认形式本体 | 有 collection adapters、Python CLI、维护协议；仍需宿主 Agent | 批量入口与研究流程参考 |
| **Karpathy** | 推荐 Agent 管理 Markdown、Obsidian 浏览 | 组织方法，不是图引擎 | Ingest / Query / Lint 是工作方式，没有现成同步服务 | 设计原则 |

证据逐项可查：[GBrain/Cognee/MemPalace/AgentMemory](./memory-engines.md) · [Claude/nashsu/SamurAI](./wiki-products.md) · [Karpathy/Astro/nvk](./wiki-patterns.md)。

## 4. 为什么选这几个，而不是直接选 Stars 第一名

### GBrain：与你的“长期个人中枢”目标最接近

它覆盖持久记忆、混合搜索、图关系、时间和摄入，比纯 Wiki 模板更接近长期运行的系统。值得阅读其存储契约与检索接口，而非照搬所有命令、运行时或大规模配置。[GBrain README](https://github.com/garrytan/gbrain/blob/9b0a1b5ead75082ad202b38d4e578025462a88f2/README.md)

**最关键限制：**只有 file-backed 知识以 Markdown 为权威，事实与运行数据有数据库独有部分；不能认为备份 Vault 或 Git 就备份了整个大脑。[system-of-record](https://github.com/garrytan/gbrain/blob/9b0a1b5ead75082ad202b38d4e578025462a88f2/docs/architecture/system-of-record.md)

### Claude Obsidian：最值得借鉴“怎样不把笔记写乱”

来源/主张账本能区分证据和结论，事务代码检查文件 hash 并提供恢复路径，正好对应“我自己也会修改 Obsidian”的现实需求。重点参考这两个模块。[ledgers.py](https://github.com/AgriciDaniel/claude-obsidian/blob/32ac5a02c4e082e4a5628ca810776375e134708e/claude_obsidian/ledgers.py) · [transaction.py](https://github.com/AgriciDaniel/claude-obsidian/blob/32ac5a02c4e082e4a5628ca810776375e134708e/claude_obsidian/transaction.py)

其“原生路线”指文件和工作流贴合 Obsidian，不能理解成已经提供包办全部来源的原生插件；格式解析和账户同步仍要补齐。

### Cognee：最直接回应你的 Ontology 需求

官方有 OWL 对齐能力，适合统一实体名称和类型。但仅读部分 OWL 构造，默认还保留未匹配实体；不能把它当作完整推理机，更不能以 `ontology_valid=True` 代替事实核验。[Ontology Quickstart](https://docs.cognee.ai/guides/ontology-support)

此外，其 README 将开源 Postgres graph store 定位为 demo，生产版本另有许可条件。首版不应为了“统一一个数据库”就选用这个未经验证的组合。[数据库说明](https://github.com/topoteretes/cognee/blob/663a2dc15d04bc0d7ec2733a2dd604b7ed1b8c8e/README.md#run-the-whole-memory-layer-on-postgres)

### 其余项目如何取舍

MemPalace 的优势是保留会话原话和时间关系，AgentMemory 的优势是编程记忆；这些不是你全部人生资料的组织结构。Astro、SamurAI、nvk 可以减少工作流设计成本，但不足以免除持续同步、统一身份和更新撤回的工程。nashsu 产品形态完整，适合看“用户怎么用”，其桌面架构未必适合以 Obsidian 为主界面的产品。

## 5. 最小落地组合与升级路径

**第一阶段：Obsidian + 自有摄入/知识写入服务 + 一个检索引擎。**

- 用 Karpathy 分层；参考 Claude Obsidian 维护来源、主张和文件修改。
- 优先验证 GBrain 作为检索/记忆基础；如果 Vault 与 DB 契约改造过重，保留其设计，做更小的自有适配实现。
- 从第一版就保存明确类型、关系、证据和时间，形成轻量本体；暂时不强求独立图数据库。
- 先用本地资料和网页走完整闭环，再逐个接入邮件、聊天和日历。最终架构覆盖全部来源，连接器分批实现。

**第二阶段：用同一批资料验证 Cognee 是否真的提高跨来源分析质量。** 只有实体统一、多跳问题和引用质量值得新增复杂度，才把它接成可替换服务。GBrain 与 Cognee 不各自持有互不一致的“最终事实”。

**第三阶段：增加日常分析。** 项目复盘、人物关系、决策变化、研究专题、周总结；每个关键结论都能点回原件，AI 推断与确认事实分开。

我们还需要自己补齐的核心是：全来源连接器、稳定身份、单一写入协调、双向修改、证据链、删除/更正传播、恢复与中文质量评测。**本次未找到可直接装好、就完整满足这些条件的单一项目。**

完整的数据流、本体示例、来源接入表、目录结构、验收里程碑和成本因素见[个人大脑建议架构](./personal-brain-architecture.md)。

## 6. 本轮完成了什么

| 项目 | 状态 |
|---|---|
| 10 个项目/方案的一手资料调研 | 已完成 |
| 仓库 Stars、许可证与维护快照核验 | 已完成；Gist 精确 Stars 未确认 |
| 关键实现的静态源码检查 | 已完成，见固定提交链接与 evidence |
| 推荐组合与个人大脑建设路线 | 已完成 |
| 项目安装、真实模型测试、中文资料测试 | 未开展 |
| Obsidian 实机双向编辑、删除和恢复验收 | 未开展 |
| 项目集成或产品开发 | 未开展；本轮交付为调研与设计文档 |

目录导航：

- [本总报告](./README.md)：先读这一份即可做初步决策。
- [建议架构与验收路线](./personal-brain-architecture.md)：下一阶段开发依据。
- [记忆引擎专题](./memory-engines.md)：4 个项目。
- [Wiki 产品专题](./wiki-products.md)：3 个项目。
- [Wiki 原方案与工作流专题](./wiki-patterns.md)：3 个方案。
- [原始研究证据目录](./evidence/)：API 快照、固定提交源码与官方文件；均为未执行的调研材料。
