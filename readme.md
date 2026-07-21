# data.json 說明

## 全域資料

### `meta`

整個 demo 的共用資訊。

- `product`：產品名稱
- `page`：頁面識別
- `pageUpdatedAt`：頁面更新時間
- `defaultPeriod`：預設統計／評測期間

### `rules`

前端顯示狀態與比例時使用的規則說明。

- `healthStatus`：Forecast / Anomaly / Recommender 的 Health Status 判定規則
- `drift`：PSI、KS p-value 與漂移狀態門檻
- `formulas`：各比例欄位的計算公式
- `guardrailBlockedRuns`：Guardrail Blocked Runs 的計數定義

## 模型總覽頁

### `sites`

對應總覽頁「依 Site 分組的模型清單」每個 Site header。

- `id`：Site ID
- `name`：Site 名稱
- `healthStatus`：Site 整體狀態
- `stats.total`：模型總數
- `stats.normal`：Normal 模型數
- `stats.warning`：Warning 模型數
- `stats.critical`：Critical 模型數

### `overview.tabs`

對應總覽頁上方模型種類篩選。

### `overview.sortOptions`

對應總覽頁右上排序選單。

### `overview.columnsByKind`

對應 Site 內不同 model kind 的表格欄位。

- `forecast`
- `anomaly`
- `recommender`

### `overview.criticalModels`

對應總覽頁「嚴重模型」區塊。

## 單一模型詳細頁

所有模型詳細資料都放在 `models[]`。

### 共用欄位

- `id`：Model ID
- `kind`：`forecast` / `anomaly` / `recommender`
- `modelKind`：模型種類與演算法
- `siteId`：Site ID
- `siteName`：Site 名稱
- `modelVersion`：模型版本
- `healthStatus`：詳細頁 Health Status
- `lastEvaluated`：總覽頁顯示的最後時間
- `serviceInfo`：服務資訊區塊
- `metricCards`：核心指標卡片
- `charts`：圖表資料
- `featureDrift`：輸入特徵漂移監測表

## Forecast 詳細頁

Forecast model 使用：

- `targetPoint`
- `evaluatedAt`
- `evaluationPeriod`
- `overview.mape`
- `overview.skillScore`
- `overview.drift`
- `metricCards`
  - MAE
  - MAPE
  - Baseline MAPE
  - Skill Score
  - 評測覆蓋率
- `charts`
  - `forecast_metrics_trend`
  - `actual_prediction_baseline`
- `evaluationDataQuality`
  - `predictionCount`
  - `actualCount`
  - `joinedRows`
  - `excludedActualRows`

## Anomaly 詳細頁

Anomaly model 使用：

- `targetPoint`
- `evaluatedAt`
- `evaluationPeriod`
- `overview.events`
- `overview.anomalyRate`
- `overview.scoreP99`
- `metricCards`
  - 異常事件數
  - 異常率
  - Score P99
  - Threshold
  - 評測覆蓋率
- `secondaryMetrics`
  - 有 `labeledRows > 0` 才顯示
  - Precision
  - Recall
  - F1-score
- `charts`
  - `anomaly_metrics_trend`
  - `anomaly_score_trend`
- `scoringStats`
  - `scoredRows`
  - `validRows`
  - `excludedRows`
  - `labeledRows`

## Recommender 詳細頁

Recommender model 使用：

- `targetScope`
- `targetScopePoints`
- `statisticsUpdatedAt`
- `statisticsPeriod`
- `overview.successRate`
- `overview.p95Latency`
- `overview.guardrailRate`
- `metricCards`
  - 執行成功率
  - 採用率
  - Guardrail 攔截率
  - P95 Latency
- `charts`
  - `recommender_metrics_trend`
  - `recommender_latency_trend`
- `executionStats`
  - `totalRuns`
  - `successfulRuns`
  - `failedRuns`
  - `guardrailBlockedRuns`
  - `adoptedRuns`

## 圖表資料格式

每個 `charts[]` 物件都包含：

- `id`：圖表識別
- `title`：圖表標題
- `subtitle`：副標題
- `type`：`bar` 或 `line`
- `defaultMetric`：左側可切換圖的預設指標
- `xAxis`：時間軸
- `series`：圖表資料序列

每個 `series[]` 包含：

- `name`：序列名稱
- `unit`：可選，數值單位
- `values`：數值陣列

`xAxis.length` 必須等於每個 `series.values.length`。
