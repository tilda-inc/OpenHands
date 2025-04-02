# OpenHands API ドキュメント

このドキュメントでは、OpenHandsのAPIエンドポイントとWebSocket通信の実装について説明します。OpenHandsをAPIとして活用する方法を理解するためのガイドです。

## 目次

1. [概要](#概要)
2. [APIエンドポイント](#apiエンドポイント)
   - [会話管理](#会話管理)
   - [ファイル操作](#ファイル操作)
   - [設定管理](#設定管理)
   - [その他のエンドポイント](#その他のエンドポイント)
3. [WebSocket通信](#websocket通信)
   - [接続方法](#接続方法)
   - [イベント処理](#イベント処理)
   - [メッセージ送信](#メッセージ送信)
4. [認証](#認証)
5. [エラーハンドリング](#エラーハンドリング)
6. [使用例](#使用例)

## 概要

OpenHandsは、AIを活用したソフトウェア開発エージェントプラットフォームです。RESTful APIとWebSocketを通じて、コード生成、コマンド実行、ファイル操作などの機能を提供します。このドキュメントでは、これらのAPIエンドポイントとWebSocket通信の詳細を説明します。

## APIエンドポイント

OpenHandsのAPIは、FastAPIを使用して実装されています。以下に主要なエンドポイントを示します。

### 会話管理

会話はOpenHandsの中心的な概念で、ユーザーとAIエージェント間のインタラクションを表します。

#### 会話の作成

```
POST /api/conversations
```

**リクエストボディ**:
```json
{
  "selected_repository": "string | null",
  "selected_branch": "string | null",
  "initial_user_msg": "string | null",
  "image_urls": ["string"] | null,
  "replay_json": "string | null"
}
```

**レスポンス**:
```json
{
  "status": "ok",
  "conversation_id": "string"
}
```

#### 会話の取得

```
GET /api/conversations/{conversation_id}
```

**レスポンス**:
```json
{
  "conversation_id": "string",
  "title": "string",
  "last_updated_at": "string",
  "created_at": "string",
  "selected_repository": "string | null",
  "status": "RUNNING | STOPPED"
}
```

#### 会話一覧の取得

```
GET /api/conversations
```

**クエリパラメータ**:
- `page_id`: ページングのためのID（オプション）
- `limit`: 1ページあたりの結果数（デフォルト: 20）

**レスポンス**:
```json
{
  "results": [
    {
      "conversation_id": "string",
      "title": "string",
      "last_updated_at": "string",
      "created_at": "string",
      "selected_repository": "string | null",
      "status": "RUNNING | STOPPED"
    }
  ],
  "next_page_id": "string | null"
}
```

#### 会話の更新

```
PATCH /api/conversations/{conversation_id}
```

**リクエストボディ**:
```json
{
  "title": "string"
}
```

**レスポンス**: `true` または `false`

#### 会話の削除

```
DELETE /api/conversations/{conversation_id}
```

**レスポンス**: `true` または `false`

### ファイル操作

OpenHandsは、エージェントのワークスペース内のファイルを操作するためのAPIを提供します。

#### ファイル一覧の取得

```
GET /api/conversations/{conversation_id}/list-files
```

**クエリパラメータ**:
- `path`: ファイルを一覧表示するパス（オプション）

**レスポンス**: ファイルパスの配列

#### ファイルの取得

```
GET /api/conversations/{conversation_id}/select-file
```

**クエリパラメータ**:
- `file`: 取得するファイルのパス

**レスポンス**:
```json
{
  "code": "string"
}
```

#### ファイルの保存

```
POST /api/conversations/{conversation_id}/save-file
```

**リクエストボディ**:
```json
{
  "filePath": "string",
  "content": "string"
}
```

**レスポンス**:
```json
{
  "message": "string"
}
```

#### ファイルのアップロード

```
POST /api/conversations/{conversation_id}/upload-files
```

**リクエストボディ**: `multipart/form-data` 形式のファイル

**レスポンス**:
```json
{
  "message": "string"
}
```

#### ワークスペースのZIP取得

```
GET /api/conversations/{conversation_id}/zip-directory
```

**レスポンス**: ZIPファイルのバイナリデータ

### 設定管理

ユーザー設定を管理するためのAPIエンドポイントです。

#### 設定の取得

```
GET /api/settings
```

**レスポンス**:
```json
{
  "llm_model": "string",
  "llm_base_url": "string",
  "agent": "string",
  "language": "string",
  "llm_api_key": "string | null",
  "confirmation_mode": "boolean",
  "security_analyzer": "string",
  "remote_runtime_resource_factor": "number | null"
}
```

#### 設定の保存

```
POST /api/settings
```

**リクエストボディ**:
```json
{
  "llm_model": "string",
  "llm_base_url": "string",
  "agent": "string",
  "language": "string",
  "llm_api_key": "string | null",
  "confirmation_mode": "boolean",
  "security_analyzer": "string",
  "remote_runtime_resource_factor": "number | null",
  "provider_tokens": {
    "provider": "string"
  },
  "user_consents_to_analytics": "boolean | null"
}
```

**レスポンス**: 成功時は200ステータスコード

#### 設定のリセット

```
POST /api/reset-settings
```

**レスポンス**: 成功時は200ステータスコード

### その他のエンドポイント

#### 利用可能なモデルの取得

```
GET /api/options/models
```

**レスポンス**: モデル名の配列

#### 利用可能なエージェントの取得

```
GET /api/options/agents
```

**レスポンス**: エージェント名の配列

#### 設定情報の取得

```
GET /api/options/config
```

**レスポンス**: 設定情報のオブジェクト

#### VSCodeのURL取得

```
GET /api/conversations/{conversation_id}/vscode-url
```

**レスポンス**:
```json
{
  "vscode_url": "string"
}
```

#### ランタイムIDの取得

```
GET /api/conversations/{conversation_id}/config
```

**レスポンス**:
```json
{
  "runtime_id": "string",
  "session_id": "string"
}
```

## WebSocket通信

OpenHandsは、Socket.IOを使用してリアルタイム通信を実装しています。WebSocketを通じて、エージェントの実行状態やメッセージをリアルタイムで受信できます。

### 接続方法

WebSocketに接続するには、以下のURLを使用します：

```
ws://{host}/socket.io
```

接続時には、以下のクエリパラメータを指定する必要があります：

- `conversation_id`: 接続する会話のID
- `latest_event_id`: 最後に受信したイベントのID（初回接続時は-1）

フロントエンドでの接続例：

```javascript
const socket = io(baseUrl, {
  transports: ["websocket"],
  query: {
    latest_event_id: lastEventId ?? -1,
    conversation_id: conversationId,
  },
});
```

### イベント処理

WebSocket接続後、以下のイベントをリッスンできます：

#### `oh_event`

エージェントからのイベント（アクション、観察結果など）を受信します。

```javascript
socket.on("oh_event", (event) => {
  // イベントの処理
});
```

受信するイベントの例：

```json
{
  "id": "string",
  "source": "agent | user",
  "type": "message | action | observation",
  "content": "string",
  "timestamp": "string"
}
```

#### `connect`

接続成功時に発火するイベント。

```javascript
socket.on("connect", () => {
  // 接続成功時の処理
});
```

#### `connect_error`

接続エラー時に発火するイベント。

```javascript
socket.on("connect_error", (error) => {
  // エラー処理
});
```

#### `disconnect`

切断時に発火するイベント。

```javascript
socket.on("disconnect", (reason) => {
  // 切断時の処理
});
```

### メッセージ送信

ユーザーからエージェントにメッセージを送信するには、`oh_user_action`イベントを使用します：

```javascript
socket.emit("oh_user_action", {
  type: "message",
  content: "こんにちは、OpenHands！",
  image_urls: [] // オプション
});
```

## 認証

OpenHandsは、アプリケーションモードに応じて異なる認証方法を提供します：

- `oss`モード: 認証なし
- `cloud`モード: 基本認証
- `saas`モード: GitHub OAuth認証

認証状態を確認するには：

```
POST /api/authenticate
```

**レスポンス**: 認証成功時は200ステータスコード

## エラーハンドリング

APIエラーは、適切なHTTPステータスコードとエラーメッセージを含むJSONレスポンスで返されます：

```json
{
  "status": "error",
  "message": "エラーメッセージ",
  "msg_id": "エラーID"
}
```

WebSocketエラーは、`connect_error`イベントで通知されます。

## 使用例

### 会話の作成と接続

```javascript
// 1. 会話の作成
const createConversation = async (initialMessage) => {
  const response = await fetch("/api/conversations", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({
      initial_user_msg: initialMessage
    })
  });
  const data = await response.json();
  return data.conversation_id;
};

// 2. WebSocketへの接続
const connectToWebSocket = (conversationId) => {
  const socket = io(window.location.host, {
    transports: ["websocket"],
    query: {
      latest_event_id: -1,
      conversation_id: conversationId
    }
  });

  // イベントリスナーの設定
  socket.on("connect", () => {
    console.log("WebSocket接続成功");
  });

  socket.on("oh_event", (event) => {
    console.log("受信イベント:", event);
    // UIの更新など
  });

  socket.on("connect_error", (error) => {
    console.error("接続エラー:", error);
  });

  return socket;
};

// 3. メッセージの送信
const sendMessage = (socket, message) => {
  socket.emit("oh_user_action", {
    type: "message",
    content: message
  });
};

// 使用例
(async () => {
  const conversationId = await createConversation("こんにちは");
  const socket = connectToWebSocket(conversationId);
  
  // 数秒後にメッセージを送信
  setTimeout(() => {
    sendMessage(socket, "新しいプロジェクトを作成してください");
  }, 3000);
})();
```

このドキュメントは、OpenHandsのAPIとWebSocket通信の基本的な使用方法を説明しています。より詳細な情報については、OpenHandsの公式ドキュメントを参照してください。
