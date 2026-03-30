---
name: kenmin-eyecatch
description: 県民秘密スポットシリーズのアイキャッチ画像を自動生成。ナビゲーター+県シンボル+タイトルテキスト付き。
---

# 県民秘密スポット アイキャッチ画像生成

「県民しか知らない秘密スポット」シリーズのnote.com記事用アイキャッチ画像を生成します。

**完成イメージ:** ナビゲーター（おじいちゃん/おばあちゃん）と県のシンボルのイラスト背景に、「北海道編！けんちゃんの秘密スポット案内」のようなタイトルテキストがオーバーレイされた16:9画像。

## いつこのスキルを使うか

- 「アイキャッチを作って」「記事の画像を生成して」と言われた場合
- 「〇〇県のけんちゃんの画像」など、県名とナビゲーター名が含まれる場合
- 「note用の画像」「サムネイル」と言われた場合
- 県民秘密スポットシリーズの投稿準備をしている場合

## 手順

### ステップ1：情報を確認する

画像に必要な情報を確認します：
- 都道府県名
- ナビゲーターの名前・年齢・元職業
- タイトルテキスト（例：「北海道編！けんちゃんの秘密スポット案内」）
- 県のシンボル（山、海、名物など）

### ステップ2：KIE AI APIでベース画像を生成する

プロンプトは英語で。ナビゲーター + 県シンボルのイラスト。

```bash
KIE_API_KEY=$(grep KIE_AI_API_KEY C:\Users\yyama\AntigravityObsidianhado15000\.env | cut -d'=' -f2 | tr -d '\r')

curl -s -X POST "https://api.kie.ai/api/v1/jobs/createTask" \
  -H "Authorization: Bearer $KIE_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "google/nano-banana",
    "input": {
      "prompt": "（英語プロンプト）",
      "output_format": "png",
      "image_size": "16:9"
    }
  }'
```

プロンプトの構成要素：
- メインキャラ：ナビゲーターの外見・服装・表情・小道具
- 背景：県のシンボル（山、海、建物、名物）
- スタイル：Studio Ghibli風の温かい水彩画
- 構図：キャラ左寄り、右に背景が広がる。**上部に暗めのグラデーション帯を入れる**（タイトル文字が載る領域）
- テキスト不要（後でHTMLで合成する）

### ステップ3：タスク結果を取得して画像をダウンロード

```bash
# ポーリング（15秒待ってから）
sleep 15
curl -s "https://api.kie.ai/api/v1/jobs/recordInfo?taskId=TASK_ID" \
  -H "Authorization: Bearer $KIE_API_KEY"

# 画像ダウンロード
curl -o base-image.png "IMAGE_URL"
```

### ステップ4：HTMLテンプレートでタイトルを合成する

以下のHTMLファイルを生成して、agent-browserでスクリーンショットを撮ります。

```html
<!DOCTYPE html>
<html><head><meta charset="UTF-8">
<style>
  * { margin: 0; padding: 0; }
  body { width: 1200px; height: 675px; overflow: hidden; }
  .container {
    width: 1200px; height: 675px;
    background-image: url('base-image.png');
    background-size: cover;
    background-position: center;
    position: relative;
    font-family: 'Shippori Mincho B1', 'Yu Mincho', serif;
  }
  /* 上部グラデーション帯 */
  .title-area {
    position: absolute; top: 0; left: 0; right: 0;
    padding: 30px 40px 40px;
    background: linear-gradient(180deg, rgba(0,0,0,0.7) 0%, rgba(0,0,0,0.4) 70%, transparent 100%);
  }
  .series-label {
    font-size: 16px; color: #ffd700; letter-spacing: 0.2em;
    font-weight: 700; margin-bottom: 8px;
  }
  .main-title {
    font-size: 42px; color: white; font-weight: 800;
    line-height: 1.3; text-shadow: 2px 2px 8px rgba(0,0,0,0.8);
  }
  .main-title .prefecture {
    color: #ffd700; font-size: 48px;
  }
  /* 下部ナビゲーター情報 */
  .navigator-info {
    position: absolute; bottom: 0; left: 0; right: 0;
    padding: 30px 40px 20px;
    background: linear-gradient(0deg, rgba(0,0,0,0.6) 0%, transparent 100%);
  }
  .nav-name {
    font-size: 20px; color: white; font-weight: 700;
    text-shadow: 1px 1px 4px rgba(0,0,0,0.8);
  }
  .nav-desc {
    font-size: 14px; color: rgba(255,255,255,0.8);
    text-shadow: 1px 1px 4px rgba(0,0,0,0.8);
  }
</style>
<link href="https://fonts.googleapis.com/css2?family=Shippori+Mincho+B1:wght@700;800&display=swap" rel="stylesheet">
</head><body>
<div class="container">
  <div class="title-area">
    <div class="series-label">県民しか知らない秘密スポット</div>
    <div class="main-title">
      <span class="prefecture">北海道</span>編！<br>
      けんちゃんの秘密スポット案内
    </div>
  </div>
  <div class="navigator-info">
    <div class="nav-name">案内人：けんちゃん（72歳）</div>
    <div class="nav-desc">元漁師 ｜ 口癖「なまらいいっしょ」</div>
  </div>
</div>
</body></html>
```

### ステップ5：agent-browserでスクリーンショットを撮る

```bash
# HTMLをローカルで開いてスクリーンショット
agent-browser --allow-file-access open "file:///path/to/eyecatch-template.html"
agent-browser set viewport 1200 675
agent-browser wait 2000
agent-browser screenshot output.png
agent-browser close
```

### ステップ6：最終画像を保存する

```
保存先: C:\Users\yyama\skills_asaichi\marketing\kenmin\images\{番号}-{県名}-eyecatch.png
例: 01-hokkaido-eyecatch.png
```

## 重要な注意点

- ベース画像のプロンプトは英語で書く
- プロンプトに「dark gradient overlay at top for title text space」を含めると、テキストが載りやすい画像になる
- HTMLテンプレートの県名・ナビゲーター名は各県ごとに書き換える
- フォントはGoogle Fontsの「Shippori Mincho B1」を使用（和紙・朝市の雰囲気に合う）
- 最終画像は1200x675px（16:9）で、note.comのアイキャッチに最適
- agent-browserがインストール済みであること
