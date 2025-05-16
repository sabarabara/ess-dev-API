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
  ```
- **レスポンス**: 同内容が返却されます

---

### `/submissions/select` `GET`
全ユーザーの添削済み投稿を一覧で取得します。

- **レスポンス**:
  ```json
  [
    {
      "id": "abc123",
      "text": "Original text",
      "res": "Corrected text",
      "corrections": ["Correction A", "Correction B"]
    },
    ...
  ]
  ```

---

## 💬 コメント機能

### `/submissions/comment` `POST`
投稿された英文に対してコメントを投稿します。

- **リクエスト**:
  ```json
  {
    "id": "abc123",
    "text": "Great correction!"
  }
  ```
- **レスポンス**:
  ```json
  {
    "message": "Comment posted successfully"
  }
  ```

---

## ⚠️ エラーハンドリング

全てのエンドポイントは以下のようなエラーに対応しています：

| ステータス | 意味 |
|------------|------|
| `400` | リクエストの形式が正しくない（必須フィールド不足など） |
| `401` | 認証が必要（未ログイン状態） |
| `404` | 対象のリソースが存在しない（例: 投稿が見つからない） |
| `409` | 登録済みユーザー名の重複などの競合 |
| `500` | サーバー内部のエラー（AI処理失敗・DB不具合など） |

---

## 🔒 セキュリティ

このAPIは `sessionid` を Cookie に含めてユーザー認証を行います。

```yaml
components:
  securitySchemes:
    SessionAuth:
      type: apiKey
      in: cookie
      name: sessionid

security:
  - SessionAuth: []
```

---

## 🧪 開発者向けメモ

- OpenAPI 3.0 準拠
- リクエスト・レスポンス形式はすべて `application/json`
- `corrections[]` は AI による修正箇所（文字列の配列）
- 投稿・コメントはIDに紐づいて保存・管理される

---

## 🛠 使用例

### 投稿 + 添削取得 + コメント

1. ユーザーがログイン（セッション保持）
2. 英文を `/submissions/input` に投稿
3. 添削結果を `/submissions/select` で取得
4. 他ユーザーが `/submissions/comment` でコメント

---

## 📂 ライセンス

本API仕様はプロジェクト用途に合わせて自由に拡張・変更してください。
