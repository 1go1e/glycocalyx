# claims.csv Schema 變更紀錄

依 `ledger/SCHEMA.md` §11：任何 schema 變更必須遞增 `schema_version`，並在本檔記錄變更內容與日期。

---

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
