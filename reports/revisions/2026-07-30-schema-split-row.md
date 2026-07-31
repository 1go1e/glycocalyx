# 修訂提案：混合宣稱列的拆列程序

```
提案日期: 2026-07-30
適用檔案: ledger/SCHEMA.md §6（主要）、§5、§7
提出批次: reports/backfill/2026-07-29-2.2-b1.md（第一例）
          reports/backfill/2026-07-30-therapeutic.md（第二例，判定前即被擋下）
schema_version: 5 → 6（已核准）
狀態: 已核准並執行 2026-07-31（含拆 5.4）
```

> 本提案與同日的 `2026-07-30-schema-nonaligned-evidence.md` 皆為 schema 變更。
> 若兩案同時核准，建議合併為單一 `schema_version: 6`、單一 CHANGELOG 條目、單一 commit。

---

## 1. 問題

SCHEMA §6 明文要求「一列一個主張。若一句話包含兩個可獨立證偽的宣稱，拆成兩列。」
但這條規則只約束**新建列**。既有列被發現是混合宣稱時，**沒有任何程序可以拆**：

- §3：`claim_id` 永不重用、永不改名
- §7：不得刪列
- §5：`superseded` 的定義是「已被更新或更強的證據取代」，語意不涵蓋「被拆解」

三條規則相乘的結果是：混合宣稱列**進得來、出不去**。

### 已累積兩例，且第二例的成本更高

**第一例 `2.2-glomerular-hs-charge`（2026-07-29，判定後才發現）**

```
statement: 腎絲球微血管糖萼極厚且富含 HS，藉負電荷排斥白蛋白以維持電荷選擇性屏障。
status:    contested        tier: animal
evidence:  PMID:17942961;PMID:29058634;!PMID:26608651;!DOI:10.1152/ajprenal.00126.2019
```

三個子宣稱的判定結果各不相同：「極厚」被反駁、「富含 HS」未見反駁、
「電荷選擇性由 HS 承擔」被同一篇支持文獻的對照組推翻。三者共用一個 `tier=animal`，
而那個 animal token 只支持結構覆蓋、不涉及電荷機制。讀表的人無從得知
`tier` 指的是哪一個子宣稱（STATUS §3.11）。

**第二例 `5.4-s1p-preconditioning`（2026-07-30，判定前就卡住）**

```
statement: S1P 預處理可減輕缺血再灌注造成的糖萼脫落並保護移植器官。
status:    unverified（NEEDS-REVIEW）    last_checked: 2026-07-30
```

兩個子宣稱判定結果相反：子宣稱一被 `PMID:32017300` 直接推翻
（S1P 預處理縮小梗塞面積，但對 syndecan-1 無影響，作者明言保護非由糖萼介導），
子宣稱二查無原始研究。**無法寫出一個不誤導的 `status`**，
因此該列檢索完成卻停在 `unverified`，`last_checked` 已填。

〔推論〕兩例的差別就是本提案的急迫性所在。第一例的成本是「事後補記」——
判定寫進去了，只是 `tier` 誤導；第二例的成本是**當場停工**——
一條 high 優先序的主張查完了卻無法入帳。程序缺口已經從記帳品質問題
變成產能問題，而混合宣稱列不會只有這兩條：三檔抽 claim 預估新增 140–195 條，
其中必然有同型。

---

## 2. 選項

### 選項 A：沿用 `superseded`，擴充其定義（建議）

原列 `status=superseded`，`note` 寫 `split_into: {id1};{id2};{id3}`。

- 優點：不新增 `status` 合法值，既有解析與月報統計不需改
- 優點：`superseded` 的實質語意本來就是「這一列不再是活的，去看 note 指的那些列」，拆列符合這個語意
- 缺點：`superseded` 混入兩種成因（被更強證據取代／被拆解），統計時無法只靠 `status` 區分。
  緩解方式是 `note` 用固定前綴 `split_into:` 與既有的 `superseded_by:` 區分，可機器判別

### 選項 B：新增 `status` 合法值 `split`

- 優點：語意最乾淨，統計時一眼可分
- 缺點：`status` 是 §5 的封閉列舉，也是月報與將來前端讀取的主要欄位。
  新增合法值等於要求所有消費端同步更新，§11 明文要求檢查
- 缺點：為兩條列新增一個 status 值，成本高於收益

### 選項 C：保留原列作為「最窄的存活子宣稱」，其餘另建新列

- 缺點：需要改寫原列的 statement 使其變窄。若原列已達 `verified`，§6 明文禁止改變語意
- 缺點：由哪一個子宣稱「繼承」原 `claim_id` 是任意的，等於在永久識別碼上做了一個沒有依據的選擇
- **不建議**

**建議採 A。**

---

## 3. 若核准，具體改動

### 3.1 `ledger/SCHEMA.md` §6「statement 撰寫要求」之後，新增小節

```markdown
### 既有列發現為混合宣稱時的拆列程序

「一列一個主張」約束的是新建列。既有列被發現含兩個以上可獨立證偽的宣稱時，
依下列程序拆解。**拆列不是刪列**，原列永久保留。

**步驟**

1. **原列**：`status` 改為 `superseded`，`note` 追加一行
   `split_into: {新 claim_id 1};{新 claim_id 2}[;...]`。
   原列的 `statement`、`evidence`、`tier`、`last_checked` **一律不動**——
   它們記錄的是拆列前的真實歷史狀態。
2. **新列**：每個子宣稱一列。`claim_id` 由原 id 衍生並加上區辨後綴
   （例：`2.2-glomerular-hs-charge` → `2.2-glomerular-thickness-extreme`、
   `2.2-glomerular-hs-rich`、`2.2-glomerular-charge-albumin`）。
   `section`、`claim_type`、`priority`、`source_ref` 由原列繼承。
   `note` 開頭必須寫 `split_from: {原 claim_id}`。
3. **證據的承接**：逐 token 判斷該 token 支持或反駁的是**哪一個**子宣稱。
   - 能明確歸屬者，連同其前綴寫入對應新列的 `evidence`，該新列的
     `status`、`tier`、`last_checked` 一併承接。
   - **無法明確歸屬者不承接**，該識別碼寫入對應新列的 `note` 作為線索，
     新列維持 `status=unverified`、`last_checked` 留空。
   - 一個 token 可同時歸屬多列（同一篇文獻可以同時支持兩個子宣稱）。

**核准層級**

- 原列 `status` 從未達到 `verified` 者：拆列屬 agent 可執行動作，
  但必須在批次報告中列出拆列前後對照表。
- 原列曾達 `verified` 者：**須人工核准**。理由與 §6 禁止改變已 verified 列語意相同——
  該 `claim_id` 可能已被外部引用。

**回填時發現混合宣稱列的處置順序**

先拆再判，不要先判再拆。若判定已完成才發現（如 `2.2-glomerular-hs-charge`），
判定照常寫入，拆列另案處理，原列的判定紀錄即為拆列前的歷史狀態。
```

### 3.2 `ledger/SCHEMA.md` §5 `status` 合法值表

`superseded` 一列的條件欄，現行：

```
已被更新或更強的證據取代。`note` 必須指出取代它的 `claim_id`
```

改為：

```
已被更新或更強的證據取代，**或已依 §6 拆列**。
前者 `note` 寫 `superseded_by: {claim_id}`，後者寫 `split_into: {id};{id}[;...]`。
兩者必須使用這兩個固定前綴，以便機器區分成因。
```

### 3.3 `ledger/SCHEMA.md` §7 Append-only

在該節加一句，明確拆列不違反 append-only：

```markdown
拆列（§6）不違反本節：原列保留、`claim_id` 不重用、識別碼不移除，
新增的是列與 `note` 內容。原列的 `status` 由 `superseded` 覆寫是 §5 允許的狀態轉移。
```

### 3.4 `ledger/SCHEMA.md` 檔頭與 `ledger/CHANGELOG.md`

```
schema_version: 5  →  schema_version: 6
last_updated: 2026-07-28  →  last_updated: (核准日期)
```

```markdown
## schema_version 6 — (核准日期)

- **§6**：新增既有列的拆列程序（原列 superseded + split_into、新列 split_from、
  證據逐 token 歸屬、核准層級依原列是否曾達 verified）。
- **§5**：`superseded` 的條件擴充為涵蓋拆列，並規定 `superseded_by:` 與
  `split_into:` 兩個固定前綴。
- **§7**：明訂拆列不違反 append-only。
- 提案：`reports/revisions/2026-07-30-schema-split-row.md`
- 受影響列：2 列待拆，見提案 §4
```

---

## 4. 若核准，兩條待拆列的執行草案

### 4.1 `5.4-s1p-preconditioning`（建議優先，它正卡著）

原列 → `status=superseded`，`note` 追加 `split_into: 5.4-s1p-iri-shedding;5.4-s1p-transplant`。

| 新 claim_id | statement | 承接的 evidence | status | tier |
|---|---|---|---|---|
| `5.4-s1p-iri-shedding` | S1P 預處理可減輕缺血再灌注造成的糖萼脫落。 | `PMID:24285115;!PMID:32017300` | `contested` | `in_vitro` |
| `5.4-s1p-transplant` | S1P 預處理可保護移植器官。 | （空，`PMID:33671524` 入 note 作線索） | `unsupported` | （空） |

兩列的 `claim_type=therapeutic`、`priority=high`、
`source_ref=glycocalyx/v3#5.4;intake/2026-07-28-糖萼層生理#L116` 均由原列繼承。

`5.4-s1p-iri-shedding` 判 `contested` 而 `tier=in_vitro`：支持側是體外且非 IRI 情境
（`PMID:24285115`），反駁側是 ex vivo 大鼠心臟的直接 IRI 實驗（`PMID:32017300`）。
依 §5（v2）反駁 token 不計入 `tier`，且 `note` 必須寫明反駁文獻的等級與內容——
本例正是 v2 當初設計的情境：**反駁比支持更有力，但 `tier` 不得因此升高**。

〔推論〕拆完之後這條主張的真實面貌才顯現出來：支持它的是一個不相干系統裡的體外實驗，
反駁它的是唯一一個真的做了 IRI 的實驗。混在一列時，這個對比被
「還有另一半宣稱查無資料」稀釋掉了。

### 4.2 `2.2-glomerular-hs-charge`

原列 → `status=superseded`，`note` 追加
`split_into: 2.2-glomerular-thickness-extreme;2.2-glomerular-hs-rich;2.2-glomerular-charge-albumin`。

拆法沿用 `reports/backfill/2026-07-29-2.2-b1.md` 的草案。
四個 token 的歸屬需在執行時逐一判定，本提案不預先指派——
該批次報告已記錄各 token 支持或反駁的是哪一個子宣稱，執行時據以填入。

〔注意〕`PMID:29058634` 的取得過程在另一提案
（`2026-07-30-schema-nonaligned-evidence.md` §4 末）有疑義，
建議先裁示該問題再執行本列的拆解，以免把來源存疑的 token 複製到新列。

---

## 5. 本提案不涵蓋

**`source_ref` 誤配 token 的移除程序**（STATUS §5「待決 — `source_ref` 機械回填的誤配」）。

該問題與本案同屬「SCHEMA 只有 append-only、沒有移除／重組程序」的缺口，
STATUS 也建議一併寫進 §2。但兩者的判斷基準不同：拆列處理的是**一列裝了太多主張**，
`source_ref` 誤配處理的是**一列掛了錯誤的出處**，後者還牽涉是否要對 53 條雙出處列
做一次以數值為鍵的全庫校驗。合併會讓核准變成包裹表決。

建議另開提案。若你希望合併，我可以把兩案併成一份。

---

## 6. 核准欄

- [x] 採選項 A（`superseded` + `split_into`）
- [ ] 採選項 B（新增 `status=split`）
- [ ] 採選項 C（保留原列為最窄子宣稱）
- [ ] 其他：

核准人／日期：Chen, 31/07/2026

執行範圍（可分開勾選）：

- [ ] 僅寫入 SCHEMA 程序，兩條待拆列另批執行
- [x] 同時執行 `5.4-s1p-preconditioning` 的拆解
- [ ] 同時執行 `2.2-glomerular-hs-charge` 的拆解（保留，待 `PMID:29058634` 來源狀態裁示後執行）

---

## 7. 執行紀錄（2026-07-31）

### 拆列前後對照（SCHEMA §6 要求）

| | claim_id | status | tier | evidence | last_checked |
|---|---|---|---|---|---|
| 前 | `5.4-s1p-preconditioning` | unverified | （空） | （空） | 2026-07-30 |
| 後 | 同上 | **superseded** | （空，保留） | （空，保留） | 2026-07-30（保留） |
| 新 | `5.4-s1p-iri-shedding` | contested | in_vitro | `PMID:24285115;!PMID:32017300` | 2026-07-31 |
| 新 | `5.4-s1p-transplant` | unsupported | （空） | （空） | 2026-07-31 |

原列的 `statement`、`evidence`、`tier`、`last_checked` 一律未動，僅 `status` 改為
`superseded`、`note` 追加 `split_into:`。

### 執行時發現的一件事

§4.1 草案原本要把 `5.4-s1p-transplant` 直接判 `unsupported`，但該子宣稱在
2026-07-30 的 therapeutic 批次中**只跑過一式檢索**（移植那半句是附帶查到的）。
`queries.md` §1.4 要求三式才能判 `unsupported`——**原列的檢索紀錄不自動及於子宣稱**。

2026-07-31 補跑兩式後才寫入判定。三式均查無以移植結果為終點的 S1P 預處理原始研究，
僅得兩篇綜述（`PMID:33671524`、`DOI:10.3390/ijms22084019`），均明言證據多屬實驗性。

〔推論〕這一點已回寫進 SCHEMA §6 的拆列程序作為明文注意事項。
它是拆列特有的失效模式：原列的 `last_checked` 看起來像是整列都查過了，
但混合宣稱列的檢索深度在各子宣稱之間本來就可能不均。

### 補跑檢索的副產物

查到一篇方向相反的移植研究：大鼠異位心臟移植，冷保存損傷由 S1P 誘發，
S1PR3 拮抗劑反而減輕冷損傷。介入為受體拮抗而非給予 S1P、損傷型態為冷保存而非
缺血再灌注、無糖萼終點，屬「相關但不對位」。本次未取得識別碼，
故未依 §4 寫入 `~` token，記入 `5.4-s1p-transplant` 的 `note` 待補。

核准後執行順序：改 SCHEMA.md §6/§5/§7/檔頭 → 寫 CHANGELOG → 一個 commit
（建議訊息：`schema: v6 — 新增混合宣稱列的拆列程序`）；
拆列執行另成一個 `ledger:` commit。
