# TimeMind — iOS 上架 Checklist

## 立即要做

- [ ] **host 隱私/條款網頁**
  - [ ] 申請網域 `timemind.app` (或用現有網域)
  - [ ] 把 `store-listing/privacy.html` 上傳為 `https://timemind.app/privacy`
  - [ ] 把 `store-listing/terms.html` 上傳為 `https://timemind.app/terms`
  - [ ] 設定 `support` 子頁或 email-only link
  - **備援 (免費):** 推上 GitHub Pages:`https://YOURNAME.github.io/timemind/privacy`

- [ ] **建 App Store Connect record**
  - [ ] 登入 https://appstoreconnect.apple.com
  - [ ] 我的 App → ＋ → 新 App
  - [ ] 平台:iOS
  - [ ] 名稱:`TimeMind — AI 任務管家`
  - [ ] 主要語言:繁體中文 (zh-TW)
  - [ ] Bundle ID:`com.timemind.app` (要先在 Apple Developer 註冊)
  - [ ] SKU:`TIMEMIND-2026-001`

- [ ] **填 Privacy Nutrition Labels**
  - 參考 `app-store-connect-metadata.json` 的 `app_privacy_labels` 段
  - 每個類別要勾 linked to user identity 與否

- [ ] **填 App metadata**
  - 從 `app-store-connect-metadata.json` 拷貝 description / subtitle / keywords

- [ ] **確認 Apple Developer 上註冊 bundle ID**
  - https://developer.apple.com/account/resources/identifiers/list
  - `com.timemind.app` 要先建好

## 接下來

- [ ] **截圖**
  - [ ] 用 production build 在 iPhone 跑,抓 6.7" 截圖至少 4 張
  - [ ] iPhone 6.5" 至少 1 張(向下相容)
  - [ ] iPhone 5.5" 至少 1 張(向下相容)
  - [ ] 抓 Spotlight / Widget 截圖各 1 張

- [ ] **App icon 1024x1024** 確認無透明,否則會被 reject

- [ ] **建 production build**
  - 把 development / preview 換為 production profile
  - `eas build --platform ios --profile production`

- [ ] **TestFlight beta**(建議,降低被 reject 機率)
  - 上 production build 到 TestFlight
  - 自己先測,reviewer 才不會找到明顯 bug

- [ ] **建立 demo 帳號**
  - `demo@timemind.app`
  - 預先 populate 範例任務
  - 密碼存為 EAS Secret:`APP_REVIEW_DEMO_PASSWORD`

- [ ] **reviewer notes** 從 metadata JSON 拷貝

- [ ] **確認 EAS Secrets 真的 set 了**
  - `DEEPSEEK_API_KEY`
  - `ASSEMBLYAI_API_KEY`
  - `eas env:list` 應該看得到這兩個(值不會被顯示)

## 最後送審前

- [ ] 全文 search 程式碼有沒有 `TODO` / `FIXME` / placeholder text 出現在 UI
- [ ] iPhone 上完整流程跑一遍(註冊/登入/建任務/語音輸入/分類/提醒)
- [ ] 確認 in-app 任何連結到隱私政策的 UI 都指向實際 host 的 URL(若還沒 host,在 review 期間可能會被問)
- [ ] 確認 `app.json` 是 production 配置
  - `userInterfaceStyle` 改為 single mode(避免自動切換讓 reviewer 困惑)
  - `bundleNumber` / `version` 鎖好

## 送審

- [ ] App Store Connect → Submit for Review
- [ ] 等 1–3 天審查
- [ ] 若被 reject:細看 Apple feedback,修完重 submit

## 補充:免費 host 隱私政策的方法

### 選項 A — GitHub Pages (推薦,完全免費)

1. 在 `store-listing/` 加 `index.html` (指向 privacy / terms)
2. 把 store-listing 推到 GitHub repo
3. Repo Settings → Pages → 來源 = main branch / store-listing folder
4. 網址:`https://YOURNAME.github.io/REPONAME/privacy.html`

### 選項 B — Notion / Carrd

1. Notion 把 privacy / terms 內容做成公開頁
2. 點 Share → Publish to web

### 選項 C — Cloudflare Pages (免費,彈性最大)

1. 註冊 Cloudflare
2. Pages → Connect to Git
3. 自動 deploy

### 選項 D — 直接買網域

建議域名:`timemind.app` (~$12-20 USD/year)
Cloudflare Registrar 最便宜
