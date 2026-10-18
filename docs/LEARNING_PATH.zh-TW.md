# TraceFlow 學習路線

查核日期：2026-10-19。以下是建議學習順序，**不代表作者已完成課程或取得證書**。專案使用 ASP.NET Core Minimal API、`Microsoft.Data.Sqlite`、參數化 SQL 與 SQLite；課程範例與專案實作的差異列在下方。

## 三個互補方向

| 方向 | 官方課程或教材 | 級別／官方時間 | 費用與可分享成果 | 對接 TraceFlow |
| --- | --- | --- | --- | --- |
| 製造業務概念 | University at Buffalo / SUNY：[Digital Manufacturing & Design](https://www.coursera.org/learn/digital-manufacturing-design/) | Beginner；2 modules，約 10 小時 | 提供可分享 course certificate；證書體驗需付費。公開頁未能核實所在地價格，試用／訂閱資格以實際結帳頁為準 | digital thread → 同一 SN 的跨站事件；data / closed loop → 報工結果與追溯 |
| C# HTTP API | Microsoft：[Create a Minimal API with ASP.NET Core](https://learn.microsoft.com/en-us/aspnet/core/tutorials/min-web-api?view=aspnetcore-10.0) | 官方教學文章；未列固定總時數 | 免費閱讀；這篇教學沒有 module badge | route handler、JSON 請求、狀態碼、查詢與寫入 API |
| 關聯式資料與持久化 | Microsoft Learn：[Explore fundamental relational data concepts](https://learn.microsoft.com/en-us/training/modules/explore-relational-data-offerings/) | Beginner；7 units，37 分鐘 | 免費 module；登入並自行完成要求後可獲 Learn achievement | 主外鍵、SN 唯一性、工單／單件／事件的關聯與 SQL |

Coursera 頁面的「Enroll for free」不保證免費取得完整課程或證書。該課程 FAQ 說明 certificate experience 要付費，可能提供 trial 或無證書選項；本文件沒有替使用者報名，也不假設任何付費方案。課程內容為數位製造導論，並非 MES 產品實作認證。[課程與費用說明](https://www.coursera.org/learn/digital-manufacturing-design/)

關聯式 module 的 37 分鐘取自 Microsoft 的[官方 learning path 索引](https://learn.microsoft.com/en-gb/training/paths/azure-data-fundamentals-explore-relational-data/)。所有時數是平台學習量標示，並非完成 TraceFlow 的工時估計。

## 先走這條免費路線

1. **先定義生產流程。** 閱讀 ISA 官方 [Manufacturing Execution Systems 第二版公開節錄](https://www.isa.org/getmedia/6f55cb74-0bf1-4fd4-aac9-9d0b1be443eb/Manufacturing-Execution-Systems-Second-Ed-Excerpt-and-Table-of-Contents.pdf) 的 Chapter 2 開頭（書頁 15、PDF 第 16 頁）。把工廠層級的流程、工作站與逐件活動記錄，縮小為 `ASSEMBLY → TEST → PACK`。這份免費節錄沒有課程證書，也不等於讀完完整書。
2. **跑通 Minimal API。** 依官方教學認識 HTTP handler 與回應。接著對照專案 API，解釋建立工單、報工與查詢追溯的用途；不要把教學的 todo 名稱直接換字就視為完成製造規則。
3. **補關聯式觀念。** 完成上表的 relational data module，再畫出工單、SN 與事件的關係。說清楚為何 SN 不可重複、事件應保留歷史，以及「目前站點」與「曾經發生的事件」有何差異。
4. **讀實際使用的資料存取文件。** 對照 Microsoft 的 [Microsoft.Data.Sqlite 概觀](https://learn.microsoft.com/en-us/dotnet/standard/data/sqlite/)、[參數化 SQL](https://learn.microsoft.com/en-us/dotnet/standard/data/sqlite/parameters) 與[交易](https://learn.microsoft.com/en-us/dotnet/standard/data/sqlite/transactions)。在程式中找到參數、`BeginTransaction`、狀態更新與事件寫入的同一提交範圍。
5. **用實作證明理解。** 依 [Demo guide](DEMO_GUIDE.zh-TW.md) 示範跳站受阻、Hold 原因、釋放後仍需重測，以及 PACK 完成。閱讀測試結果後再描述通過哪些驗證。

SQLite 交易可將多個 SQL 操作一起提交或回滾；同時僅有一個交易能持有待寫入變更。這是理解本專案資料一致性的起點，並不自動代表已處理所有並行、逾時與重試情境。[Microsoft.Data.Sqlite transactions](https://learn.microsoft.com/en-us/dotnet/standard/data/sqlite/transactions)

## 想要可分享的 Learn achievement

除了上表的關聯式 module，可選這兩項補充，依自己的進度完成：

| 補充 module | 官方標示 | 如何使用 |
| --- | --- | --- |
| [Create a web API with ASP.NET Core controllers](https://learn.microsoft.com/en-us/training/modules/build-web-api-aspnet-core/) | Beginner；41 分鐘；免費 | 學 REST、CRUD 與 HTTP 測試。課程採 controllers，本專案採 Minimal API；API 概念可比較，程式架構不同 |
| [Use a database with minimal API, Entity Framework Core, and ASP.NET Core](https://learn.microsoft.com/en-us/training/modules/build-web-api-minimal-database/) | Beginner；35 分鐘；免費 | 學持久化與 SQLite 的概念。課程使用 EF Core；**TraceFlow 刻意使用原始參數化 SQL，未使用 EF Core**。實際交易實作請看上面的 Microsoft.Data.Sqlite 文件 |

41／35 分鐘取自 Microsoft 的[官方訓練索引](https://learn.microsoft.com/en-us/training/browse/?expanded=dotnet&products=aspnet-core)。本專案沒有 `DbContext` 或 EF Core `SaveChanges`；不得以 EF Core 範例描述這份程式已實作的機制。

Microsoft Learn 的內容可免費閱讀，無須訂閱；個人 profile 可記錄完成的 module badge 與 learning path trophy，並分享至社群。**Learn achievement 不等於 Microsoft Certification 或通過證照考試**。只有自己登入、完成平台要求並實際看到成果後，才分享對應連結。[官方費用與 achievement FAQ](https://learn.microsoft.com/en-us/training/support/faq)

## 每一步留下可檢查的成果

| 學習步驟 | 放在 GitHub 或分享中的證據 |
| --- | --- |
| 製造概念 | 固定站序圖；工單、SN、Hold、Release 的定義；原型範圍 |
| API | 真實 endpoint、請求範例、成功與拒絕回應 |
| SQL／交易 | schema、參數化 SQL、狀態與事件一起提交的程式碼、相關測試 |
| 展示 | 60–90 秒操作錄影、可重跑步驟、實際測試指令與結果 |
| 課程 | 真實完成日期與平台 achievement／certificate 連結；尚未完成就寫 planned 或 currently learning |

分享時可寫：「我正在透過官方教材學習 ASP.NET Core 與 SQLite，並將工單、站序與追溯觀念應用到 TraceFlow 學習原型。」不要將這個合成資料專案描述為真實 MES 部署、ISA-95 認證、具備正式登入權限或已完成設備／ERP 整合。
