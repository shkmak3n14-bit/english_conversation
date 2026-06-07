# 生成AI英会話アプリ

## 概要

生成AI英会話アプリは、音声入力と生成AI（OpenAI / LangChain）を組み合わせて、英会話を練習できる Streamlit アプリです。

以下の 3 つのモードで、リスニング・スピーキング・ライティングをバランスよく鍛えることを目的としています。

- 日常英会話: 英語講師AIとの自由会話（音声入出力）

- シャドーイング: AIが読み上げた英文を真似して発話し、精度を評価

- ディクテーション: AIが読み上げた英文を書き取り、正確さを評価

## 機能一覧

### 共通機能

- 音声入力: ブラウザ上で録音し、Whisper（`whisper-1`）で英語テキストに変換

- 音声出力: OpenAI TTS（`tts-1`）で英語音声を生成し、PyAudio で再生

- 会話履歴管理: LangChain の `ConversationSummaryBufferMemory` による要約付きメモリ

- 英語レベル選択: 初級者 / 中級者 / 上級者（現状は UI のみ、ロジック拡張前提）

- 再生速度調整: 0.6〜2.0倍速で音声再生速度を変更可能

## モード別機能

### 1. 日常英会話モード（`MODE_1`）

- 役割: 英語講師としての自由会話

- プロンプト: `SYSTEM_TEMPLATE_BASIC_CONVERSATION`

    - 文法ミスをさりげなく訂正しつつ、自然な会話を継続

    - 必要に応じて会話後に簡単な説明も可能

- フロー:

    1. ユーザーが音声で発話

    2. Whisper で文字起こし

    3. LangChain `ConversationChain` で応答生成

    4. 応答をTTSで音声化し再生

    5. 画面にテキストも表示

### 2. シャドーイングモード（`MODE_2`）

- 役割: 聞こえた英文を真似して発話するトレーニング

- 問題文生成プロンプト: `SYSTEM_TEMPLATE_CREATE_PROBLEM`

    - 約15語の自然な英文（カジュアル会話・ビジネス・友人同士など）

- 評価プロンプト: `SYSTEM_TEMPLATE_EVALUATION`

    - 単語の正確性

    - 文法的な正確性

    - 文の完成度

    - 日本語でフィードバック + 励ましコメント

- フロー:

    1. AIが英文を1文生成し、音声で読み上げ

    2. ユーザーがそれを真似して発話

    3. Whisperで文字起こし

    4. LLMが元の英文とユーザー発話を比較し、評価コメントを生成

    5. 評価結果をチャット欄に表示

### 3. ディクテーションモード（`MODE_3`）

- 役割: 聞こえた英文を書き取るトレーニング

- 問題文生成: シャドーイングと同じく `SYSTEM_TEMPLATE_CREATE_PROBLEM`

- 評価: `SYSTEM_TEMPLATE_EVALUATION` を用いて、書き取りの正確性を分析

- フロー:

    1. AIが英文を生成し、音声で読み上げ

    2. ユーザーは画面下部のチャット欄に英文を入力

    3. LLMが問題文と入力文を比較し、日本語で評価 + アドバイスを表示

## 画面構成・操作

### 上部コントロール

- 開始ボタン

    - モードに応じた処理を開始

- 再生速度

    - `2.0 / 1.5 / 1.2 / 1.0 / 0.8 / 0.6`

- モード選択

    - 「日常英会話」「シャドーイング」「ディクテーション」

    - モード変更時に関連フラグやカウンタをリセット

- 英語レベル

    - 「初級者」「中級者」「上級者」

### メッセージ表示

- `st.session_state.messages` に会話履歴を保持

    - `role="assistant"`: AIメッセージ（AIアイコン）

    - `role="user"`: ユーザーメッセージ（ユーザーアイコン）

    - `role="other"`: 区切り用

### モード専用UI

- シャドーイング中: 「シャドーイング開始」ボタン

- ディクテーション中: 「ディクテーション開始」ボタン + チャット入力欄

- ディクテーション時のみ、チャット入力が有効（それ以外は送信不可）

## 技術構成

### 使用ライブラリ・サービス

- フロントエンド / アプリ基盤

    - Streamlit

- 音声処理

    - `audiorecorder`: ブラウザ録音

    - `pydub`: 音声フォーマット変換・速度変更

    - `pyaudio`: 音声再生

    - `wave`: WAVファイル操作

- LLM / 音声モデル

    - OpenAI API

        - Chat: `gpt-4o-mini`

        - 音声認識: `whisper-1`

        - 音声合成: `tts-1`

- LLMオーケストレーション

    - LangChain

        - `ChatOpenAI`

        - `ConversationChain`

        - `ConversationSummaryBufferMemory`

        - `ChatPromptTemplate` など

### 主なファイル

- `main.py`

    - Streamlit アプリ本体

    - 画面構成、モード制御、状態管理（`st.session_state`）

- `constants.py`

    - アプリ名、モード名、ディレクトリ、プロンプトテンプレートなどの定数

- `functions.py`

    - 音声録音・再生・変換

    - Whisper 文字起こし

    - 問題文生成 + 音声再生

    - 評価用チェーン生成

## セットアップ

### 1. 必要環境

- Python 3.9+

- OpenAI API キー

- マイク入力・音声出力が可能な環境

### 2. 依存ライブラリ（例）

```bash
pip install streamlit openai langchain langchain-openai python-dotenv pydub pyaudio audiorecorder scipy
```

`pyaudio` は環境によって追加のセットアップが必要な場合があります。

### 3. 環境変数

`.env` ファイルなどで OpenAI API キーを設定します。

```bash
OPENAI_API_KEY=your_api_key_here
```

### 4. ディレクトリ構成（例）

```text
project_root/
    main.py
    functions.py
    constants.py
    images/
        ai_icon.jpg
        user_icon.jpg
    audio/
        input/
        output/
    .env
```

`audio/input` と `audio/output` は、起動時および保存時に自動作成されます。

## 起動方法

```bash
streamlit run main.py
```

ブラウザが立ち上がったら、次の順で操作します。

1. モードと再生速度、英語レベルを選択

2. 「開始」ボタンを押す

3. 各モードの指示に従って、発話または入力を行う

## 今後の拡張アイデア

- 英語レベルに応じた文の難易度制御

- 学習履歴に基づく弱点フィードバックの蓄積

- 単語・表現ごとのスコアリング・可視化

- モバイル環境向けUI最適化
