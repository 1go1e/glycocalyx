# claims.csv Schema 變更紀錄

依 `ledger/SCHEMA.md` §11：任何 schema 變更必須遞增 `schema_version`，並在本檔記錄變更內容與日期。

---

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
