# 前端儀表板資料契約

這份文件說明要讓目前 `index.html` 顯示完整儀表板時，`data.json` 需要提供哪些資料。

前端載入方式：

```js
fetch("./data.json")
```

前端只負責顯示、排序、篩選、切換圖表與繪製 SVG。所有模型資料、圖表數列、統計數字、狀態與時間都應由 `data.json` 提供。

## 1. 最外層結構

`data.json` 最外層需要有：

```json
{
  "meta": {},
  "rules": {},
  "sites": [],
  "overview": {},
  "models": []
}
```

用途：

- `meta`：頁面共用資訊與預設期間。
- `rules`：狀態判定與公式說明，主要用於 tooltip 或文件說明。
- `sites`：總覽頁依 Site 分組用。
- `overview`：總覽頁 tabs、排序選項、嚴重模型清單。
- `models`：每一個模型的完整詳細資料，也是圖表資料來源。

## 2. meta

提供整個儀表板共用資訊。

```json
{
  "product": "AIoTers DevOps",
  "page": "model-management-demo",
  "pageUpdatedAt": "2024-07-15 14:45:00",
  "defaultPeriod": {
    "label": "2024-07-01 ~ 2024-07-15",
    "start": "2024-07-01",
    "end": "2024-07-15"
  }
}
```

前端使用：

- `pageUpdatedAt`：總覽 Critical 區塊的 Last updated。
- `defaultPeriod.label`：模型沒有自己的期間時可當 fallback。

## 3. sites

提供總覽頁「依 Site 分組的模型清單」。

```json
{
  "id": "site_tpe_chw_01",
  "name": "台北冰水主機房",
  "healthStatus": "Warning",
  "stats": {
    "total": 3,
    "normal": 2,
    "warning": 1,
    "critical": 0
  }
}
```

必要欄位：

- `id`：Site ID，必須能對應到 `models[].siteId`。
- `name`：Site 顯示名稱。
- `healthStatus`：Site 整體狀態，建議使用 `Normal`、`Warning`、`Critical`。
- `stats.total`：該 Site 模型總數。
- `stats.normal`：Normal 模型數。
- `stats.warning`：Warning 模型數。
- `stats.critical`：Critical 模型數。

## 4. overview

提供總覽頁控制項與 Critical 模型表格。

```json
{
  "tabs": [
    { "key": "all", "label": "全部" },
    { "key": "forecast", "label": "forecast" },
    { "key": "anomaly", "label": "anomaly" },
    { "key": "recommender", "label": "recommender" }
  ],
  "sortOptions": [
    { "key": "modelId", "label": "Model ID" },
    { "key": "kind", "label": "Kind" },
    { "key": "status", "label": "Status" },
    { "key": "metric", "label": "主要指標" },
    { "key": "skill", "label": "Skill / 比較" },
    { "key": "drift", "label": "Drift" },
    { "key": "lastEval", "label": "最後評測" }
  ],
  "criticalModels": []
}
```

### overview.tabs

- `key`：篩選值。`all` 是全部，其餘需對應 `models[].kind`。
- `label`：tab 顯示文字。

### overview.sortOptions

目前前端支援這些 `key`：

- `modelId`
- `kind`
- `status`
- `metric`
- `skill`
- `drift`
- `lastEval`

### overview.criticalModels

```json
{
  "modelId": "predict.chiller_power_01",
  "siteName": "台中空調機房",
  "issueValue": "Skill Score -6.4% / Drift Critical",
  "impactedPoint": "plant.chiller_01.power",
  "duration": "已持續 3 天",
  "severity": "Critical"
}
```

必要欄位：

- `modelId`：點擊後會切到對應 `models[].id`。
- `siteName`：Site 名稱。
- `issueValue`：異常摘要。
- `impactedPoint`：影響測點。
- `duration`：持續時間。
- `severity`：嚴重程度，建議使用 `Critical`。

## 5. models 共用欄位

每個模型都需要放在 `models[]`。

```json
{
  "id": "predict.chiller_power_01",
  "kind": "forecast",
  "modelKind": "forecast / XGBoost Regressor",
  "siteId": "site_txg_hvac_02",
  "siteName": "台中空調機房",
  "modelVersion": "v3.2.1",
  "healthStatus": "Critical",
  "lastEvaluated": "07-15 14:08",
  "serviceInfo": {},
  "metricCards": [],
  "charts": [],
  "featureDrift": []
}
```

共用必要欄位：

- `id`：Model ID，必須唯一。
- `kind`：模型類型，只能是 `forecast`、`anomaly`、`recommender`。
- `modelKind`：模型類型與演算法顯示文字。
- `siteId`：必須對應 `sites[].id`。
- `siteName`：Site 顯示名稱。
- `modelVersion`：模型版本。
- `healthStatus`：模型狀態，建議使用 `Normal`、`Warning`、`Critical`。
- `lastEvaluated`：總覽表格的最後評測時間。
- `serviceInfo`：服務資訊。
- `metricCards`：詳細頁上方指標卡。
- `charts`：詳細頁兩張圖的資料來源。
- `featureDrift`：輸入特徵漂移表格。

## 6. serviceInfo

Forecast 和 Anomaly 會顯示完整服務資訊。

```json
{
  "online": "Online",
  "lastSuccessfulRunAt": "2024-07-15 14:30:00",
  "successRate": "99.1%",
  "errorRate": "0.9%",
  "p95Latency": "180 ms",
  "deployedAt": "2024-07-08 10:30:00"
}
```

Recommender 目前只顯示：

```json
{
  "online": "Online",
  "lastSuccessfulRunAt": "2024-07-15 14:20:00",
  "deployedAt": "2024-07-10 09:20:00"
}
```

## 7. metricCards

詳細頁上方指標卡。

```json
{
  "label": "MAPE",
  "value": "31.5%",
  "tone": "bad",
  "formula": "1 - 31.5 / 17.6"
}
```

欄位：

- `label`：卡片標題。
- `value`：卡片主數值。
- `tone`：顏色狀態，支援 `ok`、`warn`、`bad`。
- `formula`：可選，前端會當成 tooltip。
- `tooltip`：可選，如果有提供會優先使用。

## 8. featureDrift

詳細頁「輸入特徵漂移監測」表格。

```json
{
  "featureName": "flow_rate",
  "psi": 0.132,
  "ksPValue": 0.078,
  "importance": 0.174,
  "status": "Warning"
}
```

必要欄位：

- `featureName`：特徵名稱。
- `psi`：PSI 數值。
- `ksPValue`：KS p-value。
- `importance`：特徵重要度。
- `status`：漂移狀態，建議使用 `Normal`、`Warning`、`Critical`。

前端目前會依 `psi` 重新判斷顯示狀態：

- `psi < 0.10`：Normal
- `0.10 <= psi <= 0.25`：Warning
- `psi > 0.25`：Critical

## 9. charts

每個模型的詳細頁有兩張圖，都從 `models[].charts[]` 讀。

```json
{
  "id": "forecast_metrics_trend",
  "title": "MAE / MAPE / Skill Score 趨勢",
  "subtitle": "評測期間：2024-07-01 ~ 2024-07-15，單位：kW",
  "defaultMetric": "MAPE",
  "type": "bar",
  "xAxis": ["07-01", "07-02", "07-03"],
  "series": [
    { "name": "MAE", "unit": "kW", "values": [42, 45, 44] },
    { "name": "MAPE", "unit": "%", "values": [78, 82, 74] }
  ]
}
```

共用欄位：

- `id`：圖表識別。
- `title`：圖表標題。
- `subtitle`：圖表副標題。
- `type`：`bar` 或 `line`。
- `defaultMetric`：只有 `bar` 需要，代表預設顯示哪個 `series.name`。
- `xAxis`：X 軸標籤。
- `series[].name`：資料序列名稱。
- `series[].unit`：可選，單位。
- `series[].values`：數值陣列。

硬性規則：

- 每個模型至少要有一個 `type: "bar"` chart。
- 每個模型至少要有一個 `type: "line"` chart。
- `xAxis.length` 必須等於每個 `series.values.length`。
- `defaultMetric` 必須等於其中一個 `series[].name`。

## 10. Forecast 模型

`kind: "forecast"` 需要這些專屬欄位。

```json
{
  "targetPoint": "plant.chiller_power_kw",
  "evaluatedAt": "2024-07-15 14:08:00",
  "evaluationPeriod": "2024-07-01 ~ 2024-07-15",
  "driftStatus": "Critical",
  "overview": {
    "mape": "31.5%",
    "skillScore": "-6.4%",
    "drift": "Critical"
  },
  "evaluationDataQuality": {
    "predictionCount": 18420,
    "actualCount": 18001,
    "joinedRows": 16972,
    "excludedActualRows": 1029
  }
}
```

Forecast overview 對應總覽表格：

- `overview.mape`：MAPE 欄。
- `overview.skillScore`：Skill Score 欄。
- `overview.drift` 或 `driftStatus`：Drift 欄。

Forecast charts 建議：

- `type: "bar"`：MAE / MAPE / Skill Score 趨勢。
- `type: "line"`：Actual / Prediction / Baseline。

Forecast quality 對應詳細頁：

- `predictionCount`
- `actualCount`
- `joinedRows`
- `excludedActualRows`

## 11. Anomaly 模型

`kind: "anomaly"` 需要這些專屬欄位。

```json
{
  "targetPoint": "tower.ct_03.temp",
  "evaluatedAt": "2024-07-15 14:12:00",
  "evaluationPeriod": "2024-07-01 ~ 2024-07-15",
  "overview": {
    "events": "126 events",
    "anomalyRate": "7.8%",
    "scoreP99": "0.94"
  },
  "secondaryMetrics": null,
  "scoringStats": {
    "scoredRows": 1662,
    "validRows": 1615,
    "excludedRows": 47,
    "labeledRows": 0
  }
}
```

Anomaly overview 對應總覽表格：

- `overview.events`：Events 欄。
- `overview.anomalyRate`：Anomaly Rate 欄。
- `overview.scoreP99`：Score P99 欄。

如果有標註資料，提供 `secondaryMetrics`：

```json
{
  "title": "標註資料評測",
  "labeledRows": 1460,
  "metrics": [
    { "label": "Precision", "value": "92.4%", "tone": "ok" },
    { "label": "Recall", "value": "88.1%", "tone": "ok" },
    { "label": "F1-score", "value": "90.2%", "tone": "ok" }
  ]
}
```

沒有標註資料時可用：

```json
"secondaryMetrics": null
```

Anomaly charts 建議：

- `type: "bar"`：異常數量 / 異常率 / 異常分數。
- `type: "line"`：P95 Score / P99 Score / Threshold。

Anomaly quality 對應詳細頁：

- `scoredRows`
- `validRows`
- `excludedRows`
- `labeledRows`

## 12. Recommender 模型

`kind: "recommender"` 需要這些專屬欄位。

```json
{
  "targetScope": "3 output points",
  "targetScopePoints": [
    "plant.cooling_schedule",
    "plant.chw_supply_setpoint",
    "plant.chiller_sequence"
  ],
  "statisticsUpdatedAt": "2024-07-15 14:05:00",
  "statisticsPeriod": "2024-07-01 ~ 2024-07-15",
  "overview": {
    "successRate": "97.8%",
    "p95Latency": "240",
    "guardrailRate": "4.6%"
  },
  "executionStats": {
    "totalRuns": 1284,
    "successfulRuns": 1256,
    "failedRuns": 28,
    "guardrailBlockedRuns": 59,
    "adoptedRuns": 752
  }
}
```

Recommender overview 對應總覽表格：

- `targetScope`：Target / Scope 欄。
- `overview.successRate`：Success Rate 欄。
- `overview.p95Latency`：P95 Latency 欄。
- `overview.guardrailRate`：Guardrail Rate 欄。

Recommender charts 建議：

- `type: "bar"`：成功率 / 採用率 / Guardrail 攔截率。
- `type: "line"`：P95 Latency。

Recommender quality 對應詳細頁：

- `totalRuns`
- `successfulRuns`
- `failedRuns`
- `guardrailBlockedRuns`
- `adoptedRuns`

## 13. 最小可顯示模型範例

以下是每個模型至少要具備的欄位輪廓。

```json
{
  "id": "model.id",
  "kind": "forecast",
  "modelKind": "forecast / Algorithm",
  "siteId": "site_id",
  "siteName": "Site 名稱",
  "targetPoint": "target.point",
  "modelVersion": "v1.0.0",
  "healthStatus": "Normal",
  "lastEvaluated": "07-15 14:30",
  "evaluatedAt": "2024-07-15 14:30:00",
  "evaluationPeriod": "2024-07-01 ~ 2024-07-15",
  "overview": {},
  "serviceInfo": {},
  "metricCards": [],
  "charts": [
    {
      "id": "main_trend",
      "title": "主指標趨勢",
      "subtitle": "期間說明",
      "defaultMetric": "Metric A",
      "type": "bar",
      "xAxis": ["07-01"],
      "series": [
        { "name": "Metric A", "values": [1] }
      ]
    },
    {
      "id": "comparison_trend",
      "title": "比較趨勢",
      "subtitle": "單位說明",
      "type": "line",
      "xAxis": ["07-01"],
      "series": [
        { "name": "Line A", "values": [1] }
      ]
    }
  ],
  "featureDrift": [],
  "evaluationDataQuality": {}
}
```

## 14. 交付前檢查清單

資料產生端交付 `data.json` 前，請確認：

- `sites[].id` 都能被 `models[].siteId` 對到。
- `models[].id` 不重複。
- `models[].kind` 只使用 `forecast`、`anomaly`、`recommender`。
- 每個 model 都有 `type: "bar"` 和 `type: "line"` 的 chart。
- 每個 chart 的 `xAxis.length` 等於每個 `series.values.length`。
- `bar` chart 的 `defaultMetric` 等於其中一個 `series[].name`。
- `healthStatus`、`featureDrift[].status` 使用一致的狀態值。
- Critical 模型的 `overview.criticalModels[].modelId` 可以對到 `models[].id`。
