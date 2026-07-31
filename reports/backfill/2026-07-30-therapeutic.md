# 回填批次 therapeutic — 2026-07-30

處理 7 條： verified 0 / partial 4 / contested 0 / unsupported 2 / needs-review 1

本批**不限定同一 section**（跨 2.4、4.2.3、5.4、6.1），違反 `runbooks/backfill.md` §2 的批次規則。
這是 STATUS §4 刻意的分組：以 `claim_type` 而非 `section` 劃批，理由是治療宣稱的風險型態相同、
判定約束相同（SCHEMA §2：動物或體外證據不得判 `verified`），共用背景脈絡的效益仍在。
記此一筆，避免日後被讀成流程失誤。

`ledger/queries.md` §2.3 原本標記「治療類尚無實測檢索式」，本批為其首次填充。

---

## 判定摘要

- `2.4-precursor-insufficient`: **partial** — 人體 RCT 證實前體補充無效，但無人量測過 statement 的前提（硫酸化酶受損）
- `4.2.3-hrt-rct-2025`: **unsupported** — 三式查無 HRT 與糖萼的人體研究；漂移來源已定位，介入被換掉
- `5.4-s1p-preconditioning`: **NEEDS-REVIEW** — 唯一的 IRI 直接實驗推翻方向，且本列是混合宣稱
- `6.1-heparanase-inhibitor-target`: **partial** — 動物證據紮實；但「高特異性」在已進入人體者中不成立
- `6.1-metabolic-reprogramming-target`: **partial** — 線粒體半句體外成立，六糖胺半句查無且方向存疑
- `6.1-microbiome-target`: **unsupported** — 相鄰文獻豐富，但無一以糖萼為終點
- `6.1-sulodexide-mixed`: **partial** — 兩半句各有支持，但「重症」子句未獲支持

---

### `4.2.3-hrt-rct-2025` → **unsupported**

〔文獻〕三式檢索（HRT＋糖萼厚度＋隨機試驗／雌激素＋停經＋PBR GlycoCheck／
`"glycocalyx"` 與 `"hormone replacement therapy"` 精確詞組）均查無任何以 HRT 為介入、
以糖萼厚度為終點的人體研究。

**但查到了一篇幾乎全中的論文。** `DOI:10.14814/phy2.70428`
（Gimblet et al. 2025, *Physiological Reports* 13(12):e70428，`NCT:NCT06071728` 的事後分析）：

| 原 statement 的要素 | Gimblet 2025 | 相符 |
|---|---|---|
| 2025 年 | 2025-06-17 | ✓ |
| 小型 | n = 22 | ✓ |
| 隨機試驗 | 雙盲隨機安慰劑對照 | ✓ |
| 停經後女性 | 停經後女性 vs 老年男性 | ✓ |
| 糖萼厚度改善 | 停經後女性 PBR 4–25 下降 0.178 ± 0.148 μm（男性反而上升） | ✓ |
| **荷爾蒙替代療法** | **Endocalyx Pro 3712 mg/日，12 週** | ✗ |

六項要素中五項相符，錯的是**介入本身**。
更關鍵的是母試驗的排除條件明文包含「近 6 個月內使用或曾使用荷爾蒙治療」——
這篇論文的受試者**恰好是排除了 HRT 的一群人**。

〔推論〕這篇既不支持也不反駁本主張（量的是別的介入），依 SCHEMA §4 未填入 `evidence`，
全部寫進 `note`。這是「相關但不對位」的**第六個樣本**，見本報告末節。

另有一條線索未動用：Gimblet 2025 引述 Diebel et al. 2021，稱雌激素可保護 HUVEC 在
仿生休克條件下的糖萼。該文僅由引述得知，依 `queries.md` §1.6 不得代替原文，未取識別碼、未據以判定。
即使取得，體外研究依 SCHEMA §2 也不足以讓 `therapeutic` 條目達到 `verified`。

---

### `5.4-s1p-preconditioning` → **NEEDS-REVIEW**

本列 statement 含兩個可獨立證偽的子宣稱，判定結果相反：

**子宣稱一：S1P 預處理可減輕 IRI 造成的糖萼脫落 → 被直接推翻**

`PMID:32017300`（大鼠離體心臟，Langendorff，S1P 10 nmol/L 缺血前預處理）：

- 梗塞面積確實縮小（P = .01 vs IR）
- 但冠脈流出液 syndecan-1 濃度**無差異**
- 心肌 syndecan-1 免疫染色強度**無差異**
- 作者結論：S1P 的心臟保護**並非**由維持糖萼中的 syndecan-1 所介導

支持側只有 `PMID:24285115`（大鼠脂肪墊內皮，以去除血漿蛋白誘發脫落，S1P > 100 nM
經 S1P1 保留 HS、CS 與 syndecan-1 外域）——體外，且脫落的誘因不是 IRI。

**子宣稱二：保護移植器官 → 查無原始研究**

`PMID:33671524`（腎移植 IRI 與糖萼綜述）指出該領域證據多屬實驗性，
呼籲進行臨床研究。依 SCHEMA §5，綜述不足以單獨支持 `verified`，
循其引用亦未追到以移植結果為終點的 S1P 研究。

**為何不自行判定**

兩條規則同時觸發：

1. `runbooks/backfill.md` §4 第 1 款——找到的文獻推翻了原 statement 的方向（不是數值不同，是結論相反）
2. STATUS §3.11——一列多宣稱時 `tier` 必然誤導。若判 `contested`，`tier` 只能取 `in_vitro`
   （唯一的支持 token），而讀者無從得知那個 `in_vitro` 支持的是哪個子宣稱，
   更無從得知子宣稱二根本沒有任何 token

拆列是正解，但**拆列的程序仍是 STATUS §5 的待決事項**，不自行執行。
`status` 維持 `unverified`，`last_checked` 已更新，兩篇文獻寫入 `note`。

〔推論〕這是 §3.11 記錄的問題第二次實際擋下判定，且這次是在判定**之前**擋下的
（上一次 `2.2-glomerular-hs-charge` 是判完才發現）。程序缺口的成本已經從「事後補記」
變成「當場停工」，值得列為優先處理。

---

### `2.4-precursor-insufficient` → **partial**（含 statement 修訂）

**先修訂再檢索**（依 `backfill.md` 步驟 1 的改寫檢查）：

```
原  若細胞內硫酸化酶已受損，單純補充 GAG 前體無法恢復糖萼功能。
新  若細胞內硫酸化酶已受損，單純補充 GAG 前體未必能恢復糖萼功能。
```

原文 `v3` §2.4 的原句是「單純補充 GAG 前體**未必能**恢復功能」。
抽取進 ledger 時被寫成「無法」——**模態被強化了一級**，從或然變成全稱。
該列未達 `verified`，依 SCHEMA §6 可自由修訂，已改回與原文一致。

〔文獻〕`DOI:10.1152/japplphysiol.00651.2024`（Smith et al. 2024, *J Appl Physiol* 137(6)）
是同一篇論文裡的三段實驗：

| 系統 | 介入 | 結果 |
|---|---|---|
| db/db 小鼠 | DSGP 4 週 | 主動脈糖萼長度恢復（AFM），FMD 改善、動脈硬度下降 |
| 培養內皮 | 同配方 | 促進糖萼生長 |
| **第二型糖尿病退伍軍人 n=22** | **DSGP 8 週，雙盲隨機安慰劑對照** | **糖萼完整性與血管功能指標均未優於安慰劑** |

人體陰性結果支持「未必能恢復」。

**判 partial 而非 verified 的理由**：statement 的前提是「硫酸化酶已受損」，
而這篇（以及任何一篇）都沒有量測硫酸化酶功能。人體失敗與小鼠成功的差異可以有很多解釋
（暴露量、病程、量測敏感度、族群），把它歸因於硫酸化酶是原文的假說，不是這篇的結論。

反向對照亦須記下：`PMID:20865240` 顯示舒洛地特在第二型糖尿病可部分恢復糖萼維度。
故「補前體無效」並非普遍成立，這正是「未必能」這個模態存在的意義——
也正是抽取時把它寫成「無法」會造成的損失。

---

### `6.1-heparanase-inhibitor-target` → **partial**

`note` 原本問「須確認是否有進入臨床試驗的化合物」。答案是：有，但沒有一個對得上本主張。

〔文獻〕支持側 `PMID:38302978`（Gamez et al. 2024, *Cardiovasc Diabetol* 23:50，
勘誤 `PMID:38378538`）：兩種 HS 耗竭模型（酵素移除、內皮特異性 Ext1 剔除）都造成糖萼變薄、
視網膜溶質通量與腎絲球白蛋白通透性上升；db/db 小鼠以 OVZ/HS-1638 治療後糖萼深度改善，
且同時免於 DR 與 DKD 相關的通透性變化。依 SCHEMA §2，`therapeutic` 僅有動物證據不得判 `verified`。

〔文獻〕**「高特異性」在人體階段不成立。** 已進入臨床的 heparanase 抑制劑全部是
HS mimetic 異質性混合物：

| 化合物 | 最高階段 | 適應症 | 狀態 |
|---|---|---|---|
| pixatimod (PG545) | Phase Ib/II | 實體瘤，與 nivolumab 併用 | 進行中 |
| roneparstat (SST0001) | Phase I 完成 | 多發性骨髓瘤 | 停滯 |
| necuparanib (M-402) | Phase II | 胰腺癌 | 中止 |
| muparfostat (PI-88) | Phase III | 肝癌輔助 | 終止 |

四者適應症皆為腫瘤，無一以血管糖萼或發炎-降解循環為終點。
其異質性正是特異性的反面——`PMID:38378538` 那則勘誤修正的內容，
恰好就是在討論這類衍生物的批次變異如何增加核准難度。

〔推論〕本條的錯不在「有沒有藥」，而在**把前臨床化合物的特性寫成了已在人體驗證的類別特性**。
OVZ/HS-1638 確實比商業抑制劑更專一，但它還在 IND 前；已進人體的那四個都不專一。

---

### `6.1-metabolic-reprogramming-target` → **partial**

兩個子宣稱，只有前半有證據。

〔文獻〕線粒體半句：`PMID:31680061`（Tiemeier et al. 2019, *Stem Cell Reports* 13(5):803–816）。
hiPSC 衍生內皮細胞缺乏管腔糖萼，且 mPTP 功能異常導致線粒體功能下降、ROS 上升；
以 cyclosporine A 關閉 mPTP 後線粒體功能改善，**同時恢復了糖萼**，細胞得以順流排列。
這是「線粒體功能→糖萼合成」少見的直接因果實驗，但為體外 hiPSC 系統。

〔文獻〕六糖胺半句：查無任何「抑制 HBP 過度活化可改善糖萼合成」的研究。
且方向存疑——HBP 產出的 UDP-GlcNAc 正是 GAG 合成的原料
（軟骨文獻明確稱其為 GAG 的 building block），抑制它未必增加糖萼合成。

**連帶發現：intake 檔有一句方向自相矛盾的敘述。**
`intake/2026-07-28-糖萼層生理` 寫「六糖胺途徑過度活化：**消耗** UDP-GlcNAc，減少 GAG 合成原料」。
過度活化應該是**產生**更多 UDP-GlcNAc，不是消耗。這句的方向與 HBP 的生化事實相反。
已記入 `note`，另列追查（不屬本批範圍）。

---

### `6.1-microbiome-target` → **unsupported**

〔文獻〕三式檢索：

1. `gut microbiota modulation TMAO endotoxin reduction protects endothelial glycocalyx degradation study`
2. `"TMAO" "glycocalyx" endothelial shedding syndecan-1 damage experiment`（精確詞組）
3. `butyrate short-chain fatty acid probiotic supplementation endothelial glycocalyx protection intervention trial`

三式都查無任何以糖萼為終點的腸道菌介入研究。

相鄰文獻很多，但終點一致地不對位：

| 領域 | 有的終點 | 沒有的終點 |
|---|---|---|
| TMAO 與內皮 | eNOS 活性、ROS、ICAM-1/VCAM-1、IL-6 | 糖萼厚度／體積／覆蓋率／脫落標記 |
| SCFA／丁酸與內皮 | NO 生物可用性、GPR41/43、內皮依賴性舒張 | 同上 |
| 益生菌與內毒素 | 糞便與血清內毒素濃度 | 同上 |

〔推論〕這一型的困難不是查不到東西，是查到的東西都差一站。
「腸道菌 → 內皮功能」有文獻，「糖萼 → 內皮功能」有文獻，
但「腸道菌 → 糖萼」這一段是原文自己接起來的。與 STATUS §3.10 的機制相同，
只是那裡接的是「數值與器官」，這裡接的是「上游介入與下游結構」。

---

### `6.1-sulodexide-mixed` → **partial**

statement 本身已經是雙半句，兩半各有歸屬：

〔文獻〕前半「部分研究中可增加糖萼厚度」：`PMID:20865240`（Broekhuizen et al. 2010,
*Diabetologia*）。10 名第二型糖尿病男性與 10 名對照，舒洛地特 2 個月後，
以 SDF 與螢光/ICG 血管攝影量測的舌下與視網膜糖萼維度部分恢復。
限制：非隨機、無安慰劑對照、僅男性、n=10。

〔文獻〕後半「嚴重代謝紊亂患者中療效不一」：`PMID:22034636`（Sun-MACRO，
*JASN* 2012;23:123-130）。第二型糖尿病合併腎功能不全與顯性蛋白尿（>900 mg/日）、
已用最大劑量 ARB 者，隨機雙盲安慰劑對照；原訂收 2240 人，收到 1248 人提前中止，
1029 人年追蹤後主要腎臟複合終點無差異（26 vs 30 例）。

**差異（判 partial 的理由）**：Sun-MACRO 的終點是腎功能，不是糖萼厚度。
它支持的是「臨床療效不一」，不是「糖萼療效不一」——兩者是不同的量。
這與 STATUS §3.15「量綱不同的兩個指標被合併」同族：
把腎臟終點的陰性結果讀成糖萼終點的陰性結果，是同一種抹平。

**「重症」（critically ill）一段查無任何舒洛地特研究。** 該子句未獲支持，已記入 `note`。

---

## NEEDS-REVIEW

- `5.4-s1p-preconditioning`: 唯一的 IRI 直接實驗（`PMID:32017300`）推翻方向，
  且本列含兩個判定相反的子宣稱。拆列程序屬 STATUS §5 待決，未自行執行。詳見上方。

---

## 對原文件的修訂建議

### 高優先：`v3` §4.2.3 的 HRT 句——介入被換掉

原句：

> 2025 年的小型隨機試驗顯示，荷爾蒙替代療法（HRT）與糖萼厚度的改善相關，這為性別特異性治療提供了依據。

依已定調的標註政策（STATUS §5、`runbooks/source-editing.md` §2），
`status=unsupported` 對應 `〔查無出處〕`。建議在該句後加獨立標記段落，
並在標記內指出漂移來源，因為本例的價值不在「查無出處」四個字，
而在「真正存在的那篇是什麼」。

`intake/2026-07-28-糖萼層生理#L19` 有平行敘述（「2025年研究顯示荷爾蒙替代療法（HRT）
可部分恢復糖萼層」），依 §3.8 與已定調政策，`intake/` 一律不改。

### 中優先：`v3` §6.1「精準酶抑制」一項——限定條件缺失

原句稱「開發高特異性的 Heparanase 抑制劑，阻斷炎症-降解的惡性循環」。
方向本身沒錯，且是合理的展望寫法（該節標題就是「未來的治療窗口」）。
但讀者容易把它讀成已有此類藥物。建議標註為 `〔條件限定〕`，
限定內容：已進入人體試驗者皆為異質性 HS mimetic、適應症皆為腫瘤。

### 待追查（不屬本批）：`intake/2026-07-28-糖萼層生理` 的六糖胺途徑方向

「六糖胺途徑過度活化：消耗 UDP-GlcNAc，減少 GAG 合成原料」——
過度活化應為產生而非消耗 UDP-GlcNAc。屬 STATUS §3.3 的方向型錯誤，
但主體不是逆向指標而是代謝通量。建議在三檔抽 claim 時單獨建列。

---

## 本批次有效的檢索詞

- 治療類的起手式是**介入名＋結構終點＋物種**三件套，缺結構終點就會被功能終點淹沒。
  `... endothelial glycocalyx thickness/shedding ...` 是關鍵段，
  只寫 `endothelial function` 回傳的全部是 FMD 與 NO
- 判 `unsupported` 之前必須確認「終點不對位」而非「主張不存在」。
  本批兩條 `unsupported` 都是相鄰文獻豐富、終點差一站
- `claim_type=therapeutic` 應多跑一式**陰性結果**檢索：
  加入 `fails`、`no effect`、`not efficacious`、`terminated`。
  `6.1-sulodexide-mixed` 的 Sun-MACRO 與 `2.4-precursor-insufficient` 的人體陰性段
  都是靠這個角度拿到的，且兩者都是該條判定的決定性證據
- 查「某藥有沒有進臨床」時，藥名列表比概念詞有效：
  一式同時放 `pixatimod roneparstat` 兩個代號，一次拿到完整管線與各自的終止狀態

〔推論〕給後續 `therapeutic` 批次的通則：**先查陰性結果，再查陽性結果。**
治療宣稱的文獻分布是偏斜的——陽性結果容易找，找到就容易停手；
而決定 `verified` 與 `partial` 之別的資訊，通常在陰性那一側。
本批四條 `partial` 中有三條，是先找到陽性、再靠陰性檢索才把範圍收斂回來的。

---

## 「相關但不對位」的第六個樣本

STATUS §5「觀察中」記錄了 `evidence` 欄承載不了這類文獻的問題，前五個樣本為
`2.2-sepsis-shedding-30min`、`2.2-sdc1-halflife-2-8h`、`5.3-pbr-precedes-cognitive-2-3y` 等。

本批新增第六個，且型態比前五個更純：

`4.2.3-hrt-rct-2025` 的 `DOI:10.14814/phy2.70428`——年份對、規模對、設計對、
族群對、終點對、方向對，只有介入不對。它既不支持也不反駁，
但它幾乎確定就是原主張的來源，是本條上資訊量最高的一筆文獻，
而 `evidence` 欄裝不下它。

〔推論〕前五個樣本的共同點是「量錯了別的東西」，本例是「量對了東西但用錯了藥」。
若 `~` 前綴的提案成立，這個樣本可以用來檢驗提案的語意夠不夠寬——
`~` 若定義為「量測對象不同」會漏掉本例，定義為「不足以支持或反駁本主張，
但與其來源相關」才涵蓋得住。

---

## 待辦

- 取得 Diebel et al. 2021（雌激素保護 HUVEC 糖萼，仿生休克）的識別碼，
  補入 `4.2.3-hrt-rct-2025` 的 `note`。目前僅由 Gimblet 2025 引述得知
- 取得 CV122（LPS/CLP 敗血症小鼠）與 heparastatin SF4（肺 IRI 小鼠）的識別碼，
  作為 `6.1-heparanase-inhibitor-target` 的補充動物證據
- `DOI:10.1152/japplphysiol.00651.2024` 未取得 PMID，日後補
