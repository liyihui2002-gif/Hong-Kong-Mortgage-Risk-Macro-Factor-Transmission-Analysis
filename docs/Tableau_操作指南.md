# 在 Tableau 中复刻项目看板 · 操作指南

数据文件：`HK_Mortgage_Tableau_Data.xlsx`（3 个工作表）
- `monthly_data`：96 个月 × 50 列（含真实 `Date` 日期字段）
- `negative_equity_quarterly`：32 个季末负资产观测
- `data_dictionary`：26 个关键字段的中英文对照 + 单位 + 来源

## 第 1 步：连接数据

Tableau Desktop 打开后 → 左侧「连接」→ **Microsoft Excel** → 选择 `HK_Mortgage_Tableau_Data.xlsx`。
默认会读取第一个表 `monthly_data`。字段里 `Date` 会被自动识别为日期维度（连续型）。

## 第 2 步：把 Date 设为连续日期

左侧「数据」窗格找到 `Date`，右键 → **转换为连续**（绿色图标）。这一步让折线图按时间轴平滑展开。

## 第 3 步：复刻四张核心图

### 图 1：利率 vs 申请/成交（双轴折线）

1. 把 `Date`（连续）拖到「列」。
2. 把 `hibor_3m_pct`（度量，默认 SUM，改成 **AVG**）拖到「行」。
3. 再把 `new_applications_received` 拖到**同一行的右侧轴**（出现双轴）。
4. 右键右侧轴 → **双轴**（Dual Axis），再右键 x 轴 → **同步轴**（Synchronize Axis）。
5. 颜色/标签在「标记」里调：HIBOR 用红/实线，申请用蓝/虚线。
6. 可选：把 `cycle_period` 拖到「颜色」，区分三个政策阶段。

### 图 2：三套压力指数对比（多折线）

1. 「列」放 `Date`（连续）。
2. 「行」放 `stress_index_equal`（AVG）。
3. 把 `stress_index_rate_dominant` 和 `stress_index_borrower_dominant` 分别拖到「行」的同一轴，或用「度量值」字段：把三个 stress_index 度量都拖到「行」，Tableau 自动合并成「度量值」，用「度量名称」分颜色。
4. 这样三条线同图对比，能直接看到三个峰值月份不同（等权 2024-07、利率主导 2024-07、借款人主导 2024-09）。

### 图 3：季度负资产 vs HIBOR

用第二个工作表 `negative_equity_quarterly`：
1. 新建数据源连接同一文件，选 `negative_equity_quarterly` 表。
2. 「列」放 `Date`（连续），「行」放 `negative_equity_count`（SUM）。
3. 图表类型选**条形**（bar），就是按季度末的负资产宗数柱状图。
4. 想叠加利率：可再连接 `monthly_data` 的 `hibor_3m_pct` 做双轴（需要建立日期关系），或直接在负资产图旁另放一张 HIBOR 折线。

### 图 4：滞后相关性（可简化）

Tableau 的「滞后」需要表计算。简化做法：
1. 用 `monthly_data`，把 `hibor_3m_pct`（AVG）和 `negative_equity_count`（SUM）分别拖到「行」和「列」。
2. 右键 → **趋势线** → 显示 Pearson R²（Tableau 会给出拟合度），即可作为"两变量线性相关"的可视化替代。
3. 要严格复刻 ±12 月滞后分析，建议回到 Python（`scripts/build_dataset.py` 里的 lag 循环），Tableau 不擅长做滑动滞后相关矩阵。

## 附：关键数值备忘（做图注时用）

| 指标 | 数值 |
|---|---|
| 加息期申请较加息前 | −25.3% |
| 加息期批核较加息前 | −17.8% |
| 加息期成交较加息前 | −24.3% |
| HIBOR 与申请同期相关 | −0.70 |
| 申请与成交零滞后相关 | +0.67 |
| 等权压力指数峰值 | 58.7（2024-07） |
| 利率主导峰值 | 60.7（2024-07） |
| 借款人主导峰值 | 59.9（2024-09） |
| 3M HIBOR 样本峰值 | 5.438%（2023-11） |

> 注意：`negative_equity_count` 在月度表里是「向前填充」的季度值，只有季末月份是真实观测；严格分析请用 `negative_equity_quarterly` 表。
