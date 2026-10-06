# SD画像 → 動画

Stable Diffusion で作ったメタデータ入りの画像から、ローカルLLMで動画用プロンプトを作り、ComfyUI で動画を生成する Web アプリです。

- 画面は GitHub Pages 上にありますが、**処理はすべて使う人のPCの中**で行われます（ComfyUI とローカルLLMに、ブラウザから直接つなぎます）。
- 画像・プロンプト・設定が外部のサーバーに送られることはありません。設定はブラウザ内（localStorage）に保存されます。
- 動画モデルは、ComfyUI で動くワークフローなら何でも使えます（MiniMax H3、Wan など）。
- 文章AIは、OpenAI互換サーバーなら何でも使えます（LM Studio / Bionic、Ollama、llama.cpp など）。

---

## 公開する人向け（GitHub Pages で公開する手順）

1. GitHub にログインし、右上の「＋」→「New repository」
2. リポジトリ名（例: `sd2video`）を入れ、**Public** を選んで「Create repository」
3. 「uploading an existing file」から `index.html` と `README.md` をアップロードして「Commit changes」
4. リポジトリの「Settings」→ 左の「Pages」
5. 「Build and deployment」の Source を「Deploy from a branch」、Branch を「main」「/ (root)」にして「Save」
6. 1〜2分後、同じ画面の上部に公開URLが表示されます（例: `https://あなたのID.github.io/sd2video/`）

このURLを、使ってほしい人に伝えてください。更新するときは `index.html` を差し替えるだけです。

---

## 使う人向け（最初の1回だけの準備）

使う前に、このページからの接続を自分のPCの ComfyUI とLLMに許可する必要があります。ページを初めて開くと「はじめに」画面が出て、**あなたの環境で必要なコマンドがそのまま表示されます**。

### 1. ComfyUI の起動オプションを追加する

起動オプションに次を追加して、ComfyUI を起動し直します。`https://あなたのID.github.io` の部分は、ページの「はじめに」に表示される値をコピーしてください（URLの末尾のリポジトリ名は含めません）。

```
--enable-cors-header https://あなたのID.github.io
```

ポータブル版の例（`run_nvidia_gpu.bat` をメモ帳で開き、1行目の末尾に追記）:

```
.\python_embeded\python.exe -s ComfyUI\main.py --windows-standalone-build --enable-cors-header https://あなたのID.github.io
```

> この設定は「そのサイトがあなたの ComfyUI を操作してよい」という許可です。信頼できるサイトだけを指定してください。

### 2. ローカルLLMのサーバーを起動し、CORS を有効にする

- **LM Studio**: 「Developer」でサーバーを起動し、サーバー設定の「Enable CORS」をオン
- **Bionic**: Bionic 単体では外部ツール向けのサーバーが開いていない場合があります。その場合は LM Studio でサーバーを起動してください
- **Ollama**: 環境変数 `OLLAMA_ORIGINS` に `https://あなたのID.github.io` を設定して起動し直す

### 3. ブラウザは Chrome または Edge

初回の接続時に「ローカルネットワーク上のデバイスへのアクセス」の確認が出たら「許可」を押します。Safari では動かない場合があります。

### 4. ページで設定する

1. 「設定」→「文章AI（LLM）」→「🔍 自動で探す」→「画像対応をテスト」→「保存して使う」
2. ComfyUI で、ふだん使っている画像→動画のワークフローを開き、メニューの「ワークフロー」→「エクスポート (API)」で JSON を保存
3. 「設定」→「動画（ComfyUI）」→ URL を確認 → JSON を読み込む →「保存して使う」

## 毎日の使い方

1. 画像をドロップ
2. 「実行」→ プロンプト作成から動画生成まで自動で進みます
3. 動画が画面に表示されます（「動画を保存」「プロンプトを保存」でダウンロード）
4. 気に入らなければ、プロンプトを手で直すか「AIで修正」→「このプロンプトで動画を作り直す」

## スキル

プロンプトを作るときに文章AIへ追加で渡す指示です。メイン画面のボタンでON/OFFします。

- Agent Skills 形式（`SKILL.md`）の読み込み・書き出しに対応しています。Bionic などで使っている SKILL.md もそのまま読み込めます
- スキルはこのアプリが文章AIへの指示に追加する仕組みです。Bionic 本体のスキル機能を呼び出すものではありません

## うまくいかないとき

| 表示 | 対処 |
|---|---|
| 「接続が許可されていません（CORS）」 | 上の手順1・2の許可設定を確認し、ComfyUI / LLM を起動し直す |
| 「接続できません」 | 起動しているか、URL（ポート番号）が正しいか確認。ブラウザの「ローカルネットワークへのアクセス」を許可したか確認 |
| 「モデルファイル名が見つからない」 | ComfyUI 側でワークフローが動くか確認し、もう一度「エクスポート (API)」で書き出して登録し直す |
| 動画生成がとても遅い | 文章AIと動画が同じGPUを取り合っています。動画生成前に LM Studio / Bionic でモデルを取り外すと速くなります |
| 設定を別のPCに移したい | 「設定」→「バックアップ・その他」で書き出し・読み込み |
