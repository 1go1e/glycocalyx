# 回填作業程序（Backfill Runbook）

```
版本: 1
適用: 將 ledger/claims.csv 中 status=unverified 的條目回填證據
前置: 必須先完整讀過 ledger/SCHEMA.md
```

---

## 1. 目標

把 97 條 `unverified` 主張，逐條檢索文獻後轉為
`verified` / `partial` / `contested` / `unsupported` 之一。

**這不是「找到支持文獻」的任務，是「確定每條主張的證據狀態」的任務。**
`unsupported` 是完全合格的產出，不是失敗。回填結束時若 `verified` 比例過高，
應懷疑判定標準過寬。

---

## 2. 批次規則

- **一批 5-8 條，且限定同一 `section`。**
  每條需要 2-4 次檢索，一批約 20-30 次工具呼叫，這是單一 session 內能維持判定品質的上限。
- 同章節可共用背景脈絡，減少重複檢索。
- 依 `priority` 降冪處理：先做完所有 `high`，再做 `med`，`low` 最後。

**建議起手批次**：`5.1-sdc1-cutoff-40`、`5.1-sdc1-sens-spec`、`5.1-sdc1-or-204`。
這三條互相關聯（很可能出自同一篇），且是全表風險最高的數值。先做這批，
可以同時驗證整套流程與判定標準。

---

## 3. 單條處理流程

### 步驟 1 — 拆解 statement

列出這條主張中所有**可獨立查證的要素**：

- 實體（分子、疾病、族群、血管床、物種）
- 關係（升高／降低／預測／導致／相關）
- **所有數值**，逐一列出

範例（`5.1-sdc1-cutoff-40`）：
```
實體: Syndecan-1, 血漿, 敗血症, 死亡率
關係: 預測
數值: 40 ng/mL
未指定: 族群（成人？ICU？敗血性休克 vs 敗血症）、檢測平台
```

「未指定」那一項要特別記下來——它通常就是最後判成 `partial` 的原因。


**改寫檢查（依 SCHEMA §6）。** 拆解時若 statement 含以下任一項，先改寫再檢索：

- 時間形容詞（「最新」「近年」「截至 2025 年 11 月」）→ 移除，時間入 `note`
- 未具名的權威性宣稱（「國際共識」「新典範」「臨床標準」）→ 改寫為可指名的形式，
  例如「〔某學會〕〔某年〕立場聲明將 X 列為 Y」。檢索時就查這份聲明是否存在；
  查不到即 `unsupported`，不因原文語氣強烈而放寬

### 步驟 2 — 檢索

依序嘗試，找到足夠證據即停：

1. **Europe PMC**（涵蓋最廣，含預印本與全文）
2. **PubMed**（MeSH 詞精確檢索）
3. **bioRxiv / medRxiv**（僅當前兩者無果，且主張標示為近年新發現）
4. **ClinicalTrials.gov**（僅 `claim_type=therapeutic`）

檢索式從步驟 1 的實體組合而成。若首次檢索超過 200 筆，加上限定條件；
若為 0 筆，逐步放寬並記錄放寬的過程。

### 步驟 3 — 核對數值（不可省略）

對步驟 1 列出的**每一個數值**，在候選文獻的摘要或全文中找到對應處，**逐字比對**。

- 原文 `40.4 ng/mL`，statement 寫 `40 ng/mL` → **不一致**，記為 `partial`
- 原文 `approximately 85%`，statement 寫 `85%` → 可接受，但 `note` 須註明原文為近似值
- 原文 `OR 2.04 (95% CI 1.51-2.76)`，statement 寫 `OR 2.04` → 一致
- 找不到該數值的出處 → 不得推估或換算，該數值視為未獲支持

### 步驟 4 — 判定

```
找到直接支持的原始研究？
├─ 否 → 找到的只有敘述性綜述？
│        ├─ 是 → 循其引用追原始研究。追到則回到上一層；
│        │        追不到 → status=unverified，note 記錄該綜述 PMID
│        └─ 否 → status=unsupported，note 記錄檢索範圍與關鍵字
└─ 是 → 同時找到反駁文獻？
         ├─ 是 → status=contested
         └─ 否 → 所有數值逐字相符，且研究條件與 statement 範圍一致？
                  ├─ 是 → status=verified
                  └─ 否 → status=partial，note 寫明差異在哪
```

額外約束（來自 SCHEMA §2）：
`claim_type` 為 `epidemiology` 或 `therapeutic` 的條目，
若唯一證據來自體外或動物研究，**不得判為 `verified`**，一律 `partial`。

### 步驟 5 — 寫入

更新該列的 `tier`、`evidence`、`status`、`last_checked`、`note`。
`last_checked` 一律更新為當日，**即使結果是 `unsupported`**。

若本次查證過程中發現該主張也出現在其他 `source/` 文件，在 `source_ref` 追加該位置
（格式見 SCHEMA §2）。追加前先確認符合 SCHEMA §2「認定規則」的「同一條」標準——
歸屬不同或數值不同時**不得追加**，應另建一列並在兩列的 `note` 互相標示。

`note` 的撰寫要求：寫下**下一個人需要知道的事**，不是過程流水帳。

- 好：`原文 cutoff 40.4 ng/mL，族群限 ICU 成人敗血性休克；statement 需補族群限定`
- 差：`搜尋了 Europe PMC 找到幾篇相關文獻`

---

## 4. 必須停下來問人的情況

遇到以下任一情況，**不要自行判定**。在該列 `note` 標記 `NEEDS-REVIEW`，
`status` 維持不變，並寫進本批次報告，然後繼續處理下一條：

1. 找到的文獻**推翻**了原 statement 的方向（不只是數值不同，而是結論相反）
2. 同一數值在兩篇以上文獻中出現但**互不引用**，無法判斷是否同源
3. statement 涉及的是**尚未被明確命名或研究的概念**，找不到對應詞彙
4. 需要修改 statement 的語意才能與文獻相符，而該列已是 `verified`
5. 檢索結果顯示 `source/glycocalyx/v3.md` 有**內部矛盾**（見 claims.csv 中已標記的厚度與 GAG 佔比問題）

---

## 5. 批次產出

每批次結束時：

1. 更新 `ledger/claims.csv`（依 `claim_id` 重新排序後存檔）。
   新增列必須填 `source_ref`；建列前先以數值為鍵掃描既有列，
   命中且符合 SCHEMA §2「同一條」標準時，改為在既有列追加 `source_ref`，不建新列
2. 寫入 `inbox/YYYY-MM-DD-{section}.json`：保留所有檢索的原始回傳，不做刪減
3. 寫入 `reports/backfill/YYYY-MM-DD-{section}.md`：

```markdown
# 回填批次 {section} — YYYY-MM-DD

處理 N 條： verified X / partial Y / contested Z / unsupported W / needs-review V

## 判定摘要
- {claim_id}: {status} — {一句話理由}

## NEEDS-REVIEW
- {claim_id}: {為何無法判定}

## 對原文件的修訂建議
- {section}: {建議如何修改，附 claim_id}

## 本批次有效的檢索詞
- {實際命中的關鍵字組合}
```

最後一節請確實填寫。這些實測關鍵字將來會構成 `ledger/queries.md`
的監測檢索式——它們是這個階段的副產品，但價值不低於主產出。

4. 依 SCHEMA §9 格式提交一個 commit

---

## 6. 絕對禁止

- 憑記憶或推測寫出 PMID / DOI / NCT 編號。識別碼只能來自檢索工具本次回傳的結構化欄位
- 修改 `source/` 下的任何檔案。所有修訂建議一律寫入 `reports/`
- 刪除 `claims.csv` 中的任何列
- 為了讓數字好看而放寬判定標準
