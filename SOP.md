# SOP.md — 每週資料更新標準作業程序

## 時機

每週一開始前完成（週日晚或週一早）。

---

## Step 1：確認本週日期範圍

當週 Mon–Fri 的 `YYYY-MM-DD`，填入下方使用。
（app 只顯示週一至週五，週末無需建入）

---

## Step 2：週班表（9 間）— 必更新

每間都要重抓，舊 sessions 全部替換。

### cxms（8 間）

用瀏覽器開各診所頁面，直接讀取頁面文字：

| 診所 | URL |
|------|-----|
| 維恩骨科   | http://web.cxms.com.tw/wn/hosp.php |
| 富新骨科   | http://web.cxms.com.tw/fc/hosp.php |
| 得安診所   | http://web.cxms.com.tw/da/hosp.php |
| 昌惟骨科   | http://web.cxms.com.tw/cw/hosp.php |
| 昌禾骨科   | http://web.cxms.com.tw/ch/hosp.php |
| 土城杏光   | http://web.cxms.com.tw/xq/hosp.php |
| 得揚診所   | http://web.cxms.com.tw/dy/hosp.php |
| 力康骨科   | http://web.cxms.com.tw/lk/hosp.php |

**⚠️ 得揚診所日期 bug**：cxms 顯示的日期為錯誤值，需依「星期幾」欄位手動對應本週正確日期。

**blacklist 過濾**（建入前必須排除）：

| 診所 | blacklist |
|------|-----------|
| 維恩骨科 | 李毅康、黃鉦斌 |
| 得安診所 | 郭周樺 |
| 昌惟骨科 | 莊凱如、陳柏安 |
| 力康骨科 | 林恭毅、齊燕斌 |

### 永馨復健科（sc-dr 系統）

1. 開啟 http://sc-dr.com.tw/progress/3531035027.php
2. 點「下一周」直到顯示本週
3. 讀取陳彥誌的所有時段（whitelist：陳彥誌）

---

## Step 3：月班表（5 間）— 月中換日期，月初重抓

### 判斷：本週是否跨月？

- **否（月中）**：直接把上週 sessions 的 `date` 換成本週對應日期，內容不變
- **是（月底/月初）**：依下方方式重抓新月份班表

### 月初重抓方式

**健維骨科**
- FB 粉專：facebook.com/JianWeiGuKeZhenSuo → 找最新貼文（月初發布）
- whitelist：韓文江

**誠陽復健科**
- 開啟 Google Sites：sites.google.com/view/chengyang-clinic/home/門診時間
- 找當月月曆，逐週讀取
- whitelist：楊景堯

**康澤復健科**
- URL 規律：`https://kangzereh.com/wp-content/uploads/YYYY/MM/NN土城康澤115年M月診表.jpg`
- NN 為流水號，通常從 01 開始嘗試
- whitelist：無（全顯示）

**仁祐骨科**
- FB 粉專：facebook.com/jenyuorth → 找最新貼文
- whitelist：無（全顯示，醫師：劉彥麟、陳漢祐）

**順安復健科**
- FB 粉專：facebook.com/wellnesspmr → 找最新貼文
- whitelist：無（全顯示，醫師：滕學淵、陳俊宇）

---

## Step 4：固定班表（8 間）— 預設跳過

以下診所班表幾乎不變，除非收到診所公告異動，否則只需把 sessions 的 `date` 換成本週日期：

| 診所 | 快照位置 |
|------|----------|
| 禾安復健科 | snapshots/fb/c01_禾安復健科.txt |
| 正陽骨科 | snapshots/image/c08_正陽骨科.txt |
| 板橋維力 | weili-clinic.com（固定班表圖） |
| 土城維力 | snapshots/website/c11_weili_tucheng.txt |
| 陳正傑骨科 | snapshots/fb/c12_陳正傑骨科.txt |
| 悅滿意永和 | snapshots/image/c13_悅滿意永和.txt |
| 悅滿意新店 | snapshots/image/c14_悅滿意新店.txt |
| 祥明診所 | snapshots/website/c18_shiangming.txt |

---

## Step 5：更新 schedules.json

1. 刪除舊週所有 sessions
2. 依各來源建入新週 sessions
3. 更新 `meta`：
   ```json
   "generated_at": "YYYY-MM-DDTHH:MM:SS+08:00",
   "week_start": "YYYY-MM-DD",
   "week_end": "YYYY-MM-DD"
   ```

---

## Step 6：驗證

執行以下檢查，全部 ✓ 才能 push：

1. **22 間診所都有 sessions**（無診所空資料）
2. **whitelist/blacklist 無違規**（建入前逐間確認）
3. **日期全在本週 Mon–Fri 範圍**
4. **無重複 session ID**
5. **必填欄位無空值**（id, doctor_name, clinic_id, date, slot, time_label）

快速驗證腳本（在 Claude 環境中執行）：
```python
import json
data = json.load(open('schedules.json'))
clinics = {c['id']: c for c in data['clinics']}
sessions = data['sessions']
# 各診所筆數
for cid in sorted(clinics): 
    print(cid, len([s for s in sessions if s['clinic_id']==cid]))
# whitelist/blacklist 違規
for s in sessions:
    c = clinics[s['clinic_id']]
    if c['whitelist'] and s['doctor_name'] not in c['whitelist']:
        print('WL違規', s['id'], s['doctor_name'])
    if c['blacklist'] and s['doctor_name'] in c['blacklist']:
        print('BL違規', s['id'], s['doctor_name'])
```

---

## Step 7：更新快照（週班表和月班表有變動時）

- cxms 各間：更新 `snapshots/cxms/` 對應的 `.txt` 文字快照
- FB 月班表：更新 `snapshots/fb/` 對應的 `.txt`
- website 月班表：更新 `snapshots/website/` 對應的 `.txt`

---

## Step 8：Push

```
git add schedules.json snapshots/
git commit -m "週更新 YYYY-MM-DD 週"
git push origin main
```

GitHub Pages 約 1–2 分鐘後生效。

---

## 特殊情況處理

### 國定假日／補班

- 確認各診所是否休診或縮診（部分診所會在 FB 或網站公告）
- 月班表診所（c09/c15/c16/c17/c22）通常在班表圖片中直接標注，照圖片建入
- cxms 週班表診所以當週實際顯示為準

### 診所公告固定班表異動

1. 重新到來源取得新班表（截圖或文字）
2. 刪除該診所所有舊 sessions
3. 依新班表重建 sessions
4. 更新對應快照檔
5. 若 whitelist/blacklist 有變，同步更新 clinics 定義
