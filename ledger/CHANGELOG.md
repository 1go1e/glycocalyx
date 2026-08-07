# claims.csv Schema 變更紀錄

依 `ledger/SCHEMA.md` §12：任何 schema 變更必須遞增 `schema_version`，並在本檔記錄變更內容與日期。
（v7 以前該節為 §11，v8 插入 §9 值域檢查後順延。歷史條目中的節號引用維持當時的編號，不追改。）

---

## schema_version 10 — 2026-08-07

不改欄位結構，只增加一項驗收條件。

- **§9 新增 `source_ref` 格式檢查**：每個 token 必須同時含 `/` 與 `#`，輸出須為空。
  §9 標題由「值域檢查」改為「值域與格式檢查」。
- 觸發案例：2026-08-02 的 §5B 抽取有 **37 列**把 `source_ref` 寫成裸節號 `5.5`
  而非 `glycocalyx/heparanase#5.5`，成因是寫入腳本的常數少了檔案前綴。
  **七項既有驗收全部通過**——它們只驗四個封閉值域欄，沒有一項涵蓋自由文字欄的格式。
  §2 早已明文定義該格式，但 §9 未把它納入：規格寫了，檢查沒驗。
- 該缺陷不影響任何既有檢查會看的東西，但會在回填時發作——裸節號在三份受追蹤文件裡
  都存在，指向三個不相干的地方。37 列已於 2026-08-02 §6B 批次修復。
- 記入 `STATUS.md` §3.23
- 同步檢查（§12）：`runbooks/backfill.md` §5 與 `runbooks/extraction.md` §6
  均以交叉引用方式提及 `source_ref` 格式，不複製格式定義，故無需同步。目前無前端

---

## schema_version 9 — 2026-08-02

`heparanase.md` §4 抽取時發現、§5 抽取前執行。與 v8 是同一個缺口的另一條路。

- **§2 新增「`claim_type` 的判定順序」**：判定型別時**先過人體閘門，再問型別**。
  閘門的問題是「這條 statement 的真假能不能用動物或體外研究判定」——
  不能且主詞為人體族群或人體患者 → `epidemiology`（若陳述治療效果 → `therapeutic`）。
  適用 `definition`／`fact`／`measurement` 三型；`mechanism` 明文排除
  （因果路徑本就以動物與細胞模型為主要證據，改判會使檢索取向錯置）。
- **明訂「具名量測平台 + 人體患者」一律走 `epidemiology`**，平台名稱留在 statement 內。
  反之，主詞是平台本身（規格、原理、取樣方式、正常參考範圍）者仍為 `measurement`——
  閘門問的是主詞，不是應用場景。
- 提案：`reports/revisions/2026-08-02-schema-human-gate.md`（核准 2026-08-02）
- 觸發案例：v8 的硬性分界只寫在 `fact` vs `epidemiology` 之間，
  而 `measurement` 的分界寫的是「數值出自 statement 內具名的量測平台」。
  **「具名平台」與「人體患者」可以同時成立**，
  「某疾病患者以某儀器測到某值」依字面落入 `measurement`，
  而 `measurement` 不帶「必須人體證據才能判 `verified`」的約束，閘門於是靜默失效。
  §2–§4 三批抽取累積 6 列，六項驗收與 v8 新增的 §9 值域檢查全部通過。
- 受影響列：**8 列**（`measurement` 25 → 17，`epidemiology` 36 → 44）
  - STATUS §5 已列出的 6 列：`5.6.1-dkd-sdf-sublingual-50pct`、
    `5.6.1-dkd-em-early-thinning`、`5.6.2-dr-octa-thickness-50-80pct`、
    `5.6.3-dn-nerve-biopsy-albumin`、`5.6.3-dn-ccm-thickness-40-60pct`、
    `5.6.3-dn-em-microvascular-near-complete-loss`
  - 執行時全帳本 `measurement` 逐列複核再增 2 列：`5.6.3-dn-iendf-skin-biopsy`
    （糖尿病疼痛患者的皮膚活檢）、`5.2.2-longcovid-hs-severity`
    （長新冠只存在於人體，無動物對應；主詞寫成「循環 HS 片段」而未含「患者」字樣）
  - 8 列改派前皆為 `unverified` 且 `tier` 為空，無既有判定因改型別而失效
  - 各列 `note` 末尾註記變更日期與依據
- 邊界案例（複核後**維持** `measurement`，記此備查）：
  `5.6.2-dr-adaptive-optics-early-thinning`（主詞為影像平台的能力，非患者族群）、
  `5.5-pbr-sdc1-correlation`（主詞為兩個指標之間的關係，未限定人體族群）
- 同步檢查（§12）：`runbooks/extraction.md` §6 欄位表已於 v8 改為交叉引用 SCHEMA §2、
  不複製值域，本次無需同步；`runbooks/backfill.md` 與 `CLAUDE.md` 未引用型別分界條文。
  目前無前端

---

## schema_version 8 — 2026-08-01

執行 STATUS §5「`claim_type` 六列改派」時，依核准的附帶要求做全帳本掃描而發現。

- **§2**：`claim_type` 新增合法值 `fact`（不涉及因果的事實陳述：狀態、時序、階段、
  非族群層級的數值），並明訂此欄為**封閉值域**。
- **§2**：新增「`fact` 與相鄰型別的分界」四條，其中
  `fact` vs `epidemiology` 一條為硬性規定——主詞為人體族群且動物或體外證據不足以
  判定者一律 `epidemiology`，理由是只有 `epidemiology`／`therapeutic` 帶有
  「必須人體證據才能判 `verified`」的約束，填成 `fact` 會使該閘門靜默失效。
- **新增 §9 值域檢查**：`claim_type`／`status`／`tier`／`priority` 四欄，
  每次寫入 `claims.csv` 後、commit 之前執行，輸出須為空。原 §9–§11 順延為 §10–§12。
- 提案：`reports/revisions/2026-08-01-claim-type-fact.md`（核准 2026-08-01，選項 3）
- 觸發案例：`runbooks/extraction.md` 於 2026-07-31 隨 v7 新建時，
  欄位表列出 `claim_type = fact / mechanism / therapeutic / epidemiology`，
  而 `fact` 從 v1 到 v7 從未列入 SCHEMA §2 的合法值。四批抽取共 **38 列**照填，
  列數、欄數、BOM、CRLF、排序、`section` 交叉驗證六項驗收全部通過。
  原始 121 列無一使用此值。
- 受影響列：**18 列**（38 列中）
  - 2026-08-01 先行核准的 6 列：`5.6.1-dkd-biomarker-uacr-correlation`、
    `5.6.1-dkd-sdc1-100-4-6x`、`5.6.2-dr-biomarker-severity-correlation`、
    `5.6.2-dr-biomarker-vs-hba1c` → `epidemiology`；
    `5.6.1-dkd-sdf-sublingual-50pct`、`5.6.2-dr-octa-thickness-50-80pct` → `measurement`
  - v8 逐列複核再改 12 列：
    → `definition` 2（`2.3.2-glypican1-core-protein`、`6.3-sulodexide-composition`）
    → `mechanism` 2（`3.4-ros-distress-500nm`、`3.4-ros-eustress-100nm`）
    → `epidemiology` 7（`4.2.1-dm-sdc1-shedding-3-10x`、`4.2.1-dm-thickness-50-70pct`、
      `5.5-triple-biomarker-elevation`、`5.6.1-dkd-heparanase-10-50x`、
      `5.6.1-dkd-sdc1-earlier-than-uacr`、`5.6.2-dr-heparanase-20-100x`、
      `5.6.2-dr-sdc1-earlier-than-fundus`）
    → `measurement` 1（`5.6.1-dkd-em-early-thinning`）
  - 維持 `fact` 20 列。各改動列的 `note` 末尾均註記變更日期與依據
- 同步檢查（§12）：`runbooks/extraction.md` §6 欄位表改為交叉引用 SCHEMA §2
  （不再複製值域），§7 收尾加入 §9 值域檢查，§8 新增第五個已知失效模式；
  `CLAUDE.md` 與 `runbooks/backfill.md` 的 SCHEMA 節號引用同步更新
  （§9 → §10、§11 → §12）。目前無前端
- **勘誤（2026-08-01）**：本條目的「同步檢查」在 `32ba706` 當下**並未落地**——
  `runbooks/extraction.md` 的修正版被誤置於 `reports/extraction/extraction.md`，
  該 runbook 本身仍是舊版。已於後續 commit 歸位。
  事故經過與檢討見 `STATUS.md` §7。**紀錄與實際不符，而六項驗收全部通過**

## schema_version 7 — 2026-07-31

三檔抽 claim 的前置檢查發現 `section` 節號全面碰撞後提出。

- **§2**：`section` 改為取 `ledger/TOPICS.md` 的正規主題樹節號，
  不必等於 `source_ref` 的節號。原本的 intake 例外併入此通則，不再單列。
- **§2**：`TOPICS.md` 未涵蓋的主題不得自行造節號，須提案並遞增 `topics_version`。
- **§3**：補明來源文件的章節重編號自 v7 起不再影響 `section`。
- **新建 `ledger/TOPICS.md`**（`topics_version: 1`，50 個節點）：
  v3 的 35 節原樣保留（含未編號總論段的 `1` 與 `1.3`），
  新增 10 節（`1.2.1`、`4.1.1`–`4.1.3`、`5.6`–`5.6.3`、`5.7`、`6.3`）
  與頂層節點 `7`（含 `7.1`–`7.4`）。
  建立後以 `claims.csv` 交叉驗證：帳本用到的 29 個 `section` 值全部有定義。
- 提案：`reports/revisions/2026-07-31-schema-canonical-topics.md`
  （核准 2026-07-31，選項 A；§3.2 待決取選項 1，收錄糖萼以外的肝素酶病理）
- **受影響列：無。** v3 的 33 個節號原樣成為正規樹節點，既有 123 列的 `section` 值不變。
- 同步檢查（§11）：`CLAUDE.md` 必讀清單已加入 `TOPICS.md`；
  `runbooks/extraction.md` 新建，含 `section` 歸位步驟。目前無前端。

## schema_version 6 — 2026-07-31

合併兩份提案，一次遞增。

- **§4**：新增 `~` 前綴，表示「不足以支持或反駁本主張，但與其來源相關」的文獻。
  含六維度判準表、`note` 強制說明不對位維度、禁止用於未直接取得的轉述文獻、前綴可變更。
- **§4**：主要引用必須為無前綴 token；`evidence` 僅含 `!` 或 `~` 的列沒有主要引用。
- **§5**：`tier` 改為只反映**未加任何前綴**的 token（原文為「未加 `!` 前綴」）。
  若漏改此處，`~` 會複製 v2 修掉的失效——`4.2.3-hrt-rct-2025` 的來源是一篇 RCT 事後分析，
  照舊字面會把一條 `unsupported` 標成 `tier=rct`。
- **§5**：`~` token 不影響 `status`。
- **§5**：`superseded` 的條件擴充為涵蓋拆列，規定 `superseded_by:` 與 `split_into:` 兩個固定前綴。
- **§6**：新增既有列的拆列程序（原列 `superseded` + `split_into`、新列 `split_from`、
  證據逐 token 歸屬、核准層級依原列是否曾達 `verified`、拆出的 `unsupported` 仍須三式檢索）。
- **§7**：明訂拆列不違反 append-only；前綴可變更不違反 append-only。
- 提案：`reports/revisions/2026-07-30-schema-nonaligned-evidence.md`（核准 2026-07-31，選項 A）
  `reports/revisions/2026-07-30-schema-split-row.md`（核准 2026-07-31，選項 A，含拆 `5.4`）
- 受影響列：
  - `~` 回填 4 列：`4.2.3-hrt-rct-2025`、`5.3-pbr-precedes-cognitive-2-3y`、
    `2.2-sdc1-halflife-2-8h`、`2.2-sepsis-shedding-30min`
  - 拆列 1 列 → 2 新列：`5.4-s1p-preconditioning`（superseded）→
    `5.4-s1p-iri-shedding`（contested）、`5.4-s1p-transplant`（unsupported）
  - 總列數 121 → 123
- 同步檢查（§11）：`runbooks/backfill.md` 步驟 4 決策樹已增列 `~` 分支；
  `ledger/queries.md` §1.6 已加交叉引用。目前無前端，agent 端無硬編碼邏輯

## schema_version 5 — 2026-07-28

- **§2**：新增 `section` 取值例外。主要出處位於 `intake/` 之下時，
  `section` 取主題所屬的受追蹤文件節號，而非 intake 檔的位置標籤。
- 提案：`reports/backfill/2026-07-28-intake-physiology.md` §4.3（核准 2026-07-28）
- 觸發案例：`intake/2026-07-28-糖萼層生理` 的 24 條淨新增主張無節號可用，
  照原規則會產出 `section=L154`，破壞 `section` 作為主題座標的功能
- 受影響列：既有 97 列的主要出處均為 `glycocalyx/v3`，不適用例外，無須回填
- 同步檢查（§11）：`runbooks/backfill.md` 步驟 5 已同步

## schema_version 4 — 2026-07-28

- **§6**：statement 不得含時間形容詞（「最新」「截至 2025 年 11 月」等），時間資訊入 `note`。
- **§6**：statement 不得含未具名的權威性宣稱（「國際共識」「新典範」「臨床標準」）。
  原文提出共識宣稱時，該宣稱本身建成一條 claim，改寫為可指名的形式；
  檢索不到具名指引或立場聲明時判 `unsupported`。
- 提案：`reports/revisions/2026-07-28-intake-heparanase-mito.md` §7.4（擴充後核准 2026-07-28）
- 觸發案例：`source/glycocalyx/heparanase.md` 出現「共識」12 次、「國際共識」7 次，
  但「指引／guideline」0 次，且未指名任何學會（ADA／EASD／KDIGO／ESC／ISPAD 皆 0 次）
- 受影響列：既有 97 列的 statement 未含此類措辭，無須回填。
  規則對後續所有新建列生效，尤其 `heparanase.md` 的抽取批次
- 同步檢查（§11）：`runbooks/backfill.md` 步驟 1 已同步；STATUS §3.4 已擴充

## schema_version 3 — 2026-07-28

- **§2**：新增第 11 欄 `source_ref`，記錄主張在原始文件中的出現位置。多值，`;` 分隔，第一個為主要出處。
  依 §11「新增欄位一律往後接」，置於 `note` 之後。
- **§2**：新增「一條主張，一列」與其認定規則。同一主張跨文件出現時只建一列，不得複製。
  認定規則的關鍵條款：數值相同但**歸屬不同**、或同一指標**數值不同**時，視為不同條，各自建列。
- **§6**：新增跨文件敘述不一致時的 `statement` 撰寫與拆列規則。
- **§10**：範例列補上 `source_ref` 欄。
- 提案：`reports/revisions/2026-07-28-schema-source-doc.md`（選項 A，含 §5 認定規則，核准 2026-07-28）
- 觸發案例：`糖萼層_Glycocalyx_生理.md` 與 `source/glycocalyx_v3.md` 共有 9 組相同數值，
  依現行 schema 收錄會產生重複列
- 受影響列：97 列全部，機械化回填 `source_ref = glycocalyx_v3#{section}`。
  `statement`／`claim_type`／`tier`／`evidence`／`status`／`priority`／`last_checked`／`note` 均未變動
- 同步檢查（§11）：目前無前端解析程式；`runbooks/backfill.md` 步驟 5 與 §5 已同步更新

## schema_version 2 — 2026-07-27

- **§5**：`tier` 改為只反映未加 `!` 前綴的 token；反駁文獻不計入。
  原規則會讓「被越強證據推翻的主張，`tier` 標得越高」，方向與該欄用途相反。
- **§5**：新增規定——`status=contested` 的列，`note` 必須寫明反駁文獻的等級與反駁的具體內容。
- 提案：`reports/revisions/2026-07-27-schema-tier-refuting-tokens.md`（選項 A）
- 觸發案例：`2.1.1-extreme-11um`（支持文獻 `in_vitro`、反駁文獻 `animal`）
- 受影響列：1 列。該列現行寫入的 `tier=in_vitro` 已符合新規則，**無需回填**。
- 同步檢查（§11）：目前無前端解析程式；agent 端無硬編碼 `tier` 邏輯，重讀 SCHEMA 即可。

## schema_version 1 — 2026-07-26

- 初版。
