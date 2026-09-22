# 记忆引擎专题：MemPalace、Cognee、GBrain、AgentMemory

调研时间：2026-09-22。方法：GitHub REST API、固定提交源码、官方文档；未安装、未运行项目，功能描述属于文档/源码核验，不代表实际验收。动态数字以保存的 `evidence/engines-*.json` 为本轮快照；最后推送不等同最后稳定版本，以下源码来自默认分支，可能领先 release。

## 结论

**优先参考 GBrain 的个人大脑架构，以 Cognee 补足可选本体处理层；MemPalace 和 AgentMemory 作为聊天记忆设计参考，首版不并行运行四套记忆引擎。**这是面向“Obsidian 是日常入口、个人全来源信息汇集”的选型判断，不是通用性能排名。

- GBrain 最接近完整个人中枢：来源摄入、文件与数据库边界、带出处的事实、时间变化、检索和后台任务都值得参考。但必须设计自己的 Obsidian 双向修改与备份契约。
- Cognee 更适合“让同一概念有统一名字和类型”的处理器；支持 OWL 对齐，但不是完整 OWL 推理系统。
- MemPalace 适合保留并找回聊天原话，避免只剩摘要；不适合作为 Obsidian Wiki 的唯一存储。
- AgentMemory 的重点是编程 Agent 跨会话记忆，其 Obsidian 功能是从内部状态导出 Markdown，不能等同原生 Vault 双向管理。

## 当前仓库状态

| 项目 | Stars | 许可证 | 最近推送 UTC | 最新 GitHub release | 审阅默认分支提交 |
|---|---:|---|---|---|---|
| MemPalace/mempalace | 59,220 | MIT | 2026-09-22T11:44:14Z | [v3.10.0](https://github.com/MemPalace/mempalace/releases/tag/v3.10.0)（2026-09-16） | `c4d3711ee247` |
| topoteretes/cognee | 30,906 | Apache-2.0 | 2026-09-22T11:43:41Z | [v1.6.0](https://github.com/topoteretes/cognee/releases/tag/v1.6.0)（2026-09-18） | `663a2dc15d04` |
| garrytan/gbrain | 30,224 | MIT | 2026-09-22T02:53:09Z | [v0.51.6.0](https://github.com/garrytan/gbrain/releases/tag/v0.51.6.0)（2026-09-21） | `9b0a1b5ead75` |
| rohitg00/agentmemory | 28,698 | Apache-2.0 | 2026-09-21T06:13:05Z | [v0.9.29](https://github.com/rohitg00/agentmemory/releases/tag/v0.9.29)（2026-08-16） | `e04ba88819c3` |

原始查询：[MemPalace API](https://api.github.com/repos/MemPalace/mempalace)、[Cognee API](https://api.github.com/repos/topoteretes/cognee)、[GBrain API](https://api.github.com/repos/garrytan/gbrain)、[AgentMemory API](https://api.github.com/repos/rohitg00/agentmemory)。四者本轮均未归档；Stars 只代表关注度。许可证来自仓库 API 的 SPDX 识别，具体复用以相应 LICENSE 与依赖许可证为准。

## GBrain：最适合参考主干，但不是纯 Vault

**已核验能力。**Markdown、wikilinks 和类型化关系进入图谱，支持多跳查询；来源摄入包含原生 Google 邮件/日历/联系人、会话历史、收件箱和 webhook。零密钥可使用基本记忆与关键词搜索，语义检索/合成/自动增强要另配模型能力。[README](https://github.com/garrytan/gbrain/blob/9b0a1b5ead75082ad202b38d4e578025462a88f2/README.md)

**Obsidian 边界。**跨目录的裸 `[[note-name]]` 链接默认不启用全局 basename 解析，需显式配置。更关键的是，官方存储契约仅称文件支持的知识以 Markdown/frontmatter 为权威；数据库独有知识、凭证、版本历史、写入回执、撤回记录不能只靠 Git 重建。因此不能把“Git clone=完整个人大脑备份”当设计前提。[存储契约](https://github.com/garrytan/gbrain/blob/9b0a1b5ead75082ad202b38d4e578025462a88f2/docs/architecture/system-of-record.md)

**本体能力。**Schema Packs 描述目录、页面类型、类型推断和关系动词，能适配用户已有目录。[Schema Packs](https://github.com/garrytan/gbrain/blob/9b0a1b5ead75082ad202b38d4e578025462a88f2/docs/architecture/schema-packs.md) Life Chronicle 的 ontology 源码包含维度别名归一化、事实值去重、未知维度隔离等实际逻辑；它是个人事实模型，不应据此声称完整 RDF/OWL 推理能力。[ontology.ts](https://github.com/garrytan/gbrain/blob/9b0a1b5ead75082ad202b38d4e578025462a88f2/src/core/chronicle/ontology.ts)

**建议借鉴。**原始来源和加工页面分开；每条结论保留来源；写入携带版本与幂等标识；撤回与删除有明确状态；后台任务具备恢复机制。个人版先用本地 PGLite 验证，再决定是否需要 Postgres。大量后台增强会增加维护和模型费用；中文人名、无空格中文全文搜索、别名合并需单独验收，不能凭英语演示认定已满足。

## Cognee：四者中最明确的 OWL 对齐层

**已核验能力。**关系库记录文档与来源，向量库做语义检索，图数据库保存实体关系；其定位是数据库驱动的记忆处理平台，不是 Obsidian Vault 管理器。[架构](https://docs.cognee.ai/core-concepts/architecture)

**本体边界。**官方指引明确读取 `owl:Class`、`rdf:type`、`rdfs:subClassOf`、`owl:ObjectProperty`，忽略 `owl:DatatypeProperty`。默认 annotate 仅补充和规范化，未匹配实体仍留在图中；strict 才丢弃未对齐实体，但原始文本块仍能被检索。OWL 文件初始化时加载，运行中修改不会自动生效。因此它适合术语对齐与关系补充，不等于约束校验器或完整逻辑推理机。[Ontology Quickstart](https://docs.cognee.ai/guides/ontology-support)、[解析实现](https://github.com/topoteretes/cognee/blob/663a2dc15d04bc0d7ec2733a2dd604b7ed1b8c8e/cognee/modules/ontology/rdf_xml/RDFLibOntologyResolver.py)、[模式配置](https://github.com/topoteretes/cognee/blob/663a2dc15d04bc0d7ec2733a2dd604b7ed1b8c8e/cognee/modules/ontology/ontology_config.py)

**更新边界。**源码明确：同一来源路径但内容变化必须走 `update()`；`add()` 对同内容去重，对变更内容拒绝，避免造出重复文档。该防护不覆盖 S3 对象和仓库 URL。接入 Obsidian 文件监听器时必须正确区分新增、修改与删除，不能每次变化都再次 add。[更新前置检查](https://github.com/topoteretes/cognee/blob/663a2dc15d04bc0d7ec2733a2dd604b7ed1b8c8e/cognee/tasks/ingestion/refuse_changed_existing_documents.py)

**许可证/成本重要限制。**仓库整体 Apache-2.0，但 README 单独注明：Postgres 充当 graph store 的开放版本是 demo，production-ready 版本为 licensed product。因此不要把“一台 Postgres 承载成熟的全栈图记忆”作为无需额外条件的开源交付承诺。[README 数据库说明](https://github.com/topoteretes/cognee/blob/663a2dc15d04bc0d7ec2733a2dd604b7ed1b8c8e/README.md#run-the-whole-memory-layer-on-postgres)

**建议借鉴。**将 Cognee 放在可替换的加工接口后面，输入稳定 ID 的资料和本体定义，输出实体、关系和证据。原始材料与可读知识仍放自己的 Vault；不让 Cognee 数据库成为唯一原件。首版先确定类型与关系词表，出现跨来源实体归一需求再接入。

## MemPalace：适合原话记忆侧层

**已核验能力。**核心定位是逐字保留会话、语义检索；默认 ChromaDB，另有可替换存储后端，本地 embeddings 可无需云 API。不是将资料编译成可编辑 Wiki 的产品。[README](https://github.com/MemPalace/mempalace/blob/c4d3711ee2478bb4062085fa075597af897d3a6d/README.md)

**图谱边界。**独立 SQLite 保存主语—谓词—宾语及有效时间，可 invalidate 和 supersede，并保留历史查询。`source_closet` 将关系指回原始记忆，supersede 把旧事实结束与新事实开始置于同一事务。[知识图谱说明](https://github.com/MemPalace/mempalace/blob/c4d3711ee2478bb4062085fa075597af897d3a6d/website/concepts/knowledge-graph.md)、[实现](https://github.com/MemPalace/mempalace/blob/c4d3711ee2478bb4062085fa075597af897d3a6d/mempalace/knowledge_graph.py)

**适配判断。**这是真实的时间关系图谱，但当前核验材料没有建立通用 OWL 类型约束/推理或 Vault 双向存储契约。其“失效”保留历史，不等同隐私要求的彻底删除；若接入，需要明确原文、向量、关系、导出和备份各处的删除传播。

**建议借鉴。**原话保存、时间窗口、旧事实被新事实取代的设计。若首版已由主干保存完整聊天原文与索引，不必再引入 MemPalace，避免两份记忆各自更新。README 的 benchmark 是项目自报，且检索命中不等于最终回答正确，本轮未复跑。

## AgentMemory：编程 Agent 记忆，不宜作为个人大脑主库

**已核验方向。**README 明确扩展 Karpathy LLM Wiki 模式，强调置信度、生命周期、图谱、混合搜索及编程 Agent 自动捕获。完整服务依赖 iii runtime；MCP shim 离线回退和完整在线工具面存在差别。不能把“无需外部数据库”理解为“只有几个 Markdown 文件”。[README](https://github.com/rohitg00/agentmemory/blob/e04ba88819c365c9acf9d6661ea802143e728bd6/README.md)

**Obsidian 实际能力。**`mem::obsidian-export` 从 KV 读取 memories、lessons、crystals、sessions，生成 frontmatter、wikilinks 和 MOC 索引，输出到受 export root 限制的目录。源码是导出流程，没有在该路径中将用户的 Vault 编辑读回内部状态；因此只能确认单向导出，不能确认双向同步。[obsidian-export.ts](https://github.com/rohitg00/agentmemory/blob/e04ba88819c365c9acf9d6661ea802143e728bd6/src/functions/obsidian-export.ts)

**适配判断。**可参考记忆来源、置信度与生命周期，或将其作为“编程会话来源”单独接入。全人生资料（邮件、家庭、健康、财务、阅读）的类型和来源治理并非其核心领域。本轮核验没有足够证据支持将其知识图谱称为形式化 Ontology 系统。

## 开发取舍与验收门槛（本报告建议）

| 问题 | 建议 |
|---|---|
| 谁保留原件？ | 自有资料目录；保留附件、来源 URL/原始 ID、获取时间、内容 hash。 |
| 谁是日常编辑入口？ | Obsidian；用户改笔记之后必须明确哪些事实重抽取、哪些人工编辑保留。 |
| 谁管理类型？ | 先用可读的个人词表：人、组织、项目、事件、资料、主张；关系如参与、属于、支持、反驳。 |
| 要几个运行时？ | 首版只选一个主索引/记忆引擎；Cognee 作为后续可替换服务，不是所有项目一起启动。 |
| 如何判断可靠？ | 同一资料重复导入不增加副本；改名不生成新人；原件更正能传到答案；撤回资料不再作为当前证据；备份可恢复。 |
| 中文要求？ | 用真实中文姓名、简称、多义词和长文做检索/实体合并测试；单独测中文关键词查询。 |

建议在真实个人样本上做小规模验证后再决定 fork 哪个项目：包含 Obsidian 笔记、邮件、网页、PDF、聊天各一组；每类安排更新、删除与冲突案例。当前阶段结论是“值得参考的架构和模块”，不是这些项目已经完成用户所需的全来源个人大脑。
