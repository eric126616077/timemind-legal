# TimeMind — 正式上架 Apple App Store 完整指南

> 寫給未來的自己 / 我看到這份文件的時間點。所有指令都在 TimeMind repo 根目錄執行。

---

## STEP 0 — Pre-flight (上傳前一次確認)

- [ ] 你已是 **Apple Developer Program** 成員(US$99/年)
  - Team ID:`UPY5ZFB5BQ`(已驗證)
- [ ] 你有 EAS 帳號(`eas login` 通過)
- [ ] Bundle ID `com.timemind.app` 已在 Apple Developer 註冊
  - https://developer.apple.com/account/resources/identifiers/list
  - 沒註冊的話現在去註冊,等 5 分鐘

---

## STEP 1 — EAS Secrets(必做,否則 build 會 fail)

我加的 production security gate 會驗證這些 secret 是真的:

```bash
cd TimeMind

# 1a. DeepSeek
eas env:create --name DEEPSEEK_API_KEY --value "sk-你的真key" --environment production --visibility secret

# 1b. AssemblyAI
eas env:create --name ASSEMBLYAI_API_KEY --value "你的真key" --environment production --visibility secret

# 1c. Apple 審查用的 demo 帳號密碼
eas env:create --name APP_REVIEW_DEMO_PASSWORD --value "選一個強密碼" --environment production --visibility secret
```

驗證:
```bash
eas env:list
# 應該看到三個 secret 都列在 production environment
```

⚠️ **這些 key 不會被顯示,只會看到存在與環境。**

---

## STEP 2 — Vercel 部署隱私 / 條款(馬上,5 分鐘)

### 方式 A:獨立 repo(推薦,乾淨)

```bash
# 在你機器上的任何位置
mkdir timemind-legal
cp TimeMind/store-listing/* timemind-legal/
cd timemind-legal
git init
git add .
git commit -m "init: privacy + terms"
git branch -M main
git remote add origin git@github.com:YOURNAME/timemind-legal.git
git push -u origin main
```

接著在 https://vercel.com:
1. Add New Project → Import `timemind-legal`
2. Framework Preset:Other
3. Deploy

之後網址:
- `https://timemind-legal.vercel.app/privacy`
- `https://timemind-legal.vercel.app/terms`

(這兩個會經過 `vercel.json` 的 redirect → `privacy.html` / `terms.html`)

### 方式 B:接現有 TimeMind repo 子目錄部署

Vercel Project Settings → Root Directory 改為 `store-listing/`

(其他不變)

### 拿到 URL 之後

更新 `app.json` 內沒有的 URL,但你需要回來填:
- ✅ **App Store Connect** → App Information → Privacy Policy URL:填 `https://timemind-legal.vercel.app/privacy`(或你的 custom domain)
- ✅ Apple review notes 提到 `https://timemind-legal.vercel.app/terms`

---

## STEP 3 — Production iOS build

```bash
cd TimeMind
eas build --platform ios --profile production
```

第一次 build 會自動上傳 cert + provisioning profile(若 Apple team ID 沒自訂 distribution cert,EAS 會自動建)。

約 10-20 分鐘。產出 `.ipa`。

### 驗證 build 真的注入真 key

Build log 裡搜:
```
[app.config] DEEPSEEK_API_KEY loaded: sk-a***1234
[app.config] ASSEMBLYAI_API_KEY loaded: xxxx***yyyy
```

如果看到 `loaded: ` 之後是空的 / 拋錯 → secret 沒設好,build 會 fail。

---

## STEP 4 — TestFlight beta(強烈建議)

### 4a. 上傳到 TestFlight
```bash
eas submit --platform ios --profile production --latest
```
or 手動:`eas build` 完後下載 `.ipa` → Xcode → Organizer → Distribute → TestFlight。

### 4b. App Store Connect → TestFlight tab
- 選你的 build
- 提供 export compliance(若用 ITSAppUsesNonExemptEncryption=false 會自動過)
- 預設為 internal tester,Apple 通常 24 小時內審

### 4c. 自己裝來測
- TestFlight App → 從邀請連結加入
- 跑完整流程:登入 / 建任務 / 語音輸入 / 切分類 / 推播提醒

⚠️ **如果 build 在 TestFlight 上當掉 / 出現明顯問題,在送 App Review 前修掉。這是降低被 reject 的方式。**

---

## STEP 5 — Demo 帳號(給 Apple reviewer 用)

TimeMind 的 login 強制要帳號,reviewer 沒帳號會卡在 login page。**一定會被 reject**。

### 5a. 建立 demo 帳號
在 auth system / 後端插入:
- email:`demo@timemind.app`
- 密碼:`APP_REVIEW_DEMO_PASSWORD` 的值

### 5b. Pre-populate 範例資料
至少填入 8-12 個範例任務,要涵蓋:
- 工作分類 3-4 個任務(標題、備註、優先度、已排程時間)
- 學習分類 2-3 個任務
- 生活分類 2-3 個任務
- 至少 1 個已過期任務(展示狀態 UI)
- 至少 1 個有 deadline 但未完成
- 至少 1 個已完成任務

(讓 reviewer 開 app 直接看到有資料的 UI,而不是空白畫面。)

### 5c. 密碼安全
- 設一個你不會再用到的密碼,只給 reviewer 用
- 預先 rotate:`TimeMind` 上架通過後立刻 rotate 這個 demo 帳號的密碼或刪除帳號

---

## STEP 6 — App Store Connect metadata

### 6a. 建 App record
https://appstoreconnect.apple.com
- 我的 App → ＋ → 新 App
- 平台:iOS
- 名稱:`TimeMind — AI 任務管家`
- 主要語言:繁體中文 (zh-Hant)
- Bundle ID:`com.timemind.app`
- SKU:`TIMEMIND-2026-001`
- User Access:Full Access

### 6b. 填內容
拷貝 `store-listing/app-store-connect-metadata.json` 的內容:
- App Information → Name, Subtitle, Categories(PRODUCTIVITY + UTILITIES)
- Pricing and Availability → Free, All territories
- App Privacy → 對照 JSON 內 `app_privacy_labels` 填寫
- Version Information → 拷貝 description / keywords / what's new

### 6c. Privacy Policy URL
填 `https://你的vercel域名/privacy`

### 6d. App Review Information
- Sign-in required:YES
- Email:`demo@timemind.app`
- Password:填 `APP_REVIEW_DEMO_PASSWORD` 的值
- Reviewer notes:從 `app-store-connect-metadata.json` 拷貝
- Attachment:不用

### 6e. Version Release
選 "Manually release this version"(通過審查後才下,讓你還有時間緊急撤)

### 6f. Screenshots
至少這幾種 device 的截圖:
- 6.7"(iPhone 15 Pro Max):3-5 張
- 6.5"(iPhone 11 Pro Max):1-2 張
- 5.5"(iPhone 8 Plus):1 張

怎麼截:
```bash
# 用你的 development / preview build,在真機上跑
# iPhone → 按側邊鍵 + 音量上
# 設成 6.7" 尺寸:設定 → 相機 → 保留照片格式 → 「相容性最高」
```

---

## STEP 7 — 送審

### 7a. 從 EAS Submit(或手動)
```bash
eas submit --platform ios --profile production --latest
```

(若 `eas.json` 裡 `submit.production.ios.ascAppId` 還沒填,會先問)

第一次 submit 時會請你登入 App Store Connect 做 OAuth。

### 7b. 在 App Store Connect:
- 選你的 build
- 填完所有必填欄位
- Submit for Review

### 7c. 等審查結果
- 通常 24-72 小時
- Email 通知

---

## STEP 8 — 若被 Reject

常見原因:
1. **Guided Required**:審查員要影片。建議:送審前自備 30 秒 demo 影片
2. **Privacy Policy 不夠明確**:檢查 URL 是否 public(不是 localhost)
3. **Required Permission Description 不夠清楚**:對照 Info.plist 的 NSMicrophoneUsageDescription 等
4. **Login Required but no demo account**:你已經準備好 demo 帳號,應不會遇到
5. **App Icon 有透明像素**:重新輸出 PNG
6. **Bundle ID 不符 App Store Connect 設定**

讀 Apple 給的 Resolution Center message,照著改 → 重 submit。

---

## STEP 9 — 上架後

- [ ] 立即 rotate / 刪除 `demo@timemind.app` 帳號密碼
- [ ] 監控 App Store reviews
- [ ] 訂閱 `expo.dev` 通知,知道有新 build 釋出
- [ ] 監控 DeepSeek / AssemblyAI 帳單避免被濫用

---

## 最後檢查表(送審前)

| 項目 | 狀態 |
|---|---|
| Bundle ID `com.timemind.app` 已在 Apple Developer 註冊 | ☐ |
| EAS Secrets `DEEPSEEK_API_KEY` / `ASSEMBLYAI_API_KEY` / `APP_REVIEW_DEMO_PASSWORD` 已 set | ☐ |
| `eas build --profile production` 成功 | ☐ |
| TestFlight 內部測試通過 | ☐ |
| Vercel 部署 `privacy` / `terms` URL 公開 | ☐ |
| Apple Developer ID `UPY5ZFB5BQ` 在 `eas.json` `submit.production.ios.appleTeamId` | ✅(已加) |
| App Store Connect 上 App Record 已建 | ☐ |
| Privacy Nutrition Labels 填完 | ☐ |
| 隱私政策 URL 填到 ASC | ☐ |
| Demo 帳號 `demo@timemind.app` 已預先 populate 任務 | ☐ |
| 截圖:6.7" 至少 3 張,6.5" 至少 1 張,5.5" 至少 1 張 | ☐ |
| App Icon 1024x1024 無透明 | ☐ |
| Production build 含 `EAS_BUILD_PROFILE=production` env | ✅(已加) |
| App production build 缺 secret 會 throw | ✅(已加) |

---

## 額外:免費隱私 page 替代方案

如果不想用 Vercel:
- **GitHub Pages**:把 `store-listing/` push 上 GitHub → enable Pages → 拿到 `*.github.io/timemind-legal/privacy.html`
- **Notion Public Page**:Notion 寫 privacy + 公開分享
- **Carrd.co**($9/year,有漂亮 template)

---

## 一行話總結

```
EAS secrets → Vercel deploy → production build → TestFlight beta 
→ 建 demo account → App Store Connect 填 metadata → submit
```
