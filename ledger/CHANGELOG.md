# claims.csv Schema 變更紀錄

依 `ledger/SCHEMA.md` §11：任何 schema 變更必須遞增 `schema_version`，並在本檔記錄變更內容與日期。

---

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
