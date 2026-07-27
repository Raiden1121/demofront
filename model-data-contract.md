# 前端儀表板資料契約

這份文件說明模型端需要傳什麼資料，才能顯示完整的模型總覽與詳細頁。

## 1. 責任分工

模型端／後端負責：

- 計算模型品質指標。
- 計算資料品質與漂移指標。
- 決定每個指標的 `status`。
- 決定模型的 `status` 與資料漂移的 `drift_status`。
- 整理圖表需要的 X 軸、數列、矩陣或分箱資料。
- 輸出完整 `data.json`。

前端負責：

- 讀取 `data.json`。
- 顯示總覽、詳細指標與資料品質。
- 將模型狀態畫成綠、黃、紅或灰色圓點，放在 Model ID 前方。
- 從 Site 的全部模型狀態產生 Site 最嚴重狀態圓點。
- 依 JSON 的 `type` 畫圖。
- 將 `null` 顯示成 `--`。

前端不負責計算 MAE、WAPE、R²、F1、PR-AUC、PSI 或 Health Status。

## 2. 最外層結構

```json
{
  "meta": {},
  "rules": {},
  "sites": [],
  "overview": {},
  "models": []
}
```

| 欄位 | 用途 |
|---|---|
| `meta` | 頁面更新時間與預設期間 |
| `rules` | 公式或狀態規則的說明資料 |
| `sites` | 模型總覽的 Site 分組 |
| `overview` | 篩選、排序與嚴重模型資料 |
| `models` | 每個模型的完整指標和圖表 |

所有 JSON key 必須使用 `snake_case`。

## 3. Site

```json
{
  "id": "site_tpe_chw_01",
  "name": "台北冰水主機房",
  "stats": {
    "total": 3,
    "normal": 2,
    "warning": 1,
    "critical": 0
  }
}
```

- `id` 必須能對應 `models[].site_id`。
- `stats` 是 Site 下各模型狀態的摘要。
- Site 圓點不需要獨立欄位；前端會從該 Site 顯示中的 Forecast 與 Anomaly 模型 `status` 取最嚴重狀態。Recommender 雖保留在 API，但目前不納入模型管理頁。

## 4. 模型共用結構

```json
{
  "id": "predict.energy_load_01",
  "kind": "forecast",
  "model_kind": "forecast / LightGBM",
  "site_id": "site_tpe_chw_01",
  "site_name": "台北冰水主機房",
  "target_point": "plant.energy_load",
  "model_version": "v1.0.0",
  "status": "yellow",
  "drift_status": "green",
  "last_evaluated": "07-15 14:30",
  "evaluated_at": "2024-07-15 14:30:00",
  "evaluation_period": "2024-07-01 ~ 2024-07-15",
  "service_info": {},
  "overview_metrics": [],
  "detail_metrics": {
    "core": [],
    "secondary": []
  },
  "data_quality_metrics": [],
  "charts": [],
  "feature_drift": []
}
```

Recommender 可以使用：

- `target_scope`
- `target_scope_points`
- `statistics_updated_at`
- `statistics_period`

## 5. 指標物件

四個位置都使用同一種指標格式：

- `overview_metrics[]`
- `detail_metrics.core[]`
- `detail_metrics.secondary[]`
- `data_quality_metrics[]`

```json
{
  "key": "wape",
  "label": "WAPE",
  "value": 12.4,
  "unit": "%",
  "status": "warning",
  "tooltip": "模型端提供的可選說明"
}
```

| 欄位 | 必要 | 說明 |
|---|---:|---|
| `key` | 是 | 穩定的 `snake_case` 指標識別 |
| `label` | 是 | 畫面顯示名稱 |
| `value` | 是 | 數字、字串或 `null` |
| `unit` | 否 | `%`、`kW`、`ms` 等 |
| `status` | 否 | `green`、`yellow`、`red` |
| `tooltip` | 否 | 指標定義、公式或限制 |

### 缺值

指標暫時沒有數值時仍需保留該指標物件：

```json
{
  "key": "recommendation_uplift",
  "label": "Recommendation Uplift",
  "value": null,
  "unit": "%"
}
```

前端顯示 `--`。不要把未知值填成 `0`，也不要省略必要指標。

### 狀態

模型端負責依自己的門檻產生 `status`。前端只套用顏色：

- `green`：綠色
- `yellow`：黃色
- `red`：紅色
- 未提供或未知值：中性色

## 6. Forecast 指標

### 模型總覽 `overview_metrics`

目前模型管理頁總覽顯示：

1. `rmse`
2. `r_squared`
3. `mae`
4. `mape`

目前詳細頁主要顯示：

1. `mae`
2. `mape`

API 仍可保留 `rmse`、`wape`、`r_squared`、`smape`、`bias`、`baseline_wape`、`skill_score` 與 `evaluation_coverage`，但目前不放入主要詳細指標卡。

### 建議圖表

- Actual、Prediction、Baseline 折線圖
- MAE、MAPE 趨勢圖；Skill Score 目前不顯示
- 預測誤差分布
- 殘差趨勢

## 7. Anomaly 指標

### 模型總覽 `overview_metrics`

目前模型管理頁總覽顯示：

1. `f1_score`
2. `recall`
3. `pr_auc`
4. `precision`

目前詳細頁主要顯示：

1. `precision`
2. `recall`
3. `f1_score`
4. `pr_auc`

API 仍可保留 `evaluation_coverage`、ROC-AUC、誤報率、漏報率、事件數、Threshold、Score P95／P99 等資料，但目前不放入主要詳細指標卡。

只有輸出真正分類機率的模型才使用：

- `brier_score`
- `ece`
- `log_loss`

一般 anomaly score 不是機率時，以上三項使用 `value: null`。

### 建議圖表

- F1、Precision、Recall、PR-AUC 趨勢
- Anomaly Score 與 Threshold 趨勢
- 混淆矩陣
- 異常分數分布

## 8. Recommender 指標

### 模型總覽 `overview_metrics`

固定順序：

1. `ndcg_at_k`
2. `precision_at_k`
3. `adoption_rate`
4. `psi`

### 詳細頁核心 `detail_metrics.core`

1. `ndcg_at_k`
2. `precision_at_k`
3. `recall_at_k`
4. `adoption_rate`
5. `execution_success_rate`

### 詳細頁次要 `detail_metrics.secondary`

- `map_at_k`
- `guardrail_block_rate`
- `invalid_recommendation_rate`
- `p95_latency`
- `recommendation_runs`
- `energy_savings`
- `cost_savings`
- `recommendation_uplift`

成效資料尚未建立時，後三項使用 `value: null`。

### 建議圖表

- NDCG@K、Precision@K、Recall@K 趨勢
- 採用率與執行成功率
- Guardrail 攔截率
- P95 Latency

## 9. 資料品質

每個模型至少提供：

1. `psi`
2. `missing_rate`
3. `data_coverage`
4. `data_freshness`

Forecast 與 Anomaly 可增加：

- `ks_p_value`

包含類別輸入的 Recommender 可增加：

- `entropy`

範例：

```json
[
  {
    "key": "psi",
    "label": "PSI",
    "value": 0.12,
    "unit": "",
    "status": "warning"
  },
  {
    "key": "missing_rate",
    "label": "缺失率",
    "value": 0.9,
    "unit": "%",
    "status": "normal"
  },
  {
    "key": "data_coverage",
    "label": "資料覆蓋率",
    "value": 97.1,
    "unit": "%",
    "status": "normal"
  },
  {
    "key": "data_freshness",
    "label": "資料新鮮度",
    "value": 4,
    "unit": "min",
    "status": "normal"
  }
]
```

## 10. 模型狀態與資料漂移

模型端直接提供：

```json
{
  "status": "yellow",
  "drift_status": "red"
}
```

前端呈現：

- `status: green`：Model ID 前綠點
- `status: yellow`：Model ID 前黃點
- `status: red`：Model ID 前紅點
- 其他值：灰點

模型總覽不顯示獨立的 Health Status 欄位，圓點放在 Model ID 前方。詳細頁也不顯示獨立的 Health Status 項目，圓點同樣放在 Model ID 前方。

Site 圓點會檢查該 Site 顯示中的 Forecast 與 Anomaly 模型，依下列順序取最嚴重狀態：

```text
critical > warning > normal > unknown
```

`status` 是模型整體健康狀態；`drift_status` 是輸入資料漂移狀態。Model ID 主燈號只讀 `status`，不讀 `drift_status`。

前端不根據模型指標重新計算狀態。狀態文字只存在 tooltip 與無障礙標籤。

## 11. Feature Drift

```json
{
  "feature_name": "flow_rate",
  "psi": 0.132,
  "ks_p_value": 0.078,
  "importance": 0.174,
  "status": "warning"
}
```

前端直接使用 `status`，不根據 `psi` 重新判斷。
漂移表格目前只顯示 `feature_name`、`psi`、`ks_p_value` 與 `status`；`importance` 可保留在 API，但不顯示。

## 12. Line 與 Bar 圖表

```json
{
  "id": "actual_prediction_trend",
  "type": "line",
  "title": "Actual vs Prediction",
  "subtitle": "2024-07-01 ~ 2024-07-15",
  "x_axis": ["07-01", "07-02", "07-03"],
  "series": [
    {
      "key": "actual",
      "name": "Actual",
      "unit": "kW",
      "values": [320.5, 335.2, 341.8]
    },
    {
      "key": "prediction",
      "name": "Prediction",
      "unit": "kW",
      "values": [318.1, null, 346.2]
    }
  ]
}
```

硬性規則：

- `type` 使用 `line` 或 `bar`。
- `x_axis` 必須是陣列。
- 每個 `series.values.length` 必須等於 `x_axis.length`。
- 資料缺值使用 `null`，不可自行補 `0`。
- `bar` 可提供 `default_metric`，值需對應 `series[].key` 或 `series[].name`。

## 13. Confusion Matrix

```json
{
  "id": "confusion_matrix",
  "type": "confusion_matrix",
  "title": "混淆矩陣",
  "subtitle": "實際標籤與模型判斷結果",
  "x_axis": ["Predicted Normal", "Predicted Anomaly"],
  "y_axis": ["Actual Normal", "Actual Anomaly"],
  "matrix": [
    [1412, 7],
    [5, 36]
  ]
}
```

矩陣位置：

```text
TN  FP
FN  TP
```

`matrix` 必須是 2 x 2，兩個軸都必須有兩個標籤。

## 14. Histogram

```json
{
  "id": "anomaly_score_distribution",
  "type": "histogram",
  "title": "異常分數分布",
  "subtitle": "Anomaly Score 分箱計數",
  "bins": [
    {
      "start": 0,
      "end": 0.1,
      "count": 510
    },
    {
      "start": 0.1,
      "end": 0.2,
      "count": 424
    }
  ]
}
```

每個 bin 都需要 `start`、`end` 與數值型 `count`。

## 15. 圖表錯誤處理

- 不支援的 `type`：該卡顯示「不支援的圖表類型」。
- `x_axis` 與 `values` 長度不同：該卡顯示「資料格式錯誤」。
- 混淆矩陣不是 2 x 2：該卡顯示「資料格式錯誤」。
- Histogram 缺少有效 `bins`：該卡顯示「資料格式錯誤」。
- 單張圖錯誤不影響其他圖表、指標或模型。

## 16. 交付檢查

輸出 `data.json` 前確認：

- 所有 key 都是 `snake_case`。
- `models[].id` 不重複。
- `models[].site_id` 能對應 `sites[].id`。
- `kind` 只使用 `forecast`、`anomaly`、`recommender`。
- 每個模型都有該種類要求的 `overview_metrics`。
- 每個模型有 5 個必要核心指標。
- 必要指標沒有值時使用 `value: null`，不省略物件。
- 每個模型都有 PSI、缺失率、資料覆蓋率、資料新鮮度。
- 指標狀態、模型 `status` 與 `drift_status` 由模型端決定。
- Line／Bar 的 X 軸長度等於每個序列長度。
- 圖表數值由模型端提供，前端不重新聚合。
