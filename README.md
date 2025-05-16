# 📘 English Submission Correction API

このAPIは、英作文の添削とフィードバックを行うWebサービスのためのバックエンドAPIです。  
ユーザーは英文を投稿し、AIによる添削を受け取り、それに対してコメントや人間による修正を行うことができます。

---

## 🔐 認証とユーザー登録

### `POST /auth/register`

- 管理者専用：新規ユーザーを登録  
- **リクエスト**:
```json
{
  "username": "yourname",
  "password": "yourpass"
}
```

---

### `POST /auth/login`

- ユーザーがログインし、セッションを取得  
- **リクエスト**:
```json
{
  "username": "yourname",
  "password": "yourpass"
}
```

---

### `POST /auth/logout`

- セッションを終了してログアウト  

> 認証後、Cookie に `sessionid` を保持した状態で他エンドポイントを利用

---

## ✍️ 投稿・添削機能

### `POST /submissions/input`

英文を送信し、AIが添削結果を返します。
たとえばtextに間違いが複数見られる時にはその文が配列として複数追加されます。
例が分かりにくいかもしれないです。

- **リクエスト**:
```json
{
  "id": "subm001",
  "text": "This is original text.",
  "res": "This is corrected text.",
  "corrections": [
    {
      "original": "This is original text.",
      "corrected": "This is corrected text.",
      "reason": "Subject-verb agreement."
    }
  ]
}
```

- **レスポンス**: 同じ構造のオブジェクトを返却

---

### `GET /submissions/select`

全ユーザーの添削済み投稿を一覧で取得します。

- **レスポンス**:
```json
[
  {
    "id": "abc123",
    "text": "Original text",
    "res": "Corrected text",
    "corrections": [
      {
        "original": "Original text",
        "corrected": "Corrected text",
        "reason": "Explanation here."
      }
    ],
    "comments": [
      {
        "user": "bob",
        "text": "Nice fix!"
      }
    ]
  }
]
```

```
GET /submissions/select?page=2&limit=10&sortBy=date
```
上記のようにして一挙取得における負担を軽減している

---

### `GET /submissions/by-user?username={username}`

指定ユーザーの投稿一覧を取得します。

---

## 💬 コメント機能

### `POST /submissions/comment`

添削投稿に対してコメントを追加します。

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

### `GET /submissions/comments?id={submissionId}`

指定された投稿IDに対するコメント一覧を取得します。

---

## 👤 人による修正機能

### `POST /submissions/human`

人間の修正を記録します。

- **リクエスト**:
```json
{
  "submissionId": "abc123",
  "corrections": [
    {
      "original": "He go to school.",
      "corrected": "He goes to school.",
      "reason": "Subject-verb agreement"
    }
  ]
}
```

- **レスポンス**:
```json
{
  "message": "Human correction submitted"
}
```
---

## 📤 `POST /submissions/save` – 投稿をデータベースに保存

このエンドポイントは、AIが添削した英文を「投稿」として保存するために使用します。  
ユーザーが `/input` でAI添削を確認した後、「投稿」として確定する際にこのAPIを呼び出します。

#### corrections 配列の要素:

```json
{
  "original": "has",
  "corrected": "have",
  "reason": "Subject-verb agreement"
}
```
#### レスポンス例:

```json
{
  "message": "Submission saved successfully",
  "id": "abc123xyz"
}
```

---

### 📝 使用例

```bash
POST /submissions/save
Content-Type: application/json
Cookie: sessionid=your_session_cookie

{
  "text": "I has a pen.",
  "res": "I have a pen.",
  "corrections": [
    {
      "original": "has",
      "corrected": "have",
      "reason": "Subject-verb agreement"
    }
  ]
}
```

---

### `GET /submissions/human?id={submissionId}`

指定された投稿IDに対する人間の修正一覧を取得します。

- **レスポンス**:
```json
{
  "submissionId": "abc123",
  "corrections": [
    {
      "original": "He go to school.",
      "corrected": "He goes to school.",
      "reason": "Subject-verb agreement"
    }
  ]
}
```

---

## ⚠️ エラーハンドリング

| ステータス | 内容                                           |
|------------|------------------------------------------------|
| `400`      | リクエストの形式が正しくない（必須フィールド不足など） |
| `401`      | 認証が必要（未ログイン状態）                     |
| `404`      | リソースが存在しない（例: 投稿が見つからない）     |
| `409`      | 登録済みユーザー名の重複などの競合               |
| `500`      | サーバー内部エラー（AI処理失敗・DB不具合など）     |

---

## 🔒 認証仕様（セキュリティ）

このAPIは Cookie に `sessionid` を含めてユーザー認証を行います。

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

## 🧪 使用例（典型的なフロー）

1. ユーザーがログイン（`/auth/login`）  
2. 英文を `/submissions/input` に投稿  
3. 添削結果を `/submissions/select` または `/submissions/by-user` で取得  
4. 他ユーザーが `/submissions/comment` でコメント  
5. 教師や人間が `/submissions/human` で追加修正

---

## 📂 ライセンス

このAPI仕様はプロジェクト用途に合わせて自由に拡張・変更してください。