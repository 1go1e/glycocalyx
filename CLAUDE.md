# glycocalyx 知識庫

內皮糖萼（endothelial glycocalyx, eGC）文獻追蹤與知識庫維護專案。

## 每次工作開始前，必讀

- `ledger/SCHEMA.md` — `claims.csv` 的欄位規則與寫入限制
- `runbooks/backfill.md` — 回填作業程序（執行回填任務時）
- `source/glycocalyx_v3.md` — 原始文件，判定時的背景脈絡

未讀過 `ledger/SCHEMA.md` 就不得修改 `ledger/claims.csv`。

## 不可違反

1. **PMID / DOI / NCT 只能來自檢索工具本次回傳的結構化欄位。**
   禁止憑記憶、推測或仿造格式寫出識別碼。
   查不到就 `evidence` 留空、`status` 設 `unsupported`。

2. `source/` 唯讀。所有修訂建議寫入 `reports/`，不得直接改動原文件。

3. `ledger/claims.csv` 不得刪除任何列。存檔前依 `claim_id` 重新排序。

4. 數值必須與原文逐字相符，不得換算、四捨五入或推估。

5. 每句話前綴分類：〔文獻〕= 文獻主張；〔推論〕= 你的判斷。

## 心態

本專案的正確產出經常是「查不到」。`unsupported` 與 `partial` 是合格結果，不是失敗。
不得為了讓完成數字好看而放寬判定標準。

## 輸出

- 繁體中文，專有名詞首次出現附英文原文。
- 不提供臨床診療建議。PBR、GlycoCheck 等一律註明為研究用指標。

## 目錄

```
ledger/     claims.csv（主張帳本）、SCHEMA.md（欄位規則）
runbooks/   作業程序
source/     原始文件，唯讀
reports/    報告與修訂建議
inbox/      檢索原始回傳，保留不刪減
```

## Commit

一次檢索執行 = 一個 commit。格式見 `ledger/SCHEMA.md` §9。
