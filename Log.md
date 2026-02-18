# Log.md — 開發紀錄

## 已解決問題

### [圖片來源] 初次解析錯誤（c08 / c13 / c14）
**問題**：圖片診所（正陽骨科、悅滿意永和、悅滿意新店）的 sessions 錯誤，包含不在 whitelist 的醫師，且有命名錯誤。

**根本原因**：whitelist 過濾邏輯未在首次轉錄時執行，導致非 whitelist 醫師混入；OCR 識別錯誤（c14「羅醫師」被識別為「雞醫師」）；悅滿意命名規則中途錯誤修改導致 whitelist 比對失效。

**修正**：2026-02-18 逐格重新對照圖片，刪除非 whitelist sessions，補週六時段，還原命名規則。

**教訓**：圖片轉錄時必須同步執行 whitelist 過濾；命名規則變更前確認 whitelist 字串一致性。

---

### [c08 whitelist] 曾鵬文誤入 whitelist
**修正**：2026-02-18 移除。

---

### [c11 土城維力] OCR 錯字 + 幽靈醫師
**問題**：「張善魁」全部應為「張晉魁」；「林茂森」「陳奕成」來源不明混入。

**修正**：2026-02-18 全面改名，刪除幽靈醫師。

---

### [c10 板橋維力] 來源分類錯誤
**問題**：CSV 標記為 `linevoom`，實際來源為 weili-clinic.com 固定班表圖。

**修正**：2026-02-18 改 `source_type: website`，重新建立 13 筆 sessions。

---

### [c16 康澤復健科] 班表錯誤
**問題**：週四晚診醫師誤為「李紹安」（應為「許哲維」）；週五晚診缺失；週二下午多餘一筆。

**修正**：2026-02-18 對照 2 月月班表圖片修正。

---

### [c18 祥明診所] 週二晚 / 週五午缺失
**修正**：2026-02-18 補入兩筆，清理 source_note 不一致。

---

### [c19 得揚診所] cxms 日期顯示 bug
**問題**：cxms 系統對 c19 顯示舊日期（如顯示 02/18 實為 02/25）。

**處理**：長期已知問題，每週解析時依星期幾手動對應正確日期。已補入週六（2/28）缺失的兩筆。

---

### [CXMS 全面] blacklist 醫師未過濾
**問題**：c02/c04/c05/c20 共 13 筆 blacklist 醫師（李毅康、郭周樺、莊凱如、林恭毅、齊燕斌）出現在 sessions 中。

**根本原因**：首次建立時未執行 blacklist 過濾。

**修正**：2026-02-18 靈魂拷問時發現並刪除全部 13 筆。

---

### [c14 悅滿意新店] 週四下午多餘 session
**問題**：`c14_a10`（週四下午悅滿意江）圖片該格為空，屬誤建。

**修正**：2026-02-18 刪除。

---

### [c01 禾安復健科] doctor_name 與 whitelist 不一致
**問題**：FB 圖片醫師名為「陳柏誠醫師」，whitelist 設為「陳柏誠」，精確比對會失效。

**修正**：2026-02-18 whitelist 和全部 sessions doctor_name 統一改為「陳柏誠醫師」，與圖片一致。

---

## 待辦事項

### 自動化方向
- [ ] cxms 解析腳本：已可運行，需整合進每週部署流程
- [ ] c15 誠陽、c21 永馨 HTML 文字可程式讀取，可考慮自動化
- [ ] c16 康澤月份圖片 URL 有規律，可程式組 URL 再視覺解析
- [ ] FB 月班表（c09/c17/c22）：短期人工，長期可考慮 Graph API

---

## 當前已知問題

無。

---

## 各來源技術特性

### cxms（`web.cxms.com.tw`）
- 瀏覽器可讀，bash/curl 被 egress proxy 封鎖
- 回傳純文字，格式穩定
- c19 得揚有日期顯示 bug，每週需手動修正

### website 來源 URL

| 診所 | URL | 資料格式 |
|------|-----|----------|
| c10 板橋維力 | weili-clinic.com/news/category-5/post-30 | 固定班表圖片 |
| c11 土城維力 | weili-clinic.com/news/category-5/post-30 | 固定班表圖片（同頁兩張） |
| c15 誠陽復健科 | sites.google.com/view/chengyang-clinic/home/門診時間 | 月曆文字 |
| c16 康澤復健科 | kangzereh.com/tuchengkangzeappointment | 月班表圖片，URL 含年月 |
| c18 祥明診所 | shiangming.com/time.php | 固定班表靜態 HTML |
| c21 永馨復健科 | sc-dr.com.tw/progress/3531035027.php | 即時週班表系統 |

### Facebook 來源

| 診所 | URL | 班表類型 |
|------|-----|----------|
| c01 禾安復健科 | facebook.com/share/19qBxvUV52/ | 固定（2023 貼文） |
| c09 健維骨科 | facebook.com/JianWeiGuKeZhenSuo | 月班表 |
| c12 陳正傑骨科 | facebook.com/share/1EwsVWdZka/ | 固定（2023 貼文，勾叉格式） |
| c17 仁祐骨科 | facebook.com/jenyuorth | 月班表 |
| c22 順安復健科 | facebook.com/wellnesspmr | 月班表 |
