# 📘 English Submission Correction API

このAPIは、英作文の添削とフィードバックを行うWebサービスのためのバックエンドAPIです。  
ユーザーは英文を投稿し、AIによる添削を受け取り、それに対してコメントを行うことができます。

---

## 🔐 認証とユーザー登録

### `/auth/register` `POST`
- 管理者専用：新規ユーザーを登録
- リクエスト: `username`, `password`

### `/auth/login` `POST`
- ユーザーがログインし、セッションを取得
- リクエスト: `username`, `password`

### `/auth/logout` `POST`
- セッションを終了してログアウト

> ※ 認証後は、Cookie に `sessionid` を持った状態で他エンドポイントを利用

---

## ✍️ 投稿・添削機能

### `/submissions/input` `POST`
ユーザーが英文を送信し、AIが添削を返します。

- **リクエスト**:
  ```json
  {
    "id": "string",
    "text": "This is original text.",
    "res": "This is corrected text.",
    "corrections": ["Fix 1", "Fix 2"]
  }
