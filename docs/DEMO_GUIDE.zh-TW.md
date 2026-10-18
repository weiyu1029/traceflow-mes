# TraceFlow：60–90 秒展示指南

目標：讓觀看者看到工單與 SN 的關係、固定站序、異常處理，以及保留下來的事件。這是使用合成資料的學習原型；操作者欄位是事件紀錄文字，沒有正式身分驗證。

## 錄影前準備

1. 在專案根目錄開啟終端機，使用獨立的新暫存資料庫啟動，以免上次練習改動種子狀態：

   ```sh
   traceflow_demo_dir=$(mktemp -d)
   MES_SEED_DEMO=true MES_DB_PATH="$traceflow_demo_dir/traceflow.db" dotnet run --project src/TraceFlow.Web
   ```

2. 開啟 `http://localhost:5080`。若另有指定 URL，使用終端機實際顯示的地址。確認 API 已啟動；可讀取 `http://localhost:5080/api/health`。
3. 預先選擇工單 `WO-DEMO-1001`，確認以下三個 SN。若狀態不同，先停止程式，再以上述指令建立新的暫存目錄與資料庫。

   | SN | 起始狀態 | 展示用途 |
   | --- | --- | --- |
   | `SN-DEMO-0001` | ASSEMBLY 已通過，下一站 TEST | 證明不能跳到 PACK |
   | `SN-DEMO-0002` | TEST 失敗，Held，下一站仍是 TEST | 示範原因、釋放、重測與完成 |
   | `SN-DEMO-0003` | Completed | 讓畫面同時呈現已完成單件 |

4. 放大瀏覽器到文字清楚，將視窗高度調到報工表單與追溯事件容易切換。先練習一次，再換新資料庫開始正式錄影。

固定站序為 `ASSEMBLY → TEST → PACK`。下面以操作目的描述畫面，按鈕名稱以實際版本為準。

## 逐秒腳本

| 時間 | 操作 | 畫面應證明的事 | 可直接念的旁白 |
| --- | --- | --- | --- |
| 0–10 秒 | 顯示 Dashboard、`WO-DEMO-1001` 與三個 SN | 同一工單下可同時有進行中、Held、Completed 的單件 | 「這是 TraceFlow，一個以 C# Minimal API 和 SQLite 製作的製造流程學習原型，資料全部是合成的。」 |
| 10–25 秒 | 選 `SN-DEMO-0001`，填操作者 `Demo Operator`，嘗試在 PACK 報 PASS | 出現拒絕訊息；下一站保持 TEST；沒有新增成功報工事件 | 「這件產品還沒通過 TEST。直接報 PACK 會被後端阻擋，下一站也不會被改掉。」 |
| 25–40 秒 | 切到 `SN-DEMO-0002`，查看 TEST FAIL 與原因，指出 Held | 保留原始失敗事件與原因 | 「第二件在 TEST 失敗後進入 Hold。原始失敗原因仍在追溯紀錄裡。」 |
| 40–55 秒 | 在釋放 Hold 區填操作者與原因，例如 `Demo fixture checked; retest required.`，提交釋放 | 增加 Release 事件；離開 Held，但下一站仍是 TEST | 「我必須留下釋放原因。釋放只允許重新測試，不會自動把 TEST 判成通過。」 |
| 55–70 秒 | 對同一 SN 在 TEST 報 PASS，再在 PACK 報 PASS | TEST 通過後才允許 PACK；最後為 Completed，沒有下一站 | 「重測通過後才能進 PACK。PACK 完成後，這個 SN 才真正完成固定路線。」 |
| 70–85 秒 | 打開同一 SN 的追溯紀錄，指出 FAIL → Release → TEST PASS → PACK PASS；有餘裕再匯出 CSV | 異常與恢復過程可以按事件回看 | 「追溯保留了失敗、釋放與重測歷史。我把流程規則、SQL 交易與驗證步驟一起放在 GitHub。」 |

60 秒版本可縮短開場與省略 CSV；請保留「跳站受阻 → Hold 原因 → 釋放仍需 TEST → PACK 完成 → 歷史事件」五個重點。

## 錄影後核對

- `SN-DEMO-0001` 仍在 TEST，跳站沒有造成狀態前進。
- `SN-DEMO-0002` 已 Completed；追溯能看到原始 TEST FAIL、釋放原因、後續 TEST PASS 與 PACK PASS。
- 重整瀏覽器後，SQLite 中的狀態仍保留。這與重建新資料庫不同。
- 若錄到錯誤或網路中斷，先確認實際狀態，再重新開始。不要把錯誤畫面剪掉後聲稱該次流程全部通過。

## 對照 API 與程式碼

展示以瀏覽器為主；下列實際 API 可供讀程式與檢查網路請求時對照。

| 用途 | API |
| --- | --- |
| 查看固定路線 | `GET /api/meta` |
| 工單清單／單件 | `GET /api/work-orders`、`GET /api/work-orders/WO-DEMO-1001/units` |
| SN 追溯 | `GET /api/units/{sn}/trace` |
| 報 PASS 或 FAIL | `POST /api/units/{sn}/station-events` |
| 釋放 Hold | `POST /api/units/{sn}/release-hold` |
| 事件 CSV | `GET /api/events/export.csv?workOrderNo=WO-DEMO-1001` |

寫入請求包含 `operator` 與 `requestId`。每個新操作使用新的 request ID；重送同一操作才沿用原 ID。不要把同一 ID 換 payload 後當成下一筆報工。

## 分享時的說法

可描述已實際展示與驗證的行為，例如：「這段 demo 展示固定站序、Hold／Release 與 SN 追溯；技術採 ASP.NET Core Minimal API 與參數化 SQLite SQL。」課程尚未完成時寫「正在學習」，不要列為已取得證書。只有真正跑過測試後，才附上對應結果。

本次展示範圍不包含真實產線、正式登入權限、設備／ERP 整合或完整 MES。後續學習與官方來源見 [Learning path](LEARNING_PATH.zh-TW.md)。
