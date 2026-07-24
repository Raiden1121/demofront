# data.json 說明

本 demo 的模型、站點、總覽、指標、圖表、漂移與資料品質內容都集中在 `data.json`。

`index.html` 啟動時會執行 `fetch("./data.json")` 讀取此檔，再把 JSON 正規化成畫面需要的欄位。前端只負責顯示、排序、篩選、切換圖表指標與繪製 SVG；不再在 `index.html` 內寫死模型資料或圖表數列。

> 注意：因為瀏覽器會用 `fetch` 載入 `data.json`，建議用本機 HTTP server 開啟頁面，例如在專案目錄執行 `python3 -m http.server 8000` 後瀏覽 `http://localhost:8000/`。直接用 `file://` 開啟可能會被瀏覽器阻擋讀取 JSON。

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

對應總覽頁上方模型種類篩選。前端會直接依照這個陣列產生 tab。

- `key`：篩選值，`all` 表示全部，其餘需對應 `models[].kind`
- `label`：畫面顯示文字

### `overview.sortOptions`

對應總覽頁右上排序選單。前端會直接依照這個陣列產生 select option。

- `key`：排序欄位識別，目前支援 `modelId`、`kind`、`status`、`metric`、`skill`、`drift`、`lastEval`
- `label`：畫面顯示文字

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

### 前端圖表對應規則

詳細頁有兩張 SVG 圖：

- 第一張趨勢圖：讀取 `charts[]` 中 `type: "bar"` 的圖表。
- 第二張比較／趨勢圖：讀取 `charts[]` 中 `type: "line"` 的圖表。

`type: "bar"` 的 `series[].name` 會成為圖表右上 segmented control 的切換按鈕。初始選中的指標使用該 chart 的 `defaultMetric`；如果沒有設定，前端會使用第一個 `series`。

`type: "line"` 的 `series[]` 會依序對應折線：

- 第 1 條：藍色實線
- 第 2 條：紅色實線
- 第 3 條：灰色虛線

折線圖 legend 直接使用 `series[].name`。如果只有一條線，例如 Recommender 的 `P95 Latency`，只會畫出第一條線。

兩張圖底部日期標籤都來自各自 chart 的 `xAxis`。前端只抽取起點、中間點與終點顯示，不會自行產生日期。

### 更新圖表資料的方式

要改圖表，只需要修改對應 model 的 `charts[]`：

1. 找到 `models[].id`。
2. 修改 `type: "bar"` 或 `type: "line"` 的 `xAxis`。
3. 修改該 chart 的 `series[].values`。
4. 確認每個 `series.values.length` 都等於 `xAxis.length`。

不需要修改 `index.html`，除非要改畫法、互動或版面。
