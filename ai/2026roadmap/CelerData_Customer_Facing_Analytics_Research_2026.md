# CelerData / StarRocks 2026 Customer-Facing Analytics 研究报告

**编制人:** KC (kevin.chen@celerdata.com)
**日期:** 2026-02-28
**用途:** 2026 Customer-Facing Analytics Roadmap 规划前置研究

---

## 一、用户画像 (User Personas)

### 1.1 企业级 SaaS 平台 — 嵌入式分析型

**代表客户:** Demandbase, Clio, BambooHR, Herdwatch

这类客户将 StarRocks/CelerData 作为其 SaaS 产品的分析引擎，直接面向终端用户提供数据洞察。他们的核心诉求是**低延迟查询 + 高并发 + 多租户隔离**。

- **Demandbase** (ABM 平台): 从 Druid 迁移至 StarRocks。核心场景是为 B2B 营销客户提供实时 account-level analytics dashboard。关键触发点是 Druid 运维复杂且成本高。对 sub-second 查询延迟和上千并发用户有硬性要求。
- **Clio** (法律 SaaS): 为律师事务所提供案件管理分析。替代了 Trino/Athena 方案。核心需求是简化 pipeline（去 Flink）和 MySQL 兼容性，便于现有团队快速上手。
- **BambooHR** (HR SaaS): 为中小企业 HR 部门提供人力资源分析报表。从 Snowflake 迁移，主要驱动力是成本优化和查询延迟改善。
- **Herdwatch** (农业 SaaS): 爱尔兰农业管理平台，为农场主提供牲畜和合规分析。从 BigQuery 迁移，核心需求是成本效率和 real-time ingestion。

**画像特征:**
- 团队规模: 通常 2-5 名数据工程师负责分析后端
- 终端用户量: 数千到数万 dashboard 用户
- 关键决策因素: TCO、查询延迟 < 100ms p99、pipeline 简洁性、MySQL 兼容
- 常见替代方案: Druid, Trino/Athena, Snowflake, BigQuery

---

### 1.2 超大规模平台 — 基础设施型

**代表客户:** Intuit (含 Mailchimp), Expedia, Coinbase, Pinterest

这类客户是 CelerData 最大的企业客户，StarRocks 在其基础设施中承担关键数据服务角色，面向内部和外部多个团队/产品。需求复杂度和运维要求极高。

- **Intuit** (金融科技巨头): CelerData 最大客户之一。StarRocks 支撑 Intuit 多个产品线的实时分析，包括 Mailchimp 的 email campaign 分析。已经历过 S0 级别生产事故（24小时宕机），近期有 26 个 S1/S2 事件。正在探索 Agentic AI 用例。具体需求包括:
  - Flink Connector 多表事务 (POST-997)
  - 跨 Shared Data 集群复制 (POST-634)
  - AWS Lake Formation 权限集成 (POST-1018)
  - 4.0 版本 Multi-AZ 部署 (POST-1121)
  - 扩缩容期间性能降级优化 (POST-1120)
  - 配置暴露 (POST-1199, 紧急)
  - CNGroup Cache 功能

- **Expedia** (在线旅游): 4 个并行 initiative — sugarBI（商业智能平台）、TPSP（第三方服务平台）、EGMP（全球营销平台）、EGAP（全球广告平台）。多团队、多场景同时推进。
- **Coinbase** (加密货币交易所): 为交易用户提供实时市场分析和交易洞察。替代了自建 Postgres + Redis 方案。核心需求是实时性和高并发。
- **Pinterest** (社交媒体): 为广告主提供 campaign analytics。关键需求是 lakehouse 集成 (Iceberg) 和查询性能。

**画像特征:**
- 团队规模: 10-50+ 数据/平台工程师
- 使用规模: 数十 TB 到 PB 级数据，千级并发
- 关键决策因素: 企业级稳定性、Multi-AZ HA、细粒度安全/权限、扩缩容能力、与现有云基础设施深度集成
- 运维要求: SLA 99.99%+, 7x24 支持, 事故响应 < 15 分钟

---

### 1.3 金融 / 受监管行业 — 安全合规型

**代表客户:** 光大银行, 华润信托, 微众银行

中国金融机构客户群，对安全、权限、审计有极高要求。部署通常在私有云或混合云环境。

- **光大银行**: 需求集中在认证和权限——LDAP + Stream Load 认证集成、Debezium Routine Load (Avro) 支持、Kerberos 多 Catalog 认证、Manager 并行启停和断点续传。
- **华润信托**: 需要用户级别的资源隔离监控（不仅是 Resource Group 级别），以满足不同信托产品的资源审计需求。
- **微众银行**: 物化视图 (MV) 数据质量问题 (SR-34319)，影响其面向内部用户的报表准确性。

**画像特征:**
- 部署方式: 私有云/混合云，极少公有云
- 关键决策因素: 安全认证集成 (LDAP/Kerberos/Ranger)、审计日志、资源隔离、数据质量保证
- 监管要求: 等保三级/四级、SOC2、数据不出境

---

### 1.4 高增长科技公司 — 敏捷分析型

**代表客户:** Kaspi, BingX, Eightfold AI, Celonis

快速增长的科技公司，数据规模急剧扩大，需要分析基础设施跟上业务增长节奏。

- **Kaspi** (中亚超级应用): 22K+ 张表，面临元数据规模管理挑战。需求包括 StarRocks Catalog（跨集群查询）、全局索引可见性（大小/基数）、Resource Group 展示优化。
- **BingX** (加密货币交易所): 存算分离场景下的 Warehouse 熔断机制 (SR-35649)，防止大查询影响整体稳定性。
- **Eightfold AI** (AI 人才平台): 关注 AI 集成能力和实时人才分析。
- **Celonis** (流程挖掘): 对大规模 process mining 数据的分析性能有极高要求。

**画像特征:**
- 数据增长速度: 月均 20-50%
- 关键决策因素: 可扩展性、自助式运维、成本可预测、新功能快速采纳
- 技术偏好: 存算分离、Kubernetes 原生、API 优先

---

### 1.5 企业客户综合特征矩阵

| 维度 | SaaS 嵌入型 | 超大规模平台 | 金融合规型 | 高增长科技 |
|------|------------|------------|----------|----------|
| 首要关注 | 查询延迟 + 多租户 | 稳定性 + 安全 | 权限 + 合规 | 扩展性 + 成本 |
| 终端用户 | 外部 SaaS 用户 | 内部+外部混合 | 内部分析师 | 内部+产品分析 |
| 部署方式 | BYOC 公有云 | BYOC 多区域 | 私有云/混合 | BYOC 公有云 |
| 迁移来源 | Druid/Trino/Snowflake | 自建/混合 | 传统 MPP | ClickHouse/自建 |
| 典型预算 | $5K-50K/月 | $100K-500K+/月 | 项目制 | $10K-100K/月 |

---

## 二、Use Cases and User Stories

### 2.1 嵌入式实时 Dashboard

**场景描述:** SaaS 产品将 StarRocks 驱动的分析仪表盘嵌入其应用中，终端用户（非技术人员）通过 Web UI 查看实时数据洞察。

**User Stories:**
- *"作为 Demandbase 的 B2B 营销经理，我需要实时查看 target account 的 engagement score 和 intent signals，以便在客户有购买意向时及时跟进。查询响应必须在 1 秒内。"*
- *"作为 Herdwatch 的农场主，我需要查看本月牲畜健康趋势和合规报告，即使在 4G 网络下也要快速加载。"*
- *"作为 BambooHR 的 HR 主管，我需要按部门查看员工离职率、薪酬分布等人力分析，支持下钻到个人级别。"*

**技术要求:**
- P99 查询延迟 < 100ms
- 并发支持 1000+ 同时在线用户
- 多租户数据隔离 (Row-Level Security 或 Schema/DB 级隔离)
- API/JDBC 接口稳定可靠
- MySQL 协议兼容（降低集成门槛）

---

### 2.2 实时数据管道 (无 Flink 架构)

**场景描述:** 客户利用 StarRocks 的 Routine Load、Stream Load 等内置数据摄入能力，直接从 Kafka 等消息队列消费数据，无需维护独立的 Flink/Spark Streaming 集群。

**User Stories:**
- *"作为 Clio 的数据工程师，我需要将案件事件流从 Kafka 实时写入 StarRocks，而不需要维护 Flink 作业，因为我们的小团队没有流处理专家。"*
- *"作为光大银行的数据平台运维，我需要通过 Debezium CDC + Routine Load (Avro) 将 Oracle 变更实时同步到 StarRocks，且认证必须与现有 LDAP 体系打通。"*

**技术要求:**
- Routine Load 稳定性和性能 (已成为竞争优势，多个客户因此避免 Flink)
- Debezium Avro 格式支持
- Stream Load 认证集成 (LDAP, Kerberos)
- 断点续传和错误恢复
- 数据一致性保证 (at-least-once → exactly-once)

---

### 2.3 Lakehouse 联合分析

**场景描述:** 客户在数据湖 (Iceberg, Delta Lake, Hudi) 中存储历史冷数据，用 StarRocks External Catalog 进行联合查询，实现热温冷数据分层。

**User Stories:**
- *"作为 Pinterest 的数据工程师，我需要用 StarRocks 查询 Iceberg 中的历史广告投放数据，同时 join 实时的 campaign performance 指标，为广告主提供完整的分析视图。"*
- *"作为 Intuit 的平台架构师，我需要通过 AWS Lake Formation 统一管理 StarRocks 对 S3 数据湖的访问权限 (POST-1018)，确保合规审计。"*
- *"作为 Kaspi 的数据团队 lead，我需要用 StarRocks Catalog 跨集群查询 22K+ 张表，实现不同业务域数据的联合分析。"*

**技术要求:**
- External Catalog 性能优化 (大规模表元数据管理)
- AWS Lake Formation 权限集成
- Iceberg Table Compaction
- 跨集群 StarRocks Catalog
- 统一 SQL 语义

---

### 2.4 多租户运营与资源隔离

**场景描述:** 平台型客户需要为不同租户/业务线提供独立的计算资源和数据隔离，同时保持运维统一性。

**User Stories:**
- *"作为 Expedia sugarBI 的平台负责人，我需要为不同酒店合作伙伴提供独立的分析环境，确保查询性能互不影响，且每个伙伴只能看到自己的数据。"*
- *"作为华润信托的 IT 管理员，我需要按用户级别（而非 Resource Group 级别）监控资源使用，以便为每个信托产品线精确核算计算成本。"*
- *"作为 BingX 的 DBA，我需要 Warehouse 熔断机制，当某个大查询消耗过多资源时自动限流，防止影响其他交易分析任务。"*

**技术要求:**
- 多种隔离模式: RLS, Schema 隔离, Database-per-tenant
- Resource Group → 用户级别的细粒度资源监控
- Warehouse 熔断/限流
- 多 Warehouse 弹性扩缩容
- 租户级别的 QoS 保证

---

### 2.5 企业级高可用与灾备

**场景描述:** 对于关键业务负载，客户要求 StarRocks 集群具备多可用区容灾、自动故障转移、零停机升级等企业级能力。

**User Stories:**
- *"作为 Intuit 的 SRE 负责人，经历 S0 事故（24小时宕机）后，我需要 Multi-AZ 部署 (POST-1121) 确保单个 AZ 故障不影响业务。"*
- *"作为平台工程师，我需要在集群扩缩容时最小化性能降级 (POST-1120)，实现 node-level scaling + cache 感知调度。"*

**技术要求:**
- Multi-AZ 集群部署 (4.0 版本规划)
- DW HA (SR-37274, 进行中)
- 扩缩容期间性能优化 (cache-aware scheduling)
- 跨 Shared Data 集群复制 (POST-634)
- 配置管理透明化 (POST-1199)

---

### 2.6 安全、权限与合规

**场景描述:** 客户要求与现有企业身份管理系统集成，支持细粒度权限控制，满足行业监管要求。

**User Stories:**
- *"作为光大银行的安全管理员，我需要 StarRocks 与 LDAP/Kerberos 统一认证，包括 Stream Load 接口，避免维护独立账号体系。"*
- *"作为 Intuit 的安全工程师，我需要 AWS Lake Formation 权限与 StarRocks 打通，实现 S3 数据访问的统一鉴权。"*

**技术要求:**
- LDAP/Kerberos 全链路认证 (含数据写入接口)
- Ranger 权限集成
- SAML 2.0 for Manager
- SOC2 合规 (JIRA: 17 个 critical/high 安全项)
- 审计日志完整性

---

### 2.7 Agentic AI 与智能分析 (新兴)

**场景描述:** 客户探索将 LLM/AI Agent 与 StarRocks 集成，实现自然语言查询、智能告警、自动化分析报告等能力。

**User Stories:**
- *"作为 Intuit 的 AI 产品经理，我需要让 AI Agent 通过自然语言查询 StarRocks 数据，为 QuickBooks 用户自动生成财务洞察报告。"*（Intuit 已在讨论 Agentic AI use cases）
- *"作为 SaaS 公司产品经理，我希望终端用户能通过 chat 界面提问，后台 AI 自动生成 SQL 查询 StarRocks 并返回可视化结果。"*

**技术要求:**
- Text-to-SQL / NLQ 接口
- information_schema 完整性 (支持 AI 理解表结构)
- Vector Search 集成 (语义搜索)
- 查询结果流式返回 (支持 Agent 实时响应)
- Python UDF (支持 ML model serving)

---

## 三、用户痛点 (User Pain Points)

### 3.1 🔴 稳定性与可靠性 (P0 — 最严重)

**问题描述:** 多个大客户遭遇严重生产事故，直接威胁客户关系和合同续约。

**具体案例:**
- **Intuit S0 事故:** 24小时完全宕机，导致所有依赖 StarRocks 的 Intuit 产品中断。事后回顾暴露代码质量问题。近期还有 26 个 S1/S2 级别事件。这是 CelerData 近年来最严重的客户事故。
- **Intuit Mailchimp 生产事件:** Field Engineering 团队特别关注的事件，导致了 SA/SE 交接流程和跨团队事故协调的改进讨论。
- **扩缩容性能降级 (POST-1120):** Intuit 反馈集群扩缩容期间查询性能显著下降，影响用户体验。

**影响范围:** 所有大型企业客户
**客户情绪:** 焦虑、信任受损
**建议优先级:** P0 — 必须在 2026 H1 全面解决

---

### 3.2 🟠 运维复杂度 (P1)

**问题描述:** 客户反馈集群运维操作不够自动化和透明，需要大量手动干预。

**具体痛点:**
- **配置管理不透明 (POST-1199):** Intuit 紧急要求暴露配置信息，当前配置变更缺乏可视化和审计。
- **Manager 并行启停:** 光大银行需要同时管理多个集群的启停操作，当前串行执行效率低。
- **断点续传缺失:** 大规模数据加载中断后需要从头开始，浪费大量时间。
- **存算分离采纳难:** 很多中国客户缺乏对象存储/HDFS 基础设施，导致存算分离方案落地困难。

**影响范围:** 所有客户，尤其是运维团队
**建议优先级:** P1 — 持续改进

---

### 3.3 🟠 多租户与资源管理不足 (P1)

**问题描述:** 现有 Resource Group 机制粒度不够，无法满足精细化多租户管理需求。

**具体痛点:**
- **用户级资源监控缺失 (华润信托):** 只有 Resource Group 级别的监控，无法按用户追踪资源消耗。
- **Warehouse 熔断缺失 (BingX, SR-35649):** 大查询可以不受限地消耗 warehouse 资源。
- **22K+ 表的元数据管理 (Kaspi):** Resource Group 展示在超大规模元数据下性能退化。
- **多租户实现标准缺失 (SR-36910):** JIRA 中有多租户实现计划但尚未完成。

**影响范围:** 所有多租户场景客户
**建议优先级:** P1 — 2026 Roadmap 核心方向

---

### 3.4 🟡 数据质量与一致性 (P2)

**问题描述:** 物化视图、数据写入等环节存在数据质量问题。

**具体痛点:**
- **物化视图数据不一致 (微众银行, SR-34319):** MV 数据与原始表不一致，影响报表准确性。
- **Flink Connector At-Least-Once (POST-997):** Intuit 需要多表事务支持以确保数据一致性。
- **权限规划不完善 (SR-36856):** Feishu 讨论中提到权限体系需要全面规划。

**影响范围:** 对数据准确性要求高的客户
**建议优先级:** P2 — 持续修复

---

### 3.5 🟡 生态集成不足 (P2)

**问题描述:** 与客户现有技术栈的集成还有明显空白。

**具体痛点:**
- **AWS Lake Formation 权限未打通 (POST-1018):** Intuit 等 AWS 重度用户需要统一权限管理。
- **LDAP/Kerberos 全链路支持不完整 (光大银行):** Stream Load 等接口缺乏企业认证集成。
- **SAML 2.0 for Manager:** 多个客户请求。
- **Zendesk/JIRA 集成:** Field Engineering 内部也在讨论工具整合。
- **Python UDF:** 多个客户需要在查询中直接调用 Python 逻辑。

**影响范围:** 企业级客户
**建议优先级:** P2 — 按客户优先级排序

---

### 3.6 🟡 存算分离成熟度 (P2)

**问题描述:** 存算分离作为架构方向受到认可，但实际落地仍有多个痛点。

**具体痛点:**
- **客户采纳障碍:** 很多客户（尤其中国市场）缺乏对象存储，导致采纳困难。
- **Cache 管理:** CNGroup cache 功能和 cache replica 问题 (Intuit)。
- **Multi-AZ 支持缺失:** 4.0 才规划 (POST-1121)，当前单 AZ 部署无法满足企业级灾备需求。
- **跨集群复制 (POST-634):** 关键企业级功能尚未完成。

**影响范围:** 采用存算分离架构的客户
**建议优先级:** P2 — 4.0 版本重点

---

## 四、行业在这个领域的发展方向 (Industry Development Direction)

### 4.1 市场规模与增长趋势

嵌入式分析市场正处于爆发增长期：

- **2025 年市场规模约 $234 亿**，预计 **2035 年达到 $1,010 亿** (CAGR ~15.74%)
- 超过 **80% 的软件供应商** 计划在 2026 年前将 GenAI 能力嵌入产品
- 驱动力: SaaS 普及、数据驱动决策文化、AI/ML 集成需求

### 4.2 竞争格局变化

#### ClickHouse — 开源霸主策略
- 2024 年估值 $150 亿，资金充裕
- 推出 **chDB**: 嵌入式 ClickHouse（进程内分析引擎），可直接嵌入 Python/Go/Node.js 应用
- 开源社区势能强劲，GitHub stars 遥遥领先
- **对 CelerData 的威胁:** 在开发者市场和中小客户群有替代风险

#### Snowflake — AI 原生转型
- 推出 **Cortex Agents**: 基于 LLM 的企业数据分析 Agent
- **Snowflake Native Apps**: 第三方可在 Snowflake 平台上构建和分发分析应用
- 从 "Data Warehouse" 向 "Data Cloud + AI Platform" 转型
- **对 CelerData 的启示:** 平台化和 AI 集成是大势所趋

#### Databricks — 嵌入式仪表盘
- 推出**原生嵌入式 Dashboard**，按计算量计费（非按用户数）
- 定价模型对高并发场景有吸引力
- Lakehouse 架构继续深化 (Unity Catalog, Delta Lake 4.0)
- **对 CelerData 的启示:** 计算模型定价可能更适合嵌入式分析场景

#### Apache Pinot / StarTree
- 专注实时分析 OLAP，LinkedIn 出品
- StarTree 商业化加速，获新一轮融资
- **对 CelerData 的定位:** 最直接的技术竞品

#### Rockset → OpenAI 收购
- 2024 年被 OpenAI 收购，验证了 **"实时分析 + AI" 的融合趋势**
- 表明顶级 AI 公司认可实时分析在 AI 应用中的核心价值

### 4.3 关键技术趋势

#### 4.3.1 Agentic Analytics (AI Agent 驱动的分析)

这是 2026 年最重要的行业趋势。传统 BI dashboard 正在被 AI Agent 增强甚至替代：

- **Text-to-SQL / NLQ (Natural Language Query):** 用户通过自然语言提问，AI 自动生成 SQL 并返回可视化结果
- **自主分析 Agent:** AI 不仅回答问题，还主动发现数据中的异常和趋势
- **对话式分析:** 多轮对话式数据探索，而非静态 dashboard
- **行业案例:** Intuit 已在内部讨论 Agentic AI use cases（Feishu 记录）

**对 StarRocks 的要求:**
- 完善的 information_schema（让 AI 理解数据结构）
- 低延迟查询（支持 Agent 实时交互）
- Vector Search 集成（语义搜索能力）
- Streaming 查询结果（支持 Agent 流式响应）

#### 4.3.2 真正的多租户 (True Multi-Tenancy)

行业最佳实践正在从简单的 Row-Level Security 向更成熟的多租户架构演进：

| 模式 | 隔离级别 | 适用场景 | 代表厂商做法 |
|------|---------|---------|-------------|
| RLS (行级安全) | 逻辑隔离 | 轻量级 SaaS | Snowflake Row Access Policy |
| Schema 隔离 | 中等隔离 | 中型 SaaS | Databricks Unity Catalog |
| Database-per-tenant | 强隔离 | 企业/合规场景 | ClickHouse Cluster Multi-Tenant |
| 计算隔离 (Warehouse) | 最强隔离 | 大客户独占 | Snowflake Virtual Warehouse |

**行业方向:** 支持在单一平台上灵活选择不同隔离级别，并提供统一的管理界面。

#### 4.3.3 Data Apps (数据应用化)

传统 "数据 → 报表 → 人工决策" 的模式正在被 "数据 → 智能应用 → 自动化行动" 取代：

- 嵌入式分析不再是 "看板"，而是产品核心功能的一部分
- Real-time alerts + automated actions
- 客户期望: 从 "看到数据" 到 "数据驱动行动" 的闭环

#### 4.3.4 性能基准提升

行业客户对嵌入式分析的性能期望持续提升：

- **查询延迟:** P99 < 100ms (从 "Nice to have" 变为 "Must have")
- **并发用户:** 1000+ 同时在线（中大型 SaaS 的基本要求）
- **数据新鲜度:** 秒级到分钟级实时（从 T+1 向 real-time 迁移）
- **冷启动时间:** 扩容节点 < 60 秒开始服务

### 4.4 CelerData 战略机会

基于以上分析，CelerData 在 2026 年 customer-facing analytics 方向的核心机会:

1. **Agentic Analytics Integration:** 提供 AI-ready 的分析基础设施（完善的 metadata, vector search, streaming results），使 SaaS 客户能轻松构建 AI 驱动的分析功能。

2. **Multi-Tenancy as a Service:** 将多租户从 "自己用 SQL 实现" 升级为平台原生能力，提供 RLS/Schema/DB 多种隔离模式的开箱即用方案。

3. **Zero-Flink Real-Time Pipeline:** 继续深化 Routine Load 和 Stream Load 的优势，将 "去 Flink" 作为核心卖点，降低客户实时数据管道的运维成本。

4. **Enterprise Resilience:** Multi-AZ、DW HA、graceful scaling 等企业级能力是进入 Fortune 500 的门票。

5. **Lakehouse-Native Analytics:** 深化 Iceberg/Delta Lake 集成，支持 AWS Lake Formation 等云原生权限模型，成为 Lakehouse 生态的首选分析引擎。

---

## 五、Feature Gap 矩阵 — 按客户需求密度排序

本节综合 Feishu 客户需求、JIRA (300+ issues, 含 Improvement/Feature/Bug)、StarRocks 代码库分析和行业竞品对比，按 feature area 汇总需求密度和差距程度。

### 5.1 Feature Area 需求热力图

| Feature Area | JIRA Issues | 客户提及数 | 提及客户 | 竞品差距 | 优先级建议 |
|---|---|---|---|---|---|
| **查询性能 / 优化** | 43 | 5+ | Intuit, Kaspi, Expedia, Coinbase, BingX | 低 (核心优势) | 持续投入 |
| **多租户 / 资源隔离** | 3 (JIRA) | 6+ | 华润信托, BingX, Expedia, Demandbase, Intuit, Kaspi | **高** (vs Snowflake RLS/DDM) | **P0 — 核心差距** |
| **安全 / 认证 / 合规** | 15 | 4+ | 光大银行, Intuit (Lake Formation), Eightfold, 华润信托 | 中 (Ranger 部分覆盖) | P1 |
| **数据摄入 (Routine/Stream/Flink)** | 16 | 5+ | Clio, 光大银行, Intuit (POST-997), Kaspi, 宁波银行 | 低 (竞争优势) | 持续深化 |
| **Lakehouse / Iceberg / Catalog** | 18 | 4+ | Pinterest, Intuit (POST-1018), Kaspi (SR Catalog), Expedia | 中 (vs Databricks Unity) | P1 |
| **高可用 / 灾备** | 11 | 3+ | Intuit (POST-1121, S0), Kaspi (SR-37381), Expedia | 中 (Multi-AZ 规划中) | P0 |
| **监控 / 可观测性** | 14 | 4+ | 华润信托, Intuit (POST-1199), SR-36967, Kaspi | 中 | P1 |
| **存算分离 / Warehouse** | 10 | 4+ | BingX (熔断), Intuit (cache/scaling), 多个中国客户 | 低-中 | P1 |
| **物化视图** | 8 | 2+ | 微众银行 (数据质量), 多个客户 | 低 | P2 |
| **AI / ML 集成** | 2 (JIRA) | 2+ | Intuit (Agentic AI), 行业趋势驱动 | **高** (vs Snowflake Cortex/Databricks AI) | **P0 — 战略差距** |
| **多集群 / 跨集群** | 2 | 2+ | Intuit (POST-634), Kaspi (SR Catalog) | 中 | P1 |
| **嵌入式模式 (Embedded)** | 0 | 0 (行业需求) | 行业趋势 (chDB/DuckDB) | **高** | P2 — 长线 |
| **数据治理 / 血缘** | 0 | 1 (隐性) | 企业级客户隐性需求 | **高** (vs Unity Catalog) | P2 |

### 5.2 关键 Feature Gap 详细分析

#### 🔴 Gap 1: 原生多租户能力 (Native Multi-Tenancy)

**现状:** StarRocks 通过 Resource Group 提供基础资源隔离，RLS 依赖 Apache Ranger 插件。
**竞品对比:**
- Snowflake: 原生 Row Access Policy (SQL 级声明)、Dynamic Data Masking、Virtual Warehouse 隔离
- Databricks: Unity Catalog 多工作区治理模型
- ClickHouse: Cluster-level Multi-Tenant 部署

**差距明细:**

| 能力 | StarRocks | Snowflake | Databricks | ClickHouse |
|------|-----------|-----------|------------|------------|
| Row-Level Security | Ranger 插件 | 原生 SQL | Unity Catalog | 自定义实现 |
| Dynamic Data Masking | Ranger 插件 | 原生 SQL | 列级策略 | 无 |
| 用户级资源监控 | ❌ | ✅ | ✅ | 部分 |
| 租户级 QoS | Resource Group | Virtual Warehouse | Serverless | 集群隔离 |
| Tag-Based Access | 计划中 (#67458) | ✅ | ✅ | 无 |

**客户影响:** 华润信托 (用户级监控)、BingX (Warehouse 熔断)、Expedia (多租户 BI)、Demandbase (SaaS 隔离)、Intuit (多团队隔离)、Kaspi (Resource Group 性能)
**JIRA 关联:** SR-36910 (Multi-tenant implementation)、SR-35649 (Warehouse 熔断)
**建议:** 将 RLS/DDM 从 Ranger 依赖升级为 StarRocks 原生 SQL 支持，增加用户级资源追踪。

---

#### 🔴 Gap 2: AI/ML 原生集成 (AI-Native Analytics)

**现状:** StarRocks 3.4+ 有 vector search (HNSW/IVFPQ)，有 Python UDF，ai_query() 在开发中 (#61551)。
**竞品对比:**
- Snowflake Cortex: 原生 LLM 函数 (COMPLETE, EXTRACT, SUMMARIZE)、ML 训练/预测
- Databricks: 原生 AI Functions、ML Runtime、Feature Store、向量搜索
- ClickHouse: 无原生 AI，但 chDB 可嵌入 Python ML 工作流

**差距明细:**

| 能力 | StarRocks | Snowflake Cortex | Databricks |
|------|-----------|-----------------|------------|
| Vector Search | ✅ (v3.4+) | ✅ | ✅ |
| Text-to-SQL | ❌ | ✅ (Cortex Agents) | ✅ (AI Functions) |
| 原生 ML 函数 | ❌ | ✅ | ✅ |
| Embedding 生成 | Python UDF | 原生函数 | 原生函数 |
| LLM 集成 | ai_query() (开发中) | Cortex | Foundation Models |
| 流式结果 | 部分 | ✅ | ✅ |

**客户影响:** Intuit (已讨论 Agentic AI use cases)、所有 SaaS 客户 (NLQ 趋势)
**JIRA 关联:** #61551 (ai_query)、#46678 (vector index)
**建议:** 加速 ai_query() 落地，提供 information_schema 增强（让 LLM 理解表结构），支持查询结果 streaming。

---

#### 🟠 Gap 3: 企业级高可用 (Enterprise HA)

**现状:** DW HA 进行中 (SR-37274)，Multi-AZ 计划在 4.0 (POST-1121)，集群快照已支持 (v3.5)。
**差距:**
- Multi-AZ 尚未落地（Intuit S0 事故后最迫切需求）
- 扩缩容期间性能降级 (POST-1120) 未解决
- 跨 Shared Data 集群复制 (POST-634) 未完成

**客户影响:** Intuit (S0 + 26 个 S1/S2)、Kaspi (failover)、Expedia (多区域部署)
**建议:** Multi-AZ + graceful scaling 是 4.0 的 must-have，不可延期。

---

#### 🟠 Gap 4: 数据治理与血缘 (Data Governance & Lineage)

**现状:** 有 AuditLoader 审计日志，无原生数据目录或血缘追踪。
**竞品:** Databricks Unity Catalog 提供跨工作区的数据发现、列级血缘、标签管理。Snowflake 有 Access History 和 Object Dependencies。

**差距:** 缺少列级血缘追踪、数据分类标签、数据质量指标、统一数据目录。
**客户影响:** 所有企业级客户的隐性需求，尤其是金融合规客户。
**建议:** 可先做轻量级 table→table 血缘和 data classification tags，不必一步到位做完整 catalog。

---

#### 🟠 Gap 5: 安全认证全链路覆盖 (End-to-End Auth)

**现状:** v3.5 增加了 JWT/OAuth 支持，LDAP 基础集成可用，Ranger 提供 RBAC。
**差距明细:**
- Stream Load 缺少 LDAP 认证集成（光大银行）
- Kerberos 多 Catalog 认证未完善（光大银行）
- AWS Lake Formation 权限未打通（Intuit POST-1018）
- SAML 2.0 for Manager 缺失（多客户需求）
- Masking Policy 展示不完整 (SR-36131)

**客户影响:** 光大银行 (4项)、Intuit (Lake Formation)、多个中国金融客户
**JIRA 关联:** SR-36131, POST-1018, 15 个安全相关 issue
**建议:** 逐客户推进：先解决光大银行 LDAP 全链路，再推 Lake Formation。

---

#### 🟡 Gap 6: 嵌入式模式 (Embedded Mode)

**现状:** StarRocks 是分布式集群架构，无单进程嵌入模式。
**竞品:**
- ClickHouse chDB: 可嵌入 Python/Go/Node.js，进程内分析引擎
- DuckDB: SQLite 式嵌入，零部署
- 这两个产品在开发者社区获得了极大关注

**差距:** 无法作为 library 分发，无 WASM 支持，无单机模式。
**客户影响:** 当前客户无直接需求，但行业趋势明确。
**建议:** 长线规划，可先观察市场反应，2026 不急于投入。

---

#### 🟡 Gap 7: 原生调度 (Native Task Scheduling)

**现状:** 物化视图有 REFRESH ASYNC EVERY 调度，无通用 task scheduler。
**竞品:** Snowflake Streams & Tasks 提供原生 CDC + 调度编排。
**差距:** 客户必须依赖 Airflow/Prefect 等外部工具做 ETL 编排。
**客户影响:** 增加了客户基础设施复杂度。
**建议:** 可在存量 MV 调度基础上扩展为通用 task 调度，降低客户对外部工具的依赖。

---

### 5.3 Feature Gap 与客户需求交叉矩阵

下表展示每个 feature gap 影响的客户列表，帮助优先级决策：

| Feature Gap | Intuit | Expedia | Demandbase | 光大银行 | Kaspi | BingX | 华润信托 | 微众银行 | Coinbase | Pinterest | Clio | 总计 |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| 多租户原生化 | ✅ | ✅ | ✅ | | ✅ | ✅ | ✅ | | | | | **6** |
| AI/ML 集成 | ✅ | | | | | | | | ✅ | | | **2+趋势** |
| 企业级 HA | ✅ | ✅ | | | ✅ | | | | | | | **3** |
| 安全认证全链路 | ✅ | | | ✅ | | | ✅ | | | | | **3** |
| 数据治理/血缘 | ✅ | ✅ | | ✅ | | | ✅ | | | | | **4** |
| Warehouse 熔断/QoS | ✅ | | | | | ✅ | ✅ | | | | | **3** |
| 扩缩容性能 | ✅ | | | | | | | | | | | **1 (关键)** |
| MV 数据质量 | | | | | | | | ✅ | | | | **1** |
| Lake Formation | ✅ | | | | | | | | | | | **1 (关键)** |
| Kafka Header 解析 | | | | | ✅ | | | | | | | **1** |
| 跨集群复制 | ✅ | | | | ✅ | | | | | | | **2** |

---

## 附录: 数据来源

| 来源 | 覆盖范围 | 备注 |
|------|---------|------|
| Feishu Feature Requirements | ~98 条消息 | 客户需求和功能讨论 |
| Feishu 产品 & 客户成功 | ~100 条消息 | 产品方向和客户成功案例 |
| Feishu Intuit 频道 | ~50 条消息 | Intuit 专项需求（最丰富） |
| Feishu Field Engineering | ~50 条消息 | 工程支持和事故复盘 |
| JIRA SR 项目 (Improvement/Feature) | 100 个 issues | 功能改进和新特性 |
| JIRA SR 项目 (Critical/High Bugs) | 100 个 issues | 高优先级缺陷 |
| JIRA SSU 项目 (Improvement/Feature/Task) | 100 个 issues | 客户实施和需求 |
| Slack 搜索 | 多次搜索 | 客户共享频道访问受限 |
| Google Doc (2024) | 完整文档 | 前期客户分析基础 |
| 行业公开研究 | 2025-2026 | 市场趋势和竞品分析 |

---

| StarRocks GitHub 代码库分析 | 代码结构+Issues+Roadmap | 竞品 feature gap 分析 |

*本文档基于 Feishu、Slack、JIRA (300+ issues, 含 Improvement/Feature/Bug 类型)、Google Docs、StarRocks GitHub 代码库及行业公开信息综合整理。Slack 客户专属频道因权限限制访问有限，建议后续补充直接客户访谈数据。*
