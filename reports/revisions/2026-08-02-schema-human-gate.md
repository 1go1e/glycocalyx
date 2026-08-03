# 修訂提案：`claim_type` 的人體證據閘門跨型別化（schema v9）— 2026-08-02

來源：`STATUS.md` §5「待決 — `measurement` 沒有人體證據閘門」（2026-08-01 記）。
核准 2026-08-02，同日執行。本檔兼作提案與執行紀錄。

---

## 1. 問題

schema v8 立了一條硬性分界：

> `fact` vs `epidemiology`：主詞為人體族群、且動物或體外證據不足以判定者 → `epidemiology`

理由是只有 `epidemiology` 與 `therapeutic` 帶有「必須人體證據才能判 `verified`」的約束。
一條人體族群宣稱若填成 `fact`，那道閘門不會啟動，而**沒有任何欄位會顯示它曾被繞過**。

問題是這條界線只畫在 `fact` 那一側。`measurement` 的分界寫的是：

> 數值出自 statement 內具名的量測平台，且驗證的核心是「該平台測到什麼」→ `measurement`

**「具名平台」與「人體患者」可以同時成立。** 一條「某疾病患者以某儀器測到某值」的宣稱
依字面落入 `measurement`，而 `measurement` 不帶人體證據約束。
同一道閘門在這條路上仍然是開的。

〔推論〕v8 修的是 `fact` 這條路，沒有修 `measurement` 這條。
成因是當時樣本只有三列，看起來像個別分類問題；六列並排才看得出是同一個缺口。
與本專案反覆遇到的型態一致：**要等第二批樣本才顯現**。

〔注意〕`heparanase.md` §5 整節就是「具名量測平台 + 人體患者」的集合
（SDF／GlycoCheck、CCM、流式細胞儀、雷射散斑、Dextran-40 體積測量、多光子顯微鏡），
預估會再產生 10–15 列同型。**這是選擇在 §5 抽取之前執行的唯一理由**——
先抽再改，要修正的是二十列而不是八列。

---

## 2. 決議

採行「把人體族群判準提升為跨型別第一順位」，遞增 `schema_version` 9。

SCHEMA §2 新增「`claim_type` 的判定順序（v9）」：

```
第一順位 — 人體閘門
  問：這條 statement 的真假，能不能用動物或體外研究判定？
    不能，主詞為人體族群或人體患者   → epidemiology
    不能，且陳述的是治療介入的效果   → therapeutic
    能                               → 進入第二順位

第二順位 — 型別分界
  definition / fact / mechanism / measurement 依既有分界表判定
```

兩條範圍限定：

1. **閘門適用 `definition`／`fact`／`measurement` 三型，`mechanism` 明文排除。**
   因果路徑本就以動物與細胞模型為主要證據，把描述人體場景的機制宣稱改判
   `epidemiology` 會使檢索取向錯置。若一條 `mechanism` 列的 statement 其實不含
   因果連接、只是人體觀察，那它從一開始就不該是 `mechanism`——
   那是型別分界問題，不是閘門問題。

2. **閘門問的是主詞，不是應用場景。** 「具名量測平台 + 人體患者」一律走
   `epidemiology`，平台名稱留在 statement 內不移除；但主詞若是平台本身
   （規格、原理、取樣方式、正常參考範圍），即使該平台只用於人體，仍為 `measurement`。

〔推論〕第 2 條是這次修訂真正的成本所在。改派後，`5.6.1-dkd-sdf-sublingual-50pct`
這類列的檢索取向從「SDF 這個平台測得什麼」轉為「第 2 型糖尿病族群的糖萼厚度變化」，
而平台的方法學問題（以舌下微循環代表腎小球狀態）不再由 `claim_type` 承載。
該推論本身未被檢驗，目前記在該列的 `note` 裡——**`note` 是唯一承載它的地方**。

---

## 3. 執行

### 3.1 改派（8 列，全部 `measurement` → `epidemiology`）

`STATUS.md` §5 已列出的 6 列：

```
5.6.1-dkd-sdf-sublingual-50pct                SDF，T2D 患者
5.6.1-dkd-em-early-thinning                   電子顯微鏡，DKD 患者
5.6.2-dr-octa-thickness-50-80pct              OCT-A，DR 患者
5.6.3-dn-nerve-biopsy-albumin                 神經活檢，糖尿病患者
5.6.3-dn-ccm-thickness-40-60pct               CCM，糖尿病患者
5.6.3-dn-em-microvascular-near-complete-loss  電子顯微鏡，糖尿病患者
```

執行時依 v8 的先例做全帳本 `measurement` 逐列複核（25 列），再增 2 列：

```
5.6.3-dn-iendf-skin-biopsy      皮膚活檢，糖尿病疼痛患者
5.2.2-longcovid-hs-severity     長新冠病情嚴重度標記
```

前者是 §4 抽取時與 `5.6.3-dn-nerve-biopsy-albumin` 同批建立的姊妹列，
待決清單漏了它。後者的漏失原因不同：statement 的主詞寫成「循環 HS 片段」，
沒有「患者」字樣，看起來像標記的性質宣稱——
但**長新冠只存在於人體，沒有動物對應**，閘門仍應啟動。

〔注意〕這是 v8 的重演：待決事項裡的清單是**當時樣本**，不是全集。
兩次修訂都在執行時掃出更多列（v8 是 6 → 18，v9 是 6 → 8）。
往後任何型別改派，逐列複核應視為執行的一部分，不是額外檢查。

8 列改派前皆為 `unverified` 且 `tier` 為空，**無既有判定因改型別而失效**。
各列 `note` 末尾註記
`〔2026-08-02〕claim_type 由 measurement 改為 epidemiology（schema v9 人體閘門）`。

### 3.2 邊界案例（複核後維持 `measurement`）

```
5.6.2-dr-adaptive-optics-early-thinning   主詞是影像平台的能力，非患者族群
5.5-pbr-sdc1-correlation                  主詞是兩個指標之間的關係，未限定人體族群
```

兩列都落在閘門的第 2 條範圍限定上。記此備查：
若日後這兩列在回填時發現其支持文獻全部來自人體世代，
代表「主詞」判準在實務上不夠銳利，屆時應再議。

### 3.3 順帶發現（不處理，記錄）

`1.0-egc-definition` 的 `section` 為 `1`，而 `claim_id` 前綴為 `1.0`，
`TOPICS.md` 沒有 `1.0` 節點。此列來自最初的 97 列種子，非近期抽取所建。
`claim_id` 是識別碼，變更會使既有引用失效，**不在本次範圍內自行處理**。

---

## 4. 驗收

```
273 列／11 欄／排序正確／無重複／無 BOM／無 CRLF
SCHEMA §9 四欄值域檢查：空
claim_type   measurement 25 → 17，epidemiology 36 → 44，其餘不變
status       不變（8 列皆為 unverified）
git diff     8	8	ledger/claims.csv（僅 8 行變動）
```

〔注意〕本次執行過程中發生一起工作區被改寫的事故，工作區曾一度同時存在
本次的 v9 編輯與一份來源不明的 310 列 `claims.csv`。
處置與教訓記於 `STATUS.md` §7，本次的 v9 編輯已從 HEAD 重做一次以確保可複現。
