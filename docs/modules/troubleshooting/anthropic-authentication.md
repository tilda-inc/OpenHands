---
sidebar_position: 2
title: Anthropic API認証エラーの解決
---

# Anthropic API認証エラーの解決

OpenHandsでAnthropicのAPIを使用する際に、以下のようなエラーが表示されることがあります：

```
Error authenticating with the LLM provider. Please check your API key
AuthenticationError: litellm.AuthenticationError: AnthropicException - {"type":"error","error":{"type":"authentication_error","message":"invalid x-api-key"}}
```

このエラーは、Anthropic APIの認証に問題があることを示しています。以下の手順で問題を解決できます。

## 考えられる原因

1. **APIキーが設定されていない**：APIキーが未設定または空白のままになっている
2. **APIキーが無効**：APIキーが間違っている、期限切れ、または取り消されている
3. **APIキーの形式が正しくない**：APIキーの形式が正しくないか、余分な文字（スペースや改行など）が含まれている
4. **モデル名の指定が正しくない**：AnthropicモデルのプレフィックスやバージョンIDが正しく指定されていない

## 解決方法

### 1. APIキーの取得と確認

1. [Anthropicのウェブサイト](https://www.anthropic.com/api)からAPIキーを取得します
2. APIキーが正しいフォーマット（通常は`sk-ant-`で始まる）であることを確認します
3. コピー時に余分なスペースや改行が含まれていないことを確認します

### 2. APIキーの設定（Docker使用時）

Docker実行時に環境変数を使ってAPIキーを設定する場合：

```bash
docker run -it --rm --pull=always \
    -e SANDBOX_RUNTIME_CONTAINER_IMAGE=docker.all-hands.dev/all-hands-ai/runtime:0.30-nikolaik \
    -e LOG_ALL_EVENTS=true \
    -e LLM_API_KEY=YOUR_ANTHROPIC_API_KEY \
    -e LLM_MODEL=anthropic/claude-3-5-sonnet-20241022 \
    -v /var/run/docker.sock:/var/run/docker.sock \
    -v ~/.openhands-state:/.openhands-state \
    -p 3000:3000 \
    --add-host host.docker.internal:host-gateway \
    --name openhands-app \
    docker.all-hands.dev/all-hands-ai/openhands:0.30
```

`YOUR_ANTHROPIC_API_KEY`をあなたの実際のAnthropicのAPIキーに置き換えてください。

### 3. Webインターフェースでの設定

OpenHandsのWebインターフェース（http://localhost:3000）から設定する場合：

1. 「Settings」または「設定」ページにアクセスします
2. 「LLM Provider」または「LLMプロバイダー」セクションでAnthropicを選択します
3. APIキーフィールドに有効なAnthropicのAPIキーを入力します
4. モデル名が正しいことを確認します（例：`anthropic/claude-3-5-sonnet-20241022`）
5. 設定を保存します

### 4. 設定ファイルでの設定

設定ファイル（config.toml）を使用して設定する場合：

```toml
[llm]
model = "anthropic/claude-3-5-sonnet-20241022"
api_key = "YOUR_ANTHROPIC_API_KEY"
```

`YOUR_ANTHROPIC_API_KEY`をあなたの実際のAnthropicのAPIキーに置き換えてください。

## トラブルシューティング

- APIキーを更新した後もエラーが続く場合は、OpenHandsを再起動してください
- 異なるモデルで試してみてください（例：`anthropic/claude-3-haiku-20240307`）
- APIキーの権限が適切に設定されていることを確認してください
- Anthropicのステータスページで、サービスが正常に動作していることを確認してください

## 参考情報

- [Anthropic APIドキュメント](https://docs.anthropic.com/claude/reference/authentication)
- [OpenHands LLMの設定](https://docs.all-hands.dev/modules/usage/llms)
