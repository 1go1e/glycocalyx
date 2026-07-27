# claims.csv — Schema 與寫入規則

```
schema_version: 2
last_updated: 2026-07-27
applies_to: ledger/claims.csv
```

> **給 agent：每次修改 `ledger/claims.csv` 之前，必須先完整讀過本檔案。**
> 本檔案為唯讀規格。agent 不得修改 SCHEMA.md 本身；需要變更 schema 時，寫一份提案到
> `reports/revisions/` 並停下來等待人工決定。

---

## 1. 檔案格式（硬性）

| 項目 | 規定 |
|---|---|
| 編碼 | UTF-8，**不得含 BOM** |
| 換行 | LF（`\n`），不得使用 CRLF |
| 引號 | RFC 4180；僅在必要時加引號（QUOTE_MINIMAL） |
| 排序 | **永遠依 `claim_id` 字典序升冪排列** |
| 標頭 | 第一列為標頭，欄位名與順序不得變更 |

**排序規則沒有例外。** 新增列之後必須重新排序再存檔。行序一亂，git diff 會退化成整檔重寫，
ledger 的版本歷史就失去意義——那是整套系統唯一不可替代的資產。

**禁止使用 Excel 開啟後存回。** 中文環境的 Excel 可能改寫編碼或加入 BOM。
編輯請用文字編輯器、VS Code，或以 Python `csv` 模組程式化處理。

---

## 2. 欄位定義

| # | 欄位 | 可否由 agent 寫入 | 說明 |
|---|---|---|---|
| 1 | `claim_id` | 僅新增時 | 永久識別碼。見 §3 |
| 2 | `section` | 否 | 對應 `source/glycocalyx_v3.md` 的章節編號 |
| 3 | `statement` | 條件性 | 單一、可證偽的敘述。見 §6 |
| 4 | `claim_type` | 僅新增時 | 見下方列表 |
| 5 | `tier` | 是 | 證據等級。見 §5 |
| 6 | `evidence` | 是 | 文獻識別碼。見 §4 |
| 7 | `status` | 是 | 驗證狀態。見 §5 |
| 8 | `priority` | 是 | `high` / `med` / `low`，回填順序 |
| 9 | `last_checked` | 是 | ISO 8601 `YYYY-MM-DD` |
| 10 | `note` | 是 | 自由文字。記錄疑點、衝突、限制條件 |

### `claim_type` 合法值

| 值 | 含義 | 檢索取向 |
|---|---|---|
| `definition` | 定義或分類 | 綜述、教科書 |
| `mechanism` | 生物機制、因果路徑 | 基礎研究、動物與細胞模型 |
| `measurement` | 量測方法、儀器規格、數值範圍 | 方法學文獻、儀器驗證研究 |
| `epidemiology` | 人體族群層級的關聯、預測值、風險比 | 臨床研究、世代研究、薈萃分析 |
| `therapeutic` | 治療介入的效果 | 臨床試驗、藥理研究 |

此欄的用途是**分流檢索策略**。`epidemiology` 與 `therapeutic` 的條目必須以人體研究為
主要證據；以動物或體外研究充當支持文獻，不得將 `status` 設為 `verified`。

---

## 3. `claim_id` 規則

格式：`{section}-{slug}`

```
正規表示式： ^[0-9](\.[0-9]+)*-[a-z0-9]+(-[a-z0-9]+)*$
範例：       5.1-sdc1-cutoff-40
```

三條不可違反的規則：

1. **永不重用。** 一個 `claim_id` 一旦出現在 repo 歷史中，即使該列後來被標為
   `superseded`，該 ID 也不得指向其他主張。
2. **永不改名。** 這是每條主張將來在網頁上的固定網址（`/claims/5.1-sdc1-cutoff-40`），
   也是外部引用的錨點。改名等於讓所有既有引用失效。
3. **章節重編號時不跟著改。** `section` 欄可以更新，`claim_id` 前綴保持原樣。
   ID 的前綴只是出身標記，不是位置指標。

---

## 4. `evidence` 格式

以分號分隔的 token 串，**分號前後不加空白**。

### Token 形式

| 形式 | 用於 | 範例 |
|---|---|---|
| `PMID:########` | PubMed 收錄文獻（優先使用） | `PMID:31234567` |
| `DOI:10.xxxx/yyyy` | 無 PMID 者，如 bioRxiv/medRxiv 預印本 | `DOI:10.1101/2025.03.14.643210` |
| `NCT:NCT########` | 臨床試驗登錄 | `NCT:NCT04512345` |

### 反駁文獻

在 token 前加驚嘆號 `!` 表示該文獻**反駁**此主張：

```
PMID:31234567;!PMID:33445566;PMID:35566778
```

### 排序

**第一個 token 為主要引用（primary reference）**，其後不拘。
主要引用應為證據等級最高、且最直接支持該 statement 數值的那一篇。
將來網頁只會顯示第一個。

### 絕對規則

> **PMID、DOI、NCT 編號只能來自檢索工具本次實際回傳的結構化欄位。**
> 任何情況下都不得憑記憶、推測或模式仿造寫出識別碼。
>
> 若無法透過工具取得識別碼，`evidence` 留空、`status` 設為 `unsupported`，
> 並在 `note` 說明檢索過的來源與關鍵字。

這是整個 ledger 唯一無法自動偵測的失效模式：一個格式正確但不存在的 PMID，
在檔案裡看起來與真的完全一樣。

---

## 5. `status` 與 `tier`

### `tier` 合法值

| 值 | 含義 |
|---|---|
| `in_vitro` | 細胞或無細胞系統 |
| `animal` | 動物模型 |
| `human_obs` | 人體觀察性研究（橫斷、世代、病例對照） |
| `rct` | 隨機對照試驗 |
| `meta` | 系統性回顧或薈萃分析 |
| `review` | 敘述性綜述 |

`tier` 反映 `evidence` 中**未加 `!` 前綴**的 token 裡，證據等級最高的那一篇。
**反駁文獻（`!` 前綴）不計入 `tier`。**

理由：`tier` 回答的是「支持這條主張的最強證據是什麼等級」。若把反駁文獻計入，
一條主張被越強的證據推翻，`tier` 看起來越高——方向正好相反。

`status=contested` 的列，其 `note` **必須**寫明反駁文獻的等級與反駁的具體內容
（是數值不符、條件不同，還是結論相反），因為這項資訊在欄位層已不可見。

> **`review` 不足以單獨支持 `verified`。**
> 若唯一的支持文獻是敘述性綜述，必須循其引用追到原始研究。
> 追不到就維持 `unverified`，並在 `note` 記錄該綜述的 PMID 供後續追蹤。

### `status` 合法值

| 值 | 條件 |
|---|---|
| `unverified` | 尚未檢索。初始值 |
| `verified` | 至少一篇原始研究直接支持，且 statement 中所有數值與原文一致 |
| `partial` | 找到相關文獻，但條件不同（物種、族群、血管床、劑量、測定方法），statement 需修訂 |
| `contested` | 同時存在支持與反駁文獻 |
| `superseded` | 已被更新或更強的證據取代。`note` 必須指出取代它的 `claim_id` |
| `unsupported` | 已主動檢索但找不到支持文獻。`note` 必須記錄檢索範圍 |

**`unverified` 與 `unsupported` 的差別是「還沒查」與「查了沒有」。**
這個區分是月報中最重要的訊號——`unsupported` 的累積代表原始文件有內容站不住腳。

### 狀態轉移

- `unverified` 可轉為任何其他狀態。
- **任何狀態都不得轉回 `unverified`。** 檢索過就是檢索過了。
- 轉為 `verified` 之後若出現反駁文獻，改為 `contested`，不要改回 `unverified`。

---

## 6. `statement` 的修改規則

分兩個階段，界線是該列是否曾經達到 `verified`：

**尚未 verified 時**：可自由修訂 statement，讓它更精確、更可證偽。
還沒有人引用它，改動成本為零。

**已經 verified 之後**：
- 允許：修正錯字、補上限定條件而不改變語意。
- **不允許：改變語意。** 需要改變語意時，建立一條新的 `claim_id`，
  並將舊列的 `status` 設為 `superseded`，在其 `note` 寫明 `superseded_by: {新 claim_id}`。

理由與 §3 相同：`verified` 的條目可能已被外部引用，語意漂移會讓引用者引到不同的東西。

### statement 撰寫要求

- 一列一個主張。若一句話包含兩個可獨立證偽的宣稱，拆成兩列。
- 數值必須與原文一致，**不得換算、四捨五入或推估**。
  原文寫 `approximately 85%` 而 statement 寫 `85%`，須在 `note` 註明原文為近似值。
- 不得使用「研究顯示」「一般認為」「有證據支持」等無主詞措辭。
  文獻的身分由 `evidence` 欄承載，不寫進 statement。

---

## 7. Append-only

- **不得刪除任何列。** 錯誤的主張改 `status`，不刪除。
- **不得刪除既有的 `evidence` token。** 發現引錯了，加 `!` 前綴或改 `status`，
  並在 `note` 說明。刪掉就看不出曾經引錯過。
- 新增列一律接在檔尾，存檔前重新依 `claim_id` 排序。

---

## 8. `last_checked`

每次針對某列執行檢索後更新為當日日期，**即使什麼都沒找到**。

這一欄的用途是回答「哪些條目太久沒複查」。
若某列的 `last_checked` 從未更新，代表 agent 從未真正處理過它——
`status` 停在 `unverified` 但 `last_checked` 為空，與 `status=unsupported`
是完全不同的狀態。

---

## 9. Commit 規範

一次檢索執行 = 一個 commit。

```
ledger: verify 6 claims in section 5.1

verified: 5.1-sdc1-cutoff-40, 5.1-sdc1-or-204
partial:  5.1-sdc1-sens-spec (原文族群為 ICU 成人，statement 未限定)
unsupported: 5.1-sepsis-phase3-abnormal-repair

run: 2026-08-03 / inbox/2026-08-03.json
```

不要把多次執行的結果累積成一個 commit。歷史的顆粒度就是將來 diff 的顆粒度。

---

## 10. 範例列

驗證前：

```csv
5.1-sdc1-cutoff-40,5.1,血漿 Syndecan-1 濃度大於 40 ng/mL 是預測敗血症死亡率的指標。,epidemiology,,,unverified,high,,最高優先。具體 cutoff，可能被誤用為臨床閾值
```

驗證後（假想）：

```csv
5.1-sdc1-cutoff-40,5.1,血漿 Syndecan-1 濃度大於 40 ng/mL 是預測敗血症死亡率的指標。,epidemiology,meta,PMID:36112233;PMID:34556677,partial,high,2026-08-03,原文 cutoff 為 40.4 ng/mL 且族群限於 ICU 成人敗血性休克；statement 需補上族群限定
```

注意這個例子：找到了文獻，但 statement 過度概括，所以是 `partial` 而非 `verified`。
**這是回填階段最常見的結果，不是失敗。** 逐條把過度概括的敘述收斂到文獻實際支持的範圍，
正是這份 ledger 的主要價值。

---

## 11. 變更本 Schema

新增欄位一律往後接，不得改動既有欄位的名稱、順序或語意。
任何變更必須遞增 `schema_version`，並在 `ledger/CHANGELOG.md` 記錄變更內容與日期。

`schema_version` 一旦遞增，前端解析程式與 agent 的提示詞都需同步檢查。
