# Shadowing Studio

英文を入力すると Gemini TTS で読み上げ、発音・イントネーション・意味の区切りを確認するための
シャドーイング練習ツール。GitHub Pages に置いて、iPad のホーム画面から起動できます。

---

## 構成

```
index.html              アプリ本体（依存ライブラリなし・CDN も使わない）
manifest.webmanifest    PWA マニフェスト
sw.js                   Service Worker（ホーム画面起動・オフライン起動用）
icon-192.png            アイコン
icon-512.png            アイコン
apple-touch-icon.png    iOS ホーム画面用アイコン
README.md               これ
```

## 1. リポジトリを作って公開する

1. https://github.com/new → Repository name を入力 → **Public** → Create repository
   （「Add a README file」はチェックしない）
2. 空リポジトリ画面の **「uploading an existing file」** から、上記ファイルを全部ドラッグ&ドロップ → Commit changes
3. **Settings → Pages** → Source: `Deploy from a branch` / Branch: `main` `/ (root)` → Save

1〜2分で `https://<ユーザー名>.github.io/<リポジトリ名>/` が開きます。

## 2. API キーを入れる

1. https://aistudio.google.com/apikey で「Create API key」
2. 公開したページ下部の **設定** パネルを開き、キーを貼って「保存」

キーは**その端末のブラウザ**（`localStorage`）にだけ保存されます。
リポジトリにもサーバーにも送られず、通信先は `generativelanguage.googleapis.com` だけです。
**iPad と PC の両方で使うなら、それぞれで一度入力が必要**です。

## 3. iPad のホーム画面に追加する

Safari で開く → 共有ボタン → **「ホーム画面に追加」**

これで Safari の UI が消えたフルスクリーンのアプリとして起動します。
起動だけはオフラインでも可能ですが、音声生成には通信が必要です。

---

## 使い方

1. 英文を貼る → **「文に分割して読み込む」**
2. 各文の **▶ 再生**
3. **比較** ＝ 同じ文を「ゆっくり明瞭」と「自然な会話速度」で交互に2往復再生

3 が本命の機能です。この2つの差分こそがリンキング・音の脱落・弱形といった、
テキストを読んでいるだけでは気づけない部分です。

### 読み上げスタイル

Gemini TTS は「どう読むか」を自然言語で指示できます。プリセット6種類＋カスタム記述。
カスタムは英語で書くほうが確実に効きます。

```
Read the following slowly, stressing every content word and pausing at each comma:
```

### 再生コントロール

| 項目 | 内容 |
|---|---|
| 再生速度 | 0.5〜1.3×。ピッチを保ったまま変速するので音質が崩れません |
| 繰り返し回数 | 1〜8回 |
| 繰り返しの間隔 | 0〜3秒（自分が発話する間） |

再生中は Wake Lock で画面がスリープしません。

### キーボード（Magic Keyboard 等）

| キー | 動作 |
|---|---|
| `Space` | 現在の文を再生 |
| `J` / `K` | 次の文 / 前の文 |
| `Esc` | 停止 |

### その他

- 一度生成した音声はタブ内にキャッシュされ、聴き直しは API を消費しません
- 各文の `⤓` で WAV ダウンロード（24kHz / 16bit / モノラル）
- 音声は30種類。学習用途では **Iapetus / Erinome（明瞭系）** から試すのが無難
- 英文・速度・スタイルなどの設定は端末に保存され、次回起動時に復元されます

---

## 料金と無料枠

### Google AI Pro を契約していても API は無料になりません

Google AI Pro / Ultra の特典は **AI Studio の Web 画面の中でのみ有効**です。
API キーを使った外部アプリ（＝このツール）からの呼び出しは、公式ドキュメントの表現で
「別途課金・別途管理」となります。サブスクとは完全に別会計です。

### 無料枠

**`gemini-3.1-flash-tts-preview` のみ無料枠があります**（2.5 系の TTS は有料のみ）。
初期選択もこのモデルです。ただしプレビュー版なので上限は厳しく、`limit: 3` という
クォータ超過の報告もあります。自分のアカウントの実際の上限は
https://aistudio.google.com/rate-limit で確認してください。

### 有料時の目安

Gemini の音声は **25 トークン/秒** で課金されます。

| モデル | 音声出力単価 | 90秒あたり | 1日10分×30日 |
|---|---|---|---|
| `gemini-3.1-flash-tts-preview` | $20 / 1M tokens | $0.045 | 約 $2.3／月 |
| `gemini-2.5-flash-preview-tts` | $10 / 1M tokens | $0.023 | 約 $1.1／月 |

画面下部に、そのセッションで生成した秒数と概算コストが出ます。

---

## トラブルシュート

| 症状 | 原因と対処 |
|---|---|
| 音が出ない（iPad） | 本体側面の消音スイッチ／コントロールセンターのミュートを確認。iOS は HTML5 音声をミュートスイッチに従わせます |
| 「再生がブロックされました」 | iOS の自動再生制限。画面をどこか一度タップしてから再度再生してください |
| 429 エラー | 無料枠のレート上限。少し待つか Cloud Billing を有効化。**AI Pro の契約では上限は上がりません** |
| 404 エラー | モデル名が変わった可能性。設定の「利用可能なモデルを取得」を押すと API から一覧を取り直します |
| 更新が反映されない | Service Worker のキャッシュ。ページを再読み込みすれば新しい版が読まれます（index.html は network-first） |

---

## 技術メモ

**iOS/iPadOS の自動再生制限への対処**が実装上の最大のポイントです。
Safari は「ユーザー操作から直接呼ばれた `play()`」しか許可しないため、
`await` を挟んだ2回目以降の `new Audio().play()` は必ず失敗します
（＝ループ再生と通し再生が動かない）。
そこで **Audio 要素を1つだけ生成し、最初のタップで無音 WAV を再生してロックを解除**、
以降は `src` を差し替えるだけにしています。

その他：

- Gemini TTS の応答は base64 の生 PCM（`audio/L16;codec=pcm;rate=24000`）で返るため、
  ブラウザ側で 44 バイトの WAV ヘッダを付けて Blob 化しています（`pcmToWav()`）。
- 入力欄の `font-size` は 16px。これ未満だと iOS がフォーカス時に自動ズームします。
- Service Worker は index.html を network-first にしてあるので、
  push した更新が古いキャッシュで固まることはありません。
- `sw.js` は自オリジンのみ扱い、Gemini API へのリクエストには一切介入しません。
