# D168 Sourcify 源码来源可行性试点

更新：2026-11-02。此补充包给 Grokbot 使用。样本是公开链上 token/pool 地址与日期，**不含 D47 结局标签**。请继续来源可行性核查；不要把本轮写成预测实验。

## 任务边界

只评估 Ethereum token 合约源码能否在 D47 预测截止前被 Sourcify 精确验证、稳定取得。源码是合约代码数据，不是 Telegram/X 等项目叙事文本，也不会增加 token 样本数。保持 D47-v1 原标签：第7天早期退出后，未来60天内至少出现一组连续两天、每天各至少3笔已索引交易；不涉及价格上涨、价值恢复、rug pull 或永久死亡。D36不得混入。

使用同包 `D168_SOURCIFY_BLIND_SAMPLE.csv`：固定48币，训练/开发各24，六个月份各8，原`月份 × 首周交易笔数分层`保留。不换样本、不读/推断标签。每行 token 是 Sourcify 查询目标；冻结 pool 仅作身份交叉核对，不查询通用Uniswap池源码作token特征。预测截止为 `launch_date_utc + 7 days 00:00 UTC`，与项目第7天UTC 00:00协议一致。

## 获取方式与请求边界

先给出先前8个探针的逐项日志：endpoint/参数、时间、HTTP状态、示例字段、失败、覆盖分母、官方文档和“是否读取正文”。Sourcify当前官方API文档明确其查找API以单合约为单位，禁止循环逐地址批量抓取；**不要对CSV里48个API URL做循环请求**。先核对官方BigQuery公开数据的最新表结构与必要字段，写一个只过滤本清单地址的SQL并执行dry-run，报告预计扫描量及任何费用。仅在明确零费用/免费额度内才运行该小查询；否则停止并报告估算，不能下载全库、提交合约验证或使用Dune。

官方入口：

- [Sourcify API v2](https://docs.sourcify.dev/docs/api/)
- [Sourcify BigQuery](https://docs.sourcify.dev/docs/bigquery/)
- [Sourcify数据库schema](https://docs.sourcify.dev/docs/contract-dataset/sourcify-database/)
- [Monitoring service](https://github.com/sourcifyeth/docs/blob/main/docs/6.%20Sourcify%20in%20depth/7.%20Monitoring%20service.md)
- [RPHunter原文v3](https://arxiv.org/abs/2506.18398v3)：只支持对rug风险中代码与交易信息互补的文献启发；不支持D47收益，也不是文本分类模型。

若返回的`verifiedAt`为空或其含义与官方说明不符，先记录未核验，不自行解释或改用其他时间戳。`verifiedAt`不是页面首次发布时间；只有验证发生在D47预测截止前，才可进入“截止前源码”可用性计数。

## 合格判定与来源门

对每币留存查询/数据快照证据、响应或查询时间、chain/address、匹配级别、`verifiedAt`、source文件数/字节数和每文件SHA-256、部署交易/区块（若可用）、失败/未知原因。合格源码需同时满足：chain/address一致、`runtimeMatch=exact_match`、源码非空、`verifiedAt <= D47 cutoff`。身份不匹配、晚验证、无结果、schema缺失、查询失败分开编码；未命中只代表Sourcify库未找到，**不能标为“项目无源码”或M=0**。

来源门为项目预设决策阈值（非论文标准）：至少24/48币合格，且六个月份每月不少于4/8。未过任一项即停止Sourcify扩展；通过也仅表示小样本来源可行。此48币只有极少正例，禁止在本试点拟合、选词、比较AP或宣称预测增益。

## 负对照与报告

先用一个官方文档示例合约确认schema/字段解析，再用一个预先指定的非合约地址确认“未找到”解析；二者只校验程序响应，不计入48分母。对冻结样本所有行保留结果，不以命中替换失败币。另报告源码模板/重复hash比例，及时间截止合格率按月份、分层、训练/开发角色的覆盖表。

输出：原8探针审计表、SQL及dry-run成本证据（若无法零成本执行，停在dry-run）、48行来源状态CSV、所有原文/响应的哈希与查询时间、分层覆盖摘要、是否过门、失败与局限。不得访问学校服务器、抓取社媒、绕过访问限制、批量调用单地址API或下载整个Sourcify数据集。

若来源门通过，下一阶段另行预注册代码特征，不在本试点建模：文献启发的固定代码风险特征要与D47冻结强基线在同样本、同切分、同预算比较；源码可用状态M应先于源码内容S，且只有S优于M才说明内容增量。未冻结方案并完成独立时间测试前，开发表现只能算探索。
