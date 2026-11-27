# PACE 首版设计

2026 年 11 月 27 日。以下是拟实施和验证的设计；文中的两个项目、时限及日期都是模拟数据。

## 谁在什么情况下需要它

内容运营同时维护商家工作台和创作者工作台两个项目。遇到规则变更时，她要在很短的时间内决定对业务方承诺什么。共享手册解释术语，项目规范才给出本项目的时限。讨论消息出现得最快，但可能只是提议；通知可以今天批准、明天才生效；另一个项目的经验再相似，也不能拿来代替本项目依据。

从运营视角，答案需要交代**适用项目、截至日期、当前口径和可打开的证据**。当资料不足时，系统指出缺的是哪份确认，允许成员提交问题并看到处理结果。规则维护者需要知道具体哪句话被质疑、应该改哪版、改动会影响哪些问题。评估者可以从回答追到证据和运行记录，复查系统更新前后发生了什么。

## 重点借鉴的四个开源项目

| 参考 | 从实现中借什么 | 如何落到 Ragent |
| --- | --- | --- |
| **XWiki Change Request** | 变更请求的内容修改后，会使已有的有效评审失效并重新判断是否可合并。确认针对具体修订。[固定版本监听器](https://github.com/xwiki-contrib/application-changerequest/blob/5f6fe9adaef99b9d2a5fedae4b614ecc02013ea7/application-changerequest-default/src/main/java/org/xwiki/contrib/changerequest/internal/listeners/FileChangeUpdatedListener.java#L115-L138) | 规则有固定标识和独立版本。维护者确认具体版本及来源；正文或关键生效条件变化后形成待确认的新修订。Ragent 文档处理成功只表示入库进度，业务确认单独记录。 |
| **OpenFGA** | 文件可以继承所在文件夹的读取关系，阅读和修改权限可分别声明，样例同时给出允许和拒绝的测试。[关系模型](https://github.com/openfga/sample-stores/blob/6802b4d859d2773eea72c2315b3a9f027a2893a0/stores/gdrive/model.fga#L15) · [授权用例](https://github.com/openfga/sample-stores/blob/6802b4d859d2773eea72c2315b3a9f027a2893a0/stores/gdrive/store.fga.yaml#L59) | 首版复用登录身份，在 Java 服务按项目成员关系计算可读库和可维护规则。把允许范围传入主检索、补充检索及引用原文访问。低置信度可以扩大检索策略，仍限定在同一授权范围内。 |
| **腾讯 WeKnora** | 把事实冲突、知识过期等问题挂到具体页面和可疑来源，问题有待处理、已处理等状态。[登记](https://github.com/Tencent/WeKnora/blob/v0.9.1/internal/agent/tools/wiki_flag_issue.go#L37) · [处理](https://github.com/Tencent/WeKnora/blob/v0.9.1/internal/agent/tools/wiki_update_issue.go#L18) | 接到 Ragent 已有的消息反馈上，保留提问、回答、引用版本、异议和负责人。允许普通读者报告错误，由有权限的维护者核实来源并处理。处理结果要关联到实际更新和重问记录。 |
| **Langfuse** | 把一次有问题的真实调用转成数据集条目，由熟悉业务的人填写预期结果；用同一批题比较不同版本。[官方数据集文档](https://langfuse.com/docs/evaluation/experiments/datasets) · [Ragent 自带的追踪配置](../resources/docker/langfuse/README.md) | 每个案例保存调用身份、问句、查询日期、资料快照、预期答案或澄清动作、证据文档与版本。关联已有追踪 ID；追踪未配置时先用本地 JSONL 记录。人工核实后再把线上错例收入回归集。 |

时间语义可直接借鉴 Graphiti 对记录时间、事实生效时间与失效时间的区分。首版可用关系表表达规则版本、适用项目、生效区间、确认人及证据。审批动作与未来生效时点各自记录。Plone 的内容目录把生效和到期时间用于实际查询，提示有效日期应落实到检索条件，而不是只显示在文档简介里。[Plone 检索实现](https://github.com/plone/Products.CMFPlone/blob/50e28766bf8e709d576a26d61e9a739655802737/src/Products/CMFPlone/CatalogTool.py#L413-L424) 实际业务中，GOV.UK 的 Whitehall Publisher 将“编辑审核稿件”和“现在发布或按计划发布”分开，也是可以参照的运营流程。[Graphiti 事实字段](https://github.com/getzep/graphiti/blob/v0.32.1/graphiti_core/edges.py#L232) · [Whitehall 实际发布指引](https://guidance.publishing.service.gov.uk/publish-update-retire-content/standard-content-types/publish-standard/)

## 一条问题从提问到修正怎样流转

1. **确定范围**。从登录身份得到允许访问的项目和共享资料。用户没说项目、多个可见项目的口径不同，就先追问；只在允许范围内给出澄清选项。Ragent 当前在部分低置信场景回到活跃库集合，要把授权范围传进这一步及后续补查。[检索范围入口](../rag/src/main/java/com/nageoffer/ai/ragent/rag/core/retrieval/channel/RetrievalScopeResolver.java)
2. **挑选依据**。按所属项目、规则标识、询问日期和版本状态选择证据。讨论稿可用于解释提议进度，回答当前承诺时引用已经确认且在有效期内的正式规则。缺正式依据时，用规则编号或来源链接在项目内补查。两份同级正式材料仍冲突时，展示分歧和各自出处，送维护者确认。[现有知识工具](../rag/src/main/java/com/nageoffer/ai/ragent/rag/service/KnowledgeSearchFacade.java)
3. **核实和更新**。成员在出错的回答上发起反馈，维护者看到被引用的版本、异议和来源。确认新版本后通过现有入库能力建立索引。若新口径已生效但尚未进入检索，回答需标明正在更新或转交确认；旧版不能继续被表述为现行。记录本次修订替代的版本，保留历史查询能力。[消息反馈](../rag/src/main/java/com/nageoffer/ai/ragent/rag/service/impl/MessageFeedbackServiceImpl.java) · [文档处理](../rag/src/main/java/com/nageoffer/ai/ragent/knowledge/service/impl/KnowledgeDocumentServiceImpl.java)
4. **从用户入口复测**。重问原题、抽查受影响的同规则问题，再跑另一项目和历史日期对照。单独保存最终答案、证据和实际检索范围。已有评测接口提供召回诊断，聊天入口运行给出最终用户所见结果。逐条证据来自该次对话自身的引用和运行记录；事后独立重跑的检索仅用于诊断。[检索评测接口](../rag/src/main/java/com/nageoffer/ai/ragent/rag/eval/EvalController.java)

## 决定首版是否有用的一组演示

商家工作台项目当前承诺 48 小时，创作者工作台项目正式规则为 12 小时。商家工作台运营组群聊提出 24 小时，维护者 11 月 29 日确认，正式通知指定 11 月 30 日零时生效。知识库资料均标记为模拟，权限按两类账号配置。

| 操作 | 预期可看到的行为 |
| --- | --- |
| 两项目成员问“响应时限多久”，没有给项目 | 如果当前对话也不能确定项目，先追问；用户选商家工作台后回答 48 小时并给商家工作台项目来源。 |
| 仅有商家工作台权限的成员问“群里说改成 24 小时了？” | 批准前能区分提议与当前规定；创作者工作台私有资料不进入给模型的证据、答案或引用。 |
| 11 月 29 日批准后当天重问，11 月 30 日再次重问 | 29 日仍答 48 小时，可以说明次日变更；30 日答 24 小时并引用对应版本。 |
| 12 月 1 日问“11 月 29 日当时怎么执行”，再由创作者工作台成员问本项目 | 历史仍为 48 小时并能查到旧依据；创作者工作台仍为 12 小时。 |
| 修改已确认稿的关键条款，或者模拟新版本索引尚未就绪 | 新修订需重新确认；系统能说明待确认或更新中的内容。反馈单关联处理记录，只有实际重问后才记录复测结果。 |

**实测要回答两个不同的问题。** 改知识后，同一问句是否引用新依据，历史和其他项目是否保持正确；改系统后，之前实际暴露的错误是否在固定知识快照和相同问题下得到修正。第一轮记录模型、配置、资料、身份与日期。通用大模型直接问答得到相同权限可见的原始材料，明确记录所给上下文；另跑原版 Ragent 和 PACE。逐题列出规则与版本是否正确、引用是否支持关键结论、该追问时是否追问，以及无权证据进入模型上下文的次数和用例数。尚未生成实测分数。

## 首次验证

用独立的模拟数据集核对镜像标识、聊天模式和实际连接的服务，记录一轮原版问答。首轮开发顺序是项目范围与规则版本、反馈关联与发布、端到端复测。先用少量虚构资料跑通整条链，稳定后再考虑更多文档格式、消息通知或飞书接入。
