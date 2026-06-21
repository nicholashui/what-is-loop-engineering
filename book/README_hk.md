# Loop Engineering（循環工程）

> 一本關於 **Loop Engineering（循環工程）** 的教學書籍 —— 即係設計一套
> 系統，自動向 AI 編程代理發出提示、觀察、驗證並反覆迭代，而唔係靠人手
> 逐個回合咁去提示佢哋。

呢度係本書嘅目錄，亦係本書嘅正式入口。各章節係按照一個刻意安排嘅學習脈絡
排列：基礎同概念性嘅內容行先，咁樣當你讀到動手實作、以建構為主嘅章節時，
你已經明白咗 Loop 究竟係 *乜嘢*，以及佢 *點解* 會係咁運作。

## 點樣讀呢本書

你可以由頭到尾順住睇，當成一個完整課程；又或者用下面嘅連結直接跳去某個
題目。每一章開頭都有一段「本章內容」摘要，結尾有「重點回顧」，而且每一章
都有導覽頁腳，連結去上一章同下一章，等你隨時都可以順序咁讀落去。

## 目錄

呢份清單定義咗正式嘅閱讀次序。基礎同概念性嘅章節行先，跟住先至係實作、以
建構為主嘅內容。

### 基礎篇

1. [引言與學習目標](01-introduction_hk.md) —— 本書教啲乜，以及你將會達成嘅
   明確學習目標。
2. [讀者對象與先備知識](02-audience-and-prerequisites_hk.md) —— 本書係寫俾邊啲人
   睇，以及佢假設你已經具備嘅背景知識。
3. [乜嘢係 Loop Engineering](03-what-is-loop-engineering_hk.md) —— 為呢門學科
   下定義，以及佢同人手逐個回合提示有咩分別。
4. [淵源與 ReAct 的關連](04-lineage-and-react_hk.md) —— 由 prompt engineering
   （提示工程）演進到 loop engineering 嘅過程，以及佢同 ReAct 之間嘅關係。
5. [起源與歷史背景](05-origins-and-history_hk.md) —— 呢個名點嚟，以及作為前身嘅
   Ralph Loop 技術。

### 結構與流程

6. [Loop 的解剖：各個組件](06-anatomy-of-a-loop_hk.md) —— 深入剖析 Loop 嘅
   八個構成元件。
7. [Loop 的生命週期](07-loop-lifecycle_hk.md) —— 由頭到尾嘅完整流程，以及
   繼續／衍生／停止（continue / spawn / stop）嘅決策點。

### 實作篇

8. [建構一個 Loop：實務指南](08-building-a-loop_hk.md) —— 由最簡到生產級別嘅
   實作範例，加埋設定目標同護欄（guardrails）嘅模式。

### 判斷與綜合

9. [好處與何時應該用 Loop](09-benefits_hk.md) —— 點解 Loop Engineering 咁重要，
   以及佢適用於邊幾類工作。
10. [風險、限制與緩解措施](10-risks-and-mitigations_hk.md) —— 老老實實咁討論
    成本、「slop」（低質輸出），以及 Loop 仍然需要嘅判斷力。
11. [綜合與結論](11-synthesis-and-conclusion_hk.md) —— 以一個總結性嘅角度，將
    Loop Engineering 定位成一個細小而設計精良嘅控制系統。

### Loop 庫（Loop Library）

呢幾章將本書由「起一個 Loop」延伸到「策展同運行成個 Loop *庫*」。佢哋以公開發佈嘅
[Forward Future Loop Library](https://signals.forwardfuture.ai/loop-library/) 作為實例。

12. [Loop 庫：真實世界 Loop 嘅總目錄](12-the-loop-library_hk.md) —— 將 45 個生產級
    Loop 分門別類噉整理出嚟，再睇佢哋共有嘅可重用形態。
13. [實作一個 Loop 庫](13-implementing-a-loop-library_hk.md) —— 一個 Loop 條目嘅
    schema、儲存庫結構、搜尋，以及一個把任何條目變成運行中 Loop 嘅 runner。
14. [策展、營運與擴展一個 Loop 庫](14-operating-a-loop-library_hk.md) —— 一個共享庫
    嘅貢獻流程、品質標準、版本管理、度量指標同治理。

## 輔助資料

- [詞彙表](glossary_hk.md) —— 本書術語嘅唯一權威依據。
- [參考資料與作者主張的論點](references_hk.md) —— 由所提供嘅原始材料整理出嚟嘅
  論點，每一條都標示咗需要獨立查證。
