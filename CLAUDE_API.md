# CLAUDE.md — AIハッカソン ゲーム制作ルール

> このファイルはAI（Claude / Gemini / ChatGPT）がコードを生成する際に従うべきルールです。
> プロンプトの冒頭でこのファイルを読み込ませてください。

---

## 出力ルール

- HTML + JavaScript + CSS の**単一ファイル**で出力すること
- 外部ライブラリはCDNで読み込むこと（例: Three.js → `https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js`）
- PCブラウザで動作すればOK。モバイル対応は不要
- APIキーは `const API_KEY = "YOUR_KEY_HERE";` としてファイル冒頭に定義。コード中に直書きしない

---

## デプロイ
- githubアカウントを使って、vercelにデプロイ
- AuthorはCaptain-K <Captain-K@users.noreply.github.com>  

## 生成AI
### Gemini 
- モデル　gemini-2.5-flash

### Z.AI
- z.aiのglm5.1を使うようにして。使い方は以下。
- エンドポイントは、https://api.z.ai/api/coding/paas/v4
```
HTTP API Calls

Copy page

Z.AI provides standard HTTP API interfaces that support multiple programming languages and development environments, allowing you to easily integrate Z.AI’s powerful capabilities.
​
Core Advantages
Cross-platform Compatible
Supports all programming languages and platforms that support HTTP protocol
Standard Protocol
Based on RESTful design, follows HTTP standards, easy to understand and use
Flexible Integration
Can be integrated into any existing applications and systems
Real-time Calls
Supports synchronous and asynchronous calls to meet different scenario requirements
​
Get API Key
Access Z.AI Open Platform, Register or Login.
Create an API Key in the API Keys management page.
Copy your API Key for use.
​
API Basic Information
​
General API Endpoint
https://api.z.ai/api/paas/v4/
Note: When using the GLM Coding Plan, you need to configure the dedicated
Coding endpoint - https://api.z.ai/api/coding/paas/v4
instead of the general endpoint - https://api.z.ai/api/paas/v4
Note: The Coding API endpoint is only for Coding scenarios and is not applicable to general API scenarios. Please use them accordingly.
​
Request Header Requirements
Content-Type: application/json
Authorization: Bearer YOUR_API_KEY
​
Supported Authentication Methods
API Key Authentication
JWT Token Authentication
The simplest authentication method, directly using your API Key:
curl --location 'https://api.z.ai/api/paas/v4/chat/completions' \
--header 'Authorization: Bearer YOUR_API_KEY' \
--header 'Accept-Language: en-US,en' \
--header 'Content-Type: application/json' \
--data '{
    "model": "glm-5.1",
    "messages": [
        {
            "role": "user",
            "content": "Hello"
        }
    ]
}'
​
Basic Call Examples
​
Simple Conversation
curl --location 'https://api.z.ai/api/paas/v4/chat/completions' \
--header 'Authorization: Bearer YOUR_API_KEY' \
--header 'Accept-Language: en-US,en' \
--header 'Content-Type: application/json' \
--data '{
    "model": "glm-5.1",
    "messages": [
        {
            "role": "user",
            "content": "Please introduce the development history of artificial intelligence"
        }
    ],
    "temperature": 1.0,
    "max_tokens": 1024
}'
​
Streaming Response
curl --location 'https://api.z.ai/api/paas/v4/chat/completions' \
--header 'Authorization: Bearer YOUR_API_KEY' \
--header 'Accept-Language: en-US,en' \
--header 'Content-Type: application/json' \
--data '{
    "model": "glm-5.1",
    "messages": [
        {
            "role": "user",
            "content": "Write a poem about spring"
        }
    ],
    "stream": true
}'
​
Multi-turn Conversation
curl --location 'https://api.z.ai/api/paas/v4/chat/completions' \
--header 'Authorization: Bearer YOUR_API_KEY' \
--header 'Accept-Language: en-US,en' \
--header 'Content-Type: application/json' \
--data '{
    "model": "glm-5.1",
    "messages": [
        {
            "role": "system",
            "content": "You are a professional programming assistant"
        },
        {
            "role": "user",
            "content": "What is recursion?"
        },
        {
            "role": "assistant",
            "content": "Recursion is a programming technique where a function calls itself to solve problems..."
        },
        {
            "role": "user",
            "content": "Can you give me an example of Python recursion?"
        }
    ]
}'
​
Common Programming Language Examples
Python
JavaScript
Java
import requests
import json

def call_zai_api(messages, model="glm-5.1"):
    url = "https://api.z.ai/api/paas/v4/chat/completions"

    headers = {
        "Authorization": "Bearer YOUR_API_KEY",
        "Content-Type": "application/json",
        "Accept-Language": "en-US,en"
    }

    data = {
        "model": model,
        "messages": messages,
        "temperature": 1.0
    }

    response = requests.post(url, headers=headers, json=data)

    if response.status_code == 200:
        return response.json()
    else:
        raise Exception(f"API call failed: {response.status_code}, {response.text}")

# Usage example
messages = [
    {"role": "user", "content": "Hello, please introduce yourself"}
]

result = call_zai_api(messages)
print(result['choices'][0]['message']['content'])
```

## パラメーター定義ルール

ゲームバランスに影響する値は**すべてファイル冒頭の定数ブロックにまとめる**こと。コード中にマジックナンバーを残さない。

```javascript
// === GAME PARAMETERS ===
const PLAYER_SPEED = 3.0;      // プレイヤー移動速度（大きいほど速い）
const ENEMY_SPEED = 2.0;       // 敵の移動速度（大きいほど難しい）
const SPAWN_INTERVAL = 2000;   // 敵出現間隔ms（小さいほど難しい）
const GRAVITY = 0.5;           // 重力（大きいほどジャンプが短い）
const JUMP_FORCE = -12;        // ジャンプ力（絶対値が大きいほど高く飛ぶ）
const GAME_DURATION = 60;      // 制限時間（秒）
const SCORE_PER_ENEMY = 100;   // 敵1体あたりのスコア
// === END PARAMETERS ===
```

定数名は UPPER_SNAKE_CASE。各定数に「何を変えると何が起こるか」のコメントを必ず付ける。

---

## UI/UX ルール

- **タイトル画面**: タイトル名を大きく表示＋「スペースキーでスタート」の一行のみ。他の説明は不要
- **操作説明**: ゲーム画面の隅に小さく常時表示（例: `← → : 移動 / Space : ジャンプ`）。チュートリアル画面は作らない
- **ゲームオーバー画面**: 「GAME OVER」＋最終スコア＋「Rキーでリトライ」を表示
- **リトライ処理**: ゲーム状態を完全にリセットすること。以下を必ずチェック：
  - setInterval / setTimeout を前のゲームからすべて clear する
  - イベントリスナーの重複登録を防ぐ（再登録前に removeEventListener）
  - グローバル変数をすべて初期値に戻す
  - requestAnimationFrame のループを cancelAnimationFrame で停止してから再開する
- **スコア表示**: 画面上部に常に表示

---

## 効果音・BGM ルール（Web Audio API）

MP3ファイルは使わない。すべて AudioContext で合成音として生成する。

- 各効果音は `playXxxSound()` という関数として定義し、対応するイベント箇所で呼び出す
- BGM は OscillatorNode でループ再生。`startBGM()` / `stopBGM()` 関数として定義
- BGM の音量は効果音より小さく設定する
- ゲーム開始時に再生開始、ゲームオーバー時に停止

---

## フレームレート非依存処理

requestAnimationFrame のタイムスタンプベースで処理すること。固定フレームレート前提のコードは書かない。

```javascript
let lastTime = 0;
function gameLoop(timestamp) {
  const deltaTime = (timestamp - lastTime) / 1000;
  lastTime = timestamp;
  update(deltaTime);
  render();
  requestAnimationFrame(gameLoop);
}
```

---

## AIリアルタイム要素のルール

ゲーム中にAI APIを呼ぶ場合、以下を必ず守る：

1. **非同期処理**: API呼び出しはゲーム本体のメインループをブロックしない
2. **タイムアウト**: 5秒でタイムアウト。最大2回リトライ
3. **フォールバック必須**: API失敗時は事前定義した定型データ配列からランダムに選択して表示する。エラーメッセージは出さない（ユーザーにはAIが動いていないことを悟られないようにする）
4. **定型データは最低10パターン用意する**
5. **前のリクエストが完了する前に次のイベントが発生した場合**: 前のリクエストをキャンセルする（AbortController使用）

---

## 敵の動きのルール

- 単調な一方向移動にしない
- ジグザグ走行、フォーメーション移動など「意志」を感じさせる動きにする
- 難易度に「波」を作る：「簡単→少し難しい→簡単→もっと難しい」のリズム
- 難しい局面の後に「休める区間」を入れ、プレイヤーに達成感を与える

---

## パフォーマンス制約

- パーティクル数に上限を設ける（最大100個程度）
- 画面外に出たオブジェクトは即座に削除する
- DOM操作は最小限にし、Canvas または SVG で描画する

---

## 禁止事項

- localStorage / sessionStorage は使わない（変数で保持）
- alert() / confirm() / prompt() は使わない
- 外部サーバーへのデータ送信はしない（APIキー漏洩防止）
- console.log をプロダクションコードに残さない
