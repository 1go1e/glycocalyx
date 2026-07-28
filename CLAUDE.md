# glycocalyx 知識庫

內皮糖萼（endothelial glycocalyx, eGC）文獻追蹤與知識庫維護專案。

## 每次工作開始前，必讀

- `STATUS.md` — 專案現況、下一批、待決事項。**先讀這份**，它決定本次要做什麼
- `ledger/SCHEMA.md` — `claims.csv` 的欄位規則與寫入限制
- `runbooks/backfill.md` — 回填作業程序（執行回填任務時）
- `source/glycocalyx/v3.md` — 主要原始文件，判定時的背景脈絡

未讀過 `ledger/SCHEMA.md` 就不得修改 `ledger/claims.csv`。

## 不可違反

1. **PMID / DOI / NCT 只能來自檢索工具本次回傳的結構化欄位。**
   禁止憑記憶、推測或仿造格式寫出識別碼。
   查不到就 `evidence` 留空、`status` 設 `unsupported`。

2. `source/` 唯讀。所有修訂建議寫入 `reports/`，不得直接改動原文件。

3. `ledger/claims.csv` 不得刪除任何列。存檔前依 `claim_id` 重新排序。

4. 數值必須與原文逐字相符，不得換算、四捨五入或推估。

5. 每句話前綴分類：〔文獻〕= 文獻主張；〔推論〕= 你的判斷。

6. **不得為具體治療劑量、給藥途徑或用藥時程建立 claim。**
   文獻中的劑量資訊寫入 `note` 供追溯，不進入 `statement`，不取得 `status`。
   `claim_type=therapeutic` 的 statement 只描述介入與結果的方向性關聯
   （例：「SGLT2i 可降低腎絲球 heparanase 表現」），不描述劑量。

   理由：ledger 的 `verified` 是文獻學意義的「有出處」，
   不是臨床意義的「已驗證可用」。這個語意落差在多數 claim 上無害，
   在劑量上會造成實際傷害——一個帶著驗證標記的劑量，讀者會理解為「可以照做」。

## 心態

本專案的正確產出經常是「查不到」。`unsupported` 與 `partial` 是合格結果，不是失敗。
不得為了讓完成數字好看而放寬判定標準。

## 輸出

- 繁體中文，專有名詞首次出現附英文原文。
- 不提供臨床診療建議。PBR、GlycoCheck 等一律註明為研究用指標。

## 目錄

```
STATUS.md          專案現況、下一批、待決事項。每完成一批需更新（見 STATUS.md §7）
ledger/
  claims.csv       主張帳本
  SCHEMA.md        欄位規則（唯讀，變更須經 §11 提案）
  CHANGELOG.md     schema 變更紀錄
runbooks/          作業程序
source/            原始文件，唯讀。依主題分子目錄
  glycocalyx/      糖萼相關
  mitochondria/    線粒體相關
  intake/          已評估但不列為受追蹤文件者，保留供追溯
reports/
  backfill/        批次回填報告
  revisions/       schema 與原文的修訂提案，需人工核准
inbox/             檢索原始回傳，保留不刪減
```

`source/` 的子目錄前綴會寫進 `claims.csv` 的 `source_ref` 欄，
且是網頁分站的依據。新增子目錄前先確認主題邊界，見 `ledger/SCHEMA.md` §2。

## Commit

一次檢索執行 = 一個 commit。格式見 `ledger/SCHEMA.md` §9。
