# 專案現況

```
最後更新: 2026-07-27
最後 commit: 5cd5e6c
```

> **新 session 從這裡開始。** 讀完本檔，再依 §6 決定要不要讀其他檔案。

---

## 1. 帳本狀態

```
unverified   85
partial       5
unsupported   4
contested     1
verified      2
──────────────
總計         97
```

隨時可用此指令重新確認：

```powershell
Import-Csv ledger/claims.csv | Group-Object status | Select-Object Name, Count
```

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

`source/glycocalyx_v3.md` 第 5.1 節將三個數字並列呈現，讀來像同一項研究的完整表現，
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

### 3.4 「共識值」是無出處的徵兆

`2.1.1-consensus-2-4um` 與 5.5 節的敏感度/特異度同型：宣稱有共識、但無任何文獻可追。
真的有共識時，文獻會引用具體綜述或指引。

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

低成本可併入：`2.1.1-em-underestimate`（priority=low，但 2026-07-27 批次已取得直接證據
`PMID:21474821`：同一批培養細胞傳統 TEM 0.040 μm vs RF/FS-TEM 11 μm）。

**之後的優先序**（不依 claim_id 順序）：

1. `5.3-pbr-precedes-cognitive-2-3y` — 可能與已判 unsupported 的 `5.5-biomarker-ha-lmw` 同源
2. `5.5-pbr-sdc1-correlation` — 與上一條同為 PBR 相關，可併批
3. 七條 `claim_type=therapeutic` — 治療宣稱風險最高，尤其 `4.2.3-hrt-rct-2025`（宣稱為 RCT）
4. 標示「2025 年研究」的新宣稱群：果糖 ER 壓力、TMAO、微塑膠、S1P、ecSOD
5. 其餘 `mechanism` 類——查證相對機械，適合批次自動跑

## 5. 待決事項

- **是否刪除 2.1.1 節的「共識 2-4 μm」敘述，並改寫 11 μm 的脈絡。**
  查無出處，且 11 μm 的實際來源是體外培養牛主動脈內皮細胞，非原文所述之腎絲球活體測量。
  建議改寫見 `reports/backfill/2026-07-27-2.1.1-2.2.md`。需人工決定。
- **SCHEMA §5 的 `tier` 是否應排除 `!` 前綴 token。**
  `2.1.1-extreme-11um` 的支持文獻為 `in_vitro`、反駁文獻為 `animal`，
  照字面取「最高等級」會反過來抬高該主張的可信度。本次暫記 `in_vitro`。
  依 SCHEMA §11 須寫提案至 `reports/revisions/`，尚未寫。
- **是否刪除 5.5 節表格的敏感度／特異度欄位。** 批次報告建議直接刪除而非標註待查，
  理由是這些數字外觀等同臨床診斷閾值，而讀者不會讀 note。需人工決定。
  修改 `source/` 前應先在 `reports/revisions/` 寫修訂單。
- 取得 `DOI:10.1161/JAHA.124.040179` 全文，核對 thrombomodulin 的 HR 2.10
- 取得 Daniyarova et al. 2025 的 PMID，補入 `5.1-sdc1-or-204` 的 evidence

---

## 6. 新 session 的開場

若在 Claude Code 中執行，`CLAUDE.md` 會自動載入，接著讀本檔即可。

若在對話介面中執行，上傳以下檔案：

```
ledger/claims.csv
ledger/SCHEMA.md
runbooks/backfill.md
reports/backfill/2026-07-26-5.1.md
source/glycocalyx_v3.md
STATUS.md
```

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

本檔與批次報告一同 commit。
