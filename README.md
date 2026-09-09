# Shadowing Studio

英文を入力すると Gemini TTS で読み上げ、発音・イントネーション・意味の区切りを確認するための
シングルファイル Web アプリ。GitHub Pages にそのまま置けます。

---

## 1. リポジトリを作る

GitHub で新規リポジトリ（例：`shadowing-studio`）を **Public** で作成し、
`index.html` をリポジトリ直下に置いて push します。

```bash
git init
git add index.html README.md
git commit -m "Add shadowing studio"
git branch -M main
git remote add origin https://github.com/<あなたのID>/shadowing-studio.git
git push -u origin main
```

## 2. GitHub Pages を有効化

リポジトリの **Settings → Pages** で

- Source: `Deploy from a branch`
- Branch: `main` / `/ (root)`

を選んで Save。1〜2分で `https://<あなたのID>.github.io/shadowing-studio/` が公開されます。

> Public リポジトリなので `index.html` の中身は全世界から読めます。
> **API キーはこのファイルに書かないでください。** キーは下記のとおりブラウザ側で入力します。

## 3. Gemini API キーを取得

1. https://aistudio.google.com/apikey を開く
2. 「Create API key」でキーを発行（無料枠があります）
3. 公開したページを開き、下部の **設定** パネルにキーを貼って「保存」

キーは `localStorage` にのみ保存されます。リポジトリにも、こちらのサーバーにも送られません。
通信先は `generativelanguage.googleapis.com` だけです。

**推奨：** Google Cloud コンソールで、そのキーに割り当てられたプロジェクトの
使用量アラート／上限を設定しておいてください。キーが端末に置かれる方式である以上、
共有PCでは使わないのが前提です。不要になったら「キーを削除」で消せます。

---

## 使い方

1. 英文を貼る → **「文に分割して読み込む」**
2. 各文の **▶ 再生** で読み上げ
3. **比較** ボタン＝同じ文を「ゆっくり明瞭」と「自然な会話速度」で交互に2往復再生

3 が本命の機能です。この2つの差分こそがリンキング・音の脱落・弱形といった、
テキストを読んでいるだけでは絶対に気づけない部分なので、そこを集中的に聴いてください。

### スタイル指示

Gemini TTS は「どう読むか」を自然言語で指示できます。プリセットは6種類、
「カスタム指示…」で自由記述もできます（英語で書くほうが確実に効きます）。

```
Read the following slowly, stressing every content word and pausing at each comma:
```

### 再生コントロール

| 項目 | 内容 |
|---|---|
| 再生速度 | 0.5〜1.3×。ピッチを保ったまま変速するので音質が崩れません |
| 繰り返し回数 | 1〜8回。シャドーイング用 |
| 繰り返しの間隔 | 0〜3秒。自分が発話する間 |

### キーボード

| キー | 動作 |
|---|---|
| `Space` | 現在の文を再生 |
| `J` / `K` | 次の文 / 前の文 |
| `Esc` | 停止 |

### その他

- 一度生成した音声はタブ内にキャッシュされるので、聴き直しは無課金
- 各文の `⤓` で WAV ダウンロード（24kHz / 16bit / モノラル）
- 音声は30種類。学習用途では **Iapetus / Erinome（明瞭系）** から試すのが無難

---

## 料金と無料枠

### Google AI Pro を契約していても API は無料になりません

Google AI Pro / Ultra の特典は **AI Studio の Web 画面の中でのみ有効**です。
API キーを使った外部アプリ（＝このツール）からの呼び出しは、公式ドキュメントの表現で
「別途課金・別途管理」となります。サブスクとは完全に別会計です。

### 無料枠

**`gemini-3.1-flash-tts-preview` のみ無料枠があります**（2.5 系の TTS は有料のみ）。
このツールの初期選択もこのモデルにしてあります。ただしプレビュー版のため上限はかなり厳しく、
`limit: 3` というクォータ超過の報告もあります。自分のアカウントの実際の上限は
https://aistudio.google.com/rate-limit で確認してください。

無料枠を超えると 429 エラーになります。継続的に使うなら Cloud Billing の有効化が必要です。

### 有料時の目安

Gemini の音声は **25 トークン/秒** で課金されます。

| モデル | 音声出力単価 | 90秒あたり | 1日10分×30日 |
|---|---|---|---|
| `gemini-3.1-flash-tts-preview` | $20 / 1M tokens | $0.045 | 約 $2.3／月 |
| `gemini-2.5-flash-preview-tts` | $10 / 1M tokens | $0.023 | 約 $1.1／月 |

画面右下に、そのセッションで生成した秒数と概算コストが出ます。
一度生成した音声はタブ内にキャッシュされるので、聴き直しはリクエストを消費しません。

## モデル名について

プレビュー版モデルは名前が変わることがあります。404 エラーが出たら、
設定パネルの **「利用可能なモデルを取得」** を押してください。
API から TTS 対応モデルの一覧を取得してドロップダウンを更新します。

## 技術メモ

- Gemini TTS の応答は base64 の生 PCM（`audio/L16;codec=pcm;rate=24000`）で、
  そのままでは `<audio>` で再生できません。ブラウザ側で 44 バイトの WAV ヘッダを
  付けて Blob 化しています（`pcmToWav()`）。
- ビルド不要・依存ライブラリなしの単一 HTML。外部 CDN も読み込みません。
