# claims.csv — Schema 與寫入規則

```
schema_version: 10
last_updated: 2026-08-07
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
| 2 | `section` | 否 | 主題座標。取自 `ledger/TOPICS.md` 的正規主題樹，**不必等於** `source_ref` 的節號 |
| 3 | `statement` | 條件性 | 單一、可證偽的敘述。見 §6 |
| 4 | `claim_type` | 僅新增時 | 見下方列表 |
| 5 | `tier` | 是 | 證據等級。見 §5 |
| 6 | `evidence` | 是 | 文獻識別碼。見 §4 |
| 7 | `status` | 是 | 驗證狀態。見 §5 |
| 8 | `priority` | 是 | `high` / `med` / `low`，回填順序 |
| 9 | `last_checked` | 是 | ISO 8601 `YYYY-MM-DD` |
| 10 | `note` | 是 | 自由文字。記錄疑點、衝突、限制條件 |
| 11 | `source_ref` | 是 | 主張在原始文件中的出現位置。多值。見下方 |

### `claim_type` 合法值

| 值 | 含義 | 檢索取向 |
|---|---|---|
| `definition` | 定義或分類 | 綜述、教科書 |
| `fact` | 不涉及因果的事實陳述：狀態、時序、階段、非族群層級的數值 | 原始研究、綜述 |
| `mechanism` | 生物機制、因果路徑 | 基礎研究、動物與細胞模型 |
| `measurement` | 量測方法、儀器規格、數值範圍 | 方法學文獻、儀器驗證研究 |
| `epidemiology` | 人體族群層級的關聯、預測值、風險比 | 臨床研究、世代研究、薈萃分析 |
| `therapeutic` | 治療介入的效果 | 臨床試驗、藥理研究 |

**此欄為封閉值域。上表以外的值一律不合法**，抽取與回填收尾均須檢查（見 §9）。

此欄的用途是**分流檢索策略**。`epidemiology` 與 `therapeutic` 的條目必須以人體研究為
主要證據；以動物或體外研究充當支持文獻，不得將 `status` 設為 `verified`。

### `claim_type` 的判定順序（v9）

判定 `claim_type` 時**先過人體閘門，再問型別**。順序不可對調。

**第一順位 — 人體閘門**

問一個問題：**這條 statement 的真假，能不能用動物或體外研究判定？**

| 情況 | 判定 |
|---|---|
| 不能——statement 的主詞是人體族群或人體患者，其真假取決於人體資料 | `epidemiology` |
| 不能，且該條陳述的是治療介入的效果 | `therapeutic` |
| 能 | 進入第二順位 |

閘門的適用範圍是 `definition`／`fact`／`measurement` 三型。
`mechanism` **不受此閘門約束**：因果路徑本就以動物與細胞模型為主要證據，
把描述人體場景的機制宣稱改判 `epidemiology` 會使檢索取向錯置。
若一條 `mechanism` 列的 statement 其實不含因果連接、只是人體觀察，
那它從一開始就不該是 `mechanism`——那是下方的型別分界問題，不是閘門問題。

**第二順位 — 型別分界**

`definition`／`fact`／`mechanism`／`measurement` 四者依下方分界表判定。

> **閘門先於型別，是因為型別分界會把人體宣稱吸走。**
> v8 只在 `fact` vs `epidemiology` 之間立了這道界線，
> 但 `measurement` 的分界寫的是「數值出自 statement 內具名的量測平台」——
> **「具名平台」與「人體患者」可以同時成立**，
> 一條「某疾病患者以某儀器測到某值」的宣稱會依字面落入 `measurement`，
> 而 `measurement` 不帶人體證據約束，閘門於是靜默失效。
> 這與 v8 修補的是同一個缺口，只是在另一條路上。

〔注意〕**「具名量測平台 + 人體患者」一律走 `epidemiology`**，平台名稱留在 statement 內，
不因改型別而移除。反之，若主詞是平台本身（規格、原理、取樣方式、正常參考範圍），
即使該平台只用於人體，仍為 `measurement`——閘門問的是主詞，不是應用場景。

### `fact` 與相鄰型別的分界（v8）

`fact` 是五個型別都不貼切時的落點，最容易吸走相鄰型別的條目。四條分界：

| 分界 | 判準 |
|---|---|
| `fact` vs `definition` | 陳述某物「是什麼」（組成、分類、成分比例）→ `definition`；陳述「發生了什麼」或「處於什麼狀態」→ `fact` |
| `fact` vs `mechanism` | statement 含因果連接（「經 X 導致 Y」「活化」「誘導」）→ `mechanism`；只陳述狀態或並列事實 → `fact` |
| `fact` vs `measurement` | 數值出自 statement 內具名的量測平台，且驗證的核心是「該平台測到什麼」→ `measurement`；否則 → `fact` |
| `fact` vs `epidemiology` | **主詞為人體族群、且動物或體外證據不足以判定者 → `epidemiology`**；其餘 → `fact` |

> **最後一條是硬性的，不是風格偏好。**
> `epidemiology` 與 `therapeutic` 帶有「必須人體證據才能判 `verified`」的約束，
> `fact` 沒有。一條人體族群宣稱若填成 `fact`，那道閘門就不會啟動，
> 而**沒有任何欄位會顯示它曾經被繞過**。
> 判準是「這條 statement 能不能用動物研究驗證」——不能，就是 `epidemiology`。

〔注意〕時序與階段宣稱（「高血糖出現後數週內出現 X」）歸 `fact`。
它們不是因果路徑，也不是族群統計；硬歸 `mechanism` 會使檢索取向指向基礎研究，
而這類宣稱要查的是臨床觀察與動物模型的時序實驗。

### `source_ref` 格式

格式：`{doc}#{section}`，多個位置以 `;` 分隔，**第一個為主要出處**。

```
glycocalyx/v3#5.5
glycocalyx/v3#5.5;mitochondria/axis#1.2
```

- `{doc}`：`source/` 下的相對路徑去掉 `.md`。`source/` 依主題分子目錄
  （`glycocalyx/`、`mitochondria/`），子目錄前綴同時是網頁分站的依據
- `{section}`：該文件內的節號。文件無節號時用可辨識的位置標籤（`表1`、`L122-131`）

`section` 欄的值必須是 `ledger/TOPICS.md` 所定義的正規主題樹節號，
取該主張**主題所屬**的節點，**不必等於** `source_ref` 的節號。

```
claim_id   = 2.2-brain-mouse-0.54-0.23
section    = 2.2
source_ref = intake/2026-07-28-糖萼層生理#L154

claim_id   = 5.6.2-dr-heparanase-20-100x
section    = 5.6.2
source_ref = glycocalyx/heparanase#3.2        ← 來源節號為 3.2，section 取 5.6.2
```

理由：`section` 是**主題座標**，不是位置指標。批次劃分（STATUS §4）與
`claim_id` 排序都依賴它把同主題的條目聚在一起。

受追蹤文件各有自己的節號體系且互相衝突——`v3` 33 節、`heparanase` 64 節、
`axis` 13 節全部從 `1.1` 起算，重疊 23 個節號而語意不同
（`v3 §4.1` 是酶介導降解，`heparanase §4.1` 是糖尿病神經病變的時間序）。
若照抄來源節號，同一個 `section` 會裝進不相干的主張。
intake 檔的位置標籤（`L154`）同理不承載主題資訊。
位置資訊由 `source_ref` 承載，不重複。

**`TOPICS.md` 未涵蓋的主題，不得自行造節號。** 暫記於批次報告的
「待建節點」一節，該批結束後一併寫提案，核准後遞增 `topics_version`。

〔注意〕`topics_version` 與 `schema_version` 各自遞增。
主題樹隨新文件收錄而擴充，那不是 schema 變更。

### 一條主張，一列

同一主張在多份文件出現時**只建一列**，把所有位置寫進 `source_ref`。
不得為了區分文件而複製列。

理由：判定綁 claim，不綁文件。若同一主張有兩列，就會有兩個 `status`、兩份 `evidence`、
兩段 `note`。日後某列改判 `contested` 時只有一列被更新，另一列繼續顯示 `verified`——
與撤稿造成的「過期 verified」是同一種失效，只是來源是分身而非撤稿。

**認定規則。** 新增列前，先以數值為鍵掃描既有列：

視為**同一條**（既有列追加 `source_ref`，不建新列）：

- 數值、指標、族群、終點四者皆相同，僅措辭不同
- 數值相同、措辭不同，且可判定為同一來源的轉述

視為**不同條**（各自建列，兩列的 `note` 互相標示對方的 `claim_id`）：

- 數值相同但**歸屬不同**（例：11 μm 在一份文件歸腎絲球、在另一份歸腦部）
- 同一指標但**數值不同**（例：syndecan-1 敏感度 85% 與 90%）

第二組不得合併。**歸屬或數值在轉述間漂移，本身就是「無原始出處」的線索**，
合併會把這項資訊壓進 `note` 而失去可見度。兩列各自查證、互相指認，軌跡才留得下來。

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
   自 v7 起 `section` 取正規主題樹節號，來源文件的章節重編號**不再影響** `section`——
   只有 `TOPICS.md` 本身的變更才會。

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

### 相關但不對位的文獻

在 token 前加波浪號 `~` 表示該文獻**不足以支持或反駁**此主張，但與該主張的來源相關：

```
PMID:31234567;~PMID:39494362
```

**判準**：一篇文獻該不該加 `~`，看的是它與 statement 之間**至少有一個維度不對位**，
而該維度的差異使它無法作為支持或反駁：

| 不對位的維度 | 範例 |
|---|---|
| 量測對象 | statement 講 syndecan-1，文獻量的是 HSPG 總體 |
| 量測方法 | statement 的數字來自 PBR，文獻用 ELLA |
| 量綱 | statement 講前導時間，文獻給的是迴歸軌跡的提前量 |
| 介入 | statement 講 HRT，文獻的介入是補充劑 |
| 族群或血管床 | statement 講腦微血管，文獻量的是舌下 |
| 時間或劑量條件 | 取樣點晚於 statement 所述的時窗，不足以反駁 |

**`~` token 的 `note` 必須寫明不對位的是哪一個維度。** 未寫明者不得使用 `~`——
一個沒有說明的 `~` 無法與「懶得判斷」區分，會使本前綴失去意義。

**`~` 不是引用轉述文獻的後門。** `queries.md` §1.6 規定僅由他篇引述得知、
未直接取得的文獻不得據以判定，此規定對 `~` 同樣適用：
未直接取得者連 `~` 都不得填入，只能寫進 `note` 並註明「未直接取得」。

**前綴可變更。** 若後續檢索確認某 `~` token 實為支持或反駁，直接改其前綴
（`~` → 無前綴 或 `~` → `!`），並在 `note` 記錄改動日期與理由。
§7 的 append-only 約束的是**識別碼本身不得移除**，不約束前綴——
前綴記錄的是判定關係，判定關係本來就會隨證據更新。

### 排序

**第一個 token 為主要引用（primary reference）**，其後不拘。
主要引用應為證據等級最高、且最直接支持該 statement 數值的那一篇。
將來網頁只會顯示第一個。

主要引用必須是**無前綴**的支持文獻。
若該列沒有任何支持文獻（`evidence` 僅含 `!` 或 `~` token），
則該列**沒有主要引用**，將來網頁不得顯示其第一個 token 為代表文獻。

理由：`4.2.3-hrt-rct-2025` 的 `evidence` 僅含一個 `~` token，
那篇文獻證明的正是該主張沒有出處。照「顯示第一個 token」的字面規則，
會把一篇否證該主張的論文顯示成它的代表文獻，方向完全相反。

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

`tier` 反映 `evidence` 中**未加任何前綴**的 token 裡，證據等級最高的那一篇。
**反駁文獻（`!`）與不對位文獻（`~`）均不計入 `tier`。**

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
| `superseded` | 已被更新或更強的證據取代，**或已依 §6 拆列**。前者 `note` 寫 `superseded_by: {claim_id}`，後者寫 `split_into: {id};{id}[;...]`。兩者必須使用這兩個固定前綴，以便機器區分成因 |
| `unsupported` | 已主動檢索但找不到支持文獻。`note` 必須記錄檢索範圍 |

`~` 前綴的不對位文獻不影響 `status`。`status` 僅由支持與反駁文獻決定。
一列可以同時是 `unsupported`（無任何支持文獻）且含有 `~` token（漂移來源已定位）——
這兩件事不衝突，而且同時成立時該列的資訊量最高。

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

- **不得含時間形容詞。** 「最新」「近年」「截至 2025 年 11 月」等一律移除，
  時間資訊寫入 `note`。理由：statement 是要長期存放的斷言，
  含時間形容詞者會隨時間自動變成偽陳述，而沒有任何機制會提醒你去改它。

- **不得含未具名的權威性宣稱。** 「國際共識」「已成為新典範」「臨床標準」
  等措辭若無法指名具體的學會立場聲明或臨床指引，一律移除。

  若原文確實提出共識宣稱，處理方式是**把該宣稱本身建成一條 claim**，
  statement 改寫為可查核形式（例：「ADA 2025 年立場聲明將糖萼損傷列為
  糖尿病腎病變的早期核心事件」），檢索不到具名來源時判 `unsupported`。
  原文用語寫入 `note` 供追溯。

  理由：權威性宣稱與數值宣稱同樣可證偽，但更難察覺——
  數值錯了看得出來，「國際共識」錯了看不出來，而它承載的說服力更強。

### 跨文件敘述不一致時

當一條 claim 的 `source_ref` 含多份文件，且各文件的敘述不完全相同時，
`statement` 應寫成涵蓋各版本的形式，差異寫入 `note`。

若差異已達 §2「認定規則」中「不同條」的標準（歸屬不同或數值不同），
不得涵蓋——拆成兩列，各自的 `source_ref` 只記自己的出處。

### 既有列發現為混合宣稱時的拆列程序

「一列一個主張」約束的是新建列。既有列被發現含兩個以上可獨立證偽的宣稱時，
依下列程序拆解。**拆列不是刪列**，原列永久保留。

**步驟**

1. **原列**：`status` 改為 `superseded`，`note` 追加一行
   `split_into: {新 claim_id 1};{新 claim_id 2}[;...]`。
   原列的 `statement`、`evidence`、`tier`、`last_checked` **一律不動**——
   它們記錄的是拆列前的真實歷史狀態。
2. **新列**：每個子宣稱一列。`claim_id` 由原 id 衍生並加上區辨後綴
   （例：`5.4-s1p-preconditioning` → `5.4-s1p-iri-shedding`、`5.4-s1p-transplant`）。
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
- 原列曾達 `verified` 者：**須人工核准**。理由與本節禁止改變已 verified 列語意相同——
  該 `claim_id` 可能已被外部引用。

**回填時發現混合宣稱列的處置順序**

先拆再判，不要先判再拆。若判定已完成才發現，判定照常寫入，拆列另案處理，
原列的判定紀錄即為拆列前的歷史狀態。

〔注意〕拆出的新列若判 `unsupported`，仍須符合 `queries.md` §1.4 的三式檢索要求。
原列的檢索紀錄不自動及於子宣稱——原列可能只針對其中一個子宣稱查過三式。

---

## 7. Append-only

- **不得刪除任何列。** 錯誤的主張改 `status`，不刪除。
- **不得刪除既有的 `evidence` token。** 發現引錯了，加 `!` 前綴或改 `status`，
  並在 `note` 說明。刪掉就看不出曾經引錯過。
- 新增列一律接在檔尾，存檔前重新依 `claim_id` 排序。
- **拆列（§6）不違反本節**：原列保留、`claim_id` 不重用、識別碼不移除，
  新增的是列與 `note` 內容。原列的 `status` 由 `superseded` 覆寫是 §5 允許的狀態轉移。
- **`evidence` token 的前綴可變更**（見 §4）。本節禁止的是移除識別碼，不是調整前綴。

---

## 8. `last_checked`

每次針對某列執行檢索後更新為當日日期，**即使什麼都沒找到**。

這一欄的用途是回答「哪些條目太久沒複查」。
若某列的 `last_checked` 從未更新，代表 agent 從未真正處理過它——
`status` 停在 `unverified` 但 `last_checked` 為空，與 `status=unsupported`
是完全不同的狀態。

---

## 9. 值域與格式檢查（v10）

`claim_type`、`status`、`tier`、`priority` 四欄皆為封閉值域。
**每次寫入 `claims.csv` 後、commit 之前必須執行下列檢查，輸出須為空。**

```powershell
$ct = 'definition','fact','mechanism','measurement','epidemiology','therapeutic'
$st = 'unverified','verified','partial','contested','superseded','unsupported'
$ti = '','in_vitro','animal','human_obs','rct','meta','review'
$pr = 'high','med','low'
Import-Csv ledger/claims.csv | Where-Object {
  $_.claim_type -notin $ct -or $_.status -notin $st -or
  $_.tier -notin $ti -or $_.priority -notin $pr
} | Select-Object claim_id, claim_type, status, tier, priority
```

理由：`claim_type=fact` 在 2026-07-31 由 `runbooks/extraction.md` 引入，
四批抽取共 38 列使用一個當時 SCHEMA 未定義的值，
而列數、欄數、BOM、CRLF、排序、`section` 交叉驗證六項驗收全部通過。
**列舉值域是既有驗收唯一沒有涵蓋的一類，而它一行就能檢查。**

### `source_ref` 格式檢查（v10 新增）

`source_ref` 是自由文字欄，但 §2 對它有明文格式（`{目錄}/{檔名}#{節號}`，多值以分號分隔）。
**每個 token 必須同時含 `/` 與 `#`。**

```powershell
Import-Csv ledger/claims.csv | Where-Object {
  ($_.source_ref -split ';') | Where-Object { $_ -notmatch '/' -or $_ -notmatch '#' }
} | Select-Object claim_id, source_ref
```

輸出須為空。

理由：2026-08-02 的 §5B 抽取有 37 列把 `source_ref` 寫成裸節號 `5.5`，
而**七項既有驗收全部通過**——因為它們只驗四個封閉值域欄，
沒有任何一項涵蓋自由文字欄的格式。

〔注意〕這個缺陷不影響任何既有檢查會看的東西，但會在回填時才發作：
`source_ref` 是回填時回頭找原文的唯一座標，而裸節號 `5.5` 在三份受追蹤文件裡都存在，
指向三個不相干的地方。

〔推論〕本次修訂的一般教訓：**每為欄位寫下一條格式規定，同時問「哪一項檢查會驗它」。**
本檔目前仍有明文格式而無對應檢查的欄位兩個——`claim_id`（§3 的正規表示式）
與 `evidence`（§4 的 token 形式）。兩者尚未出過事，但成因結構與本例相同。

---

## 10. Commit 規範

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

## 11. 範例列

驗證前：

```csv
5.1-sdc1-cutoff-40,5.1,血漿 Syndecan-1 濃度大於 40 ng/mL 是預測敗血症死亡率的指標。,epidemiology,,,unverified,high,,最高優先。具體 cutoff，可能被誤用為臨床閾值,glycocalyx/v3#5.1
```

驗證後（假想）：

```csv
5.1-sdc1-cutoff-40,5.1,血漿 Syndecan-1 濃度大於 40 ng/mL 是預測敗血症死亡率的指標。,epidemiology,meta,PMID:36112233;PMID:34556677,partial,high,2026-08-03,原文 cutoff 為 40.4 ng/mL 且族群限於 ICU 成人敗血性休克；statement 需補上族群限定,glycocalyx/v3#5.1
```

注意這個例子：找到了文獻，但 statement 過度概括，所以是 `partial` 而非 `verified`。
**這是回填階段最常見的結果，不是失敗。** 逐條把過度概括的敘述收斂到文獻實際支持的範圍，
正是這份 ledger 的主要價值。

---

## 12. 變更本 Schema

新增欄位一律往後接，不得改動既有欄位的名稱、順序或語意。
任何變更必須遞增 `schema_version`，並在 `ledger/CHANGELOG.md` 記錄變更內容與日期。

`schema_version` 一旦遞增，前端解析程式與 agent 的提示詞都需同步檢查。
