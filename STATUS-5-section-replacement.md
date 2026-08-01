<!-- 用這一段整段取代 STATUS.md 中
     「### 待決 — `claim_type` 與 SCHEMA §2 定義的落差（2026-08-01 新增）」
     整節（到「### 待處理 — 第二批（§2.6）的兩處補抽」之前為止） -->

### 待決 — `claim_type=fact` 不在 SCHEMA 合法值之內（2026-08-01 新增，取代下方原項）

執行下方選項 A 時全帳本掃描發現：**`fact` 從 v1 到 v7 從未列入 SCHEMA §2 的合法值**
（`definition`／`mechanism`／`measurement`／`epidemiology`／`therapeutic` 五個）。
它只出現在 `runbooks/extraction.md` §6 的欄位表，該檔 2026-07-31 隨 schema v7 新建，
四批抽取全部照它填。改完六列後仍有 **32 列**使用這個未定義的值
（`axis` 6、`heparanase` 26；原始 121 列無一使用）。

三道既有驗收都不檢查值域：抽取收尾驗列數／欄數／BOM／CRLF／排序，
`section` 交叉驗證比對的是 `claim_id` 前綴，schema v7 同步檢查看的是 `TOPICS.md` 涵蓋率。

**待決**：（1）把 `fact` 加入 SCHEMA §2 合法值，遞增 v8；
（2）廢除 `fact`，32 列逐列改派；（3）兩者兼做——合法化並附三條分界，同時逐列複核。
三個選項的成本、風險與改派方向預估見
`reports/revisions/2026-08-01-claim-type-fact.md` §5。

〔注意〕不論選哪一項，`runbooks/extraction.md` §6 的欄位表都要修，
且抽取收尾應加入 `claim_type`／`status`／`tier`／`priority` 四欄的值域檢查——
本專案已有兩次「規格說 A、執行填了 B」的事件，兩次都可被一行列舉檢查當場擋下。
指令見該提案 §6。

〔注意〕**32 列的改派不得夾帶進 §4 抽取批次。** 與 §2.6 補抽合併另成一批。

---

### ~~待決~~ 已決 — `claim_type` 與 SCHEMA §2 定義的落差（2026-08-01 新增，同日核准選項 A）

`5.6.x` 兩節共有六列的 `claim_type` 寫成 `fact`，依 SCHEMA §2 的定義應為
`epidemiology` 或 `measurement`：

| claim_id | 現用 | 依定義應為 |
|---|---|---|
| `5.6.1-dkd-sdc1-100-4-6x` | `fact` | `epidemiology` |
| `5.6.1-dkd-biomarker-uacr-correlation` | `fact` | `epidemiology` |
| `5.6.1-dkd-sdf-sublingual-50pct` | `fact` | `measurement` |
| `5.6.2-dr-biomarker-severity-correlation` | `fact` | `epidemiology` |
| `5.6.2-dr-biomarker-vs-hba1c` | `fact` | `epidemiology` |
| `5.6.2-dr-octa-thickness-50-80pct` | `fact` | `measurement` |

這不是分類美觀問題。SCHEMA §2 規定 `epidemiology` 與 `therapeutic` 的條目
必須以人體研究為主要證據，不得以動物或體外研究判 `verified`。
**一條人體族群宣稱若型別寫成 `fact`，這道閘門就不會啟動。**

`claim_type` 依 SCHEMA §2 為「僅新增時」可寫入，故第三批未逕行修改，
以維持 `5.6.1` 與 `5.6.2` 平行列的可比性（要改就六列一起改）。

~~**待決**：（A）六列一併改為正確型別；（B）維持現狀另加檢查規則。~~

**已核准選項 A 並執行（2026-08-01）。** 六列已改，各列 `note` 末尾註記變更日期與依據。
六列在上方新待決的三個選項下都成立——它們確實是人體族群宣稱或量測值。

依核准時的附帶要求「全帳本掃一次同型的列」執行掃描，結果即上方新增的待決項：
問題不是六列分類不當，是 `fact` 這個值本身不合法。**掃描的價值大於原本要修的東西。**

---


<!-- 以下這一項插入 §5「已決」開頭，接在
     「保留紀錄，防止下一個 session 重做已經決定過的事。**不要刪除本節條目。**」之後 -->

- **每批開工前先跑 `git status --short`，工作區不乾淨就先查清楚再動手**
  （核准 2026-08-01）。〔待寫入 §7，與下一批的 STATUS 更新一同 commit〕
  觸發案例：2026-08-01 套 §3 patch 時，`git apply --check` 意外掀出
  `reports/2026-07-29-scan-rhetoric.md` 在工作區被刪除（本批未碰該檔，
  推測為 OneDrive 同步所致），已由 `git checkout --` 還原。
  若當時直接開始抽 §4，該刪除會混進 §4 的 commit，看起來像是那批刪的。
  這比 §3.8「搬移檔案留下過期路徑引用」更前面一層：**檔案在 git 之外被動到，
  只有主動查才看得見**——repo 存放於 OneDrive 同步目錄，此風險長期存在。

