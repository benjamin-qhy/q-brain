# Obsidian / Wiki 三个参考项目：源码证据

调研日期：2026-09-22。范围：只读本地完整 clone，未安装、启动或运行测试。下列路径均相对本仓库根目录；行号定位本次 SHA 快照。文档说明不等于实测通过，提示词规则不等于程序强制约束。

| 项目 | 本地 HEAD | 定位 |
|---|---|---|
| claude-obsidian | `32ac5a02c4e082e4a5628ca810776375e134708e` | Agent Skills + Python 核心 + Claude Code 适配器 |
| karpathy-llm-wiki | `ac46de1ad27f92b28ac95459c782c07f6b8c964a` | 设计思想文档，不是可运行应用 |
| llm_wiki | `e8082119649e6a8e1cf85eaf289adcabfdf39d4e` | Tauri 桌面知识库，Markdown 可由 Obsidian 打开 |

## 结论

建议借鉴 Karpathy 的“原材料—知识页—编纂规则”思想，借鉴 claude-obsidian 的来源/论断账本和可恢复写入设计，参考 llm_wiki 的导入、后台处理、审阅产品体验。三个项目仅供借鉴开发，不作为本产品现在或后期的运行组件。个人脑中枢应只有一个正式知识写入入口，避免多个 AI 同时改 Obsidian 文件。

## 1. Karpathy：架构思想，不是成品

- **文档明确**：[karpathy-llm-wiki/llm-wiki.md:5](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f/ac46de1ad27f92b28ac95459c782c07f6b8c964a#file-llm-wiki-md) 自称 idea file；`:73-75` 说明其抽象且未规定实现。
- **三层**：不可变原始来源、AI 维护的 Wiki、规定结构和流程的 schema，见 `:25-33`。不能把 AI 总结当作原件。
- **知识编译**：收到新来源时更新概念、人物、主题，并保留矛盾；问答中有价值的成果再沉淀入 Wiki，见 `:11-15`、`:35-41`。
- **导航与增量**：`index.md` 负责目录，`log.md` 记录历史，见 `:43-49`。文中约 100 个来源/数百页面只是作者经验，不是容量基准测试。
- **Obsidian**：Markdown、双链、frontmatter、图谱以及 Git，见 `:53-62`。
- **缺口**：没有连接器、账号同步、状态机、并发锁、来源删除、失败恢复的执行代码。所谓低维护成本属于作者主张，不能作为成本承诺。
- **许可**：本次 clone 仅见 `llm-wiki.md`，未见 LICENSE；只能确认缺少明确本地许可文件，不能推断可随意复制再分发。

## 2. claude-obsidian：证据与写入治理最值得借鉴

### 能做什么

- **项目约定**：源代码目录和用户 Vault 分开，Vault 含 `inbox/`、`.raw/`、`wiki/`、账本、运行元数据；明确 Obsidian Markdown 约定。见 [claude-obsidian/AGENTS.md:8-28,66-79](https://github.com/AgriciDaniel/claude-obsidian/blob/32ac5a02c4e082e4a5628ca810776375e134708e/AGENTS.md#L8)。
- **工作流文档**：摄入先计算哈希，辨认资料类型，判断是否值得形成知识页，再有限读取相关页，记录局部阅读范围，区分来源陈述与 AI 综合。见 [claude-obsidian/skills/wiki-ingest/SKILL.md:66-109](https://github.com/AgriciDaniel/claude-obsidian/blob/32ac5a02c4e082e4a5628ca810776375e134708e/skills/wiki-ingest/SKILL.md#L66)。这是 Agent 工作规约，并不保证每次模型完全遵循。
- **程序契约**：source ledger 与 claim ledger 分离；来源状态包括 active、superseded、rejected，论断包括 provisional、contested、unsupported、deprecated；证据关系支持 supports/contradicts/context。见 [claude-obsidian/claude_obsidian/ledgers.py:21-49](https://github.com/AgriciDaniel/claude-obsidian/blob/32ac5a02c4e082e4a5628ca810776375e134708e/claude_obsidian/ledgers.py#L21)。
- **程序强制**：每个写入须带预期哈希；`.raw/` 来源载荷只允许 create，唯一旧元数据例外为 `.raw/.manifest.json`。见 [claude-obsidian/claude_obsidian/transaction.py:3426-3448](https://github.com/AgriciDaniel/claude-obsidian/blob/32ac5a02c4e082e4a5628ca810776375e134708e/claude_obsidian/transaction.py#L3426)。
- **恢复代码**：设计采用进程持有锁、预期哈希、持久 journal、单文件原子替换与整项操作恢复；并不声称跨文件原子事务。回滚检测文件是否被外部修改，发现变化记录失败而不盲目覆盖。见同文件 `:1-6,4054-4107`。

### 不应误判的边界

- 媒体/OCR/音视频需要宿主能力或适配器，不是这个包天然都能处理，见 `skills/wiki-ingest/SKILL.md:46-60`。
- 来源改变应新建不可变快照，见同文件 `:62-64`；但这不等于已经有微信、邮件、浏览记录等自动采集连接器。
- 并行 Worker 只产草稿，统一提交，见同文件 `:95-97`。对需要手机/电脑同步的 Vault，仍须另行验证与外部编辑器及同步工具的竞争。
- 软件是本地优先，不等于所选 AI 模型在本地；远端模型数据路径须由我们自己的模型网关控制。
- LICENSE 首行是 MIT，见 [claude-obsidian/LICENSE:1-8](https://github.com/AgriciDaniel/claude-obsidian/blob/32ac5a02c4e082e4a5628ca810776375e134708e/LICENSE#L1)。这里只记录仓库声明，不作法律意见。

## 3. llm_wiki：完整工作台体验，但不是 Obsidian 插件

### 文档能力与源码核对

- **产品文档**：多格式输入、媒体、网页剪藏、目录监视、异步 Review、可选 LanceDB 向量检索、HTTP API/MCP，见 [llm_wiki/README.md:31-53](https://github.com/nashsu/llm_wiki/blob/e8082119649e6a8e1cf85eaf289adcabfdf39d4e/README.md#L31)。
- **Obsidian 兼容宣称**：Wiki 用双链和 YAML，目录可作为 Vault，见同文件 `:69-80`。技术栈是 Tauri/Rust + React，并非必须在 Obsidian 内运行的插件，见 `:383-393`。
- **目标与规则分开**：purpose.md 表示“为什么整理”，schema.md 表示“如何整理”，见 `:98-104`。适合个人脑中枢保存长期目标、当前项目、想解决的问题。
- **先分析再生成**：实现先得到结构分析，再生成页面，见 [llm_wiki/src/lib/ingest.ts:1025-1093](https://github.com/nashsu/llm_wiki/blob/e8082119649e6a8e1cf85eaf289adcabfdf39d4e/src/lib/ingest.ts#L1025)；Review 生成失败可降级后继续写页面，见 `:1140-1179`。因此异步审阅不等于所有知识更新都经用户事前批准。
- **增量缓存**：正文 SHA-256 及之前输出文件存在性判断，见 [llm_wiki/src/lib/ingest-cache.ts:55-98](https://github.com/nashsu/llm_wiki/blob/e8082119649e6a8e1cf85eaf289adcabfdf39d4e/src/lib/ingest-cache.ts#L55)。缓存条目只有 hash/timestamp/filesWritten，见 `:10-18`；不能据此宣称规则、模型、解析器变化后自动使缓存失效，我们设计应补充这些版本键。
- **并发边界**：按任务开始顺序预留提交轮次，允许准备并行，见 [llm_wiki/src/lib/ingest-commit-coordinator.ts:8-39](https://github.com/nashsu/llm_wiki/blob/e8082119649e6a8e1cf85eaf289adcabfdf39d4e/src/lib/ingest-commit-coordinator.ts#L8)。README 所述串行队列不应简单理解为整个处理流程绝无并发。
- **合并与修订**：已有页面合并来源数组与正文，单一来源页面在该来源修正时替换旧正文，多来源页走合并；见 [llm_wiki/src/lib/ingest.ts:2017-2055](https://github.com/nashsu/llm_wiki/blob/e8082119649e6a8e1cf85eaf289adcabfdf39d4e/src/lib/ingest.ts#L2017)。这是有价值的“撤回旧说法”处理，不应只永远追加。
- **恢复局限**：单页写入失败记录 hardFailures 并继续循环，见 `:2057-2065`；旧页备份写至 page-history，见 `:3090-3102`。这不是已证明的整批回滚事务，不能将“有备份/队列恢复”说成“跨页全有或全无”。
- **原始资料边界**：README 一处用 immutable 描述 raw，另一处明确监视 raw/sources 的修改与删除，见 `README.md:73,125-132`。我们的实现应另设追加式原件归档，不直接把可变监视目录当不可变证据库。
- **许可声明**：[llm_wiki/LICENSE:1-4](https://github.com/nashsu/llm_wiki/blob/e8082119649e6a8e1cf85eaf289adcabfdf39d4e/LICENSE#L1) 为 GPL v3 文本；复用代码和仅借鉴架构要分开评估，不能默认与 MIT 同样处理。

## 4. 应转化为本项目的架构决定（建议，并非上游既有事实）

1. Obsidian 用作阅读、记录、复核的统一入口；后台服务处理账号授权、拉取、解析、队列、编译与索引。
2. 原始资料独立追加保存，来源有稳定 ID、版本、哈希、获取时间、出处；大音视频保留在附件仓库，不让 Vault 堆积全部大文件。
3. “人物/项目/主题/决策”是知识视图；可追溯的论断与证据才是关键底层记录。一个页面列出 sources 不足以证明每句话。
4. 人写笔记与 AI 正式知识页分区管理，AI 改用户正文必须生成差异待审，避免把用户判断冲掉。
5. 单写入协调器提交内容、来源/论断关系、目录及日志，哈希防冲突，失败可恢复；搜索索引可重建，不作唯一真相。
6. Review 分级：普通摘要可自动归档；改变个人偏好、重要决定、财务/健康结论、矛盾证据、删除操作进入待审队列。
7. 加入版本失效机制：原件/解析器/schema/模型/编译器的变化都能定位受影响页面，避免只按原文哈希跳过更新。
8. 来源撤回先标记失效，再重新评估依赖论断与页面；依法或按用户要求删除时须遍历原件、抽取文本、向量、缓存、派生页和备份策略，不能承诺单删文件就彻底遗忘。

## 未完成的验证

未运行安装、单元测试、真实模型、摄入样本、Obsidian 打开与双向编辑、断电恢复、多设备同步、端到端删除或外部账号授权。因此本报告支持架构取舍，不能作为上述项目已适合直接上线的验收结果。
