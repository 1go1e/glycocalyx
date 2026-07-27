# 修訂提案：`tier` 遇到反駁文獻時的取值規則

```
提案日期: 2026-07-27
適用檔案: ledger/SCHEMA.md §5
提出批次: reports/backfill/2026-07-27-2.1.1-2.2.md
schema_version: 1 → 2（若核准）
狀態: 待人工核准
```

---

## 1. 問題

SCHEMA §5 現行規定：

> `tier` 反映 `evidence` 中**證據等級最高**的那一篇。

同節又規定 `evidence` 中可用 `!` 前綴標示**反駁**該主張的文獻。兩條規則相乘，會在 `contested`
的列上產生反向結果——`tier` 會被反駁文獻抬高。

**實際踩到的列**（目前 ledger 中唯一含 `!` token 者）：

```
2.1.1-extreme-11um
evidence: PMID:21474821;!PMID:18258858;!PMID:21775768
```

| token | 文獻 | 等級 | 立場 |
|---|---|---|---|
| `PMID:21474821` | Ebong 2011，體外培養牛主動脈內皮細胞 RF/FS-TEM，11 μm | `in_vitro` | 支持 |
| `!PMID:18258858` | Potter & Damiano 2008，小鼠活體 μ-PIV，同細胞型別 0.02 μm | `animal` | 反駁 |
| `!PMID:21775768` | ATVB 同期評論，質疑為技術造成 | `review` | 反駁 |

照字面取最高等級會標成 `animal`。但這條主張並沒有任何動物活體等級的**支持**證據——
唯一的支持文獻是體外培養細胞。`tier=animal` 會讓讀表的人（以及將來的前端與月報統計）
誤以為此主張有活體證據背書。

〔推論〕這個失效模式的方向值得注意：**反駁越有力，`tier` 標得越高。**
一條主張被越強的證據推翻，欄位上看起來越可信。這與 `tier` 欄的用途正好相反。

2026-07-27 批次寫入時暫記 `in_vitro`（主要引用的等級），並在該列 `note` 標明此衝突。
本提案請求把這個處置正式化。

---

## 2. 選項

### 選項 A：`tier` 只反映非 `!` 前綴的 token（建議）

```
tier 反映 evidence 中「未加 ! 前綴」的 token 裡證據等級最高的那一篇。
反駁文獻不計入 tier。
```

- 優點：語意清楚——`tier` 回答「支持這條主張的最強證據是什麼等級」，與 §5「`review` 不足以單獨支持 `verified`」的既有精神一致（該條也是只看支持證據）
- 優點：不動欄位結構，`schema_version` 遞增即可，前端解析不需改
- 缺點：反駁文獻的等級資訊在欄位層消失，只能靠 `note`。查「這條是被什麼等級的證據推翻的」需要人工讀 note

### 選項 B：新增 `refuting_tier` 欄

依 §11 往後接一欄，專記反駁文獻的最高等級。

- 優點：資訊完整，可機器查詢「被 `rct` 等級推翻的主張有幾條」
- 缺點：97 列中目前只有 1 列有 `!` token，96 列會是空欄。為單一案例增欄，成本高於收益
- 缺點：前端解析程式與 agent 提示詞都要同步改（§11 明文要求）

### 選項 C：維持現狀，靠 `note` 說明

- 優點：不動 SCHEMA
- 缺點：不解決問題。`tier` 欄仍會給出誤導值，而月報與前端讀的是欄位不是 note

**建議採 A。** 若日後 `contested` 的列累積到 10 條以上，再評估是否升級為 B——
屆時 `refuting_tier` 可依 §11 往後接，不影響既有欄位。

---

## 3. 若核准，具體改動

### 3.1 `ledger/SCHEMA.md` §5

現行：

```markdown
`tier` 反映 `evidence` 中**證據等級最高**的那一篇。
```

改為：

```markdown
`tier` 反映 `evidence` 中**未加 `!` 前綴**的 token 裡，證據等級最高的那一篇。
**反駁文獻（`!` 前綴）不計入 `tier`。**

理由：`tier` 回答的是「支持這條主張的最強證據是什麼等級」。若把反駁文獻計入，
一條主張被越強的證據推翻，`tier` 看起來越高——方向正好相反。

`status=contested` 的列，其 `note` **必須**寫明反駁文獻的等級與反駁的具體內容
（是數值不符、條件不同，還是結論相反），因為這項資訊在欄位層已不可見。
```

### 3.2 `ledger/SCHEMA.md` 檔頭

```
schema_version: 1  →  schema_version: 2
last_updated: 2026-07-26  →  last_updated: (核准日期)
```

### 3.3 `ledger/CHANGELOG.md`（新建，§11 要求）

```markdown
# claims.csv Schema 變更紀錄

## schema_version 2 — (核准日期)

- §5：`tier` 改為只反映未加 `!` 前綴的 token；反駁文獻不計入。
- §5：新增規定，`status=contested` 的列，`note` 必須寫明反駁文獻的等級與反駁內容。
- 提案：`reports/revisions/2026-07-27-schema-tier-refuting-tokens.md`
- 受影響列：`2.1.1-extreme-11um`（現行寫入的 `tier=in_vitro` 已符合新規則，無需回填）

## schema_version 1 — 2026-07-26

- 初版。
```

---

## 4. 連帶影響

- **既有資料不需回填。** 唯一含 `!` token 的列已寫成 `in_vitro`，與新規則一致
- **§11 要求**：`schema_version` 遞增後，前端解析程式與 agent 提示詞需同步檢查。
  目前無前端；agent 端只需重讀 SCHEMA，無硬編碼邏輯
- `CLAUDE.md` 不需改動，其中未提及 `tier` 規則

---

## 5. 核准欄

- [x] 採選項 A
- [ ] 採選項 B
- [ ] 採選項 C（維持現狀）
- [ ] 其他：

核准人／日期：Chen,27/07/2026

核准後執行順序：改 SCHEMA.md → 建 CHANGELOG.md → 一個 commit
（建議訊息：`schema: v2 — tier 排除反駁文獻`）。
