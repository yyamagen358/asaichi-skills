---
name: asaichi-ux-test
description: スキル朝市のHTML全要素を自動クリック・スクショ撮影し、UX問題点と改善提案をレポートする
---

# スキル朝市 UXテスト

スキル朝市（skill_asaichi.html）の全ボタン・リンク・タブ・フォームを自動でクリックし、デスクトップとモバイル両方のスクリーンショットを撮影。UXの問題点を検出して改善提案レポートを出力します。

## いつこのスキルを使うか

以下のような場合にこのスキルを使ってください：

- 「朝市のUXテストして」「朝市をチェックして」と言われた場合
- 「朝市の表示を確認して」「朝市のスクショ撮って」と言われた場合
- 「朝市のモバイル表示を確認して」と言われた場合
- 朝市HTMLに変更を加えた後の動作確認
- 定期的なUX品質チェック

## 手順

### ステップ1：朝市HTMLをブラウザで開く

agent-browserを使ってローカルのHTMLファイルを開きます。テスト結果の保存先ディレクトリも作成します。

```bash
mkdir -p C:/Users/yyama/skills_asaichi/ux-test
agent-browser --allow-file-access open "file:///C:/Users/yyama/skills_asaichi/skill_asaichi.html"
agent-browser wait 3000
agent-browser screenshot C:/Users/yyama/skills_asaichi/ux-test/01-initial.png
```

### ステップ2：デスクトップ表示で全要素テスト

以下の順序でテストし、各ステップでスクリーンショットを撮影します。

**2a) タブ切替テスト（梅→竹→松→すべて）：**
```bash
agent-browser snapshot -i
# タブボタンの ref を使ってクリック
agent-browser click @e7   # 梅タブ
agent-browser wait 500
agent-browser screenshot C:/Users/yyama/skills_asaichi/ux-test/02-tab-ume.png
agent-browser click @e8   # 竹タブ
agent-browser wait 500
agent-browser screenshot C:/Users/yyama/skills_asaichi/ux-test/03-tab-take.png
agent-browser click @e9   # 松タブ
agent-browser wait 500
agent-browser screenshot C:/Users/yyama/skills_asaichi/ux-test/04-tab-matsu.png
agent-browser click @e6   # すべてタブ
agent-browser wait 500
```

チェック項目：
- [ ] 各タブクリックでカード表示が切り替わるか
- [ ] アクティブタブのスタイルが変わるか
- [ ] カード数がタブに表示されている数と一致するか

**2b) 検索テスト：**
```bash
agent-browser fill @e4 "俳句"
agent-browser wait 500
agent-browser screenshot C:/Users/yyama/skills_asaichi/ux-test/05-search.png
agent-browser fill @e4 ""
agent-browser wait 300
```

チェック項目：
- [ ] 検索でカードがフィルタされるか
- [ ] 検索クリアで全カード復帰するか

**2c) スキルカード → モーダル表示テスト：**
各スキルカードのモーダルを順番に開いて確認します。
```bash
# JS evalでモーダルを開く（カードのonclickはopenModal(id)）
agent-browser eval 'openModal(1)'
agent-browser wait 500
agent-browser screenshot C:/Users/yyama/skills_asaichi/ux-test/06-modal-skill1.png
agent-browser eval 'closeModal()'
agent-browser wait 300
```

全15スキル（id 1〜15）に対して繰り返し。チェック項目：
- [ ] モーダルが表示されるか
- [ ] スキル名・説明・インストールコマンドが表示されるか
- [ ] 「閉じる」で閉じるか

**2d) ☆気になるボタンテスト：**
```bash
agent-browser snapshot -i
# ☆ボタンのrefをクリック
agent-browser click @e33
agent-browser wait 300
agent-browser screenshot C:/Users/yyama/skills_asaichi/ux-test/07-star.png
```

チェック項目：
- [ ] ☆ → ★ にスタイル変化するか
- [ ] トーストメッセージが表示されるか

**2e) セットアップガイド（赤バナー）テスト：**
```bash
agent-browser scroll up 9999
agent-browser click @e2   # セットアップガイドバナー
agent-browser wait 500
agent-browser screenshot C:/Users/yyama/skills_asaichi/ux-test/08-setup-guide.png
agent-browser eval 'document.getElementById("setupModal").classList.remove("open")'
```

チェック項目：
- [ ] セットアップモーダルが表示されるか
- [ ] Node.jsリンク・コマンドコピーボタンが存在するか

**2f) 制作代行フォームテスト：**
```bash
agent-browser scroll down 9999
agent-browser wait 500
agent-browser screenshot C:/Users/yyama/skills_asaichi/ux-test/09-order-form.png
```

チェック項目：
- [ ] 参考料金めやすが表示されているか
- [ ] フォーム入力欄（名前・メール・内容）が存在するか
- [ ] 送信ボタンが存在するか

### ステップ3：モバイル表示テスト

ビューポートをスマートフォンサイズに変更してテストします。

```bash
agent-browser set viewport 375 812
agent-browser scroll up 9999
agent-browser wait 500
agent-browser screenshot C:/Users/yyama/skills_asaichi/ux-test/10-mobile-hero.png
agent-browser scroll down 400
agent-browser screenshot C:/Users/yyama/skills_asaichi/ux-test/11-mobile-nav-tabs.png
agent-browser scroll down 400
agent-browser screenshot C:/Users/yyama/skills_asaichi/ux-test/12-mobile-cards.png
agent-browser scroll down 9999
agent-browser screenshot C:/Users/yyama/skills_asaichi/ux-test/13-mobile-form.png
```

チェック項目：
- [ ] ナビゲーションが横に崩れていないか
- [ ] タブのテキストが読めるか（回転していないか）
- [ ] カードが1列に収まっているか
- [ ] フォームが画面幅に収まっているか
- [ ] フォントサイズが14px以上か（高齢者向け）
- [ ] ヒーロー画像が画面を占有しすぎていないか

### ステップ4：UXチェックリスト評価

以下の6項目を評価し、合格/不合格を判定します。

| # | チェック項目 | 合格基準 |
|---|-------------|----------|
| 1 | モバイルナビ・タブの表示 | テキストが正しく横書きで読める |
| 2 | モバイルのファーストビュー | スクロールなしでコンテンツが見える |
| 3 | 制作代行フォームの料金表示 | フォーム付近に料金めやすが見える |
| 4 | カードタップのフィードバック | クリック/タップ時に視覚的な反応がある |
| 5 | フォントサイズ | 本文が14px以上 |
| 6 | セットアップバナーの制御 | 「非表示にする」が記憶される |

### ステップ5：レポート出力

テスト結果を以下の形式でまとめます。

```
# スキル朝市 UXテストレポート
日時: YYYY-MM-DD HH:MM

## テスト結果サマリー
- デスクトップ: ○件合格 / ○件不合格
- モバイル: ○件合格 / ○件不合格

## 問題点一覧
| # | 問題 | 深刻度 | スクショ | 改善案 |
|---|------|--------|----------|--------|
| 1 | ... | 高/中/低 | ux-test/XX.png | ... |

## スクリーンショット一覧
（撮影した全画像のパスと説明）
```

### ステップ6：ブラウザを閉じる

```bash
agent-browser set viewport 1280 720   # デフォルトに戻す
agent-browser close
```

## 重要な注意点

- agent-browserがインストールされている必要があります（`npm i -g agent-browser`）
- ローカルファイルを開くには `--allow-file-access` フラグが必要です
- スナップショットのref（@e1, @e2...）はページ変更のたびに変わります。クリック前に必ず `snapshot -i` で最新のrefを取得してください
- モーダルはJSの `openModal(id)` で開き、`closeModal()` で閉じます（カードのonclickがagent-browserのクリックでは反応しにくいため）
- テスト画像は `C:/Users/yyama/skills_asaichi/ux-test/` に保存されます
- テスト後は必ず `agent-browser close` でブラウザを閉じてください
