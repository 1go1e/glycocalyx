# claims.csv Schema 變更紀錄

依 `ledger/SCHEMA.md` §11：任何 schema 變更必須遞增 `schema_version`，並在本檔記錄變更內容與日期。

---

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
