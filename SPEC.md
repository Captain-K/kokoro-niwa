# SPEC.md — 3D枯山水エディタ「心庭（こころにわ）」詳細仕様

## プロダクトコンセプト

### 一言で
**AIと一緒に3Dの枯山水を作り、審査員と記念撮影できる禅体験アプリ**

### 解決する課題
忙しい現代人が「静けさ」と「自己表現」を同時に楽しめる場所がない。本物の枯山水を作るのは大変だが、3Dなら誰でも数分で「自分だけの庭」を作れる。しかもAIが庭の意味を読み解いてくれるので、自己対話にもなる。

### ターゲット
- 瞑想や禅に興味がある全年齢
- クリエイティブな自己表現をしたい人
- SNSでシェアしたいネタを探している人
- **今回の最重要ターゲット：審査員の福岡俊弘さん**

## 画面構成

### メイン画面（1画面完結）

```
┌────────────────────────────────────────────┐
│  心庭 — こころにわ               [?] [共有]  │  ← ヘッダー
├────────────────────────────────────────────┤
│                                              │
│                                              │
│         [3Dビューポート]                     │
│           砂地 + 岩 + 砂紋                    │
│                                              │
│                                              │
├────────────────────────────────────────────┤
│ 🪨岩 🌀砂紋 🌿苔 | [AI解説] [福岡さん召喚] [📸] │  ← ツールバー
├────────────────────────────────────────────┤
│ 老師：「この庭は静寂の中に力強さを秘めて…」    │  ← AI吹き出し
└────────────────────────────────────────────┘
```

## 機能仕様

### 1. 3Dシーン

#### 1.1 基本セットアップ
- **レンダラー**: `THREE.WebGLRenderer({ antialias: true })`
- **背景色**: `#e8e4d8`（和紙色）またはグラデーション（上=空色、下=砂色）
- **影**: `renderer.shadowMap.enabled = true` / `PCFSoftShadowMap`
- **カメラ**: `PerspectiveCamera(50, aspect, 0.1, 100)`、初期位置`(5, 5, 5)`
- **ライト**:
  - `DirectionalLight` (白, 強度1.0, 位置(5,10,5)) — 太陽光
  - `AmbientLight` (白, 強度0.4) — 環境光
- **コントロール**: `OrbitControls`（Three.js examples から読み込み）
  - minDistance: 3、maxDistance: 15
  - maxPolarAngle: 1.5（地面より下に行かせない）

#### 1.2 砂地
- **形状**: `PlaneGeometry(8, 8, 128, 128)` （砂紋を描くため頂点を多めに）
- **マテリアル**: `MeshStandardMaterial`
  - テクスチャ: `sand_texture.jpg`（なければ色`#ede4cf`の単色）
  - repeat: (4, 4)
  - roughness: 0.9
- **受け影**: `receiveShadow = true`
- 囲い：四方に`BoxGeometry(8, 0.2, 0.3)`で木の枠を配置（濃い茶色 `#3e2723`）

#### 1.3 岩
- **形状**: `IcosahedronGeometry(random(0.3, 0.6), 0)` で不規則な多面体
  - もしくはglbモデルがあればそれを使用
- **マテリアル**: `MeshStandardMaterial`
  - color: `#6b6760`（濃灰）〜 `#8a7f72`（灰褐色）のランダム
  - roughness: 0.9
  - flatShading: true
- **影**: `castShadow = true`, `receiveShadow = true`
- **操作**:
  - 砂地クリックで**岩ツール選択中なら**配置
  - 岩自体クリックで選択→右クリックで削除 or 選択状態+Deleteで削除
  - ドラッグで位置調整（オプション、時間あれば）
- **ランダム回転**: 配置時にY軸ランダム回転、スケールもランダム(0.7〜1.3)

#### 1.4 砂紋
- **実装方式**: 砂地のPlaneの頂点をリアルタイムで凹ませる
  - マウスドラッグで通った線に沿って、頂点のY座標を少し下げる（-0.05程度）
  - 周囲の頂点も緩やかに下げる（ガウス分布的に）
- **岩の周りの同心円波紋**: 岩配置時に自動で同心円を2〜3周描く
  - 岩の周辺頂点を距離に応じてsin波で凹凸
- **Normal再計算**: 頂点変更後に `geometry.computeVertexNormals()` 呼ぶ

**時間がなければ代替案**: 砂紋は線のオブジェクト（TubeGeometry）を地面に貼るだけでもOK

#### 1.5 苔（オプション）
- **形状**: 小さなSphereGeometryを複数束ねる or 緑のPlaneを斜めに配置
- **マテリアル**: `MeshStandardMaterial`、color `#4a6b3a`、roughness 0.95
- **配置**: クリックで配置、岩の根元に吸着

### 2. UIコントロール

#### 2.1 ツールバー（下部固定）
| ボタン | 動作 |
|--------|------|
| 🪨 岩 | 岩配置モードON |
| 🌀 砂紋 | 砂紋描画モードON |
| 🌿 苔 | 苔配置モードON |
| 👆 選択 | 選択/削除モード |
| 🔄 リセット | 確認ダイアログ→全削除 |
| 🧙 AI解説 | 現在の庭をAIに解説させる |
| 👤 福岡さん召喚 | アバター登場（後述） |
| 📸 撮影 | スクリーンショット保存 |

**現在選択中のツール**はハイライト表示（背景色変更）

#### 2.2 AI吹き出し（下部）
- 老師アイコン（小さい水墨画風のキャラ or 絵文字🧙）
- テキストは**タイプライター風に1文字ずつ表示**（世界観演出）
- 表示時間: 自動消去なし、次の解説まで残る

#### 2.3 ヘッダー
- タイトル「心庭」（明朝体 or セリフ系フォント）
- サブタイトル小さく「こころにわ」
- 右端にヘルプ(?)と共有ボタン

### 3. AI統合

#### 3.1 庭の解説機能

```javascript
async function explainGarden(sceneState) {
  const prompt = `
あなたは禅寺の老師です。以下の枯山水の配置を見て、2〜3文で庭の意味を詩的に解説してください。
専門用語を使いすぎず、でも格調高い日本語で。優しく、でも含蓄のある口調で。

岩の数: ${sceneState.rocks.length}個
岩の配置（x, z座標、サイズ）:
${sceneState.rocks.map(r => `  (${r.x.toFixed(1)}, ${r.z.toFixed(1)}) サイズ${r.size.toFixed(1)}`).join('\n')}
砂紋の本数: ${sceneState.sandPatterns}本
苔の数: ${sceneState.moss.length}個

2〜3文のみで返答してください。「」などの括弧は不要。
`;
  
  try {
    const response = await fetch(`https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent?key=${API_KEY}`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        contents: [{ parts: [{ text: prompt }] }],
        generationConfig: { temperature: 0.9, maxOutputTokens: 200 }
      })
    });
    const data = await response.json();
    return data.candidates[0].content.parts[0].text;
  } catch (e) {
    // フォールバック：AIが落ちたときの固定メッセージ
    return getFallbackExplanation(sceneState);
  }
}

function getFallbackExplanation(sceneState) {
  const templates = [
    "静寂の中に、確かな存在感がございますな。",
    "この配置、何か語りたげに見えます。",
    "庭は心を映す鏡。見事でございます。",
  ];
  return templates[Math.floor(Math.random() * templates.length)];
}
```

**重要**: フォールバックは必ず用意。AI落ちても体験が壊れないように。

#### 3.2 庭の命名機能

AI解説の後に「この庭に名前を」ボタン → 詩的な名前を生成。
例：「静寂五岳庭」「月影苔庭」「無心波紋庭」

### 4. 福岡さんアバター召喚（⭐️このアプリの目玉）

#### 4.1 事前準備
- **理想**: 事前に福岡さんの3Dアバター（.glb形式）を作っておく
  - [Ready Player Me](https://readyplayer.me/)で写真から生成
  - または[Mixamo](https://www.mixamo.com/)のサンプルキャラに和装させる
- **妥協**: シンプルな人型（CylinderGeometry+SphereGeometryで頭）+ 名札「福岡」

#### 4.2 召喚の演出
1. ボタン押下
2. 庭の端（x=4あたり）に煙エフェクト（白いパーティクル）
3. 煙が晴れるとアバターが立っている
4. アバターが庭の中心に向かって歩く（`position.lerp`で補間）
5. 中心で一礼モーション（`rotation.x`を-0.3→0にアニメ）
6. AIが「ほう、見事な庭でございますな」と発言
7. アバターの上に💬吹き出しが出る

#### 4.3 移動の実装
```javascript
function summonFukuoka() {
  avatar.position.set(4, 0, 4);
  avatar.visible = true;
  // 煙エフェクト
  spawnSmokeEffect(4, 0, 4);
  
  // 中心に歩く
  const target = new THREE.Vector3(0, 0, 2);
  const duration = 2000; // 2秒
  const start = avatar.position.clone();
  const startTime = Date.now();
  
  function walk() {
    const elapsed = Date.now() - startTime;
    const t = Math.min(elapsed / duration, 1);
    avatar.position.lerpVectors(start, target, t);
    if (t < 1) requestAnimationFrame(walk);
    else bowAnimation();
  }
  walk();
}
```

### 5. 記念撮影機能（⭐️もう一つの目玉）

#### 5.1 撮影モード
- 「📸」ボタンを押すとカウントダウン「3, 2, 1」
- シャッター音「カシャッ」
- 画面全体が一瞬白くフラッシュ
- `renderer.domElement.toBlob()`で画像を取得
- `<a download="心庭-${date}.png">`でDL

#### 5.2 撮影時の演出
- ツールバーを一時的に非表示
- 「心庭 〜こころにわ〜」のロゴと日付を画像右下に合成
- 庭の名前（AIが付けたもの）も入れる

```javascript
function takeScreenshot() {
  // UIを隠す
  document.querySelector('.toolbar').style.display = 'none';
  document.querySelector('.header').style.display = 'none';
  
  // 少し待ってレンダリング
  requestAnimationFrame(() => {
    renderer.render(scene, camera);
    renderer.domElement.toBlob(blob => {
      // ロゴ合成（Canvas使用）
      const canvas = document.createElement('canvas');
      canvas.width = renderer.domElement.width;
      canvas.height = renderer.domElement.height;
      const ctx = canvas.getContext('2d');
      
      const img = new Image();
      img.onload = () => {
        ctx.drawImage(img, 0, 0);
        // ロゴテキスト
        ctx.font = '24px serif';
        ctx.fillStyle = 'rgba(0,0,0,0.7)';
        ctx.fillText(`${gardenName}  心庭 ${formatDate()}`, 
                     canvas.width - 400, canvas.height - 30);
        
        canvas.toBlob(finalBlob => {
          const url = URL.createObjectURL(finalBlob);
          const a = document.createElement('a');
          a.href = url;
          a.download = `心庭-${gardenName}-${Date.now()}.png`;
          a.click();
        });
      };
      img.src = URL.createObjectURL(blob);
      
      // UIを戻す
      document.querySelector('.toolbar').style.display = '';
      document.querySelector('.header').style.display = '';
    });
  });
}
```

### 6. 効果音

| 音 | トリガー | ファイル |
|-----|---------|---------|
| 鹿威し「コーン」 | ランダム30秒ごと | shishi_odoshi.mp3 |
| 砂を引く「サッ」 | 砂紋描画中 | sand_rake.mp3 |
| 石「コツン」 | 岩配置時 | rock_place.mp3 |
| 鐘「チーン」 | AI解説完了時 | bell.mp3 |
| シャッター「カシャッ」 | 撮影時 | camera.mp3 |

**実装**: Web Audio APIで`Audio(url).play()`で十分。プリロード必須。

### 7. BGM（オプション）
- ループ再生: 静かな琴の音 or 尺八
- 音量は控えめ（0.3程度）
- ON/OFF切り替えボタン

## デザイン仕様

### カラーパレット（サイト全体）
| 用途 | カラー |
|------|--------|
| 背景 | `#f5f0e8`（和紙色） |
| 砂地 | `#ede4cf` |
| 岩 | `#6b6760` 〜 `#8a7f72` |
| 苔 | `#4a6b3a` |
| 木枠 | `#3e2723` |
| テキスト（主） | `#2c2c2c` |
| テキスト（副） | `#7a6a5a` |
| アクセント（朱） | `#a23d2a` |

### フォント
- **タイトル・見出し**: "Noto Serif JP" (Weight: 600-700)
- **本文**: "Noto Sans JP" (Weight: 400)
- **AI発言**: "Noto Serif JP" (Weight: 400, 斜体)

Google Fontsから読み込み：
```html
<link href="https://fonts.googleapis.com/css2?family=Noto+Serif+JP:wght@400;600;700&family=Noto+Sans+JP:wght@400;500&display=swap" rel="stylesheet">
```

## 実装スケジュール（180分）

### Phase 1: MVP（0:00 - 0:60）
- [ ] HTML/CSSスケルトン（10分）
- [ ] Three.jsシーン初期化・砂地表示（15分）
- [ ] OrbitControls追加・カメラ動作確認（5分）
- [ ] 岩配置機能（マウスクリックで岩）（15分）
- [ ] Gemini API接続・解説文表示（15分）

**60分時点のゴール**: ブラウザで3D砂地に岩を置けて、AIが解説する状態。GitHub Pagesで公開済み。

### Phase 2: 選定（0:60 - 0:75）
黒木・大貫の2バージョンを比べて、良い方（またはマージ）を選ぶ。

### Phase 3: ブラッシュアップ（0:75 - 2:45）

**黒木**（3D・素材）:
- [ ] 砂紋描画実装（30分）
- [ ] 福岡さんアバター準備・召喚演出（40分）
- [ ] テクスチャ適用・ライティング調整（20分）

**大貫**（ロジック・演出）:
- [ ] 効果音実装（20分）
- [ ] 撮影モード実装（30分）
- [ ] AI解説のプロンプト改善（15分）
- [ ] 庭命名機能追加（15分）
- [ ] タイプライター演出（10分）

### Phase 4: テスト・仕上げ（2:45 - 3:00）
- [ ] 共有リンク動作確認
- [ ] プレゼン準備
- [ ] 動画バックアップ取得

## 3分プレゼン台本

### 1. 課題提起（30秒）
「皆さん、最後に『無』になったのはいつですか？
 毎日のスマホ、通知、SNS…  情報に溺れる現代人に、必要なのは**自分だけの静かな場所**ではないでしょうか」

### 2. 解決策の提示（30秒）
「私たちは**心庭（こころにわ）**を作りました。
 3D空間に、あなただけの枯山水を。
 石を置くたび、AIの老師があなたの心を読み解きます」

### 3. デモ（90秒）— 最大のハイライト
- **[審査員の福岡さんに直接操作してもらう]**
- 「福岡さん、お好きな場所に岩を置いてみてください」
- 審査員が岩配置 → AIが即座に解説
- 「砂紋も引いてみましょう」→ 波紋が広がる
- 「完成しました！では、この庭に **福岡さんご本人**を召喚します！」
- 👤ボタン → 福岡さんアバター登場 → 煙→歩き→一礼
- 会場ざわつく
- 「では記念撮影を！」📸ボタン → シャッター音 → 写真DL
- 「**この写真、ぜひTwitterで #心庭 をつけてシェアしてください！**」

### 4. 技術的工夫（20秒）
「Three.jsの3D空間に、GeminiのマルチモーダルAIを統合。
 岩の座標をAIに渡し、詩的な解釈を生成させています。
 **触って驚き、見て癒され、シェアして広がる**。
 3層の体験設計が差別化ポイントです」

### 5. まとめ（10秒）
「心に庭を、手のひらに禅を。**心庭** — こころにわ、でした」

## リスク管理

| リスク | 対策 |
|--------|------|
| Gemini APIがCORSで弾かれる | 公式エンドポイント使用、APIキーはURL param方式 |
| Gemini APIが落ちる | フォールバック文を10パターン用意 |
| 3Dモデルが重い | プリミティブで代替、テクスチャ軽量化 |
| 福岡さんアバターが間に合わない | 汎用人型+和装テクスチャで代用、名札「審査員」 |
| 当日ネット障害 | 動画録画済みバックアップ |
| OrbitControlsが動かない | 手動でmouse eventを書く準備 |
| 共有リンクと開発環境で挙動違う | GitHub Pagesで30分ごとに確認 |

## 審査基準への対応

| 審査項目 | このアプリの強み |
|----------|-----------------|
| 技術的な完成度 | 3D + AI + リアルタイム描画の統合、Three.js r128で安定 |
| アイデアの独自性 | 「3D枯山水エディタ×審査員アバター」は他にない |
| 実用性・社会的インパクト | 瞑想アプリ市場に刺さる、SNS拡散性高い |
| プレゼンテーションの質 | 審査員巻き込み型デモ、記念撮影でシェア誘発 |

## 福岡さんの「ツボ」への対応

| 福岡さんのツボ | 対応策 |
|---------------|-------|
| ①一般人がパッと見て面白い | 3Dの直感操作、SNSシェア映え |
| ②テック×ポップカルチャー | 伝統文化（枯山水）×最新AI |
| ③触って驚く体験 | 岩を置く→即AI解説、アバター召喚 |
| ④ストーリー・世界観 | 和紙色UI、老師キャラ、庭に名前 |

## 成功のKPI（内部評価用）

- MVPが60分以内に完成する
- 3分プレゼン中、審査員が実際に操作する
- 記念撮影の写真が生成される
- 福岡さんが「おっ」と言う
- SNSで#心庭のハッシュタグが1つでも使われる

---

以上、Claude Codeはこの仕様書とCLAUDE.mdを元に実装してください。
**動くものが最優先、完璧は不要**。
