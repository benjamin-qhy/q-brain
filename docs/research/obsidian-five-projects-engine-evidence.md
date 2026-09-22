# Obsidian 个人大脑：GBrain 与 Cognee 源码核对

调研日期：2026-09-22。范围：本地克隆的文档与关键实现，只读研究，没有安装依赖、启动服务、运行测试或验证真实模型。以下“源码确认”仅说明存在实现，不等于效果、稳定性或安全性已验收。路径以仓库根目录为基准，行号对应本次快照。

| 项目 | Git HEAD | 根许可证 |
| --- | --- | --- |
| GBrain | `9b0a1b5ead75082ad202b38d4e578025462a88f2` | MIT，[gbrain/LICENSE:1](https://github.com/garrytan/gbrain/blob/9b0a1b5ead75082ad202b38d4e578025462a88f2/LICENSE#L1) |
| Cognee | `663a2dc15d04bc0d7ec2733a2dd604b7ed1b8c8e` | Apache-2.0，[cognee/LICENSE:1](https://github.com/topoteretes/cognee/blob/663a2dc15d04bc0d7ec2733a2dd604b7ed1b8c8e/LICENSE#L1) |

许可证结论仅限根项目；采用依赖、模型权重、第三方连接器时需分别核对其许可。

## 结论

GBrain 更接近“已有个人助理的长期记忆系统”：包含笔记同步、事实更正、撤回、来源隔离和检索。Cognee 更接近“把资料加工成图谱的引擎”：资料切块、实体关系提取、图谱和向量存储、检索及增量更新。

建议初版借鉴两者的数据治理设计，先建立 Obsidian 可读的资料和知识主库。两者均仅为开发参考，不作为现在或后期接入的引擎。我们自主实现资料加工、检索、更正和撤回，用个人中文资料验证自研能力的引用准确率、成本、删除和重建效果；第一阶段不开发本体。

## GBrain：核对结果

| 主题 | 核对结论 | 证据 |
| --- | --- | --- |
| 定位与运行 | 文档称使用 Bun，默认本地 PGLite，可扩展至 Postgres/pgvector；无需密钥的关键词路径与额外模型能力分开。并非 Obsidian 插件。 | [gbrain/README.md:76](https://github.com/garrytan/gbrain/blob/9b0a1b5ead75082ad202b38d4e578025462a88f2/README.md#L76)、`:176`、`:524` |
| 来源模型 | 文档定义 brain 为数据库，source 为同一数据库中的内容仓库；页面 slug 在 source 内唯一。source 是内容归属，不自动等于强安全边界。 | [gbrain/docs/architecture/brains-and-sources.md:7](https://github.com/garrytan/gbrain/blob/9b0a1b5ead75082ad202b38d4e578025462a88f2/docs/architecture/brains-and-sources.md#L7)、`:35`；[gbrain/AGENTS.md:7](https://github.com/garrytan/gbrain/blob/9b0a1b5ead75082ad202b38d4e578025462a88f2/AGENTS.md#L7) |
| 混合检索 | 源码含关键词与向量候选、RRF 融合、重排；部分候选分支失败时保留其余候选。因此一次成功回答不能证明所有检索分支健康。 | [gbrain/src/core/search/hybrid.ts:1498](https://github.com/garrytan/gbrain/blob/9b0a1b5ead75082ad202b38d4e578025462a88f2/src/core/search/hybrid.ts#L1498)、`:1990`、`:2200`、`:2327` |
| 权限检索 | 搜索实现处理 source 范围，远端调用过滤私有内容；仍需测试所有目标接口和配置，不能由单个函数推导系统完全隔离。 | [gbrain/src/core/ops/search.ts:216](https://github.com/garrytan/gbrain/blob/9b0a1b5ead75082ad202b38d4e578025462a88f2/src/core/ops/search.ts#L216)、`:254`、`:263` |
| Obsidian 链接 | 存在 `[[path]]`、带别名及带 source 限定的链接解析。不能据此称所有 Obsidian 链接、插件和附件语义完整兼容。 | [gbrain/src/core/link-extraction.ts:200](https://github.com/garrytan/gbrain/blob/9b0a1b5ead75082ad202b38d4e578025462a88f2/src/core/link-extraction.ts#L200)、`:213`、`:230` |
| 更正 | 事实支持“被另一条事实替代”的关系；实现拒绝自指、悬空和指向失效事实的引用，保留旧事实退出活跃视图的语义。 | [gbrain/src/core/facts/supersede-resolve.ts:5](https://github.com/garrytan/gbrain/blob/9b0a1b5ead75082ad202b38d4e578025462a88f2/src/core/facts/supersede-resolve.ts#L5)、`:55`、`:74` |
| 忘记 | 先记录持久撤回，再尽力修改原笔记或数据库页面，防止重新导入旧文件把撤回事实恢复。源文件历史、备份和撤回记录不是物理擦除。仅备份 Markdown 会遗漏数据库专属撤回记录。 | [gbrain/src/core/facts/forget.ts:1](https://github.com/garrytan/gbrain/blob/9b0a1b5ead75082ad202b38d4e578025462a88f2/src/core/facts/forget.ts#L1)、`:118`、`:142`、`:168` |
| 同步 | 源码有 Git 变更清单、删除和重命名路径处理；需要进一步验证现有 Vault 在非 Git、频繁编辑及多设备冲突下的行为。 | [gbrain/src/core/sync.ts:113](https://github.com/garrytan/gbrain/blob/9b0a1b5ead75082ad202b38d4e578025462a88f2/src/core/sync.ts#L113)、`:190`、`:215`、`:698` |

**可借鉴的重点**：统一来源身份、事实有效期与替代链、持久撤回、检索分支状态。**借鉴边界**：由我们实现受控写入和事件账本，不运行参考项目。自研数据库若保存不可重建的更正／撤回状态，必须将它纳入备份。

## Cognee：核对结果

| 主题 | 核对结论 | 证据 |
| --- | --- | --- |
| 对外接口 | `remember` 的持久记忆分支实际调用 add、cognify，并可继续 improve；不是仅在提示词里建议这样做。 | [cognee/cognee/api/v1/remember/remember.py:1888](https://github.com/topoteretes/cognee/blob/663a2dc15d04bc0d7ec2733a2dd604b7ed1b8c8e/cognee/api/v1/remember/remember.py#L1888)、`:1898`、`:1926` |
| 加工流程 | 默认文档路径依次分类、切块、抽取关系与摘要、存储图和向量。 | [cognee/cognee/api/v1/cognify/cognify.py:580](https://github.com/topoteretes/cognee/blob/663a2dc15d04bc0d7ec2733a2dd604b7ed1b8c8e/cognee/api/v1/cognify/cognify.py#L580)–`:611` |
| 存储复杂度 | 配置默认分别使用 SQLite、LanceDB、Ladybug。三类存储各有职责，也意味着备份、恢复、删除一致性需要跨存储检查。 | [cognee/cognee/infrastructure/databases/relational/config.py:22](https://github.com/topoteretes/cognee/blob/663a2dc15d04bc0d7ec2733a2dd604b7ed1b8c8e/cognee/infrastructure/databases/relational/config.py#L22)；`.../vector/config.py:32`；`.../graph/config.py:47` |
| 基本可追溯性 | chunk 保存来源文档关系、文档名/ID、内容哈希、顺序及分块策略；可作为回答定位原文的基础。 | [cognee/cognee/modules/chunking/models/DocumentChunk.py:32](https://github.com/topoteretes/cognee/blob/663a2dc15d04bc0d7ec2733a2dd604b7ed1b8c8e/cognee/modules/chunking/models/DocumentChunk.py#L32)–`:58` |
| 审计来源与冲突 | 更完整的 provenance ledger 和矛盾检测均为显式可选，默认关闭；不能声称安装后就自动完整溯源并解决冲突。 | [cognee/cognee/modules/cognify/config.py:16](https://github.com/topoteretes/cognee/blob/663a2dc15d04bc0d7ec2733a2dd604b7ed1b8c8e/cognee/modules/cognify/config.py#L16)–`:23`；[cognee/cognee/api/v1/cognify/cognify.py:614](https://github.com/topoteretes/cognee/blob/663a2dc15d04bc0d7ec2733a2dd604b7ed1b8c8e/cognee/api/v1/cognify/cognify.py#L614)–`:620` |
| 时间冲突 | 针对明确声明为单值的关系，可标记旧断言被新断言替代，默认不自动启用。它不能替代对“来源说法互相矛盾”的人工判断。 | [cognee/cognee/api/v1/cognify/cognify.py:622](https://github.com/topoteretes/cognee/blob/663a2dc15d04bc0d7ec2733a2dd604b7ed1b8c8e/cognee/api/v1/cognify/cognify.py#L622)–`:632` |
| 增量更新 | 实现按旧 chunk 和新文本差异更新；不支持的后端、分块策略或缺少旧基线会回退完整流程。不是任意文档修改都只付差异成本。 | [cognee/cognee/api/v1/update/incremental.py:1](https://github.com/topoteretes/cognee/blob/663a2dc15d04bc0d7ec2733a2dd604b7ed1b8c8e/cognee/api/v1/update/incremental.py#L1)–`:35`、`:126` |
| 删除 | forget 提供单条、数据集、全部及 memory_only 语义；memory_only 保留原文件，所以不是彻底删除。接口存在不代表已验证所有衍生数据及外部备份清除。 | [cognee/cognee/api/v1/forget/forget.py:22](https://github.com/topoteretes/cognee/blob/663a2dc15d04bc0d7ec2733a2dd604b7ed1b8c8e/cognee/api/v1/forget/forget.py#L22)–`:69` |
| 检索与接入 | 存在 chunk、摘要、图谱、混合、时间等检索类型及独立 FastMCP 服务。类型枚举不构成每种检索质量已通过验收的证据。 | [cognee/cognee/modules/search/types/SearchType.py:4](https://github.com/topoteretes/cognee/blob/663a2dc15d04bc0d7ec2733a2dd604b7ed1b8c8e/cognee/modules/search/types/SearchType.py#L4)；[cognee/cognee-mcp/src/server.py:12](https://github.com/topoteretes/cognee/blob/663a2dc15d04bc0d7ec2733a2dd604b7ed1b8c8e/cognee-mcp/src/server.py#L12)–`:18` |
| 模型边界 | auto 抽取器按配置选 LLM 或本地 GLiNER demo；本地 demo 需要额外依赖。“无云 API 密钥”不等于全部多模态处理和高质量中文分析无需模型资源。 | [cognee/cognee/modules/cognify/config.py:24](https://github.com/topoteretes/cognee/blob/663a2dc15d04bc0d7ec2733a2dd604b7ed1b8c8e/cognee/modules/cognify/config.py#L24)–`:29`；[cognee/README.md:160](https://github.com/topoteretes/cognee/blob/663a2dc15d04bc0d7ec2733a2dd604b7ed1b8c8e/README.md#L160)附近 |

**可借鉴的重点**：流水线任务、chunk 身份、图谱来源账本、增量更新回退。**借鉴边界**：资料分块、来源定位和增量回退由我们自主实现并测试，不安排 Cognee 集成。其图谱和本体相关实现保留为研究材料，不进入第一阶段。

## 对个人大脑架构的具体要求

1. 原文层保存稳定来源 ID、原始 URL/文件位置、获取时间、内容版本与隐私等级；AI 总结作为衍生版本保存。
2. 每条观点区分“原文声称”“AI 推断”“本人确认”，并链接到对应版本和原文片段。图谱里的关系不是天然事实。
3. 修改与删除形成事件：原件更新后重算受影响知识；撤回后检索、缓存、图谱和知识页同步处理，重建不能恢复已撤回内容。
4. Obsidian 人工内容和 AI 生成内容采用明确所有权；先产生可审阅改动，再合并到长期知识页。维护来源、实体别名和重名处理规则。
5. 自研索引模块使用统一内部接口，至少提供 ingest、search、get_evidence、invalidate、delete 和 rebuild；不为两个参考项目建设集成适配器。
6. 阶段验收使用真实中文资料：能找到且准确引用、能区分历史事实与现状、冲突能显示、删除不复活、失败可重试、备份可恢复。未经这些测试，不宣布“大脑已建成”。
