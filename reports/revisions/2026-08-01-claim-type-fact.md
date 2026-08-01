# 修訂提案：`claim_type=fact` 不在 SCHEMA 的合法值之內 — 2026-08-01

執行 STATUS §5「`claim_type` 與 SCHEMA §2 定義的落差」選項 A 時，
全帳本掃描發現的問題比原提案大一級。**六列已依核准改完**（見 §4），
但底下這一項需要人工決定後才能動。

---

## 1. 發現

`ledger/SCHEMA.md` §2 的 `claim_type` 合法值只有五個：

```
definition / mechanism / measurement / epidemiology / therapeutic
```

**`fact` 不在其中，而且從 v1 到 v7 從未出現過**（`ledger/CHANGELOG.md` 全文無此值）。

但 `runbooks/extraction.md` §6 的欄位表寫的是：

```
| `claim_type` | `fact` / `mechanism` / `therapeutic` / `epidemiology` |
```

該 runbook 於 2026-07-31 隨 schema v7 新建。四批抽取全部照它填，
結果是 **32 列**（改完六列後）使用一個 SCHEMA 未定義的值。

## 2. 波及範圍

```
mitochondria/axis        6 列
glycocalyx/heparanase   26 列
──────────────────────────
                        32 列
```

依 `section` 分布：`5.6.2` 11、`5.6.1` 9、`1.2.1` 3、`3.4` 2、`4.2.1` 2、
`6.3` 2、`2.3.2` 1、`3.2` 1、`4.1` 1。

**原始 121 列無一使用 `fact`。** 這個值是 2026-07-31 才進入帳本的，
與 `runbooks/extraction.md` 的建立同一天。

## 3. 為什麼沒有被發現

三道既有驗收都不檢查這一欄：

- 抽取收尾（`runbooks/extraction.md` §7）驗的是列數、欄數、BOM、CRLF、排序
- `section` 交叉驗證比對的是 `claim_id` 前綴與 `section` 欄
- schema v7 的同步檢查（CHANGELOG）確認的是 `TOPICS.md` 涵蓋率

沒有任何一道把 `claim_type` 的值域對回 SCHEMA §2。

〔推論〕成因與 STATUS §3.19 同型，但**方向相反**。§3.19 是 agent 自行造節號；
本項是 agent **照著一份規格文件填**，而那份文件本身與上位規格不一致。
runbook 是「每次抽取都會讀」的檔案，SCHEMA 是「每次修改 csv 都必須讀」的檔案，
兩者都被讀了，衝突卻沒被察覺——因為讀的時候是各自照著看，沒有把兩張表對起來。

〔推論〕真正的缺口是**規格之間沒有交叉驗證**。schema v7 的同步檢查
（SCHEMA §11 要求的那一項）當時確認了 `CLAUDE.md` 與 `TOPICS.md`，
但新建 `runbooks/extraction.md` 時沒有反向確認它引用的欄位值域是否與 SCHEMA 相符。

## 4. 已執行的部分（核准範圍內）

STATUS §5 選項 A 已核准的六列，`claim_type` 已改，並在各列 `note` 末尾註記變更：

| claim_id | 原 | 新 |
|---|---|---|
| `5.6.1-dkd-biomarker-uacr-correlation` | `fact` | `epidemiology` |
| `5.6.1-dkd-sdc1-100-4-6x` | `fact` | `epidemiology` |
| `5.6.1-dkd-sdf-sublingual-50pct` | `fact` | `measurement` |
| `5.6.2-dr-biomarker-severity-correlation` | `fact` | `epidemiology` |
| `5.6.2-dr-biomarker-vs-hba1c` | `fact` | `epidemiology` |
| `5.6.2-dr-octa-thickness-50-80pct` | `fact` | `measurement` |

這六列在兩個選項下都成立——它們確實是人體族群宣稱或量測值，
不論 `fact` 最後是否合法化，改成 `epidemiology`／`measurement` 都正確。

## 5. 待決：剩下 32 列

### 選項 1 — 把 `fact` 加入 SCHEMA §2 合法值（schema v8）

需要同時定義它的檢索取向與判定約束。建議定義：

> `fact` ｜ 不涉及因果的事實陳述（組成、時序、階段、狀態） ｜ 原始研究、綜述

並補明與相鄰型別的分界，否則會繼續互相吸收：

| 分界 | 判準 |
|---|---|
| `fact` vs `measurement` | 陳述的是「有這件事」→ `fact`；陳述的是某量測法測到的數值 → `measurement` |
| `fact` vs `epidemiology` | 主詞非人體族群 → `fact`；人體族群層級的關聯、預測值、風險比 → `epidemiology` |
| `fact` vs `definition` | 陳述某物「是什麼」→ `definition`；陳述某物「發生了什麼」→ `fact` |

- 成本：一次 schema 變更；32 列不動
- 風險：`fact` 缺少 `epidemiology`／`therapeutic` 的「必須人體證據」約束。
  32 列中若有人體族群宣稱被留在 `fact`，那道閘門仍然不會啟動——
  故本選項**必須**附帶逐列複核，不能只改 SCHEMA 就結案

### 選項 2 — 廢除 `fact`，32 列逐列改派

改派方向預估（未逐列判定，僅依 statement 粗分）：

```
→ measurement   數值與量測結果（厚度降幅、倍數、濃度、比例）   約 18 列
→ fact 無對應者 時序階段（「數週內出現 X」）                  約 8 列 → 待議
→ definition    組成與分類（核心蛋白清單、成分比例）           約 4 列
→ epidemiology  人體族群宣稱                                   約 2 列
```

- 成本：32 列逐列判定並改寫 `note`；不需 schema 變更
- 風險：「時序階段」那一組（DKD 三列、DR 四列等）在現行五個型別裡**沒有貼切的家**。
  硬塞進 `mechanism` 會與該欄「分流檢索策略」的用途衝突——
  時序宣稱要查的是臨床觀察研究，不是機制研究。
  這一組正是 `fact` 當初被寫進 runbook 的實際理由

### 選項 3 — 選項 1 加上逐列複核

`fact` 合法化（schema v8，含上表三條分界），同時把 32 列逐列對照新定義，
不符者改派。等於選項 1 與 2 各取一半。

- 成本：最高
- 但這是唯一同時解決「值域不合法」與「閘門失效」兩件事的做法

〔推論〕我傾向選項 3。選項 1 單獨執行會留下一個沒有人體證據約束的通用型別，
而它現在裝著 32 列裡數量最多的一群；選項 2 單獨執行會把時序宣稱擠進不對的型別，
那正是三個月後另一個人看不懂為何 `5.6.1-dkd-timeline-*` 是 `mechanism` 的地方。
不過這是判斷不是規則，請你決定。

## 6. 不論選哪一項都要做的兩件事

1. **修正 `runbooks/extraction.md` §6 的欄位表**，使其與 SCHEMA §2 一致。
   這是本次事件的直接成因，且它每次抽取都會被讀到——不修就會繼續產生同型列。
2. **在抽取收尾驗收（`runbooks/extraction.md` §7 步驟 2）加一項值域檢查**：

   ```powershell
   Import-Csv ledger/claims.csv |
     Where-Object { $_.claim_type -notin 'definition','mechanism','measurement','epidemiology','therapeutic' } |
     Select-Object claim_id, claim_type
   ```

   `status`、`tier`、`priority` 三欄同樣是列舉值域，同樣沒有任何檢查。
   建議一併納入，一次寫成一個驗收指令。

〔推論〕第 2 點比第 1 點重要。修 runbook 只擋住這一個值；
加值域檢查擋住的是**整類**「規格說 A、執行填了 B」的失效。
本專案已有兩次同型事件（節號自行造出、`claim_type` 用了未定義值），
兩次都是在下一批工作時才被發現，而兩次都可以被一行列舉檢查當場擋下。

## 7. 建議的執行順序

本提案**不執行**任何超出 §4 範圍的改動。決定之後：

1. 若選 1 或 3 → 遞增 `schema_version` 至 8，寫 `CHANGELOG.md`
2. 32 列的改派（選 2 或 3）另成一批，與 §2.6 補抽合併執行
3. `runbooks/extraction.md` 的兩項修正隨該批一起 commit
