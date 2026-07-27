# 模型儀表板

## 畫面指標與用途

模型總覽用來快速比較模型品質與資料狀態；詳細頁則用來判斷問題來自模型、輸入資料或服務執行。

### 共通欄位

| 欄位／指標 | 用途 |
|---|---|
| Model ID 前的狀態點 | 快速辨識模型整體狀態；綠色為 Normal、黃色為 Warning、紅色為 Critical。 |
| Site 前的狀態點 | 顯示該 Site 全部模型中最嚴重的狀態。 |
| Last Evaluated | 確認目前看到的模型指標是否仍在有效評估時間內。 |
| 評測覆蓋率 | 顯示可成功配對並納入評估的資料比例；過低時，其他模型指標可能不具代表性。 |

### Forecast 數值預測

模型總覽：

| 欄位／指標 | 用途 |
|---|---|
| R² | 判斷模型解釋資料變化的能力。越接近 1 越好；小於 0 代表可能比直接使用平均值還差。 |
| RMSE | 對較大的預測誤差給予更高懲罰，用來發現模型是否存在嚴重的大誤差。越低越好。 |
| MAE | 顯示平均預測差距，使用原始單位，方便維運快速理解誤差大小。 |
| MAPE | 顯示平均百分比誤差；實際值接近 0 時可能失真，需搭配其他指標判讀。 |

詳細頁：

| 欄位／指標 | 用途 |
|---|---|
| MAE | 平均絕對誤差，以原始單位呈現，容易直接理解平均差多少 kW、°C 或其他單位。 |
| MAPE | 每筆資料的平均百分比誤差；實際值接近 0 時容易失真，因此只作補充判讀。 |
| Actual／Prediction／Baseline 圖 | 直接比較實際值、模型預測與基準預測是否同步，以及誤差發生在哪些時間。 |
| 資料品質指標 | 在詳細頁獨立檢查 PSI、缺失率、覆蓋率與資料新鮮度，不混入模型表現指標。 |

API 仍可保留 WAPE、SMAPE、Bias、Baseline、Skill Score 等資料，但目前不放在主要 UI。

Forecast 趨勢圖目前只顯示 MAE 與 MAPE，Skill Score 不顯示。

### Anomaly 異常偵測

模型總覽：

| 欄位／指標 | 用途 |
|---|---|
| F1 Score | Precision 與 Recall 的綜合分數，用來平衡誤報和漏報。越高越好。 |
| Recall | 實際異常中被模型成功找出的比例，用來判斷漏掉多少異常。越高越好。 |
| PR-AUC | 評估不同判定門檻下 Precision 與 Recall 的整體表現，適合異常樣本較少的情境。 |
| Precision | 模型判定為異常的資料中，真正異常的比例；低時代表誤報較多。 |

詳細頁：

| 欄位／指標 | 用途 |
|---|---|
| Precision | 模型判定為異常的資料中，真正異常的比例。低時代表誤報較多。 |
| Recall | 所有實際異常中，被模型抓到的比例。低時代表漏報較多。 |
| F1 Score | 同時考量 Precision 與 Recall，方便用單一數值比較模型版本。 |
| PR-AUC | 檢查模型在不同 Threshold 下對少數異常樣本的整體辨識能力。 |
| 資料品質指標 | 在詳細頁獨立檢查 PSI、缺失率、覆蓋率與資料新鮮度。 |

API 仍可保留 ROC-AUC、誤報率、漏報率、Threshold、Brier Score、ECE、Log Loss 等資料，但目前不放在主要 UI。

### Recommender 推薦模型

Recommender 的資料契約與指標先保留，方便未來加回前端。目前模型管理頁只顯示 Forecast 與 Anomaly，不顯示 Recommender。

模型總覽：

| 欄位／指標 | 用途 |
|---|---|
| NDCG@K | 同時考量前 K 名推薦的正確性與排序位置，越重要的推薦排得越前面，分數越高。 |
| Precision@K | 前 K 個推薦中有效或相關推薦的比例。越高代表推薦清單越精準。 |
| 採用率 | 推薦結果實際被使用或接受的比例，用來觀察推薦是否具現場可用性。 |
| PSI | 檢查推薦模型輸入資料是否偏離訓練或基準期間。 |

詳細頁：

| 欄位／指標 | 用途 |
|---|---|
| NDCG@K | 評估推薦順序是否把較重要的選項排在前面。 |
| Precision@K | 評估前 K 個推薦中有多少是有效推薦。 |
| Recall@K | 所有應被推薦的項目中，有多少出現在前 K 個結果內。 |
| MAP@K | 綜合多次推薦結果的平均排序精準度，用來比較不同模型版本。 |
| 採用率 | 顯示推薦被實際採用的比例。 |
| 執行成功率 | 顯示推薦服務是否成功完成推論與輸出流程。 |
| Guardrail 攔截率 | 推薦結果因安全或操作限制而被攔截的比例；過高可能表示模型輸出不符合現場限制。 |
| 無效推薦率 | 無法執行、資料不足或不符合需求的推薦比例。越低越好。 |
| P95 Latency | 95% 的請求可在此時間內完成，用來監控較慢請求的延遲。 |
| 推薦執行次數 | 顯示評估期間的推薦樣本量，避免在樣本過少時誤判其他比例。 |
| 節省能耗／節省成本 | 顯示推薦被採用後帶來的實際營運效益。 |
| Recommendation Uplift | 比較採用推薦和未採用推薦時的成效差異。 |
| 排序品質趨勢圖 | 觀察 NDCG@K、Precision@K、Recall@K 是否隨時間下降。 |
| 採用率／成功率／延遲圖 | 分別檢查推薦是否被接受、服務是否穩定，以及回應速度是否惡化。 |

### 共用資料品質

| 欄位／指標 | 用途 |
|---|---|
| PSI | 檢查整體或單一特徵的資料分布是否漂移。 |
| 缺失率 | 顯示輸入資料缺值比例；過高可能讓模型指標下降或無法推論。 |
| 資料覆蓋率 | 顯示預期資料中實際可使用的比例。 |
| 資料新鮮度 | 顯示最新資料距離目前時間多久，用來發現資料管線延遲。 |
| KS p-value | 判斷目前資料與基準資料是否有顯著分布差異；數值越低，差異通常越明顯。 |
| Entropy | 觀察類別資料的分散程度，協助發現類別比例過度集中或突然改變。 |

這個 demo 是純前端模型監控儀表板。模型、站點、指標、資料品質與圖表資料都集中在 `data.json`，`index.html` 不保存模型數值。

前端載入方式：

```js
fetch("./data.json")
```

資料流：

```text
模型端計算指標與圖表數列
  -> 輸出 data.json
  -> 前端讀取 JSON
  -> 前端排版並繪製圖表
```

前端不計算 MAE、WAPE、F1、PSI、Health Status 或圖表統計值，只負責格式化與顯示。

## 啟動

瀏覽器使用 `fetch` 讀取 JSON，因此請透過 HTTP server 開啟：

```bash
python3 -m http.server 8000
```

瀏覽：

```text
http://localhost:8000/
```

直接用 `file://` 開啟可能會被瀏覽器阻擋。

## 命名規則

- 所有 JSON key 使用 `snake_case`。
- `label`、`title`、`name` 是畫面文字，可以保留 `R²`、`PR-AUC`、`NDCG@K` 等正式名稱。
- JSON 不是 Python Class，因此不使用 PascalCase key。

## 狀態點顯示

模型狀態使用模型物件的 `status`，只顯示圓點，放在 `Model ID` 前方，不另外占用 `Health Status` 欄位：

- 綠點：Normal
- 黃點：Warning
- 紅點：Critical
- 灰點：未知值

狀態文字只放在 `title` 與 `aria-label`，不占用表格欄位空間。

Site 名稱前的圓點由前端檢查該 Site 目前顯示的 Forecast 與 Anomaly 模型後，取最嚴重的狀態：

```text
critical > warning > normal > unknown
```

Site 狀態不受搜尋篩選影響；Recommender 目前不納入模型管理頁的 Site 狀態統計。

`drift_status` 是資料漂移狀態，只用於資料品質／漂移資訊，不拿來當 Model ID 的主燈號。

輸入特徵漂移表格只顯示 `Feature Name`、`PSI`、`KS P-value` 與 `Status`；`Importance` 即使存在於 JSON，也不顯示在目前 UI。

## 模型詳細頁

每個模型使用三組 JSON 指標：

- `detail_metrics.core`：詳細頁核心指標卡。
- `detail_metrics.secondary`：次要與診斷指標。
- `data_quality_metrics`：PSI、缺失率、覆蓋率、資料新鮮度等。

單一指標格式：

```json
{
  "key": "wape",
  "label": "WAPE",
  "value": 12.4,
  "unit": "%",
  "status": "warning",
  "tooltip": "可選說明"
}
```

規則：

- `value` 可以是數字、字串或 `null`。
- `value: null` 顯示為 `--`。
- `status` 可使用 `normal`、`warning`、`critical`。
- 未提供 `status` 時使用中性色。
- 指標門檻由模型端決定，前端不重新判斷。

## 圖表

每個模型的圖表都放在 `models[].charts[]`，前端依 `type` 動態建立圖表卡，不限制圖表數量。

支援：

- `line`
- `bar`
- `confusion_matrix`
- `histogram`

`line` 與 `bar` 的 `x_axis.length` 必須等於每個 `series.values.length`。缺值使用 `null`；折線會顯示斷點，不會自動補成 `0`。

圖表格式與三類模型的完整資料契約請看 [model-data-contract.md](./model-data-contract.md)。
