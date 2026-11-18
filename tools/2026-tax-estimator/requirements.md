# 2026 加州居民税务快速估算（2026-tax-estimator）

单页、无外部依赖的 HTML 工具：`index.html`，浏览器直接打开即可使用。

## 目标

- 大致估算 2026 年的联邦税和加州税。不追求覆盖每个细节，重点是帮助做年末决策：捐多少、卖出多少股票，才能在收入和税之间取得平衡。
- 输入任意一项，结果都实时更新。

## 需求（来自用户）

1. 只做大致估算，能指导年末捐赠和卖股计划即可。
2. 按 2026 年的加州和联邦税法计算。报税身份可选：Married Filing Jointly（默认）、Single、Married Filing Separately、Head of Household。要充分考虑高收入带来的附加税（NIIT、Additional Medicare、AMT、加州 1% Mental Health Services Tax 等），以及捐赠抵扣的相关条款。
3. 必填输入：ordinary income、long-term capital gain（可正可负）、donation。
4. 实时显示：
   - 联邦税：所得税，加上全国统一的工资税（Social Security、Medicare、Additional Medicare）、NIIT、AMT；
   - 加州税：所得税、MHST、加州 AMT、SDI；
   - 实际税率；
   - 各项的边际税率（marginal rate）。

## 在需求基础上的补充（建议，已实现）

- **捐赠拆成两类：现金捐赠和增值股票捐赠**。两者的 AGI 上限不同（联邦：现金 60%、股票 30%；加州：现金 50%、股票 30%）。另外，捐股票还能免掉这部分增值的资本利得税。工具会算出“把这些股票卖掉再捐现金”要多交多少税（需要填成本基础）。
- **捐赠输入框实时显示当年最多可抵扣金额**（联邦、加州分开显示，随 AGI 变化；现金的上限要先扣掉已抵扣的股票捐赠）。输入超过上限时，输入框和提示变成红色，并显示当年不能抵扣、需要结转的金额。
- **边际分析**：每多 $1,000 普通收入、LTCG、现金捐赠，税额变化多少。用数值差分计算，所以各种 phase-out 的影响都会自动算进去。
- **档位余量**：离下一个关键门槛还差多少，包括联邦普通税率档、LTCG 0%/15%/20% 档、NIIT 门槛、SALT cap phase-out、37% 档（2/37 规则）、加州税率档、加州 $1M MHST 门槛。据此可以决定年末还能卖多少股票而不跳档。
- **提示**：捐赠超出 AGI 上限的部分会结转（carryover）、0.5% AGI floor 损失的抵扣额、资本损失结转、SALT cap 是否触顶、是否触发 AMT。
- **可选的高级输入**：短期资本利得、合格股息、其他投资收入（用于算 NIIT）、夫妻各自的 W-2 工资（Social Security 按人封顶）、18 岁以下子女数、房产税、房贷利息、ISO 行权价差（AMT）、已预扣/预缴税额（显示还需补缴多少）。
- 输入只保存在浏览器 localStorage，不上传，也不写进 URL。

## 不做的事（简化假设）

- ordinary income 视为已扣除 401(k)/HSA 等税前项后的 W-2 Box 1 金额。如果填了“W-2 工资”，超出工资的部分视为非工资普通收入，不交 FICA。
- 不处理自雇税（SE tax）、QBI、外国税收抵免、教育/EV 等其他抵免、老年人加计扣除、各类收入的 phase-in。
- 州所得税的 SALT 抵扣近似取“当年加州所得税 + SDI”的估算值（实际应为当年实际缴纳金额，按现金收付制）。
- 捐赠只考虑给 public charity / DAF。非 itemize 时的 $1,000/$2,000 现金捐赠扣除，不区分 DAF 不适用的情况。
- 联邦 AMT 和加州 AMT 都只做粗略计算：只把 SALT/房产税、标准扣除和 ISO 加回。
- 子女税收抵免按不可退还处理，并做 phase-out。

## 2026 税法参数与来源

### 联邦（IRS Rev. Proc. 2025-32 / OBBBA）

| 项目 | MFJ | Single | MFS | HoH |
|---|---|---|---|---|
| 10% 档上限 | 24,800 | 12,400 | 12,400 | 17,700 |
| 12% | 100,800 | 50,400 | 50,400 | 67,450 |
| 22% | 211,400 | 105,700 | 105,700 | 105,700 |
| 24% | 403,550 | 201,775 | 201,775 | 201,775 |
| 32% | 512,450 | 256,225 | 256,225 | 256,225 |
| 35% | 768,700 | 640,600 | 384,350 | 640,600 |
| 标准扣除 | 32,200 | 16,100 | 16,100 | 24,150 |
| LTCG 0% 上限 | 98,900 | 49,450 | 49,450 | 66,200 |
| LTCG 15% 上限 | 613,700 | 545,500 | 306,850 | 579,600 |
| NIIT / Add'l Medicare 门槛 | 250,000 | 200,000 | 125,000 | 200,000 |
| AMT 豁免 | 140,200 | 90,100 | 70,100 | 90,100 |
| AMT 豁免 phase-out 起点（50%） | 1,000,000 | 500,000 | 500,000 | 500,000 |

- Social Security：6.2%，工资上限 $184,500（按人计）；Medicare 1.45%；Additional Medicare 0.9%。
- SALT cap：$40,400（MFS $20,200）。MAGI 超过 $505,000（MFS $252,500）后，按超出部分的 30% 递减，最低降到 $10,000（MFS $5,000）。
- 捐赠（OBBBA，自 2026 年起）：
  - itemize 时，捐赠抵扣额要先减去 0.5% AGI 的 floor；
  - 不 itemize 时，现金捐赠可扣除最多 $1,000（MFJ $2,000）；
  - 37% 档纳税人的 itemized deductions 要减少 2/37 × min(itemized, AGI − 37% 档起点)，相当于把每项抵扣的价值封顶在 35%；
  - AGI 上限：现金 60%，增值股票 30%，超出部分可结转 5 年。
- 子女税收抵免：每个子女 $2,200，AGI 超过 $400k（MFJ）或 $200k（其他身份）后，每超出 $1,000 减少 $50。
- AMT 税率 26%/28%，28% 档起点 $244,500（MFS $122,250）。

### 加州（FTB 2026 年指数化备忘录，2026-09-03）

- 税率表 X/Y/Z：见 `index.html` 中的 `CA.brackets`（1%–12.3%，共 9 档）。
- 标准扣除：$5,900（Single/MFS），$11,800（MFJ/HoH）。
- 个人豁免抵免：每人 $158（MFJ 合计 $316）；每个受抚养人 $491。AGI 超过门槛后，每超出 $2,500（MFS 为 $1,250）每项抵免减 $6。
- 豁免抵免 phase-out 门槛与 itemized deduction 递减门槛相同：$260,778（Single/MFS），$391,173（HoH），$521,561（MFJ）。itemized 减少额取 min(6% × 超出部分, 80% × itemized)。
- Mental Health Services Tax：应税收入超过 $1,000,000 的部分加征 1%，不论报税身份，门槛不做通胀调整。
- 加州 AMT：税率 7%；豁免额 $127,872（MFJ）、$95,902（Single/HoH）、$63,932（MFS）；豁免额在 $479,512、$359,633、$239,752 以上按 25% 递减。
- SDI：工资的 1.3%，无工资上限。
- 加州把资本利得按普通收入征税；不适用 0.5% floor，也不适用 SALT cap（房产税可以全额抵扣，州所得税不能抵扣）。

### 数据来源

- IRS Rev. Proc. 2025-32，经第三方汇总核对：https://q3adv.com/federal-income-tax-brackets-2026/
- SALT/AMT/捐赠/SS 工资上限：https://tscpatax.com/articles/salt-deduction-cap-2026 、https://taxdeductionatlas.com/amt-alternative-minimum-tax-2026-complete-guide/ 、https://www.irs.gov/pub/irs-pdf/p15.pdf
- FTB 指数化备忘录：https://www.caltax.org/regulatory-issues/ftb/2026-PIT-Indexing-Memo.pdf
- 加州 SDI 2026：https://vensure.com/employment-law-updates/california/reminder-for-california-employers-2026-state-disability-insurance-rate-di-pfl-benefit-changes/
