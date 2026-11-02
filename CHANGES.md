# CHANGES · 项目改动清单

> 本文件记录对 DSC5002 Group 31「香港住宅按揭风险监测」项目所做的全部修改。
> 对照《DSC5002 验收报告》中列出的问题逐项修正，并补齐交付物。

## 一、修正分析错误（对照验收报告的 5 个问题）

### 1. 滞后方向写反 ✅

- **问题**：`批核 → 成交` 的最佳滞后被写成"批核领先成交一个月"。
- **修正**：最佳滞后实为 **−1**（r≈0.60765），即**成交领先批核一个月**。
- **涉及文件**：`docs/report-en.md`、`docs/report-cn.md`、`dashboard/app.js`。
- **补充说明**：交换两个变量只是同一交叉相关的镜像，不是独立稳健性检验，已在报告中澄清。

### 2. 借款人主导指数峰值报告有误 ✅

- **问题**：脚本只找等权峰值所在行，再读该行的其他权重值，导致把"7 月的借款人主导值 59.7"当成了峰值。
- **修正**：对每套权重分别计算 `idxmax()`。
- **修正后峰值**：

| 方案 | 峰值月份 | 峰值 |
|---|---:|---:|
| 等权 | 2024-07 | 58.7260 |
| 利率主导 | 2024-07 | 60.6581 |
| 借款人主导 | 2024-09 | 59.9194 |

- **涉及文件**：`scripts/build_dataset.py`（新增 `stress_peaks`，按方案各自取峰）、`docs/report-en.md`、`docs/report-cn.md`、`docs/methodology.md`、`README.md`、`dashboard/app.js`（压力图下方动态显示各方案峰值）。

### 3. "12 个月"是搜索上限，不是已识别的传导周期 ✅

- **问题**：报告把"利率领先 12 个月相关最高"表述为确定的传导周期。
- **修正**：改为"预设 ±12 个月搜索窗口内的正向边界"，并补充扩展检验：
  - 扩展至 +15 个月：水平相关 r=0.8605（n=81），高于 +12 月的 0.8464；
  - 季度对齐（季末 HIBOR vs 季末负资产）：滞后 4 季水平相关 r=0.8572（n=28），一阶差分后仅 r=0.2657（n=27）。
- **涉及文件**：`docs/report-en.md`、`docs/report-cn.md`、`dashboard/app.js`。

### 4. 周期峰值标签与实际数据不一致 ✅

- **问题**：事件标签写成 `HIBOR peak / cycle transition`，但样本内 3M HIBOR 实际于 2023-11 见顶（5.438%），而非 2023-08。
- **修正**：
  - 事件标签改为 `Cycle transition (analyst-defined)`；
  - 明确 2023-08 是研究者设定的周期转换分界，不是样本 HIBOR 峰值；
  - 网页"历史峰值"措辞收窄为"本研究样本期峰值"。
- **涉及文件**：`scripts/build_dashboard_data.py`、`scripts/build_dataset.py`（新增 `hibor_3m_sample_peak`）、`dashboard/data.json`、`dashboard/data.js`、`dashboard/index.html`、`dashboard/app.js`、`docs/report-en.md`、`docs/report-cn.md`、`docs/methodology.md`。

### 5. 可复现"固定快照"保障不足 ✅

- **问题**：下载脚本请求持续更新的官方 URL，原始数据未纳入版本控制，无快照哈希或数据版本清单。
- **修正**：`scripts/download_data.py` 新增生成 `data/raw/source_manifest.json`，记录：
  - 下载时间（UTC）、Python 版本、平台；
  - 每个源文件的下载 URL、SHA-256 哈希、字节数。
- **涉及文件**：`scripts/download_data.py`。

## 二、新增内容

### 稳健性附录

- 中英文报告各新增「Robustness Appendix / 稳健性附录」，包含：
  - 领先滞后敏感性表（水平值、季度对齐、一阶差分、扩展搜索）；
  - 季节性与负资产变化说明（明确为相对提案的范围收缩）。

### 独立 Report 与 Poster

- `docs/DSC5002_Group31_Report.docx`：英文 Word 报告，嵌入 4 图 + 3 表 + 稳健性附录，独立可读。
- `docs/DSC5002_Group31_Poster.pptx`：学术海报（48×27 英寸，单页，嵌入 4 图）。

### 图表

- `artifacts/report_figures/` 下新增 4 张图（由仓库数据用 matplotlib 重新绘制）：
  - `fig1_hibor_activity.png` — 3M HIBOR vs 申请/成交
  - `fig2_stress_index.png` — 三方案压力指数
  - `fig3_negative_equity.png` — 季度负资产 vs HIBOR
  - `fig4_lag_profile.png` — HIBOR→负资产滞后相关

### 测试扩展（4 项 → 7 项）

- `tests/test_pipeline.py` 新增：
  - `test_stress_peak_reports_each_weights_own_maximum`（各权重各自峰值）
  - `test_approvals_transactions_lag_is_transaction_lead`（滞后方向）
  - `test_hibor_sample_peak_month`（HIBOR 样本峰值月）
  - `test_dashboard_payload_is_current` 增加事件标签校验（不含 "HIBOR peak"）

### Tableau 数据集与指南（后续追加）

- `docs/HK_Mortgage_Tableau_Data.xlsx`：Tableau 一键导入数据集（月度 96 行 + 季度负资产 32 行 + 双语数据字典）。
- `docs/Tableau_操作指南.md`：在 Tableau 中复刻看板的操作步骤。

### README 措辞修正

- 峰值数值更新（含三方案各自峰值）；
- 删除"验证了完整传导链路"的因果式表述，改为探索性相关描述；
- 展示链接改为当前账号 `liyihui2002-gif`。

## 三、改动文件总览

| 文件 | 改动 |
|---|---|
| `scripts/build_dataset.py` | 各权重各自 `idxmax()`；新增 `stress_peaks`、`hibor_3m_sample_peak` |
| `scripts/build_dashboard_data.py` | 事件标签改为 analyst-defined |
| `scripts/download_data.py` | 新增 `source_manifest.json` 快照 |
| `docs/report-en.md` | 方向/峰值/12月/标签修正 + 稳健性附录 |
| `docs/report-cn.md` | 同上（中文） |
| `docs/methodology.md` | 峰值 + 周期边界说明 |
| `dashboard/index.html` | 周期说明 + 峰值注记行 |
| `dashboard/app.js` | 结论文字 + 峰值动态显示 + 阶段卡片措辞 |
| `dashboard/data.json` / `data.js` | 重新生成（峰值/事件标签） |
| `README.md` | 峰值数值 + 因果措辞 + 链接改账号 |
| `tests/test_pipeline.py` | 4 → 7 项测试 |

## 四、验证

- 所有关键数值均用仓库数据独立复算，与《DSC5002 验收报告》核对一致；
- 7 项测试全部通过；
- 重新生成的 `data.json` 与提交版差异仅在 summary/events，96 个月度行逐字一致（流水线可复现）。
