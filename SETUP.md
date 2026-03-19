# PLAMOX — Firebase セットアップガイド

アカウント作成・投稿機能を動かすまでの手順です。

---

## ⏱ 所要時間: 約15〜20分

---

## STEP 1 — Firebase プロジェクト作成

1. **https://console.firebase.google.com** にアクセス
2. **「プロジェクトを作成」** をクリック
3. プロジェクト名: `plamox`（好きな名前でOK）
4. Google アナリティクスは任意 → 「プロジェクトを作成」

---

## STEP 2 — ウェブアプリを登録

1. プロジェクトのホーム画面で **`</>`** （ウェブ）アイコンをクリック
2. アプリ名: `plamox-web`
3. 「Firebase Hosting も設定する」は**チェックしない**
4. **「アプリを登録」** → 表示される `firebaseConfig` をコピー

```javascript
// こんな形のものが表示されます
const firebaseConfig = {
  apiKey: "AIza...",
  authDomain: "plamox-xxxxx.firebaseapp.com",
  projectId: "plamox-xxxxx",
  storageBucket: "plamox-xxxxx.appspot.com",
  messagingSenderId: "123456789",
  appId: "1:123456789:web:abcdef"
};
```

5. この値を **`config.js`** に貼り付ける（後述）

---

## STEP 3 — Authentication を有効化

1. 左メニュー **「Authentication」** → **「始める」**
2. **「Sign-in method」** タブ

以下を**有効化**:

| プロバイダ | 設定方法 |
|---|---|
| **メール/パスワード** | クリックして「有効にする」→ 保存 |
| **Google** | クリックして「有効にする」→ プロジェクトのサポートメールを選択 → 保存 |
| **Twitter** | ※下記の追加手順参照 |

### Twitter (X) の設定

1. **https://developer.twitter.com** でアプリを作成
2. App settings → Keys and tokens から `API Key` と `API Secret` をコピー
3. Firebase Console の Twitter プロバイダに貼り付け
4. Firebase が表示する**コールバックURL**をTwitter Appの設定に追加

---

## STEP 4 — Firestore Database を有効化

1. 左メニュー **「Firestore Database」** → **「データベースの作成」**
2. **「本番環境モードで開始」** を選択
3. ロケーション: `asia-northeast1`（東京）推奨 → **「有効にする」**
4. **「ルール」** タブを開き、以下のルールに**置き換えて保存**:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {

    // Users: 本人のみ書き込み可、全員読み取り可
    match /users/{userId} {
      allow read: if true;
      allow write: if request.auth != null && request.auth.uid == userId;
    }

    // Posts: ログイン済みユーザーが作成可、本人のみ削除可
    match /posts/{postId} {
      allow read: if true;
      allow create: if request.auth != null;
      allow update: if request.auth != null;
      allow delete: if request.auth != null
        && resource.data.uid == request.auth.uid;
    }
  }
}
```

---

## STEP 5 — Storage を有効化

1. 左メニュー **「Storage」** → **「始める」**
2. **「本番環境モードで開始」**
3. ロケーション: Firestore と同じリージョン → **「完了」**
4. **「ルール」** タブを開き、以下に置き換えて保存:

```
rules_version = '2';
service firebase.storage {
  match /b/{bucket}/o {
    match /posts/{userId}/{allPaths=**} {
      allow read: if true;
      allow write: if request.auth != null
        && request.auth.uid == userId
        && request.resource.size < 5 * 1024 * 1024
        && request.resource.contentType.matches('image/.*');
    }
  }
}
```

---

## STEP 6 — config.js を編集

`config.js` ファイルを開き、STEP 2 でコピーした値を貼り付けます:

```javascript
const FIREBASE_CONFIG = {
  apiKey:            "AIza...",          // ← あなたの値
  authDomain:        "plamox-xxxxx.firebaseapp.com",
  projectId:         "plamox-xxxxx",
  storageBucket:     "plamox-xxxxx.appspot.com",
  messagingSenderId: "123456789",
  appId:             "1:123456789:web:abcdef",
};
```

---

## STEP 7 — GitHub Pages にデプロイ

```bash
# リポジトリ作成・プッシュ
git init
git add .
git commit -m "PLAMOX v2 - Firebase integration"
git branch -M main
git remote add origin https://github.com/YOUR_USERNAME/plamox.git
git push -u origin main
```

GitHub リポジトリ → **Settings → Pages → Source: main / (root)** → Save

数分後に `https://YOUR_USERNAME.github.io/plamox/` で公開されます。

---

## ✅ 動作確認チェックリスト

- [ ] `https://YOUR_USERNAME.github.io/plamox/` を開く
- [ ] 「LOGIN / SIGN UP」→ メールで新規登録できる
- [ ] Google ログインができる
- [ ] 「＋ POST」で写真と情報を入力し投稿できる
- [ ] フィードに投稿が表示される
- [ ] ⚡ GP ボタンでポイントを送れる
- [ ] 🏴 FLAG ボタンで国旗を送れる

---

## 🆘 よくあるエラー

| エラー | 対処 |
|---|---|
| `auth/unauthorized-domain` | Firebase Console → Authentication → Settings → 承認済みドメインに GitHub Pages の URL を追加 |
| `storage/unauthorized` | Storage のルールを確認 |
| `Missing or insufficient permissions` | Firestore のルールを確認 |
| Twitter が動かない | コールバック URL の設定を確認 |

---

*PLAMOX — Built with Firebase × Claude*
