# 自分メモ

## 構築概要

1. USDCの支払いを要するサーバーを立てる
   1. リクエストだけ投げるサーバー
   2. 実際に支払いまで実行するサーバー（CDPのAPI利用）
   3. 実際に支払いまで実行するサーバー（直接Tx実行）
      1. EIP3009の実行
      2. 署名の検証処理
2. 上記サーバーにアクセスするWebアプリ
3. 上記サーバーを用いるためのMCPサーバー
4. 上記サーバーをAI Agentで実行する

## 0. 諸々のライブラリセットアップ

[/examples/typescript](../examples/typescript/)配下でpnpmのビルドまで行う

```bash
pnpm install
pnpm build
```

## 1. USDCの支払いを請求するサーバーの構築

### 1-1. リクエストだけ投げるサーバー

[/examples/typescript/servers/express/](../examples/typescript/servers/express/)にて、下記コマンドにより、サーバーを起動する

```bash
pnpm install
pnpm dev
```

[http://localhost:4021](http://localhost:4021)にアクセスする

USDCの支払の後、JSONリクエストが見える

### 1-2. 実際に支払いまで実行するサーバー（CDPのAPI利用）

#### ① サーバーの起動
[/examples/typescript/servers/advanced/](../examples/typescript/servers/advanced/)にて作業する

[.env-local](../examples/typescript/servers/advanced/.env-local)を編集する

```text
FACILITATOR_URL=https://x402.org/facilitator
NETWORK=base-sepolia
ADDRESS=0x11...
```

下記コマンドを実行して、サーバーを立ち上げる

```bash
pnpm install
pnpm dev
```

#### ②デモ用クライアントから実行

[/examples/typescript/clients/axios/](../examples/typescript/clients/axios/)にて作業する

[/examples/typescript/clients/axios/env-local](../examples/typescript/clients/axios/env-local)を元に、.envファイルを編集する

```text
RESOURCE_SERVER_URL=http://localhost:4021
ENDPOINT_PATH=/delayed-settlement
PRIVATE_KEY=0x111...
```

最後にクライアントの処理を実行するする

```bash
pnpm dev
```

#### （参考）出力

サーバー側出力

```bash
Payment settled: eyJzdWNjZXNzIjp0cnVlLCJ0cmFuc2FjdGlvbiI6IjB4N2U0ZTI4YTY5YzVlZTA5YmQzOTlhNjc4M2E1ODlmMmRmODBhOGZiMmY4OTExNDI3Yzg3NWQxYTI2ZTdjOWEyMCIsIm5ldHdvcmsiOiJiYXNlLXNlcG9saWEiLCJwYXllciI6IjB4N2I3MThENENlNmNhODM1MzY2NjBhMzE0NjM5NTU5RjNkM2Y2ZTllMyJ9
```

実際のトランザクション

[0x7e4e28a69c5ee09bd399a6783a589f2df80a8fb2f8911427c875d1a26e7c9a20](https://base-sepolia.blockscout.com/tx/0x7e4e28a69c5ee09bd399a6783a589f2df80a8fb2f8911427c875d1a26e7c9a20)

## 3. MCPサーバーでのデモ

Claude DesktopのMCP設定ファイルを編集する

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-filesystem",
        "/Users/username/Desktop",
        "/Users/username/Downloads"
      ]
    },
    "demo": {
      "command": "pnpm",
      "args": [
        "--silent",
        "-C",
        "{リポジトリまでのパス}/x402_demo/examples/typescript/mcp",
        "dev"
      ],
      "env": {
        "PRIVATE_KEY": "{秘密鍵の値}",
        "RESOURCE_SERVER_URL": "http://localhost:4021",
        "ENDPOINT_PATH": "/weather"
      }
    }
  }
}
```
