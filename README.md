# 日常小島

以手機為主的個人行程與任務管理系統，使用 Google Firebase Authentication 登入，並以 Cloud Firestore 儲存個人資料。

## 目前功能

- Google 登入後才能進入系統；個人待辦、課表、進度與小島佈置同步至 Firestore。
- 管理今日待辦：新增、編輯、刪除、完成；完成任務會累積點數，完成當日全部待辦會獲得額外獎勵。
- 課表 CSV 匯入、課程時間軸與空堂待辦建議。匯入器僅儲存課程名稱、星期、時間和類別，不讀取教師或教室欄位。
- 小動物依連續完成日解鎖（第 5、10 天）；裝飾物依累積完成日解鎖（第 1、3 天）。
- 21 天練習進度依完成日累積，休息日不會讓進度倒退。
- 空堂建議可呼叫 Firebase Functions 使用 Gemini 2.5 Flash；使用者點選 AI 建議時，才會傳送待辦名稱、預估時間和空堂長度，後端每日限制每位使用者 20 次。

課表 CSV 範本可在網頁的「匯入／管理課表」視窗下載。匯入器支援 UTF-8／Big5 CSV、獨立星期欄位，或含星期與起訖時間的課程時間欄位。尚未以中央大學實際匯出檔驗證欄位名稱；不同版本若格式不同，請提供一份移除個資後的樣本以調整對應方式。

## Firestore 資料結構

- `users/{uid}`：點數、累積完成任務數、連續完成日、本週完成天數、每日獎勵日期及小島佈置。
- `users/{uid}/tasks/{taskId}`：待辦名稱、類別、預估時間、完成狀態、點數與日期。
- `users/{uid}/activityDays/{yyyy-mm-dd}`：完成當日所有待辦的日期紀錄，用來計算本週完成天數與連續完成日。
- `users/{uid}/courses/{courseId}`：課程名稱、星期、起訖時間和課程類別。

每個使用者只能讀寫自己的資料。資料庫安全規則位於 `firestore.rules`。

## 在 Firebase Console 建立資料庫

1. 開啟 Firebase Console，選取已用於 Google 登入的 Firebase 專案。
2. 進入 **Build → Firestore Database**，選擇 **Create database**。
3. 選擇 **Production mode**。選定資料庫位置後通常無法變更，請依使用者所在區域與延遲需求選擇；台灣使用者可優先比較亞洲區域。
4. 建立完成後，到 Firestore 的 **Rules** 頁面，貼上 `firestore.rules` 的內容並按 **Publish**。
5. 確認 **Authentication → Sign-in method** 已啟用 Google，且 **Authentication → Settings → Authorized domains** 已加入網站網域。
6. 透過本機網站伺服器或 HTTPS 網站開啟頁面。Firebase ES modules 無法可靠地從 `file://` 網址載入。

第一次登入時，系統會建立 `users/{uid}` 個人資料文件；新增待辦或完成任務後，Firestore 會自動建立對應集合及文件，不需要手動新增集合。

## 使用 Firebase CLI 部署規則

已附上 `firebase.json` 與 `.firebaserc`，預設指向此專案設定中的 Firebase 專案。安裝並登入 Firebase CLI 後，在專案目錄執行：

```sh
firebase deploy --only firestore:rules
```

目前 `firebase-config.js` 是 Firebase Web app 的用戶端設定。此設定會出現在瀏覽器，並非伺服器私密金鑰；仍應在 Google Cloud/Firebase 設定 API 限制，並維持 Firestore 安全規則。

## 尚需外部設定

- LINE 登入尚未接上。需要 LINE Login Channel ID／Channel Secret，並透過可信任的伺服器端流程簽發 Firebase 登入憑證；Channel Secret 不可放在前端或提交到 Git。
- Gemini 後端程式已備妥，但需先設定 Gemini API Key Secret，再部署 Cloud Function。API 金鑰不可放入 `firebase-config.js` 或前端程式。

### 啟用 Gemini 空堂建議

1. 在 Google AI Studio 建立 Gemini API Key。
2. 在專案資料夾的終端機執行 `firebase functions:secrets:set GEMINI_API_KEY`，並在 CLI 提示時輸入金鑰。
3. 執行 `firebase deploy --only functions:suggestTasks`。

Cloud Functions 部署可能需要 Firebase 專案啟用 Blaze 計費方案。Gemini API 與 Functions 會依各自的免費額度及用量計費；請勿把 API Key 貼在聊天或提交至 Git。
