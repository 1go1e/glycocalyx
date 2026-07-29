# queries.md — 實測檢索式與檢索通則

```
last_updated: 2026-07-28
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

---

## 2. 分類檢索式

### 2.1 厚度與血管床

```
brain microvascular endothelial glycocalyx thickness measurement micrometers
endothelial glycocalyx thickness 11 µm rapid freezing freeze substitution TEM
```

已建立的定錨值（皆為小鼠、硝酸鑭 TEM，見 `2026-07-28-2.2-brain-lung.md`）：

| 血管床 | 數值 | 出處 |
|---|---|---|
| 腦微血管 | 301.0 ± 111.8 nm | `PMID:30504908` |
| 心微血管 | 135.5 ± 59.7 nm | 同上 |
| 肺微血管 | 65.4 ± 28.4 nm | 同上 |
| 腦皮質微血管（3 月齡） | 0.540 ± 0.086 μm | `DOI:10.1038/s41586-025-08589-9` |
| 腦皮質微血管（21 月齡） | 0.232 ± 0.092 μm | 同上 |

〔推論〕這張表是**判定新厚度主張的基準**。凡新出現的血管床厚度主張，
先對照此表的量級。差一個數量級者，預設有問題。

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

（目前無條目。查過但無果的檢索式應記在此，避免下一批重複嘗試。）
