# 日常小島

以手機為主的個人行程與任務管理原型，使用 Firebase Authentication 登入，並以 Cloud Firestore 儲存個人資料。

## Firestore 資料結構

- `users/{uid}`：點數、累積完成任務數、連續完成日、本週完成天數、每日獎勵日期及小島佈置。
- `users/{uid}/tasks/{taskId}`：待辦名稱、類別、預估時間、完成狀態、點數與日期。
- `users/{uid}/activityDays/{yyyy-mm-dd}`：完成當日所有待辦的日期紀錄，用來計算本週完成天數與連續完成日。
- `users/{uid}/courses/{courseId}`：預留課表資料集合；目前課表匯入仍為介面示意，尚未寫入 Firestore。

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
