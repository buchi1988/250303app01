# API リファレンス

250303app01 の API リファレンスドキュメントです。

## 目次

- [概要](#概要)
- [認証](#認証)
- [エンドポイント](#エンドポイント)
- [エラーハンドリング](#エラーハンドリング)
- [レート制限](#レート制限)
- [SDK](#sdk)

## 概要

250303app01 は RESTful API を提供しています。すべてのリクエストは HTTPS 経由で行われ、JSON形式でデータを送受信します。

### ベースURL

```
https://api.example.com/v1
```

開発環境:
```
http://localhost:3000/api/v1
```

### リクエストヘッダー

すべてのリクエストには以下のヘッダーを含める必要があります：

```http
Content-Type: application/json
Accept: application/json
Authorization: Bearer {your_api_token}
```

## 認証

### API トークンの取得

APIを使用するには、まずAPIトークンを取得する必要があります。

#### エンドポイント

```
POST /auth/login
```

#### リクエスト

```json
{
  "username": "your_username",
  "password": "your_password"
}
```

#### レスポンス

```json
{
  "success": true,
  "data": {
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "expires_in": 3600,
    "token_type": "Bearer"
  }
}
```

#### 使用例

```bash
# cURL
curl -X POST https://api.example.com/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "username": "your_username",
    "password": "your_password"
  }'
```

```javascript
// JavaScript
const response = await fetch('https://api.example.com/v1/auth/login', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    username: 'your_username',
    password: 'your_password'
  })
});

const data = await response.json();
console.log(data.data.token);
```

```python
# Python
import requests

response = requests.post(
    'https://api.example.com/v1/auth/login',
    json={
        'username': 'your_username',
        'password': 'your_password'
    }
)

data = response.json()
print(data['data']['token'])
```

### トークンの使用

取得したトークンは、すべてのAPIリクエストの `Authorization` ヘッダーに含めます：

```http
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

## エンドポイント

### ユーザー

#### ユーザー一覧を取得

```
GET /users
```

**パラメータ**:
- `page` (optional): ページ番号（デフォルト: 1）
- `limit` (optional): 1ページあたりの件数（デフォルト: 10）
- `sort` (optional): ソート順 (`asc` または `desc`)

**レスポンス**:
```json
{
  "success": true,
  "data": {
    "users": [
      {
        "id": 1,
        "username": "user1",
        "email": "user1@example.com",
        "created_at": "2025-11-11T00:00:00Z"
      }
    ],
    "pagination": {
      "current_page": 1,
      "total_pages": 10,
      "total_items": 100,
      "items_per_page": 10
    }
  }
}
```

**使用例**:
```bash
curl -X GET "https://api.example.com/v1/users?page=1&limit=10" \
  -H "Authorization: Bearer {your_token}"
```

#### ユーザー詳細を取得

```
GET /users/{id}
```

**パラメータ**:
- `id` (required): ユーザーID

**レスポンス**:
```json
{
  "success": true,
  "data": {
    "id": 1,
    "username": "user1",
    "email": "user1@example.com",
    "profile": {
      "name": "User Name",
      "bio": "User biography"
    },
    "created_at": "2025-11-11T00:00:00Z",
    "updated_at": "2025-11-11T00:00:00Z"
  }
}
```

#### ユーザーを作成

```
POST /users
```

**リクエストボディ**:
```json
{
  "username": "newuser",
  "email": "newuser@example.com",
  "password": "secure_password",
  "profile": {
    "name": "New User",
    "bio": "User biography"
  }
}
```

**レスポンス**:
```json
{
  "success": true,
  "data": {
    "id": 2,
    "username": "newuser",
    "email": "newuser@example.com",
    "created_at": "2025-11-11T00:00:00Z"
  }
}
```

#### ユーザーを更新

```
PUT /users/{id}
```

**パラメータ**:
- `id` (required): ユーザーID

**リクエストボディ**:
```json
{
  "email": "updated@example.com",
  "profile": {
    "name": "Updated Name",
    "bio": "Updated biography"
  }
}
```

#### ユーザーを削除

```
DELETE /users/{id}
```

**パラメータ**:
- `id` (required): ユーザーID

**レスポンス**:
```json
{
  "success": true,
  "message": "User deleted successfully"
}
```

### データ操作

#### データ一覧を取得

```
GET /data
```

#### データ詳細を取得

```
GET /data/{id}
```

#### データを作成

```
POST /data
```

#### データを更新

```
PUT /data/{id}
```

#### データを削除

```
DELETE /data/{id}
```

## エラーハンドリング

### エラーレスポンス形式

すべてのエラーは以下の形式で返されます：

```json
{
  "success": false,
  "error": {
    "code": "ERROR_CODE",
    "message": "エラーメッセージ",
    "details": {
      "field": "追加の詳細情報"
    }
  }
}
```

### HTTPステータスコード

| コード | 説明 |
|--------|------|
| 200 | OK - リクエスト成功 |
| 201 | Created - リソース作成成功 |
| 204 | No Content - リクエスト成功（レスポンスボディなし）|
| 400 | Bad Request - リクエストが不正 |
| 401 | Unauthorized - 認証が必要 |
| 403 | Forbidden - アクセス権限なし |
| 404 | Not Found - リソースが見つからない |
| 422 | Unprocessable Entity - バリデーションエラー |
| 429 | Too Many Requests - レート制限超過 |
| 500 | Internal Server Error - サーバーエラー |
| 503 | Service Unavailable - サービス利用不可 |

### エラーコード

| コード | 説明 |
|--------|------|
| `INVALID_REQUEST` | リクエストが無効 |
| `UNAUTHORIZED` | 認証エラー |
| `FORBIDDEN` | アクセス権限エラー |
| `NOT_FOUND` | リソースが見つからない |
| `VALIDATION_ERROR` | バリデーションエラー |
| `RATE_LIMIT_EXCEEDED` | レート制限超過 |
| `INTERNAL_ERROR` | 内部エラー |

### エラーハンドリングの例

```javascript
try {
  const response = await fetch('https://api.example.com/v1/users/1', {
    headers: {
      'Authorization': `Bearer ${token}`
    }
  });

  if (!response.ok) {
    const error = await response.json();
    console.error(`Error ${error.error.code}: ${error.error.message}`);
    // エラー処理
  }

  const data = await response.json();
  // 成功時の処理
} catch (error) {
  console.error('Network error:', error);
}
```

## レート制限

APIへのリクエスト数には制限があります：

- **認証済みリクエスト**: 1時間あたり 1000 リクエスト
- **未認証リクエスト**: 1時間あたり 100 リクエスト

### レート制限ヘッダー

レスポンスには以下のヘッダーが含まれます：

```http
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 999
X-RateLimit-Reset: 1699564800
```

### レート制限超過時

レート制限を超えた場合、`429 Too Many Requests` エラーが返されます：

```json
{
  "success": false,
  "error": {
    "code": "RATE_LIMIT_EXCEEDED",
    "message": "Rate limit exceeded. Please try again later.",
    "details": {
      "retry_after": 3600
    }
  }
}
```

## SDK

公式SDKを使用すると、より簡単にAPIを利用できます。

### JavaScript/Node.js

```bash
npm install 250303app01-sdk
```

```javascript
const { Client } = require('250303app01-sdk');

const client = new Client({
  apiKey: 'your_api_key'
});

// ユーザー一覧を取得
const users = await client.users.list();

// ユーザーを作成
const newUser = await client.users.create({
  username: 'newuser',
  email: 'newuser@example.com'
});
```

### Python

```bash
pip install 250303app01-sdk
```

```python
from app01_sdk import Client

client = Client(api_key='your_api_key')

# ユーザー一覧を取得
users = client.users.list()

# ユーザーを作成
new_user = client.users.create(
    username='newuser',
    email='newuser@example.com'
)
```

## Webhook

特定のイベントが発生した際に、指定したURLに通知を送信できます。

### Webhookの設定

```
POST /webhooks
```

**リクエストボディ**:
```json
{
  "url": "https://your-domain.com/webhook",
  "events": ["user.created", "user.updated", "user.deleted"],
  "secret": "your_webhook_secret"
}
```

### Webhookペイロード

```json
{
  "event": "user.created",
  "timestamp": "2025-11-11T00:00:00Z",
  "data": {
    "id": 1,
    "username": "newuser",
    "email": "newuser@example.com"
  }
}
```

## ページネーション

リスト取得APIは、以下のパラメータでページネーションをサポートしています：

- `page`: ページ番号（デフォルト: 1）
- `limit`: 1ページあたりの件数（デフォルト: 10、最大: 100）

**例**:
```bash
GET /users?page=2&limit=20
```

## フィルタリング

多くのリスト取得APIは、フィルタリングをサポートしています：

```bash
GET /users?email=example.com&created_after=2025-01-01
```

## ソート

`sort` パラメータでソート順を指定できます：

```bash
GET /users?sort=created_at:desc
```

---

最終更新: 2025-11-11
