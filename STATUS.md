# 專案現況

```
最後更新: 2026-07-28
最後 commit: ba4d691
schema_version: 4
```

> **新 session 從這裡開始。** 讀完本檔，再依 §6 決定要不要讀其他檔案。

---

## 1. 帳本狀態

```
unverified   85
partial       5
unsupported   4
verified      2
contested     1
──────────────
總計         97
```

隨時可用此指令重新確認：

```powershell
Import-Csv ledger/claims.csv | Group-Object status | Select-Object Name, Count
```

```powershell
Import-Csv ledger/claims.csv | Where-Object priority -eq 'high' |
  Group-Object { $_.status -in 'verified','partial','unsupported','contested' } |
  Select-Object Name, Count
```

**依優先序的完成率**（這才是專案健康度指標；全體完成率會被大量 low/med 稀釋）：

```
high   已判定 11 / 50  = 22%
med    已判定  1 / 40  =  2%
low    已判定  0 /  7  =  0%
```

有 `evidence` 的列共 7 條，去重識別碼 11 個（9 個 PMID、2 個 DOI）。
97 列的 `source_ref` 均為 `glycocalyx/v3#{section}`。

`claim_type` 分布：mechanism 58、epidemiology 15、measurement 13、therapeutic 7、definition 4。

---

## 2. 已完成批次

| 日期 | 章節 | 條數 | 報告 |
|---|---|---|---|
| 2026-07-26 | 5.1（校準批次） | 3 | `reports/backfill/2026-07-26-5.1.md` |
| 2026-07-26 | 5.5 | 4 | `reports/backfill/2026-07-26-5.5.md` |
| 2026-07-27 | 2.1.1 / 2.2 厚度群 | 5 | `reports/backfill/2026-07-27-2.1.1-2.2.md` |

**`2026-07-26-5.1.md` 是判定標準的基準範本。** 任何新加入的執行者（人或 agent）
應先讀它，再開始工作——它示範了判定的細膩度、note 的寫法，
以及「找到答案但取不到識別碼時該怎麼做」。

---

## 3. 已確認的系統性問題

這些是跨批次浮現的模式，不是個別錯誤。處理後續章節時應預期它們重複出現。

### 3.1 跨疾病數值拼接

`source/glycocalyx/v3.md` 第 5.1 節將三個數字並列呈現，讀來像同一項研究的完整表現，
實際上分別來自：2025 年敗血症薈萃分析（OR 2.04，真）、創傷內皮病變文獻（40 ng/mL，
真但錯置）、來源不明（85%/78%）。

**通則**：凡「cutoff + 敏感度/特異度 + 效應量」三者並列的主張，預設不同源，分開查證。

### 3.2 敏感度／特異度整欄查無出處

第 5.5 節生物標記表的四組敏感度／特異度（syndecan-1、endocan、HS 片段、低分子量 HA）
全部查無出處，而同一張表的效應量（OR 2.04、OR 5.06）多半可驗證。

**通則**：看到「敏感度 X% / 特異度 Y%」而未同時給出 cutoff 與終點定義的主張，
預設有問題。真實文獻報告 ROC 結果時幾乎必然同時給出切點。

**已知陷阱**：endocan 宣稱 77%/70%，而重症肺炎預測 ARDS 的真實數據是 78.7%/70.3%
（cutoff 11.6 ng/mL）。只比對數字會誤判為 verified，必須連同疾病與終點一起核對。

### 3.3 逆向指標被寫反

2.1.1 節宣稱「臨床與活體共識厚度 2-4 μm」，查無出處。臨床實測厚度約 0.5 μm，
而 GlycoCheck 的 PBR 值恰好落在 1.8-3.0 μm——PBR 是紅血球穿透深度，**升高代表糖萼變薄**，
方向與厚度相反，且原文 2.1.2 節自己寫對了。

**通則**：凡「數值越高代表功能越差」的逆向指標（PBR、阻力指數等），
在轉述中被當成正向量的機率很高。文件中出現這類指標時，回頭核對方向。

### 3.4 「共識」宣稱是無出處的徵兆

`2.1.1-consensus-2-4um` 與 5.5 節的敏感度/特異度同型：宣稱有共識、但無任何文獻可追。
真的有共識時，文獻會引用具體綜述或指引。

**2026-07-28 擴充**：這個模式不只出現在數值上。`source/glycocalyx/heparanase.md` 全文：

```
共識                        12 次
國際共識                     7 次
指引 / guideline             0 次
ADA / EASD / KDIGO / ESC / ISPAD   0 次
```

宣稱國際共識七次，未指名任何學會或立場聲明。

**通則**：權威性宣稱（「國際共識」「新典範」「臨床標準」）與數值宣稱同樣可證偽，
但更難察覺——數值錯了看得出來，「國際共識」錯了看不出來，而它承載的說服力更強。
處理方式見 SCHEMA §6：把宣稱本身建成 claim，改寫為可指名的形式，查不到即 `unsupported`。

**附帶**：時間形容詞（「2025 年 11 月最新」）進了 statement 之後，
會隨時間自動變成偽陳述，而沒有任何機制會提醒你去改。一律移除，時間入 `note`。

### 3.5 自我引用式「更新」

新增於 2026-07-28（來源：`reports/revisions/2026-07-28-intake-physiology-md.md`）。

`糖萼層_Glycocalyx_生理.md` 的「表2：糖萼層生物標記臨床更新（基於2025年薈萃分析）」
與同一份文件前面的「表1」**六列逐字相同**，內文卻寫「endocan 新增 77%/70%；ecSOD 活性 68%/82%」
——這兩組數字在表1裡本來就有。所謂更新沒有更新任何數值。

**通則**：標題宣稱「更新版」「基於最新薈萃分析」的表格，先與被它取代的表格逐格比對。
比 §3.4 更隱蔽，因為標題本身就在提供權威性。

### 3.6 歸屬與數值在轉述間漂移

新增於 2026-07-28（同上）。

同一個 11 μm，`v3` 歸給「腎絲球微血管」、生理.md 一處歸給「腦部與腎臟」、另一處寫「腎絲球 4-11 μm」，
而實查出處（`PMID:21474821`）的樣本是體外培養的牛主動脈內皮細胞。
同理，syndecan-1 預測死亡的敏感度在生理.md 正文寫 90%、同檔表格寫 85%。

**通則**：同一主張在不同文件（或同一文件不同位置）出現時，先比對數值與歸屬是否一致。
**漂移本身就是「無原始出處」的指紋**——有出處的數字不會在轉述中換血管床或換百分比。

---

### 3.7 LLM 產出文件是多次問答的串接

新增於 2026-07-28（來源：`reports/revisions/2026-07-28-intake-heparanase-mito.md`）。

`glycocalyx_Heparanase.md` 有 7 個各自從「一、」開始的章節樹，
`The_Mitochondria-Glycocalyx_Axis.md` 有 3 個。表面看是一份文件，實際是多次獨立問答的串接。

**通則**：收錄任何 LLM 產出的長文件前，先數「一、」或 `##` 層級的重複次數。
節號碰撞會讓 `claim_id` 與 `source_ref` 無法唯一定位，必須先重編。
另檢查 Google Docs 匯出造成的跳脫 markdown（`\#\#\#`、`\-`），否則章節解析不出來。

---

### 3.8 搬移檔案會留下過期路徑引用

新增於 2026-07-28。

`source/glycocalyx_v3.md` 改為 `source/glycocalyx/v3.md` 之後，全庫殘留 20 處舊路徑引用。

**通則**：搬移或改名 `source/`、`ledger/`、`runbooks/` 下的任何檔案後，
立即全庫掃描並分三類處理：

```powershell
Get-ChildItem -Recurse -Include *.md,*.csv | Select-String "舊路徑"
```

| 類別 | 處置 |
|---|---|
| 活文件（`CLAUDE.md`、`STATUS.md`、`runbooks/`、`ledger/SCHEMA.md`） | **必改**，agent 會照著執行 |
| 待核准提案（`reports/revisions/` 未核准者） | **必改**，核准後會照著執行 |
| 歷史紀錄（`reports/backfill/`、`ledger/CHANGELOG.md`、已核准提案） | **不改**，改了就失真 |

〔推論〕第三類是關鍵。批次報告記錄的是當時的檔案狀態，
把它改成今天的路徑等於竄改紀錄，日後無法還原判定當下看到的是哪份文件。

---

## 4. 下一批

**`2.2` 剩餘條目**（厚度群已完成，其餘 8 條同章節，可共用脈絡）

```
2.2-brain-1-3um          ← 與已 verified 的 2.2-cap-0.2-0.5um 文件內部不一致，優先
2.2-lung-thin-sdc1
2.2-glomerular-hs-charge
2.2-liver-porous-physiological
2.2-pathological-50-90pct
2.2-repair-7-14d
2.2-sdc1-halflife-2-8h
2.2-sepsis-shedding-30min
```

**本批附加動作**（待 §5 第 1 項核准後生效）：前四條在 `糖萼層_Glycocalyx_生理.md`
有對應敘述，且數值與 `v3` 未必一致。查證時一併核對兩份文件的說法，
差異寫入 note，並依 §3.6 檢查歸屬是否漂移。

低成本可併入：`2.1.1-em-underestimate`（priority=low，但 2026-07-27 批次已取得直接證據
`PMID:21474821`：同一批培養細胞傳統 TEM 0.040 μm vs RF/FS-TEM 11 μm）。

**收錄批次（新增，與回填分離）**

`source/glycocalyx/heparanase.md`（42 節）與 `source/mitochondria/axis.md`（5 節）
已歸位但尚未抽 claim。抽取時：

- 只建列、填 `source_ref`、`status=unverified`，**不做查證**
- `heparanase#5.6`（治療階梯）與 `#5.7`（監測建議）依 `CLAUDE.md` 第 6 條
  不得為劑量建 claim；藥物與結果的方向性關聯照常建列
- 依 SCHEMA §6（v4），statement 不得含時間形容詞或未具名的權威性宣稱。
  `heparanase.md` 的各節結論（`#2.6`、`#3.6`、`#4.6`、`#6.6`、`#6.8`、`#7.4`）
  密集出現「2025 國際共識」，抽取時逐條改寫為可指名的形式
- `mitochondria/axis#1.1`–`#1.3` 與 `glycocalyx/v3` §1.2 主題重疊，
  依 SCHEMA §2 認定規則先掃描既有列，命中者追加 `source_ref` 而非建新列
- 預估新增 140–195 條。抽完後 §1 的 high 完成率會下降，**這是預期的**，
  不得為了讓數字好看而略過任何一節

抽取完成後，回填順序仍依本節既有優先序，新文件不插隊。

**之後的優先序**（不依 claim_id 順序）：

1. `5.3-pbr-precedes-cognitive-2-3y` — 可能與已判 unsupported 的 `5.5-biomarker-ha-lmw` 同源
2. `5.5-pbr-sdc1-correlation` — 與上一條同為 PBR 相關，可併批
3. 七條 `claim_type=therapeutic` — 治療宣稱風險最高，尤其 `4.2.3-hrt-rct-2025`（宣稱為 RCT）。
   生理.md 另有至少 5 條 therapeutic 淨新增（AAV/C1GALT1、empagliflozin、Endocalyx、
   heparanase 抑制劑、S1P），核准收錄後併入此批
4. 標示「2025 年研究」的新宣稱群：果糖 ER 壓力、TMAO、微塑膠、S1P、ecSOD
5. 其餘 `mechanism` 類——查證相對機械，適合批次自動跑

---

## 5. 待決事項

### 已定調的政策

**查無出處的數值一律標註，不從原文刪除**（人工裁示 2026-07-27）。
標註格式規範見 `reports/revisions/2026-07-27-source-2.1.1-thickness.md` §2，
三種標記 `〔查無出處〕`／`〔條件限定〕`／`〔有爭議〕` 對應 ledger 的 status。
此政策**推翻**了 5.5 批次報告中「建議直接刪除敏感度／特異度欄位」的建議。

**不得為具體劑量、給藥途徑或用藥時程建立 claim**（人工裁示 2026-07-28）。
劑量寫入 `note` 供追溯，不進 `statement`、不取得 `status`。
`claim_type=therapeutic` 的 statement 只描述方向性關聯。
完整條文見 `CLAUDE.md`「不可違反」第 6 條。

理由：`verified` 是文獻學意義的「有出處」，不是臨床意義的「已驗證可用」。
這個語意落差在劑量上會造成實際傷害。網頁上線前，`therapeutic` 的讀者層呈現須另行設計。

**單一 repo、單一帳本，主題以 `source/` 子目錄區分**（人工裁示 2026-07-28）。
不為糖萼、線粒體等主題各自開 repo。理由：分 repo 不減少查證工作量，只縮小分母；
且會把「一條主張一列」原則搬到跨 repo 層級，失去 `source_ref` 的去重機制。
糖萼研究的高價值內容多半長在主題**介面**上（糖萼—線粒體、糖萼—凝血、糖萼—免疫），
依主題切分，切線正好穿過最有價值的地方。

網頁仍分站，由 build 腳本依 `source_ref` 的子目錄前綴從單一 `claims.csv` 產生。
介面主題的 claim 在多站出現，但指向同一 `claim_id`、同一份判定。

**判定綁 claim，不綁文件**（人工裁示 2026-07-28，schema_version 3）。
同一主張在多份文件出現時只建一列，所有出現位置寫進 `source_ref`。

**但兩種情況不得合併，須各自建列**：數值相同而**歸屬不同**、
或同一指標**數值不同**。理由見 §3.6——漂移本身就是證據，
合併會把它壓進 `note` 而失去可見度。完整規則見 `ledger/SCHEMA.md` §2「一條主張，一列」。

### 待核准 — 原文修訂

- **2.1.1 節四處標註的修訂單**：`reports/revisions/2026-07-27-source-2.1.1-thickness.md`
  含 §2 標註格式規範（一併待核准）與 §3 的四處具體改動。核准後才可改 `source/`。
- **5.5 節敏感度／特異度的修訂單：尚未寫。** 依上述政策應改為標註而非刪除，
  格式套用 2.1.1 修訂單的 §2。

### 待核准 — 新文件收錄（2026-07-28 新增，擋住多文件擴充）

- **`糖萼層_Glycocalyx_生理.md` 如何收錄。**
  評估結果：該檔是「文章本體＋對該文章的檢視意見」的複合物，且檢視段有兩項自我矛盾
  （表2 與表1 逐字相同卻宣稱是更新、正文 90% 與表格 85% 不一致）。
  建議不收為 source，只抽取淨新增主張。
  提案：`reports/revisions/2026-07-28-intake-physiology-md.md`。
- **`Heparanase.md` 與 `Mitochondria-Axis.md` 的 claim 抽取尚未執行。**
  收錄本身已完成（正規化、章節重編、歸位），但**尚未建任何 claim 列**。
  `claims.csv` 目前仍是 97 列，全部來自 `glycocalyx/v3`。抽取作業見 §4「收錄批次」。
- **`糖萼層_Glycocalyx_生理.md` 如何收錄。**
  評估結果：該檔是「文章本體＋對該文章的檢視意見」的複合物，且檢視段有兩項自我矛盾
  （表2 與表1 逐字相同卻宣稱是更新、正文 90% 與表格 85% 不一致）。
  建議不收為 source，只抽取淨新增主張。
  提案：`reports/revisions/2026-07-28-intake-physiology-md.md`。
### 待辦（承接前批）

- 取得 `DOI:10.1161/JAHA.124.040179` 全文，核對 thrombomodulin 的 HR 2.10
- 取得 Daniyarova et al. 2025 的 PMID，補入 `5.1-sdc1-or-204` 的 evidence
  （目前僅有 `DOI:10.1002/mbo3.70155`）

### 已決

保留紀錄，防止下一個 session 重做已經決定過的事。**不要刪除本節條目。**

- ~~SCHEMA §5 的 `tier` 是否排除 `!` 前綴 token~~ → **已決，採選項 A**。
  schema_version 已升至 2，見 `ledger/CHANGELOG.md` 與
  `reports/revisions/2026-07-27-schema-tier-refuting-tokens.md`（核准 2026-07-27，commit `d197ffe`）。
  連帶新增規定：`status=contested` 的列，`note` 必須寫明反駁文獻的等級與反駁內容。
- ~~是否刪除 2.1.1 節的「共識 2-4 μm」敘述~~ → **已決，改為標註不刪除**。見上方「已定調的政策」。
- ~~是否刪除 5.5 節表格的敏感度／特異度欄位~~ → **已決，刪除方案作廢**，同上。
- ~~SCHEMA §6 是否禁止時間形容詞~~ → **已決，採用並擴充**，schema_version 升至 4。
  除時間形容詞外，一併禁止未具名的權威性宣稱（「國際共識」等）。
  見 `ledger/CHANGELOG.md` 與 STATUS §3.4。
- ~~是否為糖萼／線粒體分開建 repo~~ → **已決，不分**。見上方「已定調的政策」。
- ~~`Heparanase.md` 是否收為 source~~ → **已決，收**。正規化＋章節重編後存為
  `source/glycocalyx/heparanase.md`（7 段串接重編為 1.x–7.x，共 42 節）。
  原檔存 `source/intake/2026-07-28-heparanase-raw.md`。
- ~~`Mitochondria-Axis.md` 是否收為 source~~ → **已決，只收第 1 段機制論述**，
  存為 `source/mitochondria/axis.md`（1.1–1.5）。MitoQ、斷食生酮、LDL 判讀、
  居家自測、個人執行協議**不收**，全檔存 `source/intake/2026-07-28-mitochondria-axis-raw.md`。
- ~~是否為治療劑量建立 claim~~ → **已決，不得建立**。見上方「已定調的政策」與
  `CLAUDE.md` 第 6 條，提案 `reports/revisions/2026-07-28-intake-heparanase-mito.md` §2.4。
- ~~`claims.csv` 是否新增 `source_ref` 欄~~ → **已決，採選項 A 含 §5 認定規則**。
  schema_version 已升至 3，見 `ledger/CHANGELOG.md` 與
  `reports/revisions/2026-07-28-schema-source-doc.md`（核准 2026-07-28）。
  97 列已機械化回填 `source_ref`，其餘欄位未動。
  2026-07-28 `source/` 改為主題子目錄後，值為 `glycocalyx/v3#{section}`。

---

## 6. 新 session 的開場

若在 Claude Code 中執行，`CLAUDE.md` 會自動載入，接著讀本檔即可。

若在對話介面中執行，上傳以下檔案：

```
STATUS.md
ledger/claims.csv
ledger/SCHEMA.md
ledger/CHANGELOG.md
runbooks/backfill.md
reports/backfill/2026-07-26-5.1.md
source/glycocalyx/v3.md
```

若本次任務涉及 schema 或原文修訂，另加 `reports/revisions/` 中相關的提案檔。

開場指示：

> 這是內皮糖萼文獻追蹤專案。先讀 `STATUS.md`，再讀 `SCHEMA.md` 與 `backfill.md`。
> `2026-07-26-5.1.md` 是判定標準的基準範本，請照那個細膩度執行。
>
> 特別注意兩點：查無出處時判 `unsupported` 是合格結果，不是失敗；
> PMID/DOI 只能來自本次實際檢索回傳的結構化欄位，不得憑記憶寫出。
>
> 本批處理 STATUS.md §4 所列的條目。完成後停下來等我檢視。

---

## 7. 維護

每完成一批，更新本檔的 §1 帳本狀態、§2 已完成批次、§4 下一批。
若該批浮現新的系統性模式，加入 §3。
提案核准或作廢時，把該項從 §5「待核准」移到 §5「已決」並加上刪除線，**不要刪除條目**——
「已決」的用途是防止下一個 session 重做已經決定過的事。
若該決定形成通則（適用於未來所有批次，而不只是這一次），另外寫進 §5「已定調的政策」。

檔頭的「最後 commit」須填實際 HEAD 的短 hash。本檔與批次報告一同 commit。
