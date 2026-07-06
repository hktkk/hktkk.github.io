# 到站時間資訊板維護筆記

版本：v35

## 發佈位置

- GitHub Pages 入口：`/eta/`
- 主頁：`/eta/outputs/ipad-eta-dashboard.html`
- 本機工作檔：`outputs/ipad-eta-dashboard.html`
- Service worker：`outputs/sw.js`
- 目前 cache 名稱：`eta-dashboard-v35`

遠端 repo 內的 ETA 專案位於 `eta/` 目錄。本機這份工作目錄保留舊結構，發佈時要把本機 `outputs/` 同步到遠端的 `eta/outputs/`。

## 主要檔案

- `outputs/ipad-eta-dashboard.html`：全部畫面、樣式、資料取得和互動邏輯。
- `outputs/sw.js`：PWA 離線快取清單。修改 HTML、CSS 或 JS 後要提升 `CACHE_NAME`。
- `outputs/manifest.webmanifest`：PWA 名稱、圖示和顯示模式。
- `outputs/icon.svg`：PWA 圖示。
- `index.html`：本機根目錄入口，導向 `outputs/ipad-eta-dashboard.html`。

## 畫面結構

- iPad 6 橫向使用，目標畫面為 2048 x 1536 pixels。
- 左側為 ETA 主區，右側為時間、日出日落和天氣。
- 首頁左側先顯示輕鐵，再顯示巴士車站入口。
- 巴士流程：首頁車站入口 -> 第二頁路線和前往終點 -> 第三頁放大到站時間。
- 輕鐵維持首頁顯示，不進入巴士的第二、第三層流程。
- 頂部按鍵由左至右保留：切換主題、手動更新、返回主頁。

## 資料來源

- 九巴 ETA：九巴公開 ETA API。
- 城巴 ETA：城巴公開 ETA API。
- 輕鐵 ETA：港鐵輕鐵到站資料。
- 天氣實況：香港天文台 `rhrread`。
- 本港天氣預報：香港天文台 `flw`。
- 天氣警告：香港天文台 `warnsum`。
- 日出日落：香港天文台 `SRS`。
- 屯門天氣：`TUEN_MUN_WEATHER_URL` 代理。

## 更新頻率

- 時鐘：每秒更新。
- ETA：每 15 秒更新。
- 天氣和日出日落：每 5 分鐘更新。
- 手動更新：同時更新 ETA、天氣和日出日落，並刷新頂部「更新時間」。

## 車站設定

巴士車站在 `busStations` 陣列維護。每個站可包含一個或多個 stop id，用來合併同名或相鄰方向的站牌資料。

目前巴士車站：

- 中電
- 大興街
- 石排站
- 山景總站
- 山景邨景麗樓

輕鐵站使用 `LRT_STATION_ID = 170`。

## 天氣警告處理

警告顯示時要排除：

- `actionCode` 為 `CANCEL` 的項目。
- 已過 `expireTime` 的項目。

這是為了避免紅色暴雨等已取消警告繼續留在資訊板。

## 發佈檢查

修改後至少執行：

```sh
node --check outputs/sw.js
git diff --check -- outputs/ipad-eta-dashboard.html outputs/sw.js MAINTENANCE.md
```

主頁 HTML 內嵌 JavaScript，可用以下方式抽出再檢查：

```sh
perl -0ne 'print $1 if /<script>(.*)<\/script>/s' outputs/ipad-eta-dashboard.html > /tmp/eta-dashboard-script-check.js
node --check /tmp/eta-dashboard-script-check.js
```

發佈到 GitHub Pages 前，確認遠端目標是：

```sh
git@github.com:hktkk/hktkk.github.io.git
```

遠端發佈目錄是 `eta/`，不要覆蓋 repo 根目錄其他網站內容。
