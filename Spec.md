# Spec.md — 門診日曆 PWA

## 這個 app 在做什麼？

顯示新北市板橋／土城／中和周邊 22 間骨科／復健科診所的本週醫師門診時間表，讓使用者快速確認「哪天哪位醫師有診」。資料每週手動或半自動更新一次，部署在 GitHub Pages。

---

## 使用者流程

1. 開啟網址（或已加入主畫面的 PWA icon）→ 載入 `schedules.json`，預設顯示今天＋明天。
2. 左右滑動或點導覽箭頭 → 切換日期（2日檢視或整週檢視）。
3. 點右上角切換按鈕（`2日` / `週`）→ 切換檢視模式。
4. 看到想查的醫師 chip → 點擊 → 底部 sheet 展開，顯示：診所、日期、時段、備註。
5. 底部 sheet 內點「篩選此醫師」→ 全部其他醫師 dimmed，只突顯該醫師所有時段（跨診所）。
6. 點右上角齒輪 → 設定頁，可對每間診所手動追加顯示/隱藏特定醫師（存 localStorage）。
7. 下拉頁面（pull-to-refresh）→ 重新 fetch `schedules.json`。

---

## 技術限制

- **單一 HTML 檔案**：所有 CSS、JS 內嵌於 `index.html`，無外部框架、無 build step。
- **PWA（輕量）**：`apple-mobile-web-app-capable` meta tag，支援加入主畫面；無 Service Worker，無離線快取。
- **資料來源**：`schedules.json`，同目錄，每次請求帶 `?v=timestamp` 避免快取。
- **部署平台**：GitHub Pages（`stompsid-lgtm/5chedu1etra1e2`，branch: main），無後端。
- **週六/日**：預設 2日檢視會跳過週六日；週檢視只顯示週一至週五（schedules.json 也不含週末，視診所而定）。
- **使用者設定**：覆寫（extraHidden / extraShow）存在 `localStorage`，裝置間不同步。

---

## schedules.json 結構

```json
{
  "meta": {
    "generated_at": "ISO8601",
    "week_start": "YYYY-MM-DD",
    "week_end": "YYYY-MM-DD",
    "version": "string"
  },
  "clinics": [
    {
      "id": "c01",
      "name": "診所名稱",
      "color": "#hex",
      "source_type": "cxms|image|website|facebook|linevoom",
      "whitelist": ["醫師名"],   // 只顯示此清單；優先於 blacklist
      "blacklist": ["醫師名"]    // 隱藏此清單；whitelist 存在時此欄無效
    }
  ],
  "sessions": [
    {
      "id": "c02_m1",
      "doctor_name": "醫師名",
      "clinic_id": "c02",
      "date": "YYYY-MM-DD",
      "slot": "morning|afternoon|evening|other",
      "time_label": "顯示用文字",
      "source_note": "備註（可空）"
    }
  ]
}
```

**Filter 優先級**（`shouldShow` 函數）：
1. localStorage `extraHidden` → 強制隱藏
2. localStorage `extraShow` → 強制顯示
3. clinic `whitelist` 非空 → 只顯示名單內
4. clinic `blacklist` 非空 → 隱藏名單內
5. 以上皆無 → 全顯示

---

## 22 間診所清單

| ID  | 診所         | 資料來源    | whitelist              | blacklist            |
|-----|--------------|-------------|------------------------|----------------------|
| c01 | 禾安復健科   | facebook    | 陳柏誠                 |                      |
| c02 | 維恩骨科     | cxms        |                        | 李毅康, 黃鉦斌       |
| c03 | 富新骨科     | cxms        |                        |                      |
| c04 | 得安診所     | cxms        |                        | 郭周樺               |
| c05 | 昌惟骨科     | cxms        |                        | 莊凱如, 陳柏安       |
| c06 | 昌禾骨科     | cxms        |                        |                      |
| c07 | 土城杏光     | cxms        |                        |                      |
| c08 | 正陽骨科     | image       | 黃英庭, 黃旭東, 蔡馥如 |                      |
| c09 | 健維骨科     | facebook    | 韓文江                 |                      |
| c10 | 板橋維力     | linevoom    | 高逢駿, 陳書佑         |                      |
| c11 | 土城維力     | website     |                        |                      |
| c12 | 陳正傑骨科   | facebook    |                        |                      |
| c13 | 悅滿意永和   | image       | 悅滿意江               |                      |
| c14 | 悅滿意新店   | image       | 悅滿意江, 悅滿意王     |                      |
| c15 | 誠陽復健科   | website     | 楊景堯                 |                      |
| c16 | 康澤復健科   | website     |                        |                      |
| c17 | 仁祐骨科     | facebook    |                        |                      |
| c18 | 祥明診所     | website     | 黃有明                 |                      |
| c19 | 得揚診所     | cxms        |                        |                      |
| c20 | 力康骨科     | cxms        |                        | 林恭毅, 齊燕斌       |
| c21 | 永馨復健科   | website     | 陳彥誌                 |                      |
| c22 | 順安復健科   | facebook    |                        |                      |

---

## 開發要求

### 資料維護（每週）

- `schedules.json` 必須在週一前更新完畢（包含當週 Mon–Sat 的 sessions）
- 每個 session 的 `date` 必須是當週實際日期，不可沿用上週
- `meta.generated_at` 記錄產生時間；`meta.week_start` / `week_end` 標記週範圍

### 資料正確性要求

- 同一醫師不得在同日期同 slot 出現於多間診所（時段衝突）
- doctor_name 命名規則：
  - 一般診所：直接用醫師姓名（如 `黃旭東`）
  - 悅滿意系列：診所前綴＋姓（如 `悅滿意江`），用於區分跨診所同姓醫師
- whitelist / blacklist 字串必須與 sessions 中的 doctor_name **完全匹配**（`Array.includes` 精確比對）

### 各來源更新方式

| 來源        | 更新頻率   | 方式                            |
|-------------|------------|---------------------------------|
| cxms        | 每週       | web_fetch 解析，自動化          |
| image       | 極低       | 靜態，初次確認正確後幾乎不動    |
| website     | 每週～每月 | 依各診所頁面結構，半自動或手動  |
| facebook    | 每週       | 需人工截圖或提供貼文文字        |
| linevoom    | 每週       | 需人工截圖                      |

### 不做的事

- 不實作 Service Worker（避免快取問題）
- 不使用前端框架（保持單一 HTML 可直接編輯）
- 不做使用者帳號/雲端同步（過濾偏好存 localStorage 即可）
- 不做即時看診進度顯示（不同於 cxms / sc-dr 的掛號系統）
