# Daily Report: 2026-02-08

## プロジェクト: YouTube Transcriber Tool

### 1. 目的
YouTubeチャンネルの動画から音声を抽出し、文字起こしを行う自動化ツールを作成する。
実行環境のスペック（CPUのみ、GPU弱）とコスト（API無料枠活用）を考慮し、最適化されたパイプラインを構築する。

### 2. 成果物
- **プロジェクトディレクトリ**: `~/workspace/for_gemini/youtube_transcriber/`
- **データ保存先**: `/mnt/videos/youtube_audio_data/`
    - チャンネルごとにフォルダ分けして音声とメタデータを保存
- **主な機能**:
    1. **字幕優先**: YouTubeの字幕（自動生成含む）があれば最優先で取得（最速・無料）。
    2. **Gemini 2.0 Flash**: 字幕がない場合、Gemini APIの無料枠を使用して高速・高精度に文字起こし。
    3. **ローカルフォールバック**: API制限やネットワークエラー時は、自動的にローカルの `faster-whisper` (int8量子化) に切り替え。

### 3. 技術スタック & 設定
- **言語**: Python 3.12 (venv環境)
- **ライブラリ**:
    - `yt-dlp`: 動画情報取得、音声・字幕ダウンロード
    - `google-genai`: 最新のGemini APIクライアント
    - `faster-whisper`: 高速なローカルWhisper実装
    - `ffmpeg`: 音声変換処理
- **採用モデル**: `gemini-2.0-flash` (旧 `1.5-flash` から変更)

### 4. 発生した課題と解決策
| 課題 | 詳細 | 解決策 |
| :--- | :--- | :--- |
| **権限エラー** | `/mnt/videos` への書き込み不可 | ユーザーによる `chown` 実行で解決 |
| **APIモデルエラー** | `gemini-1.5-flash` が 404 Not Found | モデル一覧を調査し、最新の `gemini-2.0-flash` を採用 |
| **ライブラリ非推奨** | `google-generativeai` が Deprecated | 推奨される新ライブラリ `google-genai` へ移行しコードを書き換え |
| **YouTube制限** | 字幕取得時に `HTTP 429 Too Many Requests` | エラーを捕捉し、即座に「音声DL + AI解析」フローへ移行するロジックを追加 |

### 5. 実行方法
```bash
cd youtube_transcriber
source .venv/bin/activate
python main.py "https://www.youtube.com/watch?v=VIDEO_ID"
```

### 6. 今後の展望
- チャンネルURLを指定しての全動画一括処理
- 文字起こし結果の要約（Geminiを活用）
- 長時間の動画に対する分割処理の最適化
