# 前端儀表板資料契約

這份文件說明要讓目前 `index.html` 顯示完整儀表板時，`data.json` 需要提供哪些資料。

前端載入方式：

```js
fetch("./data.json")
```

前端只負責顯示、排序、篩選、切換圖表與繪製 SVG。所有模型資料、圖表數列、統計數字、狀態與時間都應由 `data.json` 提供。

命名規則依 Python / PEP 8 慣例：JSON key 使用 `snake_case`。`label`、`title`、`name` 這類畫面顯示文字可維持原本大小寫；JSON 不是 Class 定義，所以 PascalCase 不適用。

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
  "page_updated_at": "2024-07-15 14:45:00",
  "default_period": {
    "label": "2024-07-01 ~ 2024-07-15",
    "start": "2024-07-01",
    "end": "2024-07-15"
  }
}
```

前端使用：

- `page_updated_at`：總覽 Critical 區塊的 Last updated。
- `default_period.label`：模型沒有自己的期間時可當 fallback。

## 3. sites

提供總覽頁「依 Site 分組的模型清單」。

```json
{
  "id": "site_tpe_chw_01",
  "name": "台北冰水主機房",
  "health_status": "Warning",
  "stats": {
    "total": 3,
    "normal": 2,
    "warning": 1,
    "critical": 0
  }
}
```

必要欄位：

- `id`：Site ID，必須能對應到 `models[].site_id`。
- `name`：Site 顯示名稱。
- `health_status`：Site 整體狀態，建議使用 `Normal`、`Warning`、`Critical`。
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
  "sort_options": [
    { "key": "model_id", "label": "Model ID" },
    { "key": "kind", "label": "Kind" },
    { "key": "status", "label": "Status" },
    { "key": "metric", "label": "主要指標" },
    { "key": "skill", "label": "Skill / 比較" },
    { "key": "drift", "label": "Drift" },
    { "key": "lastEval", "label": "最後評測" }
  ],
  "critical_models": []
}
```

### overview.tabs

- `key`：篩選值。`all` 是全部，其餘需對應 `models[].kind`。
- `label`：tab 顯示文字。

### overview.sort_options

目前前端支援這些 `key`：

- `model_id`
- `kind`
- `status`
- `metric`
- `skill`
- `drift`
- `lastEval`

### overview.critical_models

```json
{
  "model_id": "predict.chiller_power_01",
  "site_name": "台中空調機房",
  "issue_value": "Skill Score -6.4% / Drift Critical",
  "impacted_point": "plant.chiller_01.power",
  "duration": "已持續 3 天",
  "severity": "Critical"
}
```

必要欄位：

- `model_id`：點擊後會切到對應 `models[].id`。
- `site_name`：Site 名稱。
- `issue_value`：異常摘要。
- `impacted_point`：影響測點。
- `duration`：持續時間。
- `severity`：嚴重程度，建議使用 `Critical`。

## 5. models 共用欄位

每個模型都需要放在 `models[]`。

```json
{
  "id": "predict.chiller_power_01",
  "kind": "forecast",
  "model_kind": "forecast / XGBoost Regressor",
  "site_id": "site_txg_hvac_02",
  "site_name": "台中空調機房",
  "model_version": "v3.2.1",
  "health_status": "Critical",
  "last_evaluated": "07-15 14:08",
  "service_info": {},
  "metric_cards": [],
  "charts": [],
  "feature_drift": []
}
```

共用必要欄位：

- `id`：Model ID，必須唯一。
- `kind`：模型類型，只能是 `forecast`、`anomaly`、`recommender`。
- `model_kind`：模型類型與演算法顯示文字。
- `site_id`：必須對應 `sites[].id`。
- `site_name`：Site 顯示名稱。
- `model_version`：模型版本。
- `health_status`：模型狀態，建議使用 `Normal`、`Warning`、`Critical`。
- `last_evaluated`：總覽表格的最後評測時間。
- `service_info`：服務資訊。
- `metric_cards`：詳細頁上方指標卡。
- `charts`：詳細頁兩張圖的資料來源。
- `feature_drift`：輸入特徵漂移表格。

## 6. service_info

Forecast 和 Anomaly 會顯示完整服務資訊。

```json
{
  "online": "Online",
  "last_successful_run_at": "2024-07-15 14:30:00",
  "success_rate": "99.1%",
  "error_rate": "0.9%",
  "p95_latency": "180 ms",
  "deployed_at": "2024-07-08 10:30:00"
}
```

Recommender 目前只顯示：

```json
{
  "online": "Online",
  "last_successful_run_at": "2024-07-15 14:20:00",
  "deployed_at": "2024-07-10 09:20:00"
}
```

## 7. metric_cards

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

## 8. feature_drift

詳細頁「輸入特徵漂移監測」表格。

```json
{
  "feature_name": "flow_rate",
  "psi": 0.132,
  "ks_p_value": 0.078,
  "importance": 0.174,
  "status": "Warning"
}
```

必要欄位：

- `feature_name`：特徵名稱。
- `psi`：PSI 數值。
- `ks_p_value`：KS p-value。
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
  "default_metric": "MAPE",
  "type": "bar",
  "x_axis": ["07-01", "07-02", "07-03"],
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
- `default_metric`：只有 `bar` 需要，代表預設顯示哪個 `series.name`。
- `x_axis`：X 軸標籤。
- `series[].name`：資料序列名稱。
- `series[].unit`：可選，單位。
- `series[].values`：數值陣列。

硬性規則：

- 每個模型至少要有一個 `type: "bar"` chart。
- 每個模型至少要有一個 `type: "line"` chart。
- `x_axis.length` 必須等於每個 `series.values.length`。
- `default_metric` 必須等於其中一個 `series[].name`。

## 10. Forecast 模型

`kind: "forecast"` 需要這些專屬欄位。

```json
{
  "target_point": "plant.chiller_power_kw",
  "evaluated_at": "2024-07-15 14:08:00",
  "evaluation_period": "2024-07-01 ~ 2024-07-15",
  "driftStatus": "Critical",
  "overview": {
    "mape": "31.5%",
    "skill_score": "-6.4%",
    "drift": "Critical"
  },
  "evaluation_data_quality": {
    "prediction_count": 18420,
    "actual_count": 18001,
    "joined_rows": 16972,
    "excluded_actual_rows": 1029
  }
}
```

Forecast overview 對應總覽表格：

- `overview.mape`：MAPE 欄。
- `overview.skill_score`：Skill Score 欄。
- `overview.drift` 或 `driftStatus`：Drift 欄。

Forecast charts 建議：

- `type: "bar"`：MAE / MAPE / Skill Score 趨勢。
- `type: "line"`：Actual / Prediction / Baseline。

Forecast quality 對應詳細頁：

- `prediction_count`
- `actual_count`
- `joined_rows`
- `excluded_actual_rows`

## 11. Anomaly 模型

`kind: "anomaly"` 需要這些專屬欄位。

```json
{
  "target_point": "tower.ct_03.temp",
  "evaluated_at": "2024-07-15 14:12:00",
  "evaluation_period": "2024-07-01 ~ 2024-07-15",
  "overview": {
    "events": "126 events",
    "anomaly_rate": "7.8%",
    "score_p99": "0.94"
  },
  "secondary_metrics": null,
  "scoring_stats": {
    "scored_rows": 1662,
    "valid_rows": 1615,
    "excluded_rows": 47,
    "labeled_rows": 0
  }
}
```

Anomaly overview 對應總覽表格：

- `overview.events`：Events 欄。
- `overview.anomaly_rate`：Anomaly Rate 欄。
- `overview.score_p99`：Score P99 欄。

如果有標註資料，提供 `secondary_metrics`：

```json
{
  "title": "標註資料評測",
  "labeled_rows": 1460,
  "metrics": [
    { "label": "Precision", "value": "92.4%", "tone": "ok" },
    { "label": "Recall", "value": "88.1%", "tone": "ok" },
    { "label": "F1-score", "value": "90.2%", "tone": "ok" }
  ]
}
```

沒有標註資料時可用：

```json
"secondary_metrics": null
```

Anomaly charts 建議：

- `type: "bar"`：異常數量 / 異常率 / 異常分數。
- `type: "line"`：P95 Score / P99 Score / Threshold。

Anomaly quality 對應詳細頁：

- `scored_rows`
- `valid_rows`
- `excluded_rows`
- `labeled_rows`

## 12. Recommender 模型

`kind: "recommender"` 需要這些專屬欄位。

```json
{
  "target_scope": "3 output points",
  "target_scope_points": [
    "plant.cooling_schedule",
    "plant.chw_supply_setpoint",
    "plant.chiller_sequence"
  ],
  "statistics_updated_at": "2024-07-15 14:05:00",
  "statistics_period": "2024-07-01 ~ 2024-07-15",
  "overview": {
    "success_rate": "97.8%",
    "p95_latency": "240",
    "guardrail_rate": "4.6%"
  },
  "execution_stats": {
    "total_runs": 1284,
    "successful_runs": 1256,
    "failed_runs": 28,
    "guardrail_blocked_runs": 59,
    "adopted_runs": 752
  }
}
```

Recommender overview 對應總覽表格：

- `target_scope`：Target / Scope 欄。
- `overview.success_rate`：Success Rate 欄。
- `overview.p95_latency`：P95 Latency 欄。
- `overview.guardrail_rate`：Guardrail Rate 欄。

Recommender charts 建議：

- `type: "bar"`：成功率 / 採用率 / Guardrail 攔截率。
- `type: "line"`：P95 Latency。

Recommender quality 對應詳細頁：

- `total_runs`
- `successful_runs`
- `failed_runs`
- `guardrail_blocked_runs`
- `adopted_runs`

## 13. 最小可顯示模型範例

以下是每個模型至少要具備的欄位輪廓。

```json
{
  "id": "model.id",
  "kind": "forecast",
  "model_kind": "forecast / Algorithm",
  "site_id": "site_id",
  "site_name": "Site 名稱",
  "target_point": "target.point",
  "model_version": "v1.0.0",
  "health_status": "Normal",
  "last_evaluated": "07-15 14:30",
  "evaluated_at": "2024-07-15 14:30:00",
  "evaluation_period": "2024-07-01 ~ 2024-07-15",
  "overview": {},
  "service_info": {},
  "metric_cards": [],
  "charts": [
    {
      "id": "main_trend",
      "title": "主指標趨勢",
      "subtitle": "期間說明",
      "default_metric": "Metric A",
      "type": "bar",
      "x_axis": ["07-01"],
      "series": [
        { "name": "Metric A", "values": [1] }
      ]
    },
    {
      "id": "comparison_trend",
      "title": "比較趨勢",
      "subtitle": "單位說明",
      "type": "line",
      "x_axis": ["07-01"],
      "series": [
        { "name": "Line A", "values": [1] }
      ]
    }
  ],
  "feature_drift": [],
  "evaluation_data_quality": {}
}
```

## 14. 交付前檢查清單

資料產生端交付 `data.json` 前，請確認：

- `sites[].id` 都能被 `models[].site_id` 對到。
- `models[].id` 不重複。
- `models[].kind` 只使用 `forecast`、`anomaly`、`recommender`。
- 每個 model 都有 `type: "bar"` 和 `type: "line"` 的 chart。
- 每個 chart 的 `x_axis.length` 等於每個 `series.values.length`。
- `bar` chart 的 `default_metric` 等於其中一個 `series[].name`。
- `health_status`、`feature_drift[].status` 使用一致的狀態值。
- Critical 模型的 `overview.critical_models[].model_id` 可以對到 `models[].id`。
