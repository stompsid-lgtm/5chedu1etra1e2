# SOP.md — 每週資料更新作業程序

## 執行時機

每週日晚或週一早上，在下週班表生效前完成。

---

## 前置確認

1. 確認本週日期範圍（週一至週五）
2. 確認是否跨月（月初需重抓月班表，否則沿用上月資料只換日期）

---

## Step 1｜週班表（9 間）— 每週必抓

用瀏覽器依序開啟，複製頁面文字，解析醫師與時段：

| 診所 | URL |
|------|-----|
| 維恩骨科 | http://web.cxms.com.tw/wn/hosp.php |
| 富新骨科 | http://web.cxms.com.tw/fc/hosp.php |
| 得安診所 | http://web.cxms.com.tw/da/hosp.php |
| 昌惟骨科 | http://web.cxms.com.tw/cw/hosp.php |
| 昌禾骨科 | http://web.cxms.com.tw/ch/hosp.php |
| 土城杏光 | http://web.cxms.com.tw/xq/hosp.php |
| 得揚診所 | http://web.cxms.com.tw/dy/hosp.php |
| 力康骨科 | http://web.cxms.com.tw/lk/hosp.php |
| 永馨復健科 | http://sc-dr.com.tw/progress/3531035027.php |

**cxms 解析規則：**
- 頁面文字格式：`上午一診醫師A日期醫師B日期...`
- 欄位依序對應週一~週五（c19 得揚例外，見下）
- 套用 blacklist（維恩/得安/昌惟/力康）過濾後才建入 sessions
- ⚠️ **c19 得揚**：系統顯示日期有 bug（顯示上上週日期），須依「星期幾」對應本週正確日期，不可直接使用頁面上的日期

**永馨解析規則：**
- 頁面為即時週系統，若預設顯示非本週需點「下一周」導覽
- 只顯示 whitelist 醫師陳彥誌

---

## Step 2｜月班表（5 間）— 月中只換日期，月初重抓

### 月中（同月份內）

直接將上週 sessions 的 `date` 欄位替換為本週對應日期（星期幾對應）。不需重新開啟來源頁面。

### 月初（換月第一週）

需重新取得新月份班表：

| 診所 | 來源 | 取得方式 |
|------|------|----------|
| 健維骨科 | facebook.com/JianWeiGuKeZhenSuo | 開 FB 粉專 Posts，找最新月班表貼文，截圖或逐格轉錄 |
| 仁祐骨科 | facebook.com/jenyuorth | 同上 |
| 順安復健科 | facebook.com/wellnesspmr | 同上 |
| 誠陽復健科 | sites.google.com/view/chengyang-clinic/home/門診時間 | 瀏覽器開啟，找本月月曆，轉錄當週列 |
| 康澤復健科 | kangzereh.com/tuchengkangzeappointment | 圖片 URL 規律：`wp-content/uploads/YYYY/MM/NN土城康澤115年M月診表.jpg`，更換年月後直接開啟 |

**FB 月班表注意事項：**
- bash/fetch 無法存取，需瀏覽器登入
- 健維、仁祐、順安均有特殊公休標記（例如 228 假日、春節），需逐格確認
- 228/國定假日期間通常改為僅早診或全院休息，依圖片標注為準

---

## Step 3｜固定班表（8 間）— 預設跳過

以下診所班表幾乎不變，**正常情況不做任何動作**：

| 診所 | 來源 | 備註 |
|------|------|------|
| 禾安復健科 | FB 2023 貼文 | whitelist: 陳柏誠醫師 |
| 正陽骨科 | 使用者提供圖片 | whitelist: 黃英庭/黃旭東/蔡馥如 |
| 板橋維力 | weili-clinic.com 圖片 | whitelist: 高逢駿/陳書佑 |
| 土城維力 | weili-clinic.com 圖片 | 無 whitelist |
| 陳正傑骨科 | FB 2023 貼文 | 單人診所 |
| 悅滿意永和 | 使用者提供圖片 | whitelist: 悅滿意江 |
| 悅滿意新店 | 使用者提供圖片 | whitelist: 悅滿意江/悅滿意王 |
| 祥明診所 | shiangming.com/time.php | whitelist: 黃有明 |

收到診所異動通知時才重新處理，並更新對應快照。

---

## Step 4｜寫入 schedules.json

1. 刪除本週舊 sessions（保留 clinics 定義不動）
2. 建入新 sessions，欄位：
   - `id`：格式 `{clinic_id}_{slot縮寫}{序號}_{診別}`，如 `c02_m1_上午一`
   - `doctor_name`：必須與 whitelist 字串**完全一致**
   - `date`：`YYYY-MM-DD`，只建週一到週五
   - `slot`：`morning` / `afternoon` / `evening`
   - `time_label`：顯示用文字，如 `上午 08:30–12:00`
   - `source_note`：診別或備註，可空
3. 更新 `meta.week_start`、`meta.week_end`、`meta.generated_at`

---

## Step 5｜驗證

執行以下檢查，全部通過才 push：

```
□ 22 間診所都有 sessions（週末只顯示週一~五，app 不顯示週六日）
□ 無 whitelist / blacklist 違規
□ 日期全在本週範圍（週一~週五）
□ 無重複 session ID
□ 無必填欄位缺漏
□ doctor_name 與 whitelist 字串完全一致
```

---

## Step 6｜Push

```bash
cd repo
git add schedules.json
git commit -m "週更新 YYYY-MM-DD ~ YYYY-MM-DD"
git push origin main
```

GitHub Pages 自動部署，約 1 分鐘後生效。

---

## 快照更新規則

- 週班表（cxms/永馨）：每週更新快照，存入 `snapshots/cxms/` 或 `snapshots/website/`
- 月班表：換月時更新快照
- 固定班表：異動時才更新快照
- 快照格式：純文字，記錄來源 URL、抓取時間、原始班表內容、過濾規則

---

## 常見問題

**Q：某診所 cxms 頁面顯示「本週無門診」或空白？**
A：記錄於 source_note，該診所本週無 sessions，屬正常。

**Q：FB 貼文找不到本月班表？**
A：往下捲 Posts 區，有些月班表發在月中而非月初。若確實沒有，沿用上月資料並在 source_note 標記「沿用上月」。

**Q：c19 得揚日期對不上怎麼辦？**
A：以頁面頂部的「星期一~星期五」欄位順序為準，忽略頁面日期數字，手動對應本週正確日期。

**Q：月班表圖片有特殊標記（紅字/底色）？**
A：通常代表異動或代診，依標記內容建入正確醫師名稱，並在 source_note 記錄（如「代診」）。
