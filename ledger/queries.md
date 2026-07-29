# queries.md — 實測檢索式與檢索通則

```
last_updated: 2026-07-29
來源: reports/backfill/ 各批次的「本批次有效的檢索詞」一節
用途: (1) 回填時的起手式  (2) 將來監測迴圈的檢索式來源
```

> **給 agent：** 本檔是**經驗紀錄**，不是規格。與 `ledger/SCHEMA.md` 或
> `runbooks/backfill.md` 衝突時，以那兩份為準。
>
> 每完成一批，把該批報告的「本批次有效的檢索詞」整理進 §2，
> 新浮現的通則寫進 §1。**不刪舊條目**——失效的檢索式標記為失效並註明日期，
> 它記錄的是「這樣查過、沒用」，本身就是資訊。

---

## 1. 檢索通則

依重要性排序。前四條是四個批次反覆驗證過的。

### 1.1 把「測量方法」當作必要檢索詞

同一血管、同一物種，雙光子與 μ-PIV 可以差 200 倍；傳統 TEM 與硝酸鑭 TEM 可以差 250 倍。
只搜「thickness + 血管名」會拿到互相矛盾的一堆數字而無法判定。

有效範例（2026-07-27、2026-07-28 兩批次）：

```
endothelial glycocalyx thickness 11 µm rapid freezing freeze substitution TEM
brain microvascular endothelial glycocalyx thickness measurement micrometers
```

〔推論〕第二式一次命中兩篇關鍵原始研究。有效原因是同時含
「thickness」「measurement」與單位詞——避開了大量只談功能不給數值的綜述。

### 1.2 敏感度／特異度必須連同「疾病 + 終點」一起查

只比對數字會誤判為 `verified`。

已知陷阱：endocan 宣稱 77%/70%，而重症肺炎預測 ARDS 的真實數據是 78.7%/70.3%
（cutoff 11.6 ng/mL）。數字接近但疾病與終點不同。

### 1.3 查極端值時，一併搜尋該值的評論或回應文章

`!PMID:21775768` 就是靠這個習慣找到的——它比原始論文更快說明了爭議所在。

檢索式加上 `comment` / `letter` / `response to` 或直接搜原文標題。

### 1.4 用精確數值加引號可有效**證否**

```
"85%" "specificity" "78%"
```

〔推論〕證否與證實需要不同的檢索策略。證實靠概念詞，證否靠精確字串。
三次不同措辭都查不到，才寫得下 `unsupported`。

### 1.5 交錯領域要分開查

敗血症糖萼標記文獻與創傷內皮病變文獻高度交錯，數值互相污染的風險高。
追查 cutoff 出處時，**同時檢索創傷（trauma）情境**是關鍵。

同型的交錯領域（尚未逐一驗證）：糖尿病腎病變 vs 一般 CKD、
COVID 內皮病變 vs 一般敗血症、老化 vs 神經退化。

### 1.6 引述不可代替原文

透過他人論文的引述取得數值，違反 `runbooks/backfill.md` 步驟 3。
引述可能已經失真——而失真正是本專案要偵測的對象。

〔推論〕2026-07-28 批次刻意留下 `PMID:29058634` 未判定，只記為線索，就是這條的實踐。

### 1.7 追歸屬錯誤時，把「數值」與「器官名」放進同一個查詢

證否一個數值（§1.4）與追出它**為什麼被安到這個器官上**是兩件事。
後者的檢索目標不是量測研究，而是那篇**同時提到該數值與該器官**的文章。

有效範例（2026-07-29 批次）：

```
"glycocalyx" brain microvessel thickness "11 μm" OR "11 microns" kidney glomerular
```

命中 `DOI:10.1371/journal.pone.0161610`——一篇研究對象為人腦內皮與人腎絲球內皮、
但在引言處引用了與器官無關的通則範圍（0.5-11 μm）的論文。它自己沒測過任何厚度。

〔推論〕這類論文是歸屬漂移的引擎：**研究對象**與**引言處的通則數值**在同一頁上，
轉述者把兩者接起來就成了「該器官可達該數值」。
凡遇到一個極端值被安在特定器官上而查無實測，先找這種論文，通常一次就中。

### 1.8 查機制歸屬時，查「切掉什麼、結果如何」，不要查機制名稱

查「電荷選擇性」「屏障功能」這類**機制名稱**只會拿到綜述。
要證否一個機制歸屬，檢索目標是**有酵素對照組的原始實驗**——
把酵素名（`heparinase`、`neuraminidase`、`hyaluronidase`、`chondroitinase`）放進查詢。

有效範例（2026-07-29 B1 批次）：

```
glomerular endothelial glycocalyx heparan sulfate charge selectivity albumin permeability heparinase in vivo
```

命中 `PMID:17942961`。該文同時做了神經胺酸酶（切唾液酸）與 heparinase III／human heparanase
（切 HS）兩組：前者使跨內皮電阻降 59%、白蛋白通量增 207%，後者對電阻無影響、
白蛋白通過僅增 40% 與 39%。

〔推論〕正是那組對照推翻了「電荷選擇性由 HS 承擔」。
若只查機制名稱，拿到的綜述會照著原 statement 的說法複述一遍，反而像是驗證成功。
**歸屬錯誤不只發生在器官上，也發生在分子上**（STATUS §3.6 的分子版）。

### 1.9 證否數字時，把數字連同情境一起精確檢索——它會告訴你數字真正屬於哪裡

§1.4 用精確數值加引號來**證否**。本條是它的延伸：
把數值與疾病情境一起放進查詢，往往不只證否，還會找出該數字真正對應的東西。

有效範例（2026-07-29 B2a 批次）：

```
sepsis endotoxin LPS glycocalyx degradation early time course "30 min" syndecan-1 plasma rise minutes after onset
```

這一式沒有找到「敗血症 30 分鐘內脫落」，卻找到「敗血症小鼠經脂質體奈米載體輸注後
糖萼於 30 分鐘內**恢復**」。疾病對、數字對、時間尺度對，只有過程方向相反。

〔推論〕這類錯誤數值查核抓不到——數字是真的，疾病是真的，文獻是真的。
唯一的破綻是過程方向，而方向不在數字裡。見 STATUS §3.13。

---

## 2. 分類檢索式

### 2.1 厚度與血管床

```
brain microvascular endothelial glycocalyx thickness measurement micrometers
endothelial glycocalyx thickness 11 µm rapid freezing freeze substitution TEM
glomerular endothelial glycocalyx thickness micrometers measurement electron microscopy
two-photon laser scanning microscopy glycocalyx thickness 4.5 μm mouse carotid artery
```

第三式一次命中三篇腎絲球實測（2026-07-29 批次）。有效原因與第一式同：
同時含 `thickness`、`measurement` 與單位詞，避開只談功能不給數值的綜述。

已建立的定錨值：

| 血管床 | 數值 | 樣本與方法 | 出處 |
|---|---|---|---|
| 腦微血管 | 301.0 ± 111.8 nm | 小鼠，硝酸鑭 TEM | `PMID:30504908` |
| 心微血管 | 135.5 ± 59.7 nm | 同上 | 同上 |
| 肺微血管 | 65.4 ± 28.4 nm | 同上 | 同上 |
| 腦皮質微血管（3 月齡） | 0.540 ± 0.086 μm | 小鼠，硝酸鑭 TEM | `DOI:10.1038/s41586-025-08589-9` |
| 腦皮質微血管（21 月齡） | 0.232 ± 0.092 μm | 同上 | 同上 |
| 腎絲球 | 50 nm 以下至 300 nm | 小鼠灌流固定，cThO2 TEM | `PMID:26608651` |
| 腎絲球內皮細胞 | 約 200 nm | 人類 ciGEnC，體外 | `PMID:17942961` |
| 腎臟大血管 | 約 250 nm | 小鼠，EM | `PMID:37184739` |
| 腎小管周圍微血管 | 最高 800 nm | 同上 | 同上 |
| 頸動脈（總／外） | 2.3 ± 0.1／2.5 ± 0.1 μm | 小鼠離體灌流，雙光子 | `PMID:21273784` |

〔推論〕這張表是**判定新厚度主張的基準**。凡新出現的血管床厚度主張，
先對照此表的量級。差一個數量級者，預設有問題。

〔推論〕表末的頸動脈列是刻意加入的**對照組**，不是微血管定錨值。
既有的兩個微米級極端值（11 μm、4-11 μm）在追溯後都指向大血管或培養細胞，
把大血管的真實量級放在同一張表上，下次遇到微米級主張時可以立刻看出
「這是大血管的量級被搬到微血管上」，而不必重跑一次追溯。

### 2.1.1 脫落與修復的時間尺度

已建立的定錨值：

| 情境 | 時間尺度 | 指標 | 出處 |
|---|---|---|---|
| 敗血症（小鼠 LPS 20 mg/kg） | 12 h 7.8 ng/ml；24 h 14.4 ng/ml；48 h 回基線 | 血漿 syndecan-1 | `PMID:29058634` |
| 缺血再灌流（人類，循環停止） | 再灌流 0-15 分鐘內升 42 倍 | 血漿 syndecan-1 | `PMID:17923576` |
| 缺氧／缺血再灌流（天竺鼠離體心臟） | 缺氧灌流一開始即釋出 | syndecan-1、HS | `PMID:21890663` |

〔推論〕**敗血症是小時級，缺血再灌流是分鐘級。**
凡看到「敗血症中數分鐘內脫落」，先假設是從缺血再灌流文獻搬過來的（§1.5 交錯領域）。

### 2.2 生物標記與效應量

```
syndecan-1 sepsis mortality meta-analysis odds ratio
```

〔推論〕效應量（OR / HR）容易查證，通常一次檢索即命中薈萃分析。
敏感度／特異度則相反——見 §1.2。

### 2.3 待建立

以下領域尚無實測檢索式，下次觸及時請補上本節：

- 機制類（`claim_type=mechanism`，佔帳本約半數）
- 治療類（`claim_type=therapeutic`）。注意 `CLAUDE.md` 第 6 條：不得為劑量建 claim，
  檢索目標是介入與結果的方向性關聯
- 權威性宣稱的證否。依 SCHEMA §6（v4），「國際共識」須改寫為可指名的形式後檢索，
  即查該學會立場聲明或指引是否存在

---

## 3. 檢索來源的優先序

依 `runbooks/backfill.md` 步驟 2：

1. Europe PMC（涵蓋最廣，含預印本與全文）
2. PubMed（MeSH 詞精確檢索）
3. bioRxiv / medRxiv（僅當前兩者無果，且主張標示為近年新發現）
4. ClinicalTrials.gov（僅 `claim_type=therapeutic`）

〔推論〕2026-07-28 批次的實際路徑是先一般網頁檢索命中期刊頁面、再直接取全文。
對「需要看 Methods 才能判定」的厚度類主張，這條路徑比只讀摘要有效——
Ando 2018 的三個血管床數值全在 Results 正文，摘要只有覆蓋率百分比。

---

## 4. 失效或無效的檢索式

**syndecan-1 半衰期**（2026-07-29 B1 批次，兩式均無果）：

```
syndecan-1 turnover half-life endothelial cell surface hours shedding kinetics
endothelial glycocalyx "half-life" syndecan-1 "hours" turnover "2" "8" heparan sulfate proteoglycan recycling
```

兩式都拿到 syndecan-1 的**脫落機制**與**臨床濃度時序**文獻，
但沒有任何一篇給出細胞表面 syndecan-1 的半衰期數值。
B2 批次處理 `2.2-sdc1-halflife-2-8h` 時不必重試這兩式，改換角度
（例：放射標記脈衝追蹤、`pulse-chase`、`metabolic labeling`）。

## 5. 檢索管道的環境限制

2026-07-29 記錄：在對話介面執行時，容器的網路允許清單不含
`eu-europepmc.org`、`ncbi.nlm.nih.gov` 等網域，無法直接呼叫 Europe PMC 或 PubMed API。
實際路徑為一般網頁檢索命中 PubMed 或期刊頁面後，自頁面的結構化欄位取識別碼
（PubMed URL 路徑中的 PMID、期刊頁的 `meta-citation_doi`）。

〔推論〕這條路徑仍符合 `SCHEMA.md` §4 的絕對規則——識別碼來自本次回傳的結構化欄位，
不是憑記憶寫出。但它比 API 檢索脆弱：查不到不等於不存在，可能只是搜尋引擎沒排上來。
判 `unsupported` 前的三次不同措辭（§1.4）在這個管道下更不能省。
