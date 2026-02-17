# 診所門診時間 PWA

週資料：2026-02-23 (一) ~ 2026-02-27 (五)  
228 和平紀念日：2/27

## 快速部署

```bash
git init
git add .
git commit -m "週資料 2026-02-23~27"
git branch -M main
git remote add origin https://github.com/YOUR_USERNAME/clinic-calendar.git
git push -u origin main

# Settings > Pages > Source: main branch
# 訪問: https://YOUR_USERNAME.github.io/clinic-calendar/
```

## 檔案

- `index.html` - PWA 主程式 (25KB)
- `schedules.json` - 本週資料 (22 clinics, 269 sessions)

## 本週資料

- 22 診所完整資料
- 269 時段 (週一~五)
- 228 異動：健維全休、仁祐僅早診、順安無晚診

## 每週更新

只需更新 `schedules.json`，推送即可。

---

Generated: 2026-02-18
