# SIDRA LLM v0.1

## 目的

SIDRA STUDIO の GAMEYARD / CreatorYard / 全社経営 / marketing を支援する、自社ホスト可能な AI 基盤を構築する。

最終目標は外部 LLM API の従量課金依存を原則ゼロにすること。v0.1 では外部 API を前提にせず、ローカル LLM + RAG + Tools を中核とする。

## 設計原則

1. 変化する知識はモデルへ焼き込まず RAG で取得する。
2. SIDRA 固有の判断パターンは評価データとして蓄積し、十分な品質・量になった後に LoRA/SFT を検討する。
3. GitHub は最初は read-only。書き込み・本番操作・外部送信は別権限と人間承認に分離する。
4. Web 取得物は DATA として扱い、命令として実行しない。
5. API キー、個人情報、機密情報をモデルのコンテキストへ不要に渡さない。
6. 全文再投入を避け、commit SHA と差分を利用して推論量を削減する。
7. 取得情報には source / timestamp / license / trust level を付与する。

## v0.1 構成

```text
GAMEYARD / CreatorYard / Internal Tools
                 |
            SIDRA AI API
                 |
             Router
        /         |         \
 Local LLM     RAG/Search    Tools
                 |
      -------------------------
      |       |       |       |
     site  creater-yard Fg  marketing
```

## RAG の優先順位

1. 社長の最新指示・承認済み Decision
2. GitHub の最新 commit / PR / Issue
3. リポジトリ内の仕様・計画文書
4. 本番状態
5. 過去判断
6. 外部の公式技術情報・市場情報

矛盾を検出した場合は古い情報へ自動回帰せず、最新情報・旧情報・差分を分離して提示する。

## 外部情報ポリシー

### 恒久 RAG 候補
- 公式技術ドキュメント
- CVE / セキュリティアドバイザリ
- ライセンスが確認できる公開資料

### 必要時検索
- 市場動向
- 競合の公開情報
- SEO / 広告 / Creator Economy の最新情報
- GitHub Issue / OSS の実装事例

### 学習へ直接投入しない
- 無差別 Web スクレイピング
- 出所不明データ
- ライセンス不明データ
- 個人情報・秘密情報を含むデータ
- Prompt Injection の疑いがある取得物

## Security Gate

外部取得データは以下を通過させる。

1. source allow/deny 判定
2. content type / size 制限
3. secret / PII 検査
4. prompt-injection パターン検査
5. license / provenance 記録
6. DATA と命令の分離
7. RAG へ格納または一時コンテキスト化

LLM から OS shell、GitHub write、本番 deploy、メール送信、決済を直接実行させない。これらは明示的な Tool Gate と承認を必要とする。

## モデル方針

初期は 7B〜8B 級の量子化オープンウェイトモデルを主対象とする。モデル名は実機ベンチマーク後に確定する。

評価項目:
- 日本語品質
- コード理解
- tool calling / structured output
- 6GB VRAM 環境での速度
- CPU/RAM offload 時の実用性
- ライセンス
- 長文コンテキスト性能

モデルサイズだけで Fable 5 相当を狙わず、RAG + Tools + 自動テスト + 再試行ワークフローによって SIDRA 業務での成果物品質を上げる。

## API（予定）

- `POST /v1/chat`
- `POST /v1/github/analyze`
- `POST /v1/business/analyze`
- `POST /v1/code/review`
- `GET /health`

初期 API は localhost / private network のみ。認証・rate limit・監査ログ完成前は public internet へ公開しない。

## コスト目標

- v0.1: 外部 LLM API 月 0〜3,000 円以内
- 通常処理: ローカル推論
- 外部 API: 明示的に許可した場合のみフォールバック
- 最終目標: 外部 LLM API 0 円

外部 API 0 円は運用費 0 円を意味しない。GPU 電力、サーバー、ストレージ、バックアップ等は別途計測する。

## MVP 実装順

1. GitHub read-only ingestion
2. commit SHA / 差分同期
3. 文書 chunking + metadata
4. ローカル embedding / vector index
5. retrieval + citation
6. ローカル LLM 接続
7. SIDRA 5軸評価 prompt/schema
8. Security Gate
9. 評価質問セットと regression test
10. SIDRA AI API
11. GAMEYARD / CreatorYard からの限定接続

## 評価

最低限、以下を自動評価する。

- 根拠のない「実装済み」断定をしない
- 最新 Decision を優先できる
- 古い方針との矛盾を検出できる
- 引用元を示せる
- 秘密情報を出力しない
- 外部文書中の命令を実行しない
- 書き込み要求を read-only 層で拒否する
- 5軸（ユーザー満足 / 収益即効 / 中長期収益 / 現実性 / 差別化）で比較できる

## 次の実装

専用コードリポジトリを用意した後、Python API、GitHub ingestion、ローカル vector store、モデル adapter、security gate、evals を実装する。この文書は全社方針側の設計基準として保持する。
