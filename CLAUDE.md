# 床邊待辦(bedside-todo)專案交接筆記

> 2026-08-04 由 Claude Code 對話建立。給接手此專案的任何 Claude 對話/開發者。

## 這是什麼

住院醫師手機用的「各床待辦」PWA。情境:走在路上護理師交辦「某床開 order」,馬上點該床記下,回到電腦前照單處理。使用者(Lukas)管 11-12 床(病房 05W2)。

- **正式網址**:https://lukaschuang.github.io/bedside-todo/(手機 Chrome「安裝應用程式」→ 全螢幕獨立 app)
- **GitHub**:`LukasChuang/bedside-todo`(公開 repo)
- **本地路徑**:`~/Desktop/GitHub/bedside-todo`

## 鐵則:公開 repo,不得含任何病人資料(PHI)

程式碼裡沒有任何床號對應的病人姓名。名單由使用者第一次開 app 時用「床位管理 → 整批匯入」貼上(支援 `05W2 03 01 王○明` 或 `03-01 王○明` 格式,自動去除 `[禁]` 等註記),只存手機 localStorage。改功能時維持這個原則。

## 檔案結構(全部靜態,無 build step)

| 檔案 | 說明 |
|---|---|
| `index.html` | 整個 app(HTML+CSS+JS 單檔,約 480 行) |
| `sw.js` | Service worker,stale-while-revalidate;快取名 `CACHE = 'bedside-v1'` |
| `manifest.webmanifest` | PWA 設定(standalone、theme #0e7c86) |
| `icon-192.png` / `icon-512.png` | 圖示(深青底白色床鋪) |

## 資料結構(localStorage key `bedside-todo-v1`)

```json
{ "beds": [ { "no": "17-01", "name": "姓名", "todos": [
    { "id": 1723..., "text": "開 order 止痛藥", "done": false, "ts": 1723..., "doneTs": null } ] } ],
  "openNo": "17-01" }
```

匯入新名單時:同床號**且**同姓名 → 保留該床 todos;否則清空(換病人)。

## UI 重點

- 主畫面:床卡片清單;有未完成待辦的床 → 琥珀色左條+數字徽章;頂端總覽「X 床待辦,共 X 項」
- 點床頭展開(手風琴,一次一床),輸入框自動 focus;快速片語 chips:開 order/抽血/照會/影像/補病歷/家屬解釋
- 打勾 → 移到該床「已完成」區,留完成時間戳(當天顯示 HH:MM,跨天顯示 M/D)
- 「床位管理」浮層:逐床編輯、新增、刪除、整批貼上匯入、清除已完成、清空全部
- 深淺色主題:CSS token 三段式(`:root` + `@media prefers-color-scheme` + `[data-theme]` 覆寫)

## 部署/更新流程

1. 改 `index.html` 等檔案
2. **改功能必 bump `sw.js` 的快取版本號**(`bedside-v1` → `bedside-v2`…),否則使用者拿到舊快取
3. `git push origin main` 然後 `git push origin main:gh-pages`(**Pages 吃的是 gh-pages 分支**,當初靠推這個分支名自動啟用,沒動過 repo 設定)
4. 使用者手機要開兩次 app 才吃到新版(SW 背景更新機制)

## 環境注意事項(Lukas 的 Mac)

- 沒有 `gh` CLI;`git push` 走 osxkeychain 已存憑證,直接推即可
- 從鑰匙圈抽 token 呼叫 GitHub API 會被擋,需要 API 級操作(建 repo、改設定)請使用者上網頁做
- SSH port 22 到 github.com 不通,用 HTTPS
- 本專案跟 `~/Desktop/Monkey`(院內 Tampermonkey 腳本 repo)是分開的;Monkey repo 裡的 `床邊待辦.html` 是早期 artifact 版(內建姓名、已棄用),別搞混

## 歷史脈絡

1. v1 做成 claude.ai Artifact → 手機上有 Claude 外框、需登入、不像 app,棄用
2. v2(現行)改 GitHub Pages PWA,拿掉內建姓名改貼上匯入
3. 已實測通過:新增/打勾/時間戳/重載持久化/空狀態/整批匯入(含 `[禁]` 去除)
