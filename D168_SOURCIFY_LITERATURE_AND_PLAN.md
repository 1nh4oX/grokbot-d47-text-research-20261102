# D168：Sourcify合约源码可行性文献与来源审查

日期：2026-11-02。类型：资料先行、尚未采集的阶段计划；本轮只读项目记录与公开原文/官方文档，未查询Sourcify地址、未运行模型、未下载数据。D47-v1标签、时间起点与基线未改。

## 当前瓶颈与判断

当前瓶颈是**历史数据来源的截止前覆盖与可追溯性**，不是模型容量。D118/D119在冻结48币上只得到5个合格首周网页；训练/开发各仅1个持续恢复正例，开发正例没有合格网页，故该网页来源不能支持公平文本模型比较。结果只说明“源码入口→准确首页→首周档案”路线覆盖低，不说明所有历史文本都不存在。见[D118/D119记录](D118_D119_CROSSPERIOD_RESULTS_20261025.md)及[D109/D113记录](D109_D113_HISTORY_RESULTS_20261025.md)。

Grokbot报告Sourcify和Internet Archive探针较可靠，但尚未提供8个探针的端点、参数、原始响应字段、覆盖分母与样本地址；目前只能称接口探测线索，不能称来源门通过。Internet Archive的[官方CDX接口说明](https://github.com/internetarchive/wayback/blob/master/wayback-cdx-server/README.md?plain=1)列出捕获时间、原始URL、状态与digest；捕获时间不是首次发布时刻。项目既有档案也只证明少数精确页面快照。

## 文献依据与迁移边界

### 已有论文#042：RPHunter

Wu等，arXiv:2506.18398 v3（2025-07-08）。本轮阅读原文HTML摘要、§III、§IV-B、§V-A、§V-E、§V-F；未复现代码或逐节通读全文。[原文v3](https://arxiv.org/abs/2506.18398v3)

论文从安全平台收集1048个rug事件，排除131个无源码和272个无交易史后，手工分析645个rug案例；良性候选1806个中保留1675个有交易记录样本。方法通过`getCode`读取部署字节码、Gigahorse反编译为IR、做控制/数据流分析，以识别售出限制、可操纵税率/地址/外部调用、隐藏铸造/销毁、余额篡改；再把代码风险图与交易行为图用GNN及注意力融合。方法段报告60/20/20训练/验证/测试比例，没有说明时间前向切分。作者报告实验集precision 95.3%、recall 93.8%、F1 94.5%。

**文献能支持**：代码机制和交易行为可作为互补的rug风险信息；代码风险应从可执行逻辑/数据流理解，而不只是关键词袋。**不能支持**：源码文本能提升D47，或论文指标可迁移到D47。Rug pull标签与D47持续交易恢复不同；RPHunter又以有码、且有交易记录的事件筛样，可能有明显可得性选择。更新了知识库#042卡片；论文总数不变。

### Sourcify官方来源文档

- [API v2文档](https://docs.sourcify.dev/docs/api/)说明可按chain/address查询已验证合约，选择`verifiedAt`、匹配状态、源码等字段。官方fair-use说明API面向单合约，不应循环抓取大量地址；批量研究应使用其数据集或BigQuery。
- [监控服务说明](https://github.com/sourcifyeth/docs/blob/main/docs/6.%20Sourcify%20in%20depth/7.%20Monitoring%20service.md)称监控器对受监控链的新合约检查元数据哈希，从Swarm/IPFS取metadata和source；若部署附近已发布且可取到，默认约30分钟后尝试自动验证。这仅是部分链/部分部署路径，不意味着数据库覆盖完整。
- [公开BigQuery说明](https://docs.sourcify.dev/docs/bigquery/)称数据集随新验证持续更新；[数据库文档](https://docs.sourcify.dev/docs/contract-dataset/sourcify-database/)列出部署、验证、编译及source表关系。BigQuery小查询的计费和字段schema本轮未核验。

**时间解释**：`verifiedAt`表示Sourcify记录的验证时间，可作严格的截止前可用性代理；不能当合约的首次公开时间。仅在`verifiedAt <= D47 prediction cutoff`且合约精确匹配时，源码才进入D47候选特征；晚验证代码不计入D47文本，未命中Sourcify仅记“该库未找到”，不能记作无源码或M=0。

## 下一步预注册草案（等待用户说“开始”后再执行）

**可证伪问题**：在既有冻结跨期48币面板中，至少一半能否取得非空、精确匹配且在D47预测时点前验证的token合约源码，并且覆盖不集中于单一月份？这是来源可行性问题，不是模型效果假设。

**样本**：重用[既有D117冻结48名单](../outputs/D117_crossperiod_text_preparation/plan.json)，不增删币、不按结局替换：2025-09/10/11训练期各8币、2026-02/03/04开发期各8币；分层维持原`month × first-week transaction bin`。名单不含标签，发送给Grokbot时不提供标签。每币查询Ethereum chain ID 1的token合约；冻结pool只用于资产身份核对，Uniswap pool通用代码不作token文本。D47预测截止严格为候选UTC上线日期+7天00:00（观察日0–6），不以“第7天结束”替代。

**字段与判定**：记录响应URL/时间/哈希、HTTP状态、chain/address、`match`/`creationMatch`/`runtimeMatch`、`verifiedAt`、source文件数/字节数/digest、部署transaction/block。可用源码须address/chain一致、`runtimeMatch=exact_match`、非空source，且`verifiedAt <= UTC launch date + 7 days 00:00 UTC`；其他命中另存但不计合格。未验证/缺失/查询失败分别记为未知或采集状态，不改写成零文本。

**来源路线**：不循环调用逐合约API。先核验Grokbot的8个探针回执；如果确需继续，以官方BigQuery按48地址过滤并先做dry-run/扫描量评估为候选路径；本轮不执行查询、不下载整个Sourcify数据集，也不使用Dune。BigQuery字段和费用需在任何正式运行前核验。

**覆盖门槛（项目自定，不是论文结论）**：至少24/48币合格，且六个月份各至少4/8合格；任一门未过则停止该来源扩展。通过也只说明小面板来源可行；48币中正例过少，不能训练或评价D47文本模型。不得据此选择标签、阈值或模型。

**后续若来源门通过**：再冻结提取实现与特征字典，先检查来源覆盖、源码模板/重复代码率、训练/开发月和结局组覆盖；代码风险规则候选仅按RPHunter已有taxonomy审读。届时预先定义对照：D47冻结RF、源码risk features单独、现有交易特征+源码risk features；同样本/切分/预算，开发结果仅探索，不称贡献；保留独立时间测试前冻结方案。可能失败原因是大多数token未及时验证、模板源码无法区分、选择性验证偏差，或rug风险结构与持续交易恢复无关。

## 交给Grokbot的下一条要求

先回交8个探针的逐项审计表：精确URL/endpoint、参数、执行时间、HTTP状态、响应字段示例、覆盖分母、错误与重试、官方文档链接、是否读取源码正文。说明其Sourcify请求是否符合“不逐地址批量爬取”规则，并给出48地址过滤的BigQuery schema与dry-run方案；不要自行批量跑查询或请求完整数据集。若此前探针只测了连通性，不要写成覆盖率结论。

## 阶段结果

本轮无新的来源命中数、覆盖率或预测指标。文献#042的原文阅读范围已补充到知识库；不新增论文编号。下一步仅待Grokbot探针回执与用户启动来源可行性查询。
