# 提案：拆分 `TOPICS.md` 節點 `6.3`

```
日期: 2026-08-09
狀態: 已裁示 2026-08-13 — 核准拆 6.3.1／6.3.2；6.3.3 不建；§4.1 一併核准（backfill.md §3A）
影響: ledger/TOPICS.md（topics_version 2 → 3）、ledger/claims.csv 的 section 欄 28 列
實際執行: 2026-08-13，另附帶 schema v12 新增 stale_after 欄（見 CHANGELOG）
不影響: 任何 claim_id、任何 statement、任何 status
時機: 建議在 heparanase.md §7 抽取之前執行
```

---

## 1. 為什麼現在提

`6.3`（標靶治療：肝素酶抑制與 SGLT2i）現有 **109 列，全部 `unverified`**，
是全帳本最大的單一節點，也是回填佇列裡最大的一塊。

`STATUS.md` §4 自 2026-08-02 起記著「§7 抽取前應先提案拆 `6.3`」，理由是量綱混雜。
本提案把當時只寫了結論的那句話補上實際清點。

**時機的關鍵**：§7 全節是 SGLT2i，會再往 `6.3` 加數十列，
而**它加的全部是同一種量綱**（某藥在某器官的試驗結果）。
等 §7 抽完再拆，分母變大但混雜結構不變——判斷不會更清楚，工作量會更大。

---

## 2. 清點：109 列裡有幾種東西

依 `claim_type` 分：`therapeutic` 66、`fact` 28、`mechanism` 14、`definition` 1。

`claim_type` 不是有用的切法——真正的差別在**查證來源**。逐列讀 28 條 `fact` 後，
可以切出兩個與其餘 81 列**查證流程不同**的群：

### 群 A — 藥物開發與法規狀態（20 列）

```
6.3-alpha-lipoic-acid-approval-eu          6.3-anti-heparanase-mab-phase1-2
6.3-clinical-candidates-hs-mimetic-cancer  6.3-diabetic-complications-no-human-trial
6.3-heparanase-inhibitor-dr-phase1         6.3-heparanase-inhibitor-phase2
6.3-heparanase-inhibitors-20-plus-trials   6.3-next-gen-inhibitors-5-8-ind-enabling
6.3-no-approved-specific-heparanase-inhibitor
6.3-ovz-hs1638-preclinical-small-molecule  6.3-pg545-pixatimod-phase2-cancers
6.3-pixatimod-no-diabetes-trial            6.3-pixatimod-only-active-oncology-compound
6.3-pixatimod-phase1b-2                    6.3-roneparstat-development-stalled
6.3-roneparstat-phase1-complete-phase2-halted
6.3-sst0001-phase2-pancreatic-dkd          6.3-sulodexide-approval-eu-asia
6.3-sulodexide-only-clinically-available   6.3-sulodexide-phase3-4-as-inhibitor
```

**查證來源是註冊資料庫與藥證資料庫，不是期刊文獻。**
`runbooks/backfill.md` 步驟 2 的檢索順序（Europe PMC → PubMed → 預印本 → ClinicalTrials.gov）
對這 20 列是**倒過來的**：前三個來源查不到「Phase II 中止」或「歐洲已核准」，
查得到的只有 ClinicalTrials.gov、EMA／各國藥證資料庫、公司公告。

三個具體後果：

1. **`tier` 欄對這群沒有意義。** `in_vitro`／`animal`／`human_obs`／`rct`／`meta`／`review`
   六個值沒有一個能描述「NCT 編號顯示該試驗狀態為 terminated」。
2. **`status=unsupported` 的語意不同。** 對文獻類，`unsupported` 是「這個宣稱站不住腳」；
   對註冊類，查不到往往只代表**該試驗從未登記**或**登記在非英語註冊系統**，
   而這在中國、俄羅斯、伊朗的試驗上很常見——本群至少 5 列涉及這些地區。
3. **這群會過期。** 「處於 Phase II」是一個**帶時間戳的狀態**，不是恆定事實。
   文獻類的判定做完就固定了，這群做完仍需定期重驗。目前 schema 沒有任何欄位記錄這件事。

〔注意〕第 3 點是本提案最實質的理由。**把會過期的宣稱與不會過期的宣稱混在同一個節點，
將來的監測迴圈無法只挑該重跑的那些。**

### 群 B — 抑制劑的化學身分與分類（8 列）

```
6.3-heparin-analog-sulodexide-class        6.3-muparfostat-sulfated-oligosaccharide
6.3-natural-inhibitors-sulforaphane-quercetin
6.3-necuparanib-glycol-split-mimetic       6.3-noncoag-heparin-minimal-anticoagulation
6.3-ovz-hs1638-more-stable-cheaper         6.3-pixatimod-pg545-same-compound
6.3-roneparstat-n-acetylated-glycol-split
```

結構、代號對應、藥物類別歸屬。查證來源是藥物資料庫與化學文獻，
一次查定終身，且**與治療效果無關**——`6.3-pixatimod-pg545-same-compound`
（三個代號是同一化合物）為真或為假，不影響任何療效宣稱的判定，
但它決定了**其他列該不該併為同一條**（SCHEMA §2 的「同一條」認定）。

〔推論〕群 B 其實是**其他列的前置依賴**。它應該最先回填，而不是排在 109 列裡碰運氣。

### 其餘 81 列

某藥在某器官的試驗結果、糖萼修復幅度、證據等級、監測目標值。
四者量綱不同，但**查證來源相同**（期刊文獻），
`runbooks/backfill.md` 現行流程可直接適用，不必為它們分節點。

---

## 3. 提案

`TOPICS.md` 新增兩個子節點，`6.3` 保留為父節點：

| 節號 | 主題 | 列數 | 查證來源 |
|---|---|---:|---|
| `6.3` | 標靶治療：療效、修復幅度與證據等級 | 81 | 期刊文獻 |
| `6.3.1` | 藥物開發與法規狀態 | 20 | 註冊資料庫、藥證資料庫、公司公告 |
| `6.3.2` | 抑制劑的化學身分與分類 | 8 | 藥物資料庫、化學文獻 |

**執行成本極低**：只改 28 列的 `section` 欄。
依 SCHEMA §3 規則 3，`claim_id` 前綴**不跟著改**——`6.3-pixatimod-phase1b-2`
移到 `6.3.1` 之後 ID 維持原樣。不動 `statement`、不動 `status`、不動任何 evidence。

**回填順序建議**：`6.3.2`（8 列，一批做完，且是其他列的前置依賴）
→ `6.3`（81 列，走現行流程）→ `6.3.1`（20 列，需先訂註冊類的判定準則）。

---

## 4. 需要一併裁示的兩件

1. **`6.3.1` 的判定準則要不要另寫。** 現行 `runbooks/backfill.md` 的檢索順序與
   `status` 判定樹都預設證據是期刊文獻。若同意拆，建議在該 runbook 加一節
   「註冊類主張的回填」，處理 §2 列的三個後果（`tier` 留空、`unsupported` 的語意、過期）。
   **不同意拆的話這節就不必寫**，故一併提。
2. **「證據等級」那 7 列（1A／1B／2A／2B／2C）要不要第三個節點。**
   它們是**對證據本身的後設宣稱**，查證方式是找具名指引或 GRADE 評級，
   與療效宣稱不同。本提案暫不拆，理由是列數少且與療效列高度耦合
   （多為「某藥的證據等級為 X」，與該藥的療效列同源）。
   若你認為該拆，`6.3.3` 一併建立即可，成本同樣只是 `section` 欄。

---

## 5. 不做的替代方案

- **不拆，等 §7 抽完再說**：§7 只加同一種量綱，混雜結構不變，分母更大。已否決。
- **改用 `claim_type` 區分而不動 TOPICS**：`fact` 同時涵蓋群 A 與群 B，
  而群 B 的 8 列裡有 1 列是 `definition`。`claim_type` 切不出查證來源的差別。
- **把群 A 整批標為不建列**：這違反 append-only，且該群本身有查證價值——
  §6.5 表中的「Phase II 進行中」若實為已中止，那是原文最容易過期的一類錯誤。
